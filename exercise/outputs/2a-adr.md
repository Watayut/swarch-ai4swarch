Agent: Claude Code (model: Claude Fable 5.1) — run **a** (zero-shot, context: none)

# ADR-0002: Publish an "OrderPlaced" event and run post-order actions as independent subscribers

- **Status:** Proposed
- **Date:** 2026-10-21
- **Deciders:** Online Boutique architecture team (names not given — see Open questions)

## Context and problem

The requirement states that after an order is placed, Online Boutique must:

1. send the order confirmation email through **MailFast**, a new external email provider, replacing the mock `emailservice` logic;
2. award **loyalty points** through a new `loyaltyservice` (to be built);
3. record an **"order placed" analytics event** for the marketing team.

Marketing expects **more post-order actions next year** (SMS, fraud scoring, a recommendations update).

Forces at play:

- Three new post-order actions arrive at once, and the list is explicitly expected to grow. Each new action must not require re-opening and re-testing the order-placement path.
- One action (MailFast) is an **external, third-party** dependency whose availability and latency we do not control. A slow or down email provider must not block or fail order placement.
- The actions differ in criticality: the customer-facing order result should not depend on an analytics event being recorded.
- `emailservice` currently exists only as a mock, so the real sending logic (MailFast integration) has to be written in any case; the question is *where* it is triggered from and *how* it is coupled to order placement.
- Nothing in this run's inputs describes the existing order-placement code, the inter-service protocol, the deployment platform, or an existing message broker. Statements about those are marked **[Assumption]** below; see Open questions.

**[Assumption]** Order placement is performed by a single service (referred to below as the *order-placement service*; commonly `checkoutservice` in Online Boutique) that today calls `emailservice` directly as the last step of placing an order. Not evidenced in the provided inputs (context: none).

The problem to decide: **how should the order-placement step trigger a growing set of post-order actions, so that adding an action is cheap and a failing action cannot break order placement?**

## Decision drivers

Most important first:

1. **Modifiability / extensibility** — adding SMS, fraud scoring, recommendations update (and later actions) should require *no change* to the order-placement service (open–closed principle at the service level). This is stated explicitly by Marketing.
2. **Availability of order placement** — a MailFast outage, a slow `loyaltyservice`, or an analytics sink failure must not cause order placement to fail or hang.
3. **Performance (checkout latency)** — the customer-facing order result should not wait for N downstream actions, especially an external HTTP call to MailFast.
4. **Reliability of side effects** — once an order is placed, each action should eventually happen exactly once from the user's perspective (one email, points awarded once, one analytics event).
5. **Isolation of external dependencies** — MailFast credentials, rate limits and API quirks should be confined to one component.
6. **Cost and operational complexity** — the team must be able to run and observe whatever is introduced; a new infrastructure component is not free.
7. **Testability** — each post-order action should be testable in isolation, and the order flow should be testable without MailFast.

## Considered options

1. **Synchronous fan-out from the order-placement service** — after persisting the order, the order-placement service calls, in sequence or in parallel: a MailFast-backed `emailservice`, the new `loyaltyservice`, and an analytics endpoint. Each new action = a new direct call.
2. **Asynchronous "OrderPlaced" event via a message broker (publish/subscribe)** — the order-placement service publishes one `OrderPlaced` event to a broker topic and returns. `emailservice` (now a MailFast adapter), `loyaltyservice`, and an analytics consumer each subscribe independently. Future actions are new subscribers.
3. **Dedicated post-order orchestrator service** — the order-placement service makes one call to a new `postorderservice`, which then fans out (synchronously or with its own internal queue) to email, loyalty, analytics, and future actions. The orchestrator owns the list of actions.

### Pros and cons of the options

| Option | Pros | Cons |
|---|---|---|
| 1. Synchronous fan-out | Simplest to build with the existing call style; no new infrastructure; easy to trace a single request end-to-end; immediate feedback if an action fails. | Order-placement service must change for **every** new action (poor modifiability — directly contradicts the "more actions next year" force). Checkout latency grows with each action and with MailFast's external latency (performance). A down or slow dependency can fail or delay order placement unless each call is wrapped in timeouts/fallbacks, which must be re-implemented per call (availability). Retry and partial-failure semantics ("email sent but points not awarded") have to be hand-coded (reliability). |
| 2. Async event + subscribers | Order-placement service is **closed for modification**: new actions are new subscribers, deployed independently (modifiability). Order placement returns as soon as the event is published; MailFast latency and outages are absorbed by the subscriber (performance, availability). Broker retention/redelivery gives built-in retry per subscriber; one subscriber failing does not affect the others (reliability, fault isolation). Each subscriber is a small, independently testable service; MailFast is confined to the email subscriber (isolation, testability). Analytics gets a durable event stream it can replay. | Introduces a **message broker** — a new stateful infrastructure component to deploy, secure, monitor and pay for (cost, operational complexity). Eventual consistency: the email/points arrive slightly after the order result; the UI cannot promise "points awarded" synchronously. Needs an event schema and versioning discipline. Requires **idempotent** subscribers because brokers typically give at-least-once delivery (duplicate email / double points otherwise). Dual-write risk between "order saved" and "event published" needs an outbox or equivalent. Harder end-to-end tracing across the async boundary. |
| 3. Post-order orchestrator service | Order-placement service changes only once (one call to the orchestrator); the list of actions lives in one place that is easy to read and reason about; can enforce ordering/dependencies between actions (e.g. fraud score *before* email). No broker needed if the orchestrator calls targets directly. | Moves the fan-out problem rather than solving it: the orchestrator becomes the new change hotspot and a single point of failure for all post-order actions (modifiability only partly improved; availability concentrated). If synchronous, it still couples checkout latency to the slowest action unless the orchestrator itself becomes async — at which point it needs a queue anyway and converges on Option 2 with an extra hop. Tends to accumulate business logic from many teams (marketing, loyalty, fraud) in one service — organisational coupling. |

## Decision

**Choose Option 2: publish an `OrderPlaced` event and implement each post-order action as an independent subscriber.**

Rationale against the drivers:

- **Modifiability (driver 1):** the explicit expectation of SMS, fraud scoring and recommendations next year is the deciding force. With Option 2 each of those is a new subscriber with zero change to the order-placement service; with Options 1 and 3 some central component must be edited and redeployed each time.
- **Availability and performance (drivers 2–3):** the external MailFast call is taken off the order-placement path entirely. A MailFast outage results in delayed emails, not failed orders.
- **Reliability (driver 4):** per-subscriber redelivery plus idempotent handlers gives "effectively once" per action without hand-written retry code in checkout.
- **Isolation (driver 5):** MailFast credentials and API handling live only in the rewritten `emailservice` subscriber.

Option 3 is rejected because it recreates a central change point and, once made asynchronous to meet driver 2, is Option 2 with an extra service. Option 1 is rejected because it fails drivers 1–3 as the action list grows.

Decision details (scope of this ADR):

- The order-placement service publishes exactly one event type, `OrderPlaced`, carrying the data subscribers need (order id, customer contact/email, line items and totals, currency, timestamp, and a unique event id for idempotency). Exact fields are an open question.
- Publication must be reliable relative to the order being recorded: use a **transactional outbox** (write order + outbox row atomically, relay to the broker) *or*, if the order-placement service has no durable store of its own, publish the event *before* returning success and treat publish failure as order-placement failure. Which applies depends on facts not available in this run — see Open questions.
- Three initial subscribers: (a) `emailservice` rewritten as a MailFast adapter, (b) new `loyaltyservice`, (c) an analytics subscriber that forwards the event to the marketing team's analytics sink. Each subscriber consumes from its own subscription/consumer group so it has independent retry and offset.
- Every subscriber is **idempotent** on the event id (e.g. store processed event ids; MailFast send keyed by order id if the provider supports idempotency keys).
- Each subscriber has a **dead-letter** destination and an alert when messages land there.
- The broker technology is **not** decided here (none is given in the inputs). It is recorded as an open question and should be a follow-up ADR (or an amendment) once team skills, platform, and any already-available broker are known.

## Consequences

- **Positive:** New post-order actions (SMS, fraud scoring, recommendations) are added as new subscribers without touching or redeploying the order-placement service (quality attribute: **modifiability / extensibility**).
- **Positive:** MailFast latency and outages no longer affect whether an order is placed or how fast the customer sees the result (quality attributes: **availability**, **performance**).
- **Positive:** Each subscriber fails, retries and scales independently; one misbehaving action does not affect the others (quality attributes: **fault isolation**, **scalability**).
- **Positive:** The `OrderPlaced` stream can be replayed to backfill analytics or re-send emails after a provider incident (quality attribute: **recoverability**).
- **Positive:** MailFast integration and credentials are confined to one small service (quality attributes: **security**, **testability**).
- **Negative:** A message broker must be provisioned, secured, monitored and paid for; the team must learn to operate it (quality attributes: **cost**, **operability**).
- **Negative:** Email, loyalty points and analytics are **eventually consistent** with the order; the confirmation page cannot truthfully say "points have been awarded", only "will be awarded" (quality attribute: **consistency / usability**).
- **Negative:** Debugging a single order now spans an asynchronous boundary; distributed tracing must propagate a trace/correlation id through the event (quality attribute: **observability**).
- **Negative:** More deployable units (three subscribers instead of one email call) and a schema contract to version (quality attribute: **complexity / maintainability**).
- **Risks and mitigations:**
  - *Duplicate side effects (double email, double points) from at-least-once delivery* → unique event id in every event; subscribers persist processed ids and skip duplicates; use MailFast idempotency keys if available.
  - *Lost event when order is saved but publish fails (dual write)* → transactional outbox with a relay, or publish-before-ack as described in Decision; monitor outbox lag.
  - *Poison messages block a subscriber* → per-subscriber dead-letter queue with alerting and a documented replay procedure.
  - *Schema drift breaks subscribers when `OrderPlaced` changes* → versioned event schema, additive-only changes, consumer-side tolerant readers, contract tests per subscriber.
  - *Sensitive customer data (email address, order contents) flows through the broker* → encrypt in transit, restrict topic ACLs per subscriber, define retention; verify against data-protection constraints (open question).
  - *Broker becomes a new single point of failure* → run it with replication/HA as the platform allows; if the broker is unavailable the order-placement service fails fast or buffers in the outbox rather than silently dropping events.
  - *Future actions that must run **before** the order is accepted (e.g. a blocking fraud decision) do not fit a fire-and-forget event* → scope this ADR to post-acceptance actions; a pre-acceptance fraud check would be a separate synchronous step and its own ADR.

## Validation

- **Extensibility check:** add a trivial fourth subscriber (e.g. a log-only "SMS stub") and confirm it goes live with **zero** code or config change to the order-placement service and no redeploy of it.
- **Availability test:** with MailFast (or a stub for it) returning errors/timeouts, place orders and confirm 100 % still succeed and the user-visible latency does not change; confirm emails are delivered once MailFast recovers.
- **Latency measurement:** compare order-placement latency before and after; the post-change latency must be no worse than before adding the three actions (target numbers are an open question).
- **Idempotency test:** redeliver the same `OrderPlaced` event to every subscriber and confirm exactly one email, one points credit, one analytics record.
- **Outbox / loss test:** kill the broker (or the relay) mid-flow and confirm no order lacks its event after recovery.
- **Operational readiness:** dashboards exist for consumer lag, dead-letter counts and MailFast error rate; an alert fires in a drill.
- **Review date:** revisit this ADR when the first of next year's actions (SMS, fraud scoring, recommendations) is designed, and again if the broker choice ADR changes assumptions made here.

## Open questions

The following were needed to make the decision precise but are not given in the requirement (run a, context: none). They are **not** invented above.

1. Which service places the order today, and does it call `emailservice` synchronously? (Assumed `checkoutservice` and yes.)
2. Is there already a message broker or pub/sub facility in the deployment platform? Which brokers does the team have skills in? (Determines the follow-up broker ADR and cost.)
3. Does the order-placement service have its own durable store (needed to decide outbox vs. publish-before-ack)?
4. Quality targets: acceptable order-placement latency (p50/p99), acceptable delay for the confirmation email and for points to appear, required throughput (orders/s, peak).
5. MailFast facts: API style (REST/SMTP/SDK), rate limits, idempotency-key support, delivery-status webhooks, SLA, contract/cost.
6. `loyaltyservice` facts: who owns it, its data store, points rules, whether points must be visible immediately on the confirmation page.
7. Analytics sink for the marketing team: what system, what event format/schema, retention and privacy requirements.
8. Exact `OrderPlaced` payload fields and whether PII (email address, name, address) may be placed on the broker; data-protection constraints and retention policy.
9. Ordering requirements between actions (e.g. must fraud scoring complete before the email is sent?) and whether any future action is pre-acceptance (blocking) rather than post-order.
10. Who the deciders are, and the team's capacity to operate a broker and three new/changed services.
