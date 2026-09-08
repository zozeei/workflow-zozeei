# workflow-zozeei

สกิลภาษาไทยสำหรับวางแผนและพัฒนาซอฟต์แวร์อย่างเรียบง่าย
ประกอบด้วยการทำ requirement ให้ชัด เลือกวิธี แตกงาน ลงมือ และตรวจผล
รวมเส้นทางวินิจฉัยบั๊ก การรับมือผลตรวจผิดคาด และการตรวจ findings ก่อนแก้ตามรีวิว

ใช้ `SKILL.md` ไฟล์เดียวเป็นต้นฉบับสำหรับเครื่องมือที่รองรับ Agent Skills
คู่มือนี้ตรวจสอบเอกสารทางการเมื่อ 8 กันยายน 2026

## ไฟล์ในชุด

```text
workflow-zozeei/
├── .claude-plugin/
│   ├── marketplace.json    ทำให้ repository นี้เป็น marketplace ของตัวเอง
│   └── plugin.json         ข้อมูล plugin สำหรับ Claude Code
├── skills/
│   └── workflow-zozeei/
│       └── SKILL.md        คำแนะนำที่ AI อ่าน (ต้นฉบับไฟล์เดียว)
└── README.md               วิธีติดตั้ง ใช้งาน และย้ายเครื่อง
```

ชุดนี้ไม่มี executable script หรือ dependency ที่ต้องติดตั้ง
สกิลที่อ้างถึง เช่น `grilling`, `grill-me`, `diagnosing-bugs`, `ponytail`,
`to-tickets`, `tdd`, `ponytail-review` และ `code-review` ไม่ได้รวมมาในชุด
หากเครื่องนั้นไม่มี ให้ AI ใช้ขั้นตอนพื้นฐานใน `SKILL.md` แทน
หากต้องการ workflow ของสกิลย่อยฉบับเต็ม ต้องติดตั้งสกิลนั้นแยกต่างหาก
ชื่อสกิลเดียวกันอาจมีหน้าที่ต่างกัน จึงต้องอ่านขอบเขตก่อนเรียก

## ติดตั้งจาก GitHub

Repository: [zozeei/workflow-zozeei](https://github.com/zozeei/workflow-zozeei)

มีสองเส้นทาง เลือกทางเดียว การติดตั้งทั้งสองทางจะได้สกิลซ้ำสองชุด

- **Claude Code plugin** — ติดตั้งเป็นชุดที่ระบบจัดการให้ อัปเดตด้วย `/plugin update`
- **skills.sh** — คัดลอกไฟล์สกิลลงเครื่องเป็นไฟล์ที่แก้เองได้ ใช้ได้กับ Codex, Gemini CLI และ agent อื่น

### Claude Code

repository นี้เป็น marketplace ของตัวเอง จึงต้องเพิ่ม marketplace ก่อนหนึ่งครั้ง
แล้วจึงติดตั้ง plugin:

```text
/plugin marketplace add zozeei/workflow-zozeei
/plugin install workflow-zozeei@workflow-zozeei
```

หรือรันจาก terminal:

```bash
claude plugin marketplace add zozeei/workflow-zozeei
claude plugin install workflow-zozeei@workflow-zozeei
```

ตรวจผลด้วย `/plugin` หรือ `/skills` แล้วเรียกใช้ด้วย `/workflow-zozeei`
อัปเดตภายหลังด้วย `/plugin update workflow-zozeei`

อ้างอิง: [Claude Code plugins](https://code.claude.com/docs/en/discover-plugins),
[plugin marketplaces](https://code.claude.com/docs/en/plugin-marketplaces)

### Codex, Gemini CLI และ agent อื่น

plugin ด้านบนใช้ได้กับ Claude Code เท่านั้น เครื่องมืออื่นใช้ตัวติดตั้ง
[skills.sh](https://skills.sh) ซึ่งคัดลอกไฟล์สกิลลงเครื่อง:

```bash
npx skills@latest add zozeei/workflow-zozeei
```

ตัวติดตั้งจะถามว่าจะติดตั้งลง agent ตัวใด เลือก `codex`, `gemini-cli`
หรือตัวอื่นที่ใช้อยู่ ระบุตรง ๆ ได้ด้วย:

```bash
npx skills@latest add zozeei/workflow-zozeei -a codex gemini-cli
```

ค่าเริ่มต้นติดตั้งลงโปรเจกต์ปัจจุบัน (`.agents/skills/`)
เติม `-g` เพื่อติดตั้งระดับผู้ใช้ (`~/.codex/skills/`, `~/.gemini/skills/`)
อัปเดตภายหลังด้วย `npx skills@latest update workflow-zozeei`

เมื่อติดตั้งเสร็จแล้ว หากยังไม่พบสกิล ให้เริ่มเครื่องมือนั้นใหม่

อ้างอิง: [Codex skills](https://developers.openai.com/codex/skills),
[Gemini CLI skills](https://geminicli.com/docs/cli/using-agent-skills/),
[skills CLI](https://github.com/vercel-labs/skills)

## เรียกใช้สกิล

`/workflow-zozeei` ใช้กับ Claude Code ครับ แต่แต่ละเครื่องมือใช้รูปแบบต่างกัน:

| เครื่องมือ | วิธีเรียกโดยตรง |
|---|---|
| Codex CLI / IDE | `$workflow-zozeei ตามด้วยงานที่ต้องการ` |
| Claude Code | `/workflow-zozeei ตามด้วยงานที่ต้องการ` |
| Gemini CLI | พิมพ์ `ใช้สกิล workflow-zozeei` ตามด้วยงานที่ต้องการ |

ตัวอย่าง:

```text
# Codex
$workflow-zozeei ช่วยวางแผนเพิ่มระบบอนุมัติเอกสาร ยังไม่ต้องแก้ไฟล์
```

```text
# Claude Code
/workflow-zozeei ช่วยวางแผนเพิ่มระบบอนุมัติเอกสาร ยังไม่ต้องแก้ไฟล์
```

```text
# Gemini CLI
ใช้สกิล workflow-zozeei ช่วยวางแผนเพิ่มระบบอนุมัติเอกสาร ยังไม่ต้องแก้ไฟล์
```

Codex และ Claude Code อาจเลือกใช้สกิลโดยอัตโนมัติเมื่อคำขอตรงกับ `description`
ส่วน Gemini CLI เปิดใช้สกิลจากคำขอภาษาธรรมชาติและอาจขออนุญาตก่อนเปิดใช้
การระบุชื่อสกิลโดยตรงช่วยให้เลือก workflow นี้ได้แน่นอนขึ้น

## ติดตั้งด้วยการคัดลอกไฟล์

ใช้เมื่อไม่ต้องการตัวติดตั้ง เช่น เครื่องที่ไม่มี Node.js
คำสั่งด้านล่างสำหรับ Linux, macOS หรือ WSL โดยรันจาก root ของ repository
ที่ clone หรือแตก ZIP ไว้ (โฟลเดอร์ที่มี `skills/workflow-zozeei/`)
หากติดตั้งแล้ว ให้ใช้ขั้นตอนอัปเดตด้านล่างแทนการคัดลอกซ้อน

### Codex และ Gemini CLI

ทั้งสองรองรับโฟลเดอร์ส่วนตัว `~/.agents/skills/`
คัดลอกครั้งเดียวในเครื่องเดียวกันก็ให้ทั้งสองค้นพบได้

```bash
mkdir -p "$HOME/.agents/skills"
cp -R ./skills/workflow-zozeei "$HOME/.agents/skills/"
```

ปลายทางต้องเป็น `~/.agents/skills/workflow-zozeei/SKILL.md`
Gemini CLI รองรับ `~/.gemini/skills/` ด้วย เลือกตำแหน่งใดตำแหน่งหนึ่ง

อ้างอิง: [Codex skills](https://learn.chatgpt.com/docs/build-skills),
[Gemini CLI skills](https://geminicli.com/docs/cli/using-agent-skills/)

### Claude Code

```bash
mkdir -p "$HOME/.claude/skills"
cp -R ./skills/workflow-zozeei "$HOME/.claude/skills/"
```

ปลายทางต้องเป็น `~/.claude/skills/workflow-zozeei/SKILL.md`

อ้างอิง: [Claude Code skills](https://code.claude.com/docs/en/skills)

### Windows ที่ไม่ได้ใช้ WSL

ใช้ File Explorer คัดลอกโฟลเดอร์ `skills\workflow-zozeei` ไปใต้ตำแหน่งเหล่านี้
สร้างโฟลเดอร์แม่หากยังไม่มี:

- Codex และ Gemini CLI: `%USERPROFILE%\.agents\skills\`
- Claude Code: `%USERPROFILE%\.claude\skills\`

หากใช้ WSL ให้ติดตั้งใน home ของ WSL ด้วยคำสั่ง Linux ด้านบน

## ติดตั้งเฉพาะโปรเจกต์

คัดลอกโฟลเดอร์สกิลไปยังตำแหน่งภายในโปรเจกต์:

| เครื่องมือ | ตำแหน่งไฟล์ |
|---|---|
| Codex | `.agents/skills/workflow-zozeei/SKILL.md` |
| Claude Code | `.claude/skills/workflow-zozeei/SKILL.md` |
| Gemini CLI | `.agents/skills/workflow-zozeei/SKILL.md` หรือ `.gemini/skills/workflow-zozeei/SKILL.md` |

เลือกติดตั้งระดับผู้ใช้หรือโปรเจกต์ตามการใช้งาน
หลีกเลี่ยงหลายสำเนาชื่อเดียวกันโดยไม่ตั้งใจ เพราะกติกาการเลือกสำเนาของแต่ละเครื่องมือต่างกัน
Gemini CLI อาจต้องให้ความเชื่อถือกับ workspace ก่อนใช้งานสกิลของโปรเจกต์

อ้างอิง: [Codex](https://learn.chatgpt.com/docs/build-skills),
[Claude Code](https://code.claude.com/docs/en/skills),
[Gemini CLI](https://geminicli.com/docs/cli/tutorials/skills-getting-started/)

## ตรวจสกิลหลังติดตั้งด้วยการคัดลอก

### Codex CLI / IDE

เปิด `/skills` เพื่อตรวจว่าพบชื่อสกิล แล้วเรียกใช้ด้วย `$workflow-zozeei`

```text
$workflow-zozeei ช่วยวางแผนเพิ่มระบบอนุมัติเอกสาร ยังไม่ต้องแก้ไฟล์หรือสร้าง issue
```

หากยังไม่พบสกิล ให้เริ่ม Codex ใหม่
อ้างอิง: [Codex skill invocation](https://learn.chatgpt.com/docs/build-skills)

### Claude Code

เปิด `/skills` เพื่อตรวจว่าพบชื่อสกิล แล้วเรียกใช้ด้วย `/workflow-zozeei`

หากเพิ่งสร้างโฟลเดอร์ skills ครั้งแรกและยังไม่พบ ให้เริ่ม Claude Code ใหม่
อ้างอิง: [Claude Code skill invocation](https://code.claude.com/docs/en/skills)

### Gemini CLI

หลังคัดลอกไฟล์ พิมพ์คำสั่งใน Gemini CLI:

```text
/skills reload
/skills list
```

จากนั้นส่งคำขอภาษาธรรมชาติ:

```text
ใช้สกิล workflow-zozeei ช่วยวางแผนเพิ่มระบบอนุมัติเอกสาร ยังไม่ต้องแก้ไฟล์หรือสร้าง issue
```

Gemini CLI จะเลือกเปิดใช้สกิลผ่านกลไกของเครื่องมือและอาจขออนุญาตเปิดใช้
ไม่ได้สร้าง slash command ชื่อ `/workflow-zozeei` จากไฟล์นี้
อ้างอิง: [Gemini CLI skill activation](https://geminicli.com/docs/cli/using-agent-skills/)

## ตัวอย่างคำขออื่น

```text
ใช้ workflow-zozeei วางแผนแก้บั๊ก ผู้ใช้แก้ไขโปรไฟล์ของคนอื่นได้
ตรวจหาสาเหตุก่อน ยังไม่ต้องแก้ไฟล์
```

```text
ใช้ workflow-zozeei ทำงานต่อจากแผนที่ตกลงกัน
อนุมัติให้แก้โค้ดและรันการตรวจที่ปลอดภัยตามกฎโปรเจกต์
ยังไม่อนุมัติให้สร้าง issue, เปลี่ยนฐานข้อมูลจริง หรือ deploy
```

```text
ใช้ workflow-zozeei ตรวจข้อเสนอจากรีวิวนี้ก่อนแก้ตาม
เทียบกับ requirement และโค้ดจริง แล้วสรุปว่าข้อไหนควรแก้พร้อมเหตุผล
```

## ใช้ต่างเครื่องผ่าน GitHub

GitHub repository นี้เป็นต้นฉบับกลางสำหรับติดตั้งบนเครื่องอื่น
บนเครื่องใหม่ ให้ใช้ขั้นตอนในหัวข้อ “ติดตั้งจาก GitHub” ตามเครื่องมือที่ต้องการ
การติดตั้งระดับผู้ใช้ครอบคลุมทุกโปรเจกต์ในเครื่องนั้น

การแก้ไฟล์บน GitHub ไม่อัปเดตสำเนาที่ติดตั้งไว้โดยอัตโนมัติ
ให้สั่งอัปเดตเองด้วย `/plugin update workflow-zozeei` (Claude Code)
หรือ `npx skills@latest update workflow-zozeei` (เครื่องมืออื่น)

## อัปเดตหรือเลิกใช้

- ต้นฉบับมีชุดเดียวคือ `skills/workflow-zozeei/SKILL.md` บน GitHub
- Claude Code plugin: `/plugin update workflow-zozeei`
- ตัวติดตั้ง skills.sh: `npx skills@latest update workflow-zozeei`
- สำหรับการติดตั้งด้วยการคัดลอก ให้แทนที่ `SKILL.md`
  ภายในโฟลเดอร์สกิลเดิมบนแต่ละเครื่อง แล้ว reload หรือเริ่มเครื่องมือใหม่
- หากเคยแก้สำเนาที่ติดตั้งไว้ ให้เก็บการแก้นั้นก่อนแทนที่
- เมื่อต้องการเลิกใช้ ย้ายโฟลเดอร์ `workflow-zozeei`
  ออกจากตำแหน่งค้นหาสกิลของเครื่องมือนั้น แล้ว reload หรือเริ่มใหม่

## ใช้กับเว็บแชต

### Claude บนเว็บ

รองรับการอัปโหลด custom skill ZIP ผ่าน Customize → Skills
ZIP ต้องมีโฟลเดอร์ `workflow-zozeei/` ครอบ `SKILL.md`
สร้างจาก `skills/workflow-zozeei/` ใน repository นี้
จากนั้นเปิดใช้สกิลและเรียกชื่อในคำขอ
description ของชุดนี้จำกัดไว้ไม่เกิน 200 ตัวอักษรสำหรับรูปแบบดังกล่าว
ความพร้อมของเมนูขึ้นอยู่กับบัญชีและการตั้งค่าขององค์กร
อ้างอิง: [Claude custom skills](https://claude.com/docs/skills/how-to)

### Gemini บนเว็บหรือแอป

ใช้ Gem เป็นทางเลือก: สร้าง Gem ชื่อ `workflow-zozeei`
แล้วนำเนื้อหา Markdown หลัง YAML frontmatter ของ `SKILL.md`
ไปวางในช่อง Instructions อาจแนบไฟล์เป็น Knowledge เพิ่มได้
วิธีนี้เป็นการใช้คำแนะนำผ่าน Gem ไม่ใช่การติดตั้งสกิลของ Gemini CLI
อ้างอิง: [สร้างและใช้ Gems](https://support.google.com/gemini/answer/15146780)

การอ่านคำแนะนำไม่ได้เพิ่มสิทธิ์อ่าน repository รัน test
หรือเรียกสกิลอื่นให้แอปแชตโดยอัตโนมัติ
ผลลัพธ์ขึ้นอยู่กับเครื่องมือและข้อมูลที่มีในแอปนั้น

## ลองตรวจพฤติกรรมหลังติดตั้ง

| คำขอหรือสถานการณ์ | พฤติกรรมที่คาดหวัง |
|---|---|
| แก้คำผิดเล็กน้อย | ใช้ขั้นตอนสั้น ไม่บังคับสัมภาษณ์หรือแตก ticket |
| ขอแผนเท่านั้น | ส่งแผน โดยไม่แก้ไฟล์หรือสร้าง issue |
| ไม่ได้ติดตั้งสกิลย่อย | ทำตามขั้นตอนพื้นฐานและแจ้งตามจริง |
| รายงานบั๊กโดยยังไม่ทราบสาเหตุ | ตรวจหลักฐานก่อนเลือกวิธีแก้ |
| test เริ่มไม่ได้เพราะ environment | รายงานว่ายังยืนยันพฤติกรรมไม่ได้ |
| รีวิวเสนอให้ลบ authorization ที่จำเป็น | ตรวจเทียบ requirement และคงการตรวจสิทธิ์ไว้ |

`SKILL.md` ใช้ frontmatter มาตรฐานเฉพาะ `name` และ `description`
เพื่อให้ย้ายข้ามเครื่องมือได้ง่าย แต่พฤติกรรมของแต่ละ AI อาจต่างกัน
อ้างอิง: [Agent Skills specification](https://agentskills.io/specification)
