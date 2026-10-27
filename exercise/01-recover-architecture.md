**ภาษา:** ภาษาไทย | [English](01-recover-architecture.en.md)

# งานที่ 1: กู้คืนสถาปัตยกรรม (ประมาณ 15 นาที)

> **เชื่อมโยงกับการบรรยาย:** T7 การกู้คืนสถาปัตยกรรม, AICH2 (การตามรอย), AICH3 (บริบท) และข้อค้นพบจาก JSS ว่า 93% ของงานวิจัยไม่เคยตรวจสอบผลลัพธ์จาก AI
> **คำถาม:** AI กู้คืนสถาปัตยกรรมได้มากแค่ไหน และขึ้นอยู่กับหลักฐานที่ AI ได้อ่านมากเพียงใด

**ระหว่างทำงานนี้ ห้ามเปิด repository ต้นฉบับของ Online Boutique** เพราะ README ของต้นฉบับมีแผนภาพสถาปัตยกรรมที่นิสิตกำลังกู้คืนอยู่

ในแต่ละ run ให้เริ่ม **แชตหรือ session ใหม่** แล้วส่งข้อความด้านล่างให้ agent โดย agent จะทำตาม [`prompts/1-recover-architecture.md`](../prompts/1-recover-architecture.md) และเขียนผลลัพธ์ไว้ที่ `exercise/outputs/1<run>-architecture.md`

**Run A: เฉพาะ interface**
```
Follow prompts/1-recover-architecture.md
run: A
evidence: system/online-boutique/protos/demo.proto
```

**Run B: interface + ไฟล์ตั้งค่าการ deploy**
```
Follow prompts/1-recover-architecture.md
run: B
evidence: system/online-boutique/protos/demo.proto, system/online-boutique/kubernetes-manifests/
```

ระหว่างที่ agent ทำงาน ให้สังเกตว่า agent เปิดไฟล์ใดบ้าง หาก agent อ่านไฟล์นอกเหนือจากหลักฐานที่อนุญาต ให้บันทึกไว้ใน prompt log

## ผลลัพธ์

- Run A: [`outputs/1A-architecture.md`](outputs/1A-architecture.md)
- Run B: [`outputs/1B-architecture.md`](outputs/1B-architecture.md)

เปิดแต่ละไฟล์ด้วย **Markdown: Open Preview** (`Ctrl+Shift+V` / `⇧⌘V`) เพื่อดูแผนภาพ

## ตรวจสอบ Run B ด้วยตัวเอง (ส่วนที่สำคัญที่สุด)

ตรวจสอบ dependency ในตารางของ Run B **อย่างน้อยห้ารายการ** เทียบกับไฟล์จริง โดยเปิดไฟล์ที่ AI อ้างถึงและหาหลักฐาน นิสิตสามารถตรวจซอร์สโค้ดของ service ได้ด้วย เช่น `src/checkoutservice/main.go`

| ผู้เรียก → ผู้ถูกเรียก | หลักฐานที่ AI อ้าง | ยืนยันแล้วหรือไม่ ✅ / ❌ / ❓ | สิ่งที่นิสิตพบ |
|---|---|---|---|
| D12 `checkoutservice` → `paymentservice` | `checkoutservice.yaml` env `PAYMENT_SERVICE_ADDR=paymentservice:50051` | ✅ | `kubernetes-manifests/checkoutservice.yaml` บรรทัด 61–62 มี env ค่าตรงตามที่อ้าง และใน `src/checkoutservice/main.go` บรรทัด 116 อ่าน env นี้ บรรทัด 123 เปิด gRPC connection บรรทัด 370 เรียก `Charge` จริง ดังนั้น RPC ที่ AI ระบุว่า [Inferred] ก็ถูกต้อง |
| D4 `frontend` → `cartservice` | `frontend.yaml` env `CART_SERVICE_ADDR=cartservice:7070` | ✅ | `frontend.yaml` บรรทัด 73–74 มี env จริง และ `src/frontend/main.go` บรรทัด 134 `mustMapEnv(&svc.cartSvcAddr, "CART_SERVICE_ADDR")` ยืนยันว่า frontend ใช้ค่านี้เชื่อมต่อ |
| D17 `cartservice` → `redis-cart` | `cartservice.yaml` env `REDIS_ADDR=redis-cart:6379`; Service `redis-cart` port 6379 | ✅ | `cartservice.yaml` บรรทัด 50–51 มี env จริง และ `src/cartservice/src/Startup.cs` อ่าน `REDIS_ADDR` เพื่อเลือกใช้ Redis เป็น cart store (ไฟล์เป็น C# แปลว่า cartservice เขียนด้วย .NET ซึ่ง AI บอกถูกว่า "ภาษาไม่มีหลักฐานใน manifest") |
| D16 `recommendationservice` → `productcatalogservice` | `recommendationservice.yaml` env `PRODUCT_CATALOG_SERVICE_ADDR=productcatalogservice:3550` | ✅ | `recommendationservice.yaml` บรรทัด 60–61 มี env จริง และ `src/recommendationservice/recommendation_server.py` บรรทัด 131–133 อ่าน env นี้และ raise exception ถ้าไม่มี เป็น dependency บังคับ (ภาษา Python) |
| D13 `checkoutservice` → `emailservice` | `checkoutservice.yaml` env `EMAIL_SERVICE_ADDR=emailservice:5000`; `emailservice.yaml` port 5000 → 8080 | ✅ | `checkoutservice.yaml` บรรทัด 63–64 มี env จริง และ `main.go` บรรทัด 115 อ่าน env บรรทัด 380 เรียก `SendOrderConfirmation` จริง ตรงกับคอลัมน์ RPC ที่ AI อนุมาน |
| (ไม่อยู่ในตาราง) `frontend` → `shoppingassistantservice` | AI ระบุใน Gaps ว่ามีแค่ address ใน `frontend.yaml` ไม่มี manifest จึง**ไม่ใส่ในแผนภาพ** | ❓ | `frontend.yaml` บรรทัด 83–84 ตั้ง `SHOPPING_ASSISTANT_SERVICE_ADDR=shoppingassistantservice:80` และ `src/frontend/main.go` บรรทัด 139 ใช้ `mustMapEnv` อ่านค่านี้ (frontend จะ start ไม่ได้ถ้าไม่มี env) แต่ฟีเจอร์ถูกปิดด้วย `ENABLE_ASSISTANT` ที่ comment ไว้ (บรรทัด 93–94; `handlers.go` บรรทัด 48) และมีโฟลเดอร์ `src/shoppingassistantservice/` อยู่จริงแม้ไม่มี manifest สรุป: เป็น dependency ที่ "ตั้งค่าไว้แต่ยังไม่ใช้งาน" AI ตัดสินใจสมเหตุสมผลตามหลักฐานที่ได้รับ แต่ภาพรวมไม่ครบ |

- **สิ่งที่ผิดหรือขาดหายไป** ใน Run B (dependency ที่ไม่มีจริง, dependency ที่ขาดไป, container ที่น่าสงสัย):
  - **ขาด** `shoppingassistantservice`: มีซอร์สโค้ดอยู่ใน `src/` และ frontend บังคับต้องมี env ของมัน แต่แผนภาพไม่มี เพราะ AI เห็นแค่ manifests ซึ่งไม่มีไฟล์ของ service นี้
  - **container ที่น่าสงสัย** `loadgenerator`: AI ใส่เป็น container ในระบบทั้งที่ `kustomization.yaml` บรรทัด 24 comment ไว้ (AI เองก็ตั้งข้อสังเกตนี้) ในแผนภาพสำหรับทีมพัฒนาควรแยกเป็นเครื่องมือทดสอบ ไม่ใช่ส่วนหนึ่งของระบบ
  - **ไม่มี dependency ปลอม** จากที่ตรวจ 5 รายการ ทุกรายการมี env จริงและมีโค้ดเรียกจริง คอลัมน์ "Likely RPCs [Inferred]" ที่ตรวจ (Charge, SendOrderConfirmation) ก็ถูกต้อง
  - **ภาษาโปรแกรม** AI เขียน "Not evidenced" ทุก service ซึ่งซื่อสัตย์ แต่ทำให้แผนภาพ C4 ขาดคอลัมน์ Technology ที่มีประโยชน์ (จริง ๆ คือ Go, C#, Python, Node.js, Java ปนกัน) ซึ่งดูได้จาก `src/` เท่านั้น

## สะท้อนคิด (ข้อละ 2–3 ประโยค)

1. เปรียบเทียบ A กับ B: agent **ไม่สามารถ**รู้อะไรได้จากไฟล์ `.proto` เพียงไฟล์เดียว และเพราะเหตุใด
   จาก `demo.proto` agent รู้เพียงว่า "มี service อะไรบ้างและแต่ละตัวให้บริการ RPC อะไร" จึงได้ 9 container แต่ได้ dependency 0 รายการ ส่วน Run B ได้ 17 รายการ เพราะ `.proto` เป็นสัญญาฝั่งผู้ให้บริการ ไม่มีโค้ดฝั่งผู้เรียก ไม่มี address ไม่มี env และไม่มี data store ใด ๆ (Redis ไม่ปรากฏใน proto เลย) จึงบอกไม่ได้ว่าใครเรียกใคร มี frontend หรือไม่ และระบบ deploy อย่างไร ข้อมูลเหล่านี้อยู่ใน deployment config และซอร์สโค้ดเท่านั้น

2. ข้อผิดพลาด**ข้อใดข้อหนึ่ง**ที่จะทำให้สมาชิกใหม่ของทีมเข้าใจผิดมากที่สุด หากเผยแพร่แผนภาพนี้โดยไม่ตรวจสอบ
   การที่แผนภาพไม่มี `shoppingassistantservice` ทั้งที่ `frontend` ใช้ `mustMapEnv` อ่าน address ของมันตอน start สมาชิกใหม่ที่เชื่อแผนภาพจะคิดว่า frontend มี dependency แค่ 7 ตัว แล้วแปลกใจเมื่อ frontend crash ตอนลบ env นี้ออก หรือไม่รู้ว่ามีโค้ด service ที่ 12 รอเปิดใช้อยู่ ข้อผิดพลาดนี้เกิดจากหลักฐานที่ให้ AI ไม่ครบ ไม่ใช่ AI เดาผิด จึงยิ่งอันตรายเพราะทุกบรรทัดในแผนภาพดูมีหลักฐานรองรับ
