# Lesson 2: `axios.post()`

أنت عرفت إن:

```ts
axios.get("/users")
```

معناها:

> ابعت GET Request إلى `/users`.

نفس الفكرة:

```ts
axios.post("/users")
```

معناها:

> ابعت POST Request إلى `/users`.

لكن `POST` غالبًا محتاج **Body**.

---

## 1. أبسط POST

مثلاً عايزين نعمل User جديد:

```ts
const response = await axios.post("/users");
```

ده بيبعت:

```http
POST /users
```

لكن مفيش بيانات مع الطلب.

---

## 2. إرسال Body

عندنا User:

```ts
const user = {
  name: "Ahmed",
  email: "ahmed@example.com",
};
```

نقدر نبعت البيانات:

```ts
const response = await axios.post("/users", user);
```

لاحظ إن `post()` هنا عندها:

```text
axios.post(
  URL,
  DATA
)
```

يعني:

```text
الـ argument الأول
↓
URL

الـ argument الثاني
↓
Request Body
```

---

## 3. مثال مباشر

```ts
const response = await axios.post("/users", {
  name: "Ahmed",
  email: "ahmed@example.com",
});
```

Axios هتبعت تقريبًا:

```http
POST /users
Content-Type: application/json

{
  "name": "Ahmed",
  "email": "ahmed@example.com"
}
```

لاحظ إننا لم نكتب:

```ts
body:
```

في Axios.

نحن ببساطة نضع الـ data كـ **argument ثاني**.

---

# 4. طيب فين الـ TypeScript؟

هنا عندنا نوعان مختلفان محتاجين نفصل بينهم.

مثلاً:

```ts
interface User {
  id: number;
  name: string;
  email: string;
}
```

والبيانات التي نرسلها لإنشاء User:

```ts
interface CreateUserRequest {
  name: string;
  email: string;
}
```

لاحظ:

```text
User
↓
البيانات التي يرجعها السيرفر

CreateUserRequest
↓
البيانات التي نرسلها للسيرفر
```

فنكتب:

```ts
const userData: CreateUserRequest = {
  name: "Ahmed",
  email: "ahmed@example.com",
};

const response = await axios.post<User>(
  "/users",
  userData
);
```

هنا:

```ts
<User>
```

تعني:

> أنا متوقع إن `response.data` يكون `User`.

أما:

```ts
userData
```

فهو:

> الـ Body الذي سأرسله للسيرفر.

ودي نقطة مهمة جدًا:

```text
axios.post<User>()
              ↑
       Response Type

axios.post("/users", userData)
                     ↑
                    Body
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

interface CreateProductRequest {
  name: string;
  price: number;
}
```

وعايز تعمل Product جديد:

```text
name = "iPhone"
price = 50000
```

اكتب:

```ts
const response = ...
```

بحيث:

- الـ endpoint يكون `/products`
- الـ Body يحتوي `name` و `price`
- الـ Response المتوقع يكون `Product`

**الخطوة التالية: اكتب سطر `axios.post()` بنفسك.**
<details>
<summary>&lt;CreateUserRequest&gt;</summary>
user
<CreateUserRequest>
ليه مش كده وايه الفرق
ChatGPT
**الهدف: نفهم الفرق بين `<Product>` و `<CreateProductRequest>` في `axios.post()`، لأن دي نقطة TypeScript مهمة جدًا.** الوقت: **5 دقائق**.

ببساطة، في:

```ts
axios.post<Product>("/products", productData)
```

الـ `<Product>` هنا معناها **نوع الـ Response المتوقع**، وليس نوع الـ Body.

يعني:

```text
axios.post<Product>(
       ↑
       Response
)
```

أما:

```ts
productData
```

فهو **الـ Body الذي سترسله**.

---

### طيب `CreateProductRequest` بيستخدم فين؟

هنا:

```ts
const productData: CreateProductRequest = {
  name: "iPhone",
  price: 50000,
};

const response = await axios.post<Product>(
  "/products",
  productData
);
```

عندنا نوعين مختلفين:

```text
CreateProductRequest
        ↓
البيانات التي أرسلها

Product
        ↓
البيانات التي أتوقعها من السيرفر
```

مثلاً السيرفر يستقبل:

```json
{
  "name": "iPhone",
  "price": 50000
}
```

لكن بعد الإنشاء ممكن يرجع:

```json
{
  "id": 10,
  "name": "iPhone",
  "price": 50000
}
```

لاحظ إن الـ Response فيه:

```text
id
```

لكن الـ Request مفيهوش `id`.

لذلك منطقي جدًا يكون عندنا:

```ts
interface CreateProductRequest {
  name: string;
  price: number;
}
```

و:

```ts
interface Product {
  id: number;
  name: string;
  price: number;
}
```

---

### إذن سؤالك مهم جدًا

لو كتبت:

```ts
axios.post<CreateProductRequest>("/products", productData)
```

فأنت **بتقول لـ Axios**:

> أنا متوقع إن الـ Response يكون `CreateProductRequest`.

مش:

> أنا ببعت `CreateProductRequest`.

وده سبب إن `<CreateProductRequest>` مش هو المكان الصحيح لتحديد Body.

### احفظها كده الآن:

```ts
axios.post<ResponseType>(
  url,
  requestBody
)
```

مثال:

```ts
axios.post<Product>(
  "/products",
  productData
)
```

```text
<Product>
↓
Response

productData
↓
Request Body
```

**الخطوة التالية: اكتب لي بنفسك `axios.post()` لإنشاء Product، وأنا أصححه فقط بدون إضافة Concept جديد.**
</details>
<CreateUserRequest>
ليه مش كده وايه الفرق
ChatGPT
**الهدف: نفهم الفرق بين `<Product>` و `<CreateProductRequest>` في `axios.post()`، لأن دي نقطة TypeScript مهمة جدًا.** الوقت: **5 دقائق**.

ببساطة، في:

```ts
axios.post<Product>("/products", productData)
```

الـ `<Product>` هنا معناها **نوع الـ Response المتوقع**، وليس نوع الـ Body.

يعني:

```text
axios.post<Product>(
       ↑
       Response
)
```

أما:

```ts
productData
```

فهو **الـ Body الذي سترسله**.

---

### طيب `CreateProductRequest` بيستخدم فين؟

هنا:

```ts
const productData: CreateProductRequest = {
  name: "iPhone",
  price: 50000,
};

const response = await axios.post<Product>(
  "/products",
  productData
);
```

عندنا نوعين مختلفين:

```text
CreateProductRequest
        ↓
البيانات التي أرسلها

Product
        ↓
البيانات التي أتوقعها من السيرفر
```

مثلاً السيرفر يستقبل:

```json
{
  "name": "iPhone",
  "price": 50000
}
```

لكن بعد الإنشاء ممكن يرجع:

```json
{
  "id": 10,
  "name": "iPhone",
  "price": 50000
}
```

لاحظ إن الـ Response فيه:

```text
id
```

لكن الـ Request مفيهوش `id`.

لذلك منطقي جدًا يكون عندنا:

```ts
interface CreateProductRequest {
  name: string;
  price: number;
}
```

و:

```ts
interface Product {
  id: number;
  name: string;
  price: number;
}
```

---

### إذن سؤالك مهم جدًا

لو كتبت:

```ts
axios.post<CreateProductRequest>("/products", productData)
```

فأنت **بتقول لـ Axios**:

> أنا متوقع إن الـ Response يكون `CreateProductRequest`.

مش:

> أنا ببعت `CreateProductRequest`.

وده سبب إن `<CreateProductRequest>` مش هو المكان الصحيح لتحديد Body.

### احفظها كده الآن:

```ts
axios.post<ResponseType>(
  url,
  requestBody
)
```

مثال:

```ts
axios.post<Product>(
  "/products",
  productData
)
```

```text
<Product>
↓
Response

productData
↓
Request Body
```

**الخطوة التالية: اكتب لي بنفسك `axios.post()` لإنشاء Product، وأنا أصححه فقط بدون إضافة Concept جديد.**
<details>
<summary>const userData: &lt;CreateUserRequest&gt; = {</summary>
user
const userData: <CreateUserRequest> = {
  name: "Ahmed",
  email: "ahmed@example.com",
};

const response = await axios.post<User>(
  "/users",
  userData
);

انا بكلمك علي <> الاقواس دي ليه معملنهاش كده 
ChatGPT
**الهدف: نفهم تحديدًا لماذا `CreateUserRequest` لا نكتب حوله `< >` عند تعريف المتغير، لأنك بتسأل عن الـ syntax نفسه.** الوقت: **5 دقائق**.

أنت تقصد لماذا لا نكتب:

```ts
const userData: <CreateUserRequest> = {
```

بدل:

```ts
const userData: CreateUserRequest = {
```

الإجابة: **لأن `< >` في TypeScript لها استخدام مختلف عن `:`.**

### 1. عند تعريف Variable

هنا:

```ts
const userData: CreateUserRequest = {
  name: "Ahmed",
  email: "ahmed@example.com",
};
```

الـ `:` معناها:

> نوع المتغير هو `CreateUserRequest`.

لذلك نكتب:

```text
variable : Type
```

مثل:

```ts
const name: string = "Ahmed";
const age: number = 38;
const user: User = {...};
```

**مفيش `< >` هنا.**

---

### 2. عند استخدام Generic

لكن هنا:

```ts
axios.post<User>(
  "/users",
  userData
);
```

`<User>` معناها:

> مرر `User` كـ **Type Argument** إلى الـ Generic Function.

يعني:

```text
axios.post<User>
           ↑
      Generic Type
```

---

### الفرق بصريًا

```ts
const userData: CreateUserRequest
              ↑
          Type Annotation
```

بينما:

```ts
axios.post<User>()
            ↑
      Generic Argument
```

الاتنين بيتعاملوا مع **Types**، لكن الـ syntax مختلف لأنهم بيعملوا وظيفتين مختلفتين.

---

### ❌ إذن ده غلط

```ts
const userData: <CreateUserRequest> = {
```

لأن TypeScript بعد `:` مستني **Type**، مش Generic Argument.

### ✅ الصحيح

```ts
const userData: CreateUserRequest = {
```

**الخطوة التالية: اكتب لي `const userData` فقط، وأنا أسألك سؤال صغير للتأكد إن الفرق بين `:` و`<>` ثبت.**
</details>
const userData: <CreateUserRequest> = {
  name: "Ahmed",
  email: "ahmed@example.com",
};

const response = await axios.post<User>(
  "/users",
  userData
);

انا بكلمك علي <> الاقواس دي ليه معملنهاش كده 
ChatGPT
**الهدف: نفهم تحديدًا لماذا `CreateUserRequest` لا نكتب حوله `< >` عند تعريف المتغير، لأنك بتسأل عن الـ syntax نفسه.** الوقت: **5 دقائق**.

أنت تقصد لماذا لا نكتب:

```ts
const userData: <CreateUserRequest> = {
```

بدل:

```ts
const userData: CreateUserRequest = {
```

الإجابة: **لأن `< >` في TypeScript لها استخدام مختلف عن `:`.**

### 1. عند تعريف Variable

هنا:

```ts
const userData: CreateUserRequest = {
  name: "Ahmed",
  email: "ahmed@example.com",
};
```

الـ `:` معناها:

> نوع المتغير هو `CreateUserRequest`.

لذلك نكتب:

```text
variable : Type
```

مثل:

```ts
const name: string = "Ahmed";
const age: number = 38;
const user: User = {...};
```

**مفيش `< >` هنا.**

---

### 2. عند استخدام Generic

لكن هنا:

```ts
axios.post<User>(
  "/users",
  userData
);
```

`<User>` معناها:

> مرر `User` كـ **Type Argument** إلى الـ Generic Function.

يعني:

```text
axios.post<User>
           ↑
      Generic Type
```

---

### الفرق بصريًا

```ts
const userData: CreateUserRequest
              ↑
          Type Annotation
```

بينما:

```ts
axios.post<User>()
            ↑
      Generic Argument
```

الاتنين بيتعاملوا مع **Types**، لكن الـ syntax مختلف لأنهم بيعملوا وظيفتين مختلفتين.

---

### ❌ إذن ده غلط

```ts
const userData: <CreateUserRequest> = {
```

لأن TypeScript بعد `:` مستني **Type**، مش Generic Argument.

### ✅ الصحيح

```ts
const userData: CreateUserRequest = {
```

**الخطوة التالية: اكتب لي `const userData` فقط، وأنا أسألك سؤال صغير للتأكد إن الفرق بين `:` و`<>` ثبت.**
<details>
<summary>تدريب</summary>
user
تدريب

```
interface Product {
  id: number;
  name: string;
  price: number;
}

interface CreateProductRequest{
  name: string;
  price: number;
}

const product: CreateProductRequest{}
```

const response =  await axios.post\<Product[]>("/products", )
ChatGPT
**الهدف: تصحيح تدريب `axios.post<Product>()` فقط، بدون إضافة أي Concept جديد.** أنت قريب جدًا، لكن عندك خطأين في كتابة `product` وناقص الـ Body في `post`. الوقت: **3 دقائق**.

### 1. تعريف الـ product

أنت كتبت:

```ts
const product: CreateProductRequest{}
```

الصحيح:

```ts
const product: CreateProductRequest = {
  name: "iPhone",
  price: 50000,
};
```

لأن:

```text
const product: CreateProductRequest
             ↑
             نوع المتغير
```

ثم:

```text
= {
   ...
}
```

هي القيمة.

---

### 2. `axios.post`

أنت كتبت:

```ts
const response = await axios.post<Product[]>("/products", )
```

هنا محتاج نحدد **نوع الـ Response المتوقع**.

لو السيرفر بعد إنشاء Product بيرجع **Product واحد**:

```ts
const response = await axios.post<Product>(
  "/products",
  product
);
```

لاحظ:

```text
axios.post<Product>
           ↑
      Response Type
```

و:

```text
product
       ↑
   Request Body
```

إذن الشكل العام:

```ts
axios.post<ResponseType>(
  url,
  requestBody
);
```

### 🟢 تدريبك بعد التصحيح

```ts
interface Product {
  id: number;
  name: string;
  price: number;
}

interface CreateProductRequest {
  name: string;
  price: number;
}

const product: CreateProductRequest = {
  name: "iPhone",
  price: 50000,
};

const response = await axios.post<Product>(
  "/products",
  product
);
```

**ملاحظة واحدة فقط:** أنت كتبت `Product[]`، لكن في سيناريو إنشاء Product، نحن افترضنا أن السيرفر يرجع **Product واحد**، لذلك استخدمنا `Product`. لو الـ API فعلًا يرجع Array، وقتها `Product[]` تكون صحيحة.

**الخطوة التالية: نكمل `axios.post()` ونفهم الـ 3 أجزاء الأساسية فيه: URL، Body، وResponse Type.**
</details>
تدريب

```
interface Product {
  id: number;
  name: string;
  price: number;
}

interface CreateProductRequest{
  name: string;
  price: number;
}

const product: CreateProductRequest{}
```

const response =  await axios.post\<Product[]>("/products", )
ChatGPT
**الهدف: تصحيح تدريب `axios.post<Product>()` فقط، بدون إضافة أي Concept جديد.** أنت قريب جدًا، لكن عندك خطأين في كتابة `product` وناقص الـ Body في `post`. الوقت: **3 دقائق**.

### 1. تعريف الـ product

أنت كتبت:

```ts
const product: CreateProductRequest{}
```

الصحيح:

```ts
const product: CreateProductRequest = {
  name: "iPhone",
  price: 50000,
};
```

لأن:

```text
const product: CreateProductRequest
             ↑
             نوع المتغير
```

ثم:

```text
= {
   ...
}
```

هي القيمة.

---

### 2. `axios.post`

أنت كتبت:

```ts
const response = await axios.post<Product[]>("/products", )
```

هنا محتاج نحدد **نوع الـ Response المتوقع**.

لو السيرفر بعد إنشاء Product بيرجع **Product واحد**:

```ts
const response = await axios.post<Product>(
  "/products",
  product
);
```

لاحظ:

```text
axios.post<Product>
           ↑
      Response Type
```

و:

```text
product
       ↑
   Request Body
```

إذن الشكل العام:

```ts
axios.post<ResponseType>(
  url,
  requestBody
);
```

### 🟢 تدريبك بعد التصحيح

```ts
interface Product {
  id: number;
  name: string;
  price: number;
}

interface CreateProductRequest {
  name: string;
  price: number;
}

const product: CreateProductRequest = {
  name: "iPhone",
  price: 50000,
};

const response = await axios.post<Product>(
  "/products",
  product
);
```

**ملاحظة واحدة فقط:** أنت كتبت `Product[]`، لكن في سيناريو إنشاء Product، نحن افترضنا أن السيرفر يرجع **Product واحد**، لذلك استخدمنا `Product`. لو الـ API فعلًا يرجع Array، وقتها `Product[]` تكون صحيحة.

**الخطوة التالية: نكمل `axios.post()` ونفهم الـ 3 أجزاء الأساسية فيه: URL، Body، وResponse Type.**
<details>
<summary>كمل</summary>
user
كمل
ChatGPT
**الهدف: نكمل `axios.post()` ونثبت الفرق بين الـ URL والـ Body والـ Response Type فقط.** الوقت: **10 دقائق**.

# Lesson 2: `axios.post()`، الجزء 2

بص على السطر ده:

```ts
const response = await axios.post<Product>(
  "/products",
  product
);
```

الـ `post()` هنا عندها 3 حاجات مهمة:

```text
axios.post<Product>(
        ↑
        1. Response Type

"/products"
        ↑
        2. URL

product
        ↑
        3. Body
)
```

---

## 1. Response Type

```ts
axios.post<Product>()
```

`<Product>` معناها:

> أنا متوقع أن البيانات التي سيرجعها السيرفر تكون `Product`.

وبالتالي:

```ts
response.data
```

نوعها:

```ts
Product
```

---

## 2. URL

```ts
axios.post<Product>(
  "/products"
)
```

ده نفس الـ URL اللي اتعلمناه في GET.

يعني:

```text
POST /products
```

---

## 3. Body

هنا:

```ts
const product: CreateProductRequest = {
  name: "iPhone",
  price: 50000,
};
```

وبعدين:

```ts
axios.post<Product>(
  "/products",
  product
);
```

يبقى `product` هو الـ **Request Body**.

كأنك بتبعت:

```json
{
  "name": "iPhone",
  "price": 50000
}
```

---

# أهم فرق في الدرس

عندنا نوعين:

```ts
Product
```

و:

```ts
CreateProductRequest
```

### `CreateProductRequest`

بيصف **البيانات اللي هتبعتها**:

```ts
{
  name: string;
  price: number;
}
```

### `Product`

بيصف **البيانات اللي متوقع تستلمها**:

```ts
{
  id: number;
  name: string;
  price: number;
}
```

لذلك:

```ts
const product: CreateProductRequest = {
  name: "iPhone",
  price: 50000,
};
```

ثم:

```ts
const response = await axios.post<Product>(
  "/products",
  product
);
```

الصورة:

```text
CreateProductRequest
        ↓
     Request
        ↓
      POST
        ↓
     Server
        ↓
    Response
        ↓
      Product
```

دي نقطة مهمة جدًا، ثبتها كويس لأنها هتظهر معانا كثير جدًا في الـ API Architecture.

---

# 🧪 تدريب

عندك:

```ts
interface User {
  id: number;
  name: string;
  email: string;
}

interface CreateUserRequest {
  name: string;
  email: string;
}
```

وعايز تعمل User جديد:

```text
name = "Ahmed"
email = "ahmed@example.com"
```

اكتب الكود:

```ts
const userData: ...
```

ثم:

```ts
const response = ...
```

**الخطوة التالية: اكتب الكود كاملًا بنفسك، وأنا أصححه فقط.**
</details>
كمل
ChatGPT
**الهدف: نكمل `axios.post()` ونثبت الفرق بين الـ URL والـ Body والـ Response Type فقط.** الوقت: **10 دقائق**.

# Lesson 2: `axios.post()`، الجزء 2

بص على السطر ده:

```ts
const response = await axios.post<Product>(
  "/products",
  product
);
```

الـ `post()` هنا عندها 3 حاجات مهمة:

```text
axios.post<Product>(
        ↑
        1. Response Type

"/products"
        ↑
        2. URL

product
        ↑
        3. Body
)
```

---

## 1. Response Type

```ts
axios.post<Product>()
```

`<Product>` معناها:

> أنا متوقع أن البيانات التي سيرجعها السيرفر تكون `Product`.

وبالتالي:

```ts
response.data
```

نوعها:

```ts
Product
```

---

## 2. URL

```ts
axios.post<Product>(
  "/products"
)
```

ده نفس الـ URL اللي اتعلمناه في GET.

يعني:

```text
POST /products
```

---

## 3. Body

هنا:

```ts
const product: CreateProductRequest = {
  name: "iPhone",
  price: 50000,
};
```

وبعدين:

```ts
axios.post<Product>(
  "/products",
  product
);
```

يبقى `product` هو الـ **Request Body**.

كأنك بتبعت:

```json
{
  "name": "iPhone",
  "price": 50000
}
```

---

# أهم فرق في الدرس

عندنا نوعين:

```ts
Product
```

و:

```ts
CreateProductRequest
```

### `CreateProductRequest`

بيصف **البيانات اللي هتبعتها**:

```ts
{
  name: string;
  price: number;
}
```

### `Product`

بيصف **البيانات اللي متوقع تستلمها**:

```ts
{
  id: number;
  name: string;
  price: number;
}
```

لذلك:

```ts
const product: CreateProductRequest = {
  name: "iPhone",
  price: 50000,
};
```

ثم:

```ts
const response = await axios.post<Product>(
  "/products",
  product
);
```

الصورة:

```text
CreateProductRequest
        ↓
     Request
        ↓
      POST
        ↓
     Server
        ↓
    Response
        ↓
      Product
```

دي نقطة مهمة جدًا، ثبتها كويس لأنها هتظهر معانا كثير جدًا في الـ API Architecture.

---

# 🧪 تدريب

عندك:

```ts
interface User {
  id: number;
  name: string;
  email: string;
}

interface CreateUserRequest {
  name: string;
  email: string;
}
```

وعايز تعمل User جديد:

```text
name = "Ahmed"
email = "ahmed@example.com"
```

اكتب الكود:

```ts
const userData: ...
```

ثم:

```ts
const response = ...
```

**الخطوة التالية: اكتب الكود كاملًا بنفسك، وأنا أصححه فقط.**
<details>
<summary>const userData:  CreateUserRequest  {</summary>
user
const userData:  CreateUserRequest  {
name = "Ahmed"
email = "ahmed@example.com"
}
const response =   await axios.post<User>("/users", userData);
ChatGPT
**الهدف: تثبيت `axios.post()`، وإجابتك في الجزء الخاص بـ Axios صحيحة، والخطأ فقط في Syntax تعريف `userData`.** الوقت: **3 دقائق**.

### إجابتك

```ts
const userData: CreateUserRequest {
  name = "Ahmed"
  email = "ahmed@example.com"
}

const response = await axios.post<User>("/users", userData);
```

## التصحيح

الصحيح:

```ts
const userData: CreateUserRequest = {
  name: "Ahmed",
  email: "ahmed@example.com",
};

const response = await axios.post<User>("/users", userData);
```

### الخطأ الأول

كتبت:

```ts
const userData: CreateUserRequest {
```

لازم:

```ts
const userData: CreateUserRequest = {
```

لأننا بنقول:

```text
userData
   ↓
نوعه CreateUserRequest
   ↓
وقيمته = object
```

### الخطأ الثاني

كتبت:

```ts
name = "Ahmed"
```

داخل الـ object لازم:

```ts
name: "Ahmed"
```

نفس القاعدة:

```text
TypeScript type annotation
↓
:

Object property
↓
:
```

أما `=` فنستخدمها في **إسناد قيمة للمتغير**:

```ts
const x = 10;
```

---

## 🟢 التقييم

```text
فهم POST              ✅
فهم Request Body      ✅
فهم Response Type     ✅
استخدام Generic       ✅
TypeScript syntax     🟡
```

**التقييم: 9/10**

والأهم إنك فهمت الجزء الصعب:

```ts
axios.post<User>("/users", userData);
```

أنت الآن فاهم أن:

```text
<User>
↓
Response

userData
↓
Request Body
```

**الخطوة التالية: نكمل `axios.put()` ونشوف إزاي يختلف عن `post()`.**
</details>
const userData:  CreateUserRequest  {
name = "Ahmed"
email = "ahmed@example.com"
}
const response =   await axios.post<User>("/users", userData);
ChatGPT
**الهدف: تثبيت `axios.post()`، وإجابتك في الجزء الخاص بـ Axios صحيحة، والخطأ فقط في Syntax تعريف `userData`.** الوقت: **3 دقائق**.

### إجابتك

```ts
const userData: CreateUserRequest {
  name = "Ahmed"
  email = "ahmed@example.com"
}

const response = await axios.post<User>("/users", userData);
```

## التصحيح

الصحيح:

```ts
const userData: CreateUserRequest = {
  name: "Ahmed",
  email: "ahmed@example.com",
};

const response = await axios.post<User>("/users", userData);
```

### الخطأ الأول

كتبت:

```ts
const userData: CreateUserRequest {
```

لازم:

```ts
const userData: CreateUserRequest = {
```

لأننا بنقول:

```text
userData
   ↓
نوعه CreateUserRequest
   ↓
وقيمته = object
```

### الخطأ الثاني

كتبت:

```ts
name = "Ahmed"
```

داخل الـ object لازم:

```ts
name: "Ahmed"
```

نفس القاعدة:

```text
TypeScript type annotation
↓
:

Object property
↓
:
```

أما `=` فنستخدمها في **إسناد قيمة للمتغير**:

```ts
const x = 10;
```

---

## 🟢 التقييم

```text
فهم POST              ✅
فهم Request Body      ✅
فهم Response Type     ✅
استخدام Generic       ✅
TypeScript syntax     🟡
```

**التقييم: 9/10**

والأهم إنك فهمت الجزء الصعب:

```ts
axios.post<User>("/users", userData);
```

أنت الآن فاهم أن:

```text
<User>
↓
Response

userData
↓
Request Body
```

**الخطوة التالية: نكمل `axios.put()` ونشوف إزاي يختلف عن `post()`.**
<details>
<summary>انت كده مشي علي الخطة قسم قسم واحنا واقفين فين حاليا</summary>
user
انت كده مشي علي الخطة قسم قسم واحنا واقفين فين حاليا 
ChatGPT
**الهدف: نثبت مكاننا في الخطة بدون قفز، وحاليًا إحنا داخل Section 1، Lesson 2.** الوقت: **دقيقتان**.

## 🗺️ الخطة الحالية

### SECTION 0: Architecture Foundation
- ⬜ HTTP قبل Axios
- ⬜ Axios abstraction
- ⬜ TypeScript Foundation
- ⬜ Feature-Based Architecture

**الحالة: لم نبدأه كـ Section مستقل.**

لكن عمليًا أخذنا جزء الـ HTTP كتمهيد، وده اللي خلانا نفهم Axios.

---

# SECTION 1: Axios Fundamentals 🟢

### Lesson 1: Axios Request

- ✅ `axios.get()`
- ✅ `await`
- ✅ `response`
- ✅ `response.data`
- ✅ `axios.get<T>()`
- ✅ Generic `<T>`

### Lesson 2: Axios POST 🟢

- ✅ `axios.post()`
- ✅ URL
- ✅ Request Body
- ✅ Response Type
- ✅ الفرق بين:
  ```ts
  CreateUserRequest
  ```
  و:
  ```ts
  User
  ```
- 🟢 إحنا **خلصنا الجزء الأساسي من POST**

### المتبقي في Section 1

- ⬜ `PUT`
- ⬜ `PATCH`
- ⬜ `DELETE`
- ⬜ Request Config
- ⬜ `params`
- ⬜ `headers`
- ⬜ `data`
- ⬜ `timeout`
- ⬜ `signal`

---

## بعد Section 1

```text
SECTION 2
TypeScript + Axios
        ↓
SECTION 3
Error Handling
        ↓
SECTION 4
Interceptors
        ↓
SECTION 5
API Architecture
        ↓
SECTION 6
TanStack Query
        ↓
SECTION 7
Authentication
        ↓
SECTION 8
Forms
        ↓
SECTION 9
Tailwind + shadcn/ui
        ↓
SECTION 10
Storybook
        ↓
SECTION 11
Testing
        ↓
SECTION 12
Production
        ↓
SECTION 13
Final Project
```

### 📍 مكاننا بالضبط الآن

```text
SECTION 1
└── Lesson 2: POST
    ├── axios.post()       ✅
    ├── Request Body       ✅
    └── Response Type      ✅
    
    ↓ التالي
    
    PUT ⬜
```

**الخطوة التالية: نبدأ `PUT` فقط، بنفس أسلوبنا، شرح ثم مثال ثم تدريب.**
</details>
انت كده مشي علي الخطة قسم قسم واحنا واقفين فين حاليا 
ChatGPT
**الهدف: نثبت مكاننا في الخطة بدون قفز، وحاليًا إحنا داخل Section 1، Lesson 2.** الوقت: **دقيقتان**.

## 🗺️ الخطة الحالية

### SECTION 0: Architecture Foundation
- ⬜ HTTP قبل Axios
- ⬜ Axios abstraction
- ⬜ TypeScript Foundation
- ⬜ Feature-Based Architecture

**الحالة: لم نبدأه كـ Section مستقل.**

لكن عمليًا أخذنا جزء الـ HTTP كتمهيد، وده اللي خلانا نفهم Axios.

---

# SECTION 1: Axios Fundamentals 🟢

### Lesson 1: Axios Request

- ✅ `axios.get()`
- ✅ `await`
- ✅ `response`
- ✅ `response.data`
- ✅ `axios.get<T>()`
- ✅ Generic `<T>`

### Lesson 2: Axios POST 🟢

- ✅ `axios.post()`
- ✅ URL
- ✅ Request Body
- ✅ Response Type
- ✅ الفرق بين:
  ```ts
  CreateUserRequest
  ```
  و:
  ```ts
  User
  ```
- 🟢 إحنا **خلصنا الجزء الأساسي من POST**

### المتبقي في Section 1

- ⬜ `PUT`
- ⬜ `PATCH`
- ⬜ `DELETE`
- ⬜ Request Config
- ⬜ `params`
- ⬜ `headers`
- ⬜ `data`
- ⬜ `timeout`
- ⬜ `signal`

---

## بعد Section 1

```text
SECTION 2
TypeScript + Axios
        ↓
SECTION 3
Error Handling
        ↓
SECTION 4
Interceptors
        ↓
SECTION 5
API Architecture
        ↓
SECTION 6
TanStack Query
        ↓
SECTION 7
Authentication
        ↓
SECTION 8
Forms
        ↓
SECTION 9
Tailwind + shadcn/ui
        ↓
SECTION 10
Storybook
        ↓
SECTION 11
Testing
        ↓
SECTION 12
Production
        ↓
SECTION 13
Final Project
```

### 📍 مكاننا بالضبط الآن

```text
SECTION 1
└── Lesson 2: POST
    ├── axios.post()       ✅
    ├── Request Body       ✅
    └── Response Type      ✅
    
    ↓ التالي
    
    PUT ⬜
```

**الخطوة التالية: نبدأ `PUT` فقط، بنفس أسلوبنا، شرح ثم مثال ثم تدريب.**
<details>
<summary>SECTION 0: Architecture Foundation</summary>
user
SECTION 0: Architecture Foundation  
ده هناخده امتي عشان ميتنسيش 
2 عاوزين ديما الخطة تكون حاضره في زهننا في كل درس وبعدين وفين سؤال الانتر فيو معاك 
ChatGPT
**الهدف: نخلي الخطة ثابتة قدامنا في كل درس، ونأخذ Section 0 في وقته قبل التعمق في Axios، ومع كل Concept هيكون فيه Interview مناسب لما تكون المعلومة اتشرحت.** تعديل الخطة وتنظيمها: **3 دقائق**.

## 1. إمتى ناخد SECTION 0؟

**هنرجع له بعد ما نخلص الأساسيات الضرورية جدًا من HTTP وAxios Fundamentals.**

لكن مهم جدًا: أنا لا أريد أسيب Architecture لآخر الكورس، لأن وقتها ممكن تكون حفظت Axios من غير ما تفهم مكانه في المشروع.

لذلك الترتيب المعدل هيكون:

```text
SECTION 0
Architecture Foundation
        ↓
SECTION 1
Axios Fundamentals
        ↓
SECTION 2
TypeScript + Axios
        ↓
SECTION 3
Error Handling
...
```

**وإحنا حاليًا أخدنا تمهيد HTTP + جزء من Section 1، لكن هنرجع رسميًا لـ Section 0 قبل ما ندخل في Architecture المتقدمة.**

---

# 2. الخطة هتكون حاضرة في كل درس

من دلوقتي كل Lesson هبدأه بالشكل ده:

```text
📍 ROADMAP

SECTION 1: Axios Fundamentals 🟢

Lesson 2: POST
├── axios.post()       ✅
├── Request Body       ✅
└── Response Type      ✅

CURRENT:
└── PUT ⬅️ نحن هنا
```

وبعدها الدرس.

وفي النهاية:

```text
📊 PROGRESS

Section 1
██████░░░░ 60%

Completed:
✅ GET
✅ POST

Next:
⬜ PUT
```

كده مش هتتوه عن إحنا فين.

---

# 3. والـ Interview Questions؟

**هتكون جزء أساسي من المنهج، مش حاجة إضافية.**

لكن هنستخدمها في المكان الصح.

مثلاً إحنا شرحنا:

```ts
axios.get<User[]>("/users");
```

فالسؤال المناسب:

> What does `<User[]>` mean in this Axios request?

أنت جاوبت:

> أنا متوقع Array من Product/User.

✅ ممتاز.

لكن مش هسألك:

> How would you implement concurrent token refresh?

وأنت لسه ما درستش Interceptors.

---

## نظام الـ Interview

كل Lesson:

```text
شرح
 ↓
مثال
 ↓
تدريب
 ↓
تصحيح
 ↓
🎤 Interview Question
 ↓
تقييم إجابتك
 ↓
Next Lesson
```

ومع تقدمنا مستوى السؤال يزيد:

### Beginner

```text
What does axios.get() do?
```

### Intermediate

```text
Why use axios.get<User[]>()?
```

### Advanced

```text
How would you design
a reusable Axios client?
```

### Senior

```text
How would you handle
concurrent 401 responses?
```

لكن **كل سؤال في وقته**.

---

# 4. وفي نهاية كل Section

هيكون فيه:

```text
🎯 SECTION REVIEW

10 Questions
    ↓
Coding Challenge
    ↓
Architecture Question
    ↓
Real-world Scenario
```

وأنا هقيّمك:

```text
Knowledge       8/10
TypeScript      7/10
Architecture    6/10
Problem Solving 9/10
```

وبعدها نقرر هل ننتقل للـ Section التالي أم محتاجين Review.

---

# 📍 مكاننا الآن

```text
AXIOS PROFESSIONAL ROADMAP

SECTION 0  Architecture Foundation    🟡
SECTION 1  Axios Fundamentals         🟢 ← نحن هنا
SECTION 2  TypeScript + Axios         ⬜
SECTION 3  Error Handling             ⬜
SECTION 4  Interceptors               ⬜
SECTION 5  API Architecture           ⬜
SECTION 6  TanStack Query              ⬜
SECTION 7  Authentication              ⬜
SECTION 8  Forms                       ⬜
SECTION 9  Tailwind + shadcn/ui        ⬜
SECTION 10 Storybook                   ⬜
SECTION 11 Testing                     ⬜
SECTION 12 Production Architecture    ⬜
SECTION 13 Final Project               ⬜
```

**وحاليًا تحديدًا:**

```text
SECTION 1
└── Axios Fundamentals
    └── POST ✅
        ↓
      PUT ⬅️ التالي
```

ومش هننتقل للـ PUT إلا لما نخلصه **شرح + تطبيق + Interview Question**.

**الخطوة التالية: نبدأ `PUT`، وأول سطر في الدرس هيعرض الـ Roadmap ومكاننا الحالي.**
</details>
SECTION 0: Architecture Foundation  
ده هناخده امتي عشان ميتنسيش 
2 عاوزين ديما الخطة تكون حاضره في زهننا في كل درس وبعدين وفين سؤال الانتر فيو معاك 
ChatGPT
**الهدف: نخلي الخطة ثابتة قدامنا في كل درس، ونأخذ Section 0 في وقته قبل التعمق في Axios، ومع كل Concept هيكون فيه Interview مناسب لما تكون المعلومة اتشرحت.** تعديل الخطة وتنظيمها: **3 دقائق**.

## 1. إمتى ناخد SECTION 0؟

**هنرجع له بعد ما نخلص الأساسيات الضرورية جدًا من HTTP وAxios Fundamentals.**

لكن مهم جدًا: أنا لا أريد أسيب Architecture لآخر الكورس، لأن وقتها ممكن تكون حفظت Axios من غير ما تفهم مكانه في المشروع.

لذلك الترتيب المعدل هيكون:

```text
SECTION 0
Architecture Foundation
        ↓
SECTION 1
Axios Fundamentals
        ↓
SECTION 2
TypeScript + Axios
        ↓
SECTION 3
Error Handling
...
```

**وإحنا حاليًا أخدنا تمهيد HTTP + جزء من Section 1، لكن هنرجع رسميًا لـ Section 0 قبل ما ندخل في Architecture المتقدمة.**

---

# 2. الخطة هتكون حاضرة في كل درس

من دلوقتي كل Lesson هبدأه بالشكل ده:

```text
📍 ROADMAP

SECTION 1: Axios Fundamentals 🟢

Lesson 2: POST
├── axios.post()       ✅
├── Request Body       ✅
└── Response Type      ✅

CURRENT:
└── PUT ⬅️ نحن هنا
```

وبعدها الدرس.

وفي النهاية:

```text
📊 PROGRESS

Section 1
██████░░░░ 60%

Completed:
✅ GET
✅ POST

Next:
⬜ PUT
```

كده مش هتتوه عن إحنا فين.

---

# 3. والـ Interview Questions؟

**هتكون جزء أساسي من المنهج، مش حاجة إضافية.**

لكن هنستخدمها في المكان الصح.

مثلاً إحنا شرحنا:

```ts
axios.get<User[]>("/users");
```

فالسؤال المناسب:

> What does `<User[]>` mean in this Axios request?

أنت جاوبت:

> أنا متوقع Array من Product/User.

✅ ممتاز.

لكن مش هسألك:

> How would you implement concurrent token refresh?

وأنت لسه ما درستش Interceptors.

---

## نظام الـ Interview

كل Lesson:

```text
شرح
 ↓
مثال
 ↓
تدريب
 ↓
تصحيح
 ↓
🎤 Interview Question
 ↓
تقييم إجابتك
 ↓
Next Lesson
```

ومع تقدمنا مستوى السؤال يزيد:

### Beginner

```text
What does axios.get() do?
```

### Intermediate

```text
Why use axios.get<User[]>()?
```

### Advanced

```text
How would you design
a reusable Axios client?
```

### Senior

```text
How would you handle
concurrent 401 responses?
```

لكن **كل سؤال في وقته**.

---

# 4. وفي نهاية كل Section

هيكون فيه:

```text
🎯 SECTION REVIEW

10 Questions
    ↓
Coding Challenge
    ↓
Architecture Question
    ↓
Real-world Scenario
```

وأنا هقيّمك:

```text
Knowledge       8/10
TypeScript      7/10
Architecture    6/10
Problem Solving 9/10
```

وبعدها نقرر هل ننتقل للـ Section التالي أم محتاجين Review.

---

# 📍 مكاننا الآن

```text
AXIOS PROFESSIONAL ROADMAP

SECTION 0  Architecture Foundation    🟡
SECTION 1  Axios Fundamentals         🟢 ← نحن هنا
SECTION 2  TypeScript + Axios         ⬜
SECTION 3  Error Handling             ⬜
SECTION 4  Interceptors               ⬜
SECTION 5  API Architecture           ⬜
SECTION 6  TanStack Query              ⬜
SECTION 7  Authentication              ⬜
SECTION 8  Forms                       ⬜
SECTION 9  Tailwind + shadcn/ui        ⬜
SECTION 10 Storybook                   ⬜
SECTION 11 Testing                     ⬜
SECTION 12 Production Architecture    ⬜
SECTION 13 Final Project               ⬜
```

**وحاليًا تحديدًا:**

```text
SECTION 1
└── Axios Fundamentals
    └── POST ✅
        ↓
      PUT ⬅️ التالي
```

ومش هننتقل للـ PUT إلا لما نخلصه **شرح + تطبيق + Interview Question**.

**الخطوة التالية: نبدأ `PUT`، وأول سطر في الدرس هيعرض الـ Roadmap ومكاننا الحالي.**
<details>
<summary>PUT</summary>
user
PUT
ChatGPT
**الهدف: نتعلم `axios.put()` فقط، ونفهمه من الأساس ثم نطبقه، وبعدها سؤال Interview على نفس اللي اتشرح.** الوقت: **10 إلى 15 دقيقة**.

## 📍 Roadmap

```text
SECTION 1: Axios Fundamentals 🟢

GET                     ✅
POST                    ✅
PUT                     🟢 ← نحن هنا
PATCH                   ⬜
DELETE                  ⬜
Request Config          ⬜
```

---
</details>
PUT
ChatGPT
**الهدف: نتعلم `axios.put()` فقط، ونفهمه من الأساس ثم نطبقه، وبعدها سؤال Interview على نفس اللي اتشرح.** الوقت: **10 إلى 15 دقيقة**.

## 📍 Roadmap

```text
SECTION 1: Axios Fundamentals 🟢

GET                     ✅
POST                    ✅
PUT                     🟢 ← نحن هنا
PATCH                   ⬜
DELETE                  ⬜
Request Config          ⬜
```

---