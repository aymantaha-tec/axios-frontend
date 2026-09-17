## 1. أول حاجة: نعمل `refreshPromise`

بدل ما نعمل Refresh لكل Request، هنخلي عندنا **Promise واحد فقط**:

```ts
let refreshPromise: Promise<string> | null = null;
```

معناه:

```text
null
↓
مفيش Refresh شغال
```

ولو بدأ Refresh:

```text
refreshPromise
↓
Promise شغال
```

أي Request تاني يقدر يستنى نفس الـ Promise.

---

## 2. نعمل Function للـ Refresh

```ts
const refreshAccessToken = async (): Promise<string> => {
  const refreshToken = localStorage.getItem("refreshToken");

  if (!refreshToken) {
    throw new Error("No refresh token");
  }

  const response = await api.post("/auth/refresh", {
    refreshToken,
  });

  const newAccessToken = response.data.accessToken;

  localStorage.setItem(
    "accessToken",
    newAccessToken
  );

  return newAccessToken;
};
```

ركز في آخر سطر:

```ts
return newAccessToken;
```

إذن:

```text
refreshAccessToken()
↓
Promise<string>
↓
"NEW_ACCESS_TOKEN"
```

---

## 3. دلوقتي الـ Response Interceptor

```ts
api.interceptors.response.use(
  (response) => response,

  async (error) => {
    const originalRequest = error.config;

    if (error.response?.status !== 401) {
      return Promise.reject(error);
    }

    if (originalRequest._retry) {
      return Promise.reject(error);
    }

    originalRequest._retry = true;

    // مفيش Refresh شغال
    if (!refreshPromise) {
      refreshPromise = refreshAccessToken();

      try {
        const newAccessToken = await refreshPromise;

        originalRequest.headers.Authorization =
          `Bearer ${newAccessToken}`;

        return api(originalRequest);
      } finally {
        refreshPromise = null;
      }
    }

    // فيه Refresh شغال بالفعل
    const newAccessToken = await refreshPromise;

    originalRequest.headers.Authorization =
      `Bearer ${newAccessToken}`;

    return api(originalRequest);
  }
);
```

## 4. تعال نفك الكود

### Request A

```text
A → 401
```

يدخل:

```ts
if (!refreshPromise)
```

وهي حاليًا:

```ts
null
```

إذن:

```ts
refreshPromise = refreshAccessToken();
```

بدأ Refresh.

```text
A
↓
401
↓
Refresh بدأ
```

---

### Request B

في نفس الوقت:

```text
B → 401
```

يوصل هنا:

```ts
if (!refreshPromise)
```

لكن دلوقتي:

```text
refreshPromise = Promise شغال
```

فـ B **مش هيعمل Refresh جديد**.

هيعمل:

```ts
const newAccessToken = await refreshPromise;
```

يعني:

> استنى نفس الـ Refresh اللي A بدأه.

---

### Request C

نفس الكلام:

```text
C → 401
↓
Refresh موجود
↓
await refreshPromise
↓
استنى
```

فتبقى:

```text
A → Refresh
B → Wait
C → Wait
D → Wait
E → Wait
```

---

## 5. لما الـ Refresh يخلص

الـ Promise يرجع:

```text
NEW_ACCESS_TOKEN
```

فكل اللي كان بيعمل:

```ts
await refreshPromise
```

ياخد:

```text
NEW_ACCESS_TOKEN
```

وبعدين:

```ts
originalRequest.headers.Authorization =
  `Bearer ${newAccessToken}`;
```

ثم:

```ts
return api(originalRequest);
```

يعني:

```text
A → Retry
B → Retry
C → Retry
D → Retry
```

وكلهم بالـ Token الجديد.

---

## 6. أهم سطر في الكود كله

ده:

```ts
refreshPromise = refreshAccessToken();
```

**مش:**

```ts
await refreshAccessToken();
```

ليه؟

لأننا عايزين نحتفظ بالـ Promise نفسه:

```text
refreshPromise
       ↓
┌──────┼──────┐
A      B      C
↓      ↓      ↓
await  await  await
└──────┼──────┘
       ↓
 New Access Token
```

وده هو جوهر فكرة الـ Queue هنا.

---

### 🎯 تمرين عملي

مش هديك الكود كامل المرة الجاية.

اكتبلي بنفسك الجزء ده فقط:

```ts
let refreshPromise: Promise<string> | null = null;
```

وبعده Function اسمها:

```ts
refreshAccessToken
```

تجيب `refreshToken`، تعمل:

```ts
POST /auth/refresh
```

وترجع `newAccessToken`.

**الخطوة التالية:** اكتب الـ `refreshAccessToken` بنفسك هنا، وأنا هراجع الكود سطر سطر قبل ما نكمل الـ Interceptor.
<details>
<summary>refreshPromise</summary>
user
refreshPromise
ده ايه بضبط
ChatGPT
افهم `refreshPromise` كده في **5 دقائق**: هو ببساطة **متغير بنحط فيه الـ Promise الخاص بعملية تجديد الـ Access Token**.

### الأول: يعني إيه Promise هنا؟

لما نكتب:

```ts
const refreshAccessToken = async (): Promise<string> => {
  // ...
};
```

استدعاء:

```ts
refreshAccessToken();
```

لا يرجع الـ Token فورًا.

هو يرجع:

```text
Promise<string>
```

والـ Promise ده معناه:

> "أنا شغال على طلب الـ Refresh، ولما أخلص هديك الـ Access Token الجديد."

---

### طيب ليه عملنا `refreshPromise`؟

عملنا:

```ts
let refreshPromise: Promise<string> | null = null;
```

يعني المتغير ده ممكن يكون:

```text
null
```

أو:

```text
Promise<string>
```

في البداية:

```text
refreshPromise = null
```

معناها:

> مفيش Refresh شغال حاليًا.

لما أول Request يحصل له `401`:

```ts
refreshPromise = refreshAccessToken();
```

فتصبح:

```text
refreshPromise
      ↓
Promise شغال
      ↓
"أنا مستني Backend يطلعلي Access Token جديد"
```

---

### وهنا بقى السحر 🪄

لو Request تاني حصل له `401`:

بدل ما يعمل:

```ts
refreshAccessToken();
```

ويبدأ Refresh جديد، يعمل:

```ts
await refreshPromise;
```

معناها:

> استنى الـ Refresh اللي شغال بالفعل.

مثلاً:

```text
Request A → 401
     ↓
refreshPromise = Refresh #1
     ↓
     Backend
       ⏳

Request B → 401
     ↓
await refreshPromise
     ↓
ينتظر Refresh #1

Request C → 401
     ↓
await refreshPromise
     ↓
ينتظر Refresh #1
```

ولما Backend يرجع:

```text
NEW_ACCESS_TOKEN
```

كلهم ياخدوا نفس النتيجة:

```text
A → NEW_ACCESS_TOKEN
B → NEW_ACCESS_TOKEN
C → NEW_ACCESS_TOKEN
```

### إذن الفرق مهم جدًا

```ts
refreshAccessToken()
```

دي **Function** بتبدأ عملية Refresh.

أما:

```ts
refreshPromise
```

ده **المتغير اللي ماسك العملية الجارية نفسها**.

فكر فيها كده:

```text
refreshAccessToken()
        ↓
   يبدأ العملية
        ↓
   Promise
        ↓
refreshPromise
        ↓
أي Request تاني يستنى نفس الـ Promise
```

والسطر:

```ts
refreshPromise = null;
```

بعد انتهاء العملية معناه:

> خلاص الـ Refresh خلص، والمرّة الجاية لو حصل `401` ممكن نبدأ Refresh جديد.

**الخطوة التالية:** هشرح لك الفرق بين `refreshAccessToken()` و `refreshPromise` بمثال JavaScript صغير جدًا من غير Axios، عشان المفهوم يثبت نهائيًا.
</details>
refreshPromise
ده ايه بضبط
ChatGPT
افهم `refreshPromise` كده في **5 دقائق**: هو ببساطة **متغير بنحط فيه الـ Promise الخاص بعملية تجديد الـ Access Token**.

### الأول: يعني إيه Promise هنا؟

لما نكتب:

```ts
const refreshAccessToken = async (): Promise<string> => {
  // ...
};
```

استدعاء:

```ts
refreshAccessToken();
```

لا يرجع الـ Token فورًا.

هو يرجع:

```text
Promise<string>
```

والـ Promise ده معناه:

> "أنا شغال على طلب الـ Refresh، ولما أخلص هديك الـ Access Token الجديد."

---

### طيب ليه عملنا `refreshPromise`؟

عملنا:

```ts
let refreshPromise: Promise<string> | null = null;
```

يعني المتغير ده ممكن يكون:

```text
null
```

أو:

```text
Promise<string>
```

في البداية:

```text
refreshPromise = null
```

معناها:

> مفيش Refresh شغال حاليًا.

لما أول Request يحصل له `401`:

```ts
refreshPromise = refreshAccessToken();
```

فتصبح:

```text
refreshPromise
      ↓
Promise شغال
      ↓
"أنا مستني Backend يطلعلي Access Token جديد"
```

---

### وهنا بقى السحر 🪄

لو Request تاني حصل له `401`:

بدل ما يعمل:

```ts
refreshAccessToken();
```

ويبدأ Refresh جديد، يعمل:

```ts
await refreshPromise;
```

معناها:

> استنى الـ Refresh اللي شغال بالفعل.

مثلاً:

```text
Request A → 401
     ↓
refreshPromise = Refresh #1
     ↓
     Backend
       ⏳

Request B → 401
     ↓
await refreshPromise
     ↓
ينتظر Refresh #1

Request C → 401
     ↓
await refreshPromise
     ↓
ينتظر Refresh #1
```

ولما Backend يرجع:

```text
NEW_ACCESS_TOKEN
```

كلهم ياخدوا نفس النتيجة:

```text
A → NEW_ACCESS_TOKEN
B → NEW_ACCESS_TOKEN
C → NEW_ACCESS_TOKEN
```

### إذن الفرق مهم جدًا

```ts
refreshAccessToken()
```

دي **Function** بتبدأ عملية Refresh.

أما:

```ts
refreshPromise
```

ده **المتغير اللي ماسك العملية الجارية نفسها**.

فكر فيها كده:

```text
refreshAccessToken()
        ↓
   يبدأ العملية
        ↓
   Promise
        ↓
refreshPromise
        ↓
أي Request تاني يستنى نفس الـ Promise
```

والسطر:

```ts
refreshPromise = null;
```

بعد انتهاء العملية معناه:

> خلاص الـ Refresh خلص، والمرّة الجاية لو حصل `401` ممكن نبدأ Refresh جديد.

**الخطوة التالية:** هشرح لك الفرق بين `refreshAccessToken()` و `refreshPromise` بمثال JavaScript صغير جدًا من غير Axios، عشان المفهوم يثبت نهائيًا.
<details>
<summary>ياريت</summary>
user
ياريت
ChatGPT
خلينا نثبت الفرق بمثال JavaScript فقط في **5 دقائق**، من غير Axios خالص.

## 1. عندنا Function بتعمل عملية طويلة

تخيل عندنا:

```ts
const getData = async (): Promise<string> => {
  await new Promise((resolve) => setTimeout(resolve, 3000));

  return "DATA";
};
```

الـ Function دي بتحتاج 3 ثواني وبعدين ترجع `"DATA"`.

لما تعمل:

```ts
getData();
```

أنت **بدأت العملية**.

والـ Return بتاعها:

```ts
Promise<string>
```

---

## 2. هنا بقى `promise`

ممكن نمسك الـ Promise في متغير:

```ts
const promise = getData();
```

دلوقتي:

```text id="9xk31c"
getData()
   ↓
بدأت العملية
   ↓
Promise
   ↓
promise
```

لاحظ الفرق:

```ts
getData()
```

دي **Function بتبدأ العملية**.

بينما:

```ts
promise
```

ده **الـ Promise الناتج عن العملية الجارية**.

---

## 3. لو اتنين عايزين نفس العملية

```ts
const promise = getData();
```

العملية بدأت مرة واحدة.

دلوقتي:

```ts
const result1 = await promise;
const result2 = await promise;
```

الاتنين مستنيين **نفس الـ Promise**.

مش بنعمل:

```ts
getData();
getData();
```

لأن كده بدأنا عمليتين منفصلتين.

```text id="m4n8p2"
❌ غلط

getData() → عملية 1
getData() → عملية 2
```

لكن:

```text id="r7k2w5"
✅ صح

const promise = getData()

        ↓

Request A ─┐
Request B ─┼→ await promise
Request C ─┘

        ↓

   عملية واحدة
```

---

## 4. دلوقتي نرجع للـ Token Refresh

عندنا:

```ts
const refreshAccessToken = async (): Promise<string> => {
  // request للـ Backend

  return "NEW_ACCESS_TOKEN";
};
```

دي **Function**.

ولما نناديها:

```ts
refreshAccessToken();
```

بتبدأ عملية الـ Refresh وترجع:

```ts
Promise<string>
```

نمسك الـ Promise:

```ts
let refreshPromise: Promise<string> | null = null;
```

وبعدين:

```ts
refreshPromise = refreshAccessToken();
```

بقى عندنا:

```text id="f2j8k4"
refreshAccessToken()
        ↓
يبدأ Refresh
        ↓
Promise
        ↓
refreshPromise
```

---

## 5. وهنا تظهر أهمية الـ Queue

أول Request:

```ts
if (!refreshPromise) {
  refreshPromise = refreshAccessToken();
}
```

بدأ Refresh واحد.

Request تاني:

```ts
await refreshPromise;
```

يستنى.

Request تالت:

```ts
await refreshPromise;
```

يستنى.

فإحنا عندنا:

```text id="z8c4m1"
              Refresh
                 │
                 ▼
          refreshPromise
          ┌──────┼──────┐
          ▼      ▼      ▼
          A      B      C
        await  await  await
          │      │      │
          └──────┼──────┘
                 ▼
          NEW_ACCESS_TOKEN
```

**Refresh واحد فقط**، وكل الـ Requests تستنى نفس الـ Promise.

### احفظ الجملة دي:

> **`refreshAccessToken()` تبدأ عملية الـ Refresh، أما `refreshPromise` فهو الـ Promise اللي بيمثل عملية الـ Refresh الجارية، والـ Requests التانية بتعمل `await` لنفس الـ Promise بدل ما تبدأ Refresh جديد.**

**الخطوة التالية:** اكتبلي من نفسك سطرين فقط: تعريف `refreshPromise`، والسطر اللي بيخليه يمسك عملية `refreshAccessToken()`.
</details>
ياريت
ChatGPT
خلينا نثبت الفرق بمثال JavaScript فقط في **5 دقائق**، من غير Axios خالص.

## 1. عندنا Function بتعمل عملية طويلة

تخيل عندنا:

```ts
const getData = async (): Promise<string> => {
  await new Promise((resolve) => setTimeout(resolve, 3000));

  return "DATA";
};
```

الـ Function دي بتحتاج 3 ثواني وبعدين ترجع `"DATA"`.

لما تعمل:

```ts
getData();
```

أنت **بدأت العملية**.

والـ Return بتاعها:

```ts
Promise<string>
```

---

## 2. هنا بقى `promise`

ممكن نمسك الـ Promise في متغير:

```ts
const promise = getData();
```

دلوقتي:

```text id="9xk31c"
getData()
   ↓
بدأت العملية
   ↓
Promise
   ↓
promise
```

لاحظ الفرق:

```ts
getData()
```

دي **Function بتبدأ العملية**.

بينما:

```ts
promise
```

ده **الـ Promise الناتج عن العملية الجارية**.

---

## 3. لو اتنين عايزين نفس العملية

```ts
const promise = getData();
```

العملية بدأت مرة واحدة.

دلوقتي:

```ts
const result1 = await promise;
const result2 = await promise;
```

الاتنين مستنيين **نفس الـ Promise**.

مش بنعمل:

```ts
getData();
getData();
```

لأن كده بدأنا عمليتين منفصلتين.

```text id="m4n8p2"
❌ غلط

getData() → عملية 1
getData() → عملية 2
```

لكن:

```text id="r7k2w5"
✅ صح

const promise = getData()

        ↓

Request A ─┐
Request B ─┼→ await promise
Request C ─┘

        ↓

   عملية واحدة
```

---

## 4. دلوقتي نرجع للـ Token Refresh

عندنا:

```ts
const refreshAccessToken = async (): Promise<string> => {
  // request للـ Backend

  return "NEW_ACCESS_TOKEN";
};
```

دي **Function**.

ولما نناديها:

```ts
refreshAccessToken();
```

بتبدأ عملية الـ Refresh وترجع:

```ts
Promise<string>
```

نمسك الـ Promise:

```ts
let refreshPromise: Promise<string> | null = null;
```

وبعدين:

```ts
refreshPromise = refreshAccessToken();
```

بقى عندنا:

```text id="f2j8k4"
refreshAccessToken()
        ↓
يبدأ Refresh
        ↓
Promise
        ↓
refreshPromise
```

---

## 5. وهنا تظهر أهمية الـ Queue

أول Request:

```ts
if (!refreshPromise) {
  refreshPromise = refreshAccessToken();
}
```

بدأ Refresh واحد.

Request تاني:

```ts
await refreshPromise;
```

يستنى.

Request تالت:

```ts
await refreshPromise;
```

يستنى.

فإحنا عندنا:

```text id="z8c4m1"
              Refresh
                 │
                 ▼
          refreshPromise
          ┌──────┼──────┐
          ▼      ▼      ▼
          A      B      C
        await  await  await
          │      │      │
          └──────┼──────┘
                 ▼
          NEW_ACCESS_TOKEN
```

**Refresh واحد فقط**، وكل الـ Requests تستنى نفس الـ Promise.

### احفظ الجملة دي:

> **`refreshAccessToken()` تبدأ عملية الـ Refresh، أما `refreshPromise` فهو الـ Promise اللي بيمثل عملية الـ Refresh الجارية، والـ Requests التانية بتعمل `await` لنفس الـ Promise بدل ما تبدأ Refresh جديد.**

**الخطوة التالية:** اكتبلي من نفسك سطرين فقط: تعريف `refreshPromise`، والسطر اللي بيخليه يمسك عملية `refreshAccessToken()`.
<details>
<summary>حاسس ان الموضوع صعب هنا مش فاهم اووي الجزئيه دي وتعبت منها وعاوز افهما</summary>
user
حاسس ان الموضوع صعب هنا مش فاهم اووي الجزئيه دي وتعبت منها وعاوز افهما
ChatGPT
خد **10 دقائق فقط** ونفصل الـ `Axios` والـ `Interceptor` خالص، لأن المشكلة مش فيك، المشكلة إننا دخلنا في `Promise` و`Concurrency` فوق بعض.

### 🗺️ Roadmap

```text
SECTION 4: Interceptors 🟢
├─ 4.1 Interceptor              ✅
├─ 4.2 Request Interceptor      ✅
├─ 4.3 Response Interceptor     ✅
├─ 4.4 Authentication           ✅
├─ 4.5 401 Handling             ✅
└─ 4.6 Token Refresh + Queue    🟡
     ├─ Refresh Token            ✅
     ├─ error.config             ✅
     ├─ Retry                    ✅
     ├─ Infinite Loop            ✅
     └─ Queue                    🟡 ← هنثبتها فقط
```

## انسَ الـ Code دلوقتي

تخيل إن **الأسانسير عطلان**.

أنت و4 أشخاص واقفين قدامه:

```text
أنت
أحمد
محمد
علي
خالد
```

حد قال:

> أنا هروح أصلح الأسانسير.

هل كل واحد فيكم يروح يصلحه؟ ❌

لا.

واحد بس يصلحه:

```text
أحمد
 ↓
يصلح الأسانسير
```

والباقي:

```text
أنت    → تستنى
محمد   → يستنى
علي    → يستنى
خالد   → يستنى
```

لما الأسانسير يشتغل:

```text
الأسانسير جاهز ✅
       ↓
الكل يستخدمه
```

**دي هي فكرة الـ Queue بالضبط.**

---

# نرجع للـ Token

عندنا 5 Requests:

```text
Request A
Request B
Request C
Request D
Request E
```

كلهم استخدموا:

```text
OLD_ACCESS_TOKEN
```

والـ Token انتهى.

فالـ Backend يقول:

```text
A → 401
B → 401
C → 401
D → 401
E → 401
```

إحنا مش عايزين:

```text
A → Refresh
B → Refresh
C → Refresh
D → Refresh
E → Refresh
```

لأن ده **5 Refresh Requests**.

عايزين:

```text
A → Refresh
B → انتظار
C → انتظار
D → انتظار
E → انتظار
```

وبعد ما A يجيب:

```text
NEW_ACCESS_TOKEN
```

نقول:

```text
B → خد الـ Token الجديد وأعد طلبك
C → خد الـ Token الجديد وأعد طلبك
D → خد الـ Token الجديد وأعد طلبك
E → خد الـ Token الجديد وأعد طلبك
```

---

# طيب `refreshPromise` إيه بقى؟

دي النقطة اللي كانت ملخبطة عليك.

**`refreshPromise` مش هو الـ Refresh Token.**

خالص.

عندنا 3 حاجات مختلفة:

```text
Refresh Token
↓
مفتاح نستخدمه للحصول على Access Token جديد


refreshAccessToken()
↓
Function بتعمل عملية الـ Refresh


refreshPromise
↓
متغير ماسك العملية اللي شغالة حاليًا
```

احفظ الثلاثة دول فقط.

---

## مثال أبسط جدًا

عندي:

```ts
const makeTea = async (): Promise<string> => {
  // اعمل شاي
  return "Tea";
};
```

دي Function.

```text
makeTea()
```

يعني:

> ابدأ اعمل الشاي.

لو كتبت:

```ts
const teaPromise = makeTea();
```

معناه:

> بدأت أعمل الشاي، وأنا ماسك الـ Promise بتاع العملية دي في `teaPromise`.

حد تاني عايز الشاي؟

مش هقول له:

```ts
makeTea();
```

لأن ده هيبدأ **شاي جديد**.

هقوله:

```ts
await teaPromise;
```

يعني:

> استنى الشاي اللي بيتعمل بالفعل.

---

# وده بالضبط Token Refresh

```ts
let refreshPromise: Promise<string> | null = null;
```

في البداية:

```text
refreshPromise = null
```

يعني:

> مفيش Refresh شغال.

أول `401`:

```ts
refreshPromise = refreshAccessToken();
```

يعني:

```text
ابدأ Refresh
      ↓
امسك العملية في refreshPromise
```

Request تاني يحصل له `401`:

```ts
await refreshPromise;
```

يعني:

> متعملش Refresh جديد، استنى اللي شغال.

---

# الصورة اللي عايزك تحفظها

```text
             401
              ↓
       هل Refresh شغال؟
          /         \
        لا           آه
        ↓             ↓
   ابدأ Refresh     استنى
        ↓             ↓
        └──────┬──────┘
               ↓
         New Access Token
               ↓
          Retry Requests
```

**مش مطلوب منك تحفظ الكود دلوقتي.**

أنا شايف إن لو دخلنا في `resolve / reject / failedQueue` دلوقتي هنزود الحمل عليك، وده مش مفيد.

إحنا هنثبت الأول **فكرة إن عدة Requests تشترك في نفس عملية Refresh**، وبعدها نرجع للكود.

### اختبار سريع جدًا

عندنا:

```text
A → 401
B → 401
C → 401
```

هل نعمل **3 Refresh Requests**، ولا **Refresh واحد وB وC يستنوا نتيجته**؟

**الخطوة التالية:** جاوبني بكلمة واحدة: `3` أو `1`.
</details>
حاسس ان الموضوع صعب هنا مش فاهم اووي الجزئيه دي وتعبت منها وعاوز افهما
ChatGPT
خد **10 دقائق فقط** ونفصل الـ `Axios` والـ `Interceptor` خالص، لأن المشكلة مش فيك، المشكلة إننا دخلنا في `Promise` و`Concurrency` فوق بعض.

### 🗺️ Roadmap

```text
SECTION 4: Interceptors 🟢
├─ 4.1 Interceptor              ✅
├─ 4.2 Request Interceptor      ✅
├─ 4.3 Response Interceptor     ✅
├─ 4.4 Authentication           ✅
├─ 4.5 401 Handling             ✅
└─ 4.6 Token Refresh + Queue    🟡
     ├─ Refresh Token            ✅
     ├─ error.config             ✅
     ├─ Retry                    ✅
     ├─ Infinite Loop            ✅
     └─ Queue                    🟡 ← هنثبتها فقط
```

## انسَ الـ Code دلوقتي

تخيل إن **الأسانسير عطلان**.

أنت و4 أشخاص واقفين قدامه:

```text
أنت
أحمد
محمد
علي
خالد
```

حد قال:

> أنا هروح أصلح الأسانسير.

هل كل واحد فيكم يروح يصلحه؟ ❌

لا.

واحد بس يصلحه:

```text
أحمد
 ↓
يصلح الأسانسير
```

والباقي:

```text
أنت    → تستنى
محمد   → يستنى
علي    → يستنى
خالد   → يستنى
```

لما الأسانسير يشتغل:

```text
الأسانسير جاهز ✅
       ↓
الكل يستخدمه
```

**دي هي فكرة الـ Queue بالضبط.**

---

# نرجع للـ Token

عندنا 5 Requests:

```text
Request A
Request B
Request C
Request D
Request E
```

كلهم استخدموا:

```text
OLD_ACCESS_TOKEN
```

والـ Token انتهى.

فالـ Backend يقول:

```text
A → 401
B → 401
C → 401
D → 401
E → 401
```

إحنا مش عايزين:

```text
A → Refresh
B → Refresh
C → Refresh
D → Refresh
E → Refresh
```

لأن ده **5 Refresh Requests**.

عايزين:

```text
A → Refresh
B → انتظار
C → انتظار
D → انتظار
E → انتظار
```

وبعد ما A يجيب:

```text
NEW_ACCESS_TOKEN
```

نقول:

```text
B → خد الـ Token الجديد وأعد طلبك
C → خد الـ Token الجديد وأعد طلبك
D → خد الـ Token الجديد وأعد طلبك
E → خد الـ Token الجديد وأعد طلبك
```

---

# طيب `refreshPromise` إيه بقى؟

دي النقطة اللي كانت ملخبطة عليك.

**`refreshPromise` مش هو الـ Refresh Token.**

خالص.

عندنا 3 حاجات مختلفة:

```text
Refresh Token
↓
مفتاح نستخدمه للحصول على Access Token جديد


refreshAccessToken()
↓
Function بتعمل عملية الـ Refresh


refreshPromise
↓
متغير ماسك العملية اللي شغالة حاليًا
```

احفظ الثلاثة دول فقط.

---

## مثال أبسط جدًا

عندي:

```ts
const makeTea = async (): Promise<string> => {
  // اعمل شاي
  return "Tea";
};
```

دي Function.

```text
makeTea()
```

يعني:

> ابدأ اعمل الشاي.

لو كتبت:

```ts
const teaPromise = makeTea();
```

معناه:

> بدأت أعمل الشاي، وأنا ماسك الـ Promise بتاع العملية دي في `teaPromise`.

حد تاني عايز الشاي؟

مش هقول له:

```ts
makeTea();
```

لأن ده هيبدأ **شاي جديد**.

هقوله:

```ts
await teaPromise;
```

يعني:

> استنى الشاي اللي بيتعمل بالفعل.

---

# وده بالضبط Token Refresh

```ts
let refreshPromise: Promise<string> | null = null;
```

في البداية:

```text
refreshPromise = null
```

يعني:

> مفيش Refresh شغال.

أول `401`:

```ts
refreshPromise = refreshAccessToken();
```

يعني:

```text
ابدأ Refresh
      ↓
امسك العملية في refreshPromise
```

Request تاني يحصل له `401`:

```ts
await refreshPromise;
```

يعني:

> متعملش Refresh جديد، استنى اللي شغال.

---

# الصورة اللي عايزك تحفظها

```text
             401
              ↓
       هل Refresh شغال؟
          /         \
        لا           آه
        ↓             ↓
   ابدأ Refresh     استنى
        ↓             ↓
        └──────┬──────┘
               ↓
         New Access Token
               ↓
          Retry Requests
```

**مش مطلوب منك تحفظ الكود دلوقتي.**

أنا شايف إن لو دخلنا في `resolve / reject / failedQueue` دلوقتي هنزود الحمل عليك، وده مش مفيد.

إحنا هنثبت الأول **فكرة إن عدة Requests تشترك في نفس عملية Refresh**، وبعدها نرجع للكود.

### اختبار سريع جدًا

عندنا:

```text
A → 401
B → 401
C → 401
```

هل نعمل **3 Refresh Requests**، ولا **Refresh واحد وB وC يستنوا نتيجته**؟

**الخطوة التالية:** جاوبني بكلمة واحدة: `3` أو `1`.
<details>
<summary>١</summary>
user
١ 
ودي انا فاهمها
ChatGPT
ممتاز. بما إن **فكرة الـ Refresh الواحد والـ Requests اللي تستنى واضحة عندك**، مش هنضيع وقت في شرحها تاني.

المشكلة الحقيقية كانت: **ليه محتاجين `refreshPromise` عشان ننفذ الفكرة دي؟**

خلينا نشيل الـ Queue مؤقتًا ونفهم السطر ده وحده في **5 دقائق**:

```ts
let refreshPromise: Promise<string> | null = null;
```

معناه ببساطة:

> عندي متغير اسمه `refreshPromise`، ممكن يكون فاضي `null`، أو ممكن يمسك عملية الـ Refresh الحالية.

في البداية:

```ts
refreshPromise = null;
```

يعني:

```text
مفيش Refresh شغال
```

لما أول Request يعمل `401`:

```ts
refreshPromise = refreshAccessToken();
```

بقى:

```text
refreshPromise
      ↓
عملية Refresh شغالة
```

Request تاني يعمل `401`:

```ts
await refreshPromise;
```

معناه:

> **استنى العملية اللي موجودة بالفعل.**

بس كده.

### الفرق في سطرين

```ts
refreshAccessToken()
```

= **ابدأ Refresh جديد**

```ts
await refreshPromise
```

= **استنى Refresh شغال بالفعل**

وده هو كل سر `refreshPromise`.

ومش لازم تحفظ أي كود أكتر من كده دلوقتي.

**الخطوة التالية:** اكتبلي بإيدك السطرين دول فقط، وأنا هقولك إذا فهمت `refreshPromise` فعلاً قبل ما نرجع للكود الكبير.
</details>
١ 
ودي انا فاهمها
ChatGPT
ممتاز. بما إن **فكرة الـ Refresh الواحد والـ Requests اللي تستنى واضحة عندك**، مش هنضيع وقت في شرحها تاني.

المشكلة الحقيقية كانت: **ليه محتاجين `refreshPromise` عشان ننفذ الفكرة دي؟**

خلينا نشيل الـ Queue مؤقتًا ونفهم السطر ده وحده في **5 دقائق**:

```ts
let refreshPromise: Promise<string> | null = null;
```

معناه ببساطة:

> عندي متغير اسمه `refreshPromise`، ممكن يكون فاضي `null`، أو ممكن يمسك عملية الـ Refresh الحالية.

في البداية:

```ts
refreshPromise = null;
```

يعني:

```text
مفيش Refresh شغال
```

لما أول Request يعمل `401`:

```ts
refreshPromise = refreshAccessToken();
```

بقى:

```text
refreshPromise
      ↓
عملية Refresh شغالة
```

Request تاني يعمل `401`:

```ts
await refreshPromise;
```

معناه:

> **استنى العملية اللي موجودة بالفعل.**

بس كده.

### الفرق في سطرين

```ts
refreshAccessToken()
```

= **ابدأ Refresh جديد**

```ts
await refreshPromise
```

= **استنى Refresh شغال بالفعل**

وده هو كل سر `refreshPromise`.

ومش لازم تحفظ أي كود أكتر من كده دلوقتي.

**الخطوة التالية:** اكتبلي بإيدك السطرين دول فقط، وأنا هقولك إذا فهمت `refreshPromise` فعلاً قبل ما نرجع للكود الكبير.
<details>
<summary>refreshPromise = refreshAccessToken()</summary>
user
refreshPromise = refreshAccessToken()
await refreshPromise
ChatGPT
أيوه، **بالضبط**. كده أنت ماسك الفكرة الأساسية صح.

```ts
refreshPromise = refreshAccessToken();
```

معناها:

> ابدأ عملية الـ Refresh، وخلي الـ Promise بتاعها موجود في `refreshPromise`.

وبعدين:

```ts
await refreshPromise;
```

معناها:

> استنى العملية دي تخلص.

### بص عليها كده

```text
refreshAccessToken()
       ↓
بدأ Refresh
       ↓
Promise
       ↓
refreshPromise
       ↓
await
       ↓
استنى
       ↓
NEW_ACCESS_TOKEN
```

والفرق المهم جدًا:

```ts
refreshAccessToken();
```

🔥 **بيبدأ عملية جديدة**

بينما:

```ts
await refreshPromise;
```

⏳ **بيستنى العملية الموجودة بالفعل**

وده بالضبط اللي بيخلينا نعمل **Refresh واحد فقط** حتى لو عندنا 15 Request حصل لهم `401`.

أنت كده فهمت أصعب جزء في الفكرة. 👍

**الخطوة التالية:** نكتب الـ `refreshAccessToken()` نفسها سطر سطر، وبعدها نركبها مع `refreshPromise` في الـ Interceptor.
</details>
refreshPromise = refreshAccessToken()
await refreshPromise
ChatGPT
أيوه، **بالضبط**. كده أنت ماسك الفكرة الأساسية صح.

```ts
refreshPromise = refreshAccessToken();
```

معناها:

> ابدأ عملية الـ Refresh، وخلي الـ Promise بتاعها موجود في `refreshPromise`.

وبعدين:

```ts
await refreshPromise;
```

معناها:

> استنى العملية دي تخلص.

### بص عليها كده

```text
refreshAccessToken()
       ↓
بدأ Refresh
       ↓
Promise
       ↓
refreshPromise
       ↓
await
       ↓
استنى
       ↓
NEW_ACCESS_TOKEN
```

والفرق المهم جدًا:

```ts
refreshAccessToken();
```

🔥 **بيبدأ عملية جديدة**

بينما:

```ts
await refreshPromise;
```

⏳ **بيستنى العملية الموجودة بالفعل**

وده بالضبط اللي بيخلينا نعمل **Refresh واحد فقط** حتى لو عندنا 15 Request حصل لهم `401`.

أنت كده فهمت أصعب جزء في الفكرة. 👍

**الخطوة التالية:** نكتب الـ `refreshAccessToken()` نفسها سطر سطر، وبعدها نركبها مع `refreshPromise` في الـ Interceptor.
<details>
<summary>Promise</summary>
user
Promise
هو الظاهر عندي مشكله في كلمة دي
ChatGPT
آه، **دي غالبًا هي العقدة الأساسية فعلًا**. خلينا نوقف `Token Refresh` خالص لمدة **10 دقائق** ونفهم `Promise` من الصفر، لأن لو فهمت `Promise`، موضوع `refreshPromise` هيبقى سهل جدًا.

## 1. Promise يعني إيه؟

ببساطة جدًا:

**Promise = وعد إن نتيجة هتجيلك بعد شوية.**

مثال:

```ts
const result = fetch("/users");
```

الـ `fetch` مش بيرجعلك بيانات المستخدمين فورًا.

هو بيقولك:

> "أنا بدأت أطلب البيانات، ولما أخلص هديك النتيجة."

فاللي رجعلك هو:

```ts
Promise
```

---

## 2. تخيل طلب بيتزا 🍕

أنت طلبت بيتزا:

```text
أنت
 ↓
اطلب بيتزا
 ↓
المطعم يقول:
"حاضر، استنى"
```

الـ `"حاضر، استنى"` دي هي فكرة الـ **Promise**.

لسه مفيش بيتزا.

لكن فيه **وعد** إن النتيجة هتيجي.

بعد شوية:

```text
🍕 وصلت
```

---

## 3. في JavaScript

مثلاً:

```ts
const promise = new Promise((resolve) => {
  setTimeout(() => {
    resolve("Pizza");
  }, 3000);
});
```

هنا:

```text
promise
 ↓
استنى 3 ثواني
 ↓
"Pizza"
```

والـ Promise له 3 حالات أساسية:

```text
Pending
↓
لسه مستني

Fulfilled
↓
نجح وجاب النتيجة

Rejected
↓
فشل
```

---

## 4. طيب `await` بيعمل إيه؟

لما تكتب:

```ts
const result = await promise;
```

أنت بتقول:

> **استنى الـ Promise ده لحد ما يخلص، وبعدها اديني النتيجة.**

مثلاً:

```ts
const promise = getData();

const result = await promise;
```

لو `getData()` رجعت:

```text
Promise<string>
```

فالـ `await` يستنى لحد ما النتيجة الفعلية توصل:

```text
Promise<string>
      ↓ await
string
```

---

# 5. دلوقتي نرجع لـ `refreshPromise`

شوف الكود ده:

```ts
const refreshPromise = refreshAccessToken();
```

الـ `refreshAccessToken()` بتعمل Request للـ Backend.

إذن:

```text
refreshAccessToken()
        ↓
Request للـ Backend
        ↓
Promise
```

فنحن مسكنا الـ Promise في:

```ts
refreshPromise
```

يعني:

> **`refreshPromise` هو الوعد/العملية اللي مستنيين منها الـ Access Token الجديد.**

---

## 6. وهنا هتفهم ليه استخدمناه

عندنا:

```text
Request A → 401
```

نعمل:

```ts
refreshPromise = refreshAccessToken();
```

يعني:

```text
بدأ Refresh
      ↓
Promise شغال
      ↓
refreshPromise
```

وبعدين:

```text
Request B → 401
```

مش عايزين نعمل Refresh جديد.

فنقول:

```ts
await refreshPromise;
```

يعني:

> **استنى نفس العملية اللي بدأت بالفعل.**

---

## أهم فرق تحفظه

```ts
refreshAccessToken()
```

**ابدأ عملية جديدة.**

```ts
refreshPromise
```

**العملية اللي بدأت بالفعل وممسوك الـ Promise بتاعها.**

```ts
await refreshPromise
```

**استنى العملية دي لحد ما تخلص.**

---

### مثال صغير جدًا

```ts
const promise = getData();

console.log("A");

const data = await promise;

console.log("B");
```

الترتيب:

```text
getData()
   ↓
Promise
   ↓
console.log("A")
   ↓
await
   ↓
استنى
   ↓
البيانات وصلت
   ↓
console.log("B")
```

فـ `await` **مش بيخلق Promise**.

هو فقط **ينتظر Promise موجود**.

ودي نقطة مهمة جدًا.

**الخطوة التالية:** نعمل مثال `Promise` واحد من غير Axios ولا Tokens، فيه شخصين بيستَنوا نفس الـ Promise، عشان تشوف بعينك ليه `refreshPromise` مهم.
</details>
Promise
هو الظاهر عندي مشكله في كلمة دي
ChatGPT
آه، **دي غالبًا هي العقدة الأساسية فعلًا**. خلينا نوقف `Token Refresh` خالص لمدة **10 دقائق** ونفهم `Promise` من الصفر، لأن لو فهمت `Promise`، موضوع `refreshPromise` هيبقى سهل جدًا.

## 1. Promise يعني إيه؟

ببساطة جدًا:

**Promise = وعد إن نتيجة هتجيلك بعد شوية.**

مثال:

```ts
const result = fetch("/users");
```

الـ `fetch` مش بيرجعلك بيانات المستخدمين فورًا.

هو بيقولك:

> "أنا بدأت أطلب البيانات، ولما أخلص هديك النتيجة."

فاللي رجعلك هو:

```ts
Promise
```

---

## 2. تخيل طلب بيتزا 🍕

أنت طلبت بيتزا:

```text
أنت
 ↓
اطلب بيتزا
 ↓
المطعم يقول:
"حاضر، استنى"
```

الـ `"حاضر، استنى"` دي هي فكرة الـ **Promise**.

لسه مفيش بيتزا.

لكن فيه **وعد** إن النتيجة هتيجي.

بعد شوية:

```text
🍕 وصلت
```

---

## 3. في JavaScript

مثلاً:

```ts
const promise = new Promise((resolve) => {
  setTimeout(() => {
    resolve("Pizza");
  }, 3000);
});
```

هنا:

```text
promise
 ↓
استنى 3 ثواني
 ↓
"Pizza"
```

والـ Promise له 3 حالات أساسية:

```text
Pending
↓
لسه مستني

Fulfilled
↓
نجح وجاب النتيجة

Rejected
↓
فشل
```

---

## 4. طيب `await` بيعمل إيه؟

لما تكتب:

```ts
const result = await promise;
```

أنت بتقول:

> **استنى الـ Promise ده لحد ما يخلص، وبعدها اديني النتيجة.**

مثلاً:

```ts
const promise = getData();

const result = await promise;
```

لو `getData()` رجعت:

```text
Promise<string>
```

فالـ `await` يستنى لحد ما النتيجة الفعلية توصل:

```text
Promise<string>
      ↓ await
string
```

---

# 5. دلوقتي نرجع لـ `refreshPromise`

شوف الكود ده:

```ts
const refreshPromise = refreshAccessToken();
```

الـ `refreshAccessToken()` بتعمل Request للـ Backend.

إذن:

```text
refreshAccessToken()
        ↓
Request للـ Backend
        ↓
Promise
```

فنحن مسكنا الـ Promise في:

```ts
refreshPromise
```

يعني:

> **`refreshPromise` هو الوعد/العملية اللي مستنيين منها الـ Access Token الجديد.**

---

## 6. وهنا هتفهم ليه استخدمناه

عندنا:

```text
Request A → 401
```

نعمل:

```ts
refreshPromise = refreshAccessToken();
```

يعني:

```text
بدأ Refresh
      ↓
Promise شغال
      ↓
refreshPromise
```

وبعدين:

```text
Request B → 401
```

مش عايزين نعمل Refresh جديد.

فنقول:

```ts
await refreshPromise;
```

يعني:

> **استنى نفس العملية اللي بدأت بالفعل.**

---

## أهم فرق تحفظه

```ts
refreshAccessToken()
```

**ابدأ عملية جديدة.**

```ts
refreshPromise
```

**العملية اللي بدأت بالفعل وممسوك الـ Promise بتاعها.**

```ts
await refreshPromise
```

**استنى العملية دي لحد ما تخلص.**

---

### مثال صغير جدًا

```ts
const promise = getData();

console.log("A");

const data = await promise;

console.log("B");
```

الترتيب:

```text
getData()
   ↓
Promise
   ↓
console.log("A")
   ↓
await
   ↓
استنى
   ↓
البيانات وصلت
   ↓
console.log("B")
```

فـ `await` **مش بيخلق Promise**.

هو فقط **ينتظر Promise موجود**.

ودي نقطة مهمة جدًا.

**الخطوة التالية:** نعمل مثال `Promise` واحد من غير Axios ولا Tokens، فيه شخصين بيستَنوا نفس الـ Promise، عشان تشوف بعينك ليه `refreshPromise` مهم.
<details>
<summary>ملخص أنا فين؟ أنا دلوقتي عندي مثلاً متغير اسمه X. متغير X ده بي equal function اسمها MT. الـMT دي equal متغير بترجع ST. خلاص كده؟ يبقى أنا كده عندي اللي بيطلع من الدالة اسمها T أحطه في X. أنا دلوقتي لو عايز أعمل T تاني، بدل ما أقوله X أقوله output X. يعني استني ده صح كده؟</summary>
user
ملخص أنا فين؟ أنا دلوقتي عندي مثلاً متغير اسمه X. متغير X ده بي equal function اسمها MT. الـMT دي equal متغير بترجع ST. خلاص كده؟ يبقى أنا كده عندي اللي بيطلع من الدالة اسمها T أحطه في X. أنا دلوقتي لو عايز أعمل T تاني، بدل ما أقوله X أقوله output X. يعني استني ده صح كده؟
ChatGPT
أيوه، **أنت قريب جدًا**، بس فيه نقطة واحدة محتاجة تصحيح عشان الصورة تثبت.

خد المثال ده:

```ts
const X = MT();
```

لو `MT()` بترجع `Promise<string>`، يبقى:

```text id="7wzj2e"
MT()
 ↓
Promise<string>
 ↓
X
```

إذن **X ماسك الـ Promise**.

لكن لما تقول:

```ts
await X
```

أنت **مش بتعمل T تاني**.

أنت بتقول:

> استنى الـ Promise الموجود جوه X لحد ما يخلص، واديني النتيجة.

يعني:

```text id="q9m5k4"
X
↓
Promise<string>
↓ await
string
```

مثلاً:

```ts
const X = MT();

const result = await X;
```

هنا:

```text id="w8h2p6"
X       = Promise<string>
result  = string
```

### وربطها بالـ Refresh

```ts
const refreshPromise = refreshAccessToken();
```

يبقى:

```text id="f2s7v9"
refreshAccessToken()
        ↓
 Promise<string>
        ↓
refreshPromise
```

وبعدين:

```ts
const newToken = await refreshPromise;
```

يبقى:

```text id="3kq8az"
refreshPromise
      ↓
Promise<string>
      ↓ await
newToken
      ↓
string
```

**فأنت صح في أهم نقطة:** `X` مش هو الـ Token نفسه، `X` هو **Promise مستني منه الـ Token**.

**الخطوة التالية:** اكتبلي بس: `X = Promise` و `await X = ؟` وأنا أتأكد إن الجزئية دي ثبتت.
</details>
ملخص أنا فين؟ أنا دلوقتي عندي مثلاً متغير اسمه X. متغير X ده بي equal function اسمها MT. الـMT دي equal متغير بترجع ST. خلاص كده؟ يبقى أنا كده عندي اللي بيطلع من الدالة اسمها T أحطه في X. أنا دلوقتي لو عايز أعمل T تاني، بدل ما أقوله X أقوله output X. يعني استني ده صح كده؟
ChatGPT
أيوه، **أنت قريب جدًا**، بس فيه نقطة واحدة محتاجة تصحيح عشان الصورة تثبت.

خد المثال ده:

```ts
const X = MT();
```

لو `MT()` بترجع `Promise<string>`، يبقى:

```text id="7wzj2e"
MT()
 ↓
Promise<string>
 ↓
X
```

إذن **X ماسك الـ Promise**.

لكن لما تقول:

```ts
await X
```

أنت **مش بتعمل T تاني**.

أنت بتقول:

> استنى الـ Promise الموجود جوه X لحد ما يخلص، واديني النتيجة.

يعني:

```text id="q9m5k4"
X
↓
Promise<string>
↓ await
string
```

مثلاً:

```ts
const X = MT();

const result = await X;
```

هنا:

```text id="w8h2p6"
X       = Promise<string>
result  = string
```

### وربطها بالـ Refresh

```ts
const refreshPromise = refreshAccessToken();
```

يبقى:

```text id="f2s7v9"
refreshAccessToken()
        ↓
 Promise<string>
        ↓
refreshPromise
```

وبعدين:

```ts
const newToken = await refreshPromise;
```

يبقى:

```text id="3kq8az"
refreshPromise
      ↓
Promise<string>
      ↓ await
newToken
      ↓
string
```

**فأنت صح في أهم نقطة:** `X` مش هو الـ Token نفسه، `X` هو **Promise مستني منه الـ Token**.

**الخطوة التالية:** اكتبلي بس: `X = Promise` و `await X = ؟` وأنا أتأكد إن الجزئية دي ثبتت.
<details>
<summary>سيبك بقى من البروموستكست دي.</summary>
user
سيبك بقى من البروموستكست دي.
</details>
سيبك بقى من البروموستكست دي.
<details>
<summary>Promise&lt;string</summary>
user
Promise<string
اقصد سيبك من دي
ChatGPT
تمام، **نرمي `Promise<string>` من دماغنا خالص دلوقتي**. خلينا نفهم الفكرة كأنها متغير عادي، في **3 دقائق**.

افترض عندك:

```ts
const X = MT();
```

والـ `MT()` بتطلع نتيجة.

يبقى:

```text
MT()
 ↓
النتيجة
 ↓
X
```

يعني `X` **شايل النتيجة اللي طلعت من `MT()`**.

ولو عايز تستخدم النتيجة:

```ts
X
```

لكن في موضوعنا، إحنا محتاجين نفرّق بين حاجتين:

```text
refreshAccessToken()
        ↓
تشغّل عملية الـ Refresh

refreshPromise
        ↓
يمسك العملية اللي بدأت
```

فأنت مش محتاج دلوقتي تحفظ كلمة `Promise` ولا `<string>`.

**الخطوة التالية:** نرجع للكود ونفهم `refreshPromise` كسطرين فقط، من غير أي TypeScript Types.
</details>
Promise<string
اقصد سيبك من دي
ChatGPT
تمام، **نرمي `Promise<string>` من دماغنا خالص دلوقتي**. خلينا نفهم الفكرة كأنها متغير عادي، في **3 دقائق**.

افترض عندك:

```ts
const X = MT();
```

والـ `MT()` بتطلع نتيجة.

يبقى:

```text
MT()
 ↓
النتيجة
 ↓
X
```

يعني `X` **شايل النتيجة اللي طلعت من `MT()`**.

ولو عايز تستخدم النتيجة:

```ts
X
```

لكن في موضوعنا، إحنا محتاجين نفرّق بين حاجتين:

```text
refreshAccessToken()
        ↓
تشغّل عملية الـ Refresh

refreshPromise
        ↓
يمسك العملية اللي بدأت
```

فأنت مش محتاج دلوقتي تحفظ كلمة `Promise` ولا `<string>`.

**الخطوة التالية:** نرجع للكود ونفهم `refreshPromise` كسطرين فقط، من غير أي TypeScript Types.
<details>
<summary>طيب دلوقتي خلينا نقول const أحمد. عندنا متغير اسمه أحمد. ده يا باشا IST. ساعتها هعمل الدالة اللي اسمها makeT، هو يعمله شيء. اللي هيرجع منها هيروح لأحمد، هطلعله كلمة بس اسمها... كلمة عبارة عن string اسمها T-E-A-S-T. يبقى أحمد كده هعمله شيء، الكلمة كـstring خلاص. تمام. دلوقتي جيه const تاني اسمه محمود عايز أعمله شيء. ساعتها أقوله const محمود makeOne أو أحمد.</summary>
user
طيب دلوقتي خلينا نقول const أحمد. عندنا متغير اسمه أحمد. ده يا باشا IST. ساعتها هعمل الدالة اللي اسمها makeT، هو يعمله شيء. اللي هيرجع منها هيروح لأحمد، هطلعله كلمة بس اسمها... كلمة عبارة عن string اسمها T-E-A-S-T. يبقى أحمد كده هعمله شيء، الكلمة كـstring خلاص. تمام. دلوقتي جيه const تاني اسمه محمود عايز أعمله شيء. ساعتها أقوله const محمود makeOne أو أحمد.
ChatGPT
أيوه، **أنت كده بتقرب جدًا من الفكرة الصح**، وهنسيب `Promise` خالص زي ما طلبت.

خلينا نمشي بنفس المثال بتاعك:

```ts
const makeT = () => {
  return "TEA";
};
```

دلوقتي:

```ts
const Ahmed = makeT();
```

إذن:

```text
makeT()
   ↓
"TEA"
   ↓
Ahmed
```

فـ:

```ts
Ahmed === "TEA"
```

تمام.

---

دلوقتي محمود عايز نفس الحاجة.

لو كتبت:

```ts
const Mahmoud = Ahmed;
```

فده معناه:

```text
Ahmed
 ↓
"TEA"
 ↓
Mahmoud
```

يعني:

```ts
Mahmoud === "TEA"
```

**مش شغلت `makeT` مرة تانية.**

أنت أخذت **القيمة الموجودة بالفعل في `Ahmed`** وحطيتها في `Mahmoud`.

---

أما لو كتبت:

```ts
const Mahmoud = makeT();
```

فأنت بتقول:

> شغّل `makeT` مرة جديدة.

يعني:

```text
makeT() → "TEA" → Ahmed

makeT() → "TEA" → Mahmoud
```

إذن الفرق:

```ts
const Ahmed = makeT();
```

⬅️ شغّل الدالة.

```ts
const Mahmoud = Ahmed;
```

⬅️ خد القيمة الموجودة في `Ahmed`.

```ts
const Mahmoud = makeT();
```

⬅️ شغّل الدالة مرة أخرى.

وده **بالضبط** اللي هنحتاجه بعدين في `refreshPromise`: بدل ما كل Request يشغّل عملية Refresh جديدة، الـ Requests الجديدة تمسك **نفس العملية الموجودة بالفعل**.

**الخطوة التالية:** هنأخذ مثال `Ahmed` و`Mahmoud` نفسه ونحوّله حرفيًا إلى `refreshPromise` من غير أي Types.
</details>
طيب دلوقتي خلينا نقول const أحمد. عندنا متغير اسمه أحمد. ده يا باشا IST. ساعتها هعمل الدالة اللي اسمها makeT، هو يعمله شيء. اللي هيرجع منها هيروح لأحمد، هطلعله كلمة بس اسمها... كلمة عبارة عن string اسمها T-E-A-S-T. يبقى أحمد كده هعمله شيء، الكلمة كـstring خلاص. تمام. دلوقتي جيه const تاني اسمه محمود عايز أعمله شيء. ساعتها أقوله const محمود makeOne أو أحمد.
ChatGPT
أيوه، **أنت كده بتقرب جدًا من الفكرة الصح**، وهنسيب `Promise` خالص زي ما طلبت.

خلينا نمشي بنفس المثال بتاعك:

```ts
const makeT = () => {
  return "TEA";
};
```

دلوقتي:

```ts
const Ahmed = makeT();
```

إذن:

```text
makeT()
   ↓
"TEA"
   ↓
Ahmed
```

فـ:

```ts
Ahmed === "TEA"
```

تمام.

---

دلوقتي محمود عايز نفس الحاجة.

لو كتبت:

```ts
const Mahmoud = Ahmed;
```

فده معناه:

```text
Ahmed
 ↓
"TEA"
 ↓
Mahmoud
```

يعني:

```ts
Mahmoud === "TEA"
```

**مش شغلت `makeT` مرة تانية.**

أنت أخذت **القيمة الموجودة بالفعل في `Ahmed`** وحطيتها في `Mahmoud`.

---

أما لو كتبت:

```ts
const Mahmoud = makeT();
```

فأنت بتقول:

> شغّل `makeT` مرة جديدة.

يعني:

```text
makeT() → "TEA" → Ahmed

makeT() → "TEA" → Mahmoud
```

إذن الفرق:

```ts
const Ahmed = makeT();
```

⬅️ شغّل الدالة.

```ts
const Mahmoud = Ahmed;
```

⬅️ خد القيمة الموجودة في `Ahmed`.

```ts
const Mahmoud = makeT();
```

⬅️ شغّل الدالة مرة أخرى.

وده **بالضبط** اللي هنحتاجه بعدين في `refreshPromise`: بدل ما كل Request يشغّل عملية Refresh جديدة، الـ Requests الجديدة تمسك **نفس العملية الموجودة بالفعل**.

**الخطوة التالية:** هنأخذ مثال `Ahmed` و`Mahmoud` نفسه ونحوّله حرفيًا إلى `refreshPromise` من غير أي Types.
<details>
<summary>لو قالت</summary>
user
لو قالت 
Const nahmoud = await ahmed
ChatGPT
أيوه، **هنا بالذات هنقرب جدًا من فهم `await`**، وفيه نقطة مهمة: `await Ahmed` معناها **استنى القيمة اللي `Ahmed` ماسكها لو كانت عملية لسه شغالة**.

لكن في المثال الحالي:

```ts
const Ahmed = makeT();
```

ولو `makeT()` بترجع `"TEA"` مباشرة، فـ:

```ts
const Mahmoud = await Ahmed;
```

مش هيعمل حاجة مفيدة، لأن `Ahmed` بالفعل:

```text
Ahmed = "TEA"
```

فـ محمود هياخد:

```text
Mahmoud = "TEA"
```

### المهم

`await` مش معناها:

> اعمل Ahmed مرة تانية.

ولا معناها:

> خد Output من Ahmed.

هي معناها:

> **لو Ahmed ماسك عملية لسه مخلصتش، استناها لحد ما تخلص وخد نتيجتها.**

وده سبب إن مثالنا السابق محتاج `Promise`.

لكن بما إنك عايز نفهمها بالتدريج، خلينا نثبت الجملة دي فقط:

```text
Ahmed = عملية مستنية
        ↓
await Ahmed
        ↓
استنى نتيجتها
```

**الخطوة التالية:** هديك مثال صغير جدًا فيه `Ahmed` فعلًا عملية بتاخد وقت، ونشوف `await Ahmed` بيعمل إيه بالضبط.
</details>
لو قالت 
Const nahmoud = await ahmed
ChatGPT
أيوه، **هنا بالذات هنقرب جدًا من فهم `await`**، وفيه نقطة مهمة: `await Ahmed` معناها **استنى القيمة اللي `Ahmed` ماسكها لو كانت عملية لسه شغالة**.

لكن في المثال الحالي:

```ts
const Ahmed = makeT();
```

ولو `makeT()` بترجع `"TEA"` مباشرة، فـ:

```ts
const Mahmoud = await Ahmed;
```

مش هيعمل حاجة مفيدة، لأن `Ahmed` بالفعل:

```text
Ahmed = "TEA"
```

فـ محمود هياخد:

```text
Mahmoud = "TEA"
```

### المهم

`await` مش معناها:

> اعمل Ahmed مرة تانية.

ولا معناها:

> خد Output من Ahmed.

هي معناها:

> **لو Ahmed ماسك عملية لسه مخلصتش، استناها لحد ما تخلص وخد نتيجتها.**

وده سبب إن مثالنا السابق محتاج `Promise`.

لكن بما إنك عايز نفهمها بالتدريج، خلينا نثبت الجملة دي فقط:

```text
Ahmed = عملية مستنية
        ↓
await Ahmed
        ↓
استنى نتيجتها
```

**الخطوة التالية:** هديك مثال صغير جدًا فيه `Ahmed` فعلًا عملية بتاخد وقت، ونشوف `await Ahmed` بيعمل إيه بالضبط.
<details>
<summary>ياريت</summary>
user
ياريت
ChatGPT
خلينا نعمل **مثال واحد فقط في 5 دقائق**، وهتفهم `await` من غير ما ندخل في Axios.

```ts id="n2x7qa"
const Ahmed = new Promise((resolve) => {
  setTimeout(() => {
    resolve("TEA");
  }, 3000);
});
```

هنا حصل إيه؟

```text id="x4m8ks"
Ahmed
 ↓
عملية شغالة
 ↓
استنى 3 ثواني
 ↓
"TEA"
```

يعني `Ahmed` **مش شايل `"TEA"` دلوقتي**.

هو شايل **عملية لسه شغالة**، والعملية دي لما تخلص هتطلع `"TEA"`.

---

دلوقتي محمود:

```ts id="a7p3dz"
const Mahmoud = await Ahmed;
```

معناها:

> يا محمود، استنى العملية الموجودة عند أحمد لحد ما تخلص، وبعدها خد النتيجة.

بعد الـ 3 ثواني:

```text id="q6w1zs"
Ahmed
 ↓
العملية خلصت
 ↓
"TEA"
 ↓
Mahmoud
```

إذن:

```ts id="j5r8kc"
Mahmoud === "TEA"
```

### وهنا مربط الفرس 🎯

لو محمود قال:

```ts id="r4t9vn"
const Mahmoud = Ahmed;
```

فهو **ما استناش**.

هو فقط أخد نفس العملية الموجودة عند أحمد.

لكن لو قال:

```ts id="s8k2lp"
const Mahmoud = await Ahmed;
```

فهو قال:

> **أنا مستني النتيجة.**

احفظ الفرق ده:

```text id="z1c7qm"
Mahmoud = Ahmed
↓
خد نفس العملية

Mahmoud = await Ahmed
↓
استنى العملية وخد نتيجتها
```

وده بالضبط اللي كان ناقصنا في `refreshPromise`.

**الخطوة التالية:** هنرجع للسطر `const newToken = await refreshPromise` ونفكّه كلمة كلمة باستخدام مثال أحمد ومحمود.
</details>
ياريت
ChatGPT
خلينا نعمل **مثال واحد فقط في 5 دقائق**، وهتفهم `await` من غير ما ندخل في Axios.

```ts id="n2x7qa"
const Ahmed = new Promise((resolve) => {
  setTimeout(() => {
    resolve("TEA");
  }, 3000);
});
```

هنا حصل إيه؟

```text id="x4m8ks"
Ahmed
 ↓
عملية شغالة
 ↓
استنى 3 ثواني
 ↓
"TEA"
```

يعني `Ahmed` **مش شايل `"TEA"` دلوقتي**.

هو شايل **عملية لسه شغالة**، والعملية دي لما تخلص هتطلع `"TEA"`.

---

دلوقتي محمود:

```ts id="a7p3dz"
const Mahmoud = await Ahmed;
```

معناها:

> يا محمود، استنى العملية الموجودة عند أحمد لحد ما تخلص، وبعدها خد النتيجة.

بعد الـ 3 ثواني:

```text id="q6w1zs"
Ahmed
 ↓
العملية خلصت
 ↓
"TEA"
 ↓
Mahmoud
```

إذن:

```ts id="j5r8kc"
Mahmoud === "TEA"
```

### وهنا مربط الفرس 🎯

لو محمود قال:

```ts id="r4t9vn"
const Mahmoud = Ahmed;
```

فهو **ما استناش**.

هو فقط أخد نفس العملية الموجودة عند أحمد.

لكن لو قال:

```ts id="s8k2lp"
const Mahmoud = await Ahmed;
```

فهو قال:

> **أنا مستني النتيجة.**

احفظ الفرق ده:

```text id="z1c7qm"
Mahmoud = Ahmed
↓
خد نفس العملية

Mahmoud = await Ahmed
↓
استنى العملية وخد نتيجتها
```

وده بالضبط اللي كان ناقصنا في `refreshPromise`.

**الخطوة التالية:** هنرجع للسطر `const newToken = await refreshPromise` ونفكّه كلمة كلمة باستخدام مثال أحمد ومحمود.
<details>
<summary>تمام</summary>
user
تمام
ChatGPT
دلوقتي نربط **أحمد ومحمود** بالـ `refreshPromise` في **5 دقائق**، وهتشوف إن الموضوع نفسه تمامًا.

### المثال بتاعنا

عندنا:

```ts
const Ahmed = refreshAccessToken();
```

معناها:

```text
refreshAccessToken()
       ↓
بدأ عملية Refresh
       ↓
Ahmed ماسك العملية دي
```

وبعدين محمود:

```ts
const Mahmoud = await Ahmed;
```

معناها:

```text
Ahmed
 ↓
استنى العملية تخلص
 ↓
Mahmoud ياخد النتيجة
```

---

### نبدل الأسماء

بدل `Ahmed`:

```ts
const refreshPromise = refreshAccessToken();
```

وبدل `Mahmoud`:

```ts
const newToken = await refreshPromise;
```

إذن:

```text
refreshAccessToken()
        ↓
refreshPromise
        ↓
await
        ↓
newToken
```

يعني **`refreshPromise` مش الـ Token**.

هو زي أحمد في المثال بتاعنا، ماسك **العملية اللي شغالة**.

والـ `newToken` هو النتيجة اللي محمود استلمها بعد ما العملية خلصت.

---

### ونطبقها على 3 Requests

```text
Request A → 401
     ↓
refreshPromise = refreshAccessToken()
     ↓
بدأ Refresh


Request B → 401
     ↓
await refreshPromise
     ↓
استنى


Request C → 401
     ↓
await refreshPromise
     ↓
استنى
```

لما الـ Refresh يخلص:

```text
refreshPromise
      ↓
NEW_ACCESS_TOKEN
      ↓
A + B + C
```

وهنا بقى **الـ Queue بدأت تبقى مفهومة**: مش لازم كل Request يعمل Refresh، كلهم يستنوا **نفس العملية**.

**الخطوة التالية:** نكتب الكود الحقيقي للـ `refreshAccessToken` أولًا فقط، من غير Interceptor ولا Queue، وبعدها نركبهم مع بعض.
</details>
تمام
ChatGPT
دلوقتي نربط **أحمد ومحمود** بالـ `refreshPromise` في **5 دقائق**، وهتشوف إن الموضوع نفسه تمامًا.

### المثال بتاعنا

عندنا:

```ts
const Ahmed = refreshAccessToken();
```

معناها:

```text
refreshAccessToken()
       ↓
بدأ عملية Refresh
       ↓
Ahmed ماسك العملية دي
```

وبعدين محمود:

```ts
const Mahmoud = await Ahmed;
```

معناها:

```text
Ahmed
 ↓
استنى العملية تخلص
 ↓
Mahmoud ياخد النتيجة
```

---

### نبدل الأسماء

بدل `Ahmed`:

```ts
const refreshPromise = refreshAccessToken();
```

وبدل `Mahmoud`:

```ts
const newToken = await refreshPromise;
```

إذن:

```text
refreshAccessToken()
        ↓
refreshPromise
        ↓
await
        ↓
newToken
```

يعني **`refreshPromise` مش الـ Token**.

هو زي أحمد في المثال بتاعنا، ماسك **العملية اللي شغالة**.

والـ `newToken` هو النتيجة اللي محمود استلمها بعد ما العملية خلصت.

---

### ونطبقها على 3 Requests

```text
Request A → 401
     ↓
refreshPromise = refreshAccessToken()
     ↓
بدأ Refresh


Request B → 401
     ↓
await refreshPromise
     ↓
استنى


Request C → 401
     ↓
await refreshPromise
     ↓
استنى
```

لما الـ Refresh يخلص:

```text
refreshPromise
      ↓
NEW_ACCESS_TOKEN
      ↓
A + B + C
```

وهنا بقى **الـ Queue بدأت تبقى مفهومة**: مش لازم كل Request يعمل Refresh، كلهم يستنوا **نفس العملية**.

**الخطوة التالية:** نكتب الكود الحقيقي للـ `refreshAccessToken` أولًا فقط، من غير Interceptor ولا Queue، وبعدها نركبهم مع بعض.
<details>
<summary>تمام</summary>
user
تمام
ChatGPT
اكتب معايا **`refreshAccessToken` فقط في 5 دقائق**، من غير `Interceptor` ولا `Queue`، عشان نفصل كل فكرة لوحدها.

### 1. عندنا Function

```ts
const refreshAccessToken = async () => {
  const refreshToken = localStorage.getItem("refreshToken");

  const response = await api.post("/auth/refresh", {
    refreshToken,
  });

  const newAccessToken = response.data.accessToken;

  localStorage.setItem("accessToken", newAccessToken);

  return newAccessToken;
};
```

خلينا نفهمها كسلسلة:

```text id="z3k7wp"
refreshAccessToken()
        ↓
جيب Refresh Token
        ↓
ابعت Refresh Token للـ Backend
        ↓
Backend يرجع New Access Token
        ↓
خزّن New Access Token
        ↓
return New Access Token
```

### 2. دلوقتي أهم سطر

```ts
const Ahmed = refreshAccessToken();
```

هنا أنت **بدأت عملية الـ Refresh**.

أحمد ماسك العملية اللي شغالة.

وبعدين:

```ts
const Mahmoud = await Ahmed;
```

محمود بيقول:

> استنى عملية أحمد تخلص واديني الـ New Access Token.

فيبقى:

```text id="j1c8sa"
Ahmed
 ↓
Refresh شغال
 ↓
Backend
 ↓
New Access Token
 ↓
Mahmoud
```

### 3. دلوقتي نرجع للاسم الحقيقي

```ts
const refreshPromise = refreshAccessToken();

const newToken = await refreshPromise;
```

دي **نفس فكرة أحمد ومحمود بالضبط**.

```text id="u7m2qk"
refreshAccessToken()
        ↓
refreshPromise
        ↓
await
        ↓
newToken
```

بس خد بالك من حاجة مهمة:

**إحنا لسه ما عملناش Queue.**

إحنا فقط عملنا **عملية Refresh واحدة** وخلينا متغير يمسك العملية دي.

**الخطوة التالية:** هنكتب `Response Interceptor` كامل، ونضيف عليه `refreshPromise` تدريجيًا، سطر سطر.
</details>
تمام
ChatGPT
اكتب معايا **`refreshAccessToken` فقط في 5 دقائق**، من غير `Interceptor` ولا `Queue`، عشان نفصل كل فكرة لوحدها.

### 1. عندنا Function

```ts
const refreshAccessToken = async () => {
  const refreshToken = localStorage.getItem("refreshToken");

  const response = await api.post("/auth/refresh", {
    refreshToken,
  });

  const newAccessToken = response.data.accessToken;

  localStorage.setItem("accessToken", newAccessToken);

  return newAccessToken;
};
```

خلينا نفهمها كسلسلة:

```text id="z3k7wp"
refreshAccessToken()
        ↓
جيب Refresh Token
        ↓
ابعت Refresh Token للـ Backend
        ↓
Backend يرجع New Access Token
        ↓
خزّن New Access Token
        ↓
return New Access Token
```

### 2. دلوقتي أهم سطر

```ts
const Ahmed = refreshAccessToken();
```

هنا أنت **بدأت عملية الـ Refresh**.

أحمد ماسك العملية اللي شغالة.

وبعدين:

```ts
const Mahmoud = await Ahmed;
```

محمود بيقول:

> استنى عملية أحمد تخلص واديني الـ New Access Token.

فيبقى:

```text id="j1c8sa"
Ahmed
 ↓
Refresh شغال
 ↓
Backend
 ↓
New Access Token
 ↓
Mahmoud
```

### 3. دلوقتي نرجع للاسم الحقيقي

```ts
const refreshPromise = refreshAccessToken();

const newToken = await refreshPromise;
```

دي **نفس فكرة أحمد ومحمود بالضبط**.

```text id="u7m2qk"
refreshAccessToken()
        ↓
refreshPromise
        ↓
await
        ↓
newToken
```

بس خد بالك من حاجة مهمة:

**إحنا لسه ما عملناش Queue.**

إحنا فقط عملنا **عملية Refresh واحدة** وخلينا متغير يمسك العملية دي.

**الخطوة التالية:** هنكتب `Response Interceptor` كامل، ونضيف عليه `refreshPromise` تدريجيًا، سطر سطر.
<details>
<summary>كمل</summary>
user
كمل
</details>
كمل