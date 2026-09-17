# 0.7 Dependency Direction

خلينا نبدأ من معنى كلمة **Dependency**.

لو عندي:

```ts
import { getUser } from "@/features/users";
```

يبقى الملف الحالي **يعتمد على** `getUser`.

يعني:

```text
A
↓
B
```

معناها:

> **A depends on B**

---

# 1. المشكلة

تخيل إن عندنا:

```text
Component
    ↓
API
    ↓
Axios
```

ده اتجاه منطقي.

الـComponent محتاج الـAPI.

والـAPI محتاجة Axios.

```text
UI
 ↓
Application/API
 ↓
Infrastructure
```

لكن لو حصل العكس:

```text
Axios
   ↓
Component
```

ده غير منطقي.

لأن أداة HTTP المفروض متعرفش حاجة عن الـUI.

---

# 2. قاعدة أساسية

> **Dependency Direction لازم يكون له اتجاه واضح، وما نعملش Circular Dependencies بدون داعٍ.**

مثلاً:

```text
users component
       ↓
users API
       ↓
axios
```

ممتاز.

لكن:

```text
users API
       ↓
users component
```

غالبًا تصميم سيئ.

ليه؟

لأن الـAPI Layer أصبحت تعتمد على الـUI.

---

# 3. مثال من مشروعنا

عندنا:

```text
features/
└── users/
    ├── components/
    │   └── UserCard.tsx
    ├── api/
    │   └── userApi.ts
    └── types/
        └── user.types.ts
```

ممكن يكون:

```text
UserCard
   ↓
useUser
   ↓
getUser
   ↓
Axios
```

الاتجاه:

```text
UI
 ↓
Hook
 ↓
API
 ↓
HTTP Client
```

كل طبقة تعتمد على اللي تحتها.

---

# 4. ليه الاتجاه مهم؟

تخيل:

```text
API
 ↓
Component
```

الـAPI أصبحت تعرف:

```tsx
<UserCard />
```

طيب لو بكرة استخدمت نفس الـAPI في:

```text
Mobile App
CLI
Node.js
Another UI
```

هتلاقي مشكلة.

لأن الـAPI أصبحت مربوطة بالـReact UI.

لكن لو:

```text
Component
 ↓
API
```

فالـAPI مستقلة عن الـUI.

وده أفضل.

---

# 5. Circular Dependency

دي مهمة جدًا.

مثلاً:

```text
A → B
B → A
```

يعني:

```text
A
↓
B
↩
A
```

دي **Circular Dependency**.

مثال:

```ts id="rj6d3f"
// userApi.ts
import { UserCard } from "../components/UserCard";
```

وفي نفس الوقت:

```tsx id="q4n8ps"
// UserCard.tsx
import { getUser } from "../api/userApi";
```

بقى:

```text
userApi
   ↓
UserCard
   ↓
userApi
   ↓
UserCard
```

وده تصميم لازم نتجنبه غالبًا.

---

# 6. فين يدخل `Shared`؟

دي نقطة مهمة جدًا.

عندنا:

```text
shared/
└── components/
    └── Button.tsx
```

والـUsers تستخدم:

```text
users
 ↓
shared/Button
```

والـProducts تستخدم:

```text
products
 ↓
shared/Button
```

ده طبيعي:

```text
       Users
         ↓
      Shared
         ↑
      Products
```

لكن المفروض `Shared` ما يبقاش معتمد على Users.

يعني تجنب:

```text
shared
  ↓
users
```

لأن كده الـShared لم تعد Shared فعليًا.

---

# 7. قاعدة مهمة جدًا

خلينا نرسمها:

```text
features
    ↓
shared
```

ممكن.

لكن:

```text
shared
    ↓
features
```

غالبًا **لا**.

ليه؟

لأن `shared` المفروض تكون عامة.

لو `shared` تعتمد على `users`، بقت مرتبطة بالـUsers.

---

# 8. مثال كامل

تصميم جيد:

```text
src/
├── features/
│   ├── users/
│   │   ├── components/
│   │   ├── api/
│   │   └── types/
│   │
│   └── products/
│       ├── components/
│       ├── api/
│       └── types/
│
└── shared/
    ├── components/
    ├── hooks/
    └── utils/
```

والاعتماد:

```text
Users ──────┐
            ↓
          Shared

Products ───┘
```

مثلاً:

```text
UserCard
   ↓
Button
   ↓
Shared
```

ده منطقي.

---

# 9. لكن هل Feature ممكن تعتمد على Feature أخرى؟

**أيوه، ممكن.**

لكن لازم يكون الاعتماد مقصود وواضح.

مثلاً:

```text
orders
   ↓
users
```

لو الـOrders محتاجة بيانات User.

ده ممكن يكون منطقي.

لكن لو أصبح:

```text
users → orders
orders → users
```

بدأنا نعمل:

```text
Circular Dependency
```

وهنا لازم نراجع التصميم.

---

# 10. Dependency Direction مش مجرد Imports

ودي أهم نقطة في الدرس.

لما نقول:

> Dependency Direction

مش بنتكلم فقط عن:

```ts
import ...
```

إحنا بنتكلم عن **العلاقة المعمارية**.

مثلاً:

```text
Component
   ↓
API
```

معناها:

> الـComponent يعتمد معماريًا على الـAPI abstraction.

حتى لو الـimplementation الداخلية مختلفة.

---

# 🧠 الصورة النهائية لـ Section 0

أنت بدأت من:

```text
HTTP
```

وفهمت:

```text
Request
Response
```

وبعدين:

```text
Separation of Concerns
```

وبعدين:

```text
Coupling
Cohesion
```

وبعدين:

```text
Feature-Based Architecture
```

وبعدين:

```text
Shared vs Feature
```

وبعدين:

```text
Public vs Internal API
```

والآن:

```text
Dependency Direction
```

كلهم بيبنوا فوق بعض:

```text
             Architecture
                  │
       ┌──────────┴──────────┐
       ↓                     ↓
 Separation              Dependency
 of Concerns               Direction
       │                     │
       ↓                     ↓
High Cohesion          Low Coupling
       │                     │
       └──────────┬──────────┘
                  ↓
       Feature-Based Architecture
                  │
          ┌───────┴───────┐
          ↓               ↓
       Feature          Shared
          │
          ↓
      Public API
          │
          ↓
       Internal
```