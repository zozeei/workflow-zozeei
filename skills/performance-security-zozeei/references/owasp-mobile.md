# OWASP Mobile Top 10 — 2024

ใช้เฉพาะ Android/iOS/Flutter/React Native ใน scope ตรวจ OS/API level, build variant, data sensitivity และ threat model ก่อนเลือก controls การขาด hardening ไม่เท่ากับช่องโหว่โดยอัตโนมัติ

หมวดอ้างอิง: [OWASP Mobile Top 10:2024](https://owasp.org/www-project-mobile-top-10/); ใช้ [MASVS](https://mas.owasp.org/MASVS/) เมื่อต้องการข้อกำหนด mobile ที่ละเอียดขึ้น — ตรวจเอกสาร 2026-09-09

## M1:2024 — Improper Credential Usage

- ตรวจ bundle/config/resources ว่ามี secret ที่ให้สิทธิ์จริงหรือไม่ แยก public client identifier/API key ที่ออกแบบให้เผยแพร่จาก server credential และตรวจ restrictions
- ตรวจ token persistence/access/backup: ใช้ OS-protected storage เช่น iOS Keychain หรือ encrypted storage ที่จัดการ key ผ่าน Android Keystore ตาม platform/version ไม่บังคับ library เดียวทุกแอป ดู [Android Security Best Practices](https://developer.android.com/privacy-and-security/security-best-practices)

## M2:2024 — Inadequate Supply Chain Security

- ตรวจ SDK permissions/data collection, dependency resolution/provenance และ advisories ที่ตรง version รวม native/transitive dependencies
- ตรวจ integrity ของ build/release pipeline และ signing material ที่เข้าถึงได้; binary/server ที่ไม่ได้ตรวจให้ระบุเป็น coverage gap

## M3:2024 — Insecure Authentication/Authorization

- Backend ต้องบังคับ authorization ของธุรกรรมสำคัญเอง ไม่เชื่อ `isAdmin` หรือ local login flag ที่ client เปลี่ยนได้
- แยก biometric ที่ปลดล็อก local secret จาก server authentication; เมื่อต้องป้องกันการ bypass local boolean ให้ตรวจการผูก operation กับ protected key/OS access control ตาม platform การเซ็น server challenge ใช้เมื่อ protocol ต้องการ ไม่บังคับ Android API กับ iOS

## M4:2024 — Insufficient Input/Output Validation

- ตรวจ deep/universal links, exported IPC และ WebView navigation/bridge ไปยัง privileged actions พร้อม origin/parameter validation
- เปิด JavaScript/bridges เท่าที่จำเป็นและจำกัด untrusted content ตรวจ local query APIs กับ input จริง; อ่าน [input-validation.md](input-validation.md) เฉพาะ inputs/sinks ที่เกี่ยวข้อง

## M5:2024 — Insecure Communication

- ตรวจ cleartext policy ของ production ตาม OS/domain และ TLS certificate/hostname verification; ไม่ยอมรับ trust-all implementation
- ประเมิน certificate pinning ตาม threat model และแผน rotate/backup pins ไม่รายงานว่าทุกแอปที่ไม่มี pinning มีช่องโหว่ ดู [Android Network Security Configuration](https://developer.android.com/privacy-and-security/security-config)

## M6:2024 — Inadequate Privacy Controls

- ตรวจ permissions/consent, ข้อมูลที่ SDK ส่งออก, retention และ PII ใน logs/crash reports
- หน้าจอที่มีข้อมูลอ่อนไหวให้ประเมิน background snapshots/screen capture ตามความเสี่ยงและ platform ไม่บังคับปิดทุกหน้าจอ

## M7:2024 — Insufficient Binary Protections

- ตรวจ release build, signing/integrity และความเสี่ยงจาก reverse engineering/tampering ตามระดับที่แอปต้องต้านทาน
- Obfuscation, root/jailbreak detection และ anti-hooking เป็น defense in depth ที่อาจ bypass ได้ ไม่แทน server authorization และไม่ใช่ finding รุนแรงจากการไม่มีเพียงอย่างเดียว

## M8:2024 — Security Misconfiguration

- ตรวจ exported components ทีละตัวกับ intent filters/permissions และ caller/input validation; launcher/deep-link entry points ที่ต้อง public ไม่ควรถูกปิดหรือใส่ permission จนใช้งานไม่ได้
- ตรวจ `debuggable` ใน release และ backup/data extraction rules ตาม Android version/target SDK รวม cloud backup กับ device transfer; อย่าถือ `allowBackup=false` ว่าควบคุมทุกกรณี ดู [Android Auto Backup](https://developer.android.com/identity/data/autobackup)

## M9:2024 — Insecure Data Storage

- Trace sensitive data ไป DB/cache/files/shared storage และ backups ตรวจ access control, encryption และ key lifecycle ตาม sensitivity/platform
- ประเมิน keyboard suggestions, clipboard และ screen capture ใน field ที่อ่อนไหว ให้ password manager/paste ทำงานได้ตาม product/security policy ไม่ปิด clipboard ทุกกรณี

## M10:2024 — Insufficient Cryptography

- ตรวจ algorithm/mode/KDF และ random source ที่เหมาะกับหน้าที่ ไม่เขียน crypto เองหรือใช้ deprecated primitives เพื่อคุ้มครองข้อมูลสำคัญ
- ตรวจ key storage/access control และ hardware backing เมื่อ threat model/อุปกรณ์ต้องรองรับ พร้อม fallback ที่ชัดเจน; nonce/IV ต้องตาม algorithm โดยเฉพาะ uniqueness ไม่ใช่กฎ “random IV ทุก operation” ดู [Android Keystore](https://developer.android.com/privacy-and-security/keystore)
