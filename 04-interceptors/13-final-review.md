### الكود

```ts id="m2q7vx"
let refreshPromise = null;

api.interceptors.response.use(
  (response) => {
    return response;
  },

  async (error) => {

    // 1. لو مش 401
    if (______________________________) {
      return Promise.reject(error);
    }

    // 2. الـ Request الأصلي
    const originalRequest = __________________;

    // 3. منع Infinite Loop
    if (______________________________) {
      return Promise.reject(error);
    }

    originalRequest._retry = true;

    // 4. ابدأ Refresh لو مفيش واحد شغال
    if (______________________________) {
      refreshPromise = __________________;
    }

    // 5. استنى الـ Refresh
    const newToken = __________________;

    // 6. حط الـ Token الجديد
    originalRequest.headers.Authorization =
      `Bearer ${newToken}`;

    // 7. Retry
    return ______________________________;
  }
);
```

املأ:

```text
1 = 
2 = 
3 = 
4 = 
5 = 
6 = 
```

ملاحظة: **رقم 6 موجود في الكود أصلًا**، فمش محتاج تكتب فيه حاجة. يعني فعليًا عندك **6 فراغات فقط**.

**الخطوة التالية:** ابعت إجاباتك الستة بالترتيب، حتى لو فيه أخطاء.
<details>
<summary>انا جوبتهم قبل كده</summary>
user
انا جوبتهم قبل كده
ChatGPT
تمام. بما إنك جاوبت عليهم بالفعل، **مش هعيد نفس التمرين**.

نعتبر تمرين الـ 4.6 الأساسي **مُنجز**، ونكمل على الجزء اللي ناقص قبل ما نقفل الـ Token Refresh.

### 🗺️ Roadmap

```text
SECTION 0  Architecture Foundation     ✅
SECTION 1  Axios Fundamentals          ✅
SECTION 2  TypeScript + Axios          ✅
SECTION 3  Error Handling              ✅

SECTION 4  Interceptors                🟢
 ├─ 4.1 Interceptor                   ✅
 ├─ 4.2 Request Interceptor            ✅
 ├─ 4.3 Response Interceptor           ✅
 ├─ 4.4 Authentication                 ✅
 ├─ 4.5 401 Handling                   ✅
 └─ 4.6 Token Refresh + Queue          🟢 ← هنا

SECTION 5  API Architecture             ⬜
```

**دلوقتي خلال 10 دقائق هنقفل 3 حاجات مهمة في 4.6:**  
1. ماذا يحدث لو الـ Refresh نفسه فشل؟
2. لماذا لازم نعمل `refreshPromise = null`؟
3. كيف نمنع الـ Refresh من الدخول في Refresh Loop؟

وبعدهم **سؤال Interview واحد فقط**، ولو إجابتك سليمة نقفل Section 4 بالكامل ونبدأ Section 5.
</details>
انا جوبتهم قبل كده
ChatGPT
تمام. بما إنك جاوبت عليهم بالفعل، **مش هعيد نفس التمرين**.

نعتبر تمرين الـ 4.6 الأساسي **مُنجز**، ونكمل على الجزء اللي ناقص قبل ما نقفل الـ Token Refresh.

### 🗺️ Roadmap

```text
SECTION 0  Architecture Foundation     ✅
SECTION 1  Axios Fundamentals          ✅
SECTION 2  TypeScript + Axios          ✅
SECTION 3  Error Handling              ✅

SECTION 4  Interceptors                🟢
 ├─ 4.1 Interceptor                   ✅
 ├─ 4.2 Request Interceptor            ✅
 ├─ 4.3 Response Interceptor           ✅
 ├─ 4.4 Authentication                 ✅
 ├─ 4.5 401 Handling                   ✅
 └─ 4.6 Token Refresh + Queue          🟢 ← هنا

SECTION 5  API Architecture             ⬜
```

**دلوقتي خلال 10 دقائق هنقفل 3 حاجات مهمة في 4.6:**  
1. ماذا يحدث لو الـ Refresh نفسه فشل؟
2. لماذا لازم نعمل `refreshPromise = null`؟
3. كيف نمنع الـ Refresh من الدخول في Refresh Loop؟

وبعدهم **سؤال Interview واحد فقط**، ولو إجابتك سليمة نقفل Section 4 بالكامل ونبدأ Section 5.
<details>
<summary>ماذا يحدث لو retry نفسه فشل؟ لما بيحصل فيه retry نفسه فشل فبيحصل حاجة اسمها infinite loop، وفي الحاجة دي إحنا بنعملها بحاجة اسمها flag عشان ما يعملش retry كذا مرة، بتبقى false وبعد كده بيحولها true. رقم اتنين: لماذا نبدأ بالـrefresh promise بـ? لأن في حاجة بيبقاش فيه refresh جديد، يعني بيجيب refresh جديد، لكن في الحاجة اللي أنا بجيبها بيبقى بعمله retry وبعد كده flag. كيف نمنع الـrefresh من الدخول في loop؟ حاجة اللي أنا بعمل flag عن طريق retry وfalse، وبعد ما أتأكد إن هي false أحولها true.</summary>
user
ماذا يحدث لو retry نفسه فشل؟ لما بيحصل فيه retry نفسه فشل فبيحصل حاجة اسمها infinite loop، وفي الحاجة دي إحنا بنعملها بحاجة اسمها flag عشان ما يعملش retry كذا مرة، بتبقى false وبعد كده بيحولها true. رقم اتنين: لماذا نبدأ بالـrefresh promise بـ? لأن في حاجة بيبقاش فيه refresh جديد، يعني بيجيب refresh جديد، لكن في الحاجة اللي أنا بجيبها بيبقى بعمله retry وبعد كده flag. كيف نمنع الـrefresh من الدخول في loop؟ حاجة اللي أنا بعمل flag عن طريق retry وfalse، وبعد ما أتأكد إن هي false أحولها true.
ChatGPT
**خلال 5 دقائق هنثبت النقطة دي لأن عندك الفكرة الأساسية صح، لكن فيه خلط بسيط بين `retry` و`refresh`.**

### 1. لو الـ Retry نفسه فشل

إجابتك **صح**.

السيناريو:

```text
Request
  ↓
401
  ↓
Refresh Token
  ↓
New Access Token
  ↓
Retry Original Request
  ↓
401 مرة تانية
```

هنا **ممنوع نعمل Refresh تاني لنفس الـ Request إلى ما لا نهاية**.

علشان كده:

```ts
if (originalRequest._retry) {
  return Promise.reject(error);
}

originalRequest._retry = true;
```

المعنى:

```text
أول مرة:
_retry = false / undefined
       ↓
مسموح Retry
       ↓
_retry = true

ثاني مرة:
_retry = true
       ↓
ممنوع Retry
       ↓
reject
```

**دي نقطة 10/10 عندك.**

---

### 2. ليه بنستخدم `refreshPromise`؟

هنا محتاج أصحح لك نقطة.

`refreshPromise` **مش بيجيب Refresh Token جديد**.

هو بيمسك **عملية الـ Refresh الحالية**.

تخيل عندنا 5 Requests:

```text
A → 401
B → 401
C → 401
D → 401
E → 401
```

أول Request:

```ts
if (!refreshPromise) {
  refreshPromise = refreshAccessToken();
}
```

يعني:

```text
refreshPromise = null
        ↓
A يبدأ Refresh
        ↓
refreshPromise أصبح فيه عملية Refresh شغالة
```

لما B و C و D و E يوصلوا:

```ts
await refreshPromise
```

هما **مش بيبدأوا Refresh جديد**.

هما بيقولوا:

> "طالما فيه Refresh شغال، أنا هستنى نفس العملية."

وده هو سبب إننا بنستخدم `refreshPromise`.

---

### 3. إزاي نمنع Refresh نفسه من الدخول في Loop؟

هنا إجابتك **قريبة، لكن مش بالـ `_retry` وحده**.

عندنا مشكلتين مختلفتين:

#### المشكلة الأولى: Original Request

نمنعها بـ:

```ts
originalRequest._retry
```

#### المشكلة الثانية: Refresh Request نفسه

مثلاً:

```text
GET /users
   ↓
401
   ↓
POST /auth/refresh
   ↓
401
   ↓
Interceptor يقول:
"طيب أعمل Refresh!"
   ↓
POST /auth/refresh
   ↓
401
   ↓
Refresh
   ↓
∞
```

هنا لازم نقول للـ Interceptor:

> "الطلب ده نفسه هو `/auth/refresh`، متعملش عليه Refresh."

مثلاً بشكل مبسط:

```ts
if (originalRequest.url === "/auth/refresh") {
  return Promise.reject(error);
}
```

فتصبح الصورة:

```text
Original Request
      ↓
     401
      ↓
    Refresh
      ↓
   Refresh 401
      ↓
❌ Stop
```

### الفرق المهم جدًا

احفظ الاتنين دول:

```text
_retry
↓
يمنع Original Request من الـ Retry بلا نهاية


refresh endpoint protection
↓
يمنع Refresh Request نفسه من تشغيل Refresh تاني
```

أنت فاهم الـ **Infinite Loop** كويس جدًا. المشكلة الوحيدة كانت إنك استخدمت نفس الآلية للمشكلتين.

**الخطوة التالية: جاوبني في سؤال Interview واحد: لو 5 Requests أخدوا `401` في نفس اللحظة، احكيلي بالترتيب إيه اللي يحصل من أول Request A لحد ما الخمسة يعملوا Retry.**
</details>
ماذا يحدث لو retry نفسه فشل؟ لما بيحصل فيه retry نفسه فشل فبيحصل حاجة اسمها infinite loop، وفي الحاجة دي إحنا بنعملها بحاجة اسمها flag عشان ما يعملش retry كذا مرة، بتبقى false وبعد كده بيحولها true. رقم اتنين: لماذا نبدأ بالـrefresh promise بـ? لأن في حاجة بيبقاش فيه refresh جديد، يعني بيجيب refresh جديد، لكن في الحاجة اللي أنا بجيبها بيبقى بعمله retry وبعد كده flag. كيف نمنع الـrefresh من الدخول في loop؟ حاجة اللي أنا بعمل flag عن طريق retry وfalse، وبعد ما أتأكد إن هي false أحولها true.
ChatGPT
**خلال 5 دقائق هنثبت النقطة دي لأن عندك الفكرة الأساسية صح، لكن فيه خلط بسيط بين `retry` و`refresh`.**

### 1. لو الـ Retry نفسه فشل

إجابتك **صح**.

السيناريو:

```text
Request
  ↓
401
  ↓
Refresh Token
  ↓
New Access Token
  ↓
Retry Original Request
  ↓
401 مرة تانية
```

هنا **ممنوع نعمل Refresh تاني لنفس الـ Request إلى ما لا نهاية**.

علشان كده:

```ts
if (originalRequest._retry) {
  return Promise.reject(error);
}

originalRequest._retry = true;
```

المعنى:

```text
أول مرة:
_retry = false / undefined
       ↓
مسموح Retry
       ↓
_retry = true

ثاني مرة:
_retry = true
       ↓
ممنوع Retry
       ↓
reject
```

**دي نقطة 10/10 عندك.**

---

### 2. ليه بنستخدم `refreshPromise`؟

هنا محتاج أصحح لك نقطة.

`refreshPromise` **مش بيجيب Refresh Token جديد**.

هو بيمسك **عملية الـ Refresh الحالية**.

تخيل عندنا 5 Requests:

```text
A → 401
B → 401
C → 401
D → 401
E → 401
```

أول Request:

```ts
if (!refreshPromise) {
  refreshPromise = refreshAccessToken();
}
```

يعني:

```text
refreshPromise = null
        ↓
A يبدأ Refresh
        ↓
refreshPromise أصبح فيه عملية Refresh شغالة
```

لما B و C و D و E يوصلوا:

```ts
await refreshPromise
```

هما **مش بيبدأوا Refresh جديد**.

هما بيقولوا:

> "طالما فيه Refresh شغال، أنا هستنى نفس العملية."

وده هو سبب إننا بنستخدم `refreshPromise`.

---

### 3. إزاي نمنع Refresh نفسه من الدخول في Loop؟

هنا إجابتك **قريبة، لكن مش بالـ `_retry` وحده**.

عندنا مشكلتين مختلفتين:

#### المشكلة الأولى: Original Request

نمنعها بـ:

```ts
originalRequest._retry
```

#### المشكلة الثانية: Refresh Request نفسه

مثلاً:

```text
GET /users
   ↓
401
   ↓
POST /auth/refresh
   ↓
401
   ↓
Interceptor يقول:
"طيب أعمل Refresh!"
   ↓
POST /auth/refresh
   ↓
401
   ↓
Refresh
   ↓
∞
```

هنا لازم نقول للـ Interceptor:

> "الطلب ده نفسه هو `/auth/refresh`، متعملش عليه Refresh."

مثلاً بشكل مبسط:

```ts
if (originalRequest.url === "/auth/refresh") {
  return Promise.reject(error);
}
```

فتصبح الصورة:

```text
Original Request
      ↓
     401
      ↓
    Refresh
      ↓
   Refresh 401
      ↓
❌ Stop
```

### الفرق المهم جدًا

احفظ الاتنين دول:

```text
_retry
↓
يمنع Original Request من الـ Retry بلا نهاية


refresh endpoint protection
↓
يمنع Refresh Request نفسه من تشغيل Refresh تاني
```

أنت فاهم الـ **Infinite Loop** كويس جدًا. المشكلة الوحيدة كانت إنك استخدمت نفس الآلية للمشكلتين.

**الخطوة التالية: جاوبني في سؤال Interview واحد: لو 5 Requests أخدوا `401` في نفس اللحظة، احكيلي بالترتيب إيه اللي يحصل من أول Request A لحد ما الخمسة يعملوا Retry.**
<details>
<summary>في حالة الأرباع من الكوستي دوت، ويوخ، ونعمل حاجة اسمها، والبائع يستنى ولما يخلص...</summary>
user
في حالة الأرباع من الكوستي دوت، ويوخ، ونعمل حاجة اسمها، والبائع يستنى ولما يخلص...
ChatGPT
أنت تقصد غالبًا **الـ 5 Requests: A, B, C, D, E**، وإجابتك بدأت في الاتجاه الصح، لكن محتاجة تكمل التسلسل.

**الوقت: دقيقتين.**

الصورة المطلوبة:

```text
A → 401
↓
A يبدأ Refresh
↓
refreshPromise أصبح موجود

B → 401 → يستنى نفس refreshPromise
C → 401 → يستنى نفس refreshPromise
D → 401 → يستنى نفس refreshPromise
E → 401 → يستنى نفس refreshPromise

        ↓
Refresh يخلص
        ↓
New Access Token
        ↓
A B C D E يعملوا Retry
        ↓
Requests تنجح
```

والنقطة الأهم:

**ليه B و C و D و E ما يعملوش Refresh جديد؟**

لأن:

```ts
if (!refreshPromise) {
  refreshPromise = refreshAccessToken();
}
```

أول واحد فقط يجد `refreshPromise = null`، فيبدأ Refresh.

الباقي يجد أن `refreshPromise` **موجود بالفعل**، فيعمل:

```ts
await refreshPromise;
```

ويستنى نفس العملية.

**كمّل إجابتك من لحظة ما الـ Refresh يخلص: إيه اللي يحصل لـ A وB وC وD وE؟**
</details>
في حالة الأرباع من الكوستي دوت، ويوخ، ونعمل حاجة اسمها، والبائع يستنى ولما يخلص...
ChatGPT
أنت تقصد غالبًا **الـ 5 Requests: A, B, C, D, E**، وإجابتك بدأت في الاتجاه الصح، لكن محتاجة تكمل التسلسل.

**الوقت: دقيقتين.**

الصورة المطلوبة:

```text
A → 401
↓
A يبدأ Refresh
↓
refreshPromise أصبح موجود

B → 401 → يستنى نفس refreshPromise
C → 401 → يستنى نفس refreshPromise
D → 401 → يستنى نفس refreshPromise
E → 401 → يستنى نفس refreshPromise

        ↓
Refresh يخلص
        ↓
New Access Token
        ↓
A B C D E يعملوا Retry
        ↓
Requests تنجح
```

والنقطة الأهم:

**ليه B و C و D و E ما يعملوش Refresh جديد؟**

لأن:

```ts
if (!refreshPromise) {
  refreshPromise = refreshAccessToken();
}
```

أول واحد فقط يجد `refreshPromise = null`، فيبدأ Refresh.

الباقي يجد أن `refreshPromise` **موجود بالفعل**، فيعمل:

```ts
await refreshPromise;
```

ويستنى نفس العملية.

**كمّل إجابتك من لحظة ما الـ Refresh يخلص: إيه اللي يحصل لـ A وB وC وD وE؟**
<details>
<summary>إيه هيستنى الأكسس توكن جديد والـE والـD والـC والـB هيبدأوا في الانتظار.</summary>
user
إيه هيستنى الأكسس توكن جديد والـE والـD والـC والـB هيبدأوا في الانتظار.
ChatGPT
**بالضبط، أنت فاهم الفكرة، لكن هنظبط ترتيب الأحداث بدقة في دقيقة واحدة.**

أولًا، **A هو اللي يبدأ الـ Refresh**:

```text
A → 401
↓
A يبدأ Refresh
↓
A ينتظر Access Token الجديد
```

وفي نفس الوقت:

```text
B → 401 → ينتظر
C → 401 → ينتظر
D → 401 → ينتظر
E → 401 → ينتظر
```

بعد ما الـ Refresh يخلص:

```text
Backend
↓
NEW_ACCESS_TOKEN
↓
A + B + C + D + E
↓
كل واحد يأخذ الـ NEW_ACCESS_TOKEN
↓
Retry
↓
Request ناجح
```

### النقطة المهمة جدًا

**B وC وD وE لا يبدأوا Refresh.**

هم فقط:

```ts
const newToken = await refreshPromise;
```

يعني:

> "أنا هستنى العملية اللي A بدأها، ولما تخلص اديني النتيجة."

وأنت كده فهمت جوهر الـ **Token Refresh + Concurrent Requests**.

**الخطوة التالية: جاوبني بس: بعد ما الـ Refresh يخلص، ليه بنعمل `refreshPromise = null`؟**
</details>
إيه هيستنى الأكسس توكن جديد والـE والـD والـC والـB هيبدأوا في الانتظار.
ChatGPT
**بالضبط، أنت فاهم الفكرة، لكن هنظبط ترتيب الأحداث بدقة في دقيقة واحدة.**

أولًا، **A هو اللي يبدأ الـ Refresh**:

```text
A → 401
↓
A يبدأ Refresh
↓
A ينتظر Access Token الجديد
```

وفي نفس الوقت:

```text
B → 401 → ينتظر
C → 401 → ينتظر
D → 401 → ينتظر
E → 401 → ينتظر
```

بعد ما الـ Refresh يخلص:

```text
Backend
↓
NEW_ACCESS_TOKEN
↓
A + B + C + D + E
↓
كل واحد يأخذ الـ NEW_ACCESS_TOKEN
↓
Retry
↓
Request ناجح
```

### النقطة المهمة جدًا

**B وC وD وE لا يبدأوا Refresh.**

هم فقط:

```ts
const newToken = await refreshPromise;
```

يعني:

> "أنا هستنى العملية اللي A بدأها، ولما تخلص اديني النتيجة."

وأنت كده فهمت جوهر الـ **Token Refresh + Concurrent Requests**.

**الخطوة التالية: جاوبني بس: بعد ما الـ Refresh يخلص، ليه بنعمل `refreshPromise = null`؟**
<details>
<summary>عشان ما فيش حد ماسك ساعتها الريفريش.</summary>
user
عشان ما فيش حد ماسك ساعتها الريفريش.
ChatGPT
صح جدًا. **10/10**.

بس خلّي الجملة أدق:

```ts
refreshPromise = null;
```

معناها:

> **مفيش Refresh Process شغالة حاليًا.**

مش معناها إن الـ Access Token اتمسح أو إن الـ Refresh Token اتمسح.

الصورة:

```text
Refresh شغال
↓
refreshPromise = Promise
↓
Refresh خلص
↓
refreshPromise = null
```

ولو حصل `401` جديد بعد كده:

```text
401
↓
refreshPromise = null
↓
نبدأ Refresh جديد
```

كده **4.6 Token Refresh + Queue مكتملة**، وبكده **Section 4 Interceptors كلها مكتملة**. ✅

**الخطوة التالية: نبدأ Section 5: API Architecture، وأول درس فيه هيكون "API Layer vs Component" لمدة 15 دقيقة.**
</details>
عشان ما فيش حد ماسك ساعتها الريفريش.
ChatGPT
صح جدًا. **10/10**.

بس خلّي الجملة أدق:

```ts
refreshPromise = null;
```

معناها:

> **مفيش Refresh Process شغالة حاليًا.**

مش معناها إن الـ Access Token اتمسح أو إن الـ Refresh Token اتمسح.

الصورة:

```text
Refresh شغال
↓
refreshPromise = Promise
↓
Refresh خلص
↓
refreshPromise = null
```

ولو حصل `401` جديد بعد كده:

```text
401
↓
refreshPromise = null
↓
نبدأ Refresh جديد
```

كده **4.6 Token Refresh + Queue مكتملة**، وبكده **Section 4 Interceptors كلها مكتملة**. ✅

**الخطوة التالية: نبدأ Section 5: API Architecture، وأول درس فيه هيكون "API Layer vs Component" لمدة 15 دقيقة.**
<details>
<summary>ممكن اعرف انا فين من الخطه وواقف فين وايه وكان في الميه</summary>
user
ممكن اعرف انا فين من الخطه وواقف فين وايه وكان في الميه
ChatGPT
أنت حاليًا **أنهيت Section 4 بالكامل**، وواقف على باب **Section 5: API Architecture**.

### 🗺️ خطتك بالكامل

| القسم | الموضوع | الحالة |
|---|---|---|
| **0** | Architecture Foundation | ✅ مكتمل |
| **1** | Axios Fundamentals | ✅ مكتمل |
| **2** | TypeScript + Axios | ✅ مكتمل |
| **3** | Error Handling | ✅ مكتمل |
| **4** | Interceptors + Authentication | ✅ مكتمل |
| **5** | API Architecture | ⬜ التالي |
| **6** | TanStack Query | ⬜ |
| **7** | Authentication Architecture | ⬜ |
| **8** | Forms | ⬜ |
| **9** | Tailwind + shadcn/ui | ⬜ |
| **10** | Storybook | ⬜ |
| **11** | Testing | ⬜ |
| **12** | Production Architecture | ⬜ |
| **13** | Final Project | ⬜ |

### 📊 أنت خلصت كام؟

عندنا **14 Sections** من 0 إلى 13.

أنت خلصت **5 Sections**:

```text
█████████████░░░  35.7%
```

يعني تقريبًا **36% من الخطة**.

لكن في نقطة مهمة: **الـ 36% دي نسبة الأقسام، مش نسبة مستواك الحقيقي في التعلم.**

لأن الأقسام اللي خلصتها لحد الآن تعتبر من أهم الأساسات:

```text
HTTP
 ↓
Axios
 ↓
TypeScript + Axios
 ↓
Error Handling
 ↓
Interceptors
 ↓
Authentication
 ↓
Token Refresh + Concurrency
```

وأنت في آخر نقطة دي وصلت لفهم كويس جدًا، خصوصًا إنك قدرت تشرح بنفسك فكرة:

```text
5 Requests → 401
        ↓
One Refresh
        ↓
Others wait
        ↓
New Access Token
        ↓
All Retry
```

### 📍 موقفك الحالي بالضبط

```text
SECTION 4
Interceptors

4.1 ✅
4.2 ✅
4.3 ✅
4.4 ✅
4.5 ✅
4.6 ✅  ← أنت هنا أنهيت القسم

━━━━━━━━━━━━━━━━━━

SECTION 5
API Architecture

5.1 ⬅️ التالي
```

**الخطوة التالية: نبدأ 5.1 API Layer vs Component لمدة 15 دقيقة.**
</details>
ممكن اعرف انا فين من الخطه وواقف فين وايه وكان في الميه
ChatGPT
أنت حاليًا **أنهيت Section 4 بالكامل**، وواقف على باب **Section 5: API Architecture**.

### 🗺️ خطتك بالكامل

| القسم | الموضوع | الحالة |
|---|---|---|
| **0** | Architecture Foundation | ✅ مكتمل |
| **1** | Axios Fundamentals | ✅ مكتمل |
| **2** | TypeScript + Axios | ✅ مكتمل |
| **3** | Error Handling | ✅ مكتمل |
| **4** | Interceptors + Authentication | ✅ مكتمل |
| **5** | API Architecture | ⬜ التالي |
| **6** | TanStack Query | ⬜ |
| **7** | Authentication Architecture | ⬜ |
| **8** | Forms | ⬜ |
| **9** | Tailwind + shadcn/ui | ⬜ |
| **10** | Storybook | ⬜ |
| **11** | Testing | ⬜ |
| **12** | Production Architecture | ⬜ |
| **13** | Final Project | ⬜ |

### 📊 أنت خلصت كام؟

عندنا **14 Sections** من 0 إلى 13.

أنت خلصت **5 Sections**:

```text
█████████████░░░  35.7%
```

يعني تقريبًا **36% من الخطة**.

لكن في نقطة مهمة: **الـ 36% دي نسبة الأقسام، مش نسبة مستواك الحقيقي في التعلم.**

لأن الأقسام اللي خلصتها لحد الآن تعتبر من أهم الأساسات:

```text
HTTP
 ↓
Axios
 ↓
TypeScript + Axios
 ↓
Error Handling
 ↓
Interceptors
 ↓
Authentication
 ↓
Token Refresh + Concurrency
```

وأنت في آخر نقطة دي وصلت لفهم كويس جدًا، خصوصًا إنك قدرت تشرح بنفسك فكرة:

```text
5 Requests → 401
        ↓
One Refresh
        ↓
Others wait
        ↓
New Access Token
        ↓
All Retry
```

### 📍 موقفك الحالي بالضبط

```text
SECTION 4
Interceptors

4.1 ✅
4.2 ✅
4.3 ✅
4.4 ✅
4.5 ✅
4.6 ✅  ← أنت هنا أنهيت القسم

━━━━━━━━━━━━━━━━━━

SECTION 5
API Architecture

5.1 ⬅️ التالي
```

**الخطوة التالية: نبدأ 5.1 API Layer vs Component لمدة 15 دقيقة.**
<details>
<summary>طب بنسبه لي axios خلصنا قد ايه</summary>
user
طب بنسبه لي axios خلصنا قد ايه
ChatGPT
**اعتبر Axios نفسه خلص بنسبة حوالي 80%، والـ20% الباقية هتتثبت وتدخل في Architecture وProduction.**

### 📊 تقدمك في Axios

```text
Axios Fundamentals          ✅ 100%
TypeScript + Axios          ✅ 100%
Error Handling              ✅ 100%
Interceptors                ✅ 100%
Authentication              ✅ 100%
Token Refresh + Queue       ✅ 100%

━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Axios كـ Core Technology    🟢 ~80%
```

أنت فعليًا غطيت معظم الـ Axios اللي هتستخدمه يوميًا:

- `GET / POST / PUT / PATCH / DELETE` ✅
- Config / `params` / `headers` / `data` ✅
- `timeout` / `AbortController` ✅
- `AxiosResponse` و Generics ✅
- API Response Types ✅
- Discriminated Unions ✅
- `unknown` و `AxiosError` ✅
- Error Normalization ✅
- Custom `ApiError` concept ✅
- `axios.create()` ✅
- Request Interceptor ✅
- Response Interceptor ✅
- Authentication ✅
- `401 Handling` ✅
- Token Refresh ✅
- Concurrent Requests / Queue pattern ✅

**اللي فاضل مش أساسيات Axios، وإنما استخدام Axios داخل Architecture Production بشكل احترافي**، وده هنبدأه في Section 5.

يعني لو سؤالك: **"هل أقدر أقول إني اتعلمت Axios؟"**

نعم. **تقدر تقول إنك أتقنت الـ Core Axios concepts، ولسه محتاج تثبيت الـ Architecture والـ Production patterns.**

**الخطوة التالية: نبدأ 5.1 ونشوف إزاي نحط Axios في API Architecture صح.**
</details>
طب بنسبه لي axios خلصنا قد ايه
ChatGPT
**اعتبر Axios نفسه خلص بنسبة حوالي 80%، والـ20% الباقية هتتثبت وتدخل في Architecture وProduction.**

### 📊 تقدمك في Axios

```text
Axios Fundamentals          ✅ 100%
TypeScript + Axios          ✅ 100%
Error Handling              ✅ 100%
Interceptors                ✅ 100%
Authentication              ✅ 100%
Token Refresh + Queue       ✅ 100%

━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Axios كـ Core Technology    🟢 ~80%
```

أنت فعليًا غطيت معظم الـ Axios اللي هتستخدمه يوميًا:

- `GET / POST / PUT / PATCH / DELETE` ✅
- Config / `params` / `headers` / `data` ✅
- `timeout` / `AbortController` ✅
- `AxiosResponse` و Generics ✅
- API Response Types ✅
- Discriminated Unions ✅
- `unknown` و `AxiosError` ✅
- Error Normalization ✅
- Custom `ApiError` concept ✅
- `axios.create()` ✅
- Request Interceptor ✅
- Response Interceptor ✅
- Authentication ✅
- `401 Handling` ✅
- Token Refresh ✅
- Concurrent Requests / Queue pattern ✅

**اللي فاضل مش أساسيات Axios، وإنما استخدام Axios داخل Architecture Production بشكل احترافي**، وده هنبدأه في Section 5.

يعني لو سؤالك: **"هل أقدر أقول إني اتعلمت Axios؟"**

نعم. **تقدر تقول إنك أتقنت الـ Core Axios concepts، ولسه محتاج تثبيت الـ Architecture والـ Production patterns.**

**الخطوة التالية: نبدأ 5.1 ونشوف إزاي نحط Axios في API Architecture صح.**
<details>
<summary>Section 5: API Architecture.</summary>
user
Section 5: API Architecture.
يلا بينا
ChatGPT
**ابدأ معايا 5.1 لمدة 15 دقيقة: هنفهم الفرق بين `Component` و`API Layer`، وده الأساس اللي هنبني عليه الـ API Architecture كلها.**

### 🗺️ Roadmap

```text
SECTION 0  Architecture Foundation     ✅
SECTION 1  Axios Fundamentals          ✅
SECTION 2  TypeScript + Axios          ✅
SECTION 3  Error Handling              ✅
SECTION 4  Interceptors                ✅

SECTION 5  API Architecture            🟢 ← هنا
 ├─ 5.1 API Layer vs Component         🟢
 ├─ 5.2 API Client                     ⬜
 ├─ 5.3 Feature API                    ⬜
 ├─ 5.4 Service Layer                  ⬜
 ├─ 5.5 Dependency Direction            ⬜
 └─ 5.6 Complete API Architecture      ⬜

SECTION 6  TanStack Query               ⬜
SECTION 7  Authentication Architecture ⬜
SECTION 8  Forms                        ⬜
SECTION 9  Tailwind + shadcn/ui         ⬜
SECTION 10 Storybook                    ⬜
SECTION 11 Testing                      ⬜
SECTION 12 Production Architecture     ⬜
SECTION 13 Final Project                ⬜
```
</details>
Section 5: API Architecture.
يلا بينا
ChatGPT
**ابدأ معايا 5.1 لمدة 15 دقيقة: هنفهم الفرق بين `Component` و`API Layer`، وده الأساس اللي هنبني عليه الـ API Architecture كلها.**

### 🗺️ Roadmap

```text
SECTION 0  Architecture Foundation     ✅
SECTION 1  Axios Fundamentals          ✅
SECTION 2  TypeScript + Axios          ✅
SECTION 3  Error Handling              ✅
SECTION 4  Interceptors                ✅

SECTION 5  API Architecture            🟢 ← هنا
 ├─ 5.1 API Layer vs Component         🟢
 ├─ 5.2 API Client                     ⬜
 ├─ 5.3 Feature API                    ⬜
 ├─ 5.4 Service Layer                  ⬜
 ├─ 5.5 Dependency Direction            ⬜
 └─ 5.6 Complete API Architecture      ⬜

SECTION 6  TanStack Query               ⬜
SECTION 7  Authentication Architecture ⬜
SECTION 8  Forms                        ⬜
SECTION 9  Tailwind + shadcn/ui         ⬜
SECTION 10 Storybook                    ⬜
SECTION 11 Testing                      ⬜
SECTION 12 Production Architecture     ⬜
SECTION 13 Final Project                ⬜
```