# 0.1 يعني إيه Architecture؟

خلينا نبدأ من الصفر.

لما المشروع يكبر، المشكلة مش إن الكود **يشتغل**.

المشكلة:

> **الكود هيفضل قابل للفهم والتعديل والتوسع ولا لأ؟**

مثلاً عندك Component:

```tsx
function UsersPage() {
  // UI

  // axios request

  // error handling

  // authentication

  // data transformation

  // business logic

  // loading state

  // ...
}
```

الكود ممكن يشتغل عادي جدًا.

لكن بعد شهر عندك:

```text
Users
Products
Orders
Profile
Dashboard
Authentication
Payments
...
```

وتبدأ Components تكبر جدًا.

هنا تظهر أهمية **Architecture**.

---

# 0.2 Architecture ببساطة

Architecture هي:

> **طريقة تنظيم أجزاء التطبيق والعلاقات بينها.**

يعني مش مجرد:

```text
folders
```

ولا مجرد:

```text
files
```

الـArchitecture بتحدد:

```text
مين مسؤول عن إيه؟
مين يقدر يكلم مين؟
فين نحط الكود؟
إزاي الأجزاء تعتمد على بعضها؟
```

---

# 0.3 مثال بسيط جدًا

تخيل مطعم.

عندك:

```text
Customer
   ↓
Waiter
   ↓
Kitchen
   ↓
Chef
```

العميل مش المفروض يدخل المطبخ ويبدأ يطبخ.

ليه؟

لأن كل جزء له **Responsibility**.

```text
Customer → طلب
Waiter   → استقبال الطلب
Kitchen  → تجهيز
Chef     → طبخ
```

نفس الفكرة في Software.

مثلاً:

```text
Component
    ↓
API Layer
    ↓
Axios
    ↓
Backend
```

الـComponent مش المفروض يعرف تفاصيل كل حاجة عن Axios.

---

# 0.4 Separation of Concerns

دي واحدة من أهم الأفكار في الـArchitecture.

**Separation of Concerns**

يعني:

> كل جزء من النظام يكون مسؤول عن Concern محدد.

مثلاً:

```text
Component
    ↓
UI

API Layer
    ↓
HTTP communication

Auth
    ↓
Authentication

Store
    ↓
Global State

Query
    ↓
Server State
```

بدل ما نحط كل حاجة في مكان واحد.

---

# 0.5 نربطها باللي أنت درسته

فاكر لما عملنا:

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

ليه ما عملناش Axios مباشرة داخل Component؟

بدل:

```tsx
function ProfileUser() {
  // axios.get(...)
}
```

عملنا:

```text
ProfileUser
     ↓
   getUser()
     ↓
    Axios
     ↓
   Backend
```

لأننا فصلنا المسؤوليات.

الـComponent يقول:

> أنا محتاج User.

والـAPI Layer تقول:

> أنا هتصرف إزاي أجيب الـUser.

دي **Separation of Concerns**.

---

# 0.6 أهم سؤال في Architecture

كل ما تشوف كود، اسأل نفسك:

> **مين المفروض يكون مسؤول عن الكود ده؟**

مثلاً:

```ts
axios.get("/users")
```

هل ده UI؟

❌ لا.

ده:

```text
HTTP communication
```

إذن مكانه الطبيعي يكون في:

```text
API Layer
```

---

مثال آخر:

```tsx
<button onClick={handleSubmit}>
  Save
</button>
```

ده:

```text
UI
```

إذن Component مسؤول عنه.

---

مثال:

```ts
localStorage.setItem("token", token);
```

ده متعلق بـ:

```text
Authentication / persistence
```

مش UI بشكل مباشر.

---

# 0.7 الهدف الحقيقي من Section 0

إحنا مش بندرس Architecture عشان تحفظ أسماء.

إحنا عايزين نوصل لمرحلة لما تشوف مشروع تقول:

```text
آه، الكود ده مكانه هنا.
والكود ده مكانه هناك.
وده مينفعش يعتمد على ده.
وده المفروض يتعامل مع الـAPI.
```

وبعدها لما ندخل:

```text
Interceptors
API Layer
TanStack Query
Authentication
Feature-Based Architecture
```

هتفهم **ليه بنحط كل حاجة في مكانها** بدل ما تحفظ Structure وخلاص.

---

## 🎯 اختبار سريع

**جاوب من فهمك، من غير ما ترجع للشرح:**

عندك Component اسمه:

```tsx
UsersPage
```

وفيه:

```ts
axios.get("/users")
```

وفيه كمان:

```ts
if (error.response?.status === 401) {
  // ...
}
```

وفيه:

```ts
localStorage.setItem("token", token);
```

**هل من الأفضل إن الثلاثة يكونوا داخل `UsersPage`؟ وليه؟**

**الخطوة التالية: جاوب على السؤال ده، وبعدها هقيّم إجابتك وننتقل إلى 0.2 فقط.**
<details>
<summary>لا ماينفعش عشان قائم بي كذا وظيفه</summary>
user
لا ماينفعش عشان قائم بي كذا وظيفه
ChatGPT
**صح، إجابتك صحيحة، وتقييمك 10/10، ومدتها أقل من دقيقة.**

أنت لقطت الفكرة الأساسية:

> الـ`UsersPage` بقى مسؤول عن أكتر من وظيفة، وده يخالف مبدأ **Separation of Concerns**.

عندنا 3 مسؤوليات مختلفة:

```text
UsersPage
   │
   ├── UI
   │
   ├── HTTP / Axios
   │
   ├── Authentication
   │
   └── Persistence / localStorage
```

والأفضل نفصلهم:

```text
UsersPage
   │
   ├── UI
   │
   ├── API Layer
   │      └── Axios
   │
   └── Auth Layer
          └── Token
```

### نقطة مهمة جدًا

وجود أكثر من سطر كود **مش هو المشكلة**.

المشكلة إن الـComponent عنده **أكثر من Responsibility**.

يعني ده عادي:

```tsx
function UsersPage() {
  const title = "Users";
  const showUsers = true;

  return <h1>{title}</h1>;
}
```

لأن كل ده متعلق بالـUI.

لكن:

```tsx
function UsersPage() {
  axios.get(...);
  localStorage.setItem(...);
  // UI
  // authentication
  // error handling
}
```

بدأنا نخلط **Concerns مختلفة**.

---

## 🗺️ Roadmap

```text
SECTION 0: Architecture Foundation 🟢
  ├─ 0.1 HTTP Architecture        ✅
  ├─ 0.2 Separation of Concerns    🟢 ← التالي
  ├─ 0.3 Coupling & Cohesion       ⬜
  ├─ 0.4 Feature-Based Architecture⬜
  ├─ 0.5 Shared vs Feature         ⬜
  ├─ 0.6 Public vs Internal API    ⬜
  └─ 0.7 Dependency Direction      ⬜
```

**الخطوة التالية: نبدأ 0.2 ونفهم الفرق بين `Responsibility` و`Concern` عمليًا في حوالي 15 دقيقة.**
</details>
لا ماينفعش عشان قائم بي كذا وظيفه
ChatGPT
**صح، إجابتك صحيحة، وتقييمك 10/10، ومدتها أقل من دقيقة.**

أنت لقطت الفكرة الأساسية:

> الـ`UsersPage` بقى مسؤول عن أكتر من وظيفة، وده يخالف مبدأ **Separation of Concerns**.

عندنا 3 مسؤوليات مختلفة:

```text
UsersPage
   │
   ├── UI
   │
   ├── HTTP / Axios
   │
   ├── Authentication
   │
   └── Persistence / localStorage
```

والأفضل نفصلهم:

```text
UsersPage
   │
   ├── UI
   │
   ├── API Layer
   │      └── Axios
   │
   └── Auth Layer
          └── Token
```

### نقطة مهمة جدًا

وجود أكثر من سطر كود **مش هو المشكلة**.

المشكلة إن الـComponent عنده **أكثر من Responsibility**.

يعني ده عادي:

```tsx
function UsersPage() {
  const title = "Users";
  const showUsers = true;

  return <h1>{title}</h1>;
}
```

لأن كل ده متعلق بالـUI.

لكن:

```tsx
function UsersPage() {
  axios.get(...);
  localStorage.setItem(...);
  // UI
  // authentication
  // error handling
}
```

بدأنا نخلط **Concerns مختلفة**.

---

## 🗺️ Roadmap

```text
SECTION 0: Architecture Foundation 🟢
  ├─ 0.1 HTTP Architecture        ✅
  ├─ 0.2 Separation of Concerns    🟢 ← التالي
  ├─ 0.3 Coupling & Cohesion       ⬜
  ├─ 0.4 Feature-Based Architecture⬜
  ├─ 0.5 Shared vs Feature         ⬜
  ├─ 0.6 Public vs Internal API    ⬜
  └─ 0.7 Dependency Direction      ⬜
```

**الخطوة التالية: نبدأ 0.2 ونفهم الفرق بين `Responsibility` و`Concern` عمليًا في حوالي 15 دقيقة.**
<details>
<summary>٢</summary>
user
٢
ChatGPT
**ابدأ بـ 0.2 الآن لمدة 15 دقيقة: هنتعلم `Separation of Concerns` بشكل عملي، وهنربطه مباشرة بالكود اللي أنت كتبته في Axios.**

## 🗺️ Roadmap

```text
SECTION 0: Architecture Foundation 🟢
  ├─ 0.1 HTTP Architecture          ✅
  ├─ 0.2 Separation of Concerns     🟢 ← نحن هنا
  ├─ 0.3 Coupling & Cohesion        ⬜
  ├─ 0.4 Feature-Based Architecture⬜
  ├─ 0.5 Shared vs Feature          ⬜
  ├─ 0.6 Public vs Internal API     ⬜
  └─ 0.7 Dependency Direction       ⬜
```
</details>
٢
ChatGPT
**ابدأ بـ 0.2 الآن لمدة 15 دقيقة: هنتعلم `Separation of Concerns` بشكل عملي، وهنربطه مباشرة بالكود اللي أنت كتبته في Axios.**

## 🗺️ Roadmap

```text
SECTION 0: Architecture Foundation 🟢
  ├─ 0.1 HTTP Architecture          ✅
  ├─ 0.2 Separation of Concerns     🟢 ← نحن هنا
  ├─ 0.3 Coupling & Cohesion        ⬜
  ├─ 0.4 Feature-Based Architecture⬜
  ├─ 0.5 Shared vs Feature          ⬜
  ├─ 0.6 Public vs Internal API     ⬜
  └─ 0.7 Dependency Direction       ⬜
```