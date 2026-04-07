# ═══════════════════════════════════════════════════════════════
# قائمة اختبار شاملة — Data Sync Pipeline
# ═══════════════════════════════════════════════════════════════
# 
# التعليمات:
#   1. نفّذ كل اختبار بالترتيب
#   2. بعد كل اختبار، تحقق من:
#      - اللوق (Log file)
#      - قاعدة البيانات (_SYS_SYNC_STATE + الجدول المستهدف)
#      - النتيجة المتوقعة
#   3. سجّل النتيجة: ✅ نجح / ❌ فشل + السبب
#
# الأدوات المطلوبة:
#   - DB Browser for SQLite (لفحص القاعدة)
#   - Google Sheet (لتعديل البيانات)
#   - Excel + الإضافة
# ═══════════════════════════════════════════════════════════════


# ─────────────────────────────────────────────────────────────
# المجموعة 1: الإقلاع الأساسي (Boot)
# ─────────────────────────────────────────────────────────────

## TEST 1.1: أول تشغيل (Clean Install)
الخطوات:
  1. احذف ملف MBI_Local.db بالكامل
  2. افتح Excel
المتوقع:
  - يُنشأ MBI_Local.db تلقائياً
  - تُنشأ جداول _SYS_* الخمسة
  - يُنشأ جدول _SYS_SYNC_STATE
  - اللوق يظهر "Core Orchestration: Completed successfully"
  - الـ Ribbon يعمل (الأزرار فعالة بعد اكتمال التحميل)
النتيجة: [ ]


## TEST 1.2: التشغيل العادي (Warm Start)
الخطوات:
  1. أغلق Excel
  2. افتح Excel مرة أخرى
المتوقع:
  - الإقلاع أسرع (لا يُنشئ جداول جديدة)
  - اللوق لا يحتوي أخطاء
  - الـ Ribbon يعمل
النتيجة: [ ]


## TEST 1.3: التشغيل بدون إنترنت
الخطوات:
  1. افصل الإنترنت
  2. افتح Excel
  3. اضغط Sync Data
المتوقع:
  - يحاول تحميل من SQLite (LoadFromSqliteAsync)
  - إذا كانت بيانات سابقة: "Loaded from local storage"
  - إذا لا بيانات: "No network and no local data"
النتيجة: [ ]


# ─────────────────────────────────────────────────────────────
# المجموعة 2: المزامنة الأساسية (Basic Sync)
# ─────────────────────────────────────────────────────────────

## TEST 2.1: أول مزامنة (First Sync)
الإعداد في Google Sheet:
  - SYS_DEFINITIONS: جدول واحد T_GARB_BOQ, IS_ACTIVE=TRUE
  - SYS_DATA_SOURCES: مصدر واحد, IS_ACTIVE=TRUE, SOURCE_REGION=GLOBAL
  - SYS_SCHEMA_RULES: 6 أعمدة
  - SYS_DATA_MAP: 4 mappings
الخطوات:
  1. اضغط Sync Data
المتوقع:
  - TIER-1 tables تُملأ في SQLite
  - T_GARB_BOQ يُنشأ ويُملأ بالبيانات
  - _SYS_SYNC_STATE يحتوي:
    * _METADATA_SYNC / * / SYNCED
    * T_GARB_BOQ / * / SCHEMA_CREATED أو SCHEMA_REBUILT
    * T_GARB_BOQ / SRC_xxx / SYNCED + DecisionHash + RowCount > 0
    * _LICENSE_TIER / * / LICENSE_CHECK
    * _USER_REGION / * / REGION_CHECK
  - اللوق: "[OK] T_GARB_BOQ/SRC_xxx → N rows"
النتيجة: [ ]


## TEST 2.2: مزامنة متكررة بدون تغيير
الخطوات:
  1. اضغط Sync Data مرة ثانية فوراً
المتوقع:
  - Metadata: "Metadata is fresh. Loading from SQLite" (لا يجلب من الشبكة)
  - البيانات: "[SKIP:Unchanged]" (DecisionHash لم يتغير)
  - لا يتم تحميل أي بيانات جديدة
النتيجة: [ ]


## TEST 2.3: مزامنة بـ Force Refresh
الخطوات:
  1. عدّل btnSyncData_Click لتمرير forceMetaRefresh: true
  2. اضغط Sync Data
المتوقع:
  - يجلب الـ metadata من Google Sheets (ليس SQLite)
  - لكن البيانات لا تتغير (VERSION_TAG نفسه) → SKIP:Unchanged
النتيجة: [ ]


# ─────────────────────────────────────────────────────────────
# المجموعة 3: تغيير البيانات على السيرفر
# ─────────────────────────────────────────────────────────────

## TEST 3.1: تغيير VERSION_TAG
الخطوات:
  1. في Google Sheet → SYS_DATA_SOURCES → غيّر VERSION_TAG من "2" إلى "3"
  2. اضغط Sync Data (مع forceMetaRefresh: true)
المتوقع:
  - DecisionHash يختلف عن المحفوظ
  - يُعاد تحميل البيانات
  - _SYS_SYNC_STATE يتحدث بـ VERSION_TAG=3
النتيجة: [ ]


## TEST 3.2: إضافة عمود جديد في SYS_SCHEMA_RULES
الخطوات:
  1. أضف صفاً جديداً: ENTITY_KEY=T_GARB_BOQ, ATTRIBUTE_KEY=NOTES, DATA_TYPE=Text
  2. اضغط Sync Data (force)
المتوقع:
  - SchemaHash يختلف → "Schema drift detected"
  - DROP T_GARB_BOQ + CREATE مع العمود الجديد
  - يُعاد تحميل البيانات
  - _SYS_SYNC_STATE يتحدث بـ SCHEMA_REBUILT
النتيجة: [ ]


## TEST 3.3: حذف عمود من SYS_SCHEMA_RULES
الخطوات:
  1. احذف صف NOTES الذي أضفته
  2. اضغط Sync Data (force)
المتوقع:
  - SchemaHash يختلف → DROP + CREATE بدون NOTES
  - يُعاد تحميل البيانات
النتيجة: [ ]


## TEST 3.4: إضافة مصدر بيانات جديد
الخطوات:
  1. أضف صفاً جديداً في SYS_DATA_SOURCES: SOURCE_KEY=SRC_NEW, PROFILE_KEY=GARB
  2. أضف mappings مطابقة في SYS_DATA_MAP
  3. اضغط Sync Data (force)
المتوقع:
  - المصدر الجديد يظهر ويُزامن
  - _SYS_SYNC_STATE يحتوي صفاً جديداً لـ SRC_NEW
النتيجة: [ ]


## TEST 3.5: حذف مصدر بيانات
الخطوات:
  1. احذف صف SRC_NEW من SYS_DATA_SOURCES
  2. اضغط Sync Data (force)
المتوقع:
  - CleanOrphanStates يحذف حالة SRC_NEW
  - البيانات القديمة من SRC_NEW تبقى (لا تُحذف — الجدول يحتوي بيانات أخرى)
النتيجة: [ ]


## TEST 3.6: حذف جدول كامل من SYS_DEFINITIONS
الخطوات:
  1. احذف صف T_GARB_BOQ من SYS_DEFINITIONS
  2. اضغط Sync Data (force)
المتوقع:
  - CleanOrphanTables يحذف T_GARB_BOQ من SQLite
  - ClearSyncState يحذف كل حالات T_GARB_BOQ
  - اللوق: "Orphan table removed: T_GARB_BOQ"
النتيجة: [ ]


## TEST 3.7: إعادة إضافة جدول محذوف
الخطوات:
  1. أعد صف T_GARB_BOQ في SYS_DEFINITIONS
  2. اضغط Sync Data (force)
المتوقع:
  - يُنشأ الجدول من جديد
  - يُزامن البيانات
النتيجة: [ ]


## TEST 3.8: تعطيل جدول (IS_ACTIVE=FALSE)
الخطوات:
  1. في SYS_DEFINITIONS → IS_ACTIVE=FALSE لـ T_GARB_BOQ
  2. اضغط Sync Data (force)
المتوقع:
  - الجدول لا يظهر في Registry (IS_ACTIVE filter)
  - CleanOrphanTables يحذف T_GARB_BOQ
النتيجة: [ ]


## TEST 3.9: تعطيل مصدر (IS_ACTIVE=FALSE)
الخطوات:
  1. في SYS_DATA_SOURCES → IS_ACTIVE=FALSE
  2. اضغط Sync Data (force)
المتوقع:
  - المصدر لا يظهر في context.Sources
  - "has no active sources" → TRUNCATE
النتيجة: [ ]


# ─────────────────────────────────────────────────────────────
# المجموعة 4: الرخصة (License)
# ─────────────────────────────────────────────────────────────

## TEST 4.1: جدول Free مع رخصة منتهية
الإعداد:
  - SYS_DEFINITIONS: LICENSE_TIER=Free
  - رخصة المستخدم: منتهية (Expired)
الخطوات:
  1. اضغط Sync Data
المتوقع:
  - الجدول يُزامن بشكل طبيعي (Free لا يحتاج رخصة فعالة)
النتيجة: [ ]


## TEST 4.2: جدول Premium مع رخصة Premium
الإعداد:
  - SYS_DEFINITIONS: LICENSE_TIER=Premium
  - رخصة المستخدم: Premium فعالة
الخطوات:
  1. اضغط Sync Data
المتوقع:
  - الجدول يُزامن بشكل طبيعي
النتيجة: [ ]


## TEST 4.3: جدول Premium مع رخصة Free (Teaser)
الإعداد:
  - SYS_DEFINITIONS: LICENSE_TIER=Premium
  - رخصة المستخدم: Free أو منتهية
الخطوات:
  1. اضغط Sync Data
المتوقع:
  - [TEASER] T_GARB_BOQ
  - صف واحد بقيم "Upgrade Required"
  - _SYS_SYNC_STATE: SYNC_STATUS=TEASER
النتيجة: [ ]


## TEST 4.4: ترقية الرخصة (Free → Premium)
الإعداد:
  - ابدأ بـ TEST 4.3 (teaser موجود)
  - فعّل رخصة Premium
الخطوات:
  1. اضغط Sync Data
المتوقع:
  - EnforceLicense يكتشف Upgrade
  - يمسح SyncState للجداول الجديدة المتاحة
  - يُعاد Sync كامل (بيانات حقيقية بدلاً من teaser)
النتيجة: [ ]


## TEST 4.5: تخفيض الرخصة (Premium → Free)
الإعداد:
  - ابدأ بـ TEST 4.2 (بيانات premium موجودة)
  - خفّض الرخصة إلى Free
الخطوات:
  1. اضغط Sync Data
المتوقع:
  - EnforceLicense يكتشف Downgrade
  - DROP للجداول Premium
  - ClearSyncState
  - اللوق: "License purge: T_GARB_BOQ (needs Premium)"
  - في الـ Ingestion: Teaser يُنشأ
النتيجة: [ ]


## TEST 4.6: انتهاء الرخصة (Expiry)
الإعداد:
  - رخصة Premium مع ExpiryDate = أمس
الخطوات:
  1. افتح Excel
المتوقع:
  - SecurityContext يعود لـ Free
  - نفس سلوك TEST 4.5 (Downgrade)
النتيجة: [ ]


# ─────────────────────────────────────────────────────────────
# المجموعة 5: المنطقة الجغرافية (Region)
# ─────────────────────────────────────────────────────────────

## TEST 5.1: مصدر GLOBAL + أي مستخدم
الإعداد:
  - SYS_DATA_SOURCES: SOURCE_REGION=GLOBAL
  - رخصة المستخدم: أي region
الخطوات:
  1. اضغط Sync Data
المتوقع:
  - المصدر يُزامن بنجاح (GLOBAL متاح للجميع)
النتيجة: [ ]


## TEST 5.2: مصدر EG + مستخدم EG
الإعداد:
  - SOURCE_REGION=EG
  - UserProfile.RegionCode=EG
الخطوات:
  1. اضغط Sync Data
المتوقع:
  - المصدر يُزامن (تطابق المنطقة)
النتيجة: [ ]


## TEST 5.3: مصدر EG + مستخدم SA
الإعداد:
  - SOURCE_REGION=EG
  - UserProfile.RegionCode=SA
الخطوات:
  1. اضغط Sync Data
المتوقع:
  - [SKIP:RegionMismatch]
  - لا يتم تحميل البيانات
النتيجة: [ ]


## TEST 5.4: مصدر EG + مستخدم GLOBAL
الإعداد:
  - SOURCE_REGION=EG
  - UserProfile.RegionCode=GLOBAL
الخطوات:
  1. اضغط Sync Data
المتوقع:
  - المصدر يُزامن (GLOBAL يصل لكل شيء)
النتيجة: [ ]


## TEST 5.5: تغيير region المستخدم (EG → SA)
الإعداد:
  - كانت بيانات EG مُزامنة
  - غيّر UserProfile.RegionCode إلى SA
الخطوات:
  1. اضغط Sync Data
المتوقع:
  - EnforceRegion يكتشف التغيير
  - TRUNCATE للجداول التي تحتوي بيانات EG فقط
  - ClearSyncState
  - يُعاد Sync (لكن مصادر EG تكون SKIP:RegionMismatch)
النتيجة: [ ]


## TEST 5.6: تغيير region المصدر (EG → GLOBAL)
الإعداد:
  - SYS_DATA_SOURCES: غيّر SOURCE_REGION من EG إلى GLOBAL
الخطوات:
  1. اضغط Sync Data (force)
المتوقع:
  - DecisionHash يختلف (REGION تغير)
  - يُعاد تحميل البيانات
النتيجة: [ ]


## TEST 5.7: تغيير region المصدر (GLOBAL → SA)
الإعداد:
  - غيّر SOURCE_REGION من GLOBAL إلى SA
  - المستخدم region=EG
الخطوات:
  1. اضغط Sync Data (force)
المتوقع:
  - DecisionHash يختلف
  - لكن المصدر الآن SA والمستخدم EG → SKIP:RegionMismatch
النتيجة: [ ]


# ─────────────────────────────────────────────────────────────
# المجموعة 6: Schema Drift
# ─────────────────────────────────────────────────────────────

## TEST 6.1: تغيير DATA_TYPE لعمود
الخطوات:
  1. في SYS_SCHEMA_RULES → غيّر UNIT_PRICE من Decimal إلى Text
  2. اضغط Sync Data (force)
المتوقع:
  - SchemaHash يختلف
  - DROP + CREATE مع النوع الجديد
  - يُعاد تحميل البيانات
النتيجة: [ ]


## TEST 6.2: تغيير IS_VISIBLE لعمود
الخطوات:
  1. غيّر IS_VISIBLE=FALSE لعمود
  2. اضغط Sync Data (force)
المتوقع:
  - SchemaHash يختلف (IS_VISIBLE مشمول في الـ hash)
  - DROP + CREATE
النتيجة: [ ]


## TEST 6.3: تغيير LICENSE_TIER لعمود
الخطوات:
  1. غيّر LICENSE_TIER لعمود من Free إلى Premium
  2. اضغط Sync Data (force)
المتوقع:
  - SchemaHash يختلف
  - DROP + CREATE
النتيجة: [ ]


## TEST 6.4: تغيير IS_PK
الخطوات:
  1. اجعل عمود ITEM_KEY هو IS_PK=TRUE
  2. اضغط Sync Data (force)
المتوقع:
  - SchemaHash يختلف
  - DROP + CREATE مع PRIMARY KEY constraint
النتيجة: [ ]


# ─────────────────────────────────────────────────────────────
# المجموعة 7: Presets و JSON Bags
# ─────────────────────────────────────────────────────────────

## TEST 7.1: Preset بسيط
الخطوات:
  1. أضف في SYS_CONFIG_PRESETS:
     PRESET_KEY=COL_PRICE, PROP_KEY=Width, PROP_VALUE=15, VAL_TYPE=INT, TARGET_BAG=UX_CONFIG
     PRESET_KEY=COL_PRICE, PROP_KEY=Format, PROP_VALUE=#,##0.00, VAL_TYPE=TEXT, TARGET_BAG=UX_CONFIG
  2. في SYS_SCHEMA_RULES → عمود UNIT_PRICE → UX_CONFIG = {"$preset":"COL_PRICE"}
  3. اضغط Sync Data (force)
المتوقع:
  - PresetResolver يحل {"$preset":"COL_PRICE"} → ColumnUxConfig{Width=15, Format="#,##0.00"}
  - تحقق من الـ debug: هل الأعمدة تظهر بالتنسيق الصحيح
النتيجة: [ ]


## TEST 7.2: Preset مع Override
الخطوات:
  1. UX_CONFIG = {"$preset":"COL_PRICE","Width":200}
  2. اضغط Sync Data (force)
المتوقع:
  - Width=200 (override) و Format="#,##0.00" (من preset)
النتيجة: [ ]


## TEST 7.3: JSON مباشر بدون Preset
الخطوات:
  1. UX_CONFIG = {"Width":120,"Format":"@"}
  2. اضغط Sync Data (force)
المتوقع:
  - Width=120, Format="@"
النتيجة: [ ]


# ─────────────────────────────────────────────────────────────
# المجموعة 8: حالات حدية (Edge Cases)
# ─────────────────────────────────────────────────────────────

## TEST 8.1: مسافات زائدة في Headers (Google Sheet)
الخطوات:
  1. أضف مسافة بعد اسم عمود في Google Sheet (مثل "UX_CONFIG ")
  2. اضغط Sync Data
المتوقع:
  - TsvParser يعمل Trim على الـ headers
  - لا أخطاء parsing
النتيجة: [ ]


## TEST 8.2: قيمة Enum غير معروفة
الخطوات:
  1. في SYS_SCHEMA_RULES → SEMANTIC_ROLE=UNKNOWN_VALUE
  2. اضغط Sync Data
المتوقع:
  - لا crash (ConvertValue يعود بـ default enum value)
  - SEMANTIC_ROLE = NONE (القيمة الافتراضية)
النتيجة: [ ]


## TEST 8.3: صف فارغ في TSV
الخطوات:
  1. أضف صفاً فارغاً في Google Sheet بين البيانات
  2. اضغط Sync Data
المتوقع:
  - الصف الفارغ يُتجاهل (skipped)
  - باقي البيانات تُزامن بشكل طبيعي
النتيجة: [ ]


## TEST 8.4: مصدر بـ URI خاطئ
الخطوات:
  1. عدّل SOURCE_URI لمصدر إلى رابط غير صالح
  2. اضغط Sync Data
المتوقع:
  - [FAIL] مع رسالة خطأ واضحة
  - لا يؤثر على بقية الجداول
النتيجة: [ ]


## TEST 8.5: جدول بدون أي مصادر في SYS_DATA_SOURCES
الإعداد:
  - جدول موجود في SYS_DEFINITIONS
  - لا يوجد أي صف له في SYS_DATA_SOURCES
الخطوات:
  1. اضغط Sync Data
المتوقع:
  - "has no active sources"
  - إذا الجدول موجود في SQLite → TRUNCATE
النتيجة: [ ]


## TEST 8.6: مزامنة أثناء مزامنة (Double Click)
الخطوات:
  1. اضغط Sync Data
  2. فوراً اضغط مرة أخرى
المتوقع:
  - المرة الثانية: "Sync already running. Ignored."
  - لا crash أو تعارض
النتيجة: [ ]


## TEST 8.7: جداول متعددة مع مصادر مختلطة
الإعداد:
  - 3 جداول: T_A (Free/GLOBAL), T_B (Premium/EG), T_C (Free/SA)
  - المستخدم: Premium, Region=EG
الخطوات:
  1. اضغط Sync Data
المتوقع:
  - T_A: يُزامن (Free + GLOBAL)
  - T_B: يُزامن (Premium + EG match)
  - T_C: SKIP:RegionMismatch (SA ≠ EG)
النتيجة: [ ]


# ─────────────────────────────────────────────────────────────
# المجموعة 9: الأداء (Performance)
# ─────────────────────────────────────────────────────────────

## TEST 9.1: جدول كبير (1000+ صف)
الخطوات:
  1. جهّز Google Sheet بـ 1000+ صف
  2. اضغط Sync Data
المتوقع:
  - BulkInsert يكتمل في < 2 ثانية
  - لا memory spikes كبيرة
النتيجة: [ ]


## TEST 9.2: metadata fresh check
الخطوات:
  1. اضغط Sync Data
  2. انتظر 5 ثوانٍ
  3. اضغط Sync Data مرة أخرى
المتوقع:
  - المرة الثانية أسرع بكثير (لا network fetch)
  - "Metadata is fresh. Loading from SQLite."
النتيجة: [ ]


# ─────────────────────────────────────────────────────────────
# المجموعة 10: التنظيف والإغلاق (Cleanup)
# ─────────────────────────────────────────────────────────────

## TEST 10.1: إغلاق Excel أثناء Sync
الخطوات:
  1. ابدأ Sync على جدول كبير
  2. أغلق Excel أثناء التحميل
المتوقع:
  - CancellationToken يُفعّل
  - لا crash
  - البيانات الجزئية لا تُحفظ (Transaction rollback)
النتيجة: [ ]


## TEST 10.2: COM Cleanup
الخطوات:
  1. اضغط Sync Data
  2. أغلق Excel
  3. تحقق من Task Manager
المتوقع:
  - لا يوجد EXCEL.EXE عالق
  - "COM Cleanup Service: Auto-released N objects"
النتيجة: [ ]


## TEST 10.3: إعادة التشغيل بعد Crash
الخطوات:
  1. اقتل EXCEL.EXE من Task Manager (محاكاة crash)
  2. افتح Excel مرة أخرى
المتوقع:
  - النظام يعمل بشكل طبيعي
  - WAL mode يتعامل مع الـ incomplete transaction
  - لا بيانات تالفة
النتيجة: [ ]
