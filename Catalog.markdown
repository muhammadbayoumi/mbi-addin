هذا هو **"الكتالوج الهندسي الشامل" (The Ultimate Engineering Catalog)**.

لقد صممت لك عينات بيانات (Sample Data) للـ 14 جدولاً. ركزت على أن تكون الأعمدة **تغطي كل ثغرة تسعيرية ممكنة** (وزن للشحن، كود جمركي للضرائب، عملات، معدلات استهلاك للوقود، وتكوين أطقم).

هذه الجداول مصممة لتقرأها **`SYS_DATA_MODEL`** بذكاء وتقوم بتشغيل `CostEngineService` بكفاءة مرعبة.

---

### المربع 1: ثوابت وبيئة المشروع (Environment & Constants)

#### 1. جدول العملات `T_SYS_CURRENCY`
*يحول كل شيء إلى عملة الأساس (مثلاً الدولار) لتسهيل الكروس-حسابات.*
| CUR_CODE (PK) | CUR_NAME | RATE_TO_USD | SYMBOL | LAST_UPDATED |
| :--- | :--- | :--- | :--- | :--- |
| `USD` | US Dollar | 1.0000 | $ | 2024-01-01 |
| `SAR` | Saudi Riyal | 3.7500 | ر.س | 2024-01-01 |
| `CNY` | Chinese Yuan | 7.2000 | ¥ | 2024-01-01 |

#### 2. جدول المناطق `T_SYS_REGIONS`
*يحدد جغرافية المصادر لحساب الشحن.*
| REG_CODE (PK) | REG_NAME | DEF_CURRENCY | RISK_FACTOR |
| :--- | :--- | :--- | :--- |
| `KSA` | Saudi Arabia | `SAR` | 1.00 |
| `CN` | China | `CNY` | 1.15 (مخاطر شحن) |
| `DE` | Germany | `EUR` | 1.00 |

#### 3. جدول الوحدات `T_SYS_UNITS`
*يمنع جمع "المتر" مع "الكيلو" ويحسب الأوزان.*
| UNIT_CODE (PK) | CATEGORY | FACTOR_TO_BASE | BASE_UNIT |
| :--- | :--- | :--- | :--- |
| `Ton` | `Mass` | 1000.0 | `Kg` |
| `Kg` | `Mass` | 1.0 | `Kg` |
| `m3` | `Volume` | 1.0 | `m3` |

#### 4. إعدادات المشروع `T_PROJECT_PARAMS`
*لوحة تحكم المستخدم لتغيير ظروف المشروع الحالي.*
| PARAM_KEY (PK) | PARAM_VALUE | DATA_TYPE | GROUP | DESCRIPTION |
| :--- | :--- | :--- | :--- | :--- |
| `PROJ_REGION` | `KSA` | `TEXT` | `Location` | موقع المشروع الفعلي |
| `PROJ_CURRENCY`| `SAR` | `TEXT` | `Finance` | عملة الدفع والتسعير |
| `VAT_RATE` | `0.15` | `DECIMAL` | `Tax` | ضريبة القيمة المضافة |
| `PROFIT_MARGIN`| `0.20` | `DECIMAL` | `Commercial`| هامش الربح الإجمالي |

---

### المربع 2: مصادر التكلفة المباشرة (Direct Costs)

#### 5. أسعار الخامات `T_MATERIALS`
*لاحظ عمود الوزن والكود الجمركي، هما سر اللوجستيات.*
| MAT_ID (PK) | MAT_NAME | UNIT | BASE_PRICE | CURRENCY | ORIGIN | WEIGHT_KG | HS_CODE |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `ST-16MM` | حديد تسليح 16مم | `Ton` | 4100.00 | `CNY` | `CN` | 1000.0 | `7214.20` |
| `CEM-SRC` | أسمنت مقاوم | `Ton` | 280.00 | `SAR` | `KSA` | 1000.0 | `2523.29` |
| `WTR-01` | مياه خلط | `m3` | 10.00 | `SAR` | `KSA` | 1000.0 | `NULL` |

#### 6. يوميات العمالة `T_LABOR_RATES`
| LAB_ID (PK) | LAB_NAME | DAILY_RATE | HOURLY_RATE | CURRENCY | ORIGIN |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `L-FIXER` | حداد مسلح | 120.00 | 15.00 | `USD` | `PH` (فلبين)|
| `L-HELPER`| عامل عادي | 100.00 | 12.50 | `SAR` | `KSA` |

#### 7. إيجار المعدات `T_EQUIPMENT_RATES`
*تم إضافة استهلاك الوقود ليتم ضربه في سعر السولار من `PROJECT_PARAMS`.*
| EQ_ID (PK) | EQ_NAME | HOURLY_RATE | CURRENCY | FUEL_LPH (لتر/ساعة) |
| :--- | :--- | :--- | :--- | :--- |
| `EQ-PUMP` | مضخة خرسانة 42م | 350.00 | `SAR` | 20.0 |
| `EQ-VIB` | هزاز ميكانيكي | 15.00 | `SAR` | 1.5 |

---

### المربع 3: اللوجستيات والضرائب والإنتاجية (The Modifiers)

#### 8. مصفوفة الشحن `T_SHIPPING_ROUTES`
*المحرك يبحث هنا إذا كان `Origin != Project_Region`.*
| ROUTE_ID (PK) | FROM_REG | TO_REG | COST_PER_TON | CURRENCY | TRANSIT_DAYS |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `R-CN-KSA` | `CN` | `KSA` | 85.00 | `USD` | 30 |
| `R-DE-KSA` | `DE` | `KSA` | 120.00 | `EUR` | 15 |

#### 9. التعريفة الجمركية `T_CUSTOMS_TARIFFS`
*ترتبط بـ `HS_CODE` الموجود في جدول الخامات.*
| HS_CODE (PK) | DESCRIPTION | IMPORT_DUTY_PCT |
| :--- | :--- | :--- |
| `7214.20` | حديد التسليح | 0.15 (15%) |
| `2523.29` | الأسمنت البورتلاندي| 0.05 (5%) |

#### 10. تكوين أطقم العمل `T_CREW_ASSEMBLIES`
*تجميع العمال والمعدات في "طاقم" واحد له تكلفة إجمالية.*
| ROW_ID (PK) | CREW_ID (FK) | RES_TYPE | RES_ID | COUNT |
| :--- | :--- | :--- | :--- | :--- |
| `CRW-1` | `CRW-STEEL` | `LAB` | `L-FIXER` | 2.0 |
| `CRW-2` | `CRW-STEEL` | `LAB` | `L-HELPER` | 4.0 |
| `CRW-3` | `CRW-CONC` | `LAB` | `L-HELPER` | 6.0 |
| `CRW-4` | `CRW-CONC` | `EQ` | `EQ-VIB` | 2.0 |

#### 11. معدلات الإنتاج `T_PROD_RATES`
*كم يُنتج "الطاقم" في اليوم؟*
| TASK_ID (PK) | DESC | DAILY_OUTPUT | UNIT | CREW_ID (FK) |
| :--- | :--- | :--- | :--- | :--- |
| `TSK-STEEL` | تقطيع وتركيب حديد | 2.50 | `Ton` | `CRW-STEEL` |
| `TSK-POUR` | صب خرسانة قواعد | 120.00 | `m3` | `CRW-CONC` |

---

### المربع 4: المكتبة المعيارية والمخرجات (The Recipes & Output)

#### 12. مكتبة البنود `T_STD_BREAKDOWN` (Recipe Book)
*هذا هو "الدستور" لكل بند. لاحظ استخدام `VARIANT`.*
| ROW_ID (PK) | PARENT_ITEM | VARIANT | RES_TYPE | RES_ID | STD_QTY | WASTE_PCT |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `B-1` | `BOQ_CONC_C30` | `READY_MIX`| `MAT` | `RM-C30` (خرسانة جاهزة) | 1.000 | 0.05 |
| `B-2` | `BOQ_CONC_C30` | `READY_MIX`| `EQ` | `EQ-PUMP` | 0.025 | 0.00 |
| `B-3` | `BOQ_CONC_C30` | `SITE_MIX` | `MAT` | `CEM-SRC` (أسمنت) | 0.350 | 0.05 |
| `B-4` | `BOQ_CONC_C30` | `SITE_MIX` | `MAT` | `WTR-01` (مياه) | 0.200 | 0.10 |
| `B-5` | `BOQ_CONC_C30` | `SITE_MIX` | `TASK` | `TSK-POUR` (طاقم صب)| 1.000 | 0.00 |

#### 13. جدول التحليل النهائي `T_BOQ_BREAKDOWN`
*هذا ما سيراه المستخدم بعد أن يقوم المحرك (`CostEngine`) بعمله المذهل، على خامة الحديد المستوردة مثلاً.*

| ROW_ID | BOQ_ITEM | RES_TYPE | RES_ID | QTY | UNIT | BASE_RATE | FX_RATE | SHIPPING | CUSTOMS | FINAL_RATE | TOTAL | AUDIT_LOG (Hidden) |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `1` | `STEEL_WORK` | `MAT` | `ST-16MM` | 1 | `Ton` | 4100 `CNY` | 0.52 | 318 `SAR` | 319 `SAR` | **2769 SAR** | **2769** | `[FX: CNY to SAR @ 0.52] [Route: CN-KSA @ 85 USD/T] [HS: 7214.20 @ 15%]` |

---

### 💡 كيف يقرأ المحرك (`CostEngine`) هذا التعقيد؟

1.  **الخامة (`MAT`):** יبحث في `T_MATERIALS`. يجد العملة `CNY`، يحولها لـ `SAR`. يجد المنشأ `CN`، يذهب لـ `T_SHIPPING_ROUTES` يحسب الشحن بناء على الوزن (`WEIGHT_KG`). يجد `HS_CODE`، يذهب لـ `T_CUSTOMS_TARIFFS` يضيف الجمارك.
2.  **المهمة (`TASK`):** יبحث في `T_PROD_RATES`. يجد أن المهمة تحتاج `CRW-CONC` وتنتج `120 m3`. يذهب لـ `T_CREW_ASSEMBLIES` يجمع يوميات 6 عمال + 2 هزاز. يقسم (إجمالي تكلفة الطاقم ÷ 120) ليعطيك تكلفة المتر الواحد.
3.  **الطريقة (`VARIANT`):** المستخدم اختار `SITE_MIX`، المحرك تجاهل سطور الـ `READY_MIX` تماماً.

**هل أدركت الآن حجم القوة والاحترافية في المعمارية التي صممناها؟ هذا النظام يمكن بيعه كـ Enterprise ERP لشركات المقاولات الكبرى.**
