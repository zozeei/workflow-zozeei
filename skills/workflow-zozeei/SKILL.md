---
name: workflow-zozeei
description: >-
  Use when software work needs requirement clarification, approach selection,
  multi-step planning, implementation, or risk-based review; including วางแผน,
  แตกงาน, and ลงมือพัฒนา. Not needed for general questions unrelated to software work.
metadata:
  version: "1.6.1"
---

# Workflow Zozeei

นำทางจาก requirement ไปสู่ผลลัพธ์ที่ตรวจสอบได้ ใช้ decision และสิทธิ์ที่มีอยู่แล้ว เลือกขั้นตอนตามงานและความเสี่ยงจริง

## สิทธิ์และขอบเขต — ใช้กฎนี้ทุก Phase

- ทำตามลำดับความสำคัญของคำสั่งใน runtime และ instruction file ของโปรเจกต์ เช่น `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`
- ผู้ใช้ขอแผนหรือรีวิวอย่างเดียว → อ่าน/ค้นหา และส่งผลในคำตอบ; เขียน artifact เฉพาะที่ผู้ใช้ขอ ห้ามลงมือแก้ระบบจากการขอแผน
- ผู้ใช้สั่งให้สร้าง/แก้/ลงมือ หรืออนุมัติแผนแล้ว → ทำงานและ verification ภายใน scope ต่อได้ ไม่ขออนุมัติซ้ำเมื่อเปลี่ยน Phase
- ถ้าขาด decision ที่มีผลต่อ behavior, scope, permission หรือข้อมูล → ถามเฉพาะสิ่งที่ค้นเองไม่ได้ พักส่วนที่ขึ้นกับคำตอบ และทำส่วนอิสระที่ได้รับอนุญาตต่อ
- ขอสิทธิ์เพิ่มเฉพาะการกระทำที่เกินสิทธิ์เดิม เช่น breaking change ที่ไม่ได้ตกลง, dependency ใหม่ที่เปลี่ยนข้อจำกัด/ต้นทุน, หรือลบข้อมูลสำคัญ; ตรวจ authorization เดิมก่อนถาม
- `git add/commit/push`, สร้างหรือแก้ issue, deploy และ migration/seed/reset/write-delete query ต้องมีสิทธิ์ตรงการกระทำนั้น การอนุมัติแก้โค้ดไม่ให้สิทธิ์เหล่านี้โดยอัตโนมัติ
- รักษางานเดิมของผู้ใช้ ไม่ใช้ `git reset --hard` หรือทำลายข้อมูลนอก scope; ปิดบัง secrets ในคำตอบ logs และ diff และไม่ commit `.env`/secrets

## เลือกจุดเริ่มต้น

| สถานะงาน | เริ่มที่ |
|---|---|
| Requirement คลุมเครือ | Phase 1 |
| Requirement ชัด แต่ยังต้องเลือกวิธี | Phase 2 |
| Decision ชัด ต้องแตกงาน | Phase 3 |
| มีแผนและสิทธิ์ลงมือแล้ว | Phase 4 |
| งานเล็ก ชัด และได้รับคำสั่งให้แก้ | อ่านจุดที่เกี่ยวข้อง → แก้ → ตรวจผล |

ประเมินความเสี่ยงจาก behavior/data ไม่ใช่จำนวนบรรทัด: auth, authorization, data integrity, concurrency, trust boundary และ external effects ต้องตรวจเส้นทางที่ได้รับผลกระทบ

## การใช้โมเดล

งานเล็กหรือต่อเนื่องในบริบทเดียวให้ agent ปัจจุบันทำเอง ใช้ delegation เมื่อ runtime อนุญาต มีงานอิสระที่ขอบเขตชัด และคุ้มกับการส่งต่อ/อ่านบริบทซ้ำ

เมื่อจะ delegate หรือเลือก model/effort ให้อ่าน [model-routing.md](references/model-routing.md) ก่อน ใช้ capability ที่มีจริง; หากไม่มี subagent หรือเปลี่ยนโมเดลไม่ได้ ให้ทำต่อด้วย agent ปัจจุบันและแยกขั้นอ่าน/ตัดสินใจ/ตรวจผล ไม่ต้องตั้งค่า runtime ใหม่

## ลำดับเลือกสกิล — Ponytail และ Matt Pocock เป็นหลัก

เมื่อขั้นตอนนั้นต้องใช้สกิล ให้เลือกตามลำดับนี้ทุก Phase รวมถึงงานที่ delegate:

1. **ใช้สกิลของ Ponytail หรือ Matt Pocock ที่ตรงหน้าที่ก่อน** เมื่อมีใน catalog และเรียกใช้ได้ ไม่ข้ามไปใช้สกิลอื่นหรือขั้นตอนพื้นฐานแทนตัวหลักที่มีอยู่
2. **เมื่อไม่มีตัวหลักสำหรับหน้าที่นั้น** จึงใช้สกิลอื่นที่ตรงงาน ถ้าไม่มีสกิลที่เหมาะสมเลยให้ทำตามขั้นตอนพื้นฐานใน workflow ต่อ โดยไม่ติดตั้งเพิ่มหรือหยุดงานเพียงเพราะขาดสกิล
3. ตรวจที่มาจาก catalog, namespace, manifest หรือหลักฐานที่ตรวจไว้แล้วใน session ถ้าชื่อซ้ำให้เลือกตัวจากแหล่งหลักและใช้ชื่อเรียกจริงของ runtime ไม่เดา namespace; บอกเหตุผลสั้น ๆ เมื่อใช้ fallback และไม่อ้างว่าเป็นสกิลของแหล่งหลักถ้ายืนยันที่มาไม่ได้

| หน้าที่ | สกิลหลัก / แหล่ง |
|---|---|
| เลือกวิธีและเขียนโค้ดให้เรียบง่าย | Ponytail: `ponytail` |
| ตรวจ over-engineering | Ponytail: `ponytail-review` |
| ทำ requirement ให้ชัด | Matt Pocock: `grilling` / `grill-me` |
| วินิจฉัยบั๊ก | Matt Pocock: `diagnosing-bugs` |
| ออกแบบ module / domain | Matt Pocock: `codebase-design` / `domain-modeling` |
| เก็บ spec / แตก tickets | Matt Pocock: `to-spec` / `to-tickets` |
| TDD / ตรวจ requirement และมาตรฐาน | Matt Pocock: `tdd` / `code-review` |
| เขียนเอกสารสำหรับ agent / สกิล | Matt Pocock: `writing-for-agents` |

ตารางเป็นตัวอย่างชื่อที่พบในแต่ละรุ่น งานอื่นหรือชื่อที่ต่างออกไปให้เทียบ description และที่มาจากสองแหล่งหลักก่อน ใช้ร่วมกันเฉพาะบทบาทที่ต่างกัน โหลดเฉพาะสกิลที่ตรงงาน ไม่รันครบทุกตัวหรือเพิ่ม Phase ให้กับงานเล็ก

อ่านคำสั่งจริงก่อนใช้ โดยเฉพาะ side effects/automatic commit และใช้สิทธิ์เดิมตามกฎสิทธิ์และขอบเขต หากตัวหลักต้องใช้สิทธิ์ที่ยังไม่มี ให้พักเฉพาะการกระทำนั้นหรือส่งผลในคำตอบเมื่อทำได้ ไม่เปลี่ยนไปใช้ตัวอื่นเพื่อเลี่ยงสิทธิ์ งานเฉพาะด้านที่ไม่มีตัวหลักครอบคลุม เช่น OWASP upload audit จึงใช้ `security-zozeei` ตาม scope ได้

## Phase 1 — ทำ Requirement ให้ชัด

1. อ่าน repo/schema/docs ที่เกี่ยวข้องก่อนถาม ใช้ข้อมูลจาก session เดิม
2. ระบุ goal, scope, success criteria และข้อจำกัดของข้อมูล/สิทธิ์
3. ถ้า decision สำคัญยังขาด ให้ทำตามกฎสิทธิ์และขอบเขต; เสนอทางเลือกพร้อมผลกระทบเมื่อช่วยให้ตัดสินใจได้

จบเมื่อมีข้อมูลพอเลือกวิธี เมื่อต้องซักโจทย์เพิ่มเติม ให้เลือกสกิลตามลำดับเลือกสกิลด้านบน

**Bug / Regression:** เลือกสกิลวินิจฉัยตามลำดับด้านบน; `diagnose` ใช้เป็นชื่อทางเลือกเมื่อเทียบหน้าที่และที่มาแล้ว หากไม่มีสกิลที่เหมาะสม ให้แยกข้อเท็จจริง/สมมติฐาน/สิ่งที่ยังต้องตรวจ แล้ว reproduce หรือ trace เส้นทางผิดพลาดก่อนเลือกวิธีแก้

## Phase 2 — เลือกวิธีที่เรียบง่าย

เลือกสกิลความเรียบง่ายตามลำดับด้านบน ใช้หลัก YAGNI → Reuse → Platform/stdlib → Existing dependency → Clarity → Minimum code ในการเลือกวิธีและ implementation

ตรวจ approach กับ requirement และหลักฐาน รักษา auth/authorization, validate ที่ trust boundary, ใช้ transaction เมื่อหลาย write ต้องสำเร็จร่วมกัน และพิจารณา partial failure/idempotency ตามความเสี่ยง

เสนอหลายทางเลือกเฉพาะเมื่อมี trade-off ที่ผู้ใช้ต้องตัดสินใจ จบเมื่อเลือกวิธีและ verification ได้

งานหลาย session ที่ผู้ใช้ขอเก็บ decision เป็น artifact ให้เลือกสกิล spec ตามลำดับด้านบน (`to-prd` เป็นชื่อทางเลือกที่ต้องตรวจหน้าที่/ที่มา) ตรวจปลายทางและสิทธิ์ก่อนเรียก; ถ้าไม่ได้ขอ artifact ให้สรุป decision ในคำตอบแล้วทำงานหลักต่อ

## Phase 3 — แตกงานที่ตรวจผลได้

เรียงงานตาม dependency แต่ละงานระบุผลลัพธ์, scope/ไฟล์ที่ตรวจพบ, blocker, acceptance criteria และ verification เพิ่ม role/tier เฉพาะงานที่ delegate จริง

ส่งแผนในคำตอบเป็นค่าเริ่มต้น เมื่อได้รับสิทธิ์ tracker ให้เลือกสกิล tickets ตามลำดับด้านบน (`to-issues` เป็นชื่อทางเลือกที่ต้องตรวจหน้าที่/ที่มา) เมื่อแผนพร้อมให้ตัดสินใจไป Phase 4 ตามกฎสิทธิ์และขอบเขต

## Phase 4 — Implement และ Verify

1. แก้เท่าที่จำเป็นตาม pattern โปรเจกต์ เลือกสกิลความเรียบง่ายและ TDD ตามลำดับด้านบนเมื่อเกี่ยวข้องกับงาน ยืนยัน Red ก่อน implementation ที่ใช้ TDD; งานเอกสาร/แก้เชิงกลใช้ diff หรือ check ที่ตรงงาน
2. รัน verification ที่สัมพันธ์กับการเปลี่ยนแปลง เช่น targeted tests, typecheck, build, lint หรือ runtime check ที่อยู่ในสิทธิ์
3. ตรวจ acceptance criteria และ diff; เพิ่ม review ตามความเสี่ยงด้านล่าง
4. ยืนยัน finding ก่อนแก้ แล้ว rerun เฉพาะ verification ที่ได้รับผลกระทบ ลบเฉพาะ debug/temporary files ที่สร้างสำหรับงานนี้และไม่ต้องส่งมอบ

| ความเสี่ยงที่ต้องตรวจเพิ่ม | Review ตามลำดับเลือกสกิล |
|---|---|
| Requirement / มาตรฐานโปรเจกต์ | `code-review` ของ Matt Pocock ก่อนตัวอื่นที่หน้าที่เดียวกัน |
| ความซับซ้อน / over-engineering | `ponytail-review` ของ Ponytail |
| Security / Auth / trust boundary | สกิลหลักที่ครอบคลุมการตรวจนั้น; ถ้าไม่มีจึงใช้ `security-zozeei` เฉพาะ scope ที่เกี่ยวข้อง |
| Architecture / Data Integrity | `code-review` ของ Matt Pocock เน้น invariants/failure paths; ถ้าไม่มีตัวหลักจึงใช้ `scrutinize` หรือสกิลอื่นที่ตรงงาน |

งานเล็กตรวจเองได้ด้วยสกิลหลักที่เกี่ยวข้อง ไม่ต้องเพิ่ม reviewer/subagent หรือรันทุก review ถ้าไม่มีสกิล review ที่เหมาะสมให้ตรวจ requirement, invariants, failure paths และ tests โดยตรง งานเสี่ยงสูงใช้ผู้ตรวจแยกบริบทเมื่อ runtime รองรับและการส่งต่อคุ้มค่า AI review ไม่แทนผลทดสอบจริง

**เมื่อ verification ผิดคาด:** แยก Red ที่ตั้งใจให้ fail จากปัญหา implementation/test/environment ถ้าลองสมมติฐานเดิม 2–3 ครั้งโดยไม่มีหลักฐานใหม่ ให้เปลี่ยนวิธีตรวจหรือเพิ่ม instrumentation; ถ้าต้องการ decision/สิทธิ์เพิ่มจึงถาม ห้ามลด assertion หรือข้าม check เพื่อให้ผ่าน

จบเมื่อ acceptance criteria มีหลักฐานรองรับ หรือระบุ blocker และสิ่งที่ยังยืนยันไม่ได้อย่างตรงไปตรงมา

## ส่งมอบ

ใช้ภาษาหลักของผู้ใช้ สรุปผล/แนวทาง/สิ่งที่เปลี่ยน → verification ที่รันจริง → ข้อจำกัดหรือ decision ที่ยังต้องการ ระบุ reviewer/model เฉพาะที่ใช้จริง การจบคำตอบหรือไปขั้นถัดไปใช้กฎสิทธิ์และขอบเขตเดียวกัน

### รายงาน Token Usage

- เมื่อ runtime/tool แสดง usage จริง ให้ปิดท้ายด้วย `Token usage` และตัวเลขที่ระบบให้มา แยก `input`, `output`, `cached` และ `total` เท่าที่มี พร้อมระบุขอบเขตว่าเป็น turn, task, session หรือ subagent
- ถ้ามีหลาย agent ให้รายงานแยกแต่ละ agent และยอดรวมเฉพาะเมื่อ runtime ให้ข้อมูลครบหรือคำนวณจากค่าที่แสดงได้โดยตรง ระบุส่วนที่ไม่รวม
- ถ้า runtime ไม่เปิดเผย usage ให้เขียน `Token usage: runtime ไม่เปิดเผยข้อมูล` ห้ามประมาณจากจำนวนคำ ขนาด context หรือ context-window limit
- รายงานค่าใช้จ่ายเฉพาะเมื่อระบบให้ billing/cost จริง ห้ามคูณราคาเองหากไม่ทราบ model, cached-token policy หรือ billing scope ที่แน่นอน
