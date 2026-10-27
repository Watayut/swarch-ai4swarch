**ภาษา:** ภาษาไทย | [English](00-prompt-log.en.md)

# บันทึกการใช้ prompt

**ชื่อ / รหัสนิสิต:** Watayut Aiamanan / 6970267821
**agent ที่ใช้:** Claude Code (VS Code extension ใน GitHub Codespaces) โดยแต่ละ run สั่งผ่าน sub-agent ใหม่ที่ไม่มีบริบทจากแชตก่อนหน้า
**โมเดล:** Claude Fable 5.1 (`claude-fable-5-1`)
**แพ็กเกจที่ใช้:** Claude Code (บัญชีส่วนตัว)

บันทึกทุกคำขอที่ส่งให้ agent ช่อง "ไฟล์ที่ agent อ่าน" คือไฟล์ที่ agent รายงาน หรือที่นิสิตเห็นว่า agent เปิด

| # | งาน / run | agent + โมเดล | ข้อความที่ส่ง (ย่อ) | ไฟล์ที่ agent อ่าน | อยู่ในขอบเขตหลักฐานที่อนุญาตหรือไม่ | มีประโยชน์ (1–5) | หมายเหตุ |
|---|---|---|---|---|---|---|---|
| 1 | 1-A | Claude Code / Fable 5.1 | `Read AGENTS.md first.` + `prompts/1-…` run A, evidence: `demo.proto` | `AGENTS.md`, `prompts/1-recover-architecture.md`, `system/online-boutique/protos/demo.proto` | ใช่ | 2 | ได้รายชื่อ 9 service ถูกต้อง แต่ตาราง dependency ว่าง (ถูกต้องตามหลักฐาน) ไม่เดาภาษาโปรแกรม ไม่รันคำสั่ง terminal |
| 2 | 1-B | Claude Code / Fable 5.1 | `prompts/1-…` run B, evidence: `demo.proto` + `kubernetes-manifests/` | `AGENTS.md`, prompt, `demo.proto`, ไฟล์ทั้ง 12 ไฟล์ใน `kubernetes-manifests/` | ใช่ (ไฟล์) **แต่ละเมิดกฎห้ามรัน terminal 1 ครั้ง** | 5 | agent รายงานเองว่ารัน `ls kubernetes-manifests/` เพื่อดูรายชื่อไฟล์ ไม่ได้อ่านเนื้อหาไฟล์อื่น ได้ 12 container และ 17 dependency พร้อมหลักฐาน |
| 3 | 2-a | Claude Code / Fable 5.1 | `prompts/2-…` run a, context: none, requirement: R-12 (ภาษาอังกฤษ ไม่ใส่ `language: Thai`) | `AGENTS.md`, `prompts/2-write-adr.md`, `docs/adr/template.md` | ใช่ ไม่ได้เปิด `docs/scenario.md` | 3 | โครง ADR ครบ แต่ไม่มีตัวเลขใด ๆ จาก scenario เลือก "event ผ่าน broker" โดยไม่ระบุชื่อ broker |
| 4 | 2-b | Claude Code / Fable 5.1 | `prompts/2-…` run b, context: `scenario.md`, `0001-…`, `checkoutservice/main.go`, requirement: R-12 (ภาษาอังกฤษ เช่นเดียวกับ run a) | `AGENTS.md`, prompt, `template.md`, `docs/scenario.md`, `docs/adr/0001-use-redis-for-cart-storage.md`, `src/checkoutservice/main.go` | ใช่ | 5 | ใช้ตัวเลขจาก scenario ครบ อ้างบรรทัดใน `main.go` เลือก RabbitMQ พิจารณา 4 ทางเลือก |
| 5 | 3 | Claude Code / Fable 5.1 | `prompts/3-review-adr.md`, adr: `docs/adr/0002-publish-order-placed-event-to-rabbitmq.md` (ภาษาอังกฤษ) | `AGENTS.md`, `prompts/3-review-adr.md`, ADR-0002, `docs/scenario.md` (พยายามเปิด `outputs/3-review.md` เพื่อเช็กว่ามีอยู่ก่อนหรือไม่) | ใช่ ไม่ได้แก้ ADR | 5 | คำตัดสิน Accept with changes พบ 4 Risk ที่คมมาก เช่น prefetch คุม rate 50/s ไม่ได้ และ TTL ทำลาย exactly-once |
| 6 | แก้ ADR-0002 | Claude Code (แชตหลัก) / Fable 5.1 | นิสิตตัดสินใจ 4 ประเด็น (คง RabbitMQ, ยอมรับ manual replay, สมมติ email ล่าช้าได้ 15 นาที, กฎ PII ใน event) แล้วให้ Claude Code พิมพ์แก้ให้ตามที่ตัดสินใจ | `docs/adr/0002-…`, `outputs/2b-adr.md` | ใช่ | 4 | ให้ AI ช่วยแนะนำพร้อม trade-off และฉันเลือกตัดสินใจตามประสบการณ์ส่วนตัว |

**หมายเหตุเพิ่มเติม**
- ทุก run ใช้ session ใหม่ (sub-agent ที่เริ่มจากศูนย์) ตามกติกา "เริ่มแชตใหม่ทุก run"
- worksheet ภาษาไทยแนะนำให้ใส่ `language: Thai` ใน run 2 และ 3 แต่นิสิตรันเป็นภาษาอังกฤษทั้ง run a, b และ 3 จึงยังเปรียบเทียบกันได้
- worksheet ทุกไฟล์ (00–05) นิสิตร่างร่วมกับ Claude Code ในแชตหลัก โดยนิสิตเป็นผู้ตรวจผลและตัดสินใจ
