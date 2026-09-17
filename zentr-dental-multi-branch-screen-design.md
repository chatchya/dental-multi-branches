# Zentr Dental — Screen Design สำหรับคลินิกหลายสาขา

เอกสารนี้เป็นข้อเสนอด้าน UX/UI สำหรับระบบบริหาร Zentr Dental แบบ multi-branch โดยสังเคราะห์จาก

- `zentr-dental-multi-branch-requirements.md`
- `zentr-knowledge-base.md`
- หน้า live ของ [Zentr Dental Edition](https://dental.zentr.biz/)
- คู่มือ [ราคา](https://dental.zentr.biz/guide/dental-clinic-software-price/), [Doctor Fee](https://dental.zentr.biz/guide/doctor-fee/), [Lab Management](https://dental.zentr.biz/guide/lab-management/) และ [PDPA](https://dental.zentr.biz/guide/pdpa-dental-clinic/)

สถานะของเอกสาร: **product/design proposal** ไม่ใช่ API contract, database schema หรือการยืนยันว่า prototype ปัจจุบันมีความสามารถเหล่านี้แล้ว

> ข้อมูลในตัวอย่างหน้าจอทั้งหมดเป็นข้อมูลสมมติสำหรับอธิบาย UI เท่านั้น ห้ามใช้ชื่อคนไข้จริง, HN จริง หรือข้อมูลสุขภาพจริงเป็น fixture

## 1. Design thesis

**“ศูนย์ควบคุมคลินิกที่เห็นทั้งเครือได้ในจังหวะเดียว แต่ทำงานได้เฉพาะในขอบเขตที่ได้รับอนุญาต”**

ระบบควรให้ผู้บริหารเห็นสัญญาณสำคัญของทุกสาขา ขณะที่หน้า operation ต้องพาคนหน้างานทำงานต่อได้เร็ว ตั้งแต่นัดหมายจนปิดการชำระ โดยไม่ซ่อนความเสี่ยงของข้อมูลสุขภาพ การจองชน งานแล็บไม่ทัน หรือการแก้ยอดย้อนหลังไว้หลัง UI ที่ดูสวย

หลักการที่ใช้เป็นแกน:

1. **Branch context มาก่อน** — แสดงสาขาที่กำลังทำงานใน top bar, page title, record card, side panel และเอกสารสำคัญเสมอ
2. **หนึ่ง Patient 360 ต่อหนึ่งคนไข้ภายใน tenant** — ประวัติการมารับบริการแยกตามสาขา แต่ไม่สร้างคนไข้ซ้ำเพียงเพราะเปลี่ยนสาขา
3. **ดูรวมได้ แต่แก้ต้องเลือกสาขา** — มุมมอง “ทุกสาขา” ใช้ดู/เปรียบเทียบ; การสร้างนัด เช็กอิน แก้บิล หรือแก้ clinical record ต้องมี target branch ที่ชัดเจน
4. **สถานะต้องตอบว่าเกิดอะไรขึ้นต่อ** — ทุก card สำคัญมี owner, next action, due date และ source/last synced เมื่อเหมาะสม
5. **AI เป็นผู้ช่วยสื่อสาร** — มี draft, confidence, source และ handoff; ไม่มีปุ่มหรือ copy ที่ทำให้ดูเหมือน AI วินิจฉัย สั่งยา หรือตีความ X-ray
6. **ความเสี่ยงไม่ใช่สีอย่างเดียว** — risk flag ต้องบอกเหตุผล, เจ้าของงาน, deadline และ action ที่ทำต่อได้

## 2. Information architecture

### Primary navigation

1. **ภาพรวมเครือคลินิก** — executive KPI, branch health, treatment pipeline, decision queue และ drill-down
2. **นัดหมายและคิว** — Schedule, Queue, no-show/cancellation, transfer
3. **คนไข้** — Patient 360, family/guardian, consent, duplicate review
4. **การรักษา** — Treatment Record, odontogram, Treatment Plan, orthodontics, packages
5. **งานแล็บ** — Lab board, risk queue, revisions, lab directory
6. **การเงิน** — Billing, Payment, Receipt, Aging, installment, Doctor Fee/DF
7. **Inbox** — LINE OA, Website Chat Bubble, AI draft, human handoff
8. **รายงานและ Insights** — operational, clinical-workflow, finance, branch drill-down
9. **ตั้งค่าองค์กร** — branches, users/roles, resources, catalog, policies, integrations, audit, PDPA requests

### Default landing ตามบทบาท

| บทบาท | หน้าแรกที่แนะนำ | ค่า branch scope เริ่มต้น |
|---|---|---|
| Owner / HQ | ภาพรวมเครือคลินิก | ทุกสาขา ตาม policy |
| Branch manager | ภาพรวมสาขา | สาขาที่รับผิดชอบ |
| Counter | นัดหมายวันนี้ / คิวหน้างาน | สาขาที่กำลังเข้ากะ |
| Dentist | คิวของฉัน / เคสที่ได้รับมอบหมาย | สาขาและเคสที่ได้รับสิทธิ์ |
| Assistant | งานวันนี้ / งานแล็บเสี่ยง | สาขาและเคสที่รับผิดชอบ |
| Finance | การเงินและ DF | สาขาที่อยู่ใน finance scope |

## 3. Application shell

### Desktop layout

```text
┌──────────────────────────────────────────────────────────────────────────────────────┐
│ Zentr Dental  [องค์กร]  [กำลังดู: ทุกสาขา ▾]  [ค้นหา HN/ชื่อ/นัด/บิล]  +สร้าง  🔔  ◯ │
├───────────────┬──────────────────────────────────────────────────────────────────────┤
│ ภาพรวม        │ Breadcrumb / Page title                         วันที่ 16 ก.ย. 2569 │
│ นัดหมายและคิว │ ┌──────────── Branch context banner ───────────────────────────────┐ │
│ คนไข้          │ │ มุมมองทุกสาขา · ข้อมูลรวม 3 สาขา · อัปเดตล่าสุด 09:41              │ │
│ การรักษา       │ └───────────────────────────────────────────────────────────────────┘ │
│ งานแล็บ        │                                                                      │
│ การเงิน        │                         Page content                                  │
│ Inbox          │                                                                      │
│ รายงาน         │                                                                      │
│ ตั้งค่า        │                                                                      │
│                │                                                                      │
│ [ศูนย์ช่วยเหลือ]│                                                                      │
│ [ผู้ใช้ / role] │                                                                      │
└───────────────┴──────────────────────────────────────────────────────────────────────┘
```

### Branch switcher behavior

- ตัวเลือกแรกแสดงเสมอว่า **กำลังดู: [สาขา]** หรือ **กำลังดู: ทุกสาขา**
- “ทุกสาขา” แสดงเฉพาะผู้มีสิทธิ์ และใช้ใน Dashboard/Reports เป็นหลัก
- เมื่ออยู่ใน “ทุกสาขา” ปุ่ม mutation ที่ target ไม่ชัด เช่น `+นัดหมาย`, `เช็กอิน`, `รับชำระ`, `แก้ clinical record` ต้อง disabled หรือเปิด dialog ให้เลือกสาขาก่อน
- ผู้ใช้ที่มีหลายสาขาควรจำ “สาขาที่ใช้งานล่าสุด” ได้เฉพาะใน session ที่ปลอดภัยตาม contract; server ต้องเป็นผู้ตัดสินสิทธิ์จริง
- เมื่อเปลี่ยนสาขา ให้เปลี่ยนข้อมูลและ URL state พร้อมแสดง confirmation เล็ก ๆ ว่า scope เปลี่ยนแล้ว
- ทุก record card มี branch chip แม้จะอยู่ในมุมมองสาขาเดียว เพื่อป้องกันการอ่านผิดเมื่อเปิดหลายแท็บ

### Design tokens (proposal)

| หมวด | Token | ค่าเสนอ | การใช้ |
|---|---|---:|---|
| Canvas | `canvas` | `#F3F7F5` | พื้นหลังหลักแบบเบา สอดคล้อง visual ของเว็บ Dental |
| Surface | `surface` | `#FFFFFF` | card, table, dialog |
| Ink | `ink` | `#16362E` | heading และข้อความหลัก |
| Muted | `muted` | `#64766F` | metadata, hint, timestamp |
| Dental green | `brand` | `#2D806D` | primary action, active nav |
| Mint | `brand-soft` | `#DDF4EC` | selected scope, safe context, supporting badge |
| Warning | `warning` | `#B7791F` | pending, lab risk, approval required |
| Danger | `danger` | `#B44C4C` | collision, overdue, cancelled, access blocked |
| Info | `info` | `#46739A` | neutral operational notice |
| Border | `border` | `#DCE6E1` | table/card border |

แนวทางเพิ่มเติม: ใช้ font ไทยที่อ่านง่าย เช่น Noto Sans Thai หรือ system fallback, base 14–16 px, line-height อย่างน้อย 1.5, radius 12–16 px สำหรับ card และ 8–10 px สำหรับ control, ใช้ shadow เบาเฉพาะเพื่อแบ่งชั้นข้อมูล ไม่ใช้ gradient/glass เป็นโครงสร้างหลัก

## 4. Screen 01 — Control Tower / ภาพรวมเครือคลินิก

### เป้าหมาย

ให้ Owner/HQ และ Branch manager อ่าน “สุขภาพของเครือ” ได้ในไม่กี่วินาที: รายรับและรับชำระ, สาขาที่ต่ำกว่าเป้า, pipeline การรักษา, aging และ risk ที่ต้องตัดสินใจ จากนั้นจึงค่อยเปิดหน้าปฏิบัติการเมื่อจำเป็น

หน้าหลักจึงเป็น **Executive Control Tower** ไม่ใช่จอคุมกะรายวัน โดย Schedule, Queue, Patient 360 และ Lab board ยังคงเป็น drill-down ที่เข้าถึงได้จากเมนูและการ์ด Daily operation ด้านล่าง

### โครงหน้าจอ

```text
ภาพรวมเครือคลินิก              [ทุกสาขา ▾] [เดือนนี้ ▾] [เทียบช่วงก่อนหน้า ▾]

┌──────────────┬──────────────┬──────────────┬──────────────┬──────────────┐
│ รายรับรวม     │ รับชำระแล้ว   │ ยอดค้าง/aging │ นัดหมายช่วงนี้ │ ต้องตัดสินใจ  │
│ trend         │ collection    │ aging buckets │ no-show/cancel │ lab/DF/recall │
└──────────────┴──────────────┴──────────────┴──────────────┴──────────────┘

┌──────────────────────────────────────┬──────────────────────────────┐
│ รายรับและรับชำระตามช่วงเวลา          │ รายการที่ผู้บริหารต้องตัดสินใจ │
│ trend 6 เดือน · รายรับ vs รับชำระ     │ impact · owner · next action  │
│ [ดูรายงาน]                            │ Lab risk · DF rule · policy    │
└──────────────────────────────────────┴──────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│ เปรียบเทียบสุขภาพรายสาขา                                                │
│ สาขา · รายรับ · รับชำระ · ยอดค้าง · นัดหมาย/no-show · แผน · lab risk     │
│ [ดูสาขา] [ดูสาขา] [ดูสาขา]                                                │
└─────────────────────────────────────────────────────────────────────────┘

┌──────────────────────┬──────────────────────┬─────────────────────────┐
│ Treatment pipeline    │ Risk signals         │ Daily operation          │
│ ร่าง → เสร็จสิ้น      │ lab / aging / recall │ Schedule · Queue · 360   │
│ [เปิด pipeline]       │ [เปิดรายการ]         │ เป็นทางเข้าหน้างานรอง    │
└──────────────────────┴──────────────────────┴─────────────────────────┘
```

### Interaction สำคัญ

- ช่วงเวลาหลักของหน้าเป็นเดือน/ไตรมาส ไม่ใช่ “วันนี้”; ทุก metric ต้องระบุ scope, period, last sync และสถานะข้อมูล
- KPI card คลิกแล้วไปหน้ารายการหรือ report ที่กรองตรงกัน ไม่พาไปกราฟที่ไม่มี action
- Branch table ต้องมี `สาขา`, `รายรับ`, `รับชำระ`, `ยอดค้าง`, `นัดหมาย/no-show`, `treatment pipeline`, `lab risk`, `health`, `last synced`
- Decision card ใช้โครงสร้าง `สัญญาณ → เหตุผล → owner → เปิดงาน`; อย่ารวมรายการปฏิบัติการทั้งหมดไว้บน landing
- เมื่อเลือกสาขาเดียว ตัวเลขและรายการต้องเปลี่ยนตาม branch scope; “ทุกสาขา” ใช้ดู/เปรียบเทียบ และ mutation ที่ target ไม่ชัดต้องเลือกสาขาก่อน
- แสดง “ข้อมูลสมมติ/ข้อมูลตัวอย่าง” ใน prototype และไม่ใช้เลข commercial snapshot เป็น entitlement จริง

### State ที่ต้องออกแบบ

- Loading: skeleton ของ KPI และ table ไม่แสดงเลข 0 แทนข้อมูลที่ยังโหลดไม่เสร็จ
- Empty: “ยังไม่มีข้อมูลในช่วงเวลานี้” พร้อม action เปลี่ยนช่วงเวลา/สาขา
- Partial failure: แยก card ที่โหลดไม่ได้และมี `ลองใหม่`; อย่าทำให้ทั้งหน้าเป็น blank
- Permission: ซ่อนยอดการเงิน/ข้อมูลสุขภาพตาม role พร้อมข้อความ “ขอสิทธิ์จากผู้ดูแล”
- Stale: แสดงเวลาที่ sync ล่าสุดและ banner เมื่อข้อมูลไม่สดพอสำหรับการตัดสินใจสำคัญ

## 5. Screen 02 — Schedule / ตารางนัดหมายหลายสาขา

### เป้าหมาย

เป็นแหล่งเดียวสำหรับตารางจากเคาน์เตอร์, LINE OA และ Website Chat Bubble พร้อมแสดง resource ที่ต้องว่างพร้อมกัน: สาขา, เก้าอี้ และทันตแพทย์

### โครงหน้าจอ

```text
ตารางนัดหมาย     [ทุกสาขา ▾] [วัน | สัปดาห์ | เดือน] [วันนี้] [‹ ›]
[กรอง: เก้าอี้ ▾] [ทันตแพทย์ ▾] [สถานะ ▾] [ช่องทาง ▾] [+ สร้างนัดหมาย]

      09:00       10:00       11:00       12:00       13:00
สาขา A ─────────────────────────────────────────────────────────
เก้าอี้ 1   [นัดหมายสมมติ A · ขูดหินปูน · ยืนยันแล้ว]
เก้าอี้ 2                [นัดหมายสมมติ B · รอยืนยัน]
สาขา B ─────────────────────────────────────────────────────────
เก้าอี้ 1       [นัดหมายสมมติ C · อุดฟัน · จาก LINE]

Legend: ยืนยันแล้ว  รอยืนยัน  เสร็จสิ้น  ยกเลิกแล้ว  ·  เตรียม/ฆ่าเชื้อ
```

### กติกา UX

- มุมมอง “ทุกสาขา” ใช้ branch swimlane; ถ้าจำนวนสาขามากให้เลือก “สาขาที่ปักหมุด” หรือเปลี่ยนเป็น list เพื่อลดการ scroll แนวนอน
- appointment card แสดงอย่างน้อย: เวลา, synthetic patient label/HN, procedure, duration, chair, dentist, branch, source และ status
- เวลาหัตถการต้องรวมเวลาเตรียม/พัก/ทำความสะอาดหรือฆ่าเชื้อตาม configuration ของสาขา
- slot availability ต้องมาจาก server; UI แสดงสถานะ `กำลังตรวจสอบ slot` ระหว่างยืนยัน
- หาก slot ถูกจองไปแล้ว ให้แสดงผลลัพธ์ชัดเจน: “ช่วงนี้ไม่ว่างแล้ว” + สาเหตุระดับที่เปิดเผยได้ + slot ทางเลือกจากตารางจริง
- `รอยืนยัน` ต้องแยกจาก `ยืนยันแล้ว` และแสดงเวลาหมดอายุ/ผู้อนุมัติตาม policy เมื่อ policy ถูกกำหนด
- transfer/reschedule/cancel เปิด side panel ที่เก็บเหตุผล, ผู้แก้ไข, เวลา, resource impact และ approval ที่เกี่ยวข้อง

### Appointment detail side panel

```text
นัดหมาย #SYN-0001                       [ยืนยันแล้ว]
สาขา A · 16 ก.ย. · 10:30–11:30

คนไข้: คนไข้ตัวอย่าง 01   [เปิด Patient 360]
หัตถการ: รายการสมมติ · 60 นาที
ทันตแพทย์: ผู้ให้บริการตัวอย่าง 01
เก้าอี้: ห้อง/เก้าอี้ 01
ช่องทาง: LINE OA       สถานะ consent: ตรวจแล้ว / ต้องตรวจ

[เช็กอิน] [เลื่อนนัด] [ยกเลิก] [โอนสาขา]
ประวัติการเปลี่ยนแปลง ▾
```

### Responsive behavior

- Desktop: timeline + branch lanes
- Tablet: เลือก branch เป็น tab และใช้ timeline เต็มความกว้าง
- Narrow/mobile: เปลี่ยนเป็น agenda list ตามวัน; card กดเปิด detail; ห้ามบังคับ horizontal scroll เพื่อเข้าถึง action สำคัญ

## 6. Screen 03 — Queue / คิวหน้างานและเช็กอิน

### เป้าหมาย

ให้เคาน์เตอร์และผู้ช่วยเคลื่อนคนไข้ตามสถานะจริง โดยเห็นเวลารอและสาเหตุที่คิวค้าง

### โครงหน้าจอ

```text
คิวหน้างาน · สาขา A                 [วันที่] [เก้าอี้ ▾] [ทันตแพทย์ ▾]

┌──────────────┬──────────────┬──────────────┬──────────────┬──────────────┐
│ รอเรียก      │ เรียกแล้ว    │ อยู่บนเก้าอี้ │ รอชำระเงิน    │ เสร็จสิ้น      │
│ 03            │ 01           │ 02           │ 01           │ 12           │
│ [walk-in]     │ timer 04:12  │ timer 18:30  │ บิลบางส่วน    │ ปิด encounter │
└──────────────┴──────────────┴──────────────┴──────────────┴──────────────┘

card: เวลานัด · คนไข้ตัวอย่าง · procedure · provider · chair · wait timer
      [เรียก] [เรียกซ้ำ] [ข้ามคิว] [เปิด Patient 360]
```

### Interaction

- รองรับ walk-in, เรียกซ้ำ, ข้ามคิว และตัวจับเวลารอ
- เช็กอินต้องยืนยันสาขาปัจจุบันและนัดที่ถูกต้องก่อนสร้าง encounter
- การ์ด “รอชำระเงิน” แสดงยอดคงเหลือและปุ่มส่งต่อการเงินโดยไม่เปิด clinical record เกิน role
- เมื่อกดเปลี่ยนสถานะ ให้ optimistic UI ได้เฉพาะหลัง server response ยืนยันแล้ว; ถ้าล้มเหลวให้คืนสถานะและแสดงเหตุผล

## 7. Screen 04 — Patient 360

### เป้าหมาย

รักษา master record เดียวของคนไข้ใน tenant พร้อมแสดงบริบทข้ามสาขาอย่างปลอดภัยและตรวจสิทธิ์ก่อนเปิดข้อมูลสุขภาพ

### Search / identity gate

- ค้นด้วย HN, ชื่อ, เบอร์ หรือช่องทางติดต่อที่ได้รับอนุญาต
- ถ้าพบผู้ต้องสงสัยซ้ำ ให้แสดง duplicate review ก่อนสร้าง record ใหม่
- เมื่อคนไข้จากสาขาอื่นเข้ามา: แสดงว่าพบ master record เดิม, สาขาที่เกี่ยวข้อง และระดับข้อมูลที่ผู้ใช้ role นี้เห็นได้
- ถ้า policy ต้องขอ consent เพิ่ม ให้ lock clinical tab และเสนอ action “ขอ consent/ส่งต่อผู้มีสิทธิ์” ไม่เปิดข้อมูลเงียบ ๆ

### โครงหน้าจอ

```text
Patient 360 / คนไข้ตัวอย่าง 01                         [สาขาปัจจุบัน: B ▾]
HN: SYN-HN-0001   [คนไข้ใหม่] [กำลังจัดฟัน] [มีแผนค้าง]
โทร/LINE: ซ่อนบางส่วนตาม role   ผู้ปกครอง: เชื่อมแล้ว   Consent: 3/4 จุดประสงค์

┌──────────────┬──────────────────────────────────────────────┐
│ สรุป          │ นัดถัดไป: 18 ก.ย. · สาขา B · ผู้ให้บริการสมมติ │
│ นัดหมาย       │ ยอดค้าง: ตามสิทธิ์การเงิน · แผน: กำลังรักษา      │
│ การรักษา      │ Lab: 1 รายการเสี่ยง · ล่าสุด: encounter สาขา A │
│ แผนการรักษา   │                                              │
│ ฟัน / X-ray   │ Timeline ข้ามสาขา                              │
│ งานแล็บ       │ 16 ก.ย.  สาขา B  นัดหมาย / เช็กอิน               │
│ การเงิน       │ 02 ก.ย.  สาขา A  Treatment Record               │
│ บทสนทนา       │ 25 ส.ค.  สาขา A  ใบเสร็จ / รับชำระ                │
│ Consent/Audit │                                              │
└──────────────┴──────────────────────────────────────────────┘
```

### Clinical summary

- แยกข้อมูลทั่วไปออกจากข้อมูลสุขภาพ: โรคประจำตัว, ยาประจำ, แพ้ยา/แพ้สาร, critical flags
- แสดง 32 ซี่แบบ FDI ใน tab odontogram พร้อม legend: เคยรักษา / อยู่ในแผน / ไม่มีฟัน
- X-ray และไฟล์แนบแสดง owner, วันที่, สาขา, encounter และสิทธิ์การดาวน์โหลด
- note ทางคลินิกต้องแสดงผู้เขียน, เวลา, สาขา, revision และไม่ถูกเขียนทับเมื่อย้ายสาขา

### Family / guardian

แสดง relationship graph แบบเรียบง่าย: คนไข้, ผู้ปกครอง, ผู้ติดต่อฉุกเฉิน, ผู้มีสิทธิ์ให้ consent/รับข้อมูล พร้อมวันที่ตรวจสอบล่าสุดและ source ของความสัมพันธ์

## 8. Screen 05 — Treatment Record และ Treatment Plan

### Treatment Record

โครงสร้าง record ให้ผู้มีสิทธิ์เห็นลำดับเดียวกัน:

`Appointment → Check-in/Encounter → Procedure → Treatment note → X-ray/attachment → Lab order → Payment context`

ทุกขั้นมี `branch`, ผู้สร้าง, ผู้แก้ไข, timestamp และ link กลับ Patient 360

### Treatment Plan list

แสดง status ที่สื่อความหมายตรงกับ workflow:

`ร่าง → เสนอแล้ว → ตอบรับแล้ว → กำลังรักษา → เสร็จสิ้น` และปลายทาง `ไม่ตอบรับ`

แต่ละ row มี: คนไข้, provider, สาขาเจ้าของเคส, procedure count, net amount, paid, remaining, offer expiry, next action

### Treatment Plan detail

- header แยกชัดระหว่าง **ตอบรับแผนแล้ว** กับ **เริ่มรักษาแล้ว**
- รายการหัตถการ: จำนวน, ราคา/หน่วย, ส่วนลด, ยอดสุทธิ
- payment summary: จ่ายแล้ว, คงเหลือ, ตารางผ่อน, link ไป Billing
- package summary: ใช้แล้ว/ทั้งหมด, ยอด/สิทธิ์คงเหลือ, Loyalty Basic ตาม entitlement
- timeline: เสนอแผน, ตอบรับ, เริ่มรักษา, encounter ที่เกิดจริง, เสร็จสิ้น
- การเปลี่ยนสาขา/ผู้รับผิดชอบแสดง impact ต่อ appointment, lab, billing และ DF ก่อน submit

### Orthodontic view

ใช้ sub-section เฉพาะเคสจัดฟัน: ชนิดการรักษา, เดือนที่ทำ/แผนทั้งหมด, ครั้งที่ปรับล่าสุด, นัดถัดไป, จำนวนชุด aligner, ค่ารักษา, เงินที่จ่าย, ค่างวด, หมายเหตุ และ recall warning

Recall warning เปิดงานติดตามได้ แต่ไม่เลื่อนนัดหรือให้คำแนะนำทางคลินิกอัตโนมัติ

## 9. Screen 06 — Lab Control Tower

### เป้าหมาย

ทำให้ทีมตอบได้จากหน้าจอเดียวว่า “งานอยู่ที่ไหน, จะกลับเมื่อไร, ผูกกับนัดใส่ไหน และยังมี buffer เหลือเท่าไร”

### โครงหน้าจอ

```text
งานแล็บ                         [ทุกสาขา ▾] [เสี่ยงเท่านั้น] [นัดใส่ใน 3 วัน]

┌──────────────┬──────────────┬──────────────┬──────────────┬──────────────┐
│ ส่งแล็บแล้ว  │ กำลังผลิต    │ กำลังส่งกลับ  │ ถึงคลินิกแล้ว │ ใส่เสร็จ       │
│ 05           │ 07           │ 02           │ 03           │ 18           │
└──────────────┴──────────────┴──────────────┴──────────────┴──────────────┘

card: ประเภทงาน · ตำแหน่ง/ซี่ · lab · คนไข้ตัวอย่าง · สาขาต้นทาง → ปลายทาง
      ส่งเมื่อ · กำหนดรับ · วันนัดใส่ · buffer · ต้นทุน/ค่ารักษา · revision
      [เปิดเคส] [ติดต่อผู้รับผิดชอบ] [สร้าง approval]
```

### Risk treatment

- risk chip ต้องบอกเหตุผล เช่น `กำหนดรับชนวันนัดใส่`, `เลยวันนัดใส่`, `buffer ต่ำกว่า threshold`
- threshold 2 วันจากต้นแบบควรเป็น **configurable policy** และติดป้าย “รอการอนุมัติ production rule” จนกว่าจะมี decision
- ถ้าเปลี่ยนสาขาปลายทางหรือวันนัด ระบบต้องคำนวณ risk ใหม่และบันทึก recalculation timestamp
- default operational view เน้นงานนัดใส่ใน 3 วัน เพราะเป็นช่วงที่ยังมีเวลาประสานงาน/เลื่อนนัดอย่างสุภาพ
- `ส่งซ่อม/แก้งาน` เป็น revision ใหม่ มีเหตุผล, รอบที่, deadline ใหม่, ผู้ส่ง, ค่าใช้จ่าย และ history เดิมแบบ read-only

## 10. Screen 07 — Billing, Payment และ Doctor Fee/DF

### Finance overview

```text
การเงินและ DF                    [ทุกสาขา ▾] [รอบบัญชี ▾] [Export ตามสิทธิ์]

┌──────────────┬──────────────┬──────────────┬──────────────┐
│ รายรับ        │ รับชำระ      │ ยอดค้าง      │ DF รอตรวจสอบ  │
│ ตาม policy    │ เต็ม/บางส่วน │ aging 4 กลุ่ม│ ตามรอบจ่าย    │
└──────────────┴──────────────┴──────────────┴──────────────┘

รายการที่ต้องตรวจ:  บิลบางส่วน · receipt รอตรวจ · refund/void รออนุมัติ · DF mismatch
```

### Invoice / payment detail

แยก context ที่อาจไม่ใช่สาขาเดียวกันให้เห็นเป็น field แยก:

- สาขาที่ให้บริการ
- สาขาที่รับชำระ
- นิติบุคคล/เลขที่เอกสาร
- ผู้รับรายได้
- สาขา/ผู้เป็นเจ้าของ DF
- Treatment Plan / encounter / procedure ที่เป็นต้นทาง

รองรับสถานะ `ชำระแล้ว`, `ชำระบางส่วน`, `รอชำระ`, ช่องทางเงินสด/บัตร/PromptPay-QR/โอน/สิทธิ์ตามที่เปิดใช้, ใบเสร็จ และ aging: ยังไม่เกินกำหนด, 1–7 วัน, 8–30 วัน, เกิน 30 วัน

### DF review

อย่าฝังสูตร DF ไว้ใน visual โดยไม่รู้ policy ของคลินิก ให้แสดงเป็น breakdown ที่ตรวจสอบได้:

```text
DF รอบ: [เดือน]   provider: [รายชื่อ]   branch scope: [สาขา]
ฐานคำนวณ: [ตาม policy ที่อนุมัติ]
ยอดบริการ     - ค่าแล็บ/วัสดุ     - ส่วนลด     = ฐาน DF
งานแพ็กเกจ / งานค้างข้ามเดือน / ยอดยังไม่เก็บ: แสดงเป็น rule flag

[ส่งตรวจสอบ] [ขอแก้ไขพร้อมเหตุผล] [ดู audit trail]
```

ทุก adjustment, void, refund, discount และ DF change ต้องขอเหตุผล/ผู้อนุมัติ และเพิ่ม audit event ไม่ลบรายการเดิม

## 11. Screen 08 — Inbox และ AI Messages

### เป้าหมาย

รวมบทสนทนาให้เชื่อม Patient 360, appointment และ branch context พร้อมทำให้เจ้าหน้าที่เห็นทันทีว่าอะไรตอบได้ อะไรต้องส่งต่อ

### โครงหน้าจอ

```text
Inbox     [สาขา ▾] [LINE OA | Website Chat] [tag ▾] [รอคนรับช่วง]

┌──────────────────────┬─────────────────────────────┬────────────────────┐
│ conversation list    │ transcript                   │ context / action    │
│ • ขอนัดหมาย          │ คนไข้ตัวอย่าง: ...           │ คนไข้ตัวอย่าง 01    │
│ • ขอเลื่อนนัด         │ AI: ...                      │ สาขา B              │
│ • สอบถามราคา          │ Staff: ...                   │ นัดหมายถัดไป        │
│ • ปวดฉุกเฉิน          │                              │ แผน/ยอด/consent     │
│ • ติดตามแผนรักษา      │ [approved reply source]      │ [เปิด Patient 360]  │
│                      │ [ข้อความตอบกลับ]             │ [ส่งต่อทันตแพทย์]   │
└──────────────────────┴─────────────────────────────┴────────────────────┘
```

### AI safety UI

- แสดง banner คงที่: **AI ช่วยงานสื่อสารและประสานงานเท่านั้น — ไม่วินิจฉัย ไม่สั่งยา ไม่ตีความ X-ray**
- AI draft มี confidence, source/approved template, เวลา draft และผู้ตรวจสอบ
- คำถามที่ต้องใช้ clinical judgment จะเปลี่ยนเป็น handoff state อัตโนมัติ พร้อม transcript และ context ที่จำเป็น
- ข้อความหลังหัตถการที่มีความเสี่ยงเลือกได้เฉพาะ approved template และแสดง escalation path
- ถ้าไม่ทราบสาขา ให้ถามเพื่อยืนยันหรือเข้าคิวกลางตาม policy; ห้าม route ไปสาขาเองจากการเดา
- Facebook/Instagram ไม่ควรแสดงเป็นช่องทาง production จนกว่าจะมี contract ยืนยัน

## 12. Screen 09 — Reports และ Insights

### โครงสร้าง

แบ่งเป็น 4 tabs เพื่อไม่ปะปนข้อมูล:

1. **Operations** — นัด, queue, no-show, cancellation, resource gap
2. **Patient follow-up** — return visit, recall, plan follow-up, consent status ตามสิทธิ์
3. **Lab & treatment** — งานเสี่ยง, turnaround, revision, treatment plan stage
4. **Finance** — revenue, payment, aging, installment, DF ตาม branch/entity policy

ทุก report header ต้องบอก `ช่วงเวลา`, `สาขาที่รวม`, `ผู้ใช้ที่ขอ`, `เวลา generate`, `source dataset` และระดับข้อมูลที่ถูก mask

### Drill-down

- คลิก branch row เพื่อเปิด branch dashboard โดยรักษา filter เดิม
- คลิก metric เพื่อไป operational list ที่ทำ action ต่อได้
- export ตามสิทธิ์เท่านั้น พร้อม watermark metadata ของ scope และ timestamp
- ถ้านิยาม metric ยังไม่อนุมัติ ให้แสดงสถานะ `รอนิยาม` ไม่แสดงเป็น KPI ทางธุรกิจ

## 13. Screen 10 — Organization, Branches และ Access

### Branch management

แต่ละ branch card แสดง:

- ชื่อ, ที่อยู่/พิกัด, ช่องทางติดต่อ
- เวลาทำการ, วันหยุด, รับนัดหรือไม่
- เก้าอี้, ทันตแพทย์, ผู้ช่วย, เวลาทำงาน
- ช่องทางที่เชื่อม, สาขาที่รับชำระ, status active/inactive
- `ดูประวัติการเปลี่ยนแปลง`

การปิดสาขาเป็น soft close: ปิดรับนัดใหม่ตาม policy แต่ไม่ลบ appointment, encounter, lab, invoice หรือ audit history เดิม

### Users / roles / branch scope

ใช้ matrix ที่อ่านง่ายกว่าการใช้ checkbox ยาว ๆ:

| ผู้ใช้ | Role | สาขาที่เข้าถึง | Clinical | Finance | Export | สถานะ |
|---|---|---|---|---|---|---|
| ผู้ใช้สมมติ 01 | Counter | A, B | จำเป็นต่อการทำงาน | รับชำระ | จำกัด | Active |
| ผู้ใช้สมมติ 02 | Dentist | B | เคสที่มอบหมาย | ไม่ได้ | ไม่ได้ | Active |

การเปลี่ยน role, branch membership, ผู้รับผิดชอบ หรือการปิด user ต้องแสดง effective time และสร้าง audit event

### Catalog / policy settings

แยก `ระดับเครือ` กับ `override รายสาขา` ให้เห็นใน UI แต่ติด badge **ต้องตัดสินใจ** จนกว่า product/finance จะอนุมัติ behavior สำหรับ service, procedure, package, promotion, price, loyalty, DF และ invoice numbering

## 14. Screen 11 — Consent, Audit และ Data Rights

### Consent center

แยก purpose อย่างน้อย:

- การรักษา
- นัดหมาย
- เรียกเก็บเงิน
- ข่าวสาร/โปรโมชัน

แต่ละรายการแสดงสถานะ, version ของข้อความ, หลักฐาน, เวลา, ช่องทาง, ผู้บันทึก และ withdrawal history

### Audit log

ค้นได้ตาม actor, role, branch, object, action, time, reason และ result ครอบคลุมการอ่าน/สร้าง/แก้ไข/export ข้อมูลสุขภาพ, การเงิน, consent, permission, appointment lock, billing adjustment และ DF

### Data rights request

รองรับ request access/export/delete โดยแสดง:

- requester verification
- data scope และ branch scope
- clinical/financial retention exception
- ผู้อนุมัติและสถานะ
- export package / completion timestamp

UI ไม่ควรใช้ copy ว่า “ลบทั้งหมดทันที” เพราะต้องคงหลักฐานที่กฎหมาย/นโยบาย retention บังคับเก็บ

## 15. Critical states / edge cases

### Booking conflict

```text
ช่วงเวลานี้เพิ่งถูกจองโดยรายการอื่น
ทรัพยากรที่ชน: เก้าอี้ 01 · ผู้ให้บริการสมมติ 01
สาขา: B · เวลา: 10:30–11:30

[เลือกช่วงเวลาอื่น] [กลับไปแก้ข้อมูล] [ส่งต่อเคาน์เตอร์]
```

ห้ามแสดงว่าจองสำเร็จจาก optimistic UI ก่อน server ยืนยัน

### Cross-branch access denied

“พบ Patient 360 เดิมจากสาขา A แต่ role ของคุณยังไม่มีสิทธิ์ดูข้อมูลสุขภาพข้ามสาขา” พร้อมปุ่ม `ขอสิทธิ์/ส่งต่อ` และไม่แสดง preview ที่ทำให้อนุมานข้อมูลสุขภาพได้

### Consent missing

แสดงข้อมูลเท่าที่จำเป็นต่อการนัดหมาย และ lock action ที่ต้องใช้ consent พร้อมบอก purpose ที่ขาด ไม่ใช้ error สีแดงอย่างเดียว

### Lab deadline collision

แสดง warning พร้อม due date, fitting appointment, buffer, owner และ policy mode (`warning` / `require approval` / `ยังไม่ตั้งค่า`) ไม่ hard-block จนกว่า clinical owner จะยืนยันกติกา

### Payment adjustment

เปิด approval drawer ที่มี before/after, reason, affected invoice/receipt/DF, actor, approver และ audit preview

### AI handoff

เปลี่ยน owner จาก AI เป็นทีม/ทันตแพทย์, คง transcript, tag, branch context, suggested summary และเวลาที่ส่งต่อ พร้อมสถานะว่าผู้รับช่วงรับงานแล้วหรือยัง

## 16. Responsive, accessibility และ performance

- Desktop เป็น primary สำหรับ dashboard, schedule, lab board และ finance table
- Tablet เป็น primary สำหรับ counter/assistant ระหว่างเดินหน้างาน
- Narrow view ใช้ agenda/list และ stacked cards; action สำคัญต้องแตะได้อย่างน้อยประมาณ 44 px
- ทุก icon button มี accessible name; status ไม่สื่อด้วยสีอย่างเดียว ต้องมี text และ icon/shape ประกอบ
- focus ต้องเห็นชัด, keyboard ใช้กับ filter, tab, dialog และ table action ได้
- ตารางยาวต้องมี non-drag alternative, sticky header เฉพาะที่ช่วยอ่าน และไม่ซ่อน focus ไว้ใต้ sticky element
- X-ray/file preview ต้องมี alt/metadata และไม่โหลดไฟล์เต็มก่อนผู้ใช้ขอ
- ใช้ reserved dimensions, lazy load สำหรับ chart/file preview, ลด JS ที่ไม่เกี่ยวกับ operation
- ตั้งเป้า WCAG 2.2 AA และตรวจบน narrow width, enlarged text, slow loading, rejected request และ reduced motion

## 17. Delivery order

### P0 — เปิดใช้ operation ได้อย่างปลอดภัย

1. Application shell + branch context + role gate
2. Schedule + appointment detail + server conflict/error states
3. Queue + check-in + encounter handoff
4. Patient 360 + identity/duplicate gate + family/guardian + consent
5. Treatment Record + Treatment Plan พื้นฐาน + X-ray/file permission
6. Lab board + risk flag พื้นฐาน + revision history
7. Billing/Payment/Receipt/partial payment + DF audit trail
8. Inbox LINE OA/Website Chat + approved AI draft + human handoff
9. Audit log และ access-denied states ในทุก module

### P1 — ทำให้เครือหลายสาขาบริหารได้ลึกขึ้น

1. HQ dashboard + branch drill-down + operational insights
2. Branch transfer ของนัด/เคส/แล็บ พร้อม approval
3. Aging/recall/orthodontic/lab notifications
4. Shared catalog + branch override
5. Duplicate merge, import/export และ reconciliation
6. Approval workflow สำหรับ bill/DF และ lab risk

## 18. Decisions ที่ต้องปิดก่อนล็อก UI contract

| Decision | จุดที่กระทบหน้าจอ |
|---|---|
| HN เดียวทั้งเครือหรือแยกสาขา | Patient search, duplicate review, patient header |
| สิทธิ์เห็น clinical record ข้ามสาขา | Patient 360, treatment, X-ray, handoff |
| ใครอนุมัติ branch/case transfer | Schedule, treatment, lab, DF |
| shared/local service, price, package, loyalty | Catalog, appointment, billing |
| สาขาให้บริการ vs รับชำระ vs นิติบุคคล | Invoice, receipt, finance reports |
| สูตร DF, ค่าแล็บ, ส่วนลด, package, งานค้างข้ามเดือน | DF breakdown, approval, audit |
| no-show, cancellation, deposit, refund | Appointment actions, billing, notification |
| lab risk เป็น warning หรือ require approval | Lab board, appointment readiness |
| AI approved knowledge/template ต่อสาขา | Inbox, escalation, post-treatment message |
| retention, backup, export/delete และ data residency | Consent, audit, data rights, settings |
| Facebook/Instagram อยู่ใน production scope หรือไม่ | Inbox channel filter, routing |

## 19. Design conclusion

หน้าจอที่ควรเป็น “ศูนย์กลาง” ของระบบไม่ใช่ Dashboard อย่างเดียว แต่คือการเชื่อม **Branch context + Schedule + Patient 360 + Lab readiness + Payment/DF + human handoff** ในทุก transition ของงาน คนใช้ไม่ควรต้องจำว่าข้อมูลอยู่ระบบไหน และระบบไม่ควรทำให้ผู้ใช้เดาเองว่ารายการนี้เกิดที่สาขาใด ใครเป็นผู้รับผิดชอบ หรือแก้ไขได้แค่ไหน

ข้อเสนอชุดนี้จึงวาง multi-branch เป็นโครงสร้างของทุกหน้า ไม่ใช่แค่ filter เพิ่มบน Dashboard และวาง permission/consent/audit เป็น visible product behavior ตั้งแต่ต้น ไม่ใช่สิ่งที่ค่อยเติมภายหลัง
