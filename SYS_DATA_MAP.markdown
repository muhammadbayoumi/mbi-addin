
### الجدول الإرشادي المرجعي (SYS_DATA_MAP)

هذا الجدول يوضح كيفية تعبئة بيانات خرائط التحويل (ETL Maps) في Google Sheets.

#### أولاً: الأعمدة الأساسية (Physical Columns)

| اسم العمود | الوصف | أمثلة |
| :--- | :--- | :--- |
| **`PROFILE_KEY`** | **مفتاح الخريطة.** (يربط بـ `MasterDirectory.MapProfile`). | `MAP_EZZ_STEEL` |
| **`TARGET_ATTRIBUTE_KEY`** | **العمود المستهدف.** (في جدولنا المحلي). | `UNIT_PRICE` |
| **`SOURCE_TYPE`** | **نوع البحث.**<br>`Header`: بحث بالاسم.<br>`Index`: بحث برقم العمود.<br>`Constant`: قيمة ثابتة.<br>`Context`: من JSON المصدر. | `Header`, `Constant`, `Context` |
| **`MATCH_MODE`** | **طريقة المطابقة.** (للـ Header فقط). | `Exact`, `Contains`, `Regex` |
| **`SOURCE_EXPRESSION`** | **قيمة البحث.** | `"سعر الطن"`, `"0"`, `"Currency"` |
| **`TRANSFORM_CHAIN`** | **سلسلة التنظيف.** (فصل بـ `|`). | `TRIM|TO_DECIMAL`, `UPPER` |
| **`PROCESS_CONFIG`** | **JSON Bag.** | *انظر الأسفل* |

#### ثانياً: إعدادات المعالجة (PROCESS_CONFIG JSON)

| المفتاح (Key) | الوصف | مثال |
| :--- | :--- | :--- |
| **`"Default"`** | **القيمة الافتراضية.** في حال عدم وجود بيانات. | `"0"`, `"Unknown"` |
| **`"Critical"`** | **إجباري.** إذا فشل التحويل، نرفض الصف بالكامل؟ | `true`, `false` |
| **`"Format"`** | **تنسيق المصدر.** (خاصة للتواريخ). | `"dd-MM-yyyy"` |

---

### أمثلة عملية (Smart Scenarios)

#### 1. سحب "العملة" من خصائص المصدر (وليس من ملف الإكسل)
أنت حددت في `MasterDirectory` أن المصدر عملته `EGP`. تريد إدخال هذه القيمة في عمود `CURRENCY` في الجدول.
*   **Target:** `CURRENCY_CODE`
*   **Source Type:** `Context`
*   **Expression:** `Currency` (اسم المفتاح في JSON المصدر).

#### 2. سحب السعر مع تنظيف الرموز
الملف يحتوي على "1,200 EGP". نريد الرقم فقط.
*   **Target:** `UNIT_PRICE`
*   **Source Type:** `Header`
*   **Match Mode:** `Contains`
*   **Expression:** `Price`
*   **Transform:** `REMOVE_SYMBOLS|TO_DECIMAL`

#### 3. قيمة ثابتة لنوع المورد
*   **Target:** `VENDOR_TYPE`
*   **Source Type:** `Constant`
*   **Expression:** `Local_Distributor`

بهذا، اكتملت البنية التحتية للجداول الأساسية (`Source`, `Definition`, `Schema`, `Map`) بتوحيد قياسي كامل.



إليك الجدول الإرشادي التفصيلي لجدول **`SYS_DATA_MAP`**.
اعتبر هذا الجدول هو **"المترجم" (The Translator)**. وظيفته الوحيدة هي الإجابة على سؤال: *"كيف أحصل على قيمة هذا العمود الداخلي من هذا الملف الخارجي؟"*

---

### أولاً: الأعمدة الأساسية (Physical Columns)
هذه الأعمدة تحدد "من أين" نأتي بالبيانات و "إلى أين" نذهب بها.

| اسم العمود (Column) | الوصف والدور | أمثلة عملية (Values) |
| :--- | :--- | :--- |
| **`PROFILE_KEY`** | **مفتاح خريطة التحويل.**<br>يربط هذه القاعدة بملف المصدر. يجب أن تضع نفس هذا الاسم في عمود `MAP_PROFILE_ID` في جدول المصادر `SYS_DATA_SOURCES`. | `MAP_EZZ_STEEL`<br>`MAP_CHINA_VENDORS` |
| **`TARGET_ATTRIBUTE_KEY`** | **العمود المستهدف (الداخلي).**<br>اسم العمود في نظامك أنت (كما عرفته في `SYS_SCHEMA_RULES`). | `UNIT_PRICE`<br>`ITEM_NAME`<br>`CURRENCY_CODE` |
| **`SOURCE_TYPE`** | **طريقة الصيد (كيف نجد البيانات؟).**<br>1. **`Header`**: ابحث عن اسم العمود في الملف.<br>2. **`Index`**: خذ العمود رقم X (0, 1, 2...).<br>3. **`Constant`**: ضع قيمة ثابتة لكل الصفوف.<br>4. **`Context`**: خذ القيمة من خصائص الملف (JSON) وليس من الصفوف.<br>5. **`Formula`**: معادلة حسابية (مستقبلاً). | `Header`<br>`Index`<br>`Constant`<br>`Context` |
| **`MATCH_MODE`** | **طريقة المطابقة (لـ Header فقط).**<br>1. **`Exact`**: الاسم متطابق تماماً.<br>2. **`Contains`**: يحتوي على الكلمة.<br>3. **`StartsWith`**: يبدأ بالكلمة.<br>4. **`Regex`**: تعبير نمطي معقد. | `Exact`<br>`Contains`<br>`StartsWith` |
| **`SOURCE_EXPRESSION`** | **قيمة البحث.**<br>تعتمد على `SOURCE_TYPE`: <br>- لو Header: اكتب اسم العمود في الإكسل.<br>- لو Index: اكتب رقم العمود (0, 1).<br>- لو Constant: اكتب القيمة الثابتة.<br>- لو Context: اكتب اسم المفتاح في JSON. | `"سعر الوحدة"`<br>`"0"`<br>`"MATERIAL"`<br>`"Currency"` |
| **`TRANSFORM_CHAIN`** | **سلسلة التنظيف.**<br>أوامر لتنظيف البيانات مفصولة بـ `|`.<br>- `TRIM`: إزالة مسافات.<br>- `UPPER`: تحويل لأحرف كبيرة.<br>- `TO_DECIMAL`: تحويل لرقم.<br>- `REMOVE_SYMBOLS`: إزالة عملات ورموز. | `TRIM|TO_DECIMAL`<br>`UPPER` |

---

### ثانياً: إعدادات المعالجة (PROCESS_CONFIG JSON)
حقيبة إعدادات للتحكم في الأخطاء والقيم الافتراضية.

| المفتاح (Key) | الوصف | مثال |
| :--- | :--- | :--- |
| **`"Default"`** | **القيمة البديلة.**<br>ماذا نضع إذا كانت الخلية في الملف فارغة؟ | `"0"`, `"غير محدد"` |
| **`"Format"`** | **تنسيق التاريخ.**<br>إذا كان العمود تاريخ، كيف هو مكتوب في الملف؟ | `"dd/MM/yyyy"`, `"MM-dd-yyyy"` |
| **`"Critical"`** | **درجة الأهمية.**<br>`true`: إذا لم نجد العمود، أوقف العملية وأظهر خطأ.<br>`false`: أكمل وضع القيمة الافتراضية. | `true`, `false` |

---

### سيناريوهات عملية (كيف تملأ الجدول؟)

#### السيناريو 1: البحث عن عمود بالاسم (الطبيعي)
الملف يحتوي على عمود اسمه "سعر الطن".
*   **Target:** `UNIT_PRICE`
*   **Source Type:** `Header`
*   **Match Mode:** `Contains` (تحسباً لوجود مسافات زائدة)
*   **Expression:** `سعر الطن`
*   **Transform:** `TRIM|TO_DECIMAL` (لتنظيف الرقم)

#### السيناريو 2: ملف بدون عناوين (بحث بالترتيب)
الملف عبارة عن CSV قديم، الاسم في العمود الأول والسعر في العمود الثالث.
*   **Target:** `ITEM_NAME` -> **Expression:** `0` (SourceType: `Index`)
*   **Target:** `UNIT_PRICE` -> **Expression:** `2` (SourceType: `Index`)

#### السيناريو 3: قيمة ثابتة (حقن بيانات)
نريد أن نحدد أن كل الصفوف في هذا الملف تابعة لمورد اسمه "وكالة الأهرام"، رغم أن الاسم غير مكتوب داخل الملف.
*   **Target:** `VENDOR_NAME`
*   **Source Type:** `Constant`
*   **Expression:** `وكالة الأهرام`

#### السيناريو 4: استخراج العملة من خصائص الملف (الذكاء)
أنت حددت في جدول المصادر (`MasterDirectory`) أن عملة هذا الملف `SAR` داخل الـ JSON. وتريد إدخالها في قاعدة البيانات لكل صف.
*   **Target:** `CURRENCY_CODE`
*   **Source Type:** `Context`
*   **Expression:** `Currency` (اسم المفتاح في الـ JSON).

---

### ملخص العلاقات (كيف يترابط كل شيء؟)

1.  **`MasterDirectory`**: يقول "أنا الملف رقم 1، واسم بروفايل الخريطة الخاص بي **`MAP_X`**".
2.  **`SYS_DATA_MAP`**: يقول "يا **`MAP_X`**، لكي تملأ العمود **`UNIT_PRICE`**، اذهب للعمود المسمى 'سعر البيع' في الملف".
3.  **`SYS_SCHEMA_RULES`**: يقول "العمود **`UNIT_PRICE`** هو رقم عشري (`DECIMAL`) ودوره (`PRICE`)".
