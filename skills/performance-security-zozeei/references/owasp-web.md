# OWASP Top 10 Web — 2025

ใช้กับ Web/Backend/API ใน scope เท่านั้น แต่ละข้อเป็นจุดค้นหลักฐาน ตรวจ applicability และ controls ที่ framework/deployment มีอยู่ก่อนรายงาน

หมวดอ้างอิง: [OWASP Top 10:2025](https://owasp.org/Top10/2025/) — ตรวจเอกสาร 2026-09-09

## A01:2025 — Broken Access Control

- Trace object/tenant-level permission ของ read/update/delete และ role checks ของ admin routes รวมการบังคับสิทธิ์ใน middleware/service
- เทียบสิทธิ์ user A/B และ tenant A/B ทั้ง list/detail/bulk/export/file routes ที่อยู่ใน scope รวม field-level read/write; trace tenant/owner จาก authenticated context และ membership ที่ server ตรวจ ไม่เชื่อ ID จาก body/header เพียงอย่างเดียว เสนอ fixtures ที่ทั้งปฏิเสธข้ามสิทธิ์และยังยอมรับเจ้าของที่ถูกต้อง
- CORS: ตรวจ origin allowlist/reflection และ credentialed response ที่ browser อ่านได้จริง `Access-Control-Allow-Origin: *` กับ credentials ถูก browser บล็อก ไม่ใช่หลักฐานว่าข้อมูลรั่วเอง และ CORS ไม่แทน authorization/CSRF controls ดู [Fetch Standard](https://fetch.spec.whatwg.org/#http-cors-protocol)
- ถ้ามี user-controlled server fetch ให้อ่าน URL/SSRF ใน [input-validation.md](input-validation.md)

## A02:2025 — Security Misconfiguration

- ตรวจ production debug/error responses, default credentials, public access ไป `.env`/`.git`/backup และ admin services
- ประเมิน CSP, HSTS, content type, framing และ referrer policy ตาม response/architecture ตรวจ reverse proxy ด้วย; การขาด header หนึ่งตัวไม่ยืนยันช่องโหว่เสมอ และ `frame-ancestors` อาจทำหน้าที่แทน X-Frame-Options

## A03:2025 — Software Supply Chain Failures

- ตรวจ resolved dependency versions/lockfiles กับ advisories ที่ตรง version และบริบทใช้จริง แยก vulnerable dependency จาก reachability ที่ยังไม่ทราบ; ถ้า scanner/network ใช้ไม่ได้ ให้ระบุว่า advisory status ยังไม่ตรวจ
- ตรวจแหล่งแพ็กเกจ/typosquatting, CI permissions/secrets และการดึง build scripts/actions จากแหล่งที่ไม่ได้ตรึงหรือยืนยัน integrity ไม่รัน dependency install หรือ auto-fix จาก audit

## A04:2025 — Cryptographic Failures

- Passwords ต้องเป็น adaptive hash; ข้อมูลที่ต้องอ่านคืนใช้ encryption/key management ตาม threat model ไม่ใช้คำว่า encrypt password แทน hash
- ตรวจ certificate/hostname verification และ TLS configuration: TLS 1.3 เป็นค่าแนะนำ, TLS 1.2 ที่ตั้งค่าแข็งแรงอาจใช้เพื่อ compatibility; ไม่รายงาน TLS 1.2 ว่าเป็นช่องโหว่จาก version อย่างเดียว ดู [OWASP TLS](https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Security_Cheat_Sheet.html)
- ตรวจ crypto API/mode/key storage, nonce/IV ตาม algorithm และ CSPRNG สำหรับ tokens อ่าน password/storage ใน [input-validation.md](input-validation.md) เมื่อเกี่ยวข้อง

## A05:2025 — Injection

- Trace SQL/NoSQL query construction: parameterize values, allowlist identifiers/operators ตาม stack ไม่ถือ ORM หรือ validated ID เป็นหลักฐานว่าปลอดภัยทุก query
- XSS: ตรวจ source ไป HTML/attribute/URL/script sinks พร้อม framework auto-escaping และ sanitizer ที่ใช้จริง; ชื่อ `innerHTML` อย่างเดียวไม่พอ ต้องมี untrusted flow หรือเหตุให้ข้อมูลที่เชื่อถือได้กลายเป็น untrusted
- ตรวจ shell/argument injection, eval และ user-controlled template source; อ่าน sink controls ใน [input-validation.md](input-validation.md) เมื่อเกี่ยวข้อง

## A06:2025 — Insecure Design

- ตรวจ abuse/rate limits ที่ login/OTP/recovery/payment และงานใช้ทรัพยากรสูง พร้อม account enumeration ตาม threat model
- ตรวจ replay, transaction ordering, partial failure และ idempotency ในธุรกรรมสำคัญ; timestamp อย่างเดียวไม่รับประกันป้องกัน replay
- ตรวจ check-then-write ของยอดเงิน stock คูปอง และการอนุมัติภายใต้ concurrent requests: invariant ต้องคงอยู่ด้วย atomic conditional write, constraint หรือ transaction/locking ที่เหมาะกับ DB/isolation จริง การครอบ transaction อย่างเดียวไม่พิสูจน์ว่าป้องกัน race
- Idempotency ต้องตรวจ scope ต่อผู้ใช้/tenant/operation, payload เดิมกับ key เดิม และการจอง key/บันทึกผลแบบ atomic รวม retries/webhooks และผลจากบริการภายนอก; การมี header หรือเช็ค key ก่อน insert อย่างเดียวไม่พอ เสนอการทดสอบพร้อมกันใน isolated environment ว่าตัดเงิน/ใช้คูปอง/ลด stock ได้ตามจำนวนที่อนุญาต
- เกณฑ์ธุรกรรม: [OWASP Business Logic Security](https://cheatsheetseries.owasp.org/cheatsheets/Business_Logic_Security_Cheat_Sheet.html) — ตรวจเอกสาร 2026-09-10 เมื่อพบ query/resource amplification ให้อ่าน [resource-performance.md](resource-performance.md)

## A07:2025 — Authentication Failures

- ตรวจ credential verification, MFA/recovery bypass, brute force controls และ session rotation/expiry/revocation
- JWT: trace การ verify signature/MAC ก่อนเชื่อ claims ไม่ถือ decode ว่ายืนยันตัวตน ตรวจ algorithm allowlist ฝั่ง server, trusted key/issuer, audience, expiry และ not-before ตาม token profile พร้อม clock skew ที่จำกัด; ตรวจ middleware/library config จริงก่อนสรุปว่าขาดการตรวจ
- Key selection (`kid`, `jku`, `x5u`/JWKS): จำกัดแหล่ง key ที่เชื่อถือได้และผูกกับ issuer ไม่ดึง URL หรือ path ตาม token โดยไม่มี validation; แยก token type/consumer เพื่อป้องกันนำ token คนละวัตถุประสงค์มาใช้แทนกัน ดู [RFC 8725](https://www.rfc-editor.org/rfc/rfc8725.html)
- Token/session lifecycle: ตรวจ logout, password reset, บัญชีถูกระงับ และการเปลี่ยนสิทธิ์ว่าจำกัดการใช้ token เก่าตาม policy ได้จริง รวมอายุ access token และการเพิกถอน refresh token; ถ้าใช้ refresh rotation ให้ตรวจ reuse detection/การอัปเดตแบบ atomic ตาม flow ไม่กำหนดว่าทุกระบบต้องมี JWT denylist แบบเดียวกัน
- เสนอ fixtures สำหรับ signature ผิด, issuer/audience ผิด, expired/not-yet-valid และ token หลัง revoke พร้อม valid-token control; แนวทาง [OWASP REST Security](https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html#jwt) — ตรวจเอกสาร 2026-09-10
- Session cookies: ประเมิน HttpOnly, Secure, scope และ SameSite ตาม flow; cross-site flow ที่ต้องใช้ `SameSite=None; Secure` ต้องตรวจ CSRF/origin controls แทนการบังคับ Lax/Strict ทุกระบบ
- ตรวจ CSRF ของ cookie-auth state-changing routes และ password policy/blocklist ตาม [input-validation.md](input-validation.md)

## A08:2025 — Software or Data Integrity Failures

- ตรวจ untrusted deserialization, artifact/update verification และ third-party script integrity ตามวิธี deploy
- Webhooks/messages: ตรวจ signature กับ payload ที่ถูกต้อง, key selection และ replay controls; ไม่เชื่อ flag ว่า verified จากผู้ส่ง

## A09:2025 — Security Logging and Alerting Failures

- ตรวจ audit trail ของ auth/permission denial/reset และเส้นทาง alerting ที่ต้องใช้ตอบสนองเหตุการณ์
- ป้องกัน log injection ด้วย structured/escaped fields; redaction ก่อน logs/APM และจำกัดสิทธิ์/retention ของข้อมูลอ่อนไหว

## A10:2025 — Mishandling of Exceptional Conditions

- Trace error/timeout/dependency failure ของ auth และ critical writes ให้ fail securely และไม่ทิ้งข้อมูลกึ่งสำเร็จโดยไม่มี recovery
- ตรวจ resource limits, cleanup/cancellation และ error boundary ที่เหมาะสม ไม่ถือการมี global handler ว่าป้องกัน crash/memory leak ทุกกรณี
- ส่ง error ที่เพียงพอต่อผู้ใช้แต่ไม่เปิด internals/secrets แยกรายละเอียดสำหรับ operational logs ที่ปิดบังข้อมูลแล้ว
