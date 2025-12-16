إليك الجدول الإرشادي المرجعي (Reference Guide) لجدول تعريف الجداول **`SYS_DEFINITIONS`** (الذي يمثله الكلاس `TableDefinitionEntity`).

هذا الجدول هو "الدستور" الذي سيحدد سلوك كل صفحة (Sheet) في نظامك.

---

### أولاً: الأعمدة الأساسية (Physical Columns)
هذه الأعمدة موجودة بشكل صريح في قاعدة البيانات ولها دور جوهري في البحث والتصنيف.

| اسم العمود (Column) | النوع | الوصف والدور في النظام | القيم المسموحة / أمثلة |
| :--- | :--- | :--- | :--- |
| **`ENTITY_KEY`** | `String` (PK) | **المعرف الفريد.**<br>يستخدم برمجياً لربط الجدول بالمصادر والعلاقات. يجب أن يكون بالإنجليزية وبدون مسافات. | `T_MATERIALS`<br>`T_LABOR_RATES`<br>`LST_VENDORS` |
| **`DISPLAY_NAME`** | `String` | **اسم العرض.**<br>الاسم الذي سيظهر على "لسان" الشيت (Excel Tab) وفي القوائم. | `أسعار الخامات`<br>`معدلات الإنتاج`<br>`قائمة الموردين` |
| **`ENTITY_TYPE`** | `String` | **نوع الكيان (المخ).**<br>يحدد كيف يعالج الكود هذا الجدول: <br>1. **`COST`**: يطبق منطق العملات والأسعار.<br>2. **`PERF`**: يطبق منطق الوحدات (بدون عملة).<br>3. **`REF`**: يعرض البيانات كما هي (Raw Data).<br>4. **`COMP`**: جدول تجميعي (Breakdown). | `COST`<br>`PERF`<br>`REF`<br>`COMP` |
| **`BUSINESS_DOMAIN`** | `String` | **المجال الوظيفي.**<br>يستخدم لتحديد نوع المعادلة الافتراضية (خامة تُضرب / عمالة تُقسم). | `MATERIAL`<br>`LABOR`<br>`EQUIPMENT`<br>`VENDOR`<br>`GENERAL` |
| **`PARENT_KEY`** | `String` | **الوراثة.**<br>إذا كان هذا الجدول يرث إعدادات وأعمدة جدول آخر (لعدم التكرار). | `T_TEMPLATE_COST`<br>`NULL` |
| **`IS_ACTIVE`** | `Bool` | **التفعيل.**<br>لإخفاء الجدول تماماً من النظام دون حذفه. | `TRUE` / `FALSE` |

---

### ثانياً: إعدادات العرض (UX_CONFIG JSON)
هذه المفاتيح توضع داخل عمود `UX_CONFIG` للتحكم في شكل الإكسل.

| المفتاح (JSON Key) | الوصف والدور | أمثلة |
| :--- | :--- | :--- |
| **`"Color"`** | لون التبويب (Sheet Tab Color). | `"#FF0000"` (أحمر), `"#008000"` |
| **`"Direction"`** | اتجاه الشيت. هام جداً للغة العربية. | `"RTL"` (يمين يسار), `"LTR"` |
| **`"Icon"`** | اسم الأيقونة التي تظهر في الـ Ribbon. | `"Hammer"`, `"Money"`, `"List"` |
| **`"Freeze"`** | تجميد الألواح (Freeze Panes). | `"A2"` (تجميد الرأس), `"B2"` (أول عمود) |
| **`"Group"`** | اسم المجموعة في الـ Ribbon (لتجميع الأزرار). | `"Master Data"`, `"Cost Analysis"` |
| **`"Order"`** | رقم ترتيب الزر في القائمة. | `1`, `10`, `99` |
| **`"Zoom"`** | مستوى التكبير الافتراضي عند فتح الشيت. | `100`, `85`, `120` |
| **`"Gridlines"`** | إظهار أو إخفاء خطوط الشبكة. | `true`, `false` |

---

### ثالثاً: إعدادات النظام (DATA_CONFIG JSON)
هذه المفاتيح توضع داخل عمود `DATA_CONFIG` للتحكم في السلوك التقني والباقات.

| المفتاح (JSON Key) | الوصف والدور | أمثلة |
| :--- | :--- | :--- |
| **`"LicenseTier"`** | **الباقة المطلوبة.**<br>لن يظهر هذا الجدول إلا للمشتركين في هذه الباقة أو أعلى. | `"Free"`, `"Standard"`, `"Premium"` |
| **`"StorageStrategy"`** | **استراتيجية التحديث.**<br>- `MergeUpsert`: تحديث الذكي (المفضل).<br>- `ReplaceAll`: مسح القديم وكتابة الجديد.<br>- `Append`: إضافة فقط. | `"MergeUpsert"`, `"ReplaceAll"` |
| **`"ViewMode"`** | **طريقة العرض الافتراضية.**<br>- `Table`: جدول عادي.<br>- `Breakdown`: عرض شجري (+/-). | `"Table"`, `"Breakdown"` |
| **`"PkType"`** | **توليد المفاتيح.**<br>كيف نحدد سطر البيانات الفريد؟ | `"AutoGUID"`, `"Composite"` |

---

### أمثلة عملية لتعبئة الجدول (Scenarios)

#### السيناريو 1: جدول أسعار خامات (ذكي)
نريد جدولاً يعالج العملات ويسمح بالتحليل، ومتاح فقط للباقة المدفوعة.

*   **ENTITY_KEY:** `T_MATERIALS`
*   **ENTITY_TYPE:** `COST`
*   **BUSINESS_DOMAIN:** `MATERIAL`
*   **UX_CONFIG:** `{ "Color": "#FFA500", "Direction": "RTL", "Icon": "Brick", "Group": "Costs" }`
*   **DATA_CONFIG:** `{ "LicenseTier": "Premium", "StorageStrategy": "MergeUpsert", "ViewMode": "Table" }`

#### السيناريو 2: قائمة موردين (بسيطة)
نريد عرض قائمة أسماء وهواتف فقط كما هي، متاحة للجميع.

*   **ENTITY_KEY:** `LST_VENDORS`
*   **ENTITY_TYPE:** `REF` (مرجع خام)
*   **BUSINESS_DOMAIN:** `VENDOR`
*   **UX_CONFIG:** `{ "Color": "#808080", "Direction": "RTL", "Icon": "Contact", "Group": "References" }`
*   **DATA_CONFIG:** `{ "LicenseTier": "Free", "StorageStrategy": "ReplaceAll" }`

#### السيناريو 3: تحليل سعر بند (تفاعلي)
نريد شيت الـ Breakdown الذي يظهر فيه (+/-).

*   **ENTITY_KEY:** `RPT_ITEM_ANALYSIS`
*   **ENTITY_TYPE:** `COMP` (تجميعي)
*   **BUSINESS_DOMAIN:** `GENERAL`
*   **UX_CONFIG:** `{ "Color": "#0000FF", "Direction": "RTL", "Freeze": "A5" }`
*   **DATA_CONFIG:** `{ "ViewMode": "Breakdown", "StorageStrategy": "ReplaceAll" }`
