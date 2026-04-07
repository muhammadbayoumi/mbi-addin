# mbiXaddin — Comprehensive Test Plan & Test Data
> Tests every feature, error handler, and edge case across all 6 system tables.

---

## Google Sheet Setup Required

### New columns to add in existing sheets:

**SYS_DEFINITIONS** — add these columns:
- `IS_VISIBLE` (after IS_ACTIVE)
- `RIBBON_CONFIG` (after SYS_CONFIG)
- `EXPORT_CONFIG` (after RIBBON_CONFIG)
- Remove: `UI_GROUP`, `UI_ORDER`, `ICON_KEY`

**SYS_DATA_SOURCES** — add:
- `RIBBON_CONFIG` (after CONTEXT_PROPS)

**New sheet:** `SYS_EXPORT_VIEWS` (16 columns)

---

## TABLE 1: SYS_DEFINITIONS (Test Data)

### Row 1 — Perfect entity (all features)
```
ENTITY_KEY:       T_TEST_MATERIALS
DISPLAY_NAME:     Test Materials
ENTITY_TYPE:      COST
LICENSE_TIER:     Free
IS_ACTIVE:        true
IS_VISIBLE:       true
STORAGE_STRATEGY: MergeUpsert
PARENT_KEY:       
VIEW_MODE:        Table
BUSINESS_DOMAIN:  MATERIAL
UX_CONFIG:        {"TabColor":"#4472C4","Direction":"LTR","FreezePanes":"B2","ZoomLevel":85,"ShowGridlines":true}
SYS_CONFIG:       {"AllowEdit":false}
RIBBON_CONFIG:    {"Label":"Material Prices","ScreenTip":"Latest pricing","SuperTip":"450+ items with regional pricing.\nUpdated monthly.","Icon":"TableExcel","Group":"Cost Data","Order":10}
EXPORT_CONFIG:    {"LinkedEntities":[{"EntityKey":"T_TEST_UNITS"}],"FooterText":"Source: Test Data 2026","FooterStyle":"Note"}
```

### Row 2 — Preset-based config
```
ENTITY_KEY:       T_TEST_LABOR
DISPLAY_NAME:     Test Labor Rates
ENTITY_TYPE:      COST
LICENSE_TIER:     Standard
IS_ACTIVE:        true
IS_VISIBLE:       true
STORAGE_STRATEGY: ReplaceAll
PARENT_KEY:       
VIEW_MODE:        Table
BUSINESS_DOMAIN:  LABOR
UX_CONFIG:        {"$preset":"SHEET_COST"}
SYS_CONFIG:       {"$preset":"SYS_DEFAULT"}
RIBBON_CONFIG:    {"Label":"Labor Rates","Group":"Cost Data","Order":20}
EXPORT_CONFIG:    
```

### Row 3 — With ExportViews (GARB scenario)
```
ENTITY_KEY:       T_TEST_GARB
DISPLAY_NAME:     Test GARB BOQ
ENTITY_TYPE:      COST_ENG
LICENSE_TIER:     Free
IS_ACTIVE:        true
IS_VISIBLE:       true
STORAGE_STRATEGY: ReplaceAll
PARENT_KEY:       
VIEW_MODE:        Table
BUSINESS_DOMAIN:  GARB
UX_CONFIG:        {"TabColor":"#2E7D32","Direction":"RTL","FreezePanes":"C2","ZoomLevel":90}
SYS_CONFIG:       {"AllowEdit":false}
RIBBON_CONFIG:    {"Label":"بنود الهيئة","ScreenTip":"القائمة الموحدة","Icon":"TableExcel","Group":"BOQ","Order":1}
EXPORT_CONFIG:    {"FooterText":"المصدر: الهيئة العامة للطرق والكباري","FooterStyle":"Source"}
```

### Row 4 — Reference table (linked by others)
```
ENTITY_KEY:       T_TEST_UNITS
DISPLAY_NAME:     Test Units
ENTITY_TYPE:      REF
LICENSE_TIER:     Free
IS_ACTIVE:        true
IS_VISIBLE:       true
STORAGE_STRATEGY: ReplaceAll
PARENT_KEY:       
VIEW_MODE:        Table
BUSINESS_DOMAIN:  SYSTEM
UX_CONFIG:        {"TabColor":"#808080"}
SYS_CONFIG:       
RIBBON_CONFIG:    {"Label":"Units","Group":"Reference","Order":1}
EXPORT_CONFIG:    
```

### Row 5 — IS_VISIBLE=false (syncs but hidden from menu)
```
ENTITY_KEY:       T_TEST_HIDDEN
DISPLAY_NAME:     Hidden Table
ENTITY_TYPE:      REF
LICENSE_TIER:     Free
IS_ACTIVE:        true
IS_VISIBLE:       false
STORAGE_STRATEGY: ReplaceAll
PARENT_KEY:       
VIEW_MODE:        Table
BUSINESS_DOMAIN:  SYSTEM
UX_CONFIG:        
SYS_CONFIG:       
RIBBON_CONFIG:    
EXPORT_CONFIG:    
```

### Row 6 — IS_ACTIVE=false (completely ignored)
```
ENTITY_KEY:       T_TEST_INACTIVE
DISPLAY_NAME:     Inactive Table
ENTITY_TYPE:      COST
LICENSE_TIER:     Free
IS_ACTIVE:        false
IS_VISIBLE:       true
STORAGE_STRATEGY: MergeUpsert
PARENT_KEY:       
VIEW_MODE:        Table
BUSINESS_DOMAIN:  MATERIAL
UX_CONFIG:        
SYS_CONFIG:       
RIBBON_CONFIG:    
EXPORT_CONFIG:    
```

### Row 7 — Inheritance (child)
```
ENTITY_KEY:       T_TEST_CHILD
DISPLAY_NAME:     Child Table
ENTITY_TYPE:      
LICENSE_TIER:     Free
IS_ACTIVE:        true
IS_VISIBLE:       true
STORAGE_STRATEGY: MergeUpsert
PARENT_KEY:       T_TEST_MATERIALS
VIEW_MODE:        Table
BUSINESS_DOMAIN:  
UX_CONFIG:        {"ZoomLevel":100}
SYS_CONFIG:       
RIBBON_CONFIG:    {"Label":"Child Override","Order":99}
EXPORT_CONFIG:    
```

### Row 8 — Premium (license gated)
```
ENTITY_KEY:       T_TEST_PREMIUM
DISPLAY_NAME:     Premium Data
ENTITY_TYPE:      COST
LICENSE_TIER:     Premium
IS_ACTIVE:        true
IS_VISIBLE:       true
STORAGE_STRATEGY: MergeUpsert
PARENT_KEY:       
VIEW_MODE:        Table
BUSINESS_DOMAIN:  FINANCE
UX_CONFIG:        
SYS_CONFIG:       
RIBBON_CONFIG:    {"Label":"Premium Analytics","Group":"Finance","Order":1}
EXPORT_CONFIG:    
```

### Row 9 — Conversion table (semantic roles)
```
ENTITY_KEY:       T_TEST_EXCHANGE
DISPLAY_NAME:     Exchange Rates
ENTITY_TYPE:      CONVERSION
LICENSE_TIER:     Free
IS_ACTIVE:        true
IS_VISIBLE:       true
STORAGE_STRATEGY: ReplaceAll
PARENT_KEY:       
VIEW_MODE:        Table
BUSINESS_DOMAIN:  FINANCE
UX_CONFIG:        {"TabColor":"#FF9800"}
SYS_CONFIG:       
RIBBON_CONFIG:    {"Label":"Exchange Rates","Group":"Finance","Order":2}
EXPORT_CONFIG:    
```

### Row 10 — ERROR: Duplicate ENTITY_KEY (same as Row 1)
```
ENTITY_KEY:       T_TEST_MATERIALS
DISPLAY_NAME:     DUPLICATE - Should be ignored
ENTITY_TYPE:      REF
LICENSE_TIER:     Free
IS_ACTIVE:        true
IS_VISIBLE:       true
STORAGE_STRATEGY: ReplaceAll
...
```

### Row 11 — ERROR: Missing ENTITY_KEY
```
ENTITY_KEY:       
DISPLAY_NAME:     No Key Table
ENTITY_TYPE:      COST
LICENSE_TIER:     Free
IS_ACTIVE:        true
...
```

### Row 12 — ERROR: Bad PARENT_KEY
```
ENTITY_KEY:       T_TEST_ORPHAN
DISPLAY_NAME:     Orphan Child
ENTITY_TYPE:      COST
LICENSE_TIER:     Free
IS_ACTIVE:        true
IS_VISIBLE:       true
PARENT_KEY:       T_NONEXISTENT_PARENT
...
```

### Row 13 — ERROR: Broken JSON
```
ENTITY_KEY:       T_TEST_BADJSON
DISPLAY_NAME:     Bad JSON Entity
ENTITY_TYPE:      REF
LICENSE_TIER:     Free
IS_ACTIVE:        true
IS_VISIBLE:       true
UX_CONFIG:        {BROKEN JSON!!!
RIBBON_CONFIG:    not a json
EXPORT_CONFIG:    {"LinkedEntities":[BROKEN
```

### Row 14 — Marketing footer
```
ENTITY_KEY:       T_TEST_MARKETING
DISPLAY_NAME:     Marketing Demo
ENTITY_TYPE:      COST
LICENSE_TIER:     Free
IS_ACTIVE:        true
IS_VISIBLE:       true
STORAGE_STRATEGY: ReplaceAll
UX_CONFIG:        {"TabColor":"#E91E63"}
EXPORT_CONFIG:    {"FooterText":"🔒 Upgrade to Premium for live pricing updates!","FooterStyle":"Marketing"}
RIBBON_CONFIG:    {"Label":"Demo Prices","Group":"Demo","Order":1}
```

---

## TABLE 2: SYS_SCHEMA_RULES (Test Data)

### T_TEST_MATERIALS columns
```
ENTITY_KEY | ATTRIBUTE_KEY | DISPLAY_HEADER | ORDINAL_POS | DATA_TYPE | IS_PK | IS_MANDATORY | IS_VISIBLE | SEMANTIC_ROLE | UX_CONFIG | LOGIC_CONFIG
T_TEST_MATERIALS | ITEM_CODE    | Item Code    | 1  | TEXT    | true  | true  | true  | NONE       | {"Width":12}  |
T_TEST_MATERIALS | ITEM_NAME    | Item Name    | 2  | TEXT    | false | true  | true  | NONE       | {"Width":25,"HeaderColor":"#4472C4"}  |
T_TEST_MATERIALS | UNIT         | Unit         | 3  | TEXT    | false | false | true  | UNIT       | {"Width":8,"Align":"Center"}  |
T_TEST_MATERIALS | UNIT_PRICE   | Price (EGP)  | 4  | DECIMAL | false | false | true  | PRICE      | {"Width":14,"Format":"#,##0.00","Align":"Right","HeaderColor":"#70AD47"}  | {"Min":0}
T_TEST_MATERIALS | CATEGORY     | Category     | 5  | TEXT    | false | false | true  | NONE       | {"Width":15}  |
T_TEST_MATERIALS | LAST_UPDATE  | Last Update  | 6  | DATE    | false | false | true  | NONE       | {"Width":12}  |
T_TEST_MATERIALS | INTERNAL_REF | Internal Ref | 7  | TEXT    | false | false | false | NONE       |  |
T_TEST_MATERIALS | PREMIUM_COL  | Premium Only | 8  | DECIMAL | false | false | true  | NONE       |  |
```
Note: `INTERNAL_REF` has IS_VISIBLE=false → only Admin sees it
Note: `PREMIUM_COL` has LICENSE_TIER=Premium in a separate test

### T_TEST_GARB columns (multi-year)
```
ENTITY_KEY | ATTRIBUTE_KEY | DISPLAY_HEADER | ORDINAL_POS | DATA_TYPE | IS_PK | IS_MANDATORY | IS_VISIBLE
T_TEST_GARB | ITEM_CODE  | كود البند   | 1 | TEXT    | true  | true  | true
T_TEST_GARB | ITEM_NAME  | اسم البند   | 2 | TEXT    | false | true  | true
T_TEST_GARB | UNIT       | الوحدة      | 3 | TEXT    | false | false | true
T_TEST_GARB | RATE_2018  | فئة 2018    | 4 | DECIMAL | false | false | true
T_TEST_GARB | RATE_2019  | فئة 2019    | 5 | DECIMAL | false | false | true
T_TEST_GARB | RATE_2020  | فئة 2020    | 6 | DECIMAL | false | false | true
T_TEST_GARB | RATE_2021  | فئة 2021    | 7 | DECIMAL | false | false | true
```

### T_TEST_UNITS columns
```
ENTITY_KEY | ATTRIBUTE_KEY | DISPLAY_HEADER | ORDINAL_POS | DATA_TYPE | IS_PK | IS_MANDATORY | IS_VISIBLE
T_TEST_UNITS | UNIT_CODE   | Code   | 1 | TEXT | true  | true  | true
T_TEST_UNITS | UNIT_NAME   | Name   | 2 | TEXT | false | true  | true
T_TEST_UNITS | SYSTEM      | System | 3 | TEXT | false | false | true
```

### T_TEST_EXCHANGE columns (conversion semantic roles)
```
ENTITY_KEY | ATTRIBUTE_KEY | DISPLAY_HEADER | ORDINAL_POS | DATA_TYPE | IS_PK | SEMANTIC_ROLE
T_TEST_EXCHANGE | PAIR_KEY    | Pair       | 1 | TEXT    | true  | NONE
T_TEST_EXCHANGE | SOURCE_CUR  | From       | 2 | TEXT    | false | CONV_SOURCE
T_TEST_EXCHANGE | TARGET_CUR  | To         | 3 | TEXT    | false | CONV_TARGET
T_TEST_EXCHANGE | RATE        | Rate       | 4 | DECIMAL | false | CONV_FACTOR
T_TEST_EXCHANGE | VALID_DATE  | Valid Date | 5 | DATE    | false | NONE
```

### ERROR: Orphan columns (no matching entity)
```
ENTITY_KEY | ATTRIBUTE_KEY | DISPLAY_HEADER
T_NONEXISTENT | COL_A | Orphan Column A
T_NONEXISTENT | COL_B | Orphan Column B
```

---

## TABLE 3: SYS_DATA_SOURCES (Test Data)

### Normal sources
```
SOURCE_KEY              | TARGET_ENTITY_KEY | PROFILE_KEY    | SOURCE_URI                    | SOURCE_REGION | VERSION_TAG | DISPLAY_LABEL              | MIN_LICENSE_REQ | IS_ACTIVE | CONTEXT_PROPS | RIBBON_CONFIG
SRC_MATERIALS_DEFAULT   | T_TEST_MATERIALS  | T_TEST_MATERIALS| https://docs.google.com/...1 | GLOBAL        | 2026-Q1     | Test Materials Default      | Free            | true      | {"SourceType":"GoogleSheetTsv","SyncFreq":"Daily"}  |
SRC_MATERIALS_KSA       | T_TEST_MATERIALS  | KSA_2026       | https://docs.google.com/...2 | SA            | 2026-Q1     | KSA Materials               | Standard        | true      | {"SourceType":"GoogleSheetTsv","SyncFreq":"Weekly"} |
SRC_GARB_ALL            | T_TEST_GARB       | T_TEST_GARB    | https://docs.google.com/...3 | EG            | 2024        | GARB Full Data              | Free            | true      | {"SourceType":"GoogleSheetTsv","SyncFreq":"Monthly"}|
SRC_UNITS_DEFAULT       | T_TEST_UNITS      | T_TEST_UNITS   | https://docs.google.com/...4 | GLOBAL        |             | Units Reference             | Free            | true      | {} |
SRC_EXCHANGE_DEFAULT    | T_TEST_EXCHANGE   | T_TEST_EXCHANGE| https://docs.google.com/...5 | GLOBAL        | live        | Exchange Rates              | Free            | true      | {"SyncFreq":"Hourly"} |
SRC_HIDDEN_DEFAULT      | T_TEST_HIDDEN     | T_TEST_HIDDEN  | https://docs.google.com/...6 | GLOBAL        |             | Hidden Table Source         | Free            | true      | {} |
```

### SyncFreq test sources
```
SRC_MANUAL_TEST         | T_TEST_MATERIALS  | MANUAL_PROF    | https://docs.google.com/...7 | GLOBAL |  | Manual Only Source | Free | true | {"SyncFreq":"Manual"} |
```

### ERROR: Inactive source
```
SRC_INACTIVE            | T_TEST_MATERIALS  | INACTIVE_PROF  | https://docs.google.com/...8 | GLOBAL |  | Inactive Source | Free | false | {} |
```

### ERROR: Region mismatch (user is SA, source is US)
```
SRC_REGION_BLOCK        | T_TEST_MATERIALS  | US_PROF        | https://docs.google.com/...9 | US     |  | US Only Source  | Free | true  | {} |
```

### ERROR: License insufficient
```
SRC_PREMIUM_BLOCK       | T_TEST_PREMIUM    | T_TEST_PREMIUM | https://docs.google.com/...10| GLOBAL |  | Premium Source  | Premium | true | {} |
```

### ERROR: Orphan source (no matching entity)
```
SRC_ORPHAN              | T_NONEXISTENT     | DEFAULT        | https://docs.google.com/...11| GLOBAL |  | Orphan Source   | Free | true | {} |
```

### ERROR: Missing SOURCE_URI
```
SRC_NO_URI              | T_TEST_MATERIALS  | NO_URI_PROF    |                              | GLOBAL |  | No URI Source   | Free | true | {} |
```

### ERROR: Bad Google Sheet URL (no TSV param)
```
SRC_BAD_URL             | T_TEST_MATERIALS  | BAD_URL_PROF   | https://docs.google.com/spreadsheets/d/abc/edit | GLOBAL | | Bad URL Source | Free | true | {} |
```

---

## TABLE 4: SYS_DATA_MAP (Test Data)

### T_TEST_MATERIALS mapping (Header mode)
```
PROFILE_KEY      | TARGET_ATTRIBUTE_KEY | SOURCE_TYPE | MATCH_MODE | SOURCE_EXPRESSION | TRANSFORM_CHAIN | PROCESS_CONFIG
T_TEST_MATERIALS | ITEM_CODE           | Header      | Exact      | Item Code         |                 |
T_TEST_MATERIALS | ITEM_NAME           | Header      | Exact      | Item Name         |                 | {"AutoTrim":true}
T_TEST_MATERIALS | UNIT                | Header      | Exact      | Unit              |                 |
T_TEST_MATERIALS | UNIT_PRICE          | Header      | Exact      | Price             |                 | {"AutoTrim":true,"NullStrategy":"UseDefault","DefaultValue":"0"}
T_TEST_MATERIALS | CATEGORY            | Header      | Contains   | Cat               |                 |
T_TEST_MATERIALS | LAST_UPDATE         | Header      | Exact      | Date              |                 |
T_TEST_MATERIALS | INTERNAL_REF        | Header      | Exact      | Internal          |                 |
T_TEST_MATERIALS | PREMIUM_COL         | Constant    |            | 0                 |                 |
```

### T_TEST_GARB mapping
```
T_TEST_GARB | ITEM_CODE  | Header | Exact | كود البند  | |
T_TEST_GARB | ITEM_NAME  | Header | Exact | اسم البند  | | {"AutoTrim":true}
T_TEST_GARB | UNIT       | Header | Exact | الوحدة     | |
T_TEST_GARB | RATE_2018  | Header | Exact | فئة 2018   | |
T_TEST_GARB | RATE_2019  | Header | Exact | فئة 2019   | |
T_TEST_GARB | RATE_2020  | Header | Exact | فئة 2020   | |
T_TEST_GARB | RATE_2021  | Header | Exact | فئة 2021   | |
```

### T_TEST_UNITS mapping
```
T_TEST_UNITS | UNIT_CODE | Header | Exact | Code   | |
T_TEST_UNITS | UNIT_NAME | Header | Exact | Name   | |
T_TEST_UNITS | SYSTEM    | Header | Exact | System | |
```

### T_TEST_EXCHANGE mapping
```
T_TEST_EXCHANGE | PAIR_KEY   | Header | Exact | Pair | |
T_TEST_EXCHANGE | SOURCE_CUR | Header | Exact | From | |
T_TEST_EXCHANGE | TARGET_CUR | Header | Exact | To   | |
T_TEST_EXCHANGE | RATE       | Header | Exact | Rate | |
T_TEST_EXCHANGE | VALID_DATE | Header | Exact | Date | |
```

### ERROR: Mapping for non-existent column
```
T_TEST_MATERIALS | GHOST_COLUMN | Header | Exact | Ghost | |
```

### Fuzzy match test
```
KSA_2026 | ITEM_CODE  | Header | Fuzzy | ItemCod   | |
KSA_2026 | ITEM_NAME  | Header | Fuzzy | ItemeName | |
KSA_2026 | UNIT_PRICE | Header | Fuzzy | PriceUnit | |
```

### Context source type
```
T_TEST_MATERIALS | SYNC_DATE | Context | | SYNC_DATE | |
```

### Transform chain test
```
T_TEST_MATERIALS | CATEGORY | Header | Exact | Category | UPPER;TRIM |
```

### RowFilter test
```
T_TEST_MATERIALS | UNIT_PRICE | Header | Exact | Price | | {"RowFilter":"GT:0"}
```

---

## TABLE 5: SYS_CONFIG_PRESETS (Test Data)

### SHEET_COST preset
```
PRESET_KEY  | PROP_KEY      | PROP_VALUE | VAL_TYPE | TARGET_BAG | CATEGORY | DESCRIPTION
SHEET_COST  | TabColor      | #4472C4    | TEXT     | UX_CONFIG  | Visual   | Blue tab
SHEET_COST  | Direction     | LTR        | TEXT     | UX_CONFIG  | Visual   | Left-to-right
SHEET_COST  | FreezePanes   | B2         | TEXT     | UX_CONFIG  | Visual   | Freeze headers
SHEET_COST  | ZoomLevel     | 85         | INT      | UX_CONFIG  | Visual   | Zoom 85%
SHEET_COST  | ShowGridlines | true       | BOOL     | UX_CONFIG  | Visual   | Show gridlines
```

### SYS_DEFAULT preset
```
SYS_DEFAULT | AllowEdit  | false | BOOL | SYS_CONFIG | System | Read-only
SYS_DEFAULT | ExportMode | false | BOOL | SYS_CONFIG | System | No export
```

### GOOGLE_TSV preset
```
GOOGLE_TSV | SourceType     | GoogleSheetTsv | TEXT | CONTEXT_PROPS | Source | Default source type
GOOGLE_TSV | SyncFreq       | Manual         | TEXT | CONTEXT_PROPS | Source | Manual sync
GOOGLE_TSV | TimeoutSeconds | 30             | INT  | CONTEXT_PROPS | Source | 30s timeout
```

### COL_PRICE preset
```
COL_PRICE | Width     | 14       | INT  | UX_CONFIG    | Column | Price column width
COL_PRICE | Format    | #,##0.00 | TEXT | UX_CONFIG    | Column | Price format
COL_PRICE | Align     | Right    | TEXT | UX_CONFIG    | Column | Right-aligned
COL_PRICE | Min       | 0        | INT  | LOGIC_CONFIG | Column | Min value = 0
```

---

## TABLE 6: SYS_EXPORT_VIEWS (Test Data)

### GARB views (main test scenario)
```
VIEW_KEY    | ENTITY_KEY  | LABEL        | SCREEN_TIP          | SUPER_TIP                      | ICON | COLUMNS                         | ALIASES                  | WHERE_FILTER   | SORT_BY         | EXPORT_STYLE | FOOTER_TEXT                    | GROUP_NAME   | SORT_ORDER | IS_ACTIVE | LINKED_VIEWS
GARB_2018   | T_TEST_GARB | أسعار 2018   | فئات 2018           | فئات الهيئة لعام 2018           |      | ITEM_CODE,ITEM_NAME,UNIT,RATE_2018 | {"RATE_2018":"الفئة"} |                | ITEM_NAME ASC   |              |                                | فئات سنوية   | 1          | true      |
GARB_2019   | T_TEST_GARB | أسعار 2019   | فئات 2019           |                                |      | ITEM_CODE,ITEM_NAME,UNIT,RATE_2019 | {"RATE_2019":"الفئة"} |                | ITEM_NAME ASC   |              |                                | فئات سنوية   | 2          | true      |
GARB_2020   | T_TEST_GARB | أسعار 2020   |                     |                                |      | ITEM_CODE,ITEM_NAME,UNIT,RATE_2020 | {"RATE_2020":"الفئة"} |                |                 |              |                                | فئات سنوية   | 3          | true      |
GARB_2021   | T_TEST_GARB | أسعار 2021   | فئات 2021           | أحدث فئات الهيئة               |      | ITEM_CODE,ITEM_NAME,UNIT,RATE_2021 | {"RATE_2021":"الفئة"} | RATE_2021 > 0  | RATE_2021 DESC  |              | المصدر: الهيئة 2021            | فئات سنوية   | 4          | true      | [{"EntityKey":"T_TEST_UNITS"}]
GARB_EXPENSIVE| T_TEST_GARB| بنود غالية   | بنود تتجاوز 5000    | البنود ذات الفئة العالية فقط   |      | ITEM_CODE,ITEM_NAME,UNIT,RATE_2021 | {"RATE_2021":"الفئة"} | RATE_2021>5000 | RATE_2021 DESC  |              | بنود تتجاوز 5000 جنيه          | تحليلات      | 1          | true      |
GARB_ALL    | T_TEST_GARB | كل الفئات    | عرض شامل            |                                |      |                                    |                       |                | ITEM_CODE ASC   |              |                                |              | 99         | true      | [{"EntityKey":"T_TEST_UNITS"},{"EntityKey":"T_TEST_EXCHANGE","ViewKey":"EX_EGP"}]
GARB_DEAD   | T_TEST_GARB | Inactive View|                     |                                |      | ITEM_CODE,ITEM_NAME               |                       |                |                 |              |                                |              | 100        | false     |
```

### Exchange rate views
```
EX_EGP      | T_TEST_EXCHANGE | EGP Rates | Egyptian rates only |  |  | PAIR_KEY,SOURCE_CUR,TARGET_CUR,RATE | | TARGET_CUR='EGP' | RATE DESC |  | Rates to Egyptian Pound |  | 1 | true |
EX_ALL      | T_TEST_EXCHANGE | All Rates |  |  |  |  |  |  |  |  |  |  | 99 | true |
```

### Unit views
```
UNITS_METRIC   | T_TEST_UNITS | Metric Units   | | | | UNIT_CODE,UNIT_NAME | | SYSTEM='METRIC'   | UNIT_CODE ASC | | | | 1 | true |
UNITS_IMPERIAL | T_TEST_UNITS | Imperial Units  | | | | UNIT_CODE,UNIT_NAME | | SYSTEM='IMPERIAL' | UNIT_CODE ASC | | | | 2 | true |
```

### ERROR: Orphan view (no matching entity)
```
VIEW_ORPHAN | T_NONEXISTENT | Orphan View | | | | | | | | | | | 1 | true |
```

### ERROR: Bad ALIASES JSON
```
VIEW_BADJSON | T_TEST_GARB | Bad JSON View | | | | ITEM_CODE,ITEM_NAME | {BROKEN_JSON | | | | | | 1 | true |
```

---

## TEST CHECKLIST

### Phase A: Metadata Pipeline (Boot)

| # | Test | Expected Result | Pass? |
|---|------|-----------------|-------|
| A1 | Start Excel — 6 sheets fetched | Log shows 6 fetches, no errors | |
| A2 | T_TEST_MATERIALS appears in registry | `registry.Get("T_TEST_MATERIALS") != null` | |
| A3 | T_TEST_INACTIVE skipped | Log: "IS_ACTIVE=false", not in registry | |
| A4 | Duplicate T_TEST_MATERIALS logged | Log: "[DUPLICATE] ENTITY_KEY 'T_TEST_MATERIALS'" | |
| A5 | Missing ENTITY_KEY logged | Log: "[ERR_NULL] ENTITY_KEY is mandatory" | |
| A6 | Orphan PARENT_KEY logged | Log: "[ERR_REF] PARENT_KEY='T_NONEXISTENT_PARENT'" | |
| A7 | Orphan columns logged | Log: "[ORPHAN_COLS] T_NONEXISTENT" | |
| A8 | Orphan sources logged | Log: "[ORPHAN_SRC] T_NONEXISTENT" | |
| A9 | Bad JSON logged | Log: "[ERR_FORMAT] Cannot resolve" | |
| A10 | Schema version migration | First run: "TIER-1 schema upgrade: v0 → v2" | |

### Phase B: Preset Resolution

| # | Test | Expected Result | Pass? |
|---|------|-----------------|-------|
| B1 | SHEET_COST preset applied | T_TEST_LABOR.UX_CONFIG.TabColor = "#4472C4" | |
| B2 | Direct JSON applied | T_TEST_MATERIALS.UX_CONFIG.TabColor = "#4472C4" | |
| B3 | Mixed preset + override | Override wins for ZoomLevel | |
| B4 | Missing preset logged | T_TEST_BADJSON → "[ERR_PRESET]" | |
| B5 | Broken JSON logged | T_TEST_BADJSON → "[ERR_FORMAT]" | |

### Phase C: Inheritance

| # | Test | Expected Result | Pass? |
|---|------|-----------------|-------|
| C1 | T_TEST_CHILD inherits ENTITY_TYPE | ENTITY_TYPE = COST (from parent) | |
| C2 | T_TEST_CHILD inherits BUSINESS_DOMAIN | BUSINESS_DOMAIN = MATERIAL | |
| C3 | T_TEST_CHILD overrides ZoomLevel | ZoomLevel = 100 (not 85) | |
| C4 | T_TEST_CHILD inherits TabColor | TabColor = "#4472C4" (from parent) | |
| C5 | RIBBON_CONFIG inherited | Group = "Cost Data" from parent, Label = "Child Override" | |

### Phase D: Data Sync

| # | Test | Expected Result | Pass? |
|---|------|-----------------|-------|
| D1 | Manual source skipped on auto sync | SRC_MANUAL_TEST skipped | |
| D2 | Manual source syncs on user action | SRC_MANUAL_TEST syncs when user clicks | |
| D3 | Inactive source skipped | SRC_INACTIVE → "[SKIP:Inactive]" | |
| D4 | Region blocked | SRC_REGION_BLOCK → "[SKIP:RegionMismatch]" | |
| D5 | License blocked | SRC_PREMIUM_BLOCK → "[SKIP:LicenseInsufficient]" | |
| D6 | No URI logged | SRC_NO_URI → "INVALID or EMPTY URL" | |
| D7 | Bad Google URL logged | SRC_BAD_URL → "missing TSV output parameter" | |
| D8 | Decision hash skip | Second sync same data → "No changes. Skipped" | |
| D9 | Schema drift detection | Change columns → "SCHEMA DRIFT" | |
| D10 | Mapping ghost column | GHOST_COLUMN → "[CONFIG WARNING]" | |

### Phase E: ExportViews

| # | Test | Expected Result | Pass? |
|---|------|-----------------|-------|
| E1 | ExpandViews=false | بنود الهيئة = single button | |
| E2 | ExpandViews=true | بنود الهيئة = sub-menu with 7 items | |
| E3 | GARB_2021 shows 4 columns | ITEM_CODE, ITEM_NAME, UNIT, الفئة | |
| E4 | GARB_2021 alias applied | RATE_2021 → "الفئة" | |
| E5 | GARB_2021 WHERE filter | Only rows with RATE_2021 > 0 | |
| E6 | GARB_ALL shows all columns | All 7 columns | |
| E7 | GARB_DEAD hidden | IS_ACTIVE=false → not in menu | |
| E8 | GROUP_NAME grouping | "فئات سنوية" sub-menu + "تحليلات" sub-menu | |
| E9 | LinkedEntities open | GARB_ALL → 3 sheets (GARB + Units + Exchange EGP) | |
| E10 | VIEW_ORPHAN logged | "[ORPHAN_VIEW] T_NONEXISTENT" | |
| E11 | VIEW_BADJSON handled | Bad aliases → falls back to original headers | |

### Phase F: ExportService

| # | Test | Expected Result | Pass? |
|---|------|-----------------|-------|
| F1 | Basic render | T_TEST_MATERIALS → Excel with all visible columns | |
| F2 | RTL direction | T_TEST_GARB → RTL sheet | |
| F3 | FreezePanes | T_TEST_MATERIALS → B2 frozen | |
| F4 | TabColor | T_TEST_MATERIALS → blue tab | |
| F5 | FooterText - Note | T_TEST_MATERIALS → gray italic footer | |
| F6 | FooterText - Marketing | T_TEST_MARKETING → branded footer | |
| F7 | IS_VISIBLE=false column | INTERNAL_REF hidden for non-Admin | |
| F8 | Protection | AllowEdit=false → sheet protected | |
| F9 | Zebra striping | Alternating row colors | |
| F10 | Column formats | DECIMAL → #,##0.00, DATE → dd/MM/yyyy | |

### Phase G: RibbonMenuBuilder

| # | Test | Expected Result | Pass? |
|---|------|-----------------|-------|
| G1 | Fill with All filter | All visible entities appear | |
| G2 | IsVisibleInMenu | T_TEST_HIDDEN not shown | |
| G3 | GroupedByDomain | Sub-menus: MATERIAL, LABOR, GARB, FINANCE, etc. | |
| G4 | EffectiveLabel | "Material Prices" (from RIBBON_CONFIG) | |
| G5 | EffectiveOrder | Items sorted by RIBBON_CONFIG.Order | |
| G6 | Rich ScreenTip | Hover shows Strategy, License, Columns, Status | |
| G7 | ExportViews sub-menu | GARB has فئات سنوية + تحليلات sub-groups | |
| G8 | Tag format | "T_TEST_GARB|GARB_2021" for views | |

### Phase H: ActionRouter

| # | Test | Expected Result | Pass? |
|---|------|-----------------|-------|
| H1 | EXPORT action | RenderToSheet called | |
| H2 | EXPORT with viewKey | RenderToSheet("T_TEST_GARB", "GARB_2021") | |
| H3 | DOWNLOAD action | DownloadEngine called | |
| H4 | Unknown action | MessageBox "No action configured" | |

### Phase I: Network & Recovery

| # | Test | Expected Result | Pass? |
|---|------|-----------------|-------|
| I1 | Offline start | Loads from SQLite cache | |
| I2 | Network disconnect | No crash, buttons reset to Idle | |
| I3 | Network reconnect | Auto-sync resumes after debounce | |
| I4 | Pipeline timeout | Stages timeout gracefully | |

### Phase J: Schema Migration

| # | Test | Expected Result | Pass? |
|---|------|-----------------|-------|
| J1 | Fresh install | All 6 tables created, v=2 | |
| J2 | Upgrade from v1 | Tables dropped and recreated | |
| J3 | Same version | No drop, quick CREATE IF NOT EXISTS | |

---

## SAMPLE TSV DATA (for actual Google Sheets)

### T_TEST_MATERIALS source data
```tsv
Item Code	Item Name	Unit	Price	Category	Date	Internal
MAT-001	Portland Cement	Ton	1200.50	Cement	2026-01-15	REF-001
MAT-002	Steel Rebar 12mm	Ton	22500.00	Steel	2026-01-15	REF-002
MAT-003	Fine Sand	m³	350.00	Aggregate	2026-01-15	REF-003
MAT-004	Crushed Stone	m³	280.00	Aggregate	2026-01-15	REF-004
MAT-005	Plywood 18mm	m²	185.00	Formwork	2026-01-15	REF-005
MAT-006		m²	0		2026-01-15	
MAT-007	Red Brick	1000pc	4500.00	Masonry	2026-01-15	REF-007
	Empty Code Row	Ton	100.00	Error	2026-01-15	
```
Note: Row 6 has empty name, Row 8 has empty PK → tests NULL_CRITICAL

### T_TEST_GARB source data
```tsv
كود البند	اسم البند	الوحدة	فئة 2018	فئة 2019	فئة 2020	فئة 2021
GARB-001	خرسانة عادية	م³	850	920	1100	1250
GARB-002	حديد تسليح	طن	12000	13500	15000	22000
GARB-003	أعمال حفر	م³	35	40	45	55
GARB-004	مباني طوب	م²	180	200	220	260
GARB-005	بياض أسمنتي	م²	65	70	80	95
GARB-006	دهانات زيتية	م²	45	50	55	0
```
Note: GARB-006 RATE_2021=0 → tests WHERE_FILTER "RATE_2021 > 0"

### T_TEST_UNITS source data
```tsv
Code	Name	System
m³	Cubic Meter	METRIC
m²	Square Meter	METRIC
Ton	Metric Ton	METRIC
ft³	Cubic Foot	IMPERIAL
ft²	Square Foot	IMPERIAL
lb	Pound	IMPERIAL
```

### T_TEST_EXCHANGE source data
```tsv
Pair	From	To	Rate	Date
USD-EGP	USD	EGP	50.85	2026-03-31
EUR-EGP	EUR	EGP	55.20	2026-03-31
GBP-EGP	GBP	EGP	64.10	2026-03-31
USD-SAR	USD	SAR	3.75	2026-03-31
SAR-EGP	SAR	EGP	13.56	2026-03-31
```
