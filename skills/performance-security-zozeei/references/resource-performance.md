# Query performance และ resource consumption

อ่านเมื่อผู้ใช้ขอตรวจ Query Performance/N+1/index/execution plan หรือพบ query ใน loop, lazy loading, unbounded list/batch หรืองานใช้ทรัพยากรสูง ใช้ขอบเขต read-only จาก SKILL หลักและไม่แก้ไฟล์, schema หรือ migration

## ลำดับหลักฐาน

1. ระบุ database engine/version/driver จาก config, lockfile และ deployment files ก่อนใช้คำสั่งหรือกฎเฉพาะ engine หากพบหลายฐานข้อมูลให้ผูกแต่ละ query กับ connection จริง
2. Trace query จาก route/job/resolver → ORM/query builder/SQL → serializer/consumer พร้อม path:line และรูป query หลัง scopes/filters ที่พิสูจน์ได้
3. ตรวจ schema, migrations, schema snapshot และ index definitions ทั้ง primary, unique, foreign key, partial/filtered, expression/function, included columns, sort order, collation/opclass และ index type หากสถานะ migration ไม่ชัด ให้รายงานความไม่แน่นอน; เมื่อมี read-only catalog จากฐานข้อมูลจริง ให้ถือ effective schema นั้นเหนือการเดาจาก migration history
4. ใช้ query logs/profiler/slow log และ execution plan ที่มีอยู่โดยปิดบังข้อมูล ระบุ parameter shape, row scale และ environment เท่าที่ทราบ แยกสิ่งที่วัดจริงจากสิ่งที่ต้องตรวจเพิ่ม

ข้อมูลไม่พอสำหรับยืนยันความเสี่ยงให้วางใน “ต้องตรวจเพิ่ม” โดยไม่ให้ระดับ High/Medium/Low และไม่เสนอ DDL พร้อมใช้ อย่าคาดเดาว่ามีหรือไม่มี index จากชื่อ ORM relation, foreign key หรือ convention

## จุดตรวจ Query และ N+1

- Query ที่อาจแพง: unbounded result, `SELECT *`/wide rows ที่ไม่จำเป็น, join fan-out, aggregation/sort/distinct/window บนข้อมูลมาก, correlated/nested subquery, repeated lookup และ query ที่อ่าน rows/pages มากเมื่อเทียบกับ rows ที่คืน ต้องมี code-flow, workload หรือ plan รองรับก่อนเรียกว่า “ช้า”
- N+1: trace query ต่อรายการรวม lazy loading ที่ serializer/template/GraphQL resolver กระตุ้น ยืนยัน round trips โตตาม N จาก code flow หรือ query count; loop ใน memory และ prefetch/batch จำนวน query คงที่ไม่ใช่ N+1 การ parallelize query ต่อรายการยังคง query amplification แยกการยืนยัน N+1 ออกจาก access path ของ query ลูก—ชื่อ column ว่า `id` หรือ ORM convention ไม่พิสูจน์ว่ามี primary/unique index ต้องเห็น schema/index/plan ของตารางนั้นก่อนกล่าวถึง
- พิจารณา eager loading, projection, join, batch query หรือ request-scoped batching ตาม semantics จริง ตรวจ authorization/tenant scope และ duplicate rows/memory ด้วย ไม่แก้ N+1 โดยสร้าง Cartesian explosion
- Pagination: ตรวจ deep `OFFSET`, total-count query และ ordering ที่ deterministic; เสนอ keyset/cursor เมื่อ access pattern รองรับ พร้อม unique tie-breaker และ index ที่ตรง filter/order ไม่ถือ pagination ทุกแบบว่าช้า
- Subquery/`EXISTS`/nested query: ตรวจ correlated execution, cardinality และ plan ที่ engine เลือก ไม่ rewrite เป็น JOIN หรือ `EXISTS` โดยอัตโนมัติเมื่อ semantics หรือ optimizer ยังไม่ยืนยัน

## วิเคราะห์ Index

- พิจารณา `WHERE`, `JOIN`, `ORDER BY`, `GROUP BY` ร่วมกันกับ selectivity, cardinality, null distribution, table size, read/write ratio และ query frequency ไม่แนะนำ index ทุก column
- Foreign key ที่เป็นฝั่งอ้างอิงและใช้ JOIN/filter บ่อยเป็น candidate เท่านั้น ตรวจ index จริงและพฤติกรรม engine ก่อน; constraint, primary key หรือ index ที่ฝั่งถูกอ้างอิงไม่พิสูจน์ว่าฝั่งอ้างอิงมี index ที่เหมาะสม
- Composite B-tree: เริ่มประเมิน equality predicates ที่นำหน้า ตามด้วย range/order ที่ต้องใช้จริง แต่เลือก column order จาก query set, selectivity และกฎ engine ไม่ใช้สูตรเดียวทุกกรณี ตรวจ leftmost-prefix/skip-scan/index-combination ของ engine และเสนอ index เดียวที่รองรับหลาย query เมื่อหลักฐานรองรับ
- ก่อนเรียก index ว่าซ้ำ/ไม่จำเป็น เปรียบเทียบ uniqueness, key order, prefix length, direction, included columns, predicate/filter, expression, collation/opclass และ index type รวม workload/usage statistics กับต้นทุน write/storage; left-prefix overlap อย่างเดียวไม่พอสั่งลบ
- `LIKE`/pattern search: leading wildcard, collation และชนิด pattern มีผลต่อ B-tree ต่างกันตาม engine พิจารณา full-text/trigram/specialized index เฉพาะเมื่อ workload รองรับ
- Function/cast บน indexed column อาจทำให้รูป expression ไม่ตรง index ตรวจ implicit casts และ expression/generated-column index ที่ engine รองรับ พร้อมผลต่อ writes; การเห็น function อย่างเดียวไม่พิสูจน์ว่า index ไม่ถูกใช้
- Full Table Scan/Sequential Scan อาจถูกต้องสำหรับตารางเล็ก ผลลัพธ์สัดส่วนสูง หรือ plan ที่ถูกกว่า รายงานเป็นความเสี่ยงเมื่อ plan/workload แสดง scan เกินจำเป็น, estimate ผิดมาก, filter ทิ้ง rows จำนวนมาก หรือเกิดซ้ำบน hot path

## Execution plan

เริ่มจาก estimated plan ที่ไม่รัน query เมื่อ engine รองรับ การใช้ actual plan/`ANALYZE` ต้องได้รับสิทธิ์รัน queryและมี environment/parameter/timeout ที่ปลอดภัย เพราะคำสั่งเหล่านี้อาจ execute งานจริง ใช้กับ representative data ใน staging/local โดยค่าเริ่มต้น และไม่รัน load test หรือ data-changing statement บนระบบจริงจากสิทธิ์ audit

| Engine ที่ยืนยันแล้ว | Estimated plan | Actual/runtime plan |
|---|---|---|
| PostgreSQL | `EXPLAIN (FORMAT JSON) ...` | `EXPLAIN (ANALYZE, BUFFERS, FORMAT JSON) ...` — executes statement; data-changing statements ต้องใช้ transaction/rollback ที่ตรวจแล้ว |
| MySQL | `EXPLAIN FORMAT=JSON ...` | `EXPLAIN ANALYZE ...` ตาม statement/version ที่รองรับ และ executes query |
| SQL Server | Display Estimated Execution Plan / `SET SHOWPLAN_XML ON` | Include Actual Execution Plan หรือ `SET STATISTICS XML ON` พร้อม `STATISTICS IO/TIME` เมื่อได้รับสิทธิ์ |
| SQLite | `EXPLAIN QUERY PLAN ...` | วัดใน fixture/database copy ที่เหมาะสม; output format อาจเปลี่ยนตาม SQLite version |
| Engine อื่น | ใช้ estimated-plan command จากเอกสารของ engine/version นั้น | ใช้ equivalent ที่มี runtime stats หลังตรวจ side effects และสิทธิ์ |

อ่าน plan โดยเทียบ estimated/actual rows, loops, rows scanned/returned, access path, join order/algorithm, sort/temp spill, filter/residual predicate และ I/O/buffers ตาม engine N+1 ต้องดู query count ระดับ request เพิ่ม เพราะ plan ของ query เดียวไม่แสดงจำนวนครั้งที่ application เรียก เปรียบเทียบ before/after ด้วย parameter, data, statistics และ environment เดียวกัน ตัวอย่าง verification ต้องใช้ literal สังเคราะห์หรือ parameter mechanism ที่ engine/tool รองรับจริง ไม่ทิ้ง placeholder `?` ไว้ในคำสั่งที่อ้างว่ารันได้

เอกสารต้นทางที่ตรวจ 2026-09-16: [PostgreSQL EXPLAIN](https://www.postgresql.org/docs/current/sql-explain.html), [MySQL 8.4 EXPLAIN](https://dev.mysql.com/doc/refman/8.4/en/explain.html), [SQL Server execution plans](https://learn.microsoft.com/en-us/sql/relational-databases/performance/display-and-save-execution-plans), [SQLite EXPLAIN QUERY PLAN](https://www.sqlite.org/eqp.html) และแนวทาง N+1 ของ [Microsoft EF Core](https://learn.microsoft.com/en-us/ef/core/performance/efficient-querying#beware-of-lazy-loading) โดยตัวอย่าง EF เป็นเพียงหลักฐานอธิบาย pattern ไม่ผูกคำแนะนำกับ framework

## ระดับและรูปแบบรายงาน

- **High:** มีหลักฐานว่า hot path/large workload เกิด query amplification หรือ plan/runtime แสดง scan/loops/I/O สูงมากและกระทบสำคัญ
- **Medium:** code/schema/plan ยืนยันกลไกที่เสี่ยงและ workload มีนัยสำคัญ แต่ผลกระทบยังจำกัดหรือขาด runtime measurement บางส่วน
- **Low:** ความเสี่ยงจริงขอบเขตเล็ก มี control บางส่วน หรือ optimization ที่หลักฐานรองรับแต่ผลกระทบต่ำ

จัด findings ตามระดับและรวม root cause เดียวกัน ถ้ายังพิสูจน์ไม่ได้ให้แยก “ต้องตรวจเพิ่ม” ไม่ยกระดับด้วยการคาดเดา สำหรับแต่ละ finding ใช้รูปแบบ:

```markdown
### [High / Medium / Low] ชื่อ finding

- ตำแหน่ง: repo-relative-path:line และ symbol
- Query/code: เฉพาะส่วนที่เกี่ยวข้อง ปิดบังข้อมูลอ่อนไหว
- ปัญหา: พฤติกรรมและผลกระทบที่หลักฐานรองรับ
- Index ที่มีอยู่: ชื่อ/columns/order/type/source ที่ตรวจจริง หรือ “ยืนยันไม่ได้” พร้อมเหตุผล
- Index ที่แนะนำ: candidate พร้อม columns/order/type หรือ “ไม่ต้องเพิ่ม” ห้ามเขียน DDL พร้อมใช้เมื่อ schema/engine/effective indexes ยังไม่ยืนยัน
- เหตุผล: เชื่อม query predicates/order/cardinality กับข้อแตกต่างของ engine
- วิธีตรวจด้วย execution plan: command/equivalent, representative parameters และสัญญาณ before/after ที่ต้องเทียบ
```

จบด้วย scope, database/version ที่ยืนยัน, แหล่ง schema/index, checks ที่รันจริง และส่วนที่ยังไม่ตรวจ รายงาน N+1 เป็น `performance`; จัดเป็น `security/availability` ต่อเมื่อผู้เรียกขยายงานได้และ controls ไม่พอจนมีหลักฐานความเสี่ยง availability/ค่าใช้จ่าย

## Resource consumption นอกฐานข้อมูล

- List/search/export: ตรวจ server-enforced maximum page size, response bytes และพฤติกรรมเมื่อไม่มี limit งานใหญ่ใช้ bounded streaming/job ตามบริบท
- Batch/GraphQL/fan-out: ตรวจ operations, depth/complexity, children และ concurrency ต่อผู้ใช้/tenant/operation; request rate limit อย่างเดียวอาจไม่คุมงานใน request เดียว
- ตรวจ external-call timeout, cancellation, retry budget และ queue/pool bounds รวม CPU/memory/decompression และต้นทุนบริการภายนอกเมื่ออยู่ใน scope สำหรับ upload อ่าน [fileUploadRules](input-validation.md#11-fileuploadrules)
- อ้างอิง availability/resource abuse: [OWASP API4:2023](https://github.com/OWASP/API-Security/blob/master/editions/2023/en/0xa4-unrestricted-resource-consumption.md)
