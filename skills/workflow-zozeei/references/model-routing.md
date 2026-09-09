# Model Routing / Delegation

อ่านเมื่อจะ delegate หรือเลือก model/effort เท่านั้น กฎสิทธิ์และขอบเขตอยู่ใน [SKILL.md](../SKILL.md)

## เลือก capability จาก runtime จริง

| Tier | งานที่เหมาะ |
|---|---|
| `FAST` | ค้นไฟล์/symbol, รวบรวม logs/tests/docs, mechanical edit ที่ตรวจผลได้ง่าย |
| `BALANCED` | implementation และ review ทั่วไปที่ขอบเขตชัด |
| `BEST_REASONING` | architecture, core logic, security/auth, data integrity, concurrency, debugging ซับซ้อน, external effects สำคัญ |

Tier เป็นเป้าหมายในการเลือก ไม่ใช่ชื่อโมเดลหรือคำรับรองคุณภาพ ใช้รายการ model/config และเครื่องมือที่ runtime/account เปิดให้จริงใน session นี้ ไม่เดาชื่อหรือคัดลอก config ข้าม Claude Code, Codex, Gemini CLI หรือ Antigravity

ถ้าเลือก effort ได้ ให้ปรับตามความยาก/ความเสี่ยง ไม่ตั้งระดับสูงสุดกับทุกงาน ถ้าเลือก model ต่อ subagent ไม่ได้ ให้ inherit model; ถ้าไม่มี subagent ให้ agent ปัจจุบันทำต่อ ไม่อ้างว่าได้ใช้ model/tier/fresh context ที่ไม่ได้เกิดขึ้นจริง

## เมื่อ delegation คุ้มค่า

- ส่งงานอิสระที่มีผลลัพธ์ชัด เช่นค้น call sites คนละ subsystem หรือ independent review ของจุดเสี่ยง
- มีงานที่ agent หลักทำต่อได้ระหว่างรอ และใช้ผลที่ส่งกลับตัดสินใจได้โดยไม่อ่านซ้ำทั้งหมด
- ให้ ownership ไฟล์ไม่ทับกันเมื่อมีผู้แก้หลายคน และเรียงงานที่มี dependency ก่อนส่งต่อ
- งานเล็กหรือขั้นที่ต้องใช้บริบทเดียวกันให้ทำเอง อย่าสร้าง Scout/Planner/Reviewer หลายตัวเพียงเพื่อให้ครบบทบาท

## ข้อตกลงการส่งงาน

ทุก task ระบุ goal, repo-relative scope/ไฟล์ที่เป็นเจ้าของ, evidence/inputs ที่จำเป็น, constraints/สิทธิ์, acceptance criteria และ verification ระบุ decision ที่ยังห้ามสมมติให้ชัด

ส่งต่อชื่อ/ที่มาของสกิลหลักที่เลือกแล้ว และลำดับ Ponytail/Matt Pocock → สกิลอื่นเมื่อไม่มีตัวหลักที่ตรงงาน → ขั้นตอนพื้นฐาน ตามนโยบายใน SKILL.md ให้ผู้รับงานใช้ลำดับเดียวกันเมื่อ catalog ของตนต่างจาก agent หลัก

| Role | หน้าที่และสิ่งที่ส่งกลับ |
|---|---|
| Scout / Reader | ส่งไฟล์/บรรทัด/symbol, execution path ที่ตรวจพบ, patterns, tests, invariants และ unknowns; เสนอ observation ไม่ตัดสิน architecture/security policy |
| Planner / Decision Maker | ตรวจหลักฐานสำคัญ เลือก approach/trade-off กำหนด scope, acceptance criteria, verification และผู้รับผิดชอบ |
| Implementer | ทำตามข้อตกลง ส่ง diff summary และผล verification; เมื่อหลักฐานขัดแผนให้ส่งกลับ Planner และพักเฉพาะงานที่ขึ้นกับ decision นั้น |
| Reviewer | ตรวจ code/requirement/tests โดยตรง ส่ง `ship`, `fix-then-ship` หรือ `rework`, findings พร้อม severity/evidence และ verification ที่ยังขาด |

งานทั่วไปเลือก FAST/BALANCED ตามความยาก งานเกี่ยวกับ invariants หรือข้อมูลสำคัญเลือก BEST_REASONING ที่เข้าถึงได้ สำหรับ high-risk review ใช้ fresh context เมื่อทำได้

Planner ตรวจผลที่จำเป็นก่อนรวมงาน รายงานของ Implementer และ model tier ไม่แทน tests/compiler/typecheck/static analysis หรือ runtime check ที่รันจริง
