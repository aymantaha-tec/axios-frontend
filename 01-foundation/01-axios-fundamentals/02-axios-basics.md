**الهدف: نبدأ Lesson 2 ونحوّل فهم HTTP إلى Axios عمليًا، بدون الدخول في Interceptors أو Architecture لسه.** الوقت: **30 إلى 45 دقيقة**.

# Lesson 2: Axios Fundamentals

## 1. Axios إيه بالضبط؟

Axios هي **HTTP Client Library**.

يعني بدل ما تتعامل يدويًا مع HTTP APIs، Axios توفر لك API سهلة في JavaScript وTypeScript لإرسال واستقبال HTTP Requests.

الصورة:

```text
React
  ↓
Axios
  ↓
HTTP Request
  ↓
Backend API
  ↓
HTTP Response
  ↓
Axios
  ↓
React
```

---

# 2. تثبيت Axios

في مشروع React + TypeScript:

```bash
npm install axios
```

وبعدها:

```ts
import axios from "axios";
```

دلوقتي عندك object اسمه:

```ts
axios
```

ومن خلاله تقدر تعمل:

```ts
axios.get()
axios.post()
axios.put()
axios.patch()
axios.delete()
```

وده أول ارتباط مباشر بين اللي درسناه والـ Axios.

---

# 3. أول GET Request

افترض عندنا API:

```text
GET https://api.example.com/users
```

بـ Axios:

```ts
const response = await axios.get(
  "https://api.example.com/users"
);
```

لاحظ التشابه:

```text
HTTP:
GET /users

Axios:
axios.get("/users")
```

Axios هنا بتترجم استدعاء JavaScript إلى HTTP Request حقيقي.

---

# 4. نفهم `await`

عندنا:

```ts
const response = await axios.get("/users");
```

عملية الاتصال بالسيرفر **مش بتحصل فورًا**.

الطلب بياخد وقت:

```text
React
  ↓
Request
  ↓
Internet
  ↓
Server
  ↓
Response
```

لذلك Axios بترجع:

```ts
Promise
```

وإحنا بنستخدم:

```ts
await
```

علشان نستنى نتيجة الـ Promise.

---

# 5. `response` مش الـ Data نفسها

دي نقطة مهمة جدًا.

لما تعمل:

```ts
const response = await axios.get("/users");
```

`response` مش معناها مباشرة:

```ts
[
  { id: 1, name: "Ahmed" }
]
```

Axios بترجع **Response Object** يحتوي على معلومات عن الـ Response.

أهم حاجة فيه:

```ts
response.data
```

مثلاً:

```ts
const response = await axios.get("/users");

console.log(response.data);
```

لو السيرفر رجع:

```json
[
  {
    "id": 1,
    "name": "Ahmed"
  }
]
```

يبقى:

```ts
response.data
```

هي:

```ts
[
  {
    id: 1,
    name: "Ahmed"
  }
]
```

---

# 6. ليه Axios عندها `response.data`؟

لأن الـ Response مش عبارة عن Data فقط.

السيرفر ممكن يرجع:

```text
Response
├── data
├── status
├── headers
├── statusText
└── ...
```

مثلاً:

```ts
response.status
```

ممكن تكون:

```text
200
```

و:

```ts
response.data
```

هي بيانات الـ API.

---

# 7. أول مثال TypeScript

لو الـ API بيرجع Users:

```ts
interface User {
  id: number;
  name: string;
  email: string;
}
```

ممكن نكتب:

```ts
const response = await axios.get<User[]>("/users");
```

وهنا:

```ts
User[]
```

بتقول لـ Axios وTypeScript:

> أنا متوقع إن `data` تكون Array من Users.

وبالتالي:

```ts
response.data
```

يكون TypeScript عارف إنه:

```ts
User[]
```

وده أول مكان هنبدأ فيه ندخل **TypeScript بشكل احترافي مع Axios**.

---

# 8. GET مع Query Parameters

من Lesson 1 عرفنا:

```http
GET /users?page=2&limit=10
```

في Axios ممكن تكتب:

```ts
const response = await axios.get("/users", {
  params: {
    page: 2,
    limit: 10,
  },
});
```

Axios هتعمل Request قريب من:

```http
GET /users?page=2&limit=10
```

وده أفضل من إنك تبني الـ URL يدويًا.

---

# 9. GET مع Headers

لو عندك Header:

```http
Authorization: Bearer abc123
```

ممكن:

```ts
const response = await axios.get("/users", {
  headers: {
    Authorization: "Bearer abc123",
  },
});
```

لاحظ إننا هنا بنترجم اللي اتعلمناه في HTTP إلى Axios:

```text
HTTP
↓
Authorization Header

Axios
↓
headers: {
  Authorization: "Bearer abc123"
}
```

---

# 10. الصورة الكاملة

دلوقتي عندك:

```ts
const response = await axios.get<User[]>("/users", {
  params: {
    page: 2,
    limit: 10,
  },
  headers: {
    Authorization: "Bearer abc123",
  },
});
```

وده معناه:

```text
Method
↓
GET

Path
↓
/users

Query Parameters
↓
page=2
limit=10

Headers
↓
Authorization

Response
↓
response.data
```

---

# 🧪 تدريب Lesson 2

عندك:

```text
GET /products
```

وعايز:

```text
page = 3
limit = 20
```

والـ API محتاجة:

```text
Authorization: Bearer abc123
```

والـ Response عبارة عن:

```ts
Product[]
```

اكتب **Axios Request فقط**.

ممنوع تستخدم أي حاجة لسه ما شرحناهاش، مثل:

```text
axios.create
interceptors
TanStack Query
hooks
```

**الخطوة التالية: اكتب كود الـ `axios.get` للتدريب، وأنا أصححه سطرًا سطرًا.**
<details>
<summary>const response = awite axios.get&lt;products[]&gt;("/products",</summary>
user
const response = awite axios.get<products[]>("/products",
peams: {
page = 3,
limit = 20,
}
header: {
Authorization: Bearer abc123,
}

)
ChatGPT
**الهدف: نصحح أول Axios Request بتاعك فقط، ونثبت شكل الـ config قبل ما نكمل Lesson 2.** مستواك هنا جيد، والفكرة الأساسية عندك صحيحة، والتصحيح يحتاج **5 دقائق**.

أنت فاهم الـ structure، لكن عندك أخطاء Syntax وأسماء Properties.

### الكود الصحيح

```ts
const response = await axios.get<Product[]>("/products", {
  params: {
    page: 3,
    limit: 20,
  },
  headers: {
    Authorization: "Bearer abc123",
  },
});
```

### صححنا إيه؟

1. ❌

```ts
awite
```

✅

```ts
await
```

2. ❌

```ts
products[]
```

لو الـ type اسمه `Product`:

✅

```ts
Product[]
```

3. ❌

```ts
peams
```

✅

```ts
params
```

4. ❌ داخل object:

```ts
page = 3
```

✅

```ts
page: 3
```

في JavaScript وTypeScript:

```text
key: value
```

وليس:

```text
key = value
```

5. ❌

```ts
header
```

✅

```ts
headers
```

لأن Axios تنتظر Property اسمها `headers`.

6. ❌

```ts
Authorization: Bearer abc123
```

لأن `Bearer abc123` قيمة نصية.

✅

```ts
Authorization: "Bearer abc123"
```

---

## أهم حاجة في الكود

بص على الجزء ده:

```ts
axios.get<Product[]>("/products", {
  params: {
    page: 3,
    limit: 20,
  },
  headers: {
    Authorization: "Bearer abc123",
  },
});
```

الـ `axios.get` هنا عندها:

```text
1️⃣ URL
2️⃣ Config Object
```

والـ Config Object يحتوي:

```text
params
headers
```

وده اللي هنفهمه بالتفصيل بعد كده.

### 🎯 سؤال واحد فقط

في الكود ده:

```ts
axios.get<Product[]>("/products", {
  params: {
    page: 3,
    limit: 20,
  },
});
```

**`Product[]` بتعمل إيه؟**

جاوب بجملة واحدة فقط.
</details>
const response = awite axios.get<products[]>("/products",
peams: {
page = 3,
limit = 20,
}
header: {
Authorization: Bearer abc123,
}

)
ChatGPT
**الهدف: نصحح أول Axios Request بتاعك فقط، ونثبت شكل الـ config قبل ما نكمل Lesson 2.** مستواك هنا جيد، والفكرة الأساسية عندك صحيحة، والتصحيح يحتاج **5 دقائق**.

أنت فاهم الـ structure، لكن عندك أخطاء Syntax وأسماء Properties.

### الكود الصحيح

```ts
const response = await axios.get<Product[]>("/products", {
  params: {
    page: 3,
    limit: 20,
  },
  headers: {
    Authorization: "Bearer abc123",
  },
});
```

### صححنا إيه؟

1. ❌

```ts
awite
```

✅

```ts
await
```

2. ❌

```ts
products[]
```

لو الـ type اسمه `Product`:

✅

```ts
Product[]
```

3. ❌

```ts
peams
```

✅

```ts
params
```

4. ❌ داخل object:

```ts
page = 3
```

✅

```ts
page: 3
```

في JavaScript وTypeScript:

```text
key: value
```

وليس:

```text
key = value
```

5. ❌

```ts
header
```

✅

```ts
headers
```

لأن Axios تنتظر Property اسمها `headers`.

6. ❌

```ts
Authorization: Bearer abc123
```

لأن `Bearer abc123` قيمة نصية.

✅

```ts
Authorization: "Bearer abc123"
```

---

## أهم حاجة في الكود

بص على الجزء ده:

```ts
axios.get<Product[]>("/products", {
  params: {
    page: 3,
    limit: 20,
  },
  headers: {
    Authorization: "Bearer abc123",
  },
});
```

الـ `axios.get` هنا عندها:

```text
1️⃣ URL
2️⃣ Config Object
```

والـ Config Object يحتوي:

```text
params
headers
```

وده اللي هنفهمه بالتفصيل بعد كده.

### 🎯 سؤال واحد فقط

في الكود ده:

```ts
axios.get<Product[]>("/products", {
  params: {
    page: 3,
    limit: 20,
  },
});
```

**`Product[]` بتعمل إيه؟**

جاوب بجملة واحدة فقط.
<details>
<summary>انا متوقع array من product</summary>
user
انا متوقع array من product
ChatGPT
**الهدف: تثبيت فهم الـ Generic في Axios قبل الانتقال للنقطة التالية.** إجابتك **صحيحة 100%**، وده بالضبط معنى `Product[]`. الوقت: **دقيقتان**.

```ts id="0l5c4w"
axios.get<Product[]>("/products");
```

معناها:

> أنا متوقع إن `response.data` تكون Array من `Product`.

مثلاً:

```ts id="3kgk8s"
const response = await axios.get<Product[]>("/products");

response.data
```

TypeScript هيفهمها كالتالي:

```ts id="0n8xmw"
Product[]
```

ولو:

```ts id="4tqj9k"
interface Product {
  id: number;
  name: string;
  price: number;
}
```

فـ:

```ts id="6u7w2x"
response.data[0].name
```

TypeScript يعرف إن `name`:

```ts
string
```

و:

```ts id="m8u3g1"
response.data[0].price
```

يعرف إن `price`:

```ts
number
```

**مهم:** `Product[]` لا يحوّل البيانات القادمة من السيرفر إلى Product، هو فقط يخبر **TypeScript** بالشكل المتوقع للبيانات.

**الخطوة التالية: نكمل Lesson 2 مع فهم `axios.get<T>()` نفسه، ولماذا نضع `T` بين `< >`.**
</details>
انا متوقع array من product
ChatGPT
**الهدف: تثبيت فهم الـ Generic في Axios قبل الانتقال للنقطة التالية.** إجابتك **صحيحة 100%**، وده بالضبط معنى `Product[]`. الوقت: **دقيقتان**.

```ts id="0l5c4w"
axios.get<Product[]>("/products");
```

معناها:

> أنا متوقع إن `response.data` تكون Array من `Product`.

مثلاً:

```ts id="3kgk8s"
const response = await axios.get<Product[]>("/products");

response.data
```

TypeScript هيفهمها كالتالي:

```ts id="0n8xmw"
Product[]
```

ولو:

```ts id="4tqj9k"
interface Product {
  id: number;
  name: string;
  price: number;
}
```

فـ:

```ts id="6u7w2x"
response.data[0].name
```

TypeScript يعرف إن `name`:

```ts
string
```

و:

```ts id="m8u3g1"
response.data[0].price
```

يعرف إن `price`:

```ts
number
```

**مهم:** `Product[]` لا يحوّل البيانات القادمة من السيرفر إلى Product، هو فقط يخبر **TypeScript** بالشكل المتوقع للبيانات.

**الخطوة التالية: نكمل Lesson 2 مع فهم `axios.get<T>()` نفسه، ولماذا نضع `T` بين `< >`.**
<details>
<summary>كمل</summary>
user
كمل
</details>
كمل