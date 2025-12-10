Validation Reference Table
إليك **المرجع الشامل للأخطاء والتحققات (Validation Reference Table)** للنظام المطور.
تم تصميم هذا الجدول ليساعدك كمطور في تحديد سبب المشكلة فوراً بمجرد قراءة الـ Log.

---

### مفتاح مستويات الخطورة (Severity Legend)
*   🔴 **CRITICAL / FATAL:** توقف النظام، سيتم تجاهل هذا الكيان بالكامل ولن يعمل.
*   🟠 **ERROR:** خطأ منطقي، قد يعمل الكيان جزئياً لكن سيسبب مشاكل (مثل عدم نقل البيانات).
*   🟡 **WARNING:** تنبيه، سيعمل النظام لكن قد تكون النتائج غير متوقعة (مثل الاعتماد على قيم افتراضية).
*   🔵 **INFO:** ملاحظة لتحسين الجودة (Best Practices).

---

### 1. تعريف الجدول (`TableDefinitionEntity`)

| الخطورة | الحقل | التحقق (Logic) | رسالة الخطأ المتوقعة |
| :--- | :--- | :--- | :--- |
| 🔴 | `Def_Key` | هل الحقل فارغ؟ | `CRITICAL: Missing 'Def_Key'. This row will be ignored.` |
| 🟡 | `Display_Name` | الجدول `Active` والاسم فارغ؟ | `WARNING: Table '{DefKey}' is Active but has no 'Display_Name'.` |
| 🟠 | `Cache_Policy` | القيمة غير موجودة في القائمة المسموحة؟ | `ERROR: Invalid Cache_Policy '{Value}' in table '{DefKey}'.` |
| 🟠 | `Access_Level` | القيمة لا تطابق أي `LicenseTier`؟ | `ERROR: Invalid Access_Level '{Value}' in table '{DefKey}'.` |
| 🟡 | `Color_Hex` | الصيغة لا تطابق `^#([A-Fa-f0-9]{6...)$`؟ | `WARNING: Invalid Color_Hex '{Value}' in table '{DefKey}'.` |

---

### 2. تعريف الأعمدة (`SchemaRuleEntity`)

| الخطورة | الحقل | التحقق (Logic) | رسالة الخطأ المتوقعة |
| :--- | :--- | :--- | :--- |
| 🔴 | `Def_Key` | هل الحقل فارغ؟ | `CRITICAL: Missing 'Def_Key' in schema row.` |
| 🔴 | `Col_Key` | هل الحقل فارغ؟ | `CRITICAL: Missing 'Col_Key' in schema row for '{DefKey}'.` |
| 🟡 | `Display_Name` | هل الحقل فارغ؟ | `WARNING: Column '{DefKey}.{ColKey}' has no Display_Name.` |
| 🟡 | `Data_Type` | النوع غير معروف وتم تحويله افتراضياً لـ TEXT؟ | `WARNING: Data_Type '{Value}' for '{Ctx}' is unrecognized. Defaulting to TEXT.` |
| 🟠 | `UI_Width` | القيمة أقل من 0؟ | `ERROR: Invalid UI_Width ({Value}) for '{Ctx}'. Must be >= 0.` |
| 🟡 | `Is_Primary` | مفتاح أساسي ولكن `Is_Required` = False؟ | `WARNING: Column '{Ctx}' is Primary Key but Is_Required is FALSE.` |
| 🟠 | `License_Tier` | القيمة لا تطابق أي `LicenseTier`؟ | `ERROR: Invalid License_Tier '{Value}' for '{Ctx}'.` |
| 🟠 | `Default_Value` | القيمة الافتراضية لا يمكن تحويلها لنوع العمود؟ | `ERROR: Default_Value '{Value}' for '{Ctx}' is not valid for type {Type}.` |

---

### 3. دليل المصادر (`DirectoryItemEntity`)

| الخطورة | الحقل | التحقق (Logic) | رسالة الخطأ المتوقعة |
| :--- | :--- | :--- | :--- |
| 🔴 | `Table_Key` | هل الحقل فارغ؟ | `CRITICAL: Directory item missing 'Table_Key'.` |
| 🔴 | `Def_Key` | هل الحقل فارغ؟ | `CRITICAL: Directory item [{TableKey}] missing 'Def_Key'.` |
| 🟡 | `Display_Name` | هل الحقل فارغ؟ | `WARNING: Directory item [{TableKey}] has no Display_Name.` |
| 🟡 | `Region_Code` | الطول ليس 2 أو 3 حروف؟ | `WARNING: Region_Code '{Value}' ... looks invalid (expected 2–3 letters).` |
| 🟠 | `Source_Url` | الجدول `Active` والرابط فارغ؟ | `ERROR: Active table [{TableKey}] has no 'Source_Url'.` |
| 🟠 | `Source_Url` | الرابط ليس `HTTP/HTTPS` أو صيغة خطأ؟ | `ERROR: Invalid URL format for [{TableKey}]: '{Url}'.` |
| 🟡 | `Source_Url` | الرابط `HTTP` (غير مشفر)؟ | `WARNING: Insecure HTTP URL for [{TableKey}]. Prefer HTTPS.` |
| 🟡 | `Access_Tier` | القيمة غير صالحة (سيتم اعتبارها Free)؟ | `WARNING: Invalid Access_Tier '{Value}' ... Will default to Free.` |
| 🔵 | `Update_Frequency` | القيمة غير معروفة (Daily, Weekly, etc.)؟ | `INFO: Update_Frequency '{Value}' ... is not a known value.` |
| 🟡 | `Version` | الجدول `Active` ولا يوجد إصدار؟ | `WARNING: Active table [{TableKey}] has no Version. Sync logic may fail.` |

---

### 4. خرائط البيانات (`DataMapEntity`)

| الخطورة | الحقل | التحقق (Logic) | رسالة الخطأ المتوقعة |
| :--- | :--- | :--- | :--- |
| 🔴 | `Table_Key` | لا يوجد `TableKey` ولا `MapGroupKey` (يتيم)؟ | `CRITICAL: Orphan mapping rule for '{Target}'. Missing Keys.` |
| 🔴 | `Target_Col_Key` | هل الحقل فارغ؟ | `CRITICAL: Mapping rule in '{Table}' has no Target_Col_Key.` |
| 🟠 | `Logic` | لا يوجد `Raw_Header` ولا `Static_Value`؟ | `ERROR: Mapping for ... invalid. Must provide either Raw_Header or Static.` |
| 🟡 | `Logic` | يوجد الاثنان معاً (`Raw` + `Static`)؟ | `WARNING: ... has both. Static_Value will override file data.` |
| 🟡 | `Raw_Header` | العنوان فارغ (بعد التنظيف)؟ | `WARNING: Raw_Header for ... is empty.` |
| 🟡 | `Raw_Header` | يحتوي على فواصل (`|`, `;`, `TAB`)؟ | `WARNING: Raw_Header '{Value}' ... contains separator characters.` |
| 🟡 | `Transform_Rule` | عملية غير معروفة (ليست TRIM, UPPER...)؟ | `WARNING: Transform operation '{Op}' on ... is not recognized.` |
| 🔵 | `Target_Col_Key` | الاسم يحتوي على مسافات؟ | `INFO: Target_Col_Key '{Value}' ... contains spaces.` |

---

### 5. تكامل النظام (`TableSchema Integrity`)
**هذه التحققات تتم بعد تجميع البيانات مع بعضها البعض.**

| الخطورة | السياق | التحقق (Logic) | رسالة الخطأ المتوقعة |
| :--- | :--- | :--- | :--- |
| 🔴 | `Columns` | تكرار `Col_Key` في نفس الجدول؟ | `FATAL: Duplicate column key '{Key}' in table '{DefKey}'.` |
| 🟠 | `Mappings` | الـ Map يستهدف عموداً غير موجود في الـ Schema؟ | `ERROR: Mapping targets unknown column '{Target}' in table '{DefKey}'.` |
| 🟡 | `Strategy` | استراتيجية `Merge/Upsert` بدون مفتاح أساسي؟ | `WARNING: Table '{DefKey}' uses Merge/Upsert strategy but has no Primary Key.` |
| 🟠 | `RunTime` | (عند الفتح) ملف Excel لا يحتوي عموداً مطلوباً؟ | يتم إرجاع اسم العمود المفقود في قائمة `MissingHeaders`. |

---

### نصيحة للمطور
عند تصحيح الأخطاء (Debugging)، ابدأ دائماً بحل الأخطاء ذات العلامة الحمراء **(CRITICAL/FATAL)** لأنها تمنع تحميل الجدول بالكامل. ثم انتقل للبرتقالية **(ERROR)** لأنها ستسبب فشل في نقل البيانات.
