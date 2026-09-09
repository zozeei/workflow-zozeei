# ValidatesSafeInput — 14 กลุ่มกฎ

อ่านหัวข้อที่เกี่ยวกับ input/sink ใน scope ใช้ contract ของโปรเจกต์กำหนดชนิด ความยาว และค่าที่อนุญาต ตรวจ server-side boundary และ controls ที่ sink; validation อย่างเดียวไม่แทน authorization, parameterization หรือ output encoding

อ้างอิงหลัก: [OWASP Input Validation](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html) — ตรวจเอกสาร 2026-09-09

## 1. `emailRules`

- ใช้ parser/validator ของ stack ที่รองรับรูปแบบอีเมลตาม product contract พร้อมขนาดจำกัด; regex แบบย่อไม่ใช่ตัวแทน RFC ทั้งหมด
- ปฏิเสธ CR/LF เมื่อส่งเข้า mail header และใช้ mail API ที่ปลอดภัย ยืนยันความเป็นเจ้าของอีเมลเมื่อ workflow ต้องพึ่งข้อมูลนี้
- Normalize domain เป็น lowercase; กำหนดนโยบาย local-part และเก็บค่าต้นฉบับ ไม่ lowercase ทั้งอีเมลโดยอ้างว่า RFC บังคับ เพราะ local-part อาจ case-sensitive ตาม [RFC 5321 §2.3.11](https://www.rfc-editor.org/rfc/rfc5321.html#section-2.3.11)

## 2. `passwordRules`

- เมื่อประเมินตาม [NIST SP 800-63B-4 §3.1.1.2](https://pages.nist.gov/800-63-4/sp800-63b.html#passwordver): single-factor ขั้นต่ำ **15 ตัวอักษร**; password ที่ใช้เป็นส่วนหนึ่งของ MFA ขั้นต่ำ **8 ตัวอักษร** ตรวจทุก login/recovery path ว่าเข้าเงื่อนไข MFA จริง
- SHOULD รองรับความยาวสูงสุดอย่างน้อย 64 ตัวอักษร รวม printing ASCII, space และ Unicode; นับ Unicode code points และตรวจ policy normalization ที่สม่ำเสมอ ไม่มี silent truncation หรือกฎบังคับผสมอักขระ
- เปรียบเทียบรหัสผ่านทั้งค่ากับ blocklist รหัสผ่านที่พบบ่อย/เคยรั่ว ใช้ rate limiting และจำกัดขนาด request ก่อน hash ไม่ตั้งเพดานจนขัดกับความยาวที่รับรอง
- แยกนโยบายตั้งรหัสผ่านออกจากการจัดเก็บ: ใช้ salted adaptive password hash และตรวจ cost/memory parameters จาก library/version จริง แนะนำ Argon2id; systems ที่มีข้อกำหนดเฉพาะอาจเลือก algorithm ที่ได้รับอนุญาตตามข้อกำหนดนั้น
- Legacy bcrypt: implementation ส่วนใหญ่รับสูงสุด **72 ไบต์** ซึ่งต่างจากจำนวนตัวอักษร โดยเฉพาะ Unicode ตรวจพฤติกรรม reject/truncate จริงและแผนย้ายไป hash ที่รองรับ contract ไม่เสนอ direct SHA-256 แทน password hash หรือ pre-hash แบบคิดเอง

เกณฑ์จัดเก็บและ parameters: [OWASP Password Storage](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html) การผ่าน length/hash check ไม่เท่ากับผ่าน NIST ทุกข้อ

## 3. `safeTextRules`

- ข้อความบรรทัดเดียว: ตรวจ length/type และ reject newline/control characters ที่ contract ไม่รองรับ โดยรักษาชื่อและอักขระภาษาที่ผลิตภัณฑ์รองรับ
- เลือก normalization ตาม semantics; NFC/NFKC ไม่ได้พิสูจน์ว่าปลอด homograph/spoofing และ NFKC อาจเปลี่ยนความหมาย ห้ามลบอักขระจนข้อมูลผู้ใช้เปลี่ยนโดยไม่มีนโยบาย

## 4. `safeLongTextRules`

- รองรับ multiline/whitespace ตามงาน กำหนดขีดจำกัด request bytes และ decoded length ให้ชัด ตรวจ parser/regex ต่อ input ยาวและ nesting ที่ทำให้ CPU/memory หมด
- จัดการ control characters ตามปลายทาง ใช้ structured logging/output encoding; ไม่ตัด Unicode ที่จำเป็นต่อภาษาโดยเหมารวมว่า invisible ทุกตัวอันตราย

## 5. `sanitizeText`

- HTML text/attribute/URL/JS เป็นคนละ context: ใช้ encoder หรือ safe rendering API ที่ตรง sink ตรวจ auto-escaping ของ framework ก่อนเพิ่ม encoding เพื่อหลีกเลี่ยง double encoding
- SQL values ใช้ parameterized queries; dynamic identifiers ใช้ mapping/allowlist ไม่ใช้ string replace เป็นตัวป้องกัน SQLi
- OS commands ใช้ API ที่ไม่ผ่าน shell และแยก arguments ตรวจ option/argument injection ด้วย การแยก arguments ไม่ได้ทำให้ทุกคำสั่งปลอดภัยอัตโนมัติ

## 6. `usernameSlugRules`

- เลือก allowlist, length, case/uniqueness และ reserved names ตาม routing/product contract รองรับ Unicode เมื่อผลิตภัณฑ์กำหนด ไม่บังคับชื่อคนทุกภาษาให้เหลือ ASCII
- ถ้าใช้ใน path ให้ตรวจหัวข้อ 10; ถ้าใช้เป็น identifier ให้ตรวจสิทธิ์แยกจากรูปแบบ

## 7. `numericIdRules` & `uuidRules`

- Numeric ID: parse ชนิดที่ถูกต้อง ตรวจช่วงตามฐานข้อมูลและ contract; ใน JavaScript ระวัง safe-integer overflow ของ ID ยาว ใช้ string/BigInt ตาม stack
- UUID: ใช้ parser รองรับ version ที่ contract อนุญาต รวม v7 เมื่อต้องใช้ อ้าง [RFC 9562](https://www.rfc-editor.org/rfc/rfc9562.html) ไม่ใช้ regex v1–v5 เป็นกฎกลาง
- ID รูปแบบถูกต้องยังต้องผ่าน object-level authorization และใช้ query API ที่ปลอดภัย

## 8. `phoneRules`

- Parse/normalize ตามประเทศและรูปแบบที่ product รองรับ เช่น E.164 เมื่อ contract ต้องการ แยก extension ออกจากหมายเลขหลัก
- การลบ space/dash หรือ regex ผ่านไม่ยืนยันว่าหมายเลขใช้งานได้หรือเป็นของผู้ใช้ ตรวจ ownership เมื่อจำเป็นต่อ OTP/recovery

## 9. `urlRedirectRules` & SSRF

- แยก browser redirect ออกจาก server-side fetch ใช้ URL parser ของ runtime และ allowlist scheme/host/port ตามงาน ไม่ตัดสินด้วย prefix/suffix string อย่างเดียว
- Browser redirect: resolve relative URL กับ trusted base แล้วตรวจ origin ปลายทางจริง รวม `//host`, backslash, encoding และ userinfo ตาม parser
- Server fetch: ใช้ destination allowlist เมื่อปลายทางจำกัด ถ้าต้องรับ public URLs ให้ตรวจ **ทุก A/AAAA address** ที่อาจเชื่อมต่อด้วยตัวจำแนก IP ที่ครอบคลุม private, loopback, link-local, reserved/non-public, IPv4-mapped IPv6 และ metadata endpoints; รายการ CIDR สั้น ๆ ไม่ใช่รายการครบ
- ปิด automatic redirects หรือ validate **ทุก redirect hop** ใหม่ก่อน request ถัดไป จำกัดจำนวน hop และไม่ส่ง credentials ข้าม origin
- ป้องกัน DNS rebinding/TOCTOU โดยผูก **IP ที่เชื่อมต่อจริง** กับผลที่ตรวจแล้ว รักษา hostname/TLS verification ให้ถูกต้อง; การ resolve ซ้ำแต่ client resolve เองอีกรอบยังไม่พอ ตรวจ proxy/resolver path และ egress controls
- กำหนด timeout และขนาด response ตามทรัพยากรที่ยอมรับได้ แยก code flaw ที่พบจากผลเข้าถึง internal service ที่ยังไม่ได้พิสูจน์

แนวทางและกรณี bypass: [OWASP SSRF Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html)

## 10. `filenamePathRules`

- ถ้ารับชื่อไฟล์เดี่ยว ให้ reject separators/null และชื่อสงวนตาม OS; สุ่ม storage name ที่ server สร้าง และใช้ extension ที่อนุญาต
- ถ้ารับ path ให้ decode/normalize ตาม stack แล้วตรวจ resolved path อยู่ภายใน allowed root จริงด้วย path-aware containment ตรวจ symlink/reparse point และ race เมื่อ threat model เกี่ยวข้อง ไม่อาศัยการบล็อก `..` หรือ string prefix เพียงอย่างเดียว

## 11. `fileUploadRules`

- ตรวจชนิดที่ product อนุญาตจาก extension, content และ parser; MIME จาก client ไม่น่าเชื่อถือ และ magic bytes อย่างเดียวไม่เพียงพอสำหรับ polyglot/active content
- จำกัด request/file/decompressed size เก็บด้วยชื่อ server สร้างนอก executable web root หรือ storage ที่ควบคุมสิทธิ์อ่าน/เขียนและการเสิร์ฟ
- SVG/HTML/archive ต้องมี policy เฉพาะ เช่น sanitize, sandbox/separate origin, attachment หรือ reject ตามงาน ตรวจ overwrite, archive traversal และ parser limits ตามประเภทไฟล์

อ้างอิง: [OWASP File Upload](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html)

## 12. `jsonSchemaRules`

- ตรวจ payload ทั้ง type, bounds, nesting และ field-level permission ใช้ validator ของ stack ที่มีอยู่ ไม่บังคับติดตั้ง library ใหม่
- Mass assignment: ใช้ allowlist ของ fields ที่ผู้เรียกแก้ได้ และส่ง **parsed/validated result** เข้า write ไม่ส่ง raw body หลัง validate; unknown fields อาจ strip หรือ reject ตาม contract การมี schema ที่ยอมรับ `role` ไม่ได้ให้สิทธิ์ผู้ใช้แก้ role
- ใช้ API ตาม version จริง ตัวอย่าง [Zod ปัจจุบัน](https://zod.dev/api): `z.object()` strip unknown keys โดยปริยาย; `z.strictObject()` reject unknown keys อย่าใช้ชื่อ method นี้กับ library/version อื่นโดยไม่ตรวจ

## 13. `htmlContentRules`

- Rich HTML ต้องใช้ maintained sanitizer ที่เหมาะกับ runtime เช่น DOMPurify หรือ sanitize-html พร้อม allowlist tags/attributes/URL schemes ตามฟีเจอร์
- ตรวจ event handlers, active elements, URL-bearing attributes และการแก้ DOM หลัง sanitize; plain text ใช้ safe text rendering ไม่ต้อง sanitize เป็น HTML

## 14. `sensitiveDataMaskingRules`

- Trace secrets/PII ไป logs, console, telemetry, crash reports, error responses และ exports เก็บเท่าที่จำเป็นตามสิทธิ์/retention policy และ redaction ก่อนออกจากระบบ
- Payment/PAN: แยก masking สำหรับ display ออกจาก storage/truncation ตรวจ PCI DSS edition และข้อกำหนดที่ระบบใช้จริง ไม่ถือเลข 6 หน้า/4 หลังเป็นเกณฑ์ครอบจักรวาล
- ปิดบังค่าลับในรายงาน audit และ remediation diff ด้วย ไม่คัดลอก secret มาเป็นหลักฐานเต็มค่า
