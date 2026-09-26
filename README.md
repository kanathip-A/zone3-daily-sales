# Zone 3 Daily Sales

แอปกรอกยอดขายรายวัน 9 สาขาโซน 3 เขียนลง Google Sheet
รันได้สองแบบ — **เลือกแบบ ก. ถ้าอยากใช้ Apps Script อย่างเดียว**

```
apps-script/app.html   ตัวแอปไฟล์เดียวจบ (สำหรับวางใน Apps Script)
apps-script/Code.gs    โค้ดฝั่งชีต
index.html             ตัวแอปแบบแยกไฟล์ (สำหรับ GitHub Pages / เปิดในเครื่อง)
support.js             รันไทม์ที่ index.html เรียกใช้
assets/                มาสคอต 7 รูป
_ds/nocturne-…/        สไตล์ชีต
```

---

## แบบ ก. รันบน Apps Script อย่างเดียว (แนะนำ)

ไม่ต้องใช้ GitHub ไม่ต้องโฮสต์ที่ไหน ไม่ต้องใส่ URL ในแอป — ได้ลิงก์เดียวที่เปิดใช้งานได้เลย

1. เปิด `สรุปยอดขายรวมทุกสาขา_Zone3_2026.xlsx` ใน Drive → **File → Save as Google Sheets**
2. ในชีตนั้น **Extensions → Apps Script**
3. ไฟล์ `Code.gs` — ลบของเดิม วางเนื้อหา `apps-script/Code.gs` ทั้งไฟล์
4. กด **+ → HTML** ตั้งชื่อไฟล์ว่า **`app`** (ระบบเติม `.html` ให้เอง)
   ลบเนื้อหาตัวอย่างทิ้ง แล้ววางเนื้อหา `apps-script/app.html` ทั้งไฟล์ → Save
   *(ไฟล์ใหญ่ ~1.4 MB เปิดด้วย VS Code / Notepad แล้ว Ctrl+A → Ctrl+C จะเร็วกว่าเปิดในเบราว์เซอร์)*
5. เลือกฟังก์ชัน **`checkSheets`** กด **Run** → ครั้งแรกจะขอสิทธิ์
   (Review permissions → Advanced → Go to …) แล้วดูผลที่ Execution log
   ทุกบรรทัดขึ้น ✓ = โครงชีตตรง ถ้ามี ✗ บรรทัดนั้นบอกว่าต้องแก้อะไร
6. ถ้าจะแนบรูปใบสรุป: สร้างโฟลเดอร์ใน Drive ใส่ ID ที่ `CONFIG.PHOTO_FOLDER_ID`
   (ID = ส่วนท้าย URL `drive.google.com/drive/folders/<ID>`)
7. **Deploy → New deployment → Web app**
   Execute as: **Me** · Who has access: **Anyone** (หรือ *Anyone with Google account* ถ้าอยากจำกัด) → Deploy
8. เปิด **Web app URL** ที่ได้ → แอปขึ้นมาพร้อมใช้ ไฟเขียว “รันอยู่บนชีตนี้”
   บนมือถือ: เปิด URL นั้น → **Add to Home Screen**

แอปตรวจเองว่าเสิร์ฟจาก Apps Script อยู่ จึงคุยกับชีตผ่าน `google.script.run` ตรง ๆ
ช่องใส่ Web app URL จะถูกซ่อน เพราะไม่ต้องใช้

**แก้โค้ดครั้งต่อไป** ต้อง **Deploy → Manage deployments → ✏️ → Version: New version**
ไม่งั้น URL เดิมยังรันโค้ดเก่า

---

## แบบ ข. โฮสต์แยก (GitHub Pages)

ใช้เมื่ออยากเก็บโค้ดเป็น repo หรือแก้หน้าตาแอปบ่อย

```bash
git init && git add . && git commit -m "Zone 3 Daily Sales"
git branch -M main
git remote add origin https://github.com/<user>/zone3-daily-sales.git
git push -u origin main
```

(มี GitHub CLI: `gh repo create zone3-daily-sales --private --source=. --push`)

เปิดเป็นเว็บ: repo → **Settings → Pages** · Source: **Deploy from a branch** · Branch **main** · Folder **/ (root)** → Save
ได้ URL `https://<user>.github.io/zone3-daily-sales/`

Pages ต้องเป็น repo public — ถ้าไม่อยากเปิดสาธารณะ ใช้ Cloudflare Pages หรือ Netlify (ลากโฟลเดอร์วางได้เลย) หรือใช้แบบ ก. ไปเลย

แบบนี้ต้องทำขั้น Apps Script (ข้อ 1–3, 5–7 ข้างบน) แล้วเอา Web app URL มาใส่ที่
**ตั้งค่า → เชื่อมต่อ** ในแอป → กด **ทดสอบและโหลดเป้า+วันหยุด**

---

## โครงชีต (ตรวจกับไฟล์จริงแล้ว `CONFIG` ตรงทั้งหมด)

- `บันทึกยอดขายรายวัน` — หัวตารางแถว 1 · ข้อมูลแถว 2–366 (ครบทุกวันของปี 2026)
  A วันที่ · B เลขเดือน · C วันในสัปดาห์ · D ประเภทวัน — สามช่องหลังเป็นสูตร
  สาขาละ 3 คอลัมน์ POS / Catering / รวม(สูตร):
  ACP `E,F,G` · ZR `H,I,J` · TU `K,L,M` · FPR `N,O,P` · BU `Q,R,S` · MVR `T,U,V` · CAY `W,X,Y` · SMA `Z,AA,AB` · YCR `AC,AD,AE`
  `AF,AG,AH` = บล็อกสาขาว่างสำรอง (ตรงกับแถว “OMA FPR” ในชีตเป้า) · **`AI` = รวมทั้งวัน**
  → แอปเขียนเฉพาะ POS กับ Catering คอลัมน์รวมเป็นสูตรเดิม คิดต่อเอง
- `ตั้งค่าเป้าหมาย` — หัวตารางแถว 3 · สาขาแถว 4–12 · A สาขา · B–M ม.ค.–ธ.ค. · N รวมทั้งปี
- `วันหยุดนักขัตฤกษ์` — หัวตารางแถว 2 · ข้อมูลแถว 3 ลงไป · A วันที่ · B ชื่อวันหยุด
- `Complaint และ QSC` — โค้ดสร้างให้เองครั้งแรกที่บันทึก · A ปี · B เดือน · C สาขา · D Complaint · E QSC Onsite · F QSC Online
- `สรุปรายเดือน`, `รวม` — แอปไม่แตะ สูตรเดิมทำงานต่อได้เอง

โครงชีตเปลี่ยนในอนาคต แก้ที่ `CONFIG` หัวไฟล์ `Code.gs` ที่เดียว
(`DAILY_TOTAL_COL: 35` = คอลัมน์ AI — เพิ่มสาขาต้องเลื่อนค่านี้)

## Apps Script รับคำสั่งอะไร

หน้าเว็บเรียกผ่าน `apiCall({action, …})` (แบบ ก.) หรือ POST JSON ไปที่ `/exec` (แบบ ข.)

| action | ทำอะไร |
| --- | --- |
| `getConfig` | คืนเป้าทุกสาขาทุกเดือน + รายการวันหยุด |
| `getMonth` | คืนยอดจริงรายวันของเดือนที่ขอ (`year`, `month` 1–12) |
| `saveDaily` | เขียนยอดของวันหนึ่ง (`date`, `rows[{code,pos,cat,bills}]`) |
| `saveTarget` | เขียนเป้า (`code`, `month` 1–12, `value`) |
| `addHoliday` / `removeHoliday` | เพิ่ม/ลบวันหยุด (`date`, `name`) |
| `uploadPhoto` | อัปโหลดรูปขึ้น Drive `<สาขา>/<ปี-เดือน>` (`code`, `date`, `mimeType`, `dataBase64`) |
| `saveMonthly` | เขียน Complaint + QSC ของสาขา/เดือน (`year`, `month`, `code`, `complaint`, `onsite`, `online`) |
| `getMonthly` | คืน Complaint + QSC ทุกสาขาของเดือนที่ขอ |
| `checkSheets` | ตรวจโครงชีต คืนผล ✓/✗ |

## ข้อจำกัดที่ควรรู้

- ก่อนต่อชีต ยอดย้อนหลังเป็นตัวเลขจำลองจากเป้า (หัวเรื่องขึ้น “ตัวเลขตัวอย่าง”) ไม่ใช่ยอดจริง
- `saveDaily` ต้องมีแถววันที่นั้นในชีตแล้ว (ปี 2026 มีครบ) ถ้าไม่พบจะตอบว่าไม่พบวันที่
- Catering ที่กรอก 0 เขียนเป็นช่องว่าง เพื่อให้หน้าตาชีตเหมือนเดิม (ปิดที่ `CONFIG.BLANK_ZERO_CATERING`)
- `app.html` เป็นไฟล์รวมที่คอมไพล์แล้ว **อย่าแก้ตรงนั้น** — แก้ที่ `index.html` แล้วสร้างใหม่
- Who has access แบบ Anyone = ใครมีลิงก์ก็ใช้ได้ ถ้าต้องจำกัด เลือก *Anyone with Google account* หรือใส่ `CONFIG.TOKEN`
- รูปมาสคอตในไฟล์รวมย่อเหลือ 320px เพื่อให้ไฟล์ไม่ใหญ่เกินไปสำหรับ Apps Script
