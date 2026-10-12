# نظام إدارة الموظفين — الحضور بالـ GPS، الأجور، المصروفات، والأمان

[![CI](https://github.com/YOUR_USERNAME/employee-management/actions/workflows/ci.yml/badge.svg)](https://github.com/YOUR_USERNAME/employee-management/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Node](https://img.shields.io/badge/node-%E2%89%A518-brightgreen)](package.json)
[![Vanilla JS](https://img.shields.io/badge/frontend-vanilla%20JS%20—%20no%20framework-blue)](#3-المزايا-كيف-تعمل)

نظام ويب كامل (Full‑stack) لإدارة موظفين شركة صغيرة/متوسطة: تسجيل دخول بصلاحيتين،
حضور وانصراف بالموقع الجغرافي (GPS)، حساب الأجور من الساعات الفعلية، تسجيل المصروفات،
وتتبّع أجهزة الدخول مع تنبيه فوري للأدمن عند ظهور جهاز جديد.

- الواجهة: **HTML + CSS + Vanilla JS** فقط (بدون أي Framework)، كل صفحة ملف `.html` مستقل مربوط بالروابط، والتواصل مع الخادم بـ `fetch`.
- الخادم: **Node.js + Express**، قاعدة البيانات **PostgreSQL** (مع تحوّل تلقائي إلى **SQLite** لو تعذّر الاتصال).
- الأمان: **bcrypt** لكلمات المرور + **JWT** للجلسات + تخزين ومقارنة بصمة الجهاز.
- الاتجاه RTL بالكامل ومتجاوب (موبايل/تابلت/ديسكتوب).

---

## لقطات (ملتقطة من التطبيق وهو يعمل)

| لوحة تحكم الأدمن — إحصاءات اليوم ورادار النطاق | حضور الموظف بالـ GPS — داخل النطاق |
|---|---|
| ![لوحة التحكم](docs/screenshots/02-admin-dashboard.png) | ![تسجيل الحضور](docs/screenshots/10-employee-checked-in.png) |

| الأجور من الساعات الفعلية | على موبايل 390px (RTL كامل) |
|---|---|
| ![الأجور](docs/screenshots/05-admin-payroll.png) | ![موبايل](docs/screenshots/13-mobile-dashboard.png) |

المعرض الكامل (١٥ شاشة) وطريقة إعادة توليد اللقطات من التطبيق نفسه: [`docs/SCREENSHOTS.md`](docs/SCREENSHOTS.md).

---

## 1) التشغيل السريع

```bash
npm install
cp .env.example .env      # راجع بيانات قاعدة البيانات فيه
npm run setup             # db:migrate + db:seed (بيانات تجريبية)
npm start                 # http://localhost:3000
```

للتطوير مع إعادة تشغيل تلقائية: `npm run dev`

### حسابات جاهزة بعد `db:seed`

| الدور | المستخدم | كلمة المرور |
|---|---|---|
| أدمن (كل الصلاحيات) | `admin` | `Admin@123` |
| موظف | `mona` / `ahmed` / `karim` / `sara` / `mahmoud` | `Emp@12345` |

> قاعدة البيانات فاضية؟ النظام يعمل `seed` تلقائيًا عند أول تشغيل (يمكن إيقافه بـ `SEED_ON_EMPTY=0`).

### أوامر مفيدة

```bash
npm run db:doctor           # تشخيص: المحرك الفعلي (Neon/pg/SQLite) + SSL + عدد صفوف كل جدول
npm run db:migrate        # إنشاء/تحديث الجداول فقط
npm run db:seed           # إدخال البيانات التجريبية
npm run db:seed -- --force# مسح الجداول ثم إعادة التوليد
npm run reset             # حذف الجداول كلها (DROP) ثم migrate + seed — يسأل عن تأكيد YES
npm run reset -- --yes    # بدون سؤال
npm run reset -- --no-seed# حذف وإعادة بناء القاعدة فاضية
npm run test:api          # اختبارات الـ API (54 خطوة تكامل حقيقية عبر HTTP)
npm run test:front        # فحص ساكن للواجهة: معرّفات DOM، دوال EMS، مسارات /api، الملفات المرتبطة
npm run build:firebase    # توليد ems-firebase.html + deploy-firebase/ من النسخة المستقلة
npm run test:firebase     # 58 تأكيد لطبقة Firestore فوق Firestore وهمي (بلا شبكة)
npm run test:firebase:dom # 90 تأكيد: إقلاع نسخة Firebase داخل jsdom (يتطلب jsdom)
npm run test:shapes       # عقد الاختبار بين الواجهة والخادم: كل حقل تقرأه صفحة موجود فعلًا في الرد
npm test                  # الثلاثة معًا (يتطلب سيرفرًا يعمل على 3000 للفحصين الأخيرين)
```

---

## 2) هيكل المشروع

```
employee-management/
├── server/
│   ├── app.js                    توسيط Express: أمان، static، /api، 404، معالج أخطاء
│   ├── index.js                  migrate → seed عند الحاجة → listen
│   ├── config.js                 قراءة .env، اختيار المحرك، أسرار JWT، SMTP، حدود الدخول
│   ├── db/
│   │   ├── index.js              طبقة بيانات واحدة (all/get/insert/run/exec/tx) + ? → $n
│   │   ├── schema.postgres.sql   11 جدول (PostgreSQL)
│   │   ├── schema.sqlite.sql     نفس المخطط بنسخة SQLite
│   │   ├── migrate.js            تنفيذ ملف الـ schema (idempotent)
│   │   ├── seed.js               بيانات تجريبية حتمية (5 موظفين + 181 سجل حضور…)
│   │   └── reset.js              حذف الجداول وإعادة الضبط
│   ├── lib/                      time (توقيت محلي/UTC) · geo (Haversine) · validate · hash · token · http · sanitize · bus
│   ├── middleware/auth.js        تحقق JWT + requireAdmin + وضع القراءة للموظف
│   ├── services/                 settings · device (البصمة) · attendance (المنطق) · payroll · notify (إشعارات/إيميل)
│   └── routes/                   11 ملف: auth, users, fields, attendance, payroll, expenses, settings, security, notifications, pages, reports
├── public/                       الواجهة (كل صفحة ملف مستقل)
│   ├── css/styles.css            نظام التصميم (RTL، متغيرات، بطاقات، جداول، Modal، وضع طباعة)
│   ├── js/                       api.js (EMS) · fingerprint.js · geolocation.js · map.js · layout.js + سكربت لكل صفحة
│   ├── login.html · index.html · 404.html · page.html
│   ├── admin-dashboard · admin-employees · admin-attendance · admin-payroll · admin-expenses · admin-security · admin-settings · admin-pages
│   └── employee-dashboard · attendance · employee-payroll · profile
├── tests/api.test.js             53 خطوة تكامل (دخول، GPS، صلاحيات، تسريبات، تصديرات CSV…)
├── tests/front.check.js          فحص ساكن للواجهة
├── docs/API.md                   توثيق كل نقاط الـ API مع أمثلة الطلب/الاستجابة
└── data/                         .jwt-secret (يُنشأ تلقائيًا) وقاعدة SQLite عند الحاجة
```

### خريطة الصفحات

| الصفحة | لمن | ماذا تفعل |
|---|---|---|
| `login.html` | الجميع | دخول + جمع بصمة الجهاز وتحويل حسب الصلاحية |
| `admin-dashboard.html` | أدمن | إجمالي الأجور والمصروفات، حضور اليوم، خريطة مكان العمل، تنبيهات الأجهزة |
| `admin-employees.html` | أدمن | إضافة/تعديل/حذف موظفين، سعر الساعة، الحالة، الحقول المخصصة، تصدير CSV |
| `admin-attendance.html` | أدمن | كل سجلات الحضور + تعديل يدوي + محاولات مرفوضة + مصفوفة الشهر |
| `admin-payroll.html` | أدمن | كشف أجور شهري/أسبوعي/مخصص + بدلات وخصومات + تفاصيل كل يوم |
| `admin-expenses.html` | أدمن | المصروفات (عامة/مرتبطة بموظف) + تقارير حسب الفئة/الموظف/الشهر |
| `admin-security.html` | أدمن | سجل الدخول، الأجهزة وبصماتها، اعتماد/إلغاء، تنبيهات، فحص بصمة جهازك |
| `admin-settings.html` | أدمن | كل إعدادات النظام: إحداثيات العمل والنطاق، السياسة، الأجور، الأمان، المظهر |
| `admin-pages.html` | أدمن | إنشاء صفحات وأقسام جديدة تظهر في القائمة الجانبية (HTML/Markdown) |
| `employee-dashboard.html` | موظف | ساعاته وأجره وحضوره فقط |
| `attendance.html` | موظف | زرّي «تسجيل حضور / تسجيل انصراف» بالـ GPS + سجله |
| `employee-payroll.html` | موظف | كشف راتبه (شهري/أسبوعي) ومعادلة الحساب |
| `profile.html` | الجميع | بياناته، تغيير كلمة المرور، أجهزته، بصمته |
| `page.html?slug=…` | حسب الصفحة | عرض أي صفحة/قسم مخصص |

---

## 3) المزايا وكيف تعمل

### 🔐 الصلاحيات (Admin / Employee)
- **أدمن**: كل شيء بلا قيود — إنشاء/تعديل/حذف موظفين، تغيير أي حقل (بما فيه سعر الساعة، الحالة، كلمة المرور)،
  ضبط Latitude/Longitude ونطاق السماح بالمتر، المصروفات، التقارير، الإعدادات، وإضافة صفحات/أقسام جديدة.
- **موظف**: بياناته فقط. الخادم يفرض ذلك في كل نقطة (وليست خفية في الواجهة فقط):
  محاولة قراءة/تعديل موظف آخر → `403`، ومحاولة تعديل حقول خارج قائمة الموظف → `400`.
  الموظف لا يستطيع أيضًا حذف حسابه (تُرفض من الخادم).

### 📍 الحضور/الانصراف بالـ GPS
1. الواجهة تطلب الموقع بـ `navigator.geolocation.getCurrentPosition` (مع `enableHighAccuracy`).
2. تُرسَل `lat/lng/accuracy` إلى `POST /api/attendance/check-in` أو `check-out`.
3. **الحساب في الخادم** (القرار ليس للمنفّذ النهائي): مسافة `Haversine` بين موقع الموظف ومكان العمل
   مقارنة بـ `radius_m`، وفحص دقة القراءة مقابل `max_accuracy_m`.
4. داخل النطاق ⇒ يُسجَّل الوقت والتاريخ فورًا في جدول `attendance` (قيد واحد لكل موظف لكل يوم).
   خارج النطاق ⇒ `403` بالرسالة المطلوبة نصًا:
   > **لازم تكون في مكان الشغل عشان تسجل حضور**

   ولا يتم أي تسجيل للحضور (تُسجَّل فقط محاولة مرفوضة في `attendance_events` لتظهر للأدمن).
5. رفض المتصفح إذن الموقع ⇒ رسالة تفعيل GPS + زر «أبلغ الإدارة» يستدعي `POST /api/attendance/gps-error`
   فيصل للأدمن إشعار `gps_denied`.
6. السجل: تاريخ + دخول + خروج + عدد الساعات (للوظيفة ولمديرها) مع المصدر (GPS أو يدوي) والمسافة.

> قبل الضغط على الزر تظهر معاينة «المسافة الحالية عن مكان العمل: … م / النطاق … م» عبر `POST /api/attendance/preview`
> (لا تسجّل شيئًا)، ومتابعة تلقائية للموقع باستخدام `watchPosition`.

### 💰 الأجور
- **سعر الساعة لكل موظف** (`users.hourly_rate_cents` — تُخزَّن بالمِنَسَّة/100 لتفادي أخطاء الفاصلة العائمة).
- الأجر = **الساعات من سجل الحضور الفعلي × سعر الساعة** بعد خصم وقت الراحة وتقريب الدقائق ومعامل الأوفر تايم.
- إجماليات أسبوعية وشهرية (وأي مدى مخصص) + بدلات/مكافآت/خصومات من `payroll_adjustments` ⇒ **صافي**.
- أي تغيير في سياسة الحضور من صفحة الإعدادات يعيد النظام حساب السجلات (`attendance.recalcAll`)،
  وزر «إعادة حساب السجلات» متاح يدويًا.
- تصدير CSV لكشف الأجور، وصفحة الموظف تعرض المعادلة خطوة بخطوة.

### 🧾 المصروفات
`title` (وصف) · `amount` · `spent_on` (تاريخ) · `user_id` (اختياري: مرتبطة بموظف أو عامة) · فئة · مورد · طريقة دفع · رقم فاتورة · ملاحظة.
تقرير الأدمن: الإجمالي، حسب الفئة، حسب الموظف/عام، حسب الشهر، + تصدير CSV + طباعة.
الموظف يرى مصروفاته هو فقط من `GET /api/expenses/mine`.

### 🛡️ بصمة الجهاز والتنبيهات
- عند كل دخول تُبنى بصمة من: `User-Agent` + اللغة/المناطق الزمنية + دقة الشاشة/Viewport + `hardwareConcurrency` + `deviceMemory` + نقاط اللمس + `deviceUuid` (UUID في `localStorage`) + أول `/24` من الـ IP، ثم تُجزَّأ إلى **SHA‑256**.
- أول جهاز للموظف ⇒ يُسجَّل كـ **جهاز معتاد** (`is_usual = 1`).
- دخول من جهاز مختلف ⇒ إشعار **فوري** للأدمن (in‑app عبر SSE + جرس المتصفح) وفيه اسم الموظف والوقت ومعلومات الجهاز،
  وإيميل عبر **Nodemailer** لو `email_alerts_enabled=1` و SMTP مهيأ.
- سجل عمليات الدخول (`login_events`) يفرّق: معتاد / أول جهاز / جهاز جديد / سبب الفشل، وقابل للتصدير CSV.
- صفحة الأمان تسمح للأدمن: اعتماد الجهاز، إلغاء الاعتماد، تعيين «المعتاد»، تسمية الجهاز، حذفه، وفحص بصمة جهازه الحالي.
- سياسة اختيارية `require_usual_device` تمنع تسجيل الحضور/الانصراف من جهاز غير معتاد
  (`403` بالرسالة: «لا يمكن تسجيل الحضور من جهاز غير معتاد — تواصل مع الإدارة»، `details.code = unknown_device`)
  وتسجّل المحاولة في قائمة المرفوضات + إشعار `device_policy` للأدمن. الواجهة تُرسل بصمة الجهاز مع طلب التسجيل تلقائيًا عند تفعيل السياسة.

### 🖥️ الداشبورد
- **الأدمن**: كل الموظفين، إجمالي الأجور للفترة، إجمالي المصروفات، خريطة مكان العمل (رادار SVG + رابط OSM)، تنبيهات الأجهزة غير المعتادة، حضور اليوم، أعلى المصروفات، من لم يسجّل بعد، ومواقع آخر التسجيلات.
- **الموظف**: ساعاته وأجره وسجل حضوره وأجهزته فقط — لا اسم زميل ولا رقم يظهر له.

---

## 4) الإعدادات (صفحة admin-settings.html)

كل القيم في جدول `settings` وقابلة للتعديل من الواجهة (تُبنى الحقول ديناميكيًا من `GET /api/settings/schema`).
أهمها:

| المجموعة | المفاتيح |
|---|---|
| 📍 مكان العمل | `workplace_name`, `workplace_address`, **`workplace_lat`**, **`workplace_lng`**, **`radius_m`**, `max_accuracy_m`, `allow_check_in_before`, `allow_admin_punch`, `map_zoom` |
| ⏱️ السياسة | `work_start_time`, `work_end_time`, `standard_hours_per_day`, `grace_minutes`, `break_minutes`, `min_hours_for_break`, `rounding_minutes`, `timezone`, `week_start_day`, `off_days`, `auto_close_open_shift`, **`require_usual_device`** |
| 💰 الأجور | `overtime_multiplier`, `currency_symbol`, `currency_code`, `pay_period` |
| 🛡️ الأمان | **`device_alert_enabled`**, `email_alerts_enabled`, `alert_email`, `password_min_length`, `session_ttl_hours` |
| 🎨 المظهر | `app_title`, `company_logo_text`, `footer_note`, `theme_accent` |

`radius_m` = نصف قطر السماح بالمتر حوالـ (lat,lng). ضبط الإحداثيات من الواجهة بسهولة:
زر «🛰️ استخدم موقعي الحالي كمركز» (وأنت داخل مكان العمل) أو «📏 معاينة النطاق» لقياس بُعدك الحقيقي قبل الحفظ.

---

## 5) قواعد البيانات والتوقيت

- الجداول: `users`, `settings`, `devices`, `login_events`, `attendance`, `attendance_events`,
  `expenses`, `payroll_adjustments`, `notifications`, `custom_pages`, `employee_fields`.
- كل الطوابع الزمنية **ISO‑8601 UTC** في النص، والتحويل للتوقيت المحلي (`Africa/Cairo` افتراضيًا) في الطبقة المنطقية
  (`server/lib/time.js`) — لا يعتمد النظام على منطقة زمن الخادم.
- النقود **بالسنت** (`*_cents INTEGER`)، والمنطقية `0/1`، والـ JSON نص — لتفادي اختلاف أنواع المحركين.
- `attendance` فيه `UNIQUE(user_id, work_date)` ⇒ سجل واحد لكل يوم (لو تم إغلاقه تلقائيًا عند منتصف الليل حسب `auto_close_open_shift`).

---

## 6) الاختبارات

```bash
npm test
```

- `tests/api.test.js`: 54 خطوة تكامل حقيقية عبر HTTP على سيرفر يعمل — تغطي: الدخول/الجلسة/الصلاحيات،
  الحضور داخل وخارج النطاق (الرسالة المطلوبة)، الدقة الضعيفة، رفض الإذن، رفض التسجيل من جهاز غير معتاد،
  التسجيل اليدوي للأدمن (يسجَّل في سجل الموظف المطلوب لا سجل الأدمن + احترام `allow_admin_punch`)،
  الأجور والبدلات والتصدير، المصروفات وربطها، الإعدادات وإعادة الحساب، تنبيه الجهاز الجديد واعتماده،
  الصفحات المخصصة وتنقية الـ HTML من السكربتات، ومنع تسريب بصمات/بيانات الآخرين.
- `tests/firebase.check.js`: طبقة Firestore (نسخة الملف الواحد) فوق Firestore وهمي: التهيئة، الـ diff
  (إضافة/تعديل/حذف/إعدادات)، الدفعات ≤400، الفرز بعد السحب، إعادة التحميل الآمن، فشل الكتابة + إعادة
  المحاولة، `fbWipe`، debounce، وصفوف بلا `id` — 58 تأكيد، ومُدمَجة في `npm test`.
- `tests/front.check.js`: فحص ساكن — كل `byId()` موجود في الصفحة، كل `EMS.x` مُصدَّر،
  كل `/api/...` مستخدم في الواجهة يطابق route مسجّلًا، وكل `<script>/<link>/<a href="*.html">` يشير لملف موجود.
- `tests/shapes.check.js`: يتأكد أن الردود الحقيقية تحتوي كل المفاتيح التي تقرأها صفحات الموقع
  (يفشل فورًا لو غيّر طرف اسم حقل دون الآخر)، ويتحقق أيضًا أن الموظف يُمنع `403` من مسارات `/api/security/*`.

يعمل الاختبار على المحركين: `DB_CLIENT=postgres` (افتراضي) أو `DB_CLIENT=sqlite`.

---

## 7) ملاحظات تشغيلية وأمنية

- **HTTPS إلزامي عمليًا**: المتصفح لا يعطي إذن `geolocation` إلا في صفحة آمنة (localhost مُستثنى).
  في الإنتاج شغّل الخادم خلف nginx/Caddy بشهادة TLS.
- `JWT_SECRET` يُولَّد تلقائيًا في `data/.jwt-secret` (لا يُنشر مع الكود). في الإنتاج حدّده في متغيرات البيئة وغيّره دوريًا.
- كلمات المرور bcrypt بـ `BCRYPT_ROUNDS` (10 افتراضيًا)؛ لا تُخزَّن ولا تُرسَل أبدًا — تُرسل فقط كلمة المرور الحالية عند تغييرها.
- البصمة **ليست** بديلًا عن المصادقة القوية: تُستخدم للتنبيه واكتشاف الأجهزة الجديدة فقط، وهذا ما صُمّمت من أجله.
- حدود محاولات الدخول: `LOGIN_MAX_ATTEMPTS`/`LOGIN_WINDOW_MS` مع رسالة `429` مؤقتة.
- الواجهة لا تخزّن أي سر: التوكن في `localStorage` (يمكن تحويله Cookie‑HTTP‑only لو رغبت بشدة أكبر من ذلك).
- لرؤية الإشعارات اللحظية: `GET /api/notifications/stream?access_token=…` (SSE) مربوط تلقائيًا في `layout.js` + طلب إذن إشعارات المتصفح.
- الطباعة: لكل التقارير `@media print` مخصص (كشف أجور، تقرير مصروفات، مصفوفة الحضور).
- لو تعذّر تشغيل PostgreSQL يحوّل النظام تلقائيًا إلى SQLite في `data/ems.sqlite` (يمكن ضبط `SQLITE_FILE`)،
  وفرض المحرك يتم بـ `DB_CLIENT=postgres|sqlite`.

## 8) نسخة تجريبية في ملف HTML واحد (بدون سيرفر)

`ems-offline.html` (في جذر المشروع) ملف واحد مكتمل بذاته — يفتح بدبل كليك على أي جهاز بدون Node ولا قاعدة بيانات:

- نفس منطق النظام: Haversine + نطاق السماح + حدّ دقة الـ GPS + بوابة الجهاز المعتاد + احتساب الساعات/الأجر + البدلات والخصومات + المصروفات + الصفحات المخصصة + الصلاحيات (أدمن/موظف).
- الرسائل منسوخة حرفيًا من `server/services/attendance.js` (`MESSAGES`).
- البيانات وهمية في `localStorage` تحت المفتاح `ems-offline-db-v1` (6 حسابات، ~30 يوم حضور لكل موظف مع الجمعة عطلة، 10 مصروفات، 4 محاولات مرفوضة، 3 صفحات مخصصة، 2 تسوية أجر) — وزر «↺ إعادة ضبط البيانات» في شاشة الدخول يرجّعها لأصلها.
- حسابات الدخول نفسها: `admin / Admin@123` · `mona / Emp@12345`.
- `navigator.geolocation` لا يعمل داخل إطارات المعاينة أو بدون HTTPS، لذلك في الصفحة زرّا **«موقع تجريبي داخل/خارج النطاق»** (وفي `الإعدادات` زر «موقعي الحالي كمركز») لتجربة المسارَين كاملَين.
- فحص حيّ للواجهة: `npm i -D jsdom && npm run test:offline` (`tests/offline.check.js`) يشغّل الملف داخل jsdom ويغطي 84 نقطة: الإقلاع، كل شاشات الأدمن والموظف، رفض التسجيل خارج النطاق بدون تسجيل أي شيء، الرسالة الحرفية للانصراف، حساب الساعات والأجر بعد العودة للداخل، عزل بيانات الموظف، وسجل المحاولات الفاشلة. ولو jsdom غير مثبّت يطبع رسالة تخطّي ويخرج 0.
- لا يُصدَّر منه CSV للاتصال بالخادم: كل شيء محلي، ولا يوجد أي طلب شبكة خارجي (رابط OpenStreetMap للنقر اليدوي فقط).
- **نسخة Firebase (Firestore):** `npm run build:firebase` يولّد `ems-firebase.html` — نفس المنطق،
  لكن التخزين في Firestore (مستند لكل صف، `localStorage` كاش، مزامنة عند الرجوع للتبويب، وشارة حالة
  أسفل اليسار). من غير Web Config الملف يتصرّف زي النسخة المستقلة تمامًا (`local-only`).
  `ems-offline.html` نفسه ما بيتعدّلش. الخطوات والقيود: [`docs/FIREBASE.md`](docs/FIREBASE.md).

## 9) مستندات إضافية

- توثيق الـ API كاملًا مع أمثلة: [`docs/API.md`](docs/API.md)
- خطوات ربط قاعدة Neon (Postgres مجانية) وتشخيص الاتصال: [`docs/NEON.md`](docs/NEON.md)
- ربط النسخة المستقلة بـ Firebase (Firestore) + قواعد الأمان + النشر على Hosting: [`docs/FIREBASE.md`](docs/FIREBASE.md)
- النشر على Cloudflare (Static / سيرفر خارج / Worker+D1) ومقارنة الداتا بيز المجانية: [`DEPLOY-CLOUDFLARE.md`](DEPLOY-CLOUDFLARE.md)
- معرض اللقطات الحقيقية وطريقة توليدها: [`docs/SCREENSHOTS.md`](docs/SCREENSHOTS.md)
- بيانات البيئة: `.env.example` (مُعلّق عليه بالعربي)

---

## 10) الجاهزية لـ GitHub

- **الترخيص:** MIT ([`LICENSE`](LICENSE)) · سياسة الإبلاغ عن الثغرات: [`SECURITY.md`](SECURITY.md).
- **ما لا يُرفع أبدًا** (`.gitignore`): `.env` وكل نسخه (`.env.*`)، `data/*` (قاعدة SQLite و`data/.jwt-secret`)،
  `logs/` و`*.log`، `node_modules/`، `serviceAccount*.json`، `firebase-web.json`، `.firebase/`، و`.claude/` (أدوات وكيل).
- **CI** (`.github/workflows/ci.yml`) — 4 وظائف على كل push/PR + وظيفة خامسة للنشر (main فقط):
  1. **SQLite**: `npm ci` → تشغيل السيرفر → `npm test` (54 تكامل + فحص ساكن + عقد الواجهات + طبقة Firestore «58»).
  2. **PostgreSQL 16** (خدمة داخل نفس الـ job): `db:migrate` → `db:seed` → `db:doctor` → `npm test` — نفس الاختبارات على المحرك الموصى به.
  3. **نسخة الملف الواحد**: `test:offline` (84) + `test:firebase:dom` (90) داخل jsdom.
  4. **طزاجة المُولَّد**: `node tools/build-firebase.js` ثم `git diff --exit-code` — يمنع التعديل اليدوي داخل `ems-firebase.html`.
  5. **Pages**: `ems-offline.html` → `site/index.html` (+`.nojekyll`) بعد حرس `npm run check:single` أنه بلا أي مورد خارجي — يُنشر تلقائيًا على `https://<user>.github.io/employee-management/` عند كل push على `main`.
- **قوالب المساهمة**: `.github/ISSUE_TEMPLATE/{bug_report.yml,feature_request.yml,config.yml}` (نماذج من حقول، والأمان موجَّه إلى Private vulnerability reporting) + `.github/PULL_REQUEST_TEMPLATE.md` بقائمة تحقق تربط بالاختبارات والنسخ الثلاث.
- **الاعتماديات**: Dependabot شهري لـ npm وGitHub Actions (`.github/dependabot.yml`)، بمجموعات minor/patch.
- **توحيد التحرير**: `.editorconfig` + `.gitattributes` (LF إجباري — فحص «المُولَّد مطابق» يعتمد عليه).
- **ملفات مولّدة مخفية من الـ diff**: `ems-firebase.html`، `deploy-firebase/public/index.html`، `package-lock.json`.

### إعداد المستودع بعد الإنشاء (About / Topics / Pages)

النص الجاهز للصقه في **About** (⚙️ على GitHub):

```text
نظام إدارة موظفين full-stack بلا إطار عمل: Vanilla JS + Express + Postgres/SQLite. حضور بالـ GPS بنطاق ودقة قابلين للضبط، أجور من الساعات الفعلية، مصروفات، بصمة جهاز وتنبيهات فورية. عربي RTL ومتجاوب. نسخة تجريبية في ملف HTML واحد + طبقة Firestore.
```

**Topics** (٢٠، الحد الأقصى):

```text
employee-management, attendance-system, gps-attendance, payroll, expenses, hr-system,
vanilla-js, no-framework, express, nodejs, postgresql, sqlite, firebase, firestore,
jwt, bcrypt, geolocation, rtl, arabic, self-hosted
```

**Website**: `https://<user>.github.io/employee-management/` — ثم تشغيل النشر من الـ CLI (واجهة الويب لا تعرض خيار Actions أحيانًا):

```bash
gh api repos/{owner}/employee-management/pages -X POST -f build_type=workflow
gh workflow run ci.yml        # أو: git commit --allow-empty -m "chore: trigger Pages deploy" && git push
```

> النشر يتمّ من وظيفة `pages` في نفس الـ workflow؛ لا يحتاج `CNAME` ولا فرع `gh-pages`،
> والملف المنشور هو `ems-offline.html` حرفيًا (بياناته في `localStorage`، بلا سيرفر).

### أول مرة لمساهم
```bash
git clone <الرابط> && cd employee-management
npm ci
cp .env.example .env      # اضبط DB_CLIENT / DATABASE_URL حسب بيئة التطوير
npm run setup             # migrate + seed
npm start                 # طرفية أولى — http://localhost:3000
npm test                  # طرفية ثانية (تتطلب السيرفر يعمل)
```

### النشر على GitHub
```bash
git init -b main && git add -A && git commit -m "EMS: full-stack + نسخة ملف واحد + Firestore"
gh repo create employee-management --public --source=. --push          # لو GitHub CLI موجود
# يدويًا: أنشئ مستودعًا فارغًا (بدون README/LICENSE) ثم:
git remote add origin git@github.com:USER/employee-management.git
git push -u origin main
```
> بعد إنشاء المستودع استبدل `YOUR_USERNAME`/`USER` في بادجات أعلى هذا الملف وحقول
> `repository` / `homepage` / `bugs` في `package.json`.

### أدوات اختيارية في الجذر (تُحذف بأمان إن لم تستخدمها)
| الملف | الدور |
|---|---|
| `neon.ts` + `@neon/*` (devDeps) | سياسة Neon عبر `neon deploy` — تحتاج `neon login`؛ التطبيق لا يستوردها |
| `tools/build-firebase.js` | يولّد `ems-firebase.html` و`deploy-firebase/` من `ems-offline.html` |
| `tools/firestore-driver.snippet.js` | مصدر طبقة Firestore (يُحقن في البناء — لا تحرّر النسخة داخل الـ HTML) |
| `deploy-firebase/` | مجلد النشر إلى Firebase Hosting + قواعد Firestore |
| `tests/firebase.check.js` | 58 تأكيد لطبقة Firestore فوق Firestore وهمي (بلا شبكة) |

