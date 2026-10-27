**ภาษา:** ภาษาไทย | [English](04-review-board.en.md)

# ส่วน B: คณะกรรมการรีวิวสถาปัตยกรรม (ประมาณ 25 นาที, กลุ่มละ 4 คน)

> **เชื่อมโยงกับการบรรยาย:** AICH4 (การรีวิวต้องอาศัยความเชี่ยวชาญในโดเมน; *ฟังดูดี ≠ เหมาะสม*), Pillar 5 (AI เป็น Chief Analyst มนุษย์เป็น Chief Strategist), JSS (85% ของงานวิจัยให้มนุษย์มีส่วนร่วมในกระบวนการ)
> **คำถาม:** คณะกรรมการที่เป็นมนุษย์จับประเด็นใดได้ที่ผลรีวิวของ AI พลาดไป และเพราะเหตุใด

คณะกรรมการรีวิวสถาปัตยกรรมของบริษัทประชุมเพื่ออนุมัติการออกแบบ R-12 ก่อนเทศกาลลดราคา 11.11 โดยนำ ADR หนึ่งฉบับเข้าที่ประชุม **สถาปนิก**นำเสนอ ADR, **AI analyst** นำเสนอผลรีวิวของ AI และผู้มีส่วนได้ส่วนเสียสองคนนำข้อมูลที่ AI ไม่เคยเห็นมาด้วย

## 1. จัดตั้งคณะกรรมการ (2 นาที)

ในกลุ่มละ 4 คน ให้เลือก ADR ของสมาชิก**หนึ่งคน**มารีวิว (`docs/adr/0002-*.md` พร้อม `exercise/outputs/3-review.md` ของคนนั้น) แล้วแบ่งบทบาท

| บทบาท | ใคร | สิ่งที่นำมา |
|---|---|---|
| **สถาปนิก** | Watayut (ผู้เขียน ADR-0002) | ปกป้อง ADR |
| **AI analyst** | Chetsada Winaikoson (อ่าน `outputs/3-review.md` ตามตัวอักษร) | อ่านผลรีวิวของ AI เท่านั้น ห้ามใส่ความเห็นของตัวเอง |
| **Product owner** | Waris Natirutthakorn | การ์ดบทบาท P (ฝั่งธุรกิจ) |
| **Site reliability engineer (SRE)** | Natcha Thawinpipat | การ์ดบทบาท S (ฝั่งปฏิบัติการ) |

กลุ่มละ 5 คน ให้เพิ่ม**เจ้าหน้าที่ความปลอดภัยและความเป็นส่วนตัว** (การ์ดบทบาท X) กลุ่มละ 3 คน ให้สถาปนิกรับบท AI analyst ด้วย

**การ์ดบทบาท**อยู่ใน [`docs/review-board/`](../docs/review-board/README.md) ผู้มีส่วนได้ส่วนเสียแต่ละคนอ่าน**เฉพาะการ์ดของตัวเอง** และเก็บเป็นความลับจนถึงเวลาประชุม

> **หมายเหตุ:** กลุ่ม SE 101 (กลุ่มเดียวกับโปรเจกต์ SEATS) ประชุมกันครบ 4 คน บันทึกความเห็นของ PO, SRE และคำตอบของสถาปนิกอยู่ใน [`review-board-notes/`](review-board-notes/README.md) ผลรีวิวของ AI ใน `outputs/3-review.md` ทำขึ้นก่อนประชุมและไม่เคยเห็นการ์ดบทบาท จึงเทียบได้ว่า AI จับประเด็นใดได้หรือพลาด

## 2. การประชุม (15 นาที)

| นาที | ขั้นตอน |
|---|---|
| 3 | **สถาปนิก**นำเสนอการตัดสินใจ ทางเลือกที่พิจารณา และผลที่ตามมาหลัก ๆ |
| 3 | **AI analyst** อ่านข้อค้นพบและคำตัดสินของ AI แบบคำต่อคำ |
| 7 | **อภิปรายเปิด** ผู้มีส่วนได้ส่วนเสียแต่ละคนยกข้อกังวลจากการ์ดของตัวเองด้วยคำพูดของตัวเอง สถาปนิกตอบ และทุกคนถาม AI analyst ได้ว่า "AI เห็นประเด็นนี้หรือไม่" |
| 2 | **ตัดสินใจ** คณะกรรมการลงมติ: อนุมัติ / อนุมัติโดยมีเงื่อนไขให้แก้ไข / ไม่อนุมัติ |

## 3. บันทึกผลการตัดสินใจ (8 นาที, ทำร่วมกัน)

**ADR ที่รีวิว:** [`docs/adr/0002-publish-order-placed-event-to-rabbitmq.md`](../docs/adr/0002-publish-order-placed-event-to-rabbitmq.md) โดย Watayut
**สมาชิกและบทบาท:** Watayut Aiamanan = สถาปนิก; Chetsada Winaikoson = AI analyst (อ่านผลรีวิวใน `outputs/3-review.md`); Waris Natirutthakorn = Product owner (การ์ด P); Natcha Thawinpipat = SRE (การ์ด S) (รายละเอียดใน `review-board-notes/`)

**มติ:** **อนุมัติโดยมีเงื่อนไข** (PO, SRE และสถาปนิกเห็นตรงกันว่าทิศทางหลักถูกต้อง: asynchronous event, durable queue ต่อ consumer, RabbitMQ แทน Kafka แต่ส่วน "เมื่อพัง" และส่วน operations ยังไม่พร้อมขึ้น production)

**สิ่งที่ต้องแก้ไข:**
1. **เปลี่ยน manual replay เป็น fallback อัตโนมัติ** lazy AMQP connection (ห้าม `must` ตอน start-up), publish deadline ~50 ms + circuit breaker, durable spool (Redis outbox หรือ local disk) ที่ background worker re-publish เองแบบ idempotent ถอน CLI และการ log ตัว event ออก (ยกโดย AI, SRE A5, PO A2)
2. **ออกแบบ exactly-once ของแต้มให้เป็นรูปธรรม และถอด TTL ออกจาก `q.loyalty`** `loyaltyservice` ใช้ store ที่มี transaction (PostgreSQL) เขียน `processed_order_id` (unique) และ balance ใน transaction เดียว เก็บแต้มรายคำสั่งซื้อเพื่อหักคืนได้ภายหลัง และแสดง "แต้มที่จะได้รับ" บนหน้ายืนยันด้วย shared rule package ตามคำสัญญาของแคมเปญ (ยกโดย AI, PO A1–A2)
3. **แผนแบ่งเฟสสำหรับ 11.11 พร้อม go-live gate ของ SRE** เฟส 1 RabbitMQ single node + spool, ตัด analytics consumer และ 3-node cluster ไปหลัง 11.11, load test 4 และ 7 พ.ย., game day คนเวร 2 คน, checklist dashboard/SLO-based alert/backup/runbook, แก้ driver 6 ให้ตรงความจริงเรื่องประสบการณ์ RabbitMQ และระบุเจ้าของ on-call พร้อมคำตอบ peak duration จาก PO ก่อนอนุมัติ (ยกโดย SRE A1–A4, PO A3)

เงื่อนไขเพิ่มเติมที่ทำหลังอนุมัติได้: แก้ Extensibility rule ให้ครอบคลุมเฉพาะงานหลังยืนยัน (fraud แบบ pre-confirmation ต้องมี ADR แยก), จอง `order.refunded.v1` สำหรับการคืนเงิน, เพิ่ม Redis Streams และการเจรจา MailFast ลงในตารางทางเลือก, ใช้ `order.placed.v1` ตั้งแต่แรก รายการครบ 14 ข้ออยู่ใน `review-board-notes/architect-response.md`

**ใครยกประเด็นใด** ระบุทุกข้อกังวลที่คณะกรรมการอภิปราย

| # | ข้อกังวล | ผู้ยกประเด็น (ผลรีวิว AI / สถาปนิก / PO / SRE / security) | อยู่ในผลรีวิวของ AI หรือไม่ | AI มีทางรู้ได้หรือไม่ เพราะเหตุใด |
|---|---|---|---|---|
| 1 | เส้นทาง broker ล่ม: retry กิน latency budget, `must` ตอน start-up ทำให้ checkout พึ่ง broker, manual replay 12,000 event/นาที พร้อม PII ลง log | ผลรีวิว AI (SRE และ PO ย้ำในมุมภาระคนเวร 2 คนและแต้มซ้ำ) | ใช่ | ใช่ ทุกอย่างอยู่ใน ADR เอง (ตัวเลข 200 orders/s, กฎ "ห้าม log event body" และ `mustConnGRPC` ใน Context) |
| 2 | prefetch คุม rate 50 emails/s ไม่ได้ เพราะ latency ของ MailFast แกว่ง 1.5–8 s ทำให้อัตราต่างกัน 5 เท่า | ผลรีวิว AI | ใช่ | ใช่ คำนวณได้จาก p95/p99 ใน scenario สถาปนิกยอมรับว่าเป็นความผิดพลาดทางเทคนิค |
| 3 | exactly-once ของแต้ม "ยืนยันแต่ไม่ได้ออกแบบ" และ TTL ที่ใส่เพื่อ privacy จะทิ้ง loyalty event ตอน consumer ล่มนาน | ผลรีวิว AI (PO เสริมว่าแต้ม = เงิน 100 แต้ม = 10 บาท จึงเป็นความเสี่ยงทางการเงิน ไม่ใช่แค่ reliability) | ใช่ | ใช่ จากข้อกำหนด "exactly once" ใน scenario แต่ AI ไม่รู้มูลค่าของแต้ม จึงให้น้ำหนักต่ำกว่าที่ PO มอง |
| 4 | แคมเปญ 11.11 สัญญาว่า "เห็นแต้มทันทีที่สั่งซื้อ" แต่ ADR ให้แต้มภายใน 1 นาทีและไม่มี UX หน้ายืนยัน | PO | ไม่ | ไม่ scenario บอกแค่ "within 1 minute" คำสัญญาทางการตลาดอยู่ในการ์ด P เท่านั้น |
| 5 | fraud scoring ต้องยกเลิกคำสั่งซื้อได้ก่อนยืนยัน แต่ `order.placed` ถูก publish หลังตัดเงินแล้ว ข้ออ้าง "ปีหน้าแค่ bind queue" จึงไม่จริง | PO | ไม่ | ไม่ ทั้ง scenario และ ADR เรียก fraud scoring ว่า "post-order action" เหมือนกัน AI ทำได้แค่สงสัยจากสามัญสำนึกของโดเมน |
| 6 | silent failure: checkout สำเร็จเสมอโดยตั้งใจ และ queue depth สูงโดยธรรมชาติช่วง peak จึงใช้เป็น alert ไม่ได้ ต้องใช้ SLO-based alert, correlation id และ canary order | SRE | ไม่ | บางส่วน เหตุการณ์อีเมลล่ม 20 นาทีปีก่อนอยู่ในการ์ด S แต่ข้อสรุปว่า depth ใช้เป็น alert ไม่ได้อนุมานได้จาก 200/s vs 50/s และ ADR เองก็ยอมรับว่า trace ข้าม async boundary |
| 7 | ประสบการณ์ RabbitMQ ของทีมเป็นระดับโน้ตบุ๊ก เวร on-call มี 2 คน, go-live gate ต้องใช้ 1 สัปดาห์, 3-node HA ต้องรองบ 2 สัปดาห์ จึงไม่ทัน 11.11 (เหลือ 15 วัน) | SRE | บางส่วน (AI มี trade-off "3-node vs ภาระทีม 4 คน" และเสนอ single node + spool แต่ยังนับ "two devs know RabbitMQ" เป็น Non-risk และไม่พูดถึง timeline) | ไม่ ข้อจำกัด node งบ และระดับประสบการณ์จริงอยู่ในการ์ด S แม้ AI จะสังเกตได้ว่า ADR ไม่มี sizing และวันที่ใกล้ 11.11 |
| 8 | ไม่มี event สำหรับยกเลิก/คืนเงิน แต้มที่ให้แล้วเรียกคืนไม่ได้ เปิดช่อง "สั่งแล้วคืนเพื่อเก็บแต้ม" | PO | ไม่ | ไม่ scenario และโค้ด checkout ไม่มี flow คืนเงินเลย |

**ข้อค้นพบที่ดีที่สุดของ AI** (ที่มนุษย์มองข้ามไป): ข้อ 2 "prefetch คุม 50 emails/s ไม่ได้ เพราะอัตรา ≈ prefetch/latency และ latency ของ MailFast แกว่ง 1.5–8 s" ทั้งสถาปนิก PO และ SRE ไม่มีใครเห็นจุดนี้ มันเป็นข้อผิดพลาดทางเทคนิคจริงที่คำนวณได้จากตัวเลขสองตัวใน scenario และถ้าหลุดไปถึง production MailFast จะปฏิเสธ burst ช่วง peak รองลงมาคือการจับได้ว่า TTL ที่สถาปนิกเพิ่มเองเพื่อ privacy ทำลาย exactly-once ของ loyalty ซึ่งเป็นผลข้างเคียงจากการแก้ของมนุษย์ ไม่ใช่ของร่าง AI

**ข้อค้นพบที่อ่อนที่สุดของ AI** (กว้างเกินไป ผิด หรือไม่เกี่ยวข้อง): การเสนอ **Redis Streams** บน Redis จาก ADR-0001 เป็นทางเลือกที่ "ควรมีแถวในตาราง" ทั้งที่ ADR-0001 ระบุเองว่า Redis ตั้งค่าแบบ durability ต่ำและเป็น cart store ที่ scale ตามโหลด cart การเอา event ที่มีมูลค่าเป็นเงิน (แต้ม) ไปฝากไว้กับ store นั้นขัดกับข้อกำหนด exactly-once ที่ AI เองเพิ่งยกเป็น Risk นอกจากนี้ AI ยังนับ "two of four developers know RabbitMQ" เป็น Non-risk โดยไม่ตั้งคำถามว่าเคยใช้ในระดับไหน ซึ่ง SRE ชี้ว่าเป็นระดับโน้ตบุ๊กเท่านั้น

## 4. สะท้อนคิดรายบุคคล (≤ 150 คำ, ทุกคน, ทำหลังคาบได้)

เขียนใน [`05-reflection.md`](05-reflection.md): หลังจากทำส่วน A และส่วน B แล้ว นิสิตจะไว้วางใจให้ AI ทำงานส่วนใดของสถาปนิก และจะเก็บงานส่วนใดไว้ให้มนุษย์ ยกตัวอย่างที่เป็นรูปธรรมหนึ่งตัวอย่างจากการประชุมคณะกรรมการ

> สมาชิกทุกคนในกลุ่ม commit บันทึกนี้ลงใน repository ของตัวเอง (คัดลอกข้อความเดียวกัน) เพื่อให้ทุกคนส่ง repository ที่ครบถ้วน
