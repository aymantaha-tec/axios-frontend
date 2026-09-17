# 1. يعني إيه `Response Type`؟

لما الـ API يرجع بيانات، لازم تعرف **شكل البيانات دي**.

مثلاً الـ API يرجع User:

```json
{
  "id": 10,
  "name": "Mohamed",
  "email": "mohamed@example.com",
  "age": 30
}
```

إذن نعمل Type يمثل **الشكل ده**:

```ts
interface IUser {
  id: number;
  name: string;
  email: string;
  age: number;
}
```

وده هو الـ **Response Type** في الحالة دي.

---

# 2. نستخدمه مع Axios

```ts
const response = await axios.get<IUser>("/users/10");
```

هنا:

```text
API Response
     ↓
response.data
     ↓
IUser
```

فـ TypeScript يعرف إن:

```ts
response.data.id
```

نوعها:

```text
number
```

و:

```ts
response.data.name
```

نوعها:

```text
string
```

---

# 3. لو الـ API بيرجع Array

ممكن الـ API يرجع:

```json
[
  {
    "id": 1,
    "name": "Ahmed",
    "email": "ahmed@example.com"
  },
  {
    "id": 2,
    "name": "Mohamed",
    "email": "mohamed@example.com"
  }
]
```

هنا الـ Response Type مش `IUser`.

هو:

```ts
IUser[]
```

فنكتب:

```ts
const response = await axios.get<IUser[]>("/users");
```

وبالتالي:

```text
response.data → IUser[]
```

---

# 4. أهم نقطة: الـ Type لازم يطابق الـ API

لو الـ API بيرجع:

```json
{
  "id": 10,
  "name": "Mohamed"
}
```

ممكن يكون:

```ts
interface IUser {
  id: number;
  name: string;
}
```

لكن لو الـ API بيرجع:

```json
{
  "id": 10,
  "name": "Mohamed",
  "email": "mohamed@example.com"
}
```

فالـ Type الأفضل يمثل الـ Response الحقيقي:

```ts
interface IUser {
  id: number;
  name: string;
  email: string;
}
```

الفكرة الأساسية:

> **Response Type هو وصف لشكل البيانات التي تتوقع أن تكون موجودة في `response.data`.**

---

# 5. مثال مهم جدًا

افترض أن عندك API:

```text
GET /products/5
```

والـ API بيرجع:

```json
{
  "id": 5,
  "title": "Laptop",
  "price": 25000,
  "inStock": true
}
```

نعمل:

```ts
interface IProduct {
  id: number;
  title: string;
  price: number;
  inStock: boolean;
}
```

ثم:

```ts
const response = await axios.get<IProduct>(
  "/products/5"
);
```

دلوقتي:

```text
response
   ↓
AxiosResponse<IProduct>
   ↓
response.data
   ↓
IProduct
```

---

# 6. طب لو فيه Object جواه Object؟

مثلاً:

```json
{
  "id": 5,
  "title": "Laptop",
  "category": {
    "id": 2,
    "name": "Electronics"
  }
}
```

الـ Type لازم يعكس نفس الـ structure:

```ts
interface ICategory {
  id: number;
  name: string;
}

interface IProduct {
  id: number;
  title: string;
  category: ICategory;
}
```

وبعدين:

```ts
const response = await axios.get<IProduct>(
  "/products/5"
);
```

فتقدر تعمل:

```ts
response.data.category.name
```

وTypeScript عارف إن:

```text
category → ICategory
name     → string
```

---

# 7. طب إزاي أعرف الـ Response Type؟

دي مهارة مهمة جدًا في شغل الـ Frontend.

مش المفروض تخمن.

لو عندك API Documentation أو Swagger، هتشوف الـ Response.

مثلاً:

```json
{
  "id": 1,
  "title": "Laptop",
  "price": 50000
}
```

تحولها إلى:

```ts
interface IProduct {
  id: number;
  title: string;
  price: number;
}
```

ثم:

```ts
axios.get<IProduct>("/products/1");
```

يعني العملية:

```text
API Response
     ↓
نفهم الـ JSON
     ↓
نكتب TypeScript Type
     ↓
نضعه داخل Generic
     ↓
axios.get<IProduct>()
```

---

# 8. نقطة مهمة جدًا

`IUser` مش لازم يكون اسمه `IUser`.

ده مجرد naming convention.

ممكن تكتب:

```ts
type User = {
  id: number;
  name: string;
};
```

وتستخدم:

```ts
axios.get<User>("/users/1");
```

الاسم مش هو المهم.

**المهم إن الـ Type يمثل شكل الـ Response.**

---

## 🎯 تمرين

الـ API عندك بيرجع:

```json
{
  "id": 7,
  "title": "MacBook",
  "price": 50000,
  "available": true
}
```

عايز منك حاجتين فقط:

1. اكتب `interface` مناسبة للـ Response.
2. اكتب `axios.get` باستخدام الـ interface دي كـ Generic.

**الخطوة التالية:** اكتب الـ `interface` والـ `axios.get` بنفسك.
<details>
<summary>interface IProduct {</summary>
user
interface IProduct {
id: number,
title: string,
price: number,
available: boolean,
}
const response = await axios.get<IProduct>("/products")؛
ChatGPT
إجابتك **ممتازة من ناحية الـ Type**، لكن عندنا نقطة واحدة مهمة جدًا في الـ URL. ⏱️ 3 دقائق

أنت كتبت:

```ts
interface IProduct {
  id: number;
  title: string;
  price: number;
  available: boolean;
}

const response = await axios.get<IProduct>("/products");
```

الـ `interface` **صحيحة 100%**.

لكن السؤال: هل `/products` بيرجع **Product واحد** ولا **Array من Products**؟

لو الـ API:

```text
GET /products
```

بيرجع:

```json
[
  {
    "id": 7,
    "title": "MacBook",
    "price": 50000,
    "available": true
  },
  {
    "id": 8,
    "title": "iPhone",
    "price": 30000,
    "available": true
  }
]
```

يبقى لازم:

```ts
const response = await axios.get<IProduct[]>("/products");
```

لأن:

```text
/products
   ↓
IProduct[]
```

أما لو:

```text
GET /products/7
```

بيرجع Product واحد:

```json
{
  "id": 7,
  "title": "MacBook",
  "price": 50000,
  "available": true
}
```

يبقى كودك صحيح:

```ts
const response = await axios.get<IProduct>("/products/7");
```

### 🧠 القاعدة

```text
GET /products
      ↓
غالبًا مجموعة
      ↓
IProduct[]

GET /products/7
      ↓
Resource واحد
      ↓
IProduct
```

وده مهم جدًا: **الـ Response Type بيتحدد من شكل الـ JSON الحقيقي اللي الـ API بيرجعه، مش من اسم الـ endpoint فقط.**

**الخطوة التالية:** لو `/products` بيرجع Array، اكتب لي السطر الصحيح باستخدام `IProduct[]`.
</details>
interface IProduct {
id: number,
title: string,
price: number,
available: boolean,
}
const response = await axios.get<IProduct>("/products")؛
ChatGPT
إجابتك **ممتازة من ناحية الـ Type**، لكن عندنا نقطة واحدة مهمة جدًا في الـ URL. ⏱️ 3 دقائق

أنت كتبت:

```ts
interface IProduct {
  id: number;
  title: string;
  price: number;
  available: boolean;
}

const response = await axios.get<IProduct>("/products");
```

الـ `interface` **صحيحة 100%**.

لكن السؤال: هل `/products` بيرجع **Product واحد** ولا **Array من Products**؟

لو الـ API:

```text
GET /products
```

بيرجع:

```json
[
  {
    "id": 7,
    "title": "MacBook",
    "price": 50000,
    "available": true
  },
  {
    "id": 8,
    "title": "iPhone",
    "price": 30000,
    "available": true
  }
]
```

يبقى لازم:

```ts
const response = await axios.get<IProduct[]>("/products");
```

لأن:

```text
/products
   ↓
IProduct[]
```

أما لو:

```text
GET /products/7
```

بيرجع Product واحد:

```json
{
  "id": 7,
  "title": "MacBook",
  "price": 50000,
  "available": true
}
```

يبقى كودك صحيح:

```ts
const response = await axios.get<IProduct>("/products/7");
```

### 🧠 القاعدة

```text
GET /products
      ↓
غالبًا مجموعة
      ↓
IProduct[]

GET /products/7
      ↓
Resource واحد
      ↓
IProduct
```

وده مهم جدًا: **الـ Response Type بيتحدد من شكل الـ JSON الحقيقي اللي الـ API بيرجعه، مش من اسم الـ endpoint فقط.**

**الخطوة التالية:** لو `/products` بيرجع Array، اكتب لي السطر الصحيح باستخدام `IProduct[]`.
<details>
<summary>كمل</summary>
user
كمل
ChatGPT
كمّل دلوقتي للجزء الأهم في `Response Types`: إزاي تفرق بين **Object** و **Array** و **Nested Object** في الـ API Response. ⏱️ 12 دقيقة

## 📍 Roadmap

```text
SECTION 0: Architecture Foundation       ⬜
SECTION 1: Axios Fundamentals             ✅

SECTION 2: TypeScript + Axios             🟢
├── AxiosResponse                         ✅
├── Generics مع Axios                     ✅
├── Response Types                        🟢 ← هنا
│   ├── Object                            ✅
│   ├── Array                             🟢
│   └── Nested Object                     ⬜
├── API Response Types                    ⬜
├── Discriminated Unions                  ⬜
├── unknown vs any                        ⬜
└── Type-safe API Layer                   ⬜

SECTION 3: Error Handling                 ⬜
SECTION 4: Interceptors                   ⬜
...
```
</details>
كمل
ChatGPT
كمّل دلوقتي للجزء الأهم في `Response Types`: إزاي تفرق بين **Object** و **Array** و **Nested Object** في الـ API Response. ⏱️ 12 دقيقة

## 📍 Roadmap

```text
SECTION 0: Architecture Foundation       ⬜
SECTION 1: Axios Fundamentals             ✅

SECTION 2: TypeScript + Axios             🟢
├── AxiosResponse                         ✅
├── Generics مع Axios                     ✅
├── Response Types                        🟢 ← هنا
│   ├── Object                            ✅
│   ├── Array                             🟢
│   └── Nested Object                     ⬜
├── API Response Types                    ⬜
├── Discriminated Unions                  ⬜
├── unknown vs any                        ⬜
└── Type-safe API Layer                   ⬜

SECTION 3: Error Handling                 ⬜
SECTION 4: Interceptors                   ⬜
...
```