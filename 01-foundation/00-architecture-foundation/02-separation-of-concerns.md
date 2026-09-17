# 0.2 Separation of Concerns

خلينا ناخدها بأبسط معنى:

> **كل جزء من التطبيق يهتم بمسؤولية معينة، وما نخلطش المسؤوليات المختلفة مع بعض.**

كلمة **Concern** معناها هنا: جانب أو مسؤولية من مسؤوليات النظام.

مثلاً تطبيق Users عنده:

```text
UI
API Communication
Authentication
Validation
State
Business Logic
```

دي كلها **Concerns مختلفة**.

---

# مثال سيئ

تخيل عندنا:

```tsx
function UsersPage() {
  // جلب البيانات
  const response = await axios.get("/users");

  // معالجة البيانات
  const users = response.data.payload;

  // Authentication
  const token = localStorage.getItem("token");

  // Business Logic
  const activeUsers = users.filter(user => user.active);

  // UI
  return (
    <div>
      {activeUsers.map(user => (
        <div>{user.name}</div>
      ))}
    </div>
  );
}
```

بص على الـComponent ده.

هو بيعمل:

```text
UsersPage
│
├── HTTP
├── Authentication
├── Data Processing
├── Business Logic
└── UI
```

المشكلة هنا مش إن الكود طويل فقط.

المشكلة إن **مسؤوليات مختلفة اتجمعت في نفس المكان**.

---

# الحل

نبدأ نفصل.

### API

```ts
async function getUsers() {
  const response =
    await axios.get("/users");

  return response.data.payload;
}
```

### Authentication

```ts
function getToken() {
  return localStorage.getItem("token");
}
```

### Business Logic

```ts
function getActiveUsers(users: IUser[]) {
  return users.filter(user => user.active);
}
```

### UI

```tsx
function UsersPage() {
  // يستخدم الوظائف السابقة

  return (
    <div>
      {/* UI */}
    </div>
  );
}
```

بقى عندنا:

```text
UsersPage
    │
    ├── getUsers()
    │
    ├── getToken()
    │
    └── getActiveUsers()
```

كل جزء عنده **Concern** أوضح.

---

# ليه الفصل ده مهم؟

لسبب مهم جدًا:

## 1. التعديل

افترض إن الـBackend غير:

```text
/users
```

إلى:

```text
/api/v2/users
```

لو Axios موجود في 15 Component:

```text
UsersPage
OrdersPage
Dashboard
Profile
...
```

هتعدل في أماكن كتير.

لكن لو عندك:

```text
API Layer
    ↓
getUsers()
```

تعدل في مكان واحد.

---

## 2. إعادة الاستخدام

ممكن:

```text
UsersPage
Dashboard
AdminUsersPage
```

كلهم يحتاجوا Users.

بدل تكرار:

```ts
axios.get("/users")
```

في كل مكان، عندك:

```ts
getUsers()
```

---

## 3. الاختبار

لما المسؤوليات منفصلة، تقدر تختبر كل جزء بشكل مستقل.

مثلاً:

```text
getUsers()
```

تختبر الـAPI.

و:

```text
getActiveUsers()
```

تختبر الـBusiness Logic.

والـComponent تختبر الـUI.

---

# أهم نقطة

**Separation of Concerns مش معناها إن كل سطر كود لازم يكون في ملف منفصل.**

دي غلطة شائعة جدًا.

مش مطلوب تعمل:

```text
getUser.ts
getUserName.ts
getUserEmail.ts
getUserId.ts
```

😂

المطلوب هو فصل **المسؤوليات المختلفة** بشكل منطقي.

---

# نربطها بالـAxios اللي درسناه

أنت عملت:

```ts
export const getUser = async (
  id: number
): Promise<IUser> => {
  const response =
    await axios.get<IApiResponse<IUser>>(
      `/users/${id}`
    );

  return response.data.payload;
};
```

هنا:

```text
getUser()
   ↓
API Concern
```

والـComponent:

```tsx
const user = await getUser(10);

return <div>{user.name}</div>;
```

هنا:

```text
UsersPage
   ↓
UI Concern
```

وده بالضبط تطبيق **Separation of Concerns**.

---

# 🔥 اختبار مهم

عندك:

```tsx
function ProductPage() {

  const response =
    await axios.get("/products/10");

  const product =
    response.data.payload;

  const finalPrice =
    product.price -
    product.discount;

  return (
    <div>
      <h1>{product.name}</h1>
      <p>{finalPrice}</p>
    </div>
  );
}
```

أنا مش عايز منك تصلح الكود.

عايزك تحدد فقط:

**إيه الـConcerns المختلفة الموجودة جوه `ProductPage`؟**

اكتبهم كده:

```text
1. ...
2. ...
3. ...
4. ...
```

**الخطوة التالية: حدد الـConcerns الأربعة في `ProductPage`.**
<details>
<summary>المكونات ui</summary>
user
المكونات ui
جلب بيانات
البيزنس لوجك
ChatGPT
**ممتاز، إجابتك صحيحة، وتقييمك 10/10، واحتجنا حوالي دقيقة واحدة فقط.**

أنت حددت الـ3 مسؤوليات الأساسية:

```text
ProductPage
│
├── UI
│
├── Data Fetching
│
└── Business Logic
```

### لكن فيه تفصيلة مهمة

في الكود:

```ts
const product =
  response.data.payload;
```

دي مش مسؤولية جديدة بالضرورة، دي جزء من **Data Fetching / Data Transformation**.

يعني مش محتاجين نعتبرها Concern رابعة.

---

## الشكل المعماري الأفضل

بدل:

```text
ProductPage
│
├── Axios
├── Data transformation
├── Business logic
└── UI
```

نفصل:

```text
API Layer
   │
   └── getProduct()
          ↓
       Product
          ↓
Business Logic
          ↓
      ProductPage
          ↓
          UI
```

لاحظ حاجة مهمة جدًا:

**Separation of Concerns مش هدفه إننا نعمل ملفات كتير.**

هدفه إننا نخلي كل مسؤولية في **المكان المناسب**.

وده هيبقى أساس مهم جدًا لما نوصل لـ:

```text
Feature-Based Architecture
        ↓
API Layer
        ↓
TanStack Query
        ↓
Components
```

---

## 🗺️ Roadmap

```text
SECTION 0: Architecture Foundation 🟢
  ├─ 0.1 HTTP Architecture          ✅
  ├─ 0.2 Separation of Concerns     ✅
  ├─ 0.3 Coupling & Cohesion        🟢 ← التالي
  ├─ 0.4 Feature-Based Architecture⬜
  ├─ 0.5 Shared vs Feature         ⬜
  ├─ 0.6 Public vs Internal API     ⬜
  └─ 0.7 Dependency Direction      ⬜
```

**الخطوة التالية: اكتب `3` لنبدأ `Coupling & Cohesion` في حوالي 15 دقيقة.**
</details>
المكونات ui
جلب بيانات
البيزنس لوجك
ChatGPT
**ممتاز، إجابتك صحيحة، وتقييمك 10/10، واحتجنا حوالي دقيقة واحدة فقط.**

أنت حددت الـ3 مسؤوليات الأساسية:

```text
ProductPage
│
├── UI
│
├── Data Fetching
│
└── Business Logic
```

### لكن فيه تفصيلة مهمة

في الكود:

```ts
const product =
  response.data.payload;
```

دي مش مسؤولية جديدة بالضرورة، دي جزء من **Data Fetching / Data Transformation**.

يعني مش محتاجين نعتبرها Concern رابعة.

---

## الشكل المعماري الأفضل

بدل:

```text
ProductPage
│
├── Axios
├── Data transformation
├── Business logic
└── UI
```

نفصل:

```text
API Layer
   │
   └── getProduct()
          ↓
       Product
          ↓
Business Logic
          ↓
      ProductPage
          ↓
          UI
```

لاحظ حاجة مهمة جدًا:

**Separation of Concerns مش هدفه إننا نعمل ملفات كتير.**

هدفه إننا نخلي كل مسؤولية في **المكان المناسب**.

وده هيبقى أساس مهم جدًا لما نوصل لـ:

```text
Feature-Based Architecture
        ↓
API Layer
        ↓
TanStack Query
        ↓
Components
```

---

## 🗺️ Roadmap

```text
SECTION 0: Architecture Foundation 🟢
  ├─ 0.1 HTTP Architecture          ✅
  ├─ 0.2 Separation of Concerns     ✅
  ├─ 0.3 Coupling & Cohesion        🟢 ← التالي
  ├─ 0.4 Feature-Based Architecture⬜
  ├─ 0.5 Shared vs Feature         ⬜
  ├─ 0.6 Public vs Internal API     ⬜
  └─ 0.7 Dependency Direction      ⬜
```

**الخطوة التالية: اكتب `3` لنبدأ `Coupling & Cohesion` في حوالي 15 دقيقة.**
<details>
<summary>كمل</summary>
user
كمل
ChatGPT
**ابدأ بـ `0.3 Coupling & Cohesion` الآن، ومدته حوالي 20 دقيقة، لأنه من أهم المفاهيم اللي هتخليك تفهم ليه بنفصل الـAPI والـComponents والـFeatures.**

## 🗺️ Roadmap

```text
SECTION 0: Architecture Foundation 🟢
  ├─ 0.1 HTTP Architecture           ✅
  ├─ 0.2 Separation of Concerns      ✅
  ├─ 0.3 Coupling & Cohesion         🟢 ← نحن هنا
  ├─ 0.4 Feature-Based Architecture ⬜
  ├─ 0.5 Shared vs Feature          ⬜
  ├─ 0.6 Public vs Internal API     ⬜
  └─ 0.7 Dependency Direction       ⬜

SECTION 1 Axios Fundamentals         ✅
SECTION 2 TypeScript + Axios         ✅
SECTION 3 Error Handling             ✅
SECTION 4 Interceptors               ⬜
```
</details>
كمل
ChatGPT
**ابدأ بـ `0.3 Coupling & Cohesion` الآن، ومدته حوالي 20 دقيقة، لأنه من أهم المفاهيم اللي هتخليك تفهم ليه بنفصل الـAPI والـComponents والـFeatures.**

## 🗺️ Roadmap

```text
SECTION 0: Architecture Foundation 🟢
  ├─ 0.1 HTTP Architecture           ✅
  ├─ 0.2 Separation of Concerns      ✅
  ├─ 0.3 Coupling & Cohesion         🟢 ← نحن هنا
  ├─ 0.4 Feature-Based Architecture ⬜
  ├─ 0.5 Shared vs Feature          ⬜
  ├─ 0.6 Public vs Internal API     ⬜
  └─ 0.7 Dependency Direction       ⬜

SECTION 1 Axios Fundamentals         ✅
SECTION 2 TypeScript + Axios         ✅
SECTION 3 Error Handling             ✅
SECTION 4 Interceptors               ⬜
```