# 0.6 Public vs Internal API

هنا كلمة **API** مش شرط معناها HTTP API.

إحنا بنتكلم عن:

> **الواجهة التي تسمح لجزء من الكود بالتعامل مع جزء آخر.**

مثلاً عندنا Feature:

```text id="m8h3kx"
features/
└── users/
    ├── api/
    │   └── userApi.ts
    ├── components/
    │   └── UserCard.tsx
    ├── hooks/
    │   └── useUser.ts
    └── types/
        └── user.types.ts
```

مش معنى إن كل الملفات موجودة هنا إن أي Feature تقدر تدخل وتستخدم أي function جواها.

---

# 1. Internal

**Internal = شيء مخصص للاستخدام داخل الـFeature نفسها.**

مثلاً:

```ts id="0mx3es"
function buildUserQuery() {
  // تفاصيل داخلية
}
```

لو دي مجرد implementation detail للـUsers Feature، مش محتاج باقي التطبيق يستخدمها.

إذن:

```text id="v2s9dc"
Internal
```

---

# 2. Public

**Public = الشيء الذي نسمح لأجزاء أخرى من التطبيق باستخدامه.**

مثلاً:

```ts id="i0ahpv"
export const getUser = async (id: number) => {
  // ...
};
```

ممكن نقرر إن `getUser` جزء من الـPublic API الخاصة بالـUsers Feature.

يعني Feature أخرى ممكن تستخدم:

```ts id="q2q2hi"
import { getUser } from "@/features/users";
```

بدل:

```ts id="99z1q8"
import { getUser } from "@/features/users/api/userApi";
```

وده فرق مهم.

---

# 3. ليه مش نستورد من أي مكان؟

تخيل إن عندك:

```text id="8u0w42"
features/users/
├── api/
│   ├── userApi.ts
│   └── userApi.helpers.ts
├── hooks/
│   ├── useUser.ts
│   └── useUser.internal.ts
└── ...
```

لو كل Feature تقدر تعمل:

```ts id="lq2h4m"
import { something }
from "@/features/users/api/userApi.helpers";
```

أصبح عندنا مشكلة.

لأن `userApi.helpers.ts` كان **Internal**.

دلوقتي:

```text id="6c8vpr"
users
 ↑
orders
 ↑
dashboard
 ↑
profile
```

كلهم بقوا يعتمدوا على تفاصيل داخلية.

ولو غيرت الملف الداخلي:

```text id="q1y9jf"
userApi.helpers.ts
```

ممكن تكسر Features كثيرة.

---

# 4. Public API كـ "باب"

فكر فيها كده:

```text id="3j6m2f"
        USERS FEATURE

     ┌─────────────────┐
     │                 │
     │  internal code  │
     │                 │
     │  api helpers    │
     │  private logic  │
     │                 │
     └────────┬────────┘
              │
              ↓
        Public API
              │
              ↓
       باقي التطبيق
```

الـPublic API هو **الباب الرسمي** للدخول للـFeature.

---

# 5. Barrel / Index File

هنا يظهر مفهوم مهم اسمه **Barrel File**.

مثلاً:

```text id="8x7c8b"
features/
└── users/
    ├── api/
    │   └── userApi.ts
    └── index.ts
```

في:

```ts id="k6tq6f"
export { getUser } from "./api/userApi";
```

دلوقتي بدل:

```ts id="5w4n9q"
import { getUser }
from "@/features/users/api/userApi";
```

نستخدم:

```ts id="4z5k0c"
import { getUser }
from "@/features/users";
```

الـ`index.ts` هنا بيحدد:

> إيه اللي أنا سامح لك تستخدمه من الـUsers Feature؟

---

# 6. ودي فايدتها المعمارية

افترض أن implementation تغيرت.

اليوم:

```text id="3evp5a"
api/userApi.ts
```

بكرة غيرتها إلى:

```text id="2h8z9c"
api/users.api.ts
```

لو باقي المشروع بيستورد مباشرة:

```ts id="s9g5z4"
from "@/features/users/api/userApi"
```

هتعدل imports في أماكن كثيرة.

لكن لو كلهم يستخدموا:

```ts id="3w8r5p"
from "@/features/users"
```

تغير الـimplementation داخليًا، والـPublic API تفضل كما هي.

```text id="1xj4v7"
Outside
   ↓
features/users
   ↓
internal implementation
```

دي فكرة قوية جدًا.

---

# 7. Public API لا تعني Export كل حاجة

مثلاً:

```ts id="k5o9j1"
// userApi.ts

export const getUser = ...
export const createUser = ...

const buildQuery = ...
const transformUser = ...
```

ممكن الـPublic API تكون:

```ts id="8p0f6u"
// index.ts

export {
  getUser,
  createUser,
};
```

ولا نصدر:

```ts id="n8m0v2"
buildQuery
transformUser
```

لأنهم Internal implementation details.

---

# 8. مثال قريب من مشروعك

تخيل Feature:

```text id="9p4h6x"
features/
└── users/
    ├── api/
    │   └── userApi.ts
    ├── components/
    │   └── UserCard.tsx
    ├── hooks/
    │   └── useUser.ts
    ├── types/
    │   └── user.types.ts
    └── index.ts
```

ممكن نقرر أن الـPublic API هي:

```ts id="5q6x2a"
export {
  getUser,
  useUser,
  UserCard,
};
```

وباقي التفاصيل تفضل Internal.

---

# 🧠 الخلاصة

احفظها كده:

```text id="s0j9qf"
Public API
= الباب الرسمي للـFeature

Internal
= تفاصيل التنفيذ داخل الـFeature
```

والهدف:

```text id="x6z1y8"
Hide implementation details
        ↓
Reduce coupling
        ↓
Make changes safer
        ↓
Clear architecture
```

وده مرتبط مباشرة بالـCoupling اللي أخدناه.

كل ما الناس تعتمد على **Public Contract** بدل التفاصيل الداخلية:

```text id="x7j2qk"
Lower Coupling ✅
```

---

## 🎯 اختبار

عندك:

```text id="w8g3qp"
features/users/
├── api/
│   ├── userApi.ts
│   └── userApi.helpers.ts
│
├── hooks/
│   └── useUser.ts
│
└── index.ts
```

وفي `index.ts`:

```ts id="x3f6zq"
export { getUser } from "./api/userApi";
export { useUser } from "./hooks/useUser";
```

وفي `userApi.ts`:

```ts id="a7j9wq"
export const getUser = async () => {
  // ...
};

const buildUserUrl = () => {
  // ...
};
```

**السؤال:**

1. هل `getUser` Public أم Internal؟
2. هل `buildUserUrl` Public أم Internal؟
3. هل `useUser` Public أم Internal؟
4. لماذا لا نريد أن باقي الـFeatures تستورد `buildUserUrl` مباشرة؟

**الخطوة التالية: جاوب على الأربعة، وبعدها ندخل آخر درس في Section 0: `Dependency Direction`.**
<details>
<summary>جاوب انت</summary>
user
جاوب انت
ChatGPT
**راجع الإجابات الأربع في دقيقتين، وبعدها نكون جاهزين لآخر درس في Section 0.**

### 1. `getUser`

```ts
export const getUser = async () => {
  // ...
};
```

وموجود في:

```ts
// index.ts
export { getUser } from "./api/userApi";
```

إذن:

**Public API ✅**

لأننا قررنا أن باقي التطبيق مسموح له يستخدمه.

---

### 2. `buildUserUrl`

```ts
const buildUserUrl = () => {
  // ...
};
```

ومش موجود في `index.ts`.

إذن:

**Internal ✅**

دي implementation detail داخل الـUsers Feature.

---

### 3. `useUser`

موجود في:

```ts
// index.ts
export { useUser } from "./hooks/useUser";
```

إذن:

**Public API ✅**

باقي التطبيق يستطيع استخدامه من خلال الـFeature:

```ts
import { useUser } from "@/features/users";
```

---

### 4. ليه ما نستوردش `buildUserUrl` مباشرة؟

لأننا لو سمحنا:

```ts
import { buildUserUrl }
from "@/features/users/api/userApi";
```

فـFeatures أخرى أصبحت تعتمد على **implementation detail** داخل Users.

مثلاً:

```text id="6m6xkq"
Orders
   ↓
buildUserUrl()
   ↓
Users internal code
```

لو غيرنا طريقة بناء الـURL، ممكن نكسر `Orders`.

لكن لو:

```text id="8x1r9p"
Orders
   ↓
getUser()
   ↓
Users internal implementation
```

فالـOrders تعتمد على **Public Contract** فقط.

وده يقلل الـCoupling.

---

# 🧠 الصورة الكبيرة

أنت الآن عندك 3 أفكار مرتبطة ببعض:

```text id="5r8k2w"
Separation of Concerns
        ↓
كل مسؤولية في مكانها

High Cohesion
        ↓
الأشياء المرتبطة معًا تكون معًا

Low Coupling
        ↓
الأجزاء لا تعتمد بشدة على تفاصيل بعضها
```

ثم:

```text id="3q7v1m"
Feature-Based Architecture
        ↓
نجمع الـDomain الخاص بالـFeature

Public API
        ↓
نحدد الباب الرسمي للتعامل معها

Internal
        ↓
نخفي تفاصيل التنفيذ
```

وده مستوى مهم جدًا قبل ما ندخل في **Dependency Direction**.

## 🗺️ Roadmap

```text id="9s7k2p"
SECTION 0: Architecture Foundation 🟢
  ├─ 0.1 HTTP Architecture           ✅
  ├─ 0.2 Separation of Concerns      ✅
  ├─ 0.3 Coupling & Cohesion         ✅
  ├─ 0.4 Feature-Based Architecture  ✅
  ├─ 0.5 Shared vs Feature           ✅
  ├─ 0.6 Public vs Internal API      ✅
  └─ 0.7 Dependency Direction        🟢 ← التالي
```

**الخطوة التالية: نبدأ `0.7 Dependency Direction` لمدة حوالي 20 دقيقة، وهو آخر درس في Section 0.**
</details>
جاوب انت
ChatGPT
**راجع الإجابات الأربع في دقيقتين، وبعدها نكون جاهزين لآخر درس في Section 0.**

### 1. `getUser`

```ts
export const getUser = async () => {
  // ...
};
```

وموجود في:

```ts
// index.ts
export { getUser } from "./api/userApi";
```

إذن:

**Public API ✅**

لأننا قررنا أن باقي التطبيق مسموح له يستخدمه.

---

### 2. `buildUserUrl`

```ts
const buildUserUrl = () => {
  // ...
};
```

ومش موجود في `index.ts`.

إذن:

**Internal ✅**

دي implementation detail داخل الـUsers Feature.

---

### 3. `useUser`

موجود في:

```ts
// index.ts
export { useUser } from "./hooks/useUser";
```

إذن:

**Public API ✅**

باقي التطبيق يستطيع استخدامه من خلال الـFeature:

```ts
import { useUser } from "@/features/users";
```

---

### 4. ليه ما نستوردش `buildUserUrl` مباشرة؟

لأننا لو سمحنا:

```ts
import { buildUserUrl }
from "@/features/users/api/userApi";
```

فـFeatures أخرى أصبحت تعتمد على **implementation detail** داخل Users.

مثلاً:

```text id="6m6xkq"
Orders
   ↓
buildUserUrl()
   ↓
Users internal code
```

لو غيرنا طريقة بناء الـURL، ممكن نكسر `Orders`.

لكن لو:

```text id="8x1r9p"
Orders
   ↓
getUser()
   ↓
Users internal implementation
```

فالـOrders تعتمد على **Public Contract** فقط.

وده يقلل الـCoupling.

---

# 🧠 الصورة الكبيرة

أنت الآن عندك 3 أفكار مرتبطة ببعض:

```text id="5r8k2w"
Separation of Concerns
        ↓
كل مسؤولية في مكانها

High Cohesion
        ↓
الأشياء المرتبطة معًا تكون معًا

Low Coupling
        ↓
الأجزاء لا تعتمد بشدة على تفاصيل بعضها
```

ثم:

```text id="3q7v1m"
Feature-Based Architecture
        ↓
نجمع الـDomain الخاص بالـFeature

Public API
        ↓
نحدد الباب الرسمي للتعامل معها

Internal
        ↓
نخفي تفاصيل التنفيذ
```

وده مستوى مهم جدًا قبل ما ندخل في **Dependency Direction**.

## 🗺️ Roadmap

```text id="9s7k2p"
SECTION 0: Architecture Foundation 🟢
  ├─ 0.1 HTTP Architecture           ✅
  ├─ 0.2 Separation of Concerns      ✅
  ├─ 0.3 Coupling & Cohesion         ✅
  ├─ 0.4 Feature-Based Architecture  ✅
  ├─ 0.5 Shared vs Feature           ✅
  ├─ 0.6 Public vs Internal API      ✅
  └─ 0.7 Dependency Direction        🟢 ← التالي
```

**الخطوة التالية: نبدأ `0.7 Dependency Direction` لمدة حوالي 20 دقيقة، وهو آخر درس في Section 0.**
<details>
<summary>Dependency Direction</summary>
user
Dependency Direction
ChatGPT
**ابدأ بـ `0.7 Dependency Direction` الآن، ومدته حوالي 20 دقيقة، لأنه آخر مفهوم في Section 0 وهو اللي هيحدد مين يعتمد على مين داخل الـArchitecture.**

## 🗺️ Roadmap

```text
SECTION 0: Architecture Foundation 🟢
  ├─ 0.1 HTTP Architecture            ✅
  ├─ 0.2 Separation of Concerns       ✅
  ├─ 0.3 Coupling & Cohesion          ✅
  ├─ 0.4 Feature-Based Architecture  ✅
  ├─ 0.5 Shared vs Feature            ✅
  ├─ 0.6 Public vs Internal API       ✅
  └─ 0.7 Dependency Direction         🟢 ← نحن هنا
```
</details>
Dependency Direction
ChatGPT
**ابدأ بـ `0.7 Dependency Direction` الآن، ومدته حوالي 20 دقيقة، لأنه آخر مفهوم في Section 0 وهو اللي هيحدد مين يعتمد على مين داخل الـArchitecture.**

## 🗺️ Roadmap

```text
SECTION 0: Architecture Foundation 🟢
  ├─ 0.1 HTTP Architecture            ✅
  ├─ 0.2 Separation of Concerns       ✅
  ├─ 0.3 Coupling & Cohesion          ✅
  ├─ 0.4 Feature-Based Architecture  ✅
  ├─ 0.5 Shared vs Feature            ✅
  ├─ 0.6 Public vs Internal API       ✅
  └─ 0.7 Dependency Direction         🟢 ← نحن هنا
```