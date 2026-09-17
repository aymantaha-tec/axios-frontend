## هنا يأتي `finally`

```ts id="s1t2u3"
try {
  const newToken = await refreshPromise;

  originalRequest.headers.Authorization =
    `Bearer ${newToken}`;

  return api(originalRequest);

} finally {
  refreshPromise = null;
}
```

### ليه `finally`؟

لأن `finally` بتتنفذ **سواء العملية نجحت أو فشلت**.

```text id="v4w5x6"
Refresh
   ↓
┌───────────────┐
│ نجح؟           │
│ أو فشل؟        │
└───────┬───────┘
        ↓
     finally
        ↓
refreshPromise = null
```

وده مهم جدًا.

لو الـ Refresh نجح:

```text id="y7z8a9"
New Token
 ↓
Retry
 ↓
finally
 ↓
refreshPromise = null
```

ولو الـ Refresh فشل:

```text id="b1c2d3"
Refresh Failed
 ↓
finally
 ↓
refreshPromise = null
```

### إذن الكود بقى

```ts id="e4f5g6"
if (!refreshPromise) {
  refreshPromise = refreshAccessToken();
}

try {
  const newToken = await refreshPromise;

  originalRequest.headers.Authorization =
    `Bearer ${newToken}`;

  return api(originalRequest);

} finally {
  refreshPromise = null;
}
```

### احفظ دورة حياة `refreshPromise`

```text id="h7i8j9"
null
 ↓
Refresh يبدأ
 ↓
Promise موجود
 ↓
Requests تستنى
 ↓
Refresh يخلص
 ↓
Retry
 ↓
null
```

ودي نقطة مهمة: **`refreshPromise = null` لا تعني إن الـ Access Token اتمسح.**

هي فقط بتقول:

> **مفيش Refresh Process شغال حاليًا.**

الـ Access Token الجديد لسه موجود في الـ storage.

**الخطوة التالية:** هنعمل محاكاة لـ **5 Requests في نفس اللحظة** ونمشي عليهم واحد واحد، ودي هتثبت الـ Queue نهائيًا.
<details>
<summary>كمل</summary>
user
كمل
ChatGPT
نعمل **محاكاة لـ 5 Requests في نفس اللحظة في 10 دقائق**، من غير كود جديد في البداية، عشان تشوف الـ Queue وهي شغالة فعليًا.

### 🗺️ Roadmap

```text id="x7k2m9"
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
     ├─ Reset refreshPromise       ✅
     └─ Concurrent Requests       🟡 ← هنا
```

عندنا:

```text id="m4n8q2"
Access Token = OLD_TOKEN ❌
Refresh Token = REFRESH_123 ✅
```

وفي نفس اللحظة التطبيق عمل:

```text id="p6r1t9"
A → GET /users
B → GET /products
C → GET /orders
D → GET /notifications
E → GET /profile
```

كلهم استخدموا:

```text id="u3v7w5"
OLD_TOKEN
```

والـ Backend قال:

```text id="z8c2x4"
A → 401
B → 401
C → 401
D → 401
E → 401
```

---

## Request A

أول واحد وصل:

```text id="a5b9d1"
A → 401
```

عندنا:

```text id="e2f6g8"
refreshPromise = null
```

إذن:

```ts id="h4j7k3"
refreshPromise = refreshAccessToken();
```

دلوقتي:

```text id="l9m2n5"
refreshPromise = Refresh Process #1
```

**A بدأ الـ Refresh.**

---

## Request B

وصل `401`:

```text id="q6s8u1"
B → 401
```

يفحص:

```ts id="w3y5z7"
if (!refreshPromise)
```

النتيجة:

```text id="c9v4b6"
false
```

لأن فيه Refresh شغال.

فيعمل:

```ts id="d2f8h1"
const newToken = await refreshPromise;
```

B يقول:

> أنا هستنى Refresh بتاع A.

---

## C و D و E

نفس الكلام:

```text id="j5k7m3"
C → await refreshPromise
D → await refreshPromise
E → await refreshPromise
```

بقى عندنا:

```text id="n8p2r4"
             Refresh #1
                 │
        ┌────────┼────────┐
        ↓        ↓        ↓
        A        B        C
                 ↓
                D
                 ↓
                E
```

**Refresh واحد فقط.**

---

## الـ Backend يرد

الـ Refresh يرجع:

```json id="t6v9x2"
{
  "accessToken": "NEW_TOKEN"
}
```

الـ Promise يخلص.

كل اللي كان عامل:

```ts id="k1m4q8"
await refreshPromise
```

ياخد:

```text id="s3u7w9"
NEW_TOKEN
```

---

## نعمل Retry

كل Request يغير الـ Authorization:

```text id="b5d8f1"
A → NEW_TOKEN
B → NEW_TOKEN
C → NEW_TOKEN
D → NEW_TOKEN
E → NEW_TOKEN
```

ثم:

```text id="g2j6l9"
A → Retry → 200 ✅
B → Retry → 200 ✅
C → Retry → 200 ✅
D → Retry → 200 ✅
E → Retry → 200 ✅
```

---

## وأخيرًا

```ts id="m8n3p7"
refreshPromise = null;
```

يعني:

```text id="r4t9v2"
Refresh Process
     ↓
خلص
     ↓
refreshPromise = null
```

لكن:

```text id="w6x1z5"
Access Token = NEW_TOKEN ✅
```

لسه موجود.

---

# دلوقتي شوف ليه الاسم `refreshPromise` منطقي

مش:

```text
refreshToken
```

لأن ده **Token حقيقي**.

ومش:

```text
accessToken
```

لأن ده **Token حقيقي**.

لكن:

```text id="a7c4e9"
refreshPromise
```

ده:

> **العملية اللي بنستنى نتيجتها عشان تجيب Access Token جديد.**

وده سبب إن Requests تقدر تعمل:

```ts id="f2h5j8"
await refreshPromise;
```

وتستنى **نفس العملية**.

### 🎯 آخر اختبار قبل ما نعتبر الـ Queue خلصت

عندنا:

```text
A → 401
B → 401
C → 401
```

و:

```text
refreshPromise = null
```

**مين يبدأ الـ Refresh؟ وماذا يفعل B و C؟**

جاوبها بطريقتك، وبعدها هنقفل `4.6` وننتقل للجزء التالي.
</details>
كمل
ChatGPT
نعمل **محاكاة لـ 5 Requests في نفس اللحظة في 10 دقائق**، من غير كود جديد في البداية، عشان تشوف الـ Queue وهي شغالة فعليًا.

### 🗺️ Roadmap

```text id="x7k2m9"
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
     ├─ Reset refreshPromise       ✅
     └─ Concurrent Requests       🟡 ← هنا
```

عندنا:

```text id="m4n8q2"
Access Token = OLD_TOKEN ❌
Refresh Token = REFRESH_123 ✅
```

وفي نفس اللحظة التطبيق عمل:

```text id="p6r1t9"
A → GET /users
B → GET /products
C → GET /orders
D → GET /notifications
E → GET /profile
```

كلهم استخدموا:

```text id="u3v7w5"
OLD_TOKEN
```

والـ Backend قال:

```text id="z8c2x4"
A → 401
B → 401
C → 401
D → 401
E → 401
```

---

## Request A

أول واحد وصل:

```text id="a5b9d1"
A → 401
```

عندنا:

```text id="e2f6g8"
refreshPromise = null
```

إذن:

```ts id="h4j7k3"
refreshPromise = refreshAccessToken();
```

دلوقتي:

```text id="l9m2n5"
refreshPromise = Refresh Process #1
```

**A بدأ الـ Refresh.**

---

## Request B

وصل `401`:

```text id="q6s8u1"
B → 401
```

يفحص:

```ts id="w3y5z7"
if (!refreshPromise)
```

النتيجة:

```text id="c9v4b6"
false
```

لأن فيه Refresh شغال.

فيعمل:

```ts id="d2f8h1"
const newToken = await refreshPromise;
```

B يقول:

> أنا هستنى Refresh بتاع A.

---

## C و D و E

نفس الكلام:

```text id="j5k7m3"
C → await refreshPromise
D → await refreshPromise
E → await refreshPromise
```

بقى عندنا:

```text id="n8p2r4"
             Refresh #1
                 │
        ┌────────┼────────┐
        ↓        ↓        ↓
        A        B        C
                 ↓
                D
                 ↓
                E
```

**Refresh واحد فقط.**

---

## الـ Backend يرد

الـ Refresh يرجع:

```json id="t6v9x2"
{
  "accessToken": "NEW_TOKEN"
}
```

الـ Promise يخلص.

كل اللي كان عامل:

```ts id="k1m4q8"
await refreshPromise
```

ياخد:

```text id="s3u7w9"
NEW_TOKEN
```

---

## نعمل Retry

كل Request يغير الـ Authorization:

```text id="b5d8f1"
A → NEW_TOKEN
B → NEW_TOKEN
C → NEW_TOKEN
D → NEW_TOKEN
E → NEW_TOKEN
```

ثم:

```text id="g2j6l9"
A → Retry → 200 ✅
B → Retry → 200 ✅
C → Retry → 200 ✅
D → Retry → 200 ✅
E → Retry → 200 ✅
```

---

## وأخيرًا

```ts id="m8n3p7"
refreshPromise = null;
```

يعني:

```text id="r4t9v2"
Refresh Process
     ↓
خلص
     ↓
refreshPromise = null
```

لكن:

```text id="w6x1z5"
Access Token = NEW_TOKEN ✅
```

لسه موجود.

---

# دلوقتي شوف ليه الاسم `refreshPromise` منطقي

مش:

```text
refreshToken
```

لأن ده **Token حقيقي**.

ومش:

```text
accessToken
```

لأن ده **Token حقيقي**.

لكن:

```text id="a7c4e9"
refreshPromise
```

ده:

> **العملية اللي بنستنى نتيجتها عشان تجيب Access Token جديد.**

وده سبب إن Requests تقدر تعمل:

```ts id="f2h5j8"
await refreshPromise;
```

وتستنى **نفس العملية**.

### 🎯 آخر اختبار قبل ما نعتبر الـ Queue خلصت

عندنا:

```text
A → 401
B → 401
C → 401
```

و:

```text
refreshPromise = null
```

**مين يبدأ الـ Refresh؟ وماذا يفعل B و C؟**

جاوبها بطريقتك، وبعدها هنقفل `4.6` وننتقل للجزء التالي.
<details>
<summary>نبدأ بـ OCU وينتظر لحين الانتهاء من... بعد كده نحولها لـ Falls.</summary>
user
نبدأ بـ OCU وينتظر لحين الانتهاء من... بعد كده نحولها لـ Falls.
ChatGPT
أيوه، **الفكرة اللي تقصدها صح**، بس حصل لخبطه في الكلمات. أنت غالبًا تقصد:

```text
A يبدأ الـ Refresh
B و C ينتظروا
```

وبالتحديد:

```text
A → 401
↓
refreshPromise = null
↓
A يبدأ Refresh
↓
refreshPromise أصبح فيه عملية شغالة

B → 401
↓
يلاقي refreshPromise موجود
↓
ينتظرها

C → 401
↓
يلاقي refreshPromise موجود
↓
ينتظرها
```

وبعد ما الـ Refresh يخلص:

```text
New Access Token
      ↓
A → Retry
B → Retry
C → Retry
      ↓
200 ✅
```

أما كلمة **`Falls`** فلو قصدك `false`، فهي هنا:

```ts
if (!refreshPromise)
```

في الأول:

```text
refreshPromise = null
!null = true
```

فنبدأ Refresh.

لكن بعد ما بدأ:

```text
refreshPromise = عملية شغالة
!refreshPromise = false
```

فـ B و C **مش هيبدأوا Refresh جديد**.

### أهم حاجة

أنت فهمت الـ Queue من ناحية الـ Logic. اللي ناقصك الآن هو **الـ Syntax والممارسة**، مش Concept جديد.

**الخطوة التالية:** هنقفل `4.6` بتمرين كود صغير جدًا: هديك `Response Interceptor` ناقص، وأنت تكمله من الذاكرة.
</details>
نبدأ بـ OCU وينتظر لحين الانتهاء من... بعد كده نحولها لـ Falls.
ChatGPT
أيوه، **الفكرة اللي تقصدها صح**، بس حصل لخبطه في الكلمات. أنت غالبًا تقصد:

```text
A يبدأ الـ Refresh
B و C ينتظروا
```

وبالتحديد:

```text
A → 401
↓
refreshPromise = null
↓
A يبدأ Refresh
↓
refreshPromise أصبح فيه عملية شغالة

B → 401
↓
يلاقي refreshPromise موجود
↓
ينتظرها

C → 401
↓
يلاقي refreshPromise موجود
↓
ينتظرها
```

وبعد ما الـ Refresh يخلص:

```text
New Access Token
      ↓
A → Retry
B → Retry
C → Retry
      ↓
200 ✅
```

أما كلمة **`Falls`** فلو قصدك `false`، فهي هنا:

```ts
if (!refreshPromise)
```

في الأول:

```text
refreshPromise = null
!null = true
```

فنبدأ Refresh.

لكن بعد ما بدأ:

```text
refreshPromise = عملية شغالة
!refreshPromise = false
```

فـ B و C **مش هيبدأوا Refresh جديد**.

### أهم حاجة

أنت فهمت الـ Queue من ناحية الـ Logic. اللي ناقصك الآن هو **الـ Syntax والممارسة**، مش Concept جديد.

**الخطوة التالية:** هنقفل `4.6` بتمرين كود صغير جدًا: هديك `Response Interceptor` ناقص، وأنت تكمله من الذاكرة.
<details>
<summary>اوك</summary>
user
اوك
ChatGPT
اكتب التمرين ده **من غير ما ترجع للكود السابق، في 10 دقائق**. المطلوب نثبت `Token Refresh + Queue` نهائيًا.

### 🗺️ Roadmap

```text
SECTION 4: Interceptors 🟢
├─ 4.1 What is Interceptor       ✅
├─ 4.2 Request Interceptor       ✅
├─ 4.3 Response Interceptor      ✅
├─ 4.4 Authentication            ✅
├─ 4.5 401 Handling              ✅
└─ 4.6 Token Refresh + Queue     🟡 ← الاختبار النهائي
```
</details>
اوك
ChatGPT
اكتب التمرين ده **من غير ما ترجع للكود السابق، في 10 دقائق**. المطلوب نثبت `Token Refresh + Queue` نهائيًا.

### 🗺️ Roadmap

```text
SECTION 4: Interceptors 🟢
├─ 4.1 What is Interceptor       ✅
├─ 4.2 Request Interceptor       ✅
├─ 4.3 Response Interceptor      ✅
├─ 4.4 Authentication            ✅
├─ 4.5 401 Handling              ✅
└─ 4.6 Token Refresh + Queue     🟡 ← الاختبار النهائي
```