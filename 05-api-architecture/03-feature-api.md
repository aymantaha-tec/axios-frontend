# 5.3 Feature API

الفكرة بسيطة جدًا:

> **كل Feature يكون عندها ملف مسؤول عن الـ API operations الخاصة بها.**

مثلاً عندنا Feature اسمها `users`.

```text id="h4r4b8"
features/
└── users/
    ├── api/
    │   └── userApi.ts
    ├── components/
    ├── hooks/
    └── types/
```

داخل `userApi.ts`:

```ts id="f0e6sl"
import { api } from "@/shared/api/apiClient";

export const getUsers = async () => {
  const response = await api.get("/users");

  return response.data;
};

export const getUser = async (id: number) => {
  const response = await api.get(`/users/${id}`);

  return response.data;
};

export const createUser = async (data: ICreateUser) => {
  const response = await api.post("/users", data);

  return response.data;
};

export const updateUser = async (
  id: number,
  data: IUpdateUser
) => {
  const response = await api.patch(`/users/${id}`, data);

  return response.data;
};

export const deleteUser = async (id: number) => {
  const response = await api.delete(`/users/${id}`);

  return response.data;
};
```

لاحظ إن `userApi.ts` **مش مسؤول عن الـ UI**.

هو مسؤول عن:

```text id="h1z2g8"
Users Feature
     ↓
API Operations
     ↓
Backend
```

---

## الفرق بين API Client و Feature API

دي أهم نقطة في الدرس:

### API Client

```ts id="o7y3vf"
apiClient.ts
```

مسؤوليته:

```text
كيف نتصل بالـ Backend؟
```

مثلاً:

```text
baseURL
timeout
interceptors
common configuration
```

### Feature API

```ts id="7s4l6c"
userApi.ts
```

مسؤوليته:

```text
ما العمليات التي نحتاجها من Users API؟
```

مثلاً:

```text
getUsers()
getUser()
createUser()
updateUser()
deleteUser()
```

---

## الشكل الكامل

```text id="qz3q7y"
Component
    ↓
users/api/userApi.ts
    ↓
shared/api/apiClient.ts
    ↓
Axios
    ↓
Backend
```

وده يخلي الـ Component يقول:

```ts id="q1ndv9"
const user = await getUser(10);
```

بدل ما يعرف:

```ts id="y7v4tq"
axios.get("/users/10");
```

---

### قاعدة مهمة جدًا

**Feature API تعرف الـ business/domain endpoint، لكن API Client لا يعرف أي Feature.**

يعني:

```text id="8w7m6u"
userApi → apiClient     ✅

productApi → apiClient  ✅

orderApi → apiClient    ✅

apiClient → users       ❌
apiClient → products    ❌
apiClient → orders      ❌
```

وده بيرجعنا لمبدأ **Dependency Direction** اللي أخدناه في Section 0.

### 🎯 سؤال سريع

لو عندك:

```text
users
products
orders
```

هل نعمل **3 Axios Clients**، واحد لكل Feature، ولا **Axios Client واحد مشترك** والـ Features كلها تستخدمه؟

**جاوب في سطر واحد.**
<details>
<summary>واحد فقط</summary>
user
واحد فقط
ChatGPT
**صح، 10/10.** ✅

```text id="nqj4x2"
                API Client
                    │
        ┌───────────┼───────────┐
        ↓           ↓           ↓
     users       products      orders
```

- **Axios Client واحد** في `shared/api`.
- كل Feature لها **Feature API** خاص بها.
- الـ `users` لا تعرف تفاصيل `products`.
- الـ `apiClient` لا يعرف أي Feature.

```text id="5l2k4r"
shared/api/apiClient.ts
        ↑
        │
 ┌──────┼──────┐
 ↓      ↓      ↓
users products orders
```

### 📍 تقدمك

```text
5.1 API Layer vs Component   ✅
5.2 API Client               ✅
5.3 Feature API              ✅

5.4 Service Layer            🟢 ← التالي
5.5 Dependency Direction     ⬜
5.6 Complete Architecture    ⬜
```

**الخطوة التالية: نبدأ 5.4 `Service Layer`، ونفرق لأول مرة بين `API Layer` و`Service Layer` لأن الاتنين مش نفس الحاجة.**
</details>
واحد فقط
ChatGPT
**صح، 10/10.** ✅

```text id="nqj4x2"
                API Client
                    │
        ┌───────────┼───────────┐
        ↓           ↓           ↓
     users       products      orders
```

- **Axios Client واحد** في `shared/api`.
- كل Feature لها **Feature API** خاص بها.
- الـ `users` لا تعرف تفاصيل `products`.
- الـ `apiClient` لا يعرف أي Feature.

```text id="5l2k4r"
shared/api/apiClient.ts
        ↑
        │
 ┌──────┼──────┐
 ↓      ↓      ↓
users products orders
```

### 📍 تقدمك

```text
5.1 API Layer vs Component   ✅
5.2 API Client               ✅
5.3 Feature API              ✅

5.4 Service Layer            🟢 ← التالي
5.5 Dependency Direction     ⬜
5.6 Complete Architecture    ⬜
```

**الخطوة التالية: نبدأ 5.4 `Service Layer`، ونفرق لأول مرة بين `API Layer` و`Service Layer` لأن الاتنين مش نفس الحاجة.**
<details>
<summary>5.4 Service Layer</summary>
user
5.4 Service Layer
ChatGPT
**خلال 15 دقيقة هنفهم `Service Layer`، والهدف إنك تفرق بوضوح بين `API Layer` و`Service Layer` ومتى نحتاج كل واحد.**

### 🗺️ Roadmap

```text id="x8v9p2"
SECTION 5  API Architecture 🟢
 ├─ 5.1 API Layer vs Component       ✅
 ├─ 5.2 API Client                   ✅
 ├─ 5.3 Feature API                  ✅
 ├─ 5.4 Service Layer                🟢 ← هنا
 ├─ 5.5 Dependency Direction         ⬜
 └─ 5.6 Complete API Architecture    ⬜
```
</details>
5.4 Service Layer
ChatGPT
**خلال 15 دقيقة هنفهم `Service Layer`، والهدف إنك تفرق بوضوح بين `API Layer` و`Service Layer` ومتى نحتاج كل واحد.**

### 🗺️ Roadmap

```text id="x8v9p2"
SECTION 5  API Architecture 🟢
 ├─ 5.1 API Layer vs Component       ✅
 ├─ 5.2 API Client                   ✅
 ├─ 5.3 Feature API                  ✅
 ├─ 5.4 Service Layer                🟢 ← هنا
 ├─ 5.5 Dependency Direction         ⬜
 └─ 5.6 Complete API Architecture    ⬜
```