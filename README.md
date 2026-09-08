# workflow-zozeei

สกิลภาษาไทยสำหรับวางแผนและพัฒนาซอฟต์แวร์ — ทำ requirement ให้ชัด เลือกวิธี
แตกงาน ลงมือ และตรวจผล รวมเส้นทางวินิจฉัยบั๊กและการตรวจ findings ก่อนแก้ตามรีวิว

ต้นฉบับมีไฟล์เดียวคือ `skills/workflow-zozeei/SKILL.md` ไม่มี script
หรือ dependency ที่ต้องติดตั้ง

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

ค่าเริ่มต้นติดตั้งลงโปรเจกต์ปัจจุบัน (`.agents/skills/`) เติม `-g` เพื่อติดตั้งระดับผู้ใช้

หากยังไม่พบสกิลหลังติดตั้ง ให้เริ่มเครื่องมือนั้นใหม่

## เรียกใช้

| เครื่องมือ | วิธีเรียก |
|---|---|
| Claude Code | `/workflow-zozeei ตามด้วยงานที่ต้องการ` |
| Codex CLI / IDE | `$workflow-zozeei ตามด้วยงานที่ต้องการ` |
| Gemini CLI | `ใช้สกิล workflow-zozeei` ตามด้วยงานที่ต้องการ |

ตัวอย่าง:

```text
/workflow-zozeei ช่วยวางแผนเพิ่มระบบอนุมัติเอกสาร ยังไม่ต้องแก้ไฟล์
```

```text
ใช้ workflow-zozeei วางแผนแก้บั๊ก ผู้ใช้แก้ไขโปรไฟล์ของคนอื่นได้
ตรวจหาสาเหตุก่อน ยังไม่ต้องแก้ไฟล์
```

## อัปเดต / เลิกใช้

- Claude Code: `/plugin update workflow-zozeei` — เลิกใช้ด้วย `/plugin uninstall workflow-zozeei`
- skills.sh: `npx skills@latest update workflow-zozeei` — เลิกใช้โดยลบโฟลเดอร์สกิล

## หมายเหตุ

สกิลที่ `SKILL.md` อ้างถึง (`grilling`, `diagnosing-bugs`, `ponytail`, `tdd`,
`code-review`, `scrutinize` ฯลฯ) ไม่ได้รวมมาในชุด หากเครื่องนั้นไม่มี AI จะใช้ขั้นตอนพื้นฐานแทน

อ้างอิง: [Claude Code plugins](https://code.claude.com/docs/en/plugin-marketplaces),
[skills CLI](https://github.com/vercel-labs/skills),
[Agent Skills spec](https://agentskills.io/specification)
