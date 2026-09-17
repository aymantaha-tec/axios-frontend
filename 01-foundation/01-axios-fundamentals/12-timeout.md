# 1. يعني إيه `timeout`؟

تخيل إنك بتبعت Request للسيرفر:

```ts
const response = await axios.get("/users");
```

أنت بتبعت الطلب، وبتستنى السيرفر يرد.

لكن ماذا لو السيرفر **اتأخر جدًا**؟

مثلاً:

```text
Client
  |
  | GET /users
  ↓
Server
  |
  | ... بيجهز Response
  | ... لسه
  | ... لسه
  | ... لسه
```

ممكن ماينفعش تفضل مستني إلى ما لا نهاية.

هنا يأتي دور:

```ts
timeout
```

معناه ببساطة:

> **أقصى وقت Axios هينتظره للـ request قبل ما يعتبره فشل بسبب انتهاء الوقت.**

---

# 2. مثال

```ts
const response = await axios.get("/users", {
  timeout: 5000,
});
```

`5000` هنا بالـ milliseconds.

يعني:

```text
5000 ms = 5 seconds
```

فالمعنى:

> ابعت Request إلى `/users`، ولو الوقت المحدد انتهى بدون اكتمال الطلب، Axios يلغي الانتظار ويعتبر الطلب فشل بسبب timeout.

---

# 3. مكان `timeout`

خد بالك إنه موجود داخل **Config Object**:

```ts
axios.get("/users", {
  timeout: 5000,
});
```

عندنا:

```text
"/users"
   ↓
URL

{
  timeout: 5000
}
   ↓
Config
```

زي ما عملنا قبل كده:

```ts
axios.get("/products", {
  params: {
    page: 2,
  },

  headers: {
    Authorization: "Bearer token",
  },

  timeout: 5000,
});
```

كل دول إعدادات للـ Request:

```text
params
headers
timeout
   ↓
Config
```

---

# 4. مهم جدًا: `timeout` مش معناه "السيرفر لازم يرد خلال 5 ثواني"

خلينا ناخدها بدقة.

لما تكتب:

```ts
timeout: 5000
```

أنت بتقول لـ Axios:

> لا تنتظر أكثر من المدة المحددة للطلب حسب آلية الـ timeout في Axios.

ولو حصل timeout، الـ Promise هترفض `reject`.

مثلاً:

```ts
try {
  const response = await axios.get("/users", {
    timeout: 5000,
  });

  console.log(response.data);
} catch (error) {
  console.log(error);
}
```

لو حصل timeout، التنفيذ هينتقل إلى:

```ts
catch
```

لكن **موضوع التعامل مع الخطأ بالتفصيل** هنأخده في Section 3، مش دلوقتي.

---

# 5. أشهر غلطة

ما تعملش:

```ts
axios.get("/users", {
  timeout: "5000",
});
```

لأن:

```text
"5000" → string ❌
5000   → number ✅
```

والـ milliseconds هي وحدة القياس.

مثلاً:

```ts
timeout: 1000
```

يعني تقريبًا:

```text
1 second
```

```ts
timeout: 3000
```

يعني:

```text
3 seconds
```

```ts
timeout: 10000
```

يعني:

```text
10 seconds
```

---

## 🧠 ثبتها كده

```text
params
   ↓
Query Parameters
   ↓
URL

data
   ↓
Request Body

headers
   ↓
Request Metadata

timeout
   ↓
Maximum waiting time
```

### سؤال Interview

لو عندك:

```ts
const response = await axios.get("/users", {
  timeout: 3000,
});
```

**ماذا يعني `timeout: 3000`؟**

اكتب إجابتك بجملة واحدة.
<details>
<summary>اقصي وقت لانتظار الطلب هو  3 ثواني غير يعتبره فشل بسبب الوقت</summary>
user
اقصي وقت لانتظار الطلب هو  3 ثواني غير يعتبره فشل بسبب الوقت 
ChatGPT
إجابتك **صحيحة 100%**. فقط الصياغة التقنية الأدق:

> `timeout: 3000` يعني أن Axios ينتظر الطلب لمدة أقصاها **3 ثوانٍ**، وإذا انتهت المدة دون اكتمال الطلب، يعتبره فشلًا بسبب انتهاء المهلة.

### 📍 Progress

```text
Request Configuration

├── Config Object     ✅
├── params            ✅
├── headers           ✅
├── data              ✅
├── timeout           ✅ ← خلصنا
└── signal            🟢 ← التالي
```

**الخطوة التالية:** نبدأ `signal` ونفهم الفرق الأساسي بين `timeout` و `signal`.
</details>
اقصي وقت لانتظار الطلب هو  3 ثواني غير يعتبره فشل بسبب الوقت 
ChatGPT
إجابتك **صحيحة 100%**. فقط الصياغة التقنية الأدق:

> `timeout: 3000` يعني أن Axios ينتظر الطلب لمدة أقصاها **3 ثوانٍ**، وإذا انتهت المدة دون اكتمال الطلب، يعتبره فشلًا بسبب انتهاء المهلة.

### 📍 Progress

```text
Request Configuration

├── Config Object     ✅
├── params            ✅
├── headers           ✅
├── data              ✅
├── timeout           ✅ ← خلصنا
└── signal            🟢 ← التالي
```

**الخطوة التالية:** نبدأ `signal` ونفهم الفرق الأساسي بين `timeout` و `signal`.
<details>
<summary>`signal`</summary>
user
`signal`   
ChatGPT
افهم `signal` الآن كوسيلة للتحكم في الـ Request وإيقافه من خارج Axios، والهدف إنك تفرق بينه وبين `timeout`. ⏱️ 10 دقائق.

## 📍 Roadmap

```text
SECTION 0: Architecture Foundation       ⬜

SECTION 1: Axios Fundamentals             🟢

GET                                      ✅
POST                                     ✅
PUT                                      ✅
PATCH                                    ✅
DELETE                                   ✅

Request Configuration
├── Config Object                        ✅
├── params                               ✅
├── headers                              ✅
├── data                                 ✅
├── timeout                              ✅
└── signal                                🟢 ← إحنا هنا

SECTION 2: TypeScript + Axios             ⬜
SECTION 3: Error Handling                 ⬜
SECTION 4: Interceptors                   ⬜
SECTION 5: API Architecture               ⬜
SECTION 6: TanStack Query                 ⬜
...
```
</details>
`signal`   
ChatGPT
افهم `signal` الآن كوسيلة للتحكم في الـ Request وإيقافه من خارج Axios، والهدف إنك تفرق بينه وبين `timeout`. ⏱️ 10 دقائق.

## 📍 Roadmap

```text
SECTION 0: Architecture Foundation       ⬜

SECTION 1: Axios Fundamentals             🟢

GET                                      ✅
POST                                     ✅
PUT                                      ✅
PATCH                                    ✅
DELETE                                   ✅

Request Configuration
├── Config Object                        ✅
├── params                               ✅
├── headers                              ✅
├── data                                 ✅
├── timeout                              ✅
└── signal                                🟢 ← إحنا هنا

SECTION 2: TypeScript + Axios             ⬜
SECTION 3: Error Handling                 ⬜
SECTION 4: Interceptors                   ⬜
SECTION 5: API Architecture               ⬜
SECTION 6: TanStack Query                 ⬜
...
```