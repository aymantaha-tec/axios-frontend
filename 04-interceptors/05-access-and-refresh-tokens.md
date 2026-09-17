# Refresh Token بيعمل إيه؟

أثناء الـ Login، الـ Backend ممكن يرجع:

```text id="c7n4vm"
Access Token
+
Refresh Token
```

مثلاً:

```text id="j2p8kw"
Access Token
  ↓
قصير العمر

Refresh Token
  ↓
أطول عمرًا
```

الـ Access Token تستخدمه مع الـ API requests العادية.

أما الـ Refresh Token تستخدمه لما الـ Access Token **ينتهي** للحصول على Access Token جديد.

---

## الـ Flow

```text id="r8m3qx"
Login
 ↓
Backend
 ↓
Access Token + Refresh Token
 ↓
Frontend
```

بعد شوية:

```text id="q5v7nz"
API Request
 ↓
Access Token
 ↓
Backend
 ↓
401 Token Expired
```

بدل Login:

```text id="w4k9xp"
Refresh Token
 ↓
Backend
 ↓
Access Token جديد
 ↓
إعادة الـ Request الأصلي
```

---

## مثال

عندك:

```text id="h6p2mc"
Access Token = A123
Refresh Token = R999
```

تعمل:

```text id="f9k4vx"
GET /profile
Authorization: Bearer A123
```

لكن `A123` انتهى.

Backend:

```text id="n3q7ws"
401
```

الـ application يستخدم:

```text id="u8m5ka"
R999
```

للحصول على:

```text id="b6x2pz"
Access Token جديد = A456
```

وبعدين يعيد الطلب:

```text id="q9c4ym"
GET /profile
Authorization: Bearer A456
```

والمستخدم غالبًا **مش بيحس بأي حاجة**.

---

## الفرق في جملة واحدة

```text id="s2h8kw"
Access Token
→ أستخدمه للوصول للـ API.


Refresh Token
→ أستخدمه للحصول على Access Token جديد.
```

### وهنا هتفهم ليه الـ Interceptors مهمة جدًا

الـ **Request Interceptor**:

```text id="r4m7qx"
كل Request
   ↓
حط Access Token
```

والـ **Response Interceptor**:

```text id="t6k3vp"
Response
   ↓
401؟
   ↓
نبدأ Refresh Flow
```

لكن **لسه مش هننفذ الـ 401 والـ refresh flow بالكود**. ده القسم اللي بعده.

**الخطوة التالية:** نعمل سؤال Interview واحد على `Access Token vs Refresh Token` قبل ما نقفل `4.4 Authentication`.
<details>
<summary>Ok</summary>
user
Ok
ChatGPT
جاوب على سؤال الـ **Interview** ده في دقيقتين:

### 🗺️ Roadmap

```text
SECTION 4: Interceptors 🟢
  ├─ 4.1 What is an Interceptor    ✅
  ├─ 4.2 Request Interceptor       ✅
  ├─ 4.3 Response Interceptor      ✅
  ├─ 4.4 Authentication            🟢
  │    ├─ Authentication vs Authorization ✅
  │    ├─ Access Token              ✅
  │    └─ Refresh Token             ✅
  ├─ 4.5 401 Handling              ⬜ ← التالي
  └─ 4.6 Token Refresh + Queue     ⬜
```

### 🎯 Interview

اشرح الفرق بين:

**Access Token** و **Refresh Token**

وليه لما الـ **Access Token** ينتهي، مش بنطلب من المستخدم يدخل الـ Email والـ Password مرة تانية؟

جاوب كأنك في Interview.
</details>
Ok
ChatGPT
جاوب على سؤال الـ **Interview** ده في دقيقتين:

### 🗺️ Roadmap

```text
SECTION 4: Interceptors 🟢
  ├─ 4.1 What is an Interceptor    ✅
  ├─ 4.2 Request Interceptor       ✅
  ├─ 4.3 Response Interceptor      ✅
  ├─ 4.4 Authentication            🟢
  │    ├─ Authentication vs Authorization ✅
  │    ├─ Access Token              ✅
  │    └─ Refresh Token             ✅
  ├─ 4.5 401 Handling              ⬜ ← التالي
  └─ 4.6 Token Refresh + Queue     ⬜
```

### 🎯 Interview

اشرح الفرق بين:

**Access Token** و **Refresh Token**

وليه لما الـ **Access Token** ينتهي، مش بنطلب من المستخدم يدخل الـ Email والـ Password مرة تانية؟

جاوب كأنك في Interview.
<details>
<summary>الفرق بين الـ access token والـ refresh token، الـ access token ده بيجي لنا أول مرة لما بندخل الإيميل والباسورد في التطبيق، فيبدأ الباك إند بيعرفك التأمين، وساعتها بيديك access token عشان تقدر تبعت أي API من خلاله وتحط الـ access token ده اللي هو مدخولك في الـ headers، الـ config، في interceptor، عشان يقدر بعد كده يعرفك التأمين. طيب في حالة... طيب بس هنا في مشكلة، الـ access token ده ليه مدة معينة، هنفترض مثلاً إن هي 15 دقيقة، بعد ما الـ 15 دقيقة يخلصوا، ييجي بعد كده تبعت access token، ساعتها يقولك: لا، إنت خلاص وقتك انتهى، وده مش access token ده غير صالح. فساعتها بيعمل حاجة اسمها refresh token، يعني بيقوله من الآخر إن access token ده انتهى، ومن الآخر يعمل إيه؟ access token تاني، أوكي؟ بس.</summary>
user
الفرق بين الـ access token والـ refresh token، الـ access token ده بيجي لنا أول مرة لما بندخل الإيميل والباسورد في التطبيق، فيبدأ الباك إند بيعرفك التأمين، وساعتها بيديك access token عشان تقدر تبعت أي API من خلاله وتحط الـ access token ده اللي هو مدخولك في الـ headers، الـ config، في interceptor، عشان يقدر بعد كده يعرفك التأمين. طيب في حالة... طيب بس هنا في مشكلة، الـ access token ده ليه مدة معينة، هنفترض مثلاً إن هي 15 دقيقة، بعد ما الـ 15 دقيقة يخلصوا، ييجي بعد كده تبعت access token، ساعتها يقولك: لا، إنت خلاص وقتك انتهى، وده مش access token ده غير صالح. فساعتها بيعمل حاجة اسمها refresh token، يعني بيقوله من الآخر إن access token ده انتهى، ومن الآخر يعمل إيه؟ access token تاني، أوكي؟ بس.
ChatGPT
ممتاز، **9.5/10**. الفكرة الأساسية عندك صحيحة جدًا.

بس عندك نقطة واحدة محتاجة تصحيح:

أنت قلت:

> "فساعتها بيعمل حاجة اسمها Refresh Token"

الأدق إن **الـ Refresh Token موجود أصلًا من وقت الـ Login**، مش بيتعمل وقت انتهاء الـ Access Token.

الـ flow الصحيح:

```text
Login
 ↓
Backend
 ↓
Access Token + Refresh Token
```

الـ Access Token:

```text
صالح 15 دقيقة
↓
نستخدمه في API Requests
```

بعد 15 دقيقة:

```text
API Request
 ↓
Access Token expired
 ↓
401
```

هنا نستخدم **Refresh Token الموجود بالفعل**:

```text
Refresh Token
 ↓
Backend
 ↓
Access Token جديد
 ↓
إعادة الـ Request
```

يعني:

```text
Access Token
→ مفتاح مؤقت للوصول للـ API


Refresh Token
→ مفتاح نستخدمه للحصول على Access Token جديد
```

والسبب إننا مش نطلب Email + Password تاني هو إن الـ **Refresh Token** يسمح لنا بتجديد الـ Access Token بدون إعادة تسجيل الدخول.

### 🟢 4.4 Authentication مكتملة

```text
Authentication vs Authorization ✅
Access Token                    ✅
Refresh Token                   ✅
```

**الخطوة التالية:** نبدأ `4.5 401 Handling` ونفهم بالضبط **ليه الـ Backend بيرجع 401 وإزاي الـ Response Interceptor يلتقطها**.
</details>
الفرق بين الـ access token والـ refresh token، الـ access token ده بيجي لنا أول مرة لما بندخل الإيميل والباسورد في التطبيق، فيبدأ الباك إند بيعرفك التأمين، وساعتها بيديك access token عشان تقدر تبعت أي API من خلاله وتحط الـ access token ده اللي هو مدخولك في الـ headers، الـ config، في interceptor، عشان يقدر بعد كده يعرفك التأمين. طيب في حالة... طيب بس هنا في مشكلة، الـ access token ده ليه مدة معينة، هنفترض مثلاً إن هي 15 دقيقة، بعد ما الـ 15 دقيقة يخلصوا، ييجي بعد كده تبعت access token، ساعتها يقولك: لا، إنت خلاص وقتك انتهى، وده مش access token ده غير صالح. فساعتها بيعمل حاجة اسمها refresh token، يعني بيقوله من الآخر إن access token ده انتهى، ومن الآخر يعمل إيه؟ access token تاني، أوكي؟ بس.
ChatGPT
ممتاز، **9.5/10**. الفكرة الأساسية عندك صحيحة جدًا.

بس عندك نقطة واحدة محتاجة تصحيح:

أنت قلت:

> "فساعتها بيعمل حاجة اسمها Refresh Token"

الأدق إن **الـ Refresh Token موجود أصلًا من وقت الـ Login**، مش بيتعمل وقت انتهاء الـ Access Token.

الـ flow الصحيح:

```text
Login
 ↓
Backend
 ↓
Access Token + Refresh Token
```

الـ Access Token:

```text
صالح 15 دقيقة
↓
نستخدمه في API Requests
```

بعد 15 دقيقة:

```text
API Request
 ↓
Access Token expired
 ↓
401
```

هنا نستخدم **Refresh Token الموجود بالفعل**:

```text
Refresh Token
 ↓
Backend
 ↓
Access Token جديد
 ↓
إعادة الـ Request
```

يعني:

```text
Access Token
→ مفتاح مؤقت للوصول للـ API


Refresh Token
→ مفتاح نستخدمه للحصول على Access Token جديد
```

والسبب إننا مش نطلب Email + Password تاني هو إن الـ **Refresh Token** يسمح لنا بتجديد الـ Access Token بدون إعادة تسجيل الدخول.

### 🟢 4.4 Authentication مكتملة

```text
Authentication vs Authorization ✅
Access Token                    ✅
Refresh Token                   ✅
```

**الخطوة التالية:** نبدأ `4.5 401 Handling` ونفهم بالضبط **ليه الـ Backend بيرجع 401 وإزاي الـ Response Interceptor يلتقطها**.
<details>
<summary>طب وليه اللفه دي يعني ليه يمكن في رفريش تكون بس</summary>
user
طب وليه اللفه دي يعني ليه يمكن في رفريش تكون بس
ChatGPT
ببساطة في **3 دقائق**: اللفة دي معمولة أساسًا عشان **الأمان + تجربة المستخدم**.

لو خلينا الـ `Access Token` صالح لمدة طويلة جدًا، زي سنة:

```text
Access Token
↓
صالح سنة
```

لو حد سرقه، يقدر يستخدمه فترة طويلة.

فبنخليه قصير:

```text
Access Token
↓
15 دقيقة
```

لو اتسرق، الضرر المحتمل أقل.

لكن لو خلينا المستخدم يعمل Login كل 15 دقيقة:

```text
Login
↓
15 دقيقة
↓
Login تاني
↓
15 دقيقة
↓
Login تاني
```

دي تجربة سيئة جدًا.

فبنستخدم:

```text id="7h4k2m"
Access Token
↓
قصير العمر
↓
للـ API Requests


Refresh Token
↓
أطول عمرًا
↓
يطلع Access Token جديد
```

فتجمع بين:

**Security** 🔒  
الـ Access Token قصير العمر.

**User Experience** 👤  
المستخدم مش محتاج يعمل Login كل شوية.

### تشبيه سريع

اعتبر الـ Access Token **تصريح دخول مؤقت**:

```text
تصريح صالح 15 دقيقة
```

والـ Refresh Token كأنه **وسيلة تجديد التصريح**:

```text
التصريح انتهى؟
↓
استخدم وسيلة التجديد
↓
هات تصريح جديد
```

**الخطوة التالية:** ندخل `4.5 401 Handling` ونشوف إزاي النظام يعرف إن الـ Access Token انتهى.
</details>
طب وليه اللفه دي يعني ليه يمكن في رفريش تكون بس
ChatGPT
ببساطة في **3 دقائق**: اللفة دي معمولة أساسًا عشان **الأمان + تجربة المستخدم**.

لو خلينا الـ `Access Token` صالح لمدة طويلة جدًا، زي سنة:

```text
Access Token
↓
صالح سنة
```

لو حد سرقه، يقدر يستخدمه فترة طويلة.

فبنخليه قصير:

```text
Access Token
↓
15 دقيقة
```

لو اتسرق، الضرر المحتمل أقل.

لكن لو خلينا المستخدم يعمل Login كل 15 دقيقة:

```text
Login
↓
15 دقيقة
↓
Login تاني
↓
15 دقيقة
↓
Login تاني
```

دي تجربة سيئة جدًا.

فبنستخدم:

```text id="7h4k2m"
Access Token
↓
قصير العمر
↓
للـ API Requests


Refresh Token
↓
أطول عمرًا
↓
يطلع Access Token جديد
```

فتجمع بين:

**Security** 🔒  
الـ Access Token قصير العمر.

**User Experience** 👤  
المستخدم مش محتاج يعمل Login كل شوية.

### تشبيه سريع

اعتبر الـ Access Token **تصريح دخول مؤقت**:

```text
تصريح صالح 15 دقيقة
```

والـ Refresh Token كأنه **وسيلة تجديد التصريح**:

```text
التصريح انتهى؟
↓
استخدم وسيلة التجديد
↓
هات تصريح جديد
```

**الخطوة التالية:** ندخل `4.5 401 Handling` ونشوف إزاي النظام يعرف إن الـ Access Token انتهى.
<details>
<summary>مش بيتخزن ده عشان يطلع اكسيس جديد صح كده</summary>
user
مش بيتخزن ده عشان يطلع اكسيس جديد صح كده
ChatGPT
أيوه **بالضبط**، بس خلّي الجملة أدق شوية:

**الـ Refresh Token بيتخزن من وقت الـ Login، ولما الـ Access Token ينتهي بنستخدم الـ Refresh Token عشان نطلب Access Token جديد.**

التدفق:

```text
Login
  ↓
Backend
  ↓
Access Token + Refresh Token
  ↓
Storage
```

بعد فترة:

```text
Request
  ↓
Access Token منتهي
  ↓
401 Unauthorized
  ↓
نستخدم Refresh Token
  ↓
Backend
  ↓
New Access Token
  ↓
Retry Request
```

⚠️ نقطة مهمة:  
الـ **Refresh Token نفسه مش بيتجدد في كل مرة بالضرورة**. حسب نظام الـ Backend، ممكن يفضل كما هو، أو يحصل **Refresh Token Rotation** ويتطلع واحد جديد مع الـ Access Token الجديد.

**الخطوة التالية (5 دقائق):** نبدأ **4.5 401 Handling** ونفهم بالضبط إزاي الـ Response Interceptor يعرف إن الـ Access Token انتهى.
</details>
مش بيتخزن ده عشان يطلع اكسيس جديد صح كده
ChatGPT
أيوه **بالضبط**، بس خلّي الجملة أدق شوية:

**الـ Refresh Token بيتخزن من وقت الـ Login، ولما الـ Access Token ينتهي بنستخدم الـ Refresh Token عشان نطلب Access Token جديد.**

التدفق:

```text
Login
  ↓
Backend
  ↓
Access Token + Refresh Token
  ↓
Storage
```

بعد فترة:

```text
Request
  ↓
Access Token منتهي
  ↓
401 Unauthorized
  ↓
نستخدم Refresh Token
  ↓
Backend
  ↓
New Access Token
  ↓
Retry Request
```

⚠️ نقطة مهمة:  
الـ **Refresh Token نفسه مش بيتجدد في كل مرة بالضرورة**. حسب نظام الـ Backend، ممكن يفضل كما هو، أو يحصل **Refresh Token Rotation** ويتطلع واحد جديد مع الـ Access Token الجديد.

**الخطوة التالية (5 دقائق):** نبدأ **4.5 401 Handling** ونفهم بالضبط إزاي الـ Response Interceptor يعرف إن الـ Access Token انتهى.
<details>
<summary>ممكن مثال علي تكن وفين القريش وفين التاني</summary>
user
ممكن مثال علي تكن وفين القريش وفين التاني
ChatGPT
خلينا نثبتها بمثال واحد واضح، وخد **5 دقائق** تشوف مكان كل Token بالضبط.

افترض إن عندنا تطبيق اسمه `MyShop`.

### 1. أول Login

المستخدم يبعت:

```http
POST /auth/login

{
  "email": "ahmed@example.com",
  "password": "123456"
}
```

الـ Backend يتأكد من البيانات ويرجع:

```json
{
  "accessToken": "ACCESS_abc123",
  "refreshToken": "REFRESH_xyz789"
}
```

يعني:

```text
Access Token
ACCESS_abc123
        ↓
يستخدم في طلبات الـ API

Refresh Token
REFRESH_xyz789
        ↓
يستخدم لما الـ Access Token ينتهي
```

### 2. المستخدم يعمل Request

مثلاً:

```http
GET /users/me

Authorization: Bearer ACCESS_abc123
```

الـ Backend يقول:

```text
Access Token صحيح ✅
        ↓
200 OK
```

---

### 3. بعد 15 دقيقة مثلاً

الـ `Access Token` انتهى:

```text
ACCESS_abc123 ❌ Expired
```

المستخدم يعمل Request تاني:

```http
GET /users/me

Authorization: Bearer ACCESS_abc123
```

الـ Backend يرد:

```http
401 Unauthorized
```

هنا الـ `Refresh Token` يدخل في الموضوع.

---

### 4. نستخدم الـ Refresh Token

الـ Frontend يبعت:

```http
POST /auth/refresh

{
  "refreshToken": "REFRESH_xyz789"
}
```

الـ Backend يتأكد إن الـ Refresh Token صالح، ويرجع:

```json
{
  "accessToken": "ACCESS_new456"
}
```

لاحظ هنا:

```text
Old Access Token
ACCESS_abc123 ❌

        ↓ Refresh Token

New Access Token
ACCESS_new456 ✅
```

---

### 5. نعيد الطلب الأصلي

الطلب اللي فشل بسبب انتهاء الـ Token:

```http
GET /users/me

Authorization: Bearer ACCESS_new456
```

والـ Backend:

```text
ACCESS_new456 ✅
        ↓
200 OK
```

### الخلاصة المهمة جداً

احفظ العلاقة دي:

```text
Access Token
    ↓
أستخدمه للوصول للـ API

Refresh Token
    ↓
أستخدمه للحصول على Access Token جديد
```

والـ **Refresh Token مش بيتبعت مع كل Request**.

الـ Access Token هو اللي بيتبعت مع طلبات الـ API العادية.

**الخطوة التالية:** نطبق السيناريو ده داخل `Response Interceptor` ونشوف إزاي الـ `401` يشغل عملية الـ refresh.
</details>
ممكن مثال علي تكن وفين القريش وفين التاني
ChatGPT
خلينا نثبتها بمثال واحد واضح، وخد **5 دقائق** تشوف مكان كل Token بالضبط.

افترض إن عندنا تطبيق اسمه `MyShop`.

### 1. أول Login

المستخدم يبعت:

```http
POST /auth/login

{
  "email": "ahmed@example.com",
  "password": "123456"
}
```

الـ Backend يتأكد من البيانات ويرجع:

```json
{
  "accessToken": "ACCESS_abc123",
  "refreshToken": "REFRESH_xyz789"
}
```

يعني:

```text
Access Token
ACCESS_abc123
        ↓
يستخدم في طلبات الـ API

Refresh Token
REFRESH_xyz789
        ↓
يستخدم لما الـ Access Token ينتهي
```

### 2. المستخدم يعمل Request

مثلاً:

```http
GET /users/me

Authorization: Bearer ACCESS_abc123
```

الـ Backend يقول:

```text
Access Token صحيح ✅
        ↓
200 OK
```

---

### 3. بعد 15 دقيقة مثلاً

الـ `Access Token` انتهى:

```text
ACCESS_abc123 ❌ Expired
```

المستخدم يعمل Request تاني:

```http
GET /users/me

Authorization: Bearer ACCESS_abc123
```

الـ Backend يرد:

```http
401 Unauthorized
```

هنا الـ `Refresh Token` يدخل في الموضوع.

---

### 4. نستخدم الـ Refresh Token

الـ Frontend يبعت:

```http
POST /auth/refresh

{
  "refreshToken": "REFRESH_xyz789"
}
```

الـ Backend يتأكد إن الـ Refresh Token صالح، ويرجع:

```json
{
  "accessToken": "ACCESS_new456"
}
```

لاحظ هنا:

```text
Old Access Token
ACCESS_abc123 ❌

        ↓ Refresh Token

New Access Token
ACCESS_new456 ✅
```

---

### 5. نعيد الطلب الأصلي

الطلب اللي فشل بسبب انتهاء الـ Token:

```http
GET /users/me

Authorization: Bearer ACCESS_new456
```

والـ Backend:

```text
ACCESS_new456 ✅
        ↓
200 OK
```

### الخلاصة المهمة جداً

احفظ العلاقة دي:

```text
Access Token
    ↓
أستخدمه للوصول للـ API

Refresh Token
    ↓
أستخدمه للحصول على Access Token جديد
```

والـ **Refresh Token مش بيتبعت مع كل Request**.

الـ Access Token هو اللي بيتبعت مع طلبات الـ API العادية.

**الخطوة التالية:** نطبق السيناريو ده داخل `Response Interceptor` ونشوف إزاي الـ `401` يشغل عملية الـ refresh.
<details>
<summary>تمام</summary>
user
تمام
ChatGPT
ابدأ معايا **4.5 401 Handling** في حوالي **10 دقائق**، وهنبنيها على المثال اللي فوق.

### 🗺️ Roadmap

```text
SECTION 4: Interceptors 🟢
├─ 4.1 What is Interceptor       ✅
├─ 4.2 Request Interceptor       ✅
├─ 4.3 Response Interceptor      ✅
├─ 4.4 Authentication            ✅
├─ 4.5 401 Handling              🟡 ← إحنا هنا
└─ 4.6 Token Refresh + Queue     ⬜
```
</details>
تمام
ChatGPT
ابدأ معايا **4.5 401 Handling** في حوالي **10 دقائق**، وهنبنيها على المثال اللي فوق.

### 🗺️ Roadmap

```text
SECTION 4: Interceptors 🟢
├─ 4.1 What is Interceptor       ✅
├─ 4.2 Request Interceptor       ✅
├─ 4.3 Response Interceptor      ✅
├─ 4.4 Authentication            ✅
├─ 4.5 401 Handling              🟡 ← إحنا هنا
└─ 4.6 Token Refresh + Queue     ⬜
```