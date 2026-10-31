# تشغيل Polytechnic Study Hub للطلاب

هذه الطريقة تثبّت الموقع مباشرة من Docker. **لا تحتاج ZIP أو GitHub repository أو VS Code أو Node.js.** تنطبق الخطوات على Mac وWindows.

## أول تشغيل

### 1. جهّز Docker وGitHub

1. ثبّت Docker Desktop وشغّله.
2. سيعطيك صاحب الموقع بشكل خاص اسم مستخدم التوزيع ورمز قراءة مخصصاً لتنزيل التطبيق. هذا الحساب مخصص للتوزيع، ولديه صلاحية **Read** على حزمة التطبيق فقط؛ لا يملك صلاحية نشر أو صلاحية على مستودع الكود.

### 2. سجّل الدخول إلى حزمة الموقع

افتح **Terminal** على Mac أو **PowerShell** على Windows، ثم نفّذ:

```sh
docker login ghcr.io -u DISTRIBUTION_USERNAME
```

عندما يظهر `Password:`, الصق رمز القراءة الذي أرسله لك صاحب الموقع بشكل خاص، ثم اضغط Enter. لن تظهر أحرف الرمز. لا تضع أي رمز حقيقي في هذا الملف أو في الكود. يجب أن تظهر `Login Succeeded`.

### 3. شغّل Study Hub

انسخ الأمر التالي كاملاً ونفّذه مرة واحدة في Terminal أو PowerShell:

```sh
docker run -d --name studyhub-student --restart unless-stopped -p 127.0.0.1:8081:8080 -v polytechnic-study-hub-student:/data -e NODE_ENV=production -e PORT=8080 -e HOST=0.0.0.0 -e DATABASE_PATH=/data/polytechnic-study-hub.sqlite ghcr.io/reoobh/polytechnic-study-hub:v5.2.2-private-midterm-20261031
```

قد يستغرق التنزيل أول مرة بعض الوقت. حسابك وتقدمك يُحفظان في Docker volume باسم `polytechnic-study-hub-student` على جهازك.

### 4. أنشئ حسابك الدراسي

نفّذ هذا الأمر مرة واحدة:

```sh
docker exec -it studyhub-student node scripts/provision-student.mjs
```

اختر اسم مستخدم وكلمة مرور. إدخال كلمة المرور مخفي. احفظ **رموز الاسترجاع** التي تظهر لك في مكان آمن وخاص؛ لن تظهر مرة أخرى.

### 5. افتح الموقع

اذهب إلى [http://localhost:8081](http://localhost:8081) وسجّل الدخول بحسابك الدراسي.

## في المرات القادمة

1. شغّل Docker Desktop وافتح [http://localhost:8081](http://localhost:8081).
2. إذا لم يفتح الموقع، افتح Docker Desktop → **Containers**، وابحث عن `studyhub-student` واضغط ▶ **Start**.
3. أو شغّله من Terminal على Mac أو PowerShell على Windows:

```sh
docker start studyhub-student
```

لا تحتاج إلى تكرار `docker run` أو `docker login` أو إنشاء الحساب. حسابك وتقدمك محفوظان.

## إذا نسيت كلمة المرور

شغّل الأمر التالي في Terminal أو PowerShell، ثم أدخل اسم المستخدم ورمز استرجاع غير مستخدم وكلمة مرور جديدة:

```sh
docker exec -it studyhub-student node scripts/recover-student-account.mjs
```

سيتم إلغاء الجلسات السابقة مع الاحتفاظ بحسابك ودرجاتك ومحاولاتك وتقدمك. احفظ مجموعة رموز الاسترجاع الجديدة. لا ترسل كلمة المرور أو الرموز لصاحب الموقع.

## مشاكل شائعة

- إذا ظهر `unauthorized`، تأكد أن صاحب الموقع أرسل لك اسم مستخدم التوزيع ورمز القراءة الصحيحين، ثم أعد `docker login`.
- إذا كان المنفذ 8081 مستخدماً، أغلق التطبيق الآخر الذي يستخدمه ثم أعد التشغيل.
- **لا تحذف** Docker volume المسمى `polytechnic-study-hub-student`؛ حذفه يمسح حسابك وتقدمك.
- سجّل الخروج على الأجهزة المشتركة. مسؤول الجهاز يستطيع الوصول إلى بيانات Docker المحلية؛ تسجيل الدخول لا يمنع ذلك.

هذه صورة خاصة للمراجعة الدراسية للمجموعة المصرح لها فقط. لا تشارك الصورة أو الأسئلة أو لقطات الامتحانات مع أشخاص آخرين.
