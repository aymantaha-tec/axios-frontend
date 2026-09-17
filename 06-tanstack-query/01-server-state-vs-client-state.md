# 6.1 Server State vs Client State

قبل `TanStack Query` لازم نفهم المشكلة اللي هو معمول علشان يحلها.

عندك نوعين أساسيين من الـ State.

## 1. Client State

دي البيانات اللي **التطبيق نفسه مسؤول عنها**.

مثلاً:

```text
isModalOpen
selectedTab
sidebarOpen
theme
inputValue
```

مثال:

```tsx
const [isOpen, setIsOpen] = useState(false);
```

هنا:

```text
React App
   ↓
Owns the State
```

التطبيق هو صاحب البيانات.

---

# 2. Server State

دي البيانات اللي **مصدرها Backend**.

مثلاً:

```text
Users
Products
Orders
Profile
Notifications
```

أنت تعمل:

```ts
const response = await api.get("/users");
```

لكن البيانات نفسها موجودة في:

```text
Backend / Database
```

مش داخل React.

```text
Backend
   ↓
Users Data
   ↓
React
```

إذن React عنده **نسخة من بيانات الـ Server**.

---

# 🔥 المشكلة

تخيل إن عندك:

```text
/users
```

وعملت Request:

```text
GET /users
```

جاب:

```json
[
  { "id": 1, "name": "Ahmed" },
  { "id": 2, "name": "Mohamed" }
]
```

مين المسؤول عن البيانات؟

```text
Backend
```

طيب لو Component تاني محتاج نفس البيانات؟

ممكن تعمل Request جديد:

```text
Component A → GET /users
Component B → GET /users
Component C → GET /users
```

هنا بدأت تظهر مشاكل:

```text
Duplicate Requests
Caching
Loading State
Error State
Refetching
Synchronization
Stale Data
```

وهنا بيظهر دور **TanStack Query**.

---

# TanStack Query بيحل إيه؟

هو مش بديل لـ Axios.

دي نقطة مهمة جدًا.

```text
Axios
↓
HTTP Communication
```

بينما:

```text
TanStack Query
↓
Server State Management
```

يعني ممكن يكون عندك:

```text
TanStack Query
       ↓
      Axios
       ↓
    Backend
```

مثلاً:

```ts
const getUsers = async () => {
  const response = await api.get("/users");

  return response.data;
};
```

Axios هنا بيجيب البيانات.

لكن TanStack Query يهتم بـ:

```text
Cache
Loading
Error
Refetch
Stale Data
Synchronization
```

---

# مثال يوضح الفرق

بدون TanStack Query:

```text
Component
   ↓
useEffect
   ↓
Axios
   ↓
Backend
```

وغالبًا أنت هتبدأ تكتب:

```text
loading
error
data
useEffect
refetch
```

وتتعامل مع الـ caching بنفسك.

---

باستخدام TanStack Query:

```text
Component
   ↓
useQuery
   ↓
API Function
   ↓
Axios
   ↓
Backend
```

TanStack Query يدير جزء كبير من الـ Server State lifecycle.

---

# ⚠️ أهم فرق تحفظه

```text
Client State
↓
State owned by the application

Server State
↓
State owned by the server
```

مثال:

```text
isModalOpen
→ Client State

users
→ Server State
```

و:

```text
selectedProductId
→ Client State

products
→ Server State
```

---

# طيب Zustand وTanStack Query؟

دي نقطة مهمة لأنك درست Zustand قبل كده.

مش المفروض تقول:

```text
Zustand vs TanStack Query
```

بشكل مطلق.

الأصح:

```text
Zustand
↓
Client State

TanStack Query
↓
Server State
```

مثلاً:

```text
Zustand
→ sidebarOpen
→ selectedTheme
→ filters

TanStack Query
→ users
→ products
→ orders
```

وممكن الاتنين يعيشوا في نفس المشروع.

---

# 🧠 الخلاصة

```text
                    State
                      │
             ┌────────┴────────┐
             ↓                 ↓
       Client State       Server State
             ↓                 ↓
          Zustand        TanStack Query
          useState
```

والـ Axios يظل مسؤولًا عن:

```text
HTTP
 ↓
Request / Response
```

---