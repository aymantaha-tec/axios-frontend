# `params` يعني إيه؟

في HTTP عرفنا إن عندنا **Query Parameters**:

```http
GET /products?page=2&limit=10
```

في Axios نقدر نكتبها بشكل منظم باستخدام:

```ts
params
```

مثال:

```ts
const response = await axios.get("/products", {
  params: {
    page: 2,
    limit: 10,
  },
});
```

Axios تحولها إلى Request بالشكل:

```http
GET /products?page=2&limit=10
```

---

# 1. `params` لا تعني Body

دي نقطة مهمة جدًا.

```ts
axios.get("/products", {
  params: {
    page: 2,
    limit: 10,
  },
});
```

هنا:

```text
params
  ↓
Query Parameters
  ↓
URL
```

يعني Axios في النهاية تبني:

```text
/products?page=2&limit=10
```

---

# 2. أكثر من Parameter

مثلاً:

```ts
axios.get("/products", {
  params: {
    page: 2,
    limit: 10,
    category: "phones",
  },
});
```

النتيجة تقريبًا:

```http
GET /products?page=2&limit=10&category=phones
```

يعني:

```text
page
↓
2

limit
↓
10

category
↓
phones
```

---

# 3. ليه نستخدم `params` بدل كتابة URL يدويًا؟

ممكن تعمل:

```ts
axios.get(
  "/products?page=2&limit=10"
);
```

لكن الأفضل:

```ts
axios.get("/products", {
  params: {
    page: 2,
    limit: 10,
  },
});
```

لأن Axios هي المسؤولة عن بناء الـ Query String.

وأنت بتتعامل مع JavaScript object واضح.

---

# 4. `params` مع TypeScript

ممكن يكون عندك Type للـ Query Parameters:

```ts
interface GetProductsParams {
  page: number;
  limit: number;
  category?: string;
}
```

ثم:

```ts
const params: GetProductsParams = {
  page: 2,
  limit: 10,
  category: "phones",
};
```

وبعدين:

```ts
const response = await axios.get<Product[]>("/products", {
  params,
});
```

هنا عندنا نوعين مختلفين:

```text
Product[]
↓
Response

GetProductsParams
↓
Query Parameters
```

نفس الفكرة اللي اتعلمناها مع:

```text
CreateProductRequest
↓
Request Body
```

---

# 🧠 اربطها كده

```ts
axios.get<Product[]>("/products", {
  params: {
    page: 2,
    limit: 10,
  },
});
```

اقرأها:

```text
<Product[]>
↓
أنا متوقع Response من نوع Product[]

"/products"
↓
الـ endpoint

params
↓
إعدادات الـ Query Parameters

page / limit
↓
قيم الـ Query Parameters
```

---

# 🧪 تدريب

عندك:

```ts
interface Product {
  id: number;
  name: string;
  price: number;
}

interface GetProductsParams {
  page: number;
  limit: number;
  category?: string;
}
```

عايز تجيب:

```text
page = 3
limit = 20
category = "laptops"
```

والـ Response المتوقع:

```text
Product[]
```

اكتب:

```ts
const response = ...
```

**الخطوة التالية: اكتب `axios.get()` باستخدام `Product[]` و `params` فقط.**
<details>
<summary>Convert to writing block</summary>
user
Convert to writing block
ChatGPT
:::writing{variant="standard" id="48317"}
category: "phones",
:::
</details>
Convert to writing block
ChatGPT
:::writing{variant="standard" id="48317"}
category: "phones",
:::
<details>
<summary>const params: GetProductsParams  = {</summary>
user
const params: GetProductsParams  = {
page: 3,
 limit: 20,
 category: "laptops" ,
}
const response =  await axios.get<Product[]>("/Products",  {
params,
})
ChatGPT
**الهدف: تصحيح تدريب `params` فقط، وإجابتك صحيحة تقريبًا بالكامل.** الوقت: **3 دقائق**.

### ✅ الكود الصحيح

```ts
const params: GetProductsParams = {
  page: 3,
  limit: 20,
  category: "laptops",
};

const response = await axios.get<Product[]>("/products", {
  params,
});
```

### تقييمك

```text
GetProductsParams        ✅
page                     ✅
limit                    ✅
category                 ✅
axios.get<Product[]>     ✅
params داخل config       ✅
```

عندك ملاحظة واحدة فقط:

```ts
"/Products"
```

الأفضل:

```ts
"/products"
```

لأننا عادةً نتعامل مع الـ endpoint بنفس الـ naming convention الذي يحدده الـ API، وفي كثير من الأنظمة تكون الـ URLs case-sensitive.

### 🧠 أهم حاجة فهمتها

أنت عملت:

```ts
const params: GetProductsParams = {
  page: 3,
  limit: 20,
  category: "laptops",
};
```

ثم:

```ts
axios.get<Product[]>("/products", {
  params,
});
```

فالنتيجة تكون تقريبًا:

```http
GET /products?page=3&limit=20&category=laptops
```

والـ types:

```text
GetProductsParams
↓
Query Parameters

Product[]
↓
Response Data
```

## 📊 Progress

```text
SECTION 1: Axios Fundamentals

GET                  ✅
POST                 ✅
PUT                  ✅
PATCH                ✅
DELETE               ✅

Request Configuration
├── Config Object     ✅
├── params            ✅ ← خلصناه
├── headers           ⬅️ التالي
├── data              ⬜
├── timeout           ⬜
└── signal            ⬜
```

**الخطوة التالية: نبدأ `headers` فقط.**
:::
</details>
const params: GetProductsParams  = {
page: 3,
 limit: 20,
 category: "laptops" ,
}
const response =  await axios.get<Product[]>("/Products",  {
params,
})
ChatGPT
**الهدف: تصحيح تدريب `params` فقط، وإجابتك صحيحة تقريبًا بالكامل.** الوقت: **3 دقائق**.

### ✅ الكود الصحيح

```ts
const params: GetProductsParams = {
  page: 3,
  limit: 20,
  category: "laptops",
};

const response = await axios.get<Product[]>("/products", {
  params,
});
```

### تقييمك

```text
GetProductsParams        ✅
page                     ✅
limit                    ✅
category                 ✅
axios.get<Product[]>     ✅
params داخل config       ✅
```

عندك ملاحظة واحدة فقط:

```ts
"/Products"
```

الأفضل:

```ts
"/products"
```

لأننا عادةً نتعامل مع الـ endpoint بنفس الـ naming convention الذي يحدده الـ API، وفي كثير من الأنظمة تكون الـ URLs case-sensitive.

### 🧠 أهم حاجة فهمتها

أنت عملت:

```ts
const params: GetProductsParams = {
  page: 3,
  limit: 20,
  category: "laptops",
};
```

ثم:

```ts
axios.get<Product[]>("/products", {
  params,
});
```

فالنتيجة تكون تقريبًا:

```http
GET /products?page=3&limit=20&category=laptops
```

والـ types:

```text
GetProductsParams
↓
Query Parameters

Product[]
↓
Response Data
```

## 📊 Progress

```text
SECTION 1: Axios Fundamentals

GET                  ✅
POST                 ✅
PUT                  ✅
PATCH                ✅
DELETE               ✅

Request Configuration
├── Config Object     ✅
├── params            ✅ ← خلصناه
├── headers           ⬅️ التالي
├── data              ⬜
├── timeout           ⬜
└── signal            ⬜
```

**الخطوة التالية: نبدأ `headers` فقط.**
:::
<details>
<summary>`headers`</summary>
user
`headers`   
ChatGPT
**الهدف: نفهم `headers` في Axios من الصفر، ونربطها بما درسناه في HTTP، بدون الدخول في `data` أو Interceptors.** الوقت: **10 دقائق**.

# 📍 Roadmap

```text
SECTION 1: Axios Fundamentals 🟢

GET                  ✅
POST                 ✅
PUT                  ✅
PATCH                ✅
DELETE               ✅

Request Configuration 🟢
├── Config Object     ✅
├── params            ✅
├── headers           🟢 ← نحن هنا
├── data              ⬜
├── timeout           ⬜
└── signal            ⬜
```
</details>
`headers`   
ChatGPT
**الهدف: نفهم `headers` في Axios من الصفر، ونربطها بما درسناه في HTTP، بدون الدخول في `data` أو Interceptors.** الوقت: **10 دقائق**.

# 📍 Roadmap

```text
SECTION 1: Axios Fundamentals 🟢

GET                  ✅
POST                 ✅
PUT                  ✅
PATCH                ✅
DELETE               ✅

Request Configuration 🟢
├── Config Object     ✅
├── params            ✅
├── headers           🟢 ← نحن هنا
├── data              ⬜
├── timeout           ⬜
└── signal            ⬜
```