# Zentr Dental — Multi-Branch Console (Prototype)

Prototype และเอกสารออกแบบระบบบริหารคลินิกทันตกรรม **Zentr Dental** สำหรับคลินิกที่มีหลายสาขา
โฟกัสที่มุมมองผู้บริหาร (executive control tower) พร้อม branch context, การเงิน/DF และรายงาน

> ⚠️ ข้อมูลทั้งหมดใน prototype เป็น **synthetic demo** เท่านั้น — ไม่ใช่ข้อมูลคนไข้จริง, HN จริง หรือข้อมูลสุขภาพจริง
> ตัวเลข สิทธิ์ นโยบาย และสูตร (เช่น DF) ต้องเชื่อม server และผ่านการอนุมัติก่อนถือเป็น production behavior

## ไฟล์ในโปรเจกต์

| ไฟล์ | รายละเอียด |
|---|---|
| `zentr-dental-multi-branch-dashboard.html` | Prototype ระบบครบทุกหน้า (single-file, เปิดในเบราว์เซอร์ได้เลย) |
| `zentr-dental-multi-branch-requirements.md` | Requirements baseline สำหรับคลินิกหลายสาขา |
| `zentr-dental-multi-branch-screen-design.md` | ข้อเสนอ UX/UI รายหน้าจอ |
| `zentr-knowledge-base.md` | Knowledge base อ้างอิงของ Zentr |

## หน้าใน prototype

- **ภาพรวมเครือคลินิก** — executive view: รายรับ/รับชำระ/ยอดค้าง (กราฟ SVG), รายการที่ต้องตัดสินใจ, เปรียบเทียบสุขภาพรายสาขา, treatment pipeline, risk signals
- **นัดหมายและคิว** · **คนไข้ (Patient 360)** · **การรักษา** · **งานแล็บ (Lab Control Tower)**
- **การเงินและ DF** — บิลแยกสาขาให้บริการ/รับชำระ, aging, Doctor Fee review, approval trail
- **Inbox & AI** · **รายงานและ Insights** (4 มุมมอง) · **ตั้งค่าองค์กร**

ทุกหน้า scope-aware ตาม branch switcher (ทุกสาขา / รายสาขา) โดย mutation ที่ target ไม่ชัดต้องเลือกสาขาก่อน

## การใช้งาน

เปิดไฟล์ `zentr-dental-multi-branch-dashboard.html` ในเบราว์เซอร์ได้โดยตรง — ไม่ต้อง build หรือ dependency ใดๆ
