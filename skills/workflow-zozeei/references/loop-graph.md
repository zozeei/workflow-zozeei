# Loop + Graph

อ่านเมื่อแผนมีหลายกิ่ง, งานอิสระหลายงาน หรือ dependency ที่อาจเปลี่ยนระหว่างทำ งานเดี่ยวและแผนเส้นตรงใช้ workflow ปกติ

## Execution graph

ใช้ข้อมูล task จาก Phase 3 และเพิ่มเฉพาะ:

- `id`: ชื่อสั้นที่อ้างถึงได้
- `depends_on`: `id` ที่ต้องผ่านก่อน
- `state`: `pending | ready | running | passed | failed | blocked`

`pending` รอ dependency, `ready` มี dependency ที่จำเป็นผ่านครบ, `running` กำลังทำ, `passed` มีหลักฐานตาม acceptance criteria, `failed` verification ไม่ผ่านและต้องวนแก้, `blocked` ไปต่อไม่ได้เพราะ dependency, decision, permission หรือ external condition

ก่อนเริ่มให้ตรวจว่า `id` ไม่ซ้ำ, dependency อ้างถึง node ที่มีจริง และไม่มี cycle ถ้ามี cycle ให้ส่งกลับ Phase 3 เพื่อแยก decision หรือจัด dependency ใหม่

## Execute ready nodes

1. เลือกเฉพาะ `ready` nodes งานที่เป็นอิสระและ ownership ไม่ทับกันทำพร้อมกันได้เมื่อ runtime รองรับและคุ้มค่าตาม [model-routing.md](model-routing.md); หากไม่รองรับให้ agent ปัจจุบันทำตามลำดับ
2. ทำ loop ต่อ node: **Build** เฉพาะ scope → **Check** ด้วย verification/acceptance criteria → ถ้าไม่ผ่านให้ยืนยันสาเหตุและ **Fix** → rerun เฉพาะ check ที่ได้รับผลกระทบ
3. เปลี่ยนเป็น `passed` เมื่อมีหลักฐานรองรับเท่านั้น แล้วคำนวณ `ready` nodes ใหม่ งานปลายทางที่รอหลายกิ่งเริ่มได้เมื่อ dependency ที่จำเป็นผ่านครบ
4. เมื่อ node `failed` หรือ `blocked` ให้คง downstream เป็น `blocked` ระหว่างที่เงื่อนไขยังไม่คลี่คลาย รายงาน blocker ตามกฎใน SKILL.md

ใช้กฎ verification ผิดคาดใน SKILL.md กับทุก node รวมการเปลี่ยนสมมติฐานหรือเพิ่ม instrumentation เมื่อไม่มีหลักฐานใหม่

## Update the graph

เมื่อพบ dependency ใหม่ให้เพิ่ม edge และคำนวณ state ใหม่ก่อนทำต่อ หาก upstream เปลี่ยน ให้พัก downstream ที่กำลัง `running` แล้ว trace ผลกระทบจาก inputs, contract, files และ verification: เปลี่ยนเฉพาะ downstream ที่งานหรือหลักฐานเดิมอาจใช้ไม่ได้กลับเป็น `pending` แล้วตรวจซ้ำ ส่วน node ที่พิสูจน์ได้ว่าไม่รับผลกระทบคง state เดิม

จบเมื่อ required nodes ทุกตัว `passed` หรือ `blocked` พร้อมหลักฐาน ข้อจำกัด และสิ่งที่ต้องใช้เพื่อปลด blocker
