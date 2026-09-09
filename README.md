# workflow-zozeei

ชุดสกิลภาษาไทยสำหรับ AI Agents (Claude Code, Codex, Gemini CLI) ประกอบด้วย 2 สกิลหลัก:

1. **`workflow-zozeei`**: นำทางจาก requirement ไปสู่การลงมือที่ตรวจผลได้ — ทำความต้องการให้ชัด เลือกวิธีที่เรียบง่าย แตกงาน ลงมือ และตรวจผล
2. **`security-zozeei`**: ตรวจสอบความปลอดภัยจาก code และ trust boundaries — ใช้ **ValidatesSafeInput 14 กลุ่มกฎ**, **OWASP Top 10 Web (2025)** และ **OWASP Mobile Top 10 (2024)** พร้อมแยกช่องโหว่ที่ยืนยันได้ สิ่งที่ต้องตรวจเพิ่ม และข้อเสนอ hardening

ตัวสกิลเป็น Markdown ไม่มี script หรือ dependency บังคับ มี `SKILL.md` เป็นขั้นตอนหลักและ `references/` สำหรับโหลดเฉพาะหัวข้อที่ใช้ ไม่ต้องอ่าน checklist Web และ Mobile พร้อมกันทุกงาน

Workflow ใช้สิทธิ์ที่ผู้ใช้ให้ไว้ต่อเนื่องข้าม Phase; คำขอวางแผนอย่างเดียวไม่ให้สิทธิ์แก้ระบบ งานเล็กทำด้วย agent เดียวได้ การเลือกโมเดล/delegation ขึ้นกับความสามารถจริงของ runtime

## ติดตั้ง

เลือกทางเดียว ติดตั้งทั้งสองทางจะได้สกิลซ้ำสองชุด

**Claude Code**

```bash
claude plugin marketplace add zozeei/workflow-zozeei
claude plugin install workflow-zozeei@workflow-zozeei
```

(ใน Claude Code ใช้ `/plugin marketplace add ...` และ `/plugin install ...` ได้เช่นกัน)

**Codex, Gemini CLI และ agent อื่น**

```bash
npx skills@latest add zozeei/workflow-zozeei
```

คำสั่ง `npx` ต้องมี Node.js/npm เลือกติดตั้งทั้ง `workflow-zozeei` และ `security-zozeei` พร้อม agent เป้าหมายตามตัวเลือกของ CLI ค่าเริ่มต้นเป็นระดับโปรเจกต์ เติม `-g` สำหรับระดับผู้ใช้ ตำแหน่งและ symlink/copy ขึ้นกับ agent/วิธีติดตั้ง

หากยังไม่พบสกิลหลังติดตั้ง ให้เริ่มเครื่องมือนั้นใหม่

## เรียกใช้

### 1. เรียกใช้ `workflow-zozeei` (วางแผนและพัฒนา)

| เครื่องมือ | วิธีเรียก |
|---|---|
| Claude Code (plugin) | `/workflow-zozeei:workflow-zozeei ตามด้วยงานที่ต้องการ` |
| Codex CLI / IDE | `$workflow-zozeei ตามด้วยงานที่ต้องการ` |
| Gemini CLI | `ใช้สกิล workflow-zozeei` ตามด้วยงานที่ต้องการ |

ตัวอย่าง:
```text
/workflow-zozeei:workflow-zozeei ช่วยวางแผนเพิ่มระบบอนุมัติเอกสาร ยังไม่ต้องแก้ไฟล์
```

### 2. เรียกใช้ `security-zozeei` (ตรวจสอบความปลอดภัย)

| เครื่องมือ | วิธีเรียก |
|---|---|
| Claude Code (plugin) | `/workflow-zozeei:security-zozeei ตามด้วยงานที่ต้องการตรวจ` |
| Codex CLI / IDE | `$security-zozeei ตามด้วยงานที่ต้องการตรวจ` |
| Gemini CLI | `ใช้สกิล security-zozeei` ตามด้วยงานที่ต้องการตรวจ |

ตัวอย่าง:
```text
/workflow-zozeei:security-zozeei ตรวจสอบจุดรับข้อมูลในโฟลเดอร์ src/ ตามกฎ ValidatesSafeInput
```
```text
ใช้สกิล security-zozeei ตรวจสอบความปลอดภัยของ Web API ตามมาตรฐาน OWASP Web 2025
```
```text
/workflow-zozeei:security-zozeei ตรวจสอบ AndroidManifest และ Local Storage ตามมาตรฐาน OWASP Mobile 2024
```

## อัปเดต / เลิกใช้

- Claude Code: `/plugin update workflow-zozeei` — เลิกใช้ด้วย `/plugin uninstall workflow-zozeei`
- skills CLI: `npx skills@latest update workflow-zozeei security-zozeei`
- ถอนผ่าน skills CLI: `npx skills@latest remove workflow-zozeei security-zozeei` เติม `-g` หากติดตั้งระดับผู้ใช้

ตารางคำสั่ง Claude ด้านบนใช้ namespace ของ plugin; หากติดตั้งเป็น standalone skills ให้ใช้ `/workflow-zozeei` และ `/security-zozeei` ตรวจชื่อที่ปรากฏในรายการสกิลของ runtime หลังติดตั้ง

## หมายเหตุ

Workflow ใช้สกิลจาก **Ponytail และ Matt Pocock เป็นหลักตามหน้าที่** เมื่อมี เช่น Ponytail สำหรับความเรียบง่าย/over-engineering และ Matt Pocock สำหรับ requirement, debugging, spec/tickets, TDD และ code review ถ้าไม่มีสกิลหลักที่ตรงงาน จึงเลือกสกิลอื่น; ถ้าไม่มีสกิลที่เหมาะสมเลยจึงใช้ขั้นตอนพื้นฐาน

สองแพ็กเกจนี้ไม่ได้ติดตั้งมาพร้อม workflow และไม่ถูกติดตั้งเพิ่มอัตโนมัติ ชื่อ/namespace อาจต่างตามรุ่น จึงตรวจ catalog/ที่มาก่อนเลือก การเลือกสกิลหลักไม่เพิ่มสิทธิ์สร้าง issue, commit หรือ deploy และไม่บังคับให้รันครบทุกสกิลในงานเล็ก

Security audit เป็น read-only เว้นแต่ผู้ใช้ขอให้แก้หรือบันทึก artifact โดยตรง ผลตรวจระบุ scope, edition, evidence และข้อจำกัด ไม่ใช่การรับรองว่าระบบปลอดภัยทั้งหมด แหล่งมาตรฐานอยู่ข้างกฎในแต่ละ reference

ดู [สถานการณ์ทดสอบสกิล](tests/skill-scenarios.md) สำหรับตรวจพฤติกรรมหลังเปลี่ยนคำสั่ง การแยก reference ช่วยลดเนื้อหาที่ต้องโหลด แต่ประสิทธิภาพด้านเวลา/token ต้องวัดกับงานและ runtime จริง

อ้างอิง: [Claude Code plugins](https://code.claude.com/docs/en/plugin-marketplaces),
[skills CLI](https://github.com/vercel-labs/skills),
[Agent Skills spec](https://agentskills.io/specification)
