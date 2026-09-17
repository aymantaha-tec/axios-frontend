# 1. الـ API ممكن يرجع Object واحد

مثلاً:

```json
{
  "id": 7,
  "title": "MacBook",
  "price": 50000
}
```

نعمل:

```ts
interface IProduct {
  id: number;
  title: string;
  price: number;
}
```

وبعدين:

```ts
const response = await axios.get<IProduct>(
  "/products/7"
);
```

إذن:

```text
response.data → IProduct
```

---

# 2. ممكن يرجع Array

لو الـ API رجع:

```json
[
  {
    "id": 7,
    "title": "MacBook",
    "price": 50000
  },
  {
    "id": 8,
    "title": "iPhone",
    "price": 30000
  }
]
```

نستخدم:

```ts
const response = await axios.get<IProduct[]>(
  "/products"
);
```

لاحظ الفرق:

```text
IProduct
   ↓
Object واحد

IProduct[]
   ↓
Array من Objects
```

وبالتالي تقدر تعمل:

```ts
response.data.map((product) => {
  console.log(product.title);
});
```

لأن:

```text
response.data → IProduct[]
product       → IProduct
```

---

# 3. Nested Object

دلوقتي نرفع المستوى شوية.

الـ API رجع:

```json
{
  "id": 7,
  "title": "MacBook",
  "price": 50000,
  "category": {
    "id": 2,
    "name": "Laptops"
  }
}
```

هنا `category` نفسها Object.

فنكتب:

```ts
interface ICategory {
  id: number;
  name: string;
}

interface IProduct {
  id: number;
  title: string;
  price: number;
  category: ICategory;
}
```

ثم:

```ts
const response = await axios.get<IProduct>(
  "/products/7"
);
```

دلوقتي TypeScript فاهم:

```text
response.data
      ↓
IProduct
      ↓
category
      ↓
ICategory
```

فتقدر تعمل:

```ts
console.log(response.data.category.name);
```

والـ TypeScript يعرف إن:

```text
category.name → string
```

---

# 4. Nested Array

ممكن كمان الـ Response يكون:

```json
{
  "id": 7,
  "title": "MacBook",
  "tags": [
    "Apple",
    "Laptop",
    "M3"
  ]
}
```

هنا:

```ts
interface IProduct {
  id: number;
  title: string;
  tags: string[];
}
```

لكن لو `tags` Objects:

```json
{
  "id": 7,
  "title": "MacBook",
  "tags": [
    {
      "id": 1,
      "name": "Apple"
    },
    {
      "id": 2,
      "name": "Laptop"
    }
  ]
}
```

نعمل:

```ts
interface ITag {
  id: number;
  name: string;
}

interface IProduct {
  id: number;
  title: string;
  tags: ITag[];
}
```

لاحظ إنك بتبني الـ Type **نفس شكل الـ JSON بالضبط**.

---

# 5. أهم قاعدة في `Response Types`

بص على الـ JSON الأول.

ثم اسأل نفسك:

```text
هل هو Object؟
       ↓
     Type

هل هو Array؟
       ↓
    Type[]

هل داخله Object؟
       ↓
   Type آخر

هل داخله Array؟
       ↓
   Type[]
```

مثلاً:

```json
{
  "user": {
    "id": 1
  },
  "products": [
    {
      "id": 10
    }
  ]
}
```

ممكن نمثله:

```ts
interface IUser {
  id: number;
}

interface IProduct {
  id: number;
}

interface IResponse {
  user: IUser;
  products: IProduct[];
}
```

ثم:

```ts
const response = await axios.get<IResponse>("/dashboard");
```

فتصبح:

```text
response.data.user
       ↓
IUser

response.data.products
       ↓
IProduct[]
```

وده بالضبط اللي بنسميه **Type Modeling**، يعني تحويل شكل البيانات الحقيقي إلى Types في TypeScript.

### 🎯 Interview Question

عندك الـ API بيرجع:

```json
{
  "id": 10,
  "name": "Ahmed",
  "orders": [
    {
      "id": 1,
      "total": 500
    },
    {
      "id": 2,
      "total": 800
    }
  ]
}
```

اكتب `interfaces` المناسبة للـ Response فقط.

**الخطوة التالية:** اكتب `IOrder` و`IUser` من الـ JSON بنفسك.
<details>
<summary>interface IOrder {</summary>
user
interface IOrder {
id: 1,   
total: 500,
}

interface IUser {
id: 10, 
 name: "Ahmed",  
orders:IOrder[],
}
ChatGPT
صحح نوع القيم فقط، لأنك فهمت الـ structure والـ nested relationship بشكل ممتاز. ⏱️ 3 دقائق

أنت كتبت:

```ts
interface IOrder {
  id: 1,
  total: 500,
}
```

المشكلة هنا إنك استخدمت **قيم فعلية** بدل **Types**.

داخل الـ `interface` إحنا بنقول نوع القيمة، مش القيمة نفسها.

الصحيح:

```ts
interface IOrder {
  id: number;
  total: number;
}

interface IUser {
  id: number;
  name: string;
  orders: IOrder[];
}
```

### ليه؟

لما تكتب:

```ts
id: 1;
```

أنت فعليًا بتقول:

> `id` لازم تكون قيمتها `1` فقط.

لكن الـ API ممكن يرجع:

```json
{
  "id": 25
}
```

إحنا عايزين نقول:

> `id` ممكن تكون أي رقم.

إذن:

```ts
id: number;
```

ونفس الفكرة:

```ts
name: string;
```

مش:

```ts
name: "Ahmed";
```

---

### لكن أهم حاجة أنت عملتها صح 👇

```ts
orders: IOrder[];
```

دي **ممتازة**.

لأن الـ JSON عندنا:

```json
"orders": [
  {
    "id": 1,
    "total": 500
  },
  {
    "id": 2,
    "total": 800
  }
]
```

إذن:

```text
orders
  ↓
Array
  ↓
IOrder[]
```

والـ structure بقى:

```text
IUser
│
├── id       → number
├── name     → string
└── orders   → IOrder[]
                 │
                 ├── id    → number
                 └── total → number
```

**تقييمك: 9/10**. الـ modeling نفسه ممتاز، والخطأ فقط في الفرق بين **value** و **type** داخل `interface`.

**الخطوة التالية:** اكتب `const response = await axios.get<...>` بحيث يكون الـ Generic مناسبًا للـ `IUser` اللي عملناه.
</details>
interface IOrder {
id: 1,   
total: 500,
}

interface IUser {
id: 10, 
 name: "Ahmed",  
orders:IOrder[],
}
ChatGPT
صحح نوع القيم فقط، لأنك فهمت الـ structure والـ nested relationship بشكل ممتاز. ⏱️ 3 دقائق

أنت كتبت:

```ts
interface IOrder {
  id: 1,
  total: 500,
}
```

المشكلة هنا إنك استخدمت **قيم فعلية** بدل **Types**.

داخل الـ `interface` إحنا بنقول نوع القيمة، مش القيمة نفسها.

الصحيح:

```ts
interface IOrder {
  id: number;
  total: number;
}

interface IUser {
  id: number;
  name: string;
  orders: IOrder[];
}
```

### ليه؟

لما تكتب:

```ts
id: 1;
```

أنت فعليًا بتقول:

> `id` لازم تكون قيمتها `1` فقط.

لكن الـ API ممكن يرجع:

```json
{
  "id": 25
}
```

إحنا عايزين نقول:

> `id` ممكن تكون أي رقم.

إذن:

```ts
id: number;
```

ونفس الفكرة:

```ts
name: string;
```

مش:

```ts
name: "Ahmed";
```

---

### لكن أهم حاجة أنت عملتها صح 👇

```ts
orders: IOrder[];
```

دي **ممتازة**.

لأن الـ JSON عندنا:

```json
"orders": [
  {
    "id": 1,
    "total": 500
  },
  {
    "id": 2,
    "total": 800
  }
]
```

إذن:

```text
orders
  ↓
Array
  ↓
IOrder[]
```

والـ structure بقى:

```text
IUser
│
├── id       → number
├── name     → string
└── orders   → IOrder[]
                 │
                 ├── id    → number
                 └── total → number
```

**تقييمك: 9/10**. الـ modeling نفسه ممتاز، والخطأ فقط في الفرق بين **value** و **type** داخل `interface`.

**الخطوة التالية:** اكتب `const response = await axios.get<...>` بحيث يكون الـ Generic مناسبًا للـ `IUser` اللي عملناه.
<details>
<summary>const response = await axios.get&lt;IUser&gt;("orders/");</summary>
user
const response = await axios.get<IUser>("orders/");
ChatGPT
إجابتك **صحيحة 100% من ناحية الـ Generic والـ Type Modeling**. ⏱️ دقيقتان

```ts id="g5h7mz"
const response = await axios.get<IUser>("/orders/");
```

لأن الـ API في التمرين بيرجع **Object واحد** بالشكل:

```text id="wz8z7v"
{
  id,
  name,
  orders: [...]
}
```

وده مطابق لـ:

```ts id="z8u2gp"
interface IUser {
  id: number;
  name: string;
  orders: IOrder[];
}
```

إذن:

```text id="t4gk0q"
response
   ↓
AxiosResponse<IUser>

response.data
   ↓
IUser

response.data.orders
   ↓
IOrder[]
```

### ملاحظة صغيرة

في المثال بتاعنا كان الـ endpoint افتراضيًا مش محدد، فـ `"/orders/"` مقبول كتمرين من ناحية الـ Type.

لكن في مشروع حقيقي، **الـ URL لازم يطابق الـ API contract الفعلي**. مش الـ Type هو اللي بيحدد الـ endpoint.

### 📍 Progress

```text id="w0a3j4"
SECTION 2: TypeScript + Axios

├── AxiosResponse                         ✅
├── Generics مع Axios                     ✅
├── Response Types                        ✅ ← خلصنا
├── API Response Types                    🟢 ← التالي
├── Discriminated Unions                  ⬜
├── unknown vs any                        ⬜
└── Type-safe API Layer                   ⬜
```

**الخطوة التالية:** نبدأ `API Response Types`، ونشوف الفرق بين `IUser` كـ Resource Type وبين الـ Response الكامل اللي الـ API ممكن يرجعه مثل `status`, `message`, و`payload`.
</details>
const response = await axios.get<IUser>("orders/");
ChatGPT
إجابتك **صحيحة 100% من ناحية الـ Generic والـ Type Modeling**. ⏱️ دقيقتان

```ts id="g5h7mz"
const response = await axios.get<IUser>("/orders/");
```

لأن الـ API في التمرين بيرجع **Object واحد** بالشكل:

```text id="wz8z7v"
{
  id,
  name,
  orders: [...]
}
```

وده مطابق لـ:

```ts id="z8u2gp"
interface IUser {
  id: number;
  name: string;
  orders: IOrder[];
}
```

إذن:

```text id="t4gk0q"
response
   ↓
AxiosResponse<IUser>

response.data
   ↓
IUser

response.data.orders
   ↓
IOrder[]
```

### ملاحظة صغيرة

في المثال بتاعنا كان الـ endpoint افتراضيًا مش محدد، فـ `"/orders/"` مقبول كتمرين من ناحية الـ Type.

لكن في مشروع حقيقي، **الـ URL لازم يطابق الـ API contract الفعلي**. مش الـ Type هو اللي بيحدد الـ endpoint.

### 📍 Progress

```text id="w0a3j4"
SECTION 2: TypeScript + Axios

├── AxiosResponse                         ✅
├── Generics مع Axios                     ✅
├── Response Types                        ✅ ← خلصنا
├── API Response Types                    🟢 ← التالي
├── Discriminated Unions                  ⬜
├── unknown vs any                        ⬜
└── Type-safe API Layer                   ⬜
```

**الخطوة التالية:** نبدأ `API Response Types`، ونشوف الفرق بين `IUser` كـ Resource Type وبين الـ Response الكامل اللي الـ API ممكن يرجعه مثل `status`, `message`, و`payload`.
<details>
<summary>API Response Types</summary>
user
API Response Types  
ChatGPT
ابدأ بـ **API Response Types**، والهدف هنا تفرق بين **الـ Resource نفسه** مثل `IUser` وبين **الـ Wrapper اللي الـ API بيغلف جواه البيانات** مثل `payload` و`status` و`code`. ⏱️ 15 دقيقة

## 📍 Roadmap

```text id="4n2r8m"
SECTION 0: Architecture Foundation       ⬜
SECTION 1: Axios Fundamentals             ✅

SECTION 2: TypeScript + Axios             🟢
├── AxiosResponse                         ✅
├── Generics مع Axios                     ✅
├── Response Types                        ✅
├── API Response Types                    🟢 ← هنا
├── Discriminated Unions                  ⬜
├── unknown vs any                        ⬜
└── Type-safe API Layer                   ⬜

SECTION 3: Error Handling                 ⬜
SECTION 4: Interceptors                   ⬜
...
```

# 1. المشكلة اللي بنحلها

لحد دلوقتي كنا بنفترض إن الـ API بيرجع User مباشرة:

```json id="7b0q4d"
{
  "id": 10,
  "name": "Ahmed",
  "email": "ahmed@example.com"
}
```

فكتبنا:

```ts id="d3j4eq"
interface IUser {
  id: number;
  name: string;
  email: string;
}

const response = await axios.get<IUser>("/users/10");
```

وساعتها:

```text id="8k8qpk"
response.data → IUser
```

لكن APIs الحقيقية كثيرًا ما تستخدم **Response Wrapper**.

---

# 2. مثال على API Response Wrapper

ممكن الـ API يرجع:

```json id="f7n4wq"
{
  "status": true,
  "code": 200,
  "payload": {
    "id": 10,
    "name": "Ahmed",
    "email": "ahmed@example.com"
  }
}
```

لاحظ إن `IUser` لم يعد هو الـ Response كله.

الـ Response عندنا أصبح:

```text id="j9x1r5"
API Response
│
├── status
├── code
└── payload
      ↓
    IUser
```

---

# 3. نمثل ده في TypeScript

أولًا:

```ts id="e7g5p2"
interface IUser {
  id: number;
  name: string;
  email: string;
}
```

ثم نعمل Type للـ Wrapper:

```ts id="9s4b7m"
interface IApiResponse<T> {
  status: boolean;
  code: number;
  payload: T;
}
```

ركز في:

```ts id="f8w2k1"
<T>
```

هنا استخدمنا Generic مرة أخرى.

معناه:

> الـ `payload` ممكن يكون أي Type، وأنا هحدد نوعه وقت الاستخدام.

---

# 4. استخدامه مع `IUser`

نكتب:

```ts id="8h9xk3"
const response = await axios.get<IApiResponse<IUser>>(
  "/users/10"
);
```

لاحظ إن عندنا **Generic داخل Generic**:

```text id="2q5r8t"
IApiResponse<IUser>
           ↑
        T = IUser
```

إذن TypeScript يفهم:

```text id="q8w4n2"
response
   ↓
AxiosResponse<IApiResponse<IUser>>
```

ثم:

```text id="m2k7sa"
response.data
   ↓
IApiResponse<IUser>
```

ثم:

```text id="w4f8qe"
response.data.payload
   ↓
IUser
```

---

# 5. ليه عملنا `IApiResponse<T>`؟

عشان نفس الـ API Wrapper ممكن يستخدم مع بيانات مختلفة.

مثلاً User:

```ts id="n6k3v1"
IApiResponse<IUser>
```

Product:

```ts id="e1z5pa"
IApiResponse<IProduct>
```

Order:

```ts id="d7x2qm"
IApiResponse<IOrder>
```

بدل ما نكتب 3 Interfaces مختلفة للـ Wrapper:

```ts id="4h9q2s"
interface IUserResponse {
  status: boolean;
  code: number;
  payload: IUser;
}

interface IProductResponse {
  status: boolean;
  code: number;
  payload: IProduct;
}

interface IOrderResponse {
  status: boolean;
  code: number;
  payload: IOrder;
}
```

نكتب واحدة فقط:

```ts id="8x3n7v"
interface IApiResponse<T> {
  status: boolean;
  code: number;
  payload: T;
}
```

وبعدين نحدد `T`.

---

# 6. مثال Product

لو الـ API رجع:

```json id="c8p4az"
{
  "status": true,
  "code": 200,
  "payload": {
    "id": 5,
    "title": "MacBook",
    "price": 50000
  }
}
```

نعمل:

```ts id="q2n8mw"
interface IProduct {
  id: number;
  title: string;
  price: number;
}
```

ثم:

```ts id="m5r7kx"
const response = await axios.get<IApiResponse<IProduct>>(
  "/products/5"
);
```

النتيجة:

```text id="f1s9qa"
response.data
      ↓
IApiResponse<IProduct>

response.data.payload
      ↓
IProduct
```

---

# 7. ولو الـ payload Array؟

دي مهمة جدًا.

لو الـ API بيرجع:

```json id="x9w3km"
{
  "status": true,
  "code": 200,
  "payload": [
    {
      "id": 1,
      "name": "Ahmed"
    },
    {
      "id": 2,
      "name": "Mohamed"
    }
  ]
}
```

نستخدم:

```ts id="k7p2vd"
const response = await axios.get<IApiResponse<IUser[]>>(
  "/users"
);
```

وهنا:

```text id="c4y8nh"
response.data
      ↓
IApiResponse<IUser[]>

response.data.payload
      ↓
IUser[]
```

---

# 8. ركز في مستويات الـ Types

عندنا:

```ts id="f0x3ha"
axios.get<IApiResponse<IUser[]>>(...)
```

اقرأها من الداخل للخارج:

```text id="9v2m4k"
IUser[]
   ↓
IApiResponse<IUser[]>
   ↓
AxiosResponse<IApiResponse<IUser[]>>
```

وبعد `await`:

```text id="7c5j2p"
AxiosResponse<IApiResponse<IUser[]>>
```

والبيانات:

```text id="2n8x6q"
response.data
   ↓
IApiResponse<IUser[]>
```

والـ payload:

```text id="m7q4az"
response.data.payload
   ↓
IUser[]
```

دي نقطة لازم تثبت جدًا، لأنها هتظهر معاك كثير في المشاريع الحقيقية.

---

# 9. اربطها بالكود اللي إنت كنت بتستخدمه قبل كده

إنت كنت عندك Type قريب من:

```ts id="x4q9bw"
interface ISuccessResponse<T> {
  payload: T;
  status: true;
  code: number;
}

interface IErrorResponse<R> {
  errors: R[];
  status: false;
  message: string;
  code: number;
}
```

وده في الحقيقة **مقدمة مباشرة للموضوع اللي إحنا بندرسه الآن**.

لكن فيه خطوة مهمة ناقصة عندنا، وهي:

```text id="v6k1zs"
Success Response
       VS
Error Response
```

وإزاي TypeScript يفرق بينهم بناءً على:

```ts id="q3m8af"
status: true
```

مقابل:

```ts id="x7p2kc"
status: false
```

وده تحديدًا هو موضوع **Discriminated Unions**، لكن مش هنقفز له دلوقتي.

---

## 🎯 تدريب

عندك API بيرجع:

```json id="r2c8vm"
{
  "status": true,
  "code": 200,
  "payload": {
    "id": 15,
    "title": "Laptop",
    "price": 45000
  }
}
```

وعندك:

```ts id="k3m7qa"
interface IProduct {
  id: number;
  title: string;
  price: number;
}
```

اكتب:

1. `IApiResponse<T>`
2. `axios.get` بالـ Generic الصحيح.

**الخطوة التالية:** اكتب `IApiResponse<T>` و`axios.get<...>` بنفسك.
</details>
API Response Types  
ChatGPT
ابدأ بـ **API Response Types**، والهدف هنا تفرق بين **الـ Resource نفسه** مثل `IUser` وبين **الـ Wrapper اللي الـ API بيغلف جواه البيانات** مثل `payload` و`status` و`code`. ⏱️ 15 دقيقة

## 📍 Roadmap

```text id="4n2r8m"
SECTION 0: Architecture Foundation       ⬜
SECTION 1: Axios Fundamentals             ✅

SECTION 2: TypeScript + Axios             🟢
├── AxiosResponse                         ✅
├── Generics مع Axios                     ✅
├── Response Types                        ✅
├── API Response Types                    🟢 ← هنا
├── Discriminated Unions                  ⬜
├── unknown vs any                        ⬜
└── Type-safe API Layer                   ⬜

SECTION 3: Error Handling                 ⬜
SECTION 4: Interceptors                   ⬜
...
```

# 1. المشكلة اللي بنحلها

لحد دلوقتي كنا بنفترض إن الـ API بيرجع User مباشرة:

```json id="7b0q4d"
{
  "id": 10,
  "name": "Ahmed",
  "email": "ahmed@example.com"
}
```

فكتبنا:

```ts id="d3j4eq"
interface IUser {
  id: number;
  name: string;
  email: string;
}

const response = await axios.get<IUser>("/users/10");
```

وساعتها:

```text id="8k8qpk"
response.data → IUser
```

لكن APIs الحقيقية كثيرًا ما تستخدم **Response Wrapper**.

---

# 2. مثال على API Response Wrapper

ممكن الـ API يرجع:

```json id="f7n4wq"
{
  "status": true,
  "code": 200,
  "payload": {
    "id": 10,
    "name": "Ahmed",
    "email": "ahmed@example.com"
  }
}
```

لاحظ إن `IUser` لم يعد هو الـ Response كله.

الـ Response عندنا أصبح:

```text id="j9x1r5"
API Response
│
├── status
├── code
└── payload
      ↓
    IUser
```

---

# 3. نمثل ده في TypeScript

أولًا:

```ts id="e7g5p2"
interface IUser {
  id: number;
  name: string;
  email: string;
}
```

ثم نعمل Type للـ Wrapper:

```ts id="9s4b7m"
interface IApiResponse<T> {
  status: boolean;
  code: number;
  payload: T;
}
```

ركز في:

```ts id="f8w2k1"
<T>
```

هنا استخدمنا Generic مرة أخرى.

معناه:

> الـ `payload` ممكن يكون أي Type، وأنا هحدد نوعه وقت الاستخدام.

---

# 4. استخدامه مع `IUser`

نكتب:

```ts id="8h9xk3"
const response = await axios.get<IApiResponse<IUser>>(
  "/users/10"
);
```

لاحظ إن عندنا **Generic داخل Generic**:

```text id="2q5r8t"
IApiResponse<IUser>
           ↑
        T = IUser
```

إذن TypeScript يفهم:

```text id="q8w4n2"
response
   ↓
AxiosResponse<IApiResponse<IUser>>
```

ثم:

```text id="m2k7sa"
response.data
   ↓
IApiResponse<IUser>
```

ثم:

```text id="w4f8qe"
response.data.payload
   ↓
IUser
```

---

# 5. ليه عملنا `IApiResponse<T>`؟

عشان نفس الـ API Wrapper ممكن يستخدم مع بيانات مختلفة.

مثلاً User:

```ts id="n6k3v1"
IApiResponse<IUser>
```

Product:

```ts id="e1z5pa"
IApiResponse<IProduct>
```

Order:

```ts id="d7x2qm"
IApiResponse<IOrder>
```

بدل ما نكتب 3 Interfaces مختلفة للـ Wrapper:

```ts id="4h9q2s"
interface IUserResponse {
  status: boolean;
  code: number;
  payload: IUser;
}

interface IProductResponse {
  status: boolean;
  code: number;
  payload: IProduct;
}

interface IOrderResponse {
  status: boolean;
  code: number;
  payload: IOrder;
}
```

نكتب واحدة فقط:

```ts id="8x3n7v"
interface IApiResponse<T> {
  status: boolean;
  code: number;
  payload: T;
}
```

وبعدين نحدد `T`.

---

# 6. مثال Product

لو الـ API رجع:

```json id="c8p4az"
{
  "status": true,
  "code": 200,
  "payload": {
    "id": 5,
    "title": "MacBook",
    "price": 50000
  }
}
```

نعمل:

```ts id="q2n8mw"
interface IProduct {
  id: number;
  title: string;
  price: number;
}
```

ثم:

```ts id="m5r7kx"
const response = await axios.get<IApiResponse<IProduct>>(
  "/products/5"
);
```

النتيجة:

```text id="f1s9qa"
response.data
      ↓
IApiResponse<IProduct>

response.data.payload
      ↓
IProduct
```

---

# 7. ولو الـ payload Array؟

دي مهمة جدًا.

لو الـ API بيرجع:

```json id="x9w3km"
{
  "status": true,
  "code": 200,
  "payload": [
    {
      "id": 1,
      "name": "Ahmed"
    },
    {
      "id": 2,
      "name": "Mohamed"
    }
  ]
}
```

نستخدم:

```ts id="k7p2vd"
const response = await axios.get<IApiResponse<IUser[]>>(
  "/users"
);
```

وهنا:

```text id="c4y8nh"
response.data
      ↓
IApiResponse<IUser[]>

response.data.payload
      ↓
IUser[]
```

---

# 8. ركز في مستويات الـ Types

عندنا:

```ts id="f0x3ha"
axios.get<IApiResponse<IUser[]>>(...)
```

اقرأها من الداخل للخارج:

```text id="9v2m4k"
IUser[]
   ↓
IApiResponse<IUser[]>
   ↓
AxiosResponse<IApiResponse<IUser[]>>
```

وبعد `await`:

```text id="7c5j2p"
AxiosResponse<IApiResponse<IUser[]>>
```

والبيانات:

```text id="2n8x6q"
response.data
   ↓
IApiResponse<IUser[]>
```

والـ payload:

```text id="m7q4az"
response.data.payload
   ↓
IUser[]
```

دي نقطة لازم تثبت جدًا، لأنها هتظهر معاك كثير في المشاريع الحقيقية.

---

# 9. اربطها بالكود اللي إنت كنت بتستخدمه قبل كده

إنت كنت عندك Type قريب من:

```ts id="x4q9bw"
interface ISuccessResponse<T> {
  payload: T;
  status: true;
  code: number;
}

interface IErrorResponse<R> {
  errors: R[];
  status: false;
  message: string;
  code: number;
}
```

وده في الحقيقة **مقدمة مباشرة للموضوع اللي إحنا بندرسه الآن**.

لكن فيه خطوة مهمة ناقصة عندنا، وهي:

```text id="v6k1zs"
Success Response
       VS
Error Response
```

وإزاي TypeScript يفرق بينهم بناءً على:

```ts id="q3m8af"
status: true
```

مقابل:

```ts id="x7p2kc"
status: false
```

وده تحديدًا هو موضوع **Discriminated Unions**، لكن مش هنقفز له دلوقتي.

---

## 🎯 تدريب

عندك API بيرجع:

```json id="r2c8vm"
{
  "status": true,
  "code": 200,
  "payload": {
    "id": 15,
    "title": "Laptop",
    "price": 45000
  }
}
```

وعندك:

```ts id="k3m7qa"
interface IProduct {
  id: number;
  title: string;
  price: number;
}
```

اكتب:

1. `IApiResponse<T>`
2. `axios.get` بالـ Generic الصحيح.

**الخطوة التالية:** اكتب `IApiResponse<T>` و`axios.get<...>` بنفسك.
<details>
<summary>interface IApiResponse&lt;T&gt;{</summary>
user
interface IApiResponse<T>{
status: boolean,
  code: 200,  
payload:T,
}
const response await axios.get<IApiResponse<IUser>>("/user/");
ChatGPT
صحح خطأين فقط، لأن **فكرة الـ Generic والـ Wrapper عندك صحيحة**. ⏱️ 3 دقائق

أنت كتبت:

```ts
interface IApiResponse<T> {
  status: boolean;
  code: 200;
  payload: T;
}
```

هنا:

```ts
code: 200;
```

معناها إن `code` **لازم تكون 200 فقط**.

لكن لو إحنا بنعمل Generic API Response عام، الأفضل:

```ts
code: number;
```

فتصبح:

```ts
interface IApiResponse<T> {
  status: boolean;
  code: number;
  payload: T;
}
```

أما السطر الثاني:

```ts
const response await axios.get<IApiResponse<IUser>>("/user/");
```

ناقص `=` بعد `response`:

```ts
const response = await axios.get<IApiResponse<IUser>>("/user/");
```

### إذن الكود الصحيح

```ts
interface IApiResponse<T> {
  status: boolean;
  code: number;
  payload: T;
}

const response = await axios.get<IApiResponse<IUser>>(
  "/user/"
);
```

والـ Type هنا بيتفك كده:

```text
<IUser>
   ↓
T = IUser

IApiResponse<IUser>
   ↓
payload → IUser

AxiosResponse<IApiResponse<IUser>>
   ↓
response.data → IApiResponse<IUser>

response.data.payload
   ↓
IUser
```

### ⭐ تقييمك: 9.5/10

أهم حاجة إنك **وصلت لوحدك لفكرة `IApiResponse<T>` واستخدمت `IApiResponse<IUser>` بشكل صحيح**.

وملاحظة مهمة: `code: 200` مش خطأ Syntax، هو **Literal Type** صحيح في TypeScript، لكنه مش مناسب للـ API Response العام اللي ممكن يحتوي على `201`, `400`, `404`, `500` وغيرها.

**الخطوة التالية:** اكتب لي `IApiResponse<IUser[]>` في `axios.get` لو الـ `payload` عبارة عن Array من Users.
</details>
interface IApiResponse<T>{
status: boolean,
  code: 200,  
payload:T,
}
const response await axios.get<IApiResponse<IUser>>("/user/");
ChatGPT
صحح خطأين فقط، لأن **فكرة الـ Generic والـ Wrapper عندك صحيحة**. ⏱️ 3 دقائق

أنت كتبت:

```ts
interface IApiResponse<T> {
  status: boolean;
  code: 200;
  payload: T;
}
```

هنا:

```ts
code: 200;
```

معناها إن `code` **لازم تكون 200 فقط**.

لكن لو إحنا بنعمل Generic API Response عام، الأفضل:

```ts
code: number;
```

فتصبح:

```ts
interface IApiResponse<T> {
  status: boolean;
  code: number;
  payload: T;
}
```

أما السطر الثاني:

```ts
const response await axios.get<IApiResponse<IUser>>("/user/");
```

ناقص `=` بعد `response`:

```ts
const response = await axios.get<IApiResponse<IUser>>("/user/");
```

### إذن الكود الصحيح

```ts
interface IApiResponse<T> {
  status: boolean;
  code: number;
  payload: T;
}

const response = await axios.get<IApiResponse<IUser>>(
  "/user/"
);
```

والـ Type هنا بيتفك كده:

```text
<IUser>
   ↓
T = IUser

IApiResponse<IUser>
   ↓
payload → IUser

AxiosResponse<IApiResponse<IUser>>
   ↓
response.data → IApiResponse<IUser>

response.data.payload
   ↓
IUser
```

### ⭐ تقييمك: 9.5/10

أهم حاجة إنك **وصلت لوحدك لفكرة `IApiResponse<T>` واستخدمت `IApiResponse<IUser>` بشكل صحيح**.

وملاحظة مهمة: `code: 200` مش خطأ Syntax، هو **Literal Type** صحيح في TypeScript، لكنه مش مناسب للـ API Response العام اللي ممكن يحتوي على `201`, `400`, `404`, `500` وغيرها.

**الخطوة التالية:** اكتب لي `IApiResponse<IUser[]>` في `axios.get` لو الـ `payload` عبارة عن Array من Users.
<details>
<summary>interface IApiResponse&lt;T&gt; {</summary>
user
interface IApiResponse<T> {
  status: boolean;
  code: number;
  payload: T;
}

const response = await axios.get<IApiResponse<IUser[]>>(
  "/user/"
);
ChatGPT
إجابتك **10/10**. الكود صحيح، والـ Generic هنا مستخدم بالشكل المطلوب. ⏱️ دقيقتان

```ts id="d7s9mk"
interface IApiResponse<T> {
  status: boolean;
  code: number;
  payload: T;
}

const response = await axios.get<IApiResponse<IUser[]>>(
  "/user/"
);
```

خلينا نفك السطر ده من الداخل للخارج:

```ts id="u7w8cs"
IApiResponse<IUser[]>
```

أولًا:

```text id="7g5q1d"
IUser[]
   ↓
Array من Users
```

ثم:

```text id="g4j2wp"
IApiResponse<IUser[]>
        ↓
payload → IUser[]
```

والـ `Axios` يضيف الـ `AxiosResponse` حولهم:

```text id="a2f6yr"
response
   ↓
AxiosResponse<IApiResponse<IUser[]>>
   │
   └── data
        ↓
      IApiResponse<IUser[]>
        │
        └── payload
             ↓
           IUser[]
```

يعني تقدر تعمل:

```ts id="1k5y8q"
response.data.status;
```

نوعها:

```text
boolean
```

وتقدر تعمل:

```ts id="3z8v6m"
response.data.code;
```

نوعها:

```text
number
```

وتقدر تعمل:

```ts id="c7n4xa"
response.data.payload;
```

نوعها:

```text
IUser[]
```

وكمان:

```ts id="p8w2kj"
response.data.payload[0].name;
```

نوع `name` هنا:

```text
string
```

### 📍 Progress

```text id="0r2k8f"
SECTION 2: TypeScript + Axios

├── AxiosResponse                         ✅
├── Generics مع Axios                     ✅
├── Response Types                        ✅
├── API Response Types                    ✅
├── Discriminated Unions                  🟢 ← التالي
├── unknown vs any                        ⬜
└── Type-safe API Layer                   ⬜
```

**الخطوة التالية:** نبدأ `Discriminated Unions` باستخدام `status: true` و`status: false`، وهنربطها مباشرة بالـ `Success Response` و`Error Response` اللي اشتغلنا عليهم قبل كده.
</details>
interface IApiResponse<T> {
  status: boolean;
  code: number;
  payload: T;
}

const response = await axios.get<IApiResponse<IUser[]>>(
  "/user/"
);
ChatGPT
إجابتك **10/10**. الكود صحيح، والـ Generic هنا مستخدم بالشكل المطلوب. ⏱️ دقيقتان

```ts id="d7s9mk"
interface IApiResponse<T> {
  status: boolean;
  code: number;
  payload: T;
}

const response = await axios.get<IApiResponse<IUser[]>>(
  "/user/"
);
```

خلينا نفك السطر ده من الداخل للخارج:

```ts id="u7w8cs"
IApiResponse<IUser[]>
```

أولًا:

```text id="7g5q1d"
IUser[]
   ↓
Array من Users
```

ثم:

```text id="g4j2wp"
IApiResponse<IUser[]>
        ↓
payload → IUser[]
```

والـ `Axios` يضيف الـ `AxiosResponse` حولهم:

```text id="a2f6yr"
response
   ↓
AxiosResponse<IApiResponse<IUser[]>>
   │
   └── data
        ↓
      IApiResponse<IUser[]>
        │
        └── payload
             ↓
           IUser[]
```

يعني تقدر تعمل:

```ts id="1k5y8q"
response.data.status;
```

نوعها:

```text
boolean
```

وتقدر تعمل:

```ts id="3z8v6m"
response.data.code;
```

نوعها:

```text
number
```

وتقدر تعمل:

```ts id="c7n4xa"
response.data.payload;
```

نوعها:

```text
IUser[]
```

وكمان:

```ts id="p8w2kj"
response.data.payload[0].name;
```

نوع `name` هنا:

```text
string
```

### 📍 Progress

```text id="0r2k8f"
SECTION 2: TypeScript + Axios

├── AxiosResponse                         ✅
├── Generics مع Axios                     ✅
├── Response Types                        ✅
├── API Response Types                    ✅
├── Discriminated Unions                  🟢 ← التالي
├── unknown vs any                        ⬜
└── Type-safe API Layer                   ⬜
```

**الخطوة التالية:** نبدأ `Discriminated Unions` باستخدام `status: true` و`status: false`، وهنربطها مباشرة بالـ `Success Response` و`Error Response` اللي اشتغلنا عليهم قبل كده.
<details>
<summary>Discriminated Unions</summary>
user
Discriminated Unions  
ChatGPT
ابدأ بفهم **Discriminated Unions** من خلال مشكلة بسيطة: عندنا `Success Response` و`Error Response`، وعايزين TypeScript يعرف أي واحد فيهم موجود من قيمة `status`. ⏱️ 15 دقيقة

## 📍 Roadmap

```text id="7x3kq1"
SECTION 0: Architecture Foundation       ⬜
SECTION 1: Axios Fundamentals             ✅

SECTION 2: TypeScript + Axios             🟢
├── AxiosResponse                         ✅
├── Generics مع Axios                     ✅
├── Response Types                        ✅
├── API Response Types                    ✅
├── Discriminated Unions                  🟢 ← هنا
├── unknown vs any                        ⬜
└── Type-safe API Layer                   ⬜

SECTION 3: Error Handling                 ⬜
SECTION 4: Interceptors                   ⬜
...
```
</details>
Discriminated Unions  
ChatGPT
ابدأ بفهم **Discriminated Unions** من خلال مشكلة بسيطة: عندنا `Success Response` و`Error Response`، وعايزين TypeScript يعرف أي واحد فيهم موجود من قيمة `status`. ⏱️ 15 دقيقة

## 📍 Roadmap

```text id="7x3kq1"
SECTION 0: Architecture Foundation       ⬜
SECTION 1: Axios Fundamentals             ✅

SECTION 2: TypeScript + Axios             🟢
├── AxiosResponse                         ✅
├── Generics مع Axios                     ✅
├── Response Types                        ✅
├── API Response Types                    ✅
├── Discriminated Unions                  🟢 ← هنا
├── unknown vs any                        ⬜
└── Type-safe API Layer                   ⬜

SECTION 3: Error Handling                 ⬜
SECTION 4: Interceptors                   ⬜
...
```