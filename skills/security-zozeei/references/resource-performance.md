# Query performance และ resource consumption

อ่านเมื่อผู้ใช้ขอตรวจ N+1/performance หรือพบ query ใน loop, lazy loading, unbounded list/batch หรืองานใช้ทรัพยากรสูงใน scope ใช้ขั้นตอนและขอบเขต read-only จาก SKILL หลัก

## N+1 Query Problem

- Trace route/job → service/resolver/serializer → ORM/DB/network call รวม lazy loading ที่ถูกกระตุ้นตอน serialize ค้น query ต่อรายการและ nested relation; loop ที่อ่านข้อมูลใน memory หรือ batch/prefetch ที่ทำไว้แล้วไม่ใช่หลักฐาน N+1
- ยืนยันจำนวน round trips ที่โตตามจำนวนรายการด้วย code flow หรือ query logs/profiler ที่ปิดบังข้อมูลแล้ว ระบุจำนวนรายการและขอบเขต request/job; ถ้าขึ้นกับ runtime/cache ที่ยังไม่เห็นให้ใช้ `needs-verification` และเสนอวัดด้วย fixtures อย่างน้อยสองขนาด เช่น 1 กับ 20 รายการ ภายใต้เงื่อนไขเดียวกัน แยก cold/warm cache
- เลือก eager loading, projection, batch query หรือ request-scoped batching ตาม stack จริง ตรวจ tenant/authorization ใน batch และ cache keys ด้วย ระวัง join ที่เพิ่ม rows แบบ Cartesian explosion; split queries จำนวนคงที่ไม่ใช่ N+1 โดยอัตโนมัติ และ parallelize query ต่อรายการยังคงจำนวน query เดิมพร้อมเพิ่มแรงกดดัน connection pool
- Verification ควรตรวจจำนวน query เมื่อข้อมูลโต พร้อมความถูกต้อง/สิทธิ์ของผลลัพธ์และขอบเขต memory ไม่ใช้ latency ครั้งเดียวเป็นหลักฐานว่าหายแล้ว แนวทาง ORM: [Microsoft EF Core — Efficient Querying](https://learn.microsoft.com/en-us/ef/core/performance/efficient-querying#beware-of-lazy-loading) — ตรวจเอกสาร 2026-09-10; ตรวจ API ของ framework/version ที่พบก่อนเสนอ code

## ขอบเขตการใช้ทรัพยากร

- List/search/export: ตรวจ maximum page size ที่ server บังคับ, จำนวนรายการ/response bytes และพฤติกรรมเมื่อไม่ส่ง limit; export ขนาดใหญ่ต้องมีขอบเขตหรือวิธีประมวลผลที่เหมาะกับงาน เช่น bounded streaming/job ตรวจ query plan/index เฉพาะ query ที่มีหลักฐานว่ามีปัญหา ไม่เสนอ index ทุกคอลัมน์
- Batch/GraphQL/fan-out: ตรวจจำนวน operations, depth/complexity, จำนวน children และ concurrency ตามฟีเจอร์ พร้อม limits ต่อผู้ใช้/tenant/operation; rate limit ต่อ request อย่างเดียวอาจยังปล่อยงานมหาศาลใน request เดียว
- ตรวจ DB/external-call timeout, cancellation, retry budget และ queue/pool bounds รวม CPU/memory/decompression และต้นทุน SMS/email/บริการภายนอกเมื่ออยู่ใน scope; ใช้ข้อจำกัดตาม workload ไม่กำหนดเลขเดียวให้ทุกระบบ สำหรับ upload อ่าน [fileUploadRules](input-validation.md#11-fileuploadrules)
- อ้างอิงความเสี่ยงด้านทรัพยากร: [OWASP API4:2023 — Unrestricted Resource Consumption](https://github.com/OWASP/API-Security/blob/master/editions/2023/en/0xa4-unrestricted-resource-consumption.md) — ตรวจเอกสาร 2026-09-10

## เกณฑ์รายงาน

- รายงาน N+1 ที่ยืนยันได้เป็น `performance` พร้อมหลักฐาน query growth และผลกระทบที่ทราบ ใช้สถานะตาม SKILL หลักและ `NOT RATED` สำหรับ security severity เมื่อยังไม่มีผลกระทบด้าน security; ข้อเสนอ optimization ที่ยังไม่พบปัญหาเป็น `hardening` / INFORMATIONAL
- จัดเป็น `security/availability` เมื่อมีหลักฐานเส้นทางที่ผู้เรียกขยายงานได้และ controls ไม่เพียงพอจนเกิดความเสี่ยงต่อ availability/ค่าใช้จ่าย ระบุสิทธิ์ ขอบเขตและข้อจำกัด; ชื่อ N+1 หรือ endpoint ช้าอย่างเดียวไม่ยืนยัน DoS
- แยกค่าที่วัดจริงจากการคาดการณ์ ไม่สร้าง latency/query-count/token ตัวเลขสมมติเป็นผลตรวจ และไม่ทำ load test บนระบบจริงจากสิทธิ์ audit เสนอการวัดใน isolated environment พร้อมขอบเขตเมื่อยังไม่มีข้อมูล
