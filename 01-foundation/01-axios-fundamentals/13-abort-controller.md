# 1. يعني إيه `signal`؟

في Axios ممكن تحتاج تقول:

> **وقف الـ Request ده دلوقتي.**

مش لازم تستنى `timeout`.

مثلاً المستخدم عمل Search:

```text
المستخدم كتب: React
       ↓
Request
       ↓
المستخدم كتب: React Query
       ↓
Request جديد
```

الـ Request الأول ممكن يكون لسه شغال، ومبقاش محتاجينه.

نقدر نعمل له **Cancel** باستخدام `signal`.

---

# 2. مين المسؤول عن الـ Cancel؟

Axios نفسه مش هو اللي بيقرر إمتى توقف الـ Request.

إنت بتستخدم JavaScript API اسمه:

```ts
AbortController
```

بتعمل:

```ts
const controller = new AbortController();
```

وبعدين تبعت الـ `signal` بتاعه إلى Axios:

```ts
const response = await axios.get("/users", {
  signal: controller.signal,
});
```

دلوقتي Axios بقى مرتبط بالـ `controller`.

---

# 3. إزاي أوقف الـ Request؟

في أي وقت تقدر تعمل:

```ts
controller.abort();
```

يعني:

```ts
const controller = new AbortController();

const response = await axios.get("/users", {
  signal: controller.signal,
});

controller.abort();
```

الفكرة:

```text
AbortController
       │
       │ signal
       ↓
     Axios
       │
       │ Request
       ↓
    Server

controller.abort()
       ↓
إلغاء الـ Request
```

---

# 4. ليه اسمها `signal`؟

لأن:

```ts
controller.signal
```

عبارة عن **إشارة** بتوصل للـ Request:

> لو الـ controller اتعمله abort، أوقف الـ operation المرتبط بالـ signal ده.

يعني الـ `signal` نفسه مش هو اللي بينفذ الإلغاء.

اللي بيطلب الإلغاء هو:

```ts
controller.abort();
```

والـ Axios Request بيكون متوصل بالـ:

```ts
signal: controller.signal
```

---

# 5. الفرق بين `timeout` و `signal`

دي أهم نقطة في الدرس.

### `timeout`

إنت بتقول:

> استنى مدة محددة، ولو الوقت خلص يحصل timeout.

```ts
axios.get("/users", {
  timeout: 5000,
});
```

يعني:

```text
انتظر
  ↓
5 seconds
  ↓
لو لم يكتمل الطلب
  ↓
Timeout
```

### `signal`

إنت بتقول:

> أنا عايز أكون قادر ألغي الـ Request بنفسي.

```ts
const controller = new AbortController();

axios.get("/users", {
  signal: controller.signal,
});

controller.abort();
```

يعني:

```text
Request بدأ
   ↓
أنا قررت ألغيه
   ↓
controller.abort()
   ↓
Request cancelled
```

## 🧠 احفظها بالشكل ده

```text
timeout
↓
الوقت هو اللي يحدد

signal
↓
أنا أقدر أحدد إمتى أوقفه
```

---

# 6. مثال واقعي

تخيل صفحة فيها زر:

```text
Loading users...

[ Cancel ]
```

لما المستخدم يضغط Cancel:

```ts
controller.abort();
```

فيتم إلغاء الـ Request بدل ما نفضل مستنيين.

وده مفيد جدًا في حاجات زي:

- Search requests
- Uploads
- Downloads
- Requests المرتبطة بصفحة المستخدم خرج منها
- العمليات اللي المستخدم قرر يلغيها

مش محتاج تحفظ الأمثلة دي كلها دلوقتي، المهم تفهم **الفكرة الأساسية**.

### 🎯 Interview Question

عندك:

```ts
const controller = new AbortController();

axios.get("/users", {
  signal: controller.signal,
});
```

**لو عايز ألغي الـ Request قبل ما يخلص، هتكتب إيه؟**

**الخطوة التالية:** اكتب السطر الذي يلغي الـ Request.
<details>
<summary>controller.abort();</summary>
user
controller.abort();

سمّها عندي: RTL Arabic + LTR English Inline Formatting.

ولما تطلبها مني بعد كده، ممكن تقول ببساطة:

اكتب الشرح بـ RTL عربي، وخلي الـ English Inline في مكانه الطبيعي.
عاوز تراعي ده ف اللي جاي 
ChatGPT
إجابتك صحيحة: `controller.abort()` هو السطر الذي يطلب إلغاء الـ Request.

وسأراعي من الآن تنسيق **RTL Arabic + LTR English Inline Formatting** في الشروحات القادمة. memcite

**الخطوة التالية:** نعمل تمرين صغير يجمع `params` و`headers` و`data` و`timeout` و`signal` في Request واحد.
</details>
controller.abort();

سمّها عندي: RTL Arabic + LTR English Inline Formatting.

ولما تطلبها مني بعد كده، ممكن تقول ببساطة:

اكتب الشرح بـ RTL عربي، وخلي الـ English Inline في مكانه الطبيعي.
عاوز تراعي ده ف اللي جاي 
ChatGPT
إجابتك صحيحة: `controller.abort()` هو السطر الذي يطلب إلغاء الـ Request.

وسأراعي من الآن تنسيق **RTL Arabic + LTR English Inline Formatting** في الشروحات القادمة. memcite

**الخطوة التالية:** نعمل تمرين صغير يجمع `params` و`headers` و`data` و`timeout` و`signal` في Request واحد.
<details>
<summary>تمام</summary>
user
تمام
ChatGPT
اكتب الـ Request التالي بنفسك، والهدف إنك تجمع كل اللي اتعلمناه. ⏱️ 7 دقائق

### المطلوب

اعمل `POST` على:

```text
/users
```

وأرسل في `data`:

```ts
{
  name: "Mohamed",
  email: "mohamed@example.com",
  age: 30
}
```

واستخدم:

```text
params:
page = 1

headers:
Authorization = "Bearer token123"

timeout:
5000

signal:
controller.signal
```

ابدأ أولًا بـ:

```ts
const controller = new AbortController();
```

ثم اكتب `axios.post(...)` كاملًا.

**الخطوة التالية:** اكتب الكود كاملًا من عندك بدون ما تبص على حل جاهز.
</details>
تمام
ChatGPT
اكتب الـ Request التالي بنفسك، والهدف إنك تجمع كل اللي اتعلمناه. ⏱️ 7 دقائق

### المطلوب

اعمل `POST` على:

```text
/users
```

وأرسل في `data`:

```ts
{
  name: "Mohamed",
  email: "mohamed@example.com",
  age: 30
}
```

واستخدم:

```text
params:
page = 1

headers:
Authorization = "Bearer token123"

timeout:
5000

signal:
controller.signal
```

ابدأ أولًا بـ:

```ts
const controller = new AbortController();
```

ثم اكتب `axios.post(...)` كاملًا.

**الخطوة التالية:** اكتب الكود كاملًا من عندك بدون ما تبص على حل جاهز.
<details>
<summary>const controller = new AbortController();</summary>
user
const controller = new AbortController();  
interface ICreateUser{
 name: string,
  email: string,,
  age: number,
}

interface IUser{
 name: string,
  email: string,,
  age: number,
id: number,
}
const CreateUser = {

name: "Mohamed",
  email: "mohamed@example.com",
  age: 30
}

const response = await axiose.post<IUser>("/users", CreateUser ,{
params:  {
page = 1  
},
headers:  {
Authorization = "Bearer token123"  
},
timeout:  5000,
signal: controller.signal ,
})

ولما احب اوقفه controller.bort()
ChatGPT
صحح 4 حاجات فقط في الكود، والباقي من ناحية الـ Axios concept ممتاز. ⏱️ 5 دقائق

### 1. أخطاء Syntax صغيرة

كتبت:

```ts
email: string,,
```

لازم:

```ts
email: string,
```

وفي الـ object كتبت:

```ts
page = 1
```

داخل object لازم `:`:

```ts
page: 1
```

ونفس الكلام:

```ts
Authorization: "Bearer token123"
```

مش `=`.

---

### 2. اسم Axios

كتبت:

```ts
axiose.post
```

الصحيح:

```ts
axios.post
```

---

### 3. `abort`

كتبت:

```ts
controller.bort()
```

الصحيح:

```ts
controller.abort()
```

---

### 4. الكود كاملًا

```ts
const controller = new AbortController();

interface ICreateUser {
  name: string;
  email: string;
  age: number;
}

interface IUser {
  name: string;
  email: string;
  age: number;
  id: number;
}

const createUser: ICreateUser = {
  name: "Mohamed",
  email: "mohamed@example.com",
  age: 30,
};

const response = await axios.post<IUser>(
  "/users",
  createUser,
  {
    params: {
      page: 1,
    },
    headers: {
      Authorization: "Bearer token123",
    },
    timeout: 5000,
    signal: controller.signal,
  }
);
```

ولو عايز تلغي الـ Request:

```ts
controller.abort();
```

### ⭐ تقييم التمرين

**Conceptual understanding: 10/10**

أنت فاهم مكان كل حاجة:

```text
params   → URL
data     → Body
headers  → Metadata
timeout  → Maximum waiting time
signal   → Control / cancellation
```

الأخطاء اللي وقعت فيها كلها تقريبًا **Syntax / Typing mistakes**، وده طبيعي جدًا أثناء الكتابة.

**الخطوة التالية:** جاوب: هل `page: 1` هنا مكانها الصحيح فعلًا داخل `params`، أم المفروض تكون داخل `data`؟ ولماذا؟
</details>
const controller = new AbortController();  
interface ICreateUser{
 name: string,
  email: string,,
  age: number,
}

interface IUser{
 name: string,
  email: string,,
  age: number,
id: number,
}
const CreateUser = {

name: "Mohamed",
  email: "mohamed@example.com",
  age: 30
}

const response = await axiose.post<IUser>("/users", CreateUser ,{
params:  {
page = 1  
},
headers:  {
Authorization = "Bearer token123"  
},
timeout:  5000,
signal: controller.signal ,
})

ولما احب اوقفه controller.bort()
ChatGPT
صحح 4 حاجات فقط في الكود، والباقي من ناحية الـ Axios concept ممتاز. ⏱️ 5 دقائق

### 1. أخطاء Syntax صغيرة

كتبت:

```ts
email: string,,
```

لازم:

```ts
email: string,
```

وفي الـ object كتبت:

```ts
page = 1
```

داخل object لازم `:`:

```ts
page: 1
```

ونفس الكلام:

```ts
Authorization: "Bearer token123"
```

مش `=`.

---

### 2. اسم Axios

كتبت:

```ts
axiose.post
```

الصحيح:

```ts
axios.post
```

---

### 3. `abort`

كتبت:

```ts
controller.bort()
```

الصحيح:

```ts
controller.abort()
```

---

### 4. الكود كاملًا

```ts
const controller = new AbortController();

interface ICreateUser {
  name: string;
  email: string;
  age: number;
}

interface IUser {
  name: string;
  email: string;
  age: number;
  id: number;
}

const createUser: ICreateUser = {
  name: "Mohamed",
  email: "mohamed@example.com",
  age: 30,
};

const response = await axios.post<IUser>(
  "/users",
  createUser,
  {
    params: {
      page: 1,
    },
    headers: {
      Authorization: "Bearer token123",
    },
    timeout: 5000,
    signal: controller.signal,
  }
);
```

ولو عايز تلغي الـ Request:

```ts
controller.abort();
```

### ⭐ تقييم التمرين

**Conceptual understanding: 10/10**

أنت فاهم مكان كل حاجة:

```text
params   → URL
data     → Body
headers  → Metadata
timeout  → Maximum waiting time
signal   → Control / cancellation
```

الأخطاء اللي وقعت فيها كلها تقريبًا **Syntax / Typing mistakes**، وده طبيعي جدًا أثناء الكتابة.

**الخطوة التالية:** جاوب: هل `page: 1` هنا مكانها الصحيح فعلًا داخل `params`، أم المفروض تكون داخل `data`؟ ولماذا؟
<details>
<summary>ده مكانها الصحيحح عشان ده query parametars</summary>
user
ده مكانها الصحيحح عشان ده query parametars 
ChatGPT
إجابتك **صحيحة 100%**.

`page: 1` مكانها داخل `params` لأنها **Query Parameter**، وبالتالي Axios هيضعها في الـ URL.

```ts
axios.post("/users", createUser, {
  params: {
    page: 1,
  },
});
```

والطلب هيكون بشكل تقريبي:

```text
POST /users?page=1
```

بينما:

```ts
createUser
```

هيكون في الـ **Request Body**.

### 📍 Section 1 مكتمل

```text
Request Configuration
├── Config Object     ✅
├── params            ✅
├── headers           ✅
├── data              ✅
├── timeout           ✅
└── signal            ✅
```

وبكده **Axios Fundamentals** من ناحية الـ HTTP methods والـ Request Configuration خلصت.

**الخطوة التالية:** نبدأ أول جزء من **Section 2: TypeScript + Axios**، وهو `AxiosResponse` وهنفهم بالضبط إيه اللي موجود داخل `response`.
</details>
ده مكانها الصحيحح عشان ده query parametars 
ChatGPT
إجابتك **صحيحة 100%**.

`page: 1` مكانها داخل `params` لأنها **Query Parameter**، وبالتالي Axios هيضعها في الـ URL.

```ts
axios.post("/users", createUser, {
  params: {
    page: 1,
  },
});
```

والطلب هيكون بشكل تقريبي:

```text
POST /users?page=1
```

بينما:

```ts
createUser
```

هيكون في الـ **Request Body**.

### 📍 Section 1 مكتمل

```text
Request Configuration
├── Config Object     ✅
├── params            ✅
├── headers           ✅
├── data              ✅
├── timeout           ✅
└── signal            ✅
```

وبكده **Axios Fundamentals** من ناحية الـ HTTP methods والـ Request Configuration خلصت.

**الخطوة التالية:** نبدأ أول جزء من **Section 2: TypeScript + Axios**، وهو `AxiosResponse` وهنفهم بالضبط إيه اللي موجود داخل `response`.
<details>
<summary>Section 0</summary>
user
Section 0
ده امتي عشان ميتنساش
ChatGPT
لا، **Section 0 مش هيتنسى**، وهو جزء أساسي من الخطة، لكنه **مش مكانه دلوقتي**.

إحنا حاليًا بنكمّل الأساسيات اللي محتاجينها قبل ما ندخل في الـ Architecture.

### 📍 الخطة الكاملة

```text
SECTION 0: Architecture Foundation       ⬜
│
├── HTTP fundamentals                     ⬜
├── Separation of Concerns                ⬜
├── Coupling & Cohesion                   ⬜
├── Shared vs Feature                     ⬜
├── Dependency Direction                  ⬜
├── Public vs Internal APIs               ⬜
└── Feature-Based Architecture            ⬜
                    ↓
SECTION 1: Axios Fundamentals             🟢
├── HTTP Methods                          ✅
├── Config Object                         ✅
├── params                                ✅
├── headers                               ✅
├── data                                  ✅
├── timeout                               ✅
└── signal                                ✅
                    ↓
SECTION 2: TypeScript + Axios             ⬜
SECTION 3: Error Handling                 ⬜
SECTION 4: Interceptors                   ⬜
SECTION 5: API Architecture               ⬜
SECTION 6: TanStack Query                 ⬜
SECTION 7: Authentication Architecture   ⬜
SECTION 8: Forms                          ⬜
SECTION 9: Tailwind + shadcn/ui           ⬜
SECTION 10: Storybook                     ⬜
SECTION 11: Testing                       ⬜
SECTION 12: Production Architecture      ⬜
SECTION 13: Final Project                 ⬜
```

**إمتى Section 0؟**

بعد ما نخلص **Section 1 وSection 2 وSection 3**، هنوقف وندخل **Section 0 كامل كقسم مستقل** قبل ما ندخل في الـ Interceptors والـ API Architecture.

السبب إنك وقتها هتكون شوفت Axios وTypeScript وError Handling عمليًا، وبالتالي لما أشرحلك `Separation of Concerns` و`Coupling` و`Feature-Based Architecture` هتربط الكلام بمشاكل حقيقية شفتها بالفعل، مش مجرد كلام نظري.

ومش هنتخطاه أو نعتبره اختياري.

**الخطوة التالية:** نبدأ `Section 2` بـ `AxiosResponse`، وبعدها نرجع رسميًا لـ `Section 0` في مكانه المحدد.
</details>
Section 0
ده امتي عشان ميتنساش
ChatGPT
لا، **Section 0 مش هيتنسى**، وهو جزء أساسي من الخطة، لكنه **مش مكانه دلوقتي**.

إحنا حاليًا بنكمّل الأساسيات اللي محتاجينها قبل ما ندخل في الـ Architecture.

### 📍 الخطة الكاملة

```text
SECTION 0: Architecture Foundation       ⬜
│
├── HTTP fundamentals                     ⬜
├── Separation of Concerns                ⬜
├── Coupling & Cohesion                   ⬜
├── Shared vs Feature                     ⬜
├── Dependency Direction                  ⬜
├── Public vs Internal APIs               ⬜
└── Feature-Based Architecture            ⬜
                    ↓
SECTION 1: Axios Fundamentals             🟢
├── HTTP Methods                          ✅
├── Config Object                         ✅
├── params                                ✅
├── headers                               ✅
├── data                                  ✅
├── timeout                               ✅
└── signal                                ✅
                    ↓
SECTION 2: TypeScript + Axios             ⬜
SECTION 3: Error Handling                 ⬜
SECTION 4: Interceptors                   ⬜
SECTION 5: API Architecture               ⬜
SECTION 6: TanStack Query                 ⬜
SECTION 7: Authentication Architecture   ⬜
SECTION 8: Forms                          ⬜
SECTION 9: Tailwind + shadcn/ui           ⬜
SECTION 10: Storybook                     ⬜
SECTION 11: Testing                       ⬜
SECTION 12: Production Architecture      ⬜
SECTION 13: Final Project                 ⬜
```

**إمتى Section 0؟**

بعد ما نخلص **Section 1 وSection 2 وSection 3**، هنوقف وندخل **Section 0 كامل كقسم مستقل** قبل ما ندخل في الـ Interceptors والـ API Architecture.

السبب إنك وقتها هتكون شوفت Axios وTypeScript وError Handling عمليًا، وبالتالي لما أشرحلك `Separation of Concerns` و`Coupling` و`Feature-Based Architecture` هتربط الكلام بمشاكل حقيقية شفتها بالفعل، مش مجرد كلام نظري.

ومش هنتخطاه أو نعتبره اختياري.

**الخطوة التالية:** نبدأ `Section 2` بـ `AxiosResponse`، وبعدها نرجع رسميًا لـ `Section 0` في مكانه المحدد.
<details>
<summary>**Section 2: TypeScript + Axios**،</summary>
user
**Section 2: TypeScript + Axios**،  
ChatGPT
ابدأ بـ **Section 2: TypeScript + Axios**، وأول درس فيه هو فهم `AxiosResponse` لأنك بتتعامل مع `response` من أول `GET` و`POST`، لكن لسه ما فككناش مكوناته. ⏱️ 12 دقيقة

## 📍 Roadmap

```text
SECTION 0: Architecture Foundation       ⬜
   ↓
SECTION 1: Axios Fundamentals             ✅
   ↓
SECTION 2: TypeScript + Axios             🟢 ← إحنا هنا

├── AxiosResponse                         🟢
├── Generics مع Axios                     ⬜
├── Response Types                         ⬜
├── API Response Types                     ⬜
├── Discriminated Unions                   ⬜
├── unknown vs any                         ⬜
└── Type-safe API Layer                    ⬜

SECTION 3: Error Handling                 ⬜
SECTION 4: Interceptors                   ⬜
SECTION 5: API Architecture               ⬜
...
```
</details>
**Section 2: TypeScript + Axios**،  
ChatGPT
ابدأ بـ **Section 2: TypeScript + Axios**، وأول درس فيه هو فهم `AxiosResponse` لأنك بتتعامل مع `response` من أول `GET` و`POST`، لكن لسه ما فككناش مكوناته. ⏱️ 12 دقيقة

## 📍 Roadmap

```text
SECTION 0: Architecture Foundation       ⬜
   ↓
SECTION 1: Axios Fundamentals             ✅
   ↓
SECTION 2: TypeScript + Axios             🟢 ← إحنا هنا

├── AxiosResponse                         🟢
├── Generics مع Axios                     ⬜
├── Response Types                         ⬜
├── API Response Types                     ⬜
├── Discriminated Unions                   ⬜
├── unknown vs any                         ⬜
└── Type-safe API Layer                    ⬜

SECTION 3: Error Handling                 ⬜
SECTION 4: Interceptors                   ⬜
SECTION 5: API Architecture               ⬜
...
```