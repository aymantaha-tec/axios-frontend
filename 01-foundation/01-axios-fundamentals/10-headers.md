# `headers` يعني إيه؟

إحنا اتعلمنا في HTTP إن الـ Request ممكن يحتوي على **Headers**.

مثلاً:

```http
GET /users
Authorization: Bearer abc123
Accept: application/json
```

الـ Headers هي **معلومات إضافية عن الـ Request**.

في Axios بنكتبها داخل `config`:

```ts
const response = await axios.get("/users", {
  headers: {
    Authorization: "Bearer abc123",
    Accept: "application/json",
  },
});
```

لاحظ:

```text
axios.get()
    ↓
Config
    ↓
headers
    ↓
Header Name : Header Value
```

---

# 1. Authorization

أشهر استخدام هنقابله في المشاريع:

```ts
headers: {
  Authorization: "Bearer abc123",
}
```

يعني:

```text
Authorization
↓
اسم الـ Header

Bearer abc123
↓
قيمته
```

والـ Server يستخدمه لمعرفة إن الـ Request مصرح له.

**هنشرح Authentication و Access Token بالتفصيل في Section مخصص لاحقًا.**

دلوقتي فقط افهم مكان الـ Header وطريقة كتابته.

---

# 2. Accept

مثلاً:

```ts
headers: {
  Accept: "application/json",
}
```

معناه:

> أنا أقبل أن يكون الـ Response بصيغة JSON.

---

# 3. Content-Type

لو بنرسل Body، ممكن نحدد نوعه:

```ts
headers: {
  "Content-Type": "application/json",
}
```

معناه:

> البيانات الموجودة في الـ Request Body هي JSON.

مثلاً:

```ts
await axios.post(
  "/users",
  {
    name: "Ahmed",
  },
  {
    headers: {
      "Content-Type": "application/json",
    },
  }
);
```

هنا عندنا:

```text
POST
 ↓
URL
 ↓
Body
 ↓
Config
    ↓
Headers
```

---

# 4. GET مع Headers

```ts
const response = await axios.get<User[]>("/users", {
  headers: {
    Authorization: "Bearer abc123",
    Accept: "application/json",
  },
});
```

هنا الـ config:

```ts
{
  headers: {
    Authorization: "Bearer abc123",
    Accept: "application/json",
  },
}
```

---

# 5. POST مع Headers

هنا مهم تفتكر ترتيب arguments:

```ts
axios.post(
  url,
  data,
  config
);
```

مثال:

```ts
const response = await axios.post<User>(
  "/users",
  userData,
  {
    headers: {
      Authorization: "Bearer abc123",
      "Content-Type": "application/json",
    },
  }
);
```

لاحظ الفرق:

### GET

```ts
axios.get(url, config);
```

### POST

```ts
axios.post(url, data, config);
```

لأن POST عندها Body.

---

# 🧠 أهم قاعدة في `headers`

الـ Header عبارة عن:

```text
Name : Value
```

في JavaScript:

```ts
headers: {
  Authorization: "Bearer abc123",
}
```

يعني:

```text
Authorization
     ↓
Header Name

Bearer abc123
     ↓
Header Value
```

---

# 🧪 تدريب

عندك:

```ts
interface User {
  id: number;
  name: string;
}
```

عايز تعمل:

```http
GET /users
```

والـ API محتاجة:

```text
Authorization: Bearer xyz789
Accept: application/json
```

اكتب:

```ts
const response = ...
```

باستخدام:

```text
User[]
headers
```

**الخطوة التالية: اكتب الكود، وأنا أصححه فقط ثم نسأل Interview Question عن `headers`.**
<details>
<summary>const response =  await axios.get&lt;User[]&gt;(</summary>
user
const response =  await axios.get<User[]>(
"/users ",
{
headers: {
Authorization: "Bearer xyz789  ",
Accept: application/json
}
})
ChatGPT
**الهدف: تصحيح تدريب `headers` فقط، وإجابتك صحيحة في الـ structure، وعندك خطأين بسيطين في الـ syntax والقيمة.** الوقت: **3 دقائق**.

### ✅ الشكل الصحيح

```ts
const response = await axios.get<User[]>(
  "/users",
  {
    headers: {
      Authorization: "Bearer xyz789",
      Accept: "application/json",
    },
  }
);
```

### التصحيح

1. عندك مسافة زيادة داخل الـ URL:

```ts
"/users "
```

الصحيح:

```ts
"/users"
```

2. `application/json` قيمة نصية، فلازم تكون String:

```ts
Accept: "application/json"
```

وليس:

```ts
Accept: application/json
```

3. عندك مسافات زيادة داخل الـ Authorization:

```ts
"Bearer xyz789  "
```

الأفضل:

```ts
"Bearer xyz789"
```

### 🟢 اللي أنت فهمته صح

```text
axios.get<User[]>
       ↓
Response Type

"/users"
       ↓
URL

headers
       ↓
Config

Authorization / Accept
       ↓
Headers
```

**التقييم: 9/10**. الـ concept نفسه عندك صحيح، والأخطاء كلها Syntax بسيطة.

### 🎤 Interview Question

**ما الفرق بين `headers` و `params` في Axios؟**

جاوب من فهمك، ومش هضيف أي معلومة جديدة قبل ما نصحح إجابتك.
</details>
const response =  await axios.get<User[]>(
"/users ",
{
headers: {
Authorization: "Bearer xyz789  ",
Accept: application/json
}
})
ChatGPT
**الهدف: تصحيح تدريب `headers` فقط، وإجابتك صحيحة في الـ structure، وعندك خطأين بسيطين في الـ syntax والقيمة.** الوقت: **3 دقائق**.

### ✅ الشكل الصحيح

```ts
const response = await axios.get<User[]>(
  "/users",
  {
    headers: {
      Authorization: "Bearer xyz789",
      Accept: "application/json",
    },
  }
);
```

### التصحيح

1. عندك مسافة زيادة داخل الـ URL:

```ts
"/users "
```

الصحيح:

```ts
"/users"
```

2. `application/json` قيمة نصية، فلازم تكون String:

```ts
Accept: "application/json"
```

وليس:

```ts
Accept: application/json
```

3. عندك مسافات زيادة داخل الـ Authorization:

```ts
"Bearer xyz789  "
```

الأفضل:

```ts
"Bearer xyz789"
```

### 🟢 اللي أنت فهمته صح

```text
axios.get<User[]>
       ↓
Response Type

"/users"
       ↓
URL

headers
       ↓
Config

Authorization / Accept
       ↓
Headers
```

**التقييم: 9/10**. الـ concept نفسه عندك صحيح، والأخطاء كلها Syntax بسيطة.

### 🎤 Interview Question

**ما الفرق بين `headers` و `params` في Axios؟**

جاوب من فهمك، ومش هضيف أي معلومة جديدة قبل ما نصحح إجابتك.
<details>
<summary>`headers`</summary>
user
`headers`   
دي عباره عن metadate او معلومات اضافية برسلها مع الريكوست في url
اما ال params هي بتتحكم في نوع البيانت تجيلي ازاي في كاصفحة الليمت 
ChatGPT
**الهدف: تثبيت الفرق بين `headers` و `params`، وإجابتك قريبة جدًا، لكن عندك نقطة واحدة لازم تتظبط.** الوقت: **3 دقائق**.

### 🎤 تقييم Interview

إجابتك عن `headers`:

> Metadata أو معلومات إضافية أرسلها مع الـ Request.

✅ **صحيحة.**

لكن قلت:

> في URL

❌ هنا التصحيح مهم: **الـ Headers مش جزء من الـ URL**.

مثلاً:

```http id="9y2y3h"
GET /users?page=2

Authorization: Bearer xyz789
Accept: application/json
```

هنا:

```text id="04vvap"
URL
↓
/users?page=2

Headers
↓
Authorization
Accept
```

هما منفصلان.

---

### `params`

إجابتك:

> بتتحكم في نوع البيانات تجيلي إزاي، زي الصفحة والـ limit.

✅ **صحيحة جدًا.**

مثلاً:

```ts id="9u0vqt"
params: {
  page: 2,
  limit: 10,
}
```

Axios تحولها إلى:

```text id="f1jshp"
/users?page=2&limit=10
```

يعني `params` هي **Query Parameters** وتكون جزءًا من الـ URL.

---

# 🧠 احفظ الفرق بالشكل ده

```text id="c5lhx7"
headers
↓
معلومات إضافية عن الـ Request
↓
ليست جزءًا من URL

params
↓
Query Parameters
↓
تتحول إلى جزء من URL
```

مثال واحد يجمع الاثنين:

```ts id="myk0w6"
axios.get("/users", {
  params: {
    page: 2,
    limit: 10,
  },
  headers: {
    Authorization: "Bearer xyz789",
  },
});
```

والـ Request الناتج تقريبًا:

```http id="5m4o5r"
GET /users?page=2&limit=10

Authorization: Bearer xyz789
```

**التقييم: 9/10، والـ concept عندك ثابت.**

### 📊 Progress

```text id="f4c9o2"
Request Configuration

├── Config Object     ✅
├── params            ✅
├── headers           ✅
├── data              ⬅️ التالي
├── timeout           ⬜
└── signal            ⬜
```

**الخطوة التالية: نبدأ `data` في Axios، ونفرق بينها وبين `params` لأن الاتنين ممكن يسببوا لخبطة في البداية.**
</details>
`headers`   
دي عباره عن metadate او معلومات اضافية برسلها مع الريكوست في url
اما ال params هي بتتحكم في نوع البيانت تجيلي ازاي في كاصفحة الليمت 
ChatGPT
**الهدف: تثبيت الفرق بين `headers` و `params`، وإجابتك قريبة جدًا، لكن عندك نقطة واحدة لازم تتظبط.** الوقت: **3 دقائق**.

### 🎤 تقييم Interview

إجابتك عن `headers`:

> Metadata أو معلومات إضافية أرسلها مع الـ Request.

✅ **صحيحة.**

لكن قلت:

> في URL

❌ هنا التصحيح مهم: **الـ Headers مش جزء من الـ URL**.

مثلاً:

```http id="9y2y3h"
GET /users?page=2

Authorization: Bearer xyz789
Accept: application/json
```

هنا:

```text id="04vvap"
URL
↓
/users?page=2

Headers
↓
Authorization
Accept
```

هما منفصلان.

---

### `params`

إجابتك:

> بتتحكم في نوع البيانات تجيلي إزاي، زي الصفحة والـ limit.

✅ **صحيحة جدًا.**

مثلاً:

```ts id="9u0vqt"
params: {
  page: 2,
  limit: 10,
}
```

Axios تحولها إلى:

```text id="f1jshp"
/users?page=2&limit=10
```

يعني `params` هي **Query Parameters** وتكون جزءًا من الـ URL.

---

# 🧠 احفظ الفرق بالشكل ده

```text id="c5lhx7"
headers
↓
معلومات إضافية عن الـ Request
↓
ليست جزءًا من URL

params
↓
Query Parameters
↓
تتحول إلى جزء من URL
```

مثال واحد يجمع الاثنين:

```ts id="myk0w6"
axios.get("/users", {
  params: {
    page: 2,
    limit: 10,
  },
  headers: {
    Authorization: "Bearer xyz789",
  },
});
```

والـ Request الناتج تقريبًا:

```http id="5m4o5r"
GET /users?page=2&limit=10

Authorization: Bearer xyz789
```

**التقييم: 9/10، والـ concept عندك ثابت.**

### 📊 Progress

```text id="f4c9o2"
Request Configuration

├── Config Object     ✅
├── params            ✅
├── headers           ✅
├── data              ⬅️ التالي
├── timeout           ⬜
└── signal            ⬜
```

**الخطوة التالية: نبدأ `data` في Axios، ونفرق بينها وبين `params` لأن الاتنين ممكن يسببوا لخبطة في البداية.**
<details>
<summary>`data`</summary>
user
`data`   
ChatGPT
ابدأ بفهم `data` كـ **البيانات التي بتتبعت داخل Body بتاع الـ Request**، وهنفرقها عن `params` بشكل نهائي. ⏱️ حوالي 10 دقائق.

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
├── data                                 🟢 ← إحنا هنا
├── timeout                              ⬜
└── signal                               ⬜

SECTION 2: TypeScript + Axios             ⬜
SECTION 3: Error Handling                 ⬜
SECTION 4: Interceptors                   ⬜
SECTION 5: API Architecture               ⬜
SECTION 6: TanStack Query                 ⬜
...
```
</details>
`data`   
ChatGPT
ابدأ بفهم `data` كـ **البيانات التي بتتبعت داخل Body بتاع الـ Request**، وهنفرقها عن `params` بشكل نهائي. ⏱️ حوالي 10 دقائق.

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
├── data                                 🟢 ← إحنا هنا
├── timeout                              ⬜
└── signal                               ⬜

SECTION 2: TypeScript + Axios             ⬜
SECTION 3: Error Handling                 ⬜
SECTION 4: Interceptors                   ⬜
SECTION 5: API Architecture               ⬜
SECTION 6: TanStack Query                 ⬜
...
```