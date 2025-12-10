تصور ممتاز. الانتقال إلى تبويب **"Source Inspector"** (فحص المصدر) بأسلوب **Mics/Console** سيعطيك شعوراً بأنك داخل غرفة التحكم في السيرفر.

الهدف من هذا التبويب هو الإجابة على أسئلة: "هل الرابط يعمل؟"، "هل السيرفر بطيء؟"، "ما هو شكل البيانات الخام قبل أن يلمسها الكود؟".

إليك التصميم التخيلي لتبويب **[ SOURCE LOG ]** في شاشتك السوداء:

---

### [ HUD - الشريط العلوي للمصدر ]
`(مؤشرات حية للاتصال)`

`📡 STATUS: [ ONLINE (200 OK) ]  |  ⏱️ PING: [ 145ms ]  |  📦 SIZE: [ 2.4 MB ]  |  🔒 SSL: [ VALID ]`

---

### [ THE CONSOLE REPORT ]

```text
================================================================================
:: SOURCE CONNECTIVITY & RAW DATA INSPECTION ::
================================================================================

[ 1. ENDPOINT CONFIGURATION ] (إعدادات الاتصال)
   Provider Name     : [ Internal_ERP ]
   Target Region     : [ EG ] (Egypt)
   Access Tier       : [ PREMIUM ]
   Update Freq       : [ DAILY ]
   Source Type       : [ TSV / REST API ]
   Target URL        : https://api.company.com/export/sales_eg.tsv
   Auth Method       : [ NONE ] (Public Link)

--------------------------------------------------------------------------------

[ 2. NETWORK HANDSHAKE ] (سجل الاتصال الحي)
   > Resolving host 'api.company.com'... [OK]
   > Connecting to port 443... [OK]
   > Sending GET Request...
   > User-Agent: MBI-Addin/2.0
   
   < HTTP/1.1 200 OK
   < Content-Type: text/tab-separated-values; charset=utf-8
   < Content-Length: 2,450,100 bytes
   < Last-Modified: Wed, 21 Oct 2025 07:28:00 GMT
   < Server: Google Frontend
   
   [INFO] Connection established successfully.
   [INFO] Stream opened for reading.

--------------------------------------------------------------------------------

[ 3. RAW DATA PREVIEW (HEAD) ] (أول 5 أسطر من الملف الخام)
   * This is what the parser actually sees before mapping *

   [ROW 0] Txn_Ref_No  <TAB>  Date        <TAB>  Client_Name_Full  <TAB>  Price_EGP
   [ROW 1] TXN-9901    <TAB>  2023-10-01  <TAB>  AL-AHRAM CO.      <TAB>  1500.50
   [ROW 2] TXN-9902    <TAB>  2023-10-01  <TAB>  EL-SEWEDY ELEC    <TAB>  23000.00
   [ROW 3] TXN-9903    <TAB>  2023-10-02  <TAB>  ORASCOM CONST     <TAB>  450.00
   [ROW 4] TXN-9904    <TAB>  2023-10-02  <TAB>  TALAAT MOSTAFA    <TAB>  120.00

--------------------------------------------------------------------------------

[ 4. PARSING DIAGNOSTICS ] (تحليل التنسيق)
   Delimiter Detected : [ TAB (\t) ]
   Encoding           : [ UTF-8 ]
   Header Row Count   : [ 4 ] Columns Found
   
   -- Column Mapping Check --
   1. Txn_Ref_No       -> [ MAPPED ] to Transaction_ID
   2. Date             -> [ MAPPED ] to Transaction_Date
   3. Client_Name_Full -> [ MAPPED ] to Customer_Name
   4. Price_EGP        -> [ MAPPED ] to Unit_Price

================================================================================
END OF SOURCE INSPECTION
```

---

### لماذا هذا التصميم مفيد جداً؟

1.  **اكتشاف البطء (`Ping`):** إذا كان الجدول يفتح ببطء، ستعرف من هنا إذا كان السبب هو السيرفر (Ping عالي) أم حجم الملف (Size ضخم).
2.  **تصحيح الأخطاء (`Raw Data Preview`):**
    *   أحياناً يغير المبرمج اسم العمود في السيرفر من `Date` إلى `TxnDate`.
    *   هنا سترى الاسم القديم والجديد بعينك، وستعرف لماذا فشل الـ Mapping (في القسم 4).
3.  **فحص التنسيق:** سترى بوضوح الفواصل (`<TAB>`)، مما يكشف مشاكل ملفات CSV التي تستخدم الفاصلة بدلاً من الـ Tab.

**هل هذا التصور يغطي احتياجاتك للـ Source Tab؟**
