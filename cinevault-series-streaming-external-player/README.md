# CineVault / viE

منصة مشاهدة خاصة مبنية بـ React + Vite + Express. تحتوي على أكواد دخول للمشاهدين، مشغل فيديو، حفظ التقدم، لوحة إدارة، إدارة مواسم/حلقات/مصادر فيديو، Google Drive وDirect URL ومصادر HTTP أخرى.

## الدخول المخفي للإدارة

لا يوجد زر Admin ظاهر في الموقع.

1. افتح صفحة إدخال **Access Code** العادية.
2. اكتب: `vin01012007`
3. ستفتح صفحة تسجيل دخول الإدارة.
4. اسم المستخدم: `vin`
5. كلمة المرور: `waswaswas`

الرمز المخفي يتم التحقق منه في السيرفر، ولا يتم وضعه داخل JavaScript الخاص بالواجهة. بعد إدخاله يتم إنشاء تصريح مؤقت لمدة 10 دقائق لفتح نموذج تسجيل الإدارة. واجهات `/api/admin/*` تبقى محمية بجلسة Admin فعلية.

> هذه البيانات مطلوبة كما هي في هذه النسخة. إذا أصبح المستودع عامًا أو شاركت المشروع مع أشخاص آخرين، غيّرها من متغيرات البيئة أو من لوحة الإعدادات.

## تشغيل المشروع محليًا

المطلوب: Node.js 22.12 أو أحدث.

```bash
npm install
npm run dev
```

ثم افتح:

```text
http://localhost:3000
```

الملف `.env` مرفق للتشغيل المحلي، وهو مستثنى من Git عن طريق `.gitignore`.

### أكواد مشاهدة تجريبية

- `DEMO-PASS`
- `CINE-PREMIUM`
- `STREAM-VIP-2026`

الفيديوهات الموجودة في البيانات الأولية هي فيديوهات Demo عامة للاختبار فقط. استبدلها من لوحة الإدارة بمصادر تملك حق استخدامها.

## فحص المشروع وبناء نسخة Production

```bash
npm run check
npm run build
npm start
```

- `npm run check`: فحص TypeScript.
- `npm run build`: يبني React إلى `dist/` ويبني السيرفر إلى `dist-server/server.js`.
- `npm start`: يشغل النسخة المبنية في وضع Production.

يمكن فحص حالة السيرفر من:

```text
/api/health
```

## التخزين

المشروع يستخدم تخزين JSON دائم في:

```text
data/db.json
```

ويمكن تغيير مكانه بمتغير:

```env
DATA_DIR="./data"
```

الكتابة ذرية قدر الإمكان، ويتم حفظ البيانات عند الإغلاق. جلسات الأدمن أيضًا أصبحت محفوظة في التخزين بدل الذاكرة فقط.

**مهم:** هذه النسخة مصممة لتشغيل Instance واحد. لا تشغّل عدة Replicas تشترك في ملف JSON نفسه. إذا احتجت توسع أفقي كبير لاحقًا، انقل طبقة التخزين إلى PostgreSQL/Redis.

## متغيرات البيئة

```env
NODE_ENV="development"
PORT="3000"
DATA_DIR="./data"

ADMIN_PORTAL_CODE="vin01012007"
ADMIN_USERNAME="vin"
ADMIN_PASSWORD="waswaswas"
SESSION_SECRET="ضع-قيمة-عشوائية-طويلة-هنا"

GOOGLE_CLIENT_ID=""
GOOGLE_CLIENT_SECRET=""
GOOGLE_REFRESH_TOKEN=""
GOOGLE_API_KEY=""
```

`ADMIN_USERNAME` و`ADMIN_PASSWORD` يستخدمان عند إنشاء قاعدة البيانات لأول مرة. بعد إنشاء `data/db.json` يمكن تغيير بيانات الإدارة من لوحة Settings بدون أن يتم إرجاعها تلقائيًا عند كل تشغيل.

## النشر على Railway

المشروع يحتوي `railway.json` جاهزًا للبناء والتشغيل.

1. ارفع المشروع إلى GitHub. لا ترفع ملف `.env`؛ هو مستثنى أصلًا.
2. في Railway اختر **New Project → Deploy from GitHub Repo**.
3. أضف Volume دائم واربطه بالمسار:

```text
/app/data
```

4. أضف متغيرات Railway التالية:

```env
NODE_ENV=production
DATA_DIR=/app/data
ADMIN_PORTAL_CODE=vin01012007
ADMIN_USERNAME=vin
ADMIN_PASSWORD=waswaswas
SESSION_SECRET=<ضع سرًا عشوائيًا طويلًا جدًا>
```

وأضف متغيرات Google Drive فقط إذا كنت تستخدم Google Drive الخاص.

5. Build Command الموجود في الإعدادات:

```text
npm run build
```

6. Start Command:

```text
npm start
```

7. Health Check:

```text
/api/health
```

بعد النشر يمكنك ربط Domain خاص من إعدادات Railway.

## تحسينات الأمان الموجودة

- لا يوجد زر أو رابط Admin ظاهر للمستخدم.
- بوابة الإدارة السرية تُفتح من خانة Access Code فقط.
- التحقق من رمز بوابة الإدارة يتم في السيرفر.
- نموذج Admin Login نفسه لا يقبل الطلب قبل فتح البوابة السرية.
- جلسات Admin تستخدم رموزًا عشوائية ولا يتم حفظ الرمز الخام في `db.json`؛ يتم حفظ SHA-256 فقط.
- Cookies أصبحت `HttpOnly` وبـ `SameSite=Lax` وتتحول إلى `Secure` تلقائيًا في Production.
- لم يعد Admin token محفوظًا في `localStorage`.
- Logout يلغي جلسة المشاهد فعليًا حتى لا تبقى محسوبة ضمن حد الأجهزة.
- جلسات Admin تستمر بعد Restart لأنها محفوظة في التخزين.
- Rate limiting على محاولات دخول Admin.
- Security headers وHSTS في Production.
- صفحة API غير المعروفة ترجع JSON 404 بدل صفحة React.

## ملاحظات الفيديو

المتصفح يشاهد من endpoint داخلي مثل:

```text
/api/episodes/:id/stream
```

والسيرفر يقوم بجلب المصدر الحقيقي. هذا يمنع عرض رابط المصدر مباشرة في الواجهة، لكنه لا يجعل نسخ الفيديو مستحيلًا بشكل مطلق. حماية المحتوى الاحترافية تحتاج CDN signed URLs/DRM حسب احتياجك وحقوق المحتوى.
