ChatGPT
نكمل **الـ Response Interceptor في 15 دقيقة**، وهنبنيه سطر سطر من الكود اللي فهمته بالفعل.

### 🗺️ Roadmap

```text
SECTION 4: Interceptors 🟢
├─ 4.1 What is Interceptor       ✅
├─ 4.2 Request Interceptor       ✅
├─ 4.3 Response Interceptor      ✅
├─ 4.4 Authentication            ✅
├─ 4.5 401 Handling              ✅
└─ 4.6 Token Refresh + Queue     🟡
     ├─ Token Refresh             ✅
     ├─ error.config              ✅
     ├─ Retry                     ✅
     ├─ Infinite Loop Protection   ✅
     └─ Queue                     🟡 ← هنركبها
```

## 1. نبدأ بالـ Variable

```ts id="w0f2qk"
let refreshPromise = null;
```

في البداية:

```text id="z5v9kc"
refreshPromise = null
```

يعني:

> مفيش Refresh شغال.

---

## 2. الـ Interceptor

```ts id="q7x4nm"
api.interceptors.response.use(
  (response) => {
    return response;
  },

  async (error) => {
```

لحد هنا إحنا عارفينها:

```text
Success → return response

Error → async (error)
```

---

## 3. نتأكد إن الخطأ 401

```ts id="b6k1zr"
if (error.response?.status !== 401) {
  return Promise.reject(error);
}
```

يعني:

```text
مش 401
 ↓
مش مشكلتنا
 ↓
سيب الـ Error يكمل
```

لكن لو:

```text id="h8s2pc"
401
 ↓
نكمل
```

---

## 4. نجيب الـ Request الأصلي

```ts id="n4v7ya"
const originalRequest = error.config;
```

يعني:

```text
error
 ↓
config
 ↓
Original Request
```

مثلاً:

```text id="q0k8wb"
originalRequest
├── url: "/users/me"
├── method: "get"
└── headers
```

---

## 5. نحمي نفسنا من الـ Loop

```ts id="p3r6xw"
if (originalRequest._retry) {
  return Promise.reject(error);
}

originalRequest._retry = true;
```

يعني لو الطلب حاول Refresh قبل كده:

```text
_retry = true
 ↓
❌ متعملش Refresh تاني
```

---

# 6. هنا بقى قلب الموضوع

```ts id="v9k2mc"
if (!refreshPromise) {
  refreshPromise = refreshAccessToken();
}
```

لو:

```text id="a1q8zd"
refreshPromise = null
```

نعمل:

```ts id="x3m7pf"
refreshPromise = refreshAccessToken();
```

يعني:

> ابدأ Refresh وخلي العملية موجودة في `refreshPromise`.

---

## 7. دلوقتي أي Request تاني

لو Request B حصل له `401`:

هيلاقي:

```text id="n8c4qy"
refreshPromise ≠ null
```

يعني فيه Refresh شغال.

فيعمل:

```ts id="j6w1ks"
const newToken = await refreshPromise;
```

معناها:

> استنى نفس عملية الـ Refresh.

---

## 8. بعد ما يخلص

```ts id="c5r9vd"
const newToken = await refreshPromise;
```

دلوقتي `newToken` بقى:

```text id="g7p2lm"
NEW_ACCESS_TOKEN
```

فنحطه في الـ Request الأصلي:

```ts id="y4k8zn"
originalRequest.headers.Authorization =
  `Bearer ${newToken}`;
```

وبعدين:

```ts id="s2v6qx"
return api(originalRequest);
```

يعني:

> أعد تنفيذ الـ Request اللي فشل بالـ Token الجديد.

---

# 9. لكن نسينا حاجة

بعد انتهاء الـ Refresh لازم نقول:

```ts id="e8n3wa"
refreshPromise = null;
```

عشان لو حصل `401` في المستقبل، نقدر نبدأ Refresh جديد.

نحطها في:

```ts id="p5q7mx"
finally {
  refreshPromise = null;
}
```

---

# 10. الصورة كلها

```text id="k4m8qz"
Request A
   ↓
  401
   ↓
refreshPromise = null
   ↓
ابدأ Refresh
   ↓
refreshPromise
   ↓
NEW TOKEN
   ↓
Retry A


Request B
   ↓
  401
   ↓
refreshPromise موجود
   ↓
await refreshPromise
   ↓
NEW TOKEN
   ↓
Retry B
```

**دي هي الفكرة اللي كنا بنحاول نوصل لها من الأول.**

لكن خد بالك: الكود اللي فوق **نسخة تعليمية**. في التطبيق الحقيقي هنحتاج نتعامل مع تفاصيل إضافية، أهمها إن طلب `/auth/refresh` نفسه ما يدخلش في نفس دورة الـ Refresh، وكمان التعامل الصحيح مع فشل الـ Refresh.

**الخطوة التالية:** قبل ما نزود أي كود، اكتبلي أنت الجزء ده بنفسك من الذاكرة: `if (!refreshPromise) { ... }`، وأنا أصححه لك.
<details>
<summary>if (!refreshPromise) {</summary>
user
if (!refreshPromise) {
  refreshPromise = refreshAccessToken();
}
نقلتها
ChatGPT
صح جدًا، **السطرين دول بالضبط**. خُد **5 دقائق** ونثبت معناهم قبل ما نضيف أي حاجة.

```ts
if (!refreshPromise) {
  refreshPromise = refreshAccessToken();
}
```

اقرأهم كأنهم جملة واحدة:

> **لو مفيش Refresh شغال، ابدأ Refresh وخلي العملية في `refreshPromise`.**

### أول مرة يحصل `401`

```text
refreshPromise = null
```

إذن:

```ts
if (!refreshPromise) // true
```

فننفذ:

```ts
refreshPromise = refreshAccessToken();
```

فتصبح:

```text
refreshPromise
      ↓
Refresh شغال
```

### لو Request تاني حصل له `401`

هيلاقي:

```text
refreshPromise
      ↓
فيه عملية موجودة بالفعل
```

إذن:

```ts
if (!refreshPromise)
```

تطلع `false`.

**فمش هيعمل Refresh تاني.**

وده هو الجزء اللي أنت فهمته:

```text
A → 401 → يبدأ Refresh
B → 401 → يستنى نفس Refresh
C → 401 → يستنى نفس Refresh
```

بس لسه ناقصنا السطر اللي يخلي **B و C يستنوا فعليًا**.

وهو:

```ts
const newToken = await refreshPromise;
```

**الخطوة التالية:** اكتب السطر ده بنفسك تحت الـ `if`، وبعدها هنشوف بالضبط ليه بيخلي الـ Request التاني يستنى.
</details>
if (!refreshPromise) {
  refreshPromise = refreshAccessToken();
}
نقلتها
ChatGPT
صح جدًا، **السطرين دول بالضبط**. خُد **5 دقائق** ونثبت معناهم قبل ما نضيف أي حاجة.

```ts
if (!refreshPromise) {
  refreshPromise = refreshAccessToken();
}
```

اقرأهم كأنهم جملة واحدة:

> **لو مفيش Refresh شغال، ابدأ Refresh وخلي العملية في `refreshPromise`.**

### أول مرة يحصل `401`

```text
refreshPromise = null
```

إذن:

```ts
if (!refreshPromise) // true
```

فننفذ:

```ts
refreshPromise = refreshAccessToken();
```

فتصبح:

```text
refreshPromise
      ↓
Refresh شغال
```

### لو Request تاني حصل له `401`

هيلاقي:

```text
refreshPromise
      ↓
فيه عملية موجودة بالفعل
```

إذن:

```ts
if (!refreshPromise)
```

تطلع `false`.

**فمش هيعمل Refresh تاني.**

وده هو الجزء اللي أنت فهمته:

```text
A → 401 → يبدأ Refresh
B → 401 → يستنى نفس Refresh
C → 401 → يستنى نفس Refresh
```

بس لسه ناقصنا السطر اللي يخلي **B و C يستنوا فعليًا**.

وهو:

```ts
const newToken = await refreshPromise;
```

**الخطوة التالية:** اكتب السطر ده بنفسك تحت الـ `if`، وبعدها هنشوف بالضبط ليه بيخلي الـ Request التاني يستنى.
<details>
<summary>كمل</summary>
user
كمل
ChatGPT
نكمل **السطر اللي بيخلي الـ Requests تستنى فعلًا في 5 دقائق**.

عندنا:

```ts id="y3j8pk"
if (!refreshPromise) {
  refreshPromise = refreshAccessToken();
}
```

أول Request دخل:

```text id="q1f8vz"
A → 401
   ↓
مفيش Refresh
   ↓
ابدأ Refresh
   ↓
refreshPromise
```

دلوقتي Request B دخل وحصل له `401`.

هيعدي على نفس الـ `if`، لكن `refreshPromise` موجود بالفعل.

إذن **مش هيبدأ Refresh جديد**.

بعدها نكتب:

```ts id="t5r2xn"
const newToken = await refreshPromise;
```

معناها:

> يا Request B، استنى الـ Refresh الموجود في `refreshPromise`.

فتبقى:

```text id="c7m4qa"
A → 401
     ↓
Refresh بدأ
     ↓
refreshPromise ⏳
     
B → 401
     ↓
await refreshPromise
     ↓
انتظار ⏳
```

لما الـ Refresh يخلص:

```text id="v9n2kd"
refreshPromise
      ↓
NEW_ACCESS_TOKEN
```

ساعتها:

```ts id="x6p3mw"
newToken
```

يصبح:

```text id="j8q4zs"
NEW_ACCESS_TOKEN
```

---

### وبعدها نعمل إيه؟

نرجع للـ Request الأصلي:

```ts id="m3w7yc"
originalRequest.headers.Authorization =
  `Bearer ${newToken}`;
```

وبعدين:

```ts id="e4k9tp"
return api(originalRequest);
```

يعني:

```text id="n7s2qm"
401
 ↓
Refresh
 ↓
New Token
 ↓
ضع الـ Token الجديد في Original Request
 ↓
Retry
 ↓
200 ✅
```

### إذن الجزء ده كله:

```ts id="w6k1rx"
if (!refreshPromise) {
  refreshPromise = refreshAccessToken();
}

const newToken = await refreshPromise;

originalRequest.headers.Authorization =
  `Bearer ${newToken}`;

return api(originalRequest);
```

اقرأه كقصة:

> **لو مفيش Refresh ابدأه. بعد كده استنى الـ Refresh. لما يطلع الـ Token الجديد، حطه في الطلب الأصلي، وأعد الطلب.**

وده أهم جزء في الـ `Token Refresh`.

**الخطوة التالية:** هنكتب الكود كامل من أول `401` لحد `200`، وأنت هتشرحلي كل جزء فيه بكلامك.
</details>
كمل
ChatGPT
نكمل **السطر اللي بيخلي الـ Requests تستنى فعلًا في 5 دقائق**.

عندنا:

```ts id="y3j8pk"
if (!refreshPromise) {
  refreshPromise = refreshAccessToken();
}
```

أول Request دخل:

```text id="q1f8vz"
A → 401
   ↓
مفيش Refresh
   ↓
ابدأ Refresh
   ↓
refreshPromise
```

دلوقتي Request B دخل وحصل له `401`.

هيعدي على نفس الـ `if`، لكن `refreshPromise` موجود بالفعل.

إذن **مش هيبدأ Refresh جديد**.

بعدها نكتب:

```ts id="t5r2xn"
const newToken = await refreshPromise;
```

معناها:

> يا Request B، استنى الـ Refresh الموجود في `refreshPromise`.

فتبقى:

```text id="c7m4qa"
A → 401
     ↓
Refresh بدأ
     ↓
refreshPromise ⏳
     
B → 401
     ↓
await refreshPromise
     ↓
انتظار ⏳
```

لما الـ Refresh يخلص:

```text id="v9n2kd"
refreshPromise
      ↓
NEW_ACCESS_TOKEN
```

ساعتها:

```ts id="x6p3mw"
newToken
```

يصبح:

```text id="j8q4zs"
NEW_ACCESS_TOKEN
```

---

### وبعدها نعمل إيه؟

نرجع للـ Request الأصلي:

```ts id="m3w7yc"
originalRequest.headers.Authorization =
  `Bearer ${newToken}`;
```

وبعدين:

```ts id="e4k9tp"
return api(originalRequest);
```

يعني:

```text id="n7s2qm"
401
 ↓
Refresh
 ↓
New Token
 ↓
ضع الـ Token الجديد في Original Request
 ↓
Retry
 ↓
200 ✅
```

### إذن الجزء ده كله:

```ts id="w6k1rx"
if (!refreshPromise) {
  refreshPromise = refreshAccessToken();
}

const newToken = await refreshPromise;

originalRequest.headers.Authorization =
  `Bearer ${newToken}`;

return api(originalRequest);
```

اقرأه كقصة:

> **لو مفيش Refresh ابدأه. بعد كده استنى الـ Refresh. لما يطلع الـ Token الجديد، حطه في الطلب الأصلي، وأعد الطلب.**

وده أهم جزء في الـ `Token Refresh`.

**الخطوة التالية:** هنكتب الكود كامل من أول `401` لحد `200`، وأنت هتشرحلي كل جزء فيه بكلامك.
<details>
<summary>يلا بينا</summary>
user
يلا بينا
ChatGPT
اكتب الكود كامل معايا **سطر سطر في 10 دقائق**، وبعده هخليك أنت تشرحه.

### 🗺️ Roadmap

```text
SECTION 4: Interceptors 🟢
├─ 4.1 What is Interceptor       ✅
├─ 4.2 Request Interceptor       ✅
├─ 4.3 Response Interceptor      ✅
├─ 4.4 Authentication            ✅
├─ 4.5 401 Handling              ✅
└─ 4.6 Token Refresh + Queue     🟡
     ├─ Refresh Token             ✅
     ├─ error.config              ✅
     ├─ Retry                     ✅
     ├─ Infinite Loop Protection   ✅
     └─ Queue                     🟡
```

## الكود الكامل

```ts id="q4m8vz"
let refreshPromise = null;

api.interceptors.response.use(
  (response) => {
    return response;
  },

  async (error) => {
    // 1. هل الخطأ 401؟
    if (error.response?.status !== 401) {
      return Promise.reject(error);
    }

    // 2. هات الـ Request الأصلي
    const originalRequest = error.config;

    // 3. امنع Infinite Loop
    if (originalRequest._retry) {
      return Promise.reject(error);
    }

    originalRequest._retry = true;

    // 4. لو مفيش Refresh شغال، ابدأ واحد
    if (!refreshPromise) {
      refreshPromise = refreshAccessToken();
    }

    // 5. استنى الـ Refresh
    const newToken = await refreshPromise;

    // 6. حط الـ Token الجديد في الـ Request الأصلي
    originalRequest.headers.Authorization =
      `Bearer ${newToken}`;

    // 7. أعد الـ Request الأصلي
    return api(originalRequest);
  }
);
```

## دلوقتي نمشي عليه كرحلة واحدة

Request:

```text id="a8n3kc"
GET /users/me
```

الـ Request Interceptor حط:

```text id="e7p2mz"
Authorization: Bearer OLD_TOKEN
```

الـ Backend قال:

```text id="c4v9xa"
401
```

دخلنا هنا:

```ts id="u5k2re"
if (error.response?.status !== 401)
```

الشرط `false` لأن الخطأ فعلًا `401`.

نكمل.

---

### نجيب الطلب الأصلي

```ts id="m6z1qw"
const originalRequest = error.config;
```

يعني:

```text id="n9c4yb"
originalRequest
↓
GET /users/me
↓
OLD_TOKEN
```

---

### نمنع الـ Loop

```ts id="p8x3kd"
if (originalRequest._retry) {
  return Promise.reject(error);
}

originalRequest._retry = true;
```

أول مرة:

```text id="v2m7qa"
_retry = undefined
```

نكمل.

ولو حاول نفس الطلب يدخل دورة تانية:

```text id="r6k1wp"
_retry = true
```

نرفضه.

---

### نبدأ Refresh

```ts id="s3f8nc"
if (!refreshPromise) {
  refreshPromise = refreshAccessToken();
}
```

في أول Request:

```text id="b5q9xd"
refreshPromise = null
```

إذن يبدأ Refresh.

---

### ننتظر

```ts id="t7m2vz"
const newToken = await refreshPromise;
```

لما الـ Backend يرجع:

```text id="x4k8pq"
NEW_TOKEN
```

يبقى:

```text id="j6n3wy"
newToken = NEW_TOKEN
```

---

### نعدل الطلب الأصلي

```ts id="c8r5ma"
originalRequest.headers.Authorization =
  `Bearer ${newToken}`;
```

كان:

```text id="v4q7nx"
Bearer OLD_TOKEN
```

أصبح:

```text id="w2m9kc"
Bearer NEW_TOKEN
```

---

### وآخر خطوة

```ts id="z5p1qr"
return api(originalRequest);
```

يعني:

> نفّذ الطلب الأصلي مرة ثانية.

فتصبح الرحلة:

```text id="n3k7vs"
GET /users/me
      ↓
OLD_TOKEN
      ↓
401
      ↓
Refresh
      ↓
NEW_TOKEN
      ↓
تعديل Original Request
      ↓
Retry
      ↓
GET /users/me
      ↓
NEW_TOKEN
      ↓
200 ✅
```

### 🎯 دلوقتي دورك

مش عايزك تحفظ الكود.

عايزك تشرحلي **بالعامية** الرحلة دي من لحظة ما الـ Request ياخد `401` لحد ما يعمل `Retry`.

استخدم كلامك أنت، حتى لو مش مرتب. **الخطوة التالية:** اشرح الرحلة كاملة من `401` إلى `200`.
</details>
يلا بينا
ChatGPT
اكتب الكود كامل معايا **سطر سطر في 10 دقائق**، وبعده هخليك أنت تشرحه.

### 🗺️ Roadmap

```text
SECTION 4: Interceptors 🟢
├─ 4.1 What is Interceptor       ✅
├─ 4.2 Request Interceptor       ✅
├─ 4.3 Response Interceptor      ✅
├─ 4.4 Authentication            ✅
├─ 4.5 401 Handling              ✅
└─ 4.6 Token Refresh + Queue     🟡
     ├─ Refresh Token             ✅
     ├─ error.config              ✅
     ├─ Retry                     ✅
     ├─ Infinite Loop Protection   ✅
     └─ Queue                     🟡
```

## الكود الكامل

```ts id="q4m8vz"
let refreshPromise = null;

api.interceptors.response.use(
  (response) => {
    return response;
  },

  async (error) => {
    // 1. هل الخطأ 401؟
    if (error.response?.status !== 401) {
      return Promise.reject(error);
    }

    // 2. هات الـ Request الأصلي
    const originalRequest = error.config;

    // 3. امنع Infinite Loop
    if (originalRequest._retry) {
      return Promise.reject(error);
    }

    originalRequest._retry = true;

    // 4. لو مفيش Refresh شغال، ابدأ واحد
    if (!refreshPromise) {
      refreshPromise = refreshAccessToken();
    }

    // 5. استنى الـ Refresh
    const newToken = await refreshPromise;

    // 6. حط الـ Token الجديد في الـ Request الأصلي
    originalRequest.headers.Authorization =
      `Bearer ${newToken}`;

    // 7. أعد الـ Request الأصلي
    return api(originalRequest);
  }
);
```

## دلوقتي نمشي عليه كرحلة واحدة

Request:

```text id="a8n3kc"
GET /users/me
```

الـ Request Interceptor حط:

```text id="e7p2mz"
Authorization: Bearer OLD_TOKEN
```

الـ Backend قال:

```text id="c4v9xa"
401
```

دخلنا هنا:

```ts id="u5k2re"
if (error.response?.status !== 401)
```

الشرط `false` لأن الخطأ فعلًا `401`.

نكمل.

---

### نجيب الطلب الأصلي

```ts id="m6z1qw"
const originalRequest = error.config;
```

يعني:

```text id="n9c4yb"
originalRequest
↓
GET /users/me
↓
OLD_TOKEN
```

---

### نمنع الـ Loop

```ts id="p8x3kd"
if (originalRequest._retry) {
  return Promise.reject(error);
}

originalRequest._retry = true;
```

أول مرة:

```text id="v2m7qa"
_retry = undefined
```

نكمل.

ولو حاول نفس الطلب يدخل دورة تانية:

```text id="r6k1wp"
_retry = true
```

نرفضه.

---

### نبدأ Refresh

```ts id="s3f8nc"
if (!refreshPromise) {
  refreshPromise = refreshAccessToken();
}
```

في أول Request:

```text id="b5q9xd"
refreshPromise = null
```

إذن يبدأ Refresh.

---

### ننتظر

```ts id="t7m2vz"
const newToken = await refreshPromise;
```

لما الـ Backend يرجع:

```text id="x4k8pq"
NEW_TOKEN
```

يبقى:

```text id="j6n3wy"
newToken = NEW_TOKEN
```

---

### نعدل الطلب الأصلي

```ts id="c8r5ma"
originalRequest.headers.Authorization =
  `Bearer ${newToken}`;
```

كان:

```text id="v4q7nx"
Bearer OLD_TOKEN
```

أصبح:

```text id="w2m9kc"
Bearer NEW_TOKEN
```

---

### وآخر خطوة

```ts id="z5p1qr"
return api(originalRequest);
```

يعني:

> نفّذ الطلب الأصلي مرة ثانية.

فتصبح الرحلة:

```text id="n3k7vs"
GET /users/me
      ↓
OLD_TOKEN
      ↓
401
      ↓
Refresh
      ↓
NEW_TOKEN
      ↓
تعديل Original Request
      ↓
Retry
      ↓
GET /users/me
      ↓
NEW_TOKEN
      ↓
200 ✅
```

### 🎯 دلوقتي دورك

مش عايزك تحفظ الكود.

عايزك تشرحلي **بالعامية** الرحلة دي من لحظة ما الـ Request ياخد `401` لحد ما يعمل `Retry`.

استخدم كلامك أنت، حتى لو مش مرتب. **الخطوة التالية:** اشرح الرحلة كاملة من `401` إلى `200`.
<details>
<summary>بص يا باشا، إحنا أول حاجة هيطلع الخط ده لنا بـ401. 401 دي معناها Unauthorized بـ401. دي كده مشكلة هتحصل عندنا إن access token مش متولد. في الحالة دي هعمل إيه؟ هعمل في الـinterceptor، هو باشا أخلي الـpromise refresh بـnull. أخلي الـpromise refresh إيه؟ هو refresh promise by null. بعد كده هخش بقى response، نرجع الـresponse عادي، أخش في الـerror. هنقول: When I think. أول حاجة هنشوف هو الخط ده 401 ولا لأ. لو 401 هنكمل في مشوارنا. طب لو هو غير كده، ساعتها نرجع له promise rejected error عشان نروح للـcatch. طيب الحالة التانية إحنا هنجيب الـrequest، هنجيب الـrequest الأصلي اللي هو original request، ده عبارة عن error.config. ده اللي بنعمل ده عشان نعمل حتة الـevent loop. هنقول له لو .retry، تلعب retry، يبقى برضه أعمل إيه؟ promise rejected. خلاص أنا مش هعمل كل شوية مرة، مش هعمل event loop. بعد كده هنخش في حاجة اسمها refresh، هو شغال دلوقتي ولا لأ. فقول له لو not refresh promise، اعمل refresh promise واديني function اسمها refresh access token عشان أجيب الـtoken جديد، ويقدر يرجعلي token جديد وأحطه جوه refresh promise. ساعتها أقول له إن الـtoken ده هيساوي الـrefresh promise بس أساميه عشان يستناه، ياخد إيه؟ ياخد كمان. بعد كده أخد الـtoken ده أحطه في الـrequest الأصلي original request.headers.authorization، أحط الـnew token. بعد ما عملت كده أبدأ أرجع الـrequest الأصلي، أقول له return API original request. بس الكود بصراحة مش هعرف أكتبه لوحدي.</summary>
user
بص يا باشا، إحنا أول حاجة هيطلع الخط ده لنا بـ401. 401 دي معناها Unauthorized بـ401. دي كده مشكلة هتحصل عندنا إن access token مش متولد. في الحالة دي هعمل إيه؟ هعمل في الـinterceptor، هو باشا أخلي الـpromise refresh بـnull. أخلي الـpromise refresh إيه؟ هو refresh promise by null. بعد كده هخش بقى response، نرجع الـresponse عادي، أخش في الـerror. هنقول: When I think. أول حاجة هنشوف هو الخط ده 401 ولا لأ. لو 401 هنكمل في مشوارنا. طب لو هو غير كده، ساعتها نرجع له promise rejected error عشان نروح للـcatch. طيب الحالة التانية إحنا هنجيب الـrequest، هنجيب الـrequest الأصلي اللي هو original request، ده عبارة عن error.config. ده اللي بنعمل ده عشان نعمل حتة الـevent loop. هنقول له لو .retry، تلعب retry، يبقى برضه أعمل إيه؟ promise rejected. خلاص أنا مش هعمل كل شوية مرة، مش هعمل event loop. بعد كده هنخش في حاجة اسمها refresh، هو شغال دلوقتي ولا لأ. فقول له لو not refresh promise، اعمل refresh promise واديني function اسمها refresh access token عشان أجيب الـtoken جديد، ويقدر يرجعلي token جديد وأحطه جوه refresh promise. ساعتها أقول له إن الـtoken ده هيساوي الـrefresh promise بس أساميه عشان يستناه، ياخد إيه؟ ياخد كمان. بعد كده أخد الـtoken ده أحطه في الـrequest الأصلي original request.headers.authorization، أحط الـnew token. بعد ما عملت كده أبدأ أرجع الـrequest الأصلي، أقول له return API original request. بس الكود بصراحة مش هعرف أكتبه لوحدي.
ChatGPT
أنت **فاهم الـ Flow بنسبة كبيرة جدًا**، والمشكلة عندك حاليًا في **ترجمة الفكرة إلى Syntax**، مش في الـ Concept. وده طبيعي جدًا.

بس أصحح لك **3 نقاط** فقط:

1. `401` مش معناها بالضرورة إن الـ Access Token "مش متولد"، الأدق إنه **غير صالح أو مفقود أو منتهي**.
2. `refreshPromise = null` بنعملها عشان نقول **مفيش Refresh شغال حاليًا**.
3. الـ `Retry` مش عشان نمنع Event Loop تحديدًا، إحنا بنمنع **Infinite Refresh Loop**.

### دلوقتي خلينا نكتب الكود من كلامك أنت

ابدأ بالجزء ده:

```ts
let refreshPromise = null;
```

معناه:

```text
مفيش Refresh شغال حاليًا
```

بعدها:

```ts
api.interceptors.response.use(
  (response) => {
    return response;
  },

  async (error) => {
```

وبعدين أول قرار:

```ts
if (error.response?.status !== 401) {
  return Promise.reject(error);
}
```

يعني:

```text
هل 401؟
   ↓
نعم → كمل
   ↓
لا → ارفض الـ Error وروحه للـ catch
```

بعدها:

```ts
const originalRequest = error.config;
```

وده أنت فهمته صح جدًا.

ثم:

```ts
if (originalRequest._retry) {
  return Promise.reject(error);
}

originalRequest._retry = true;
```

يعني:

```text
هل جربنا الطلب ده قبل كده بعد Refresh؟
        ↓
      نعم
        ↓
   متعملش Refresh تاني
```

ثم الجزء اللي أنت فاهمه:

```ts
if (!refreshPromise) {
  refreshPromise = refreshAccessToken();
}
```

معناه:

```text
هل فيه Refresh شغال؟
      ↓
     لا
      ↓
ابدأ Refresh
```

وبعدين:

```ts
const newToken = await refreshPromise;
```

ودي أهم نقطة أنت وصلت لها:

```text
refreshPromise
      ↓
استنى
      ↓
Refresh يخلص
      ↓
New Token
      ↓
newToken
```

ثم:

```ts
originalRequest.headers.Authorization =
  `Bearer ${newToken}`;
```

ثم:

```ts
return api(originalRequest);
```

فتصبح الرحلة:

```text
401
 ↓
Original Request
 ↓
هل عملنا Retry قبل كده؟
 ↓
لا
 ↓
هل Refresh شغال؟
 ↓
لا
 ↓
ابدأ Refresh
 ↓
استنى
 ↓
New Token
 ↓
ضعه في Original Request
 ↓
Retry
 ↓
200 ✅
```

### والكود النهائي

```ts
let refreshPromise = null;

api.interceptors.response.use(
  (response) => {
    return response;
  },

  async (error) => {
    if (error.response?.status !== 401) {
      return Promise.reject(error);
    }

    const originalRequest = error.config;

    if (originalRequest._retry) {
      return Promise.reject(error);
    }

    originalRequest._retry = true;

    if (!refreshPromise) {
      refreshPromise = refreshAccessToken();
    }

    const newToken = await refreshPromise;

    originalRequest.headers.Authorization =
      `Bearer ${newToken}`;

    return api(originalRequest);
  }
);
```

**أنت مش مطلوب منك تكتب الكود ده من الذاكرة دلوقتي.** المطلوب حاليًا إنك تبقى قادر تقرأه وتقول وظيفة كل جزء، وبعدها الكتابة هتيجي بالتدريب.

**الخطوة التالية:** هديك نفس الكود ناقص منه 5 أسطر فقط، وأنت تملأهم بنفسك.
</details>
بص يا باشا، إحنا أول حاجة هيطلع الخط ده لنا بـ401. 401 دي معناها Unauthorized بـ401. دي كده مشكلة هتحصل عندنا إن access token مش متولد. في الحالة دي هعمل إيه؟ هعمل في الـinterceptor، هو باشا أخلي الـpromise refresh بـnull. أخلي الـpromise refresh إيه؟ هو refresh promise by null. بعد كده هخش بقى response، نرجع الـresponse عادي، أخش في الـerror. هنقول: When I think. أول حاجة هنشوف هو الخط ده 401 ولا لأ. لو 401 هنكمل في مشوارنا. طب لو هو غير كده، ساعتها نرجع له promise rejected error عشان نروح للـcatch. طيب الحالة التانية إحنا هنجيب الـrequest، هنجيب الـrequest الأصلي اللي هو original request، ده عبارة عن error.config. ده اللي بنعمل ده عشان نعمل حتة الـevent loop. هنقول له لو .retry، تلعب retry، يبقى برضه أعمل إيه؟ promise rejected. خلاص أنا مش هعمل كل شوية مرة، مش هعمل event loop. بعد كده هنخش في حاجة اسمها refresh، هو شغال دلوقتي ولا لأ. فقول له لو not refresh promise، اعمل refresh promise واديني function اسمها refresh access token عشان أجيب الـtoken جديد، ويقدر يرجعلي token جديد وأحطه جوه refresh promise. ساعتها أقول له إن الـtoken ده هيساوي الـrefresh promise بس أساميه عشان يستناه، ياخد إيه؟ ياخد كمان. بعد كده أخد الـtoken ده أحطه في الـrequest الأصلي original request.headers.authorization، أحط الـnew token. بعد ما عملت كده أبدأ أرجع الـrequest الأصلي، أقول له return API original request. بس الكود بصراحة مش هعرف أكتبه لوحدي.
ChatGPT
أنت **فاهم الـ Flow بنسبة كبيرة جدًا**، والمشكلة عندك حاليًا في **ترجمة الفكرة إلى Syntax**، مش في الـ Concept. وده طبيعي جدًا.

بس أصحح لك **3 نقاط** فقط:

1. `401` مش معناها بالضرورة إن الـ Access Token "مش متولد"، الأدق إنه **غير صالح أو مفقود أو منتهي**.
2. `refreshPromise = null` بنعملها عشان نقول **مفيش Refresh شغال حاليًا**.
3. الـ `Retry` مش عشان نمنع Event Loop تحديدًا، إحنا بنمنع **Infinite Refresh Loop**.

### دلوقتي خلينا نكتب الكود من كلامك أنت

ابدأ بالجزء ده:

```ts
let refreshPromise = null;
```

معناه:

```text
مفيش Refresh شغال حاليًا
```

بعدها:

```ts
api.interceptors.response.use(
  (response) => {
    return response;
  },

  async (error) => {
```

وبعدين أول قرار:

```ts
if (error.response?.status !== 401) {
  return Promise.reject(error);
}
```

يعني:

```text
هل 401؟
   ↓
نعم → كمل
   ↓
لا → ارفض الـ Error وروحه للـ catch
```

بعدها:

```ts
const originalRequest = error.config;
```

وده أنت فهمته صح جدًا.

ثم:

```ts
if (originalRequest._retry) {
  return Promise.reject(error);
}

originalRequest._retry = true;
```

يعني:

```text
هل جربنا الطلب ده قبل كده بعد Refresh؟
        ↓
      نعم
        ↓
   متعملش Refresh تاني
```

ثم الجزء اللي أنت فاهمه:

```ts
if (!refreshPromise) {
  refreshPromise = refreshAccessToken();
}
```

معناه:

```text
هل فيه Refresh شغال؟
      ↓
     لا
      ↓
ابدأ Refresh
```

وبعدين:

```ts
const newToken = await refreshPromise;
```

ودي أهم نقطة أنت وصلت لها:

```text
refreshPromise
      ↓
استنى
      ↓
Refresh يخلص
      ↓
New Token
      ↓
newToken
```

ثم:

```ts
originalRequest.headers.Authorization =
  `Bearer ${newToken}`;
```

ثم:

```ts
return api(originalRequest);
```

فتصبح الرحلة:

```text
401
 ↓
Original Request
 ↓
هل عملنا Retry قبل كده؟
 ↓
لا
 ↓
هل Refresh شغال؟
 ↓
لا
 ↓
ابدأ Refresh
 ↓
استنى
 ↓
New Token
 ↓
ضعه في Original Request
 ↓
Retry
 ↓
200 ✅
```

### والكود النهائي

```ts
let refreshPromise = null;

api.interceptors.response.use(
  (response) => {
    return response;
  },

  async (error) => {
    if (error.response?.status !== 401) {
      return Promise.reject(error);
    }

    const originalRequest = error.config;

    if (originalRequest._retry) {
      return Promise.reject(error);
    }

    originalRequest._retry = true;

    if (!refreshPromise) {
      refreshPromise = refreshAccessToken();
    }

    const newToken = await refreshPromise;

    originalRequest.headers.Authorization =
      `Bearer ${newToken}`;

    return api(originalRequest);
  }
);
```

**أنت مش مطلوب منك تكتب الكود ده من الذاكرة دلوقتي.** المطلوب حاليًا إنك تبقى قادر تقرأه وتقول وظيفة كل جزء، وبعدها الكتابة هتيجي بالتدريب.

**الخطوة التالية:** هديك نفس الكود ناقص منه 5 أسطر فقط، وأنت تملأهم بنفسك.
<details>
<summary>تمام</summary>
user
تمام
ChatGPT
املأ **الـ 5 فراغات دول في 10 دقائق**، ومتفتحش الكود القديم. الهدف دلوقتي تحويل الفهم إلى كتابة.

```ts
let refreshPromise = null;

api.interceptors.response.use(
  (response) => {
    return response;
  },

  async (error) => {

    // 1
    if (______________________________) {
      return Promise.reject(error);
    }

    // 2
    const originalRequest = __________________;

    // 3
    if (______________________________) {
      return Promise.reject(error);
    }

    originalRequest._retry = true;

    // 4
    if (!______________________________) {
      refreshPromise = __________________;
    }

    // 5
    const newToken = __________________________;

    originalRequest.headers.Authorization =
      `Bearer ${newToken}`;

    return api(originalRequest);
  }
);
```

### المطلوب منك

املأهم فقط:

```text
1 = ؟
2 = ؟
3 = ؟
4 = ؟
5 = ؟
```

**الخطوة التالية:** ابعت الخمس إجابات حتى لو مش متأكد منها، وأنا أصححهم واحدة واحدة.
</details>
تمام
ChatGPT
املأ **الـ 5 فراغات دول في 10 دقائق**، ومتفتحش الكود القديم. الهدف دلوقتي تحويل الفهم إلى كتابة.

```ts
let refreshPromise = null;

api.interceptors.response.use(
  (response) => {
    return response;
  },

  async (error) => {

    // 1
    if (______________________________) {
      return Promise.reject(error);
    }

    // 2
    const originalRequest = __________________;

    // 3
    if (______________________________) {
      return Promise.reject(error);
    }

    originalRequest._retry = true;

    // 4
    if (!______________________________) {
      refreshPromise = __________________;
    }

    // 5
    const newToken = __________________________;

    originalRequest.headers.Authorization =
      `Bearer ${newToken}`;

    return api(originalRequest);
  }
);
```

### المطلوب منك

املأهم فقط:

```text
1 = ؟
2 = ؟
3 = ؟
4 = ؟
5 = ؟
```

**الخطوة التالية:** ابعت الخمس إجابات حتى لو مش متأكد منها، وأنا أصححهم واحدة واحدة.
<details>
<summary>1</summary>
user
1
erorr.response.stutas!==401
2
error.config
3
originalRequest._retry
4
(!refreshPromise)
refreshPromise= refrashaccesstoken()
5
await refreshPromise
ChatGPT
ممتاز يا باشا. **أنت كده فعليًا كتبت الـ Flow كله من دماغك**. النتيجة: **5/5 في الـ Concept**، وفيه بس تصحيح Syntax بسيط.

### التصحيح

**1 ✅**

أنت كتبت:

```ts
error.response.status !== 401
```

والأفضل:

```ts
error.response?.status !== 401
```

الـ `?.` عشان `response` ممكن تكون `undefined`.

---

**2 ✅**

```ts
const originalRequest = error.config;
```

صح 100%.

---

**3 ✅**

أنت كتبت:

```ts
originalRequest._retry
```

وده صحيح.

الكود:

```ts
if (originalRequest._retry) {
  return Promise.reject(error);
}
```

---

**4 ✅**

أنت فهمت الجزئية دي تمام:

```ts
if (!refreshPromise) {
  refreshPromise = refreshAccessToken();
}
```

وده أهم جزء في الموضوع كله.

---

**5 ✅**

```ts
const newToken = await refreshPromise;
```

صح.

وده معناه:

```text id="q4w8k2"
refreshPromise
      ↓
     await
      ↓
استنى الـ Refresh
      ↓
New Token
      ↓
newToken
```

### النتيجة

```text id="f7k2m9"
401                    ✅
error.config           ✅
_retry                  ✅
refreshPromise          ✅
await refreshPromise    ✅
```

**أنت مش محتاج تحفظ الكود. أنت محتاج تكرر كتابته مرتين أو ثلاثة وهتلاقي إيدك بدأت تكتبه لوحدها.**

وفي نقطة مهمة جدًا: الكود اللي بنكتبه لسه **نسخة تعليمية**. قبل ما نعتبر `4.6` مكتملة، لازم نصلح موضوع `refreshPromise = null` بعد انتهاء الـ Refresh، ونتعامل مع فشل الـ Refresh، وبعدها نعمل **اختبار نهائي للـ Queue**.

**الخطوة التالية:** نضيف `try/finally` للـ `refreshPromise` ونفهم ليه لازم نرجعه إلى `null` بعد انتهاء الـ Refresh.
</details>
1
erorr.response.stutas!==401
2
error.config
3
originalRequest._retry
4
(!refreshPromise)
refreshPromise= refrashaccesstoken()
5
await refreshPromise
ChatGPT
ممتاز يا باشا. **أنت كده فعليًا كتبت الـ Flow كله من دماغك**. النتيجة: **5/5 في الـ Concept**، وفيه بس تصحيح Syntax بسيط.

### التصحيح

**1 ✅**

أنت كتبت:

```ts
error.response.status !== 401
```

والأفضل:

```ts
error.response?.status !== 401
```

الـ `?.` عشان `response` ممكن تكون `undefined`.

---

**2 ✅**

```ts
const originalRequest = error.config;
```

صح 100%.

---

**3 ✅**

أنت كتبت:

```ts
originalRequest._retry
```

وده صحيح.

الكود:

```ts
if (originalRequest._retry) {
  return Promise.reject(error);
}
```

---

**4 ✅**

أنت فهمت الجزئية دي تمام:

```ts
if (!refreshPromise) {
  refreshPromise = refreshAccessToken();
}
```

وده أهم جزء في الموضوع كله.

---

**5 ✅**

```ts
const newToken = await refreshPromise;
```

صح.

وده معناه:

```text id="q4w8k2"
refreshPromise
      ↓
     await
      ↓
استنى الـ Refresh
      ↓
New Token
      ↓
newToken
```

### النتيجة

```text id="f7k2m9"
401                    ✅
error.config           ✅
_retry                  ✅
refreshPromise          ✅
await refreshPromise    ✅
```

**أنت مش محتاج تحفظ الكود. أنت محتاج تكرر كتابته مرتين أو ثلاثة وهتلاقي إيدك بدأت تكتبه لوحدها.**

وفي نقطة مهمة جدًا: الكود اللي بنكتبه لسه **نسخة تعليمية**. قبل ما نعتبر `4.6` مكتملة، لازم نصلح موضوع `refreshPromise = null` بعد انتهاء الـ Refresh، ونتعامل مع فشل الـ Refresh، وبعدها نعمل **اختبار نهائي للـ Queue**.

**الخطوة التالية:** نضيف `try/finally` للـ `refreshPromise` ونفهم ليه لازم نرجعه إلى `null` بعد انتهاء الـ Refresh.
<details>
<summary>كمل</summary>
user
كمل
ChatGPT
أضف `finally` دلوقتي في **5 دقائق**، لأن دي آخر قطعة أساسية في دورة حياة `refreshPromise`.

### 🗺️ Roadmap

```text
SECTION 4: Interceptors 🟢
├─ 4.1 What is Interceptor       ✅
├─ 4.2 Request Interceptor       ✅
├─ 4.3 Response Interceptor      ✅
├─ 4.4 Authentication            ✅
├─ 4.5 401 Handling              ✅
└─ 4.6 Token Refresh + Queue     🟡
     ├─ Refresh Token             ✅
     ├─ error.config              ✅
     ├─ Retry                     ✅
     ├─ Infinite Loop Protection   ✅
     ├─ Shared refreshPromise      ✅
     └─ Reset refreshPromise       🟡 ← هنا
```

عندنا حاليًا:

```ts id="a1b2c3"
if (!refreshPromise) {
  refreshPromise = refreshAccessToken();
}

const newToken = await refreshPromise;
```

أول Request يعمل `401`:

```text id="d4e5f6"
refreshPromise = null
       ↓
refreshAccessToken()
       ↓
refreshPromise = عملية Refresh
```

الـ Refresh يخلص، ويطلع:

```text id="g7h8i9"
NEW_ACCESS_TOKEN
```

طيب بعد كده؟

لو سيبنا:

```text id="j1k2l3"
refreshPromise = عملية قديمة
```

فلو حصل `401` جديد بعد فترة، الكود هيشوف:

```ts id="m4n5o6"
if (!refreshPromise)
```

والإجابة هتكون:

```text
false
```

لأن `refreshPromise` لسه ماسك الـ Promise القديم.

وده غلط.

إحنا عايزين بعد انتهاء العملية:

```text id="p7q8r9"
refreshPromise = null
```

يعني:

> الـ Refresh خلص، لو احتجنا Refresh جديد في المستقبل، ابدأ عملية جديدة.
</details>
كمل
ChatGPT
أضف `finally` دلوقتي في **5 دقائق**، لأن دي آخر قطعة أساسية في دورة حياة `refreshPromise`.

### 🗺️ Roadmap

```text
SECTION 4: Interceptors 🟢
├─ 4.1 What is Interceptor       ✅
├─ 4.2 Request Interceptor       ✅
├─ 4.3 Response Interceptor      ✅
├─ 4.4 Authentication            ✅
├─ 4.5 401 Handling              ✅
└─ 4.6 Token Refresh + Queue     🟡
     ├─ Refresh Token             ✅
     ├─ error.config              ✅
     ├─ Retry                     ✅
     ├─ Infinite Loop Protection   ✅
     ├─ Shared refreshPromise      ✅
     └─ Reset refreshPromise       🟡 ← هنا
```

عندنا حاليًا:

```ts id="a1b2c3"
if (!refreshPromise) {
  refreshPromise = refreshAccessToken();
}

const newToken = await refreshPromise;
```

أول Request يعمل `401`:

```text id="d4e5f6"
refreshPromise = null
       ↓
refreshAccessToken()
       ↓
refreshPromise = عملية Refresh
```

الـ Refresh يخلص، ويطلع:

```text id="g7h8i9"
NEW_ACCESS_TOKEN
```

طيب بعد كده؟

لو سيبنا:

```text id="j1k2l3"
refreshPromise = عملية قديمة
```

فلو حصل `401` جديد بعد فترة، الكود هيشوف:

```ts id="m4n5o6"
if (!refreshPromise)
```

والإجابة هتكون:

```text
false
```

لأن `refreshPromise` لسه ماسك الـ Promise القديم.

وده غلط.

إحنا عايزين بعد انتهاء العملية:

```text id="p7q8r9"
refreshPromise = null
```

يعني:

> الـ Refresh خلص، لو احتجنا Refresh جديد في المستقبل، ابدأ عملية جديدة.