**ภาษา:** ภาษาไทย | [English](02-write-adr.en.md)

# งานที่ 2: เขียน ADR สำหรับความต้องการใหม่ (ประมาณ 20 นาที)

> **เชื่อมโยงกับการบรรยาย:** T3 (GPT-4 แบบ zero-shot ให้ผลที่สอดคล้องกัน แต่ *ครบถ้วนน้อยกว่ามนุษย์* ส่วน few-shot ช่วยลดช่องว่างได้มาก), AICH3 (บริบท), สัปดาห์ที่ 10 (message broker)
> **คำถาม:** ADR ที่ AI เขียนดีขึ้นแค่ไหน เมื่อให้ตัวอย่างและบริบทจริงแก่ AI

อ่าน [สถานการณ์ (ภาษาไทย)](../docs/scenario.th.md) ก่อน: ความต้องการ **R-12** ว่าด้วยสิ่งที่เกิดขึ้นหลังจากลูกค้าสั่งซื้อสินค้า

ในแต่ละ run ให้เริ่ม **แชตหรือ session ใหม่** แล้วส่งข้อความด้านล่าง โดยแทนที่ `<paste R-12 here>` ด้วยข้อความ R-12 ภาษาอังกฤษที่อยู่ในกรอบอ้างอิงของ scenario

**Run a: ไม่มีตัวอย่าง (zero-shot)** → [`outputs/2a-adr.md`](outputs/2a-adr.md)
```
Follow prompts/2-write-adr.md
run: a
context: none
requirement: <paste R-12 here>
language: Thai
```

**Run b: มีตัวอย่าง + บริบท** → [`outputs/2b-adr.md`](outputs/2b-adr.md)
```
Follow prompts/2-write-adr.md
run: b
context: docs/scenario.md, docs/adr/0001-use-redis-for-cart-storage.md, system/online-boutique/src/checkoutservice/main.go
requirement: <paste R-12 here>
language: Thai
```

> บรรทัด `language: Thai` ทำให้ agent เขียน ADR เป็นภาษาไทย (หัวข้อของ template และชื่อทางเทคนิคยังคงเป็นภาษาอังกฤษ) หากต้องการภาษาอังกฤษให้ลบบรรทัดนี้ **ใช้ภาษาเดียวกันทั้ง Run a และ Run b** เพื่อให้เปรียบเทียบกันได้

> ใน Run a ให้ตรวจว่า agent ไม่ได้อ่านไฟล์อื่นนอกจาก template จริง ๆ หาก agent เปิด `docs/scenario.md` ให้บันทึกไว้ใน prompt log เพราะจะทำให้การเปรียบเทียบเปลี่ยนไป

> **หมายเหตุของนิสิต:** รันทั้งสอง run เป็นภาษาอังกฤษ (ไม่ใส่ `language: Thai`) เหมือนกันทั้งคู่ และใน Run a agent อ่านเพียง `AGENTS.md`, prompt และ `template.md` ไม่ได้เปิด `scenario.md` (ดู prompt log)

## เปรียบเทียบ a กับ b

| เกณฑ์ | Run a | Run b |
|---|---|---|
| ใช้ตัวเลขจาก scenario หรือไม่ (latency, ช่วงพีก, ข้อจำกัดของ MailFast) | **ไม่ใช้เลย** ไม่มี 500 ms, 300 ms, 200 orders/s, 50 emails/s, 1.5 s, 0.5 % หรือ "ภายใน 1 นาที" ปรากฏ แต่ยก latency target, rate limit ของ MailFast และ SLA ไปไว้ใน Open questions (10 ข้อ) แทนที่จะแต่งขึ้น | **ใช้ครบทุกตัว** และนำไปคำนวณต่อ เช่น งบ latency เหลือ ≈ 200 ms, 300 ms + 1.5 s ทำให้ทางเลือก synchronous ตกทันที, backlog อีเมลโตวินาทีละ 150 ฉบับและระบายได้ 50/s (ระบาย 3 นาทีต่อพีก 1 นาที) |
| ทางเลือกที่พิจารณา (ระบุ) | 3 ทาง: (1) synchronous fan-out จาก order-placement service (2) event `OrderPlaced` ผ่าน message broker แบบ pub/sub **โดยไม่ระบุชื่อ broker** (เลือก) (3) `postorderservice` ที่เป็น orchestrator กลาง | 4 ทาง: (1) synchronous gRPC fan-out ใน `PlaceOrder` (2) goroutine ใน process ของ `checkoutservice` (3) event ผ่าน **RabbitMQ** self-hosted (เลือก) (4) event ผ่าน Kafka และตัด managed cloud queue ออกโดยอ้างข้อจำกัด self-hosted ใน scenario |
| จัดการแต้มสะสมแบบ "ครั้งเดียวเท่านั้น" และ rate limit ของ MailFast หรือไม่ อย่างไร | พูดถึง idempotent subscriber, event id เฉพาะ และ dead-letter queue แบบทั่วไป แต่**ไม่รู้เงื่อนไข "ภายใน 1 นาที"** และ**ไม่จัดการ rate limit** เพราะไม่รู้ตัวเลข ยกเป็นคำถามว่า MailFast มี rate limit และ idempotency key หรือไม่ | จัดการทั้งสองอย่างชัดเจน: `loyaltyservice` de-duplicate ด้วย `order_id` ก่อน ack, durable queue + manual ack ให้ at-least-once แล้วรวมเป็น exactly-once; `emailservice` ใช้ token-bucket ≤ 50/s และ retry 0.5 % ที่ล้มเหลว **แต่** ยังไม่ออกแบบว่าการ dedup กับการบวกแต้มอยู่ใน transaction เดียวกันหรือไม่ (ผลรีวิว AI ในงานที่ 3 ชี้จุดนี้) |
| มีสิ่งที่แต่งขึ้นเองหรือไม่ (ตัวเลข เครื่องมือ หรือข้อเท็จจริงที่ไม่มีใน scenario) | มี: สมมติว่าระบบ "persist order" ก่อน publish (ระบบจริงไม่มี order store) และตั้งชื่อ service ใหม่ `postorderservice` เอง ส่วนตัวเลขไม่แต่งเลย ทำเครื่องหมาย [Assumption] เรื่อง checkout เรียก email แบบ synchronous ซึ่งบังเอิญถูก | มี แต่เป็นการตัดสินใจออกแบบมากกว่าแต่งข้อเท็จจริง: RabbitMQ 3 node แบบ quorum queue, เป้าหมาย "p95 ไม่เกิน 10 % ของ baseline", อ้างว่า prefetch คุม throughput ที่ 50/s ได้ (ผลรีวิว AI ชี้ว่าผิด) และอ้างว่า ADR-0001 ใช้ Locust ในการทดสอบ **ซึ่งไม่มีใน ADR-0001** (ADR-0001 พูดถึง replica ของ cartservice เท่านั้น) |

## รับผิดชอบ ADR ฉบับสุดท้ายของตัวเอง

นิสิตคือสถาปนิก ให้คัดลอก `outputs/2b-adr.md` ไปที่ `docs/adr/0002-<short-title>.md` แล้วแก้ไข**ด้วยตัวเอง**จนพร้อมจะลงชื่อรับรอง ห้ามให้ agent ทำส่วนนี้

- แก้ไขสิ่งที่ผิดหรือแต่งขึ้นเอง และเพิ่มสิ่งที่ขาดหายไป
- ตัดสินใจขั้นสุดท้ายด้วยตัวเอง นิสิตสามารถไม่เห็นด้วยกับ AI ได้

นิสิตจะต้องปกป้อง ADR นี้ต่อหน้าคณะกรรมการรีวิวในส่วน B จึงต้องอธิบายทุกการตัดสินใจใน ADR ได้

**ADR ฉบับสุดท้าย:** [`docs/adr/0002-publish-order-placed-event-to-rabbitmq.md`](../docs/adr/0002-publish-order-placed-event-to-rabbitmq.md) (ร่างของ AI เก็บไว้ที่ `outputs/2b-adr.md` เพื่อเปรียบเทียบ)

| # | สิ่งที่เปลี่ยนจากร่างของ AI | เหตุผล |
|---|---|---|
| 1 | เปลี่ยนกรณี "publish ล้มเหลวหลังตัดเงินแล้ว" จาก open question เป็น**การตัดสินใจ**: retry สั้น ๆ → log event ทั้งก้อนระดับ error + alert → re-publish ด้วยมือ และประกาศว่า transactional outbox **อยู่นอกขอบเขต** ของ ADR นี้ | ร่างของ AI ทิ้งช่องโหว่ด้าน reliability ที่ใหญ่ที่สุดไว้เป็นคำถาม สถาปนิกต้องตัดสินใจเมื่อระบบไม่มี order store ให้ทำ outbox ถ้ารอคำตอบ ADR จะไม่มีวันอนุมัติได้ (คณะกรรมการในส่วน B ไม่เห็นด้วยกับทางเลือกนี้ ดู `04-review-board.md`) |
| 2 | เพิ่ม Decision driver ข้อ 8: สมมติว่าอีเมลยืนยันล่าช้าได้ไม่เกิน **15 นาที** ช่วงพีก 11.11 พร้อมการคำนวณว่าครอบคลุมพีกประมาณ 5 นาที และต้องให้ product owner ยืนยัน | ร่างของ AI คำนวณ backlog ได้แต่ไม่มีเป้าหมายว่ายอมรับได้แค่ไหน ทำให้ validation plan ตรวจไม่ได้ว่าผ่านหรือตก |
| 3 | เพิ่มกฎข้อมูลส่วนบุคคลใน event: ห้ามใส่ข้อมูลบัตร ใช้ TLS ถึง broker, encrypt volume, ตั้ง TTL, ห้าม log ตัว event ยกเว้นเส้นทาง publish ล้มเหลว | ร่างของ AI ระบุ privacy เป็นแค่ผลเสียข้อหนึ่ง แต่ไม่มีมาตรการ SRE หรือฝ่าย security จะถามแน่นอน (และภายหลังผลรีวิว AI ชี้ว่า TTL ข้อนี้ขัดกับ exactly-once ของ loyalty ซึ่งต้องแก้ต่อ) |
| 4 | ลบบรรทัด "Drafted by Claude Code…" เป็น "ร่างด้วย AI ช่วย ตรวจและรับรองโดยผู้เขียน", ใส่ชื่อผู้ตัดสินใจ, สมมติว่ายังไม่มี broker ใน cluster, และตัด Open questions จาก 9 เหลือ 6 ข้อพร้อมระบุว่าใครเป็นผู้ตอบ | ADR ต้องมีเจ้าของที่ลงชื่อได้ และคำถามที่สถาปนิกตอบเองได้ไม่ควรค้างอยู่ให้คณะกรรมการ |
