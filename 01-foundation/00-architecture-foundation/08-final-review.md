# 🎯 اختبار Section 0

عندك:

```text
src/
├── features/
│   ├── users/
│   └── products/
│
└── shared/
```

وعندك العلاقات التالية:

```text
A: users → shared
B: shared → users
C: products → users
D: users → products
E: users → products → users
```

عايزك تحدد:

**أيهم طبيعي؟ وأيهم مشكلة؟**

مش محتاج تشرح كتير، اكتب مثلاً:

```text
A ✅
B ❌
...
```

**الخطوة التالية: جاوب على A إلى E، وبعدها نعمل Final Review كامل لـSection 0 قبل الانتقال إلى Section 4.**
<details>
<summary>أي وسي صح والباقي غلط.</summary>
user
أي وسي صح والباقي غلط.
ChatGPT
**صح إن `A` صحيحة، لكن إجابتك محتاجة تصحيح في نقطة مهمة: `C` ممكن تكون صحيحة أيضًا، ومدتها أقل من دقيقتين.**

خلينا نمسكهم واحدة واحدة:

```text
A: users → shared       ✅
```

طبيعي جدًا. الـFeature تستخدم حاجة عامة من `shared`.

```text
B: shared → users       ❌
```

غالبًا غلط، لأن `shared` المفروض تفضل مستقلة عن الـFeatures.

```text
C: products → users     ✅
```

**ممكن تكون صحيحة.**

لو الـProducts تحتاج فعلًا بيانات أو Public API من Users، فده dependency طبيعي:

```text
products
   ↓
users
```

لكن لازم يكون مقصود، ومش نعمله لمجرد إننا نقدر.

```text
D: users → products     ✅/⚠️
```

برضه **ممكن تكون صحيحة** لو الـUsers تحتاج حاجة من Products.

المشكلة مش في الاتجاه نفسه، المشكلة لو العلاقة غير منطقية أو سببت coupling عالي.

```text
E: users → products → users   ❌
```

دي المشكلة الواضحة، لأنها:

```text
users
  ↓
products
  ↓
users
```

يعني **Circular Dependency**.

### إذن الإجابة الأدق:

```text
A ✅
B ❌
C ✅ ممكن
D ✅ ممكن
E ❌
```

والنقطة اللي لازم تثبت عندك:

> **مش كل Feature-to-Feature dependency غلط. المشكلة الأساسية هي dependency غير ضرورية، أو circular dependency، أو اعتماد على تفاصيل داخلية.**

## 🗺️ Roadmap

```text
SECTION 0: Architecture Foundation
  ├─ 0.1 HTTP Architecture           ✅
  ├─ 0.2 Separation of Concerns      ✅
  ├─ 0.3 Coupling & Cohesion         ✅
  ├─ 0.4 Feature-Based Architecture  ✅
  ├─ 0.5 Shared vs Feature           ✅
  ├─ 0.6 Public vs Internal API      ✅
  └─ 0.7 Dependency Direction        ✅
```

**الخطوة التالية: نعمل الآن Final Review لـSection 0 في 7 أسئلة، سؤالًا بعد سؤال، قبل ما نفتح Section 4 Interceptors.**
</details>
أي وسي صح والباقي غلط.
ChatGPT
**صح إن `A` صحيحة، لكن إجابتك محتاجة تصحيح في نقطة مهمة: `C` ممكن تكون صحيحة أيضًا، ومدتها أقل من دقيقتين.**

خلينا نمسكهم واحدة واحدة:

```text
A: users → shared       ✅
```

طبيعي جدًا. الـFeature تستخدم حاجة عامة من `shared`.

```text
B: shared → users       ❌
```

غالبًا غلط، لأن `shared` المفروض تفضل مستقلة عن الـFeatures.

```text
C: products → users     ✅
```

**ممكن تكون صحيحة.**

لو الـProducts تحتاج فعلًا بيانات أو Public API من Users، فده dependency طبيعي:

```text
products
   ↓
users
```

لكن لازم يكون مقصود، ومش نعمله لمجرد إننا نقدر.

```text
D: users → products     ✅/⚠️
```

برضه **ممكن تكون صحيحة** لو الـUsers تحتاج حاجة من Products.

المشكلة مش في الاتجاه نفسه، المشكلة لو العلاقة غير منطقية أو سببت coupling عالي.

```text
E: users → products → users   ❌
```

دي المشكلة الواضحة، لأنها:

```text
users
  ↓
products
  ↓
users
```

يعني **Circular Dependency**.

### إذن الإجابة الأدق:

```text
A ✅
B ❌
C ✅ ممكن
D ✅ ممكن
E ❌
```

والنقطة اللي لازم تثبت عندك:

> **مش كل Feature-to-Feature dependency غلط. المشكلة الأساسية هي dependency غير ضرورية، أو circular dependency، أو اعتماد على تفاصيل داخلية.**

## 🗺️ Roadmap

```text
SECTION 0: Architecture Foundation
  ├─ 0.1 HTTP Architecture           ✅
  ├─ 0.2 Separation of Concerns      ✅
  ├─ 0.3 Coupling & Cohesion         ✅
  ├─ 0.4 Feature-Based Architecture  ✅
  ├─ 0.5 Shared vs Feature           ✅
  ├─ 0.6 Public vs Internal API      ✅
  └─ 0.7 Dependency Direction        ✅
```

**الخطوة التالية: نعمل الآن Final Review لـSection 0 في 7 أسئلة، سؤالًا بعد سؤال، قبل ما نفتح Section 4 Interceptors.**
<details>
<summary>كمل</summary>
user
كمل
ChatGPT
**ابدأ Final Review لـSection 0 الآن، ومدته حوالي 15 دقيقة، وهنقيس الفهم الحقيقي للـArchitecture قبل دخول `Interceptors`.**

## 🗺️ Roadmap

```text
SECTION 0: Architecture Foundation  🟢 FINAL REVIEW
  ├─ 0.1 HTTP Architecture           ✅
  ├─ 0.2 Separation of Concerns      ✅
  ├─ 0.3 Coupling & Cohesion         ✅
  ├─ 0.4 Feature-Based Architecture  ✅
  ├─ 0.5 Shared vs Feature           ✅
  ├─ 0.6 Public vs Internal API       ✅
  └─ 0.7 Dependency Direction         ✅

SECTION 4: Interceptors               ⬜
```

## السؤال 1/7

عندك Component:

```tsx
function UsersPage() {
  const response = await axios.get("/users");

  const users = response.data.payload;

  const activeUsers = users.filter(
    user => user.active
  );

  return (
    <div>
      {activeUsers.map(user => (
        <div>{user.name}</div>
      ))}
    </div>
  );
}
```

عايز منك تحدد **3 Concerns مختلفة** موجودة هنا.

اكتب:

```text
1. ...
2. ...
3. ...
```

**الخطوة التالية: جاوب السؤال 1 فقط، وبعد تصحيحه ننتقل للسؤال 2.**
</details>
كمل
ChatGPT
**ابدأ Final Review لـSection 0 الآن، ومدته حوالي 15 دقيقة، وهنقيس الفهم الحقيقي للـArchitecture قبل دخول `Interceptors`.**

## 🗺️ Roadmap

```text
SECTION 0: Architecture Foundation  🟢 FINAL REVIEW
  ├─ 0.1 HTTP Architecture           ✅
  ├─ 0.2 Separation of Concerns      ✅
  ├─ 0.3 Coupling & Cohesion         ✅
  ├─ 0.4 Feature-Based Architecture  ✅
  ├─ 0.5 Shared vs Feature           ✅
  ├─ 0.6 Public vs Internal API       ✅
  └─ 0.7 Dependency Direction         ✅

SECTION 4: Interceptors               ⬜
```

## السؤال 1/7

عندك Component:

```tsx
function UsersPage() {
  const response = await axios.get("/users");

  const users = response.data.payload;

  const activeUsers = users.filter(
    user => user.active
  );

  return (
    <div>
      {activeUsers.map(user => (
        <div>{user.name}</div>
      ))}
    </div>
  );
}
```

عايز منك تحدد **3 Concerns مختلفة** موجودة هنا.

اكتب:

```text
1. ...
2. ...
3. ...
```

**الخطوة التالية: جاوب السؤال 1 فقط، وبعد تصحيحه ننتقل للسؤال 2.**
<details>
<summary>المكونات اللي هو عندك الأكسس روبوتيون، عندك اللي هو الفلتر اللي هي بتبقى عبارة عن تاجز.</summary>
user
المكونات اللي هو عندك الأكسس روبوتيون، عندك اللي هو الفلتر اللي هي بتبقى عبارة عن تاجز.
ChatGPT
**صح، أنت شايف الـ3 أجزاء، لكن محتاج تسميهم بالمصطلحات المعمارية بدقة، وتقييمك 8.5/10.**

الكود فيه:

```text id="j9c4z2"
1. UI
2. Data Fetching
3. Business Logic
```

### 1. UI ✅

ده:

```tsx id="f8x3qw"
return (
  <div>
    {activeUsers.map(user => (
      <div>{user.name}</div>
    ))}
  </div>
);
```

مسؤولية الـUI.

---

### 2. Data Fetching ✅

أنت قلت:

> Axios

بالضبط:

```ts id="z8k5n1"
await axios.get("/users");
```

ده **Data Fetching / HTTP Communication**.

---

### 3. Business Logic 🟡

أنت أشرت للـ`filter`، وده صحيح:

```ts id="x2w9kd"
const activeUsers = users.filter(
  user => user.active
);
```

لكن الـ`filter` نفسه مش هو الـBusiness Logic.

**القاعدة التي تقول إننا نريد Active Users هي الـBusiness Logic.**

يعني:

```text id="d1s5vq"
"اعرض المستخدمين النشطين"
        ↓
Business Rule
```

والـ`filter()` مجرد طريقة برمجية لتنفيذ القاعدة.

---

## 🧠 دي نقطة مهمة جدًا

لو سألتك في Interview:

> What is the business logic here?

ما تقولش:

> `filter`

قول:

> **The rule that only active users should be displayed.**

أما `filter()` فهو implementation لهذه القاعدة.

---

## السؤال 2/7

عندك:

```text id="1f6q7s"
features/
├── users/
├── products/
└── orders/

shared/
├── components/
├── hooks/
└── utils/
```

وعندك:

```text id="g7x3r2"
UserCard
Button
getUser
formatCurrency
useDebounce
ProductTable
```

**صنّفهم: Feature أم Shared، ولو Feature قول تبع أنهي Feature.**

اكتب:

```text id="8j2p4v"
UserCard →
Button →
getUser →
formatCurrency →
useDebounce →
ProductTable →
```

**الخطوة التالية: جاوب السؤال 2 فقط.**
</details>
المكونات اللي هو عندك الأكسس روبوتيون، عندك اللي هو الفلتر اللي هي بتبقى عبارة عن تاجز.
ChatGPT
**صح، أنت شايف الـ3 أجزاء، لكن محتاج تسميهم بالمصطلحات المعمارية بدقة، وتقييمك 8.5/10.**

الكود فيه:

```text id="j9c4z2"
1. UI
2. Data Fetching
3. Business Logic
```

### 1. UI ✅

ده:

```tsx id="f8x3qw"
return (
  <div>
    {activeUsers.map(user => (
      <div>{user.name}</div>
    ))}
  </div>
);
```

مسؤولية الـUI.

---

### 2. Data Fetching ✅

أنت قلت:

> Axios

بالضبط:

```ts id="z8k5n1"
await axios.get("/users");
```

ده **Data Fetching / HTTP Communication**.

---

### 3. Business Logic 🟡

أنت أشرت للـ`filter`، وده صحيح:

```ts id="x2w9kd"
const activeUsers = users.filter(
  user => user.active
);
```

لكن الـ`filter` نفسه مش هو الـBusiness Logic.

**القاعدة التي تقول إننا نريد Active Users هي الـBusiness Logic.**

يعني:

```text id="d1s5vq"
"اعرض المستخدمين النشطين"
        ↓
Business Rule
```

والـ`filter()` مجرد طريقة برمجية لتنفيذ القاعدة.

---

## 🧠 دي نقطة مهمة جدًا

لو سألتك في Interview:

> What is the business logic here?

ما تقولش:

> `filter`

قول:

> **The rule that only active users should be displayed.**

أما `filter()` فهو implementation لهذه القاعدة.

---

## السؤال 2/7

عندك:

```text id="1f6q7s"
features/
├── users/
├── products/
└── orders/

shared/
├── components/
├── hooks/
└── utils/
```

وعندك:

```text id="g7x3r2"
UserCard
Button
getUser
formatCurrency
useDebounce
ProductTable
```

**صنّفهم: Feature أم Shared، ولو Feature قول تبع أنهي Feature.**

اكتب:

```text id="8j2p4v"
UserCard →
Button →
getUser →
formatCurrency →
useDebounce →
ProductTable →
```

**الخطوة التالية: جاوب السؤال 2 فقط.**
<details>
<summary>Future component. Future shared component. Future UI. Future user UI. Future shared. What currency. Shared views devices. Shared.</summary>
user
Future component. Future shared component. Future UI. Future user UI. Future shared. What currency. Shared views devices. Shared.
ChatGPT
**أنت قريب جدًا، لكن حصل خلط في بعض المصطلحات، وتقييم السؤال 2 هو 7.5/10.**

خلينا نصححهم بالترتيب:

```text
UserCard
→ Feature / users / components ✅
```

```text
Button
→ Shared / components ✅
```

```text
getUser
→ Feature / users / api ✅
```

أنت قلت إنها UI، لكن `getUser()` **مش UI**. دي API function بتجيب بيانات.

```text
formatCurrency
→ Shared / utils ✅
```

لأنها عامة ومش مرتبطة بـUser أو Product تحديدًا.

```text
useDebounce
→ Shared / hooks ✅
```

لأنها Hook عامة ممكن تستخدمها أي Feature.

```text
ProductTable
→ غالبًا Feature / products / components ✅
```

ولو كانت Generic فعلًا:

```tsx
<DataTable data={data} columns={columns} />
```

ومش عارفة أي حاجة عن Products، ممكن تكون:

```text
shared/components
```

### إذن الإجابة النهائية:

```text
UserCard      → Feature / users / components
Button        → Shared / components
getUser       → Feature / users / api
formatCurrency→ Shared / utils
useDebounce   → Shared / hooks
ProductTable  → Feature / products / components
```

### 🧠 ركز في النقطة دي

أنت أحيانًا بتقول:

> "Feature"

وده صح، لكن لازم تحدد **أي نوع من الكود**:

```text
Feature
 ├── components
 ├── api
 ├── hooks
 ├── types
 └── utils
```

يعني:

```text
getUser
→ Feature
→ users
→ api
```

مش:

```text
getUser
→ Feature
→ components
```

لأن `getUser` مش Component.

---

## السؤال 3/7

عندك:

```text
features/
└── users/
    ├── api/
    │   └── userApi.ts
    ├── components/
    │   └── UserCard.tsx
    └── index.ts
```

وفي `index.ts`:

```ts
export { getUser } from "./api/userApi";
export { UserCard } from "./components/UserCard";
```

لكن داخل `userApi.ts` عندك:

```ts
export const getUser = ...
const buildUserUrl = ...
```

**مين Public ومين Internal؟**

جاوب:

```text
getUser →
UserCard →
buildUserUrl →
```

**الخطوة التالية: جاوب السؤال 3 فقط.**
</details>
Future component. Future shared component. Future UI. Future user UI. Future shared. What currency. Shared views devices. Shared.
ChatGPT
**أنت قريب جدًا، لكن حصل خلط في بعض المصطلحات، وتقييم السؤال 2 هو 7.5/10.**

خلينا نصححهم بالترتيب:

```text
UserCard
→ Feature / users / components ✅
```

```text
Button
→ Shared / components ✅
```

```text
getUser
→ Feature / users / api ✅
```

أنت قلت إنها UI، لكن `getUser()` **مش UI**. دي API function بتجيب بيانات.

```text
formatCurrency
→ Shared / utils ✅
```

لأنها عامة ومش مرتبطة بـUser أو Product تحديدًا.

```text
useDebounce
→ Shared / hooks ✅
```

لأنها Hook عامة ممكن تستخدمها أي Feature.

```text
ProductTable
→ غالبًا Feature / products / components ✅
```

ولو كانت Generic فعلًا:

```tsx
<DataTable data={data} columns={columns} />
```

ومش عارفة أي حاجة عن Products، ممكن تكون:

```text
shared/components
```

### إذن الإجابة النهائية:

```text
UserCard      → Feature / users / components
Button        → Shared / components
getUser       → Feature / users / api
formatCurrency→ Shared / utils
useDebounce   → Shared / hooks
ProductTable  → Feature / products / components
```

### 🧠 ركز في النقطة دي

أنت أحيانًا بتقول:

> "Feature"

وده صح، لكن لازم تحدد **أي نوع من الكود**:

```text
Feature
 ├── components
 ├── api
 ├── hooks
 ├── types
 └── utils
```

يعني:

```text
getUser
→ Feature
→ users
→ api
```

مش:

```text
getUser
→ Feature
→ components
```

لأن `getUser` مش Component.

---

## السؤال 3/7

عندك:

```text
features/
└── users/
    ├── api/
    │   └── userApi.ts
    ├── components/
    │   └── UserCard.tsx
    └── index.ts
```

وفي `index.ts`:

```ts
export { getUser } from "./api/userApi";
export { UserCard } from "./components/UserCard";
```

لكن داخل `userApi.ts` عندك:

```ts
export const getUser = ...
const buildUserUrl = ...
```

**مين Public ومين Internal؟**

جاوب:

```text
getUser →
UserCard →
buildUserUrl →
```

**الخطوة التالية: جاوب السؤال 3 فقط.**
<details>
<summary>Elvablik.</summary>
user
Elvablik.
ChatGPT
**صح، المقصود `Public`، لكن خلينا نحدد الثلاثة بدقة، وده ياخد أقل من دقيقة.**

```text id="zj30w1"
getUser
→ Public ✅
```

لأنه متصدر من `index.ts`.

```text id="t8j7a4"
UserCard
→ Public ✅
```

لأنه برضه متصدر من `index.ts`.

```text id="b4q8n9"
buildUserUrl
→ Internal ✅
```

لأنه **مش متصدر**، وهو implementation detail داخل الـAPI.

والـoutside code المفروض يستخدم:

```ts id="q3m8fd"
import { getUser } from "@/features/users";
```

وليس:

```ts id="j8k1z2"
import { buildUserUrl } from "@/features/users/api/userApi";
```

**القاعدة:** الـPublic API هو الباب الرسمي، والـInternal هو تفاصيل التنفيذ اللي وراء الباب.

## 🗺️ Roadmap

```text id="6c3r5p"
SECTION 0: Architecture Foundation
  ├─ 0.1 HTTP Architecture           ✅
  ├─ 0.2 Separation of Concerns      ✅
  ├─ 0.3 Coupling & Cohesion         ✅
  ├─ 0.4 Feature-Based Architecture  ✅
  ├─ 0.5 Shared vs Feature           ✅
  ├─ 0.6 Public vs Internal API       ✅
  └─ 0.7 Dependency Direction        🟢 Final Review
```

**الخطوة التالية: السؤال 4/7: اشرح من فهمك لماذا `buildUserUrl` لا نريد أن تستورده Features أخرى مباشرة؟**
</details>
Elvablik.
ChatGPT
**صح، المقصود `Public`، لكن خلينا نحدد الثلاثة بدقة، وده ياخد أقل من دقيقة.**

```text id="zj30w1"
getUser
→ Public ✅
```

لأنه متصدر من `index.ts`.

```text id="t8j7a4"
UserCard
→ Public ✅
```

لأنه برضه متصدر من `index.ts`.

```text id="b4q8n9"
buildUserUrl
→ Internal ✅
```

لأنه **مش متصدر**، وهو implementation detail داخل الـAPI.

والـoutside code المفروض يستخدم:

```ts id="q3m8fd"
import { getUser } from "@/features/users";
```

وليس:

```ts id="j8k1z2"
import { buildUserUrl } from "@/features/users/api/userApi";
```

**القاعدة:** الـPublic API هو الباب الرسمي، والـInternal هو تفاصيل التنفيذ اللي وراء الباب.

## 🗺️ Roadmap

```text id="6c3r5p"
SECTION 0: Architecture Foundation
  ├─ 0.1 HTTP Architecture           ✅
  ├─ 0.2 Separation of Concerns      ✅
  ├─ 0.3 Coupling & Cohesion         ✅
  ├─ 0.4 Feature-Based Architecture  ✅
  ├─ 0.5 Shared vs Feature           ✅
  ├─ 0.6 Public vs Internal API       ✅
  └─ 0.7 Dependency Direction        🟢 Final Review
```

**الخطوة التالية: السؤال 4/7: اشرح من فهمك لماذا `buildUserUrl` لا نريد أن تستورده Features أخرى مباشرة؟**
<details>
<summary>منو نده.</summary>
user
منو نده.
ChatGPT
**لو قصدك "منه نبدأ؟" أو إنك عايزني أكمل، نكمل مباشرة بالسؤال 4، ومدته أقل من دقيقة.**

### السؤال 4/7

ليه **مش عايزين** Feature تانية تعمل:

```ts
import { buildUserUrl } from "@/features/users/api/userApi";
```

وتستخدمها مباشرة؟

اختار السبب الأقرب:

**A)** لأنها function صغيرة.

**B)** لأنها `Internal` implementation detail، ولو اتغيرت ممكن تكسر Features أخرى.

**C)** لأنها مكتوبة بـTypeScript.

**الخطوة التالية: اختر `A` أو `B` أو `C`.**
</details>
منو نده.
ChatGPT
**لو قصدك "منه نبدأ؟" أو إنك عايزني أكمل، نكمل مباشرة بالسؤال 4، ومدته أقل من دقيقة.**

### السؤال 4/7

ليه **مش عايزين** Feature تانية تعمل:

```ts
import { buildUserUrl } from "@/features/users/api/userApi";
```

وتستخدمها مباشرة؟

اختار السبب الأقرب:

**A)** لأنها function صغيرة.

**B)** لأنها `Internal` implementation detail، ولو اتغيرت ممكن تكسر Features أخرى.

**C)** لأنها مكتوبة بـTypeScript.

**الخطوة التالية: اختر `A` أو `B` أو `C`.**
<details>
<summary>B</summary>
user
B
ChatGPT
**إجابة صحيحة 100%، B هي الإجابة، وتقييمك 10/10.**

لأن:

```ts id="9h3v8p"
buildUserUrl()
```

هي **Internal implementation detail**.

لو Feature أخرى اعتمدت عليها مباشرة:

```text id="qv3x7r"
Orders
   ↓
buildUserUrl()
```

وبعدين غيرنا طريقة بناء الـURL، ممكن نكسر `Orders`.

لكن لو استخدمت الـPublic API:

```text id="k8j2ma"
Orders
   ↓
getUser()
   ↓
Internal implementation
```

نقدر نغير الـimplementation بدون ما نكسر المستهلكين.

---

## السؤال 5/7

عندك:

```text id="v9z3ta"
users → shared
```

و:

```text id="j7x4qp"
shared → users
```

**أي واحد منهم طبيعي معماريًا؟ وليه؟**

**الخطوة التالية: جاوب بجملة واحدة فقط.**
</details>
B
ChatGPT
**إجابة صحيحة 100%، B هي الإجابة، وتقييمك 10/10.**

لأن:

```ts id="9h3v8p"
buildUserUrl()
```

هي **Internal implementation detail**.

لو Feature أخرى اعتمدت عليها مباشرة:

```text id="qv3x7r"
Orders
   ↓
buildUserUrl()
```

وبعدين غيرنا طريقة بناء الـURL، ممكن نكسر `Orders`.

لكن لو استخدمت الـPublic API:

```text id="k8j2ma"
Orders
   ↓
getUser()
   ↓
Internal implementation
```

نقدر نغير الـimplementation بدون ما نكسر المستهلكين.

---

## السؤال 5/7

عندك:

```text id="v9z3ta"
users → shared
```

و:

```text id="j7x4qp"
shared → users
```

**أي واحد منهم طبيعي معماريًا؟ وليه؟**

**الخطوة التالية: جاوب بجملة واحدة فقط.**
<details>
<summary>الأولى.</summary>
user
الأولى.
ChatGPT
**صح، الأولى هي الطبيعية: `users → shared`، وتقييمك 10/10.**

السبب:

```text id="k8n2wf"
users
  ↓
shared
```

الـ`users` Feature ممكن تستخدم حاجة عامة من `shared`.

لكن:

```text id="g4r6yc"
shared
  ↓
users
```

ده غالبًا غلط، لأن `shared` المفروض تكون مستقلة عن الـFeatures.

---

## السؤال 6/7

عندك:

```text id="s2q8md"
users → products
products → users
```

إيه المشكلة هنا؟

**الخطوة التالية: سمّي المشكلة فقط.**
</details>
الأولى.
ChatGPT
**صح، الأولى هي الطبيعية: `users → shared`، وتقييمك 10/10.**

السبب:

```text id="k8n2wf"
users
  ↓
shared
```

الـ`users` Feature ممكن تستخدم حاجة عامة من `shared`.

لكن:

```text id="g4r6yc"
shared
  ↓
users
```

ده غالبًا غلط، لأن `shared` المفروض تكون مستقلة عن الـFeatures.

---

## السؤال 6/7

عندك:

```text id="s2q8md"
users → products
products → users
```

إيه المشكلة هنا؟

**الخطوة التالية: سمّي المشكلة فقط.**
<details>
<summary>مشكلة في الأوراق.</summary>
user
مشكلة في الأوراق.
ChatGPT
**لو قصدك "مشكلة في الـdependencies"، فأنت قريب، لكن الاسم الدقيق هو `Circular Dependency`، وتقييمك 8/10.**

لأن عندنا:

```text id="n7h4qc"
users
  ↓
products
  ↓
users
```

يعني:

```text id="e6s3zj"
A → B
B → A
```

وده اسمه:

**Circular Dependency** 🔴

المشكلة إنها بتخلي الـFeatures مرتبطة ببعض في دائرة، وده يصعّب التعديل والاختبار والصيانة.

## السؤال 7/7

آخر سؤال:

عندك:

```text id="1z8p4k"
UsersPage
   ↓
getUser()
   ↓
Axios
```

ليه الاتجاه ده أفضل من:

```text id="m5v7qx"
Axios
   ↓
UsersPage
```

**الخطوة التالية: اشرحها بكلمتين أو جملة قصيرة من فهمك.**
</details>
مشكلة في الأوراق.
ChatGPT
**لو قصدك "مشكلة في الـdependencies"، فأنت قريب، لكن الاسم الدقيق هو `Circular Dependency`، وتقييمك 8/10.**

لأن عندنا:

```text id="n7h4qc"
users
  ↓
products
  ↓
users
```

يعني:

```text id="e6s3zj"
A → B
B → A
```

وده اسمه:

**Circular Dependency** 🔴

المشكلة إنها بتخلي الـFeatures مرتبطة ببعض في دائرة، وده يصعّب التعديل والاختبار والصيانة.

## السؤال 7/7

آخر سؤال:

عندك:

```text id="1z8p4k"
UsersPage
   ↓
getUser()
   ↓
Axios
```

ليه الاتجاه ده أفضل من:

```text id="m5v7qx"
Axios
   ↓
UsersPage
```

**الخطوة التالية: اشرحها بكلمتين أو جملة قصيرة من فهمك.**
<details>
<summary>يعني... هيحتاج يستخدم...</summary>
user
يعني... هيحتاج يستخدم...
ChatGPT
**كمّل الجملة: `UsersPage` يحتاج `getUser`، لكن `Axios` لا يحتاج يعرف حاجة عن `UsersPage`، ومدتها أقل من دقيقة.**

يعني الاتجاه الطبيعي:

```text id="x8v2qf"
UsersPage
   ↓
getUser()
   ↓
Axios
```

لأن:

- الـ`UsersPage` يحتاج جلب بيانات User.
- `getUser()` مسؤول عن الـAPI.
- `Axios` مجرد HTTP client، ومش المفروض يعرف أي حاجة عن الـUI.

أما:

```text id="p3k7mz"
Axios
   ↓
UsersPage
```

فده معناه إن طبقة الـHTTP أصبحت تعتمد على طبقة الـUI، وده بيعمل **Coupling غير منطقي**.

### 🎯 نتيجة Final Review

أنت فاهم Section 0 بشكل جيد جدًا، خصوصًا:

```text
Separation of Concerns     ✅
Coupling                   ✅
Cohesion                   ✅
Feature-Based Architecture ✅
Shared vs Feature          ✅
Public vs Internal API     ✅
Dependency Direction       ✅
```

**Section 0 مكتملة ✅**

## 🗺️ Roadmap

```text id="7m2x9c"
SECTION 0 Architecture Foundation  ✅
SECTION 1 Axios Fundamentals       ✅
SECTION 2 TypeScript + Axios       ✅
SECTION 3 Error Handling           ✅

SECTION 4 Interceptors             🟢 ← التالي
  ├─ 4.1 What is an Interceptor    ⬜
  ├─ 4.2 Request Interceptor       ⬜
  ├─ 4.3 Response Interceptor      ⬜
  ├─ 4.4 Authentication            ⬜
  ├─ 4.5 401 Handling              ⬜
  └─ 4.6 Token Refresh + Queue     ⬜
```

**الخطوة التالية: نبدأ `4.1 What is an Axios Interceptor` لمدة حوالي 20 دقيقة.**
</details>
يعني... هيحتاج يستخدم...
ChatGPT
**كمّل الجملة: `UsersPage` يحتاج `getUser`، لكن `Axios` لا يحتاج يعرف حاجة عن `UsersPage`، ومدتها أقل من دقيقة.**

يعني الاتجاه الطبيعي:

```text id="x8v2qf"
UsersPage
   ↓
getUser()
   ↓
Axios
```

لأن:

- الـ`UsersPage` يحتاج جلب بيانات User.
- `getUser()` مسؤول عن الـAPI.
- `Axios` مجرد HTTP client، ومش المفروض يعرف أي حاجة عن الـUI.

أما:

```text id="p3k7mz"
Axios
   ↓
UsersPage
```

فده معناه إن طبقة الـHTTP أصبحت تعتمد على طبقة الـUI، وده بيعمل **Coupling غير منطقي**.

### 🎯 نتيجة Final Review

أنت فاهم Section 0 بشكل جيد جدًا، خصوصًا:

```text
Separation of Concerns     ✅
Coupling                   ✅
Cohesion                   ✅
Feature-Based Architecture ✅
Shared vs Feature          ✅
Public vs Internal API     ✅
Dependency Direction       ✅
```

**Section 0 مكتملة ✅**

## 🗺️ Roadmap

```text id="7m2x9c"
SECTION 0 Architecture Foundation  ✅
SECTION 1 Axios Fundamentals       ✅
SECTION 2 TypeScript + Axios       ✅
SECTION 3 Error Handling           ✅

SECTION 4 Interceptors             🟢 ← التالي
  ├─ 4.1 What is an Interceptor    ⬜
  ├─ 4.2 Request Interceptor       ⬜
  ├─ 4.3 Response Interceptor      ⬜
  ├─ 4.4 Authentication            ⬜
  ├─ 4.5 401 Handling              ⬜
  └─ 4.6 Token Refresh + Queue     ⬜
```

**الخطوة التالية: نبدأ `4.1 What is an Axios Interceptor` لمدة حوالي 20 دقيقة.**
<details>
<summary>كمل</summary>
user
كمل
ChatGPT
**ابدأ `4.1 Axios Interceptor` الآن، ومدته حوالي 20 دقيقة، وهنفهم أولًا الفكرة قبل ما نكتب أي `interceptor` للكود.**

## 🗺️ Roadmap

```text
SECTION 0 Architecture Foundation  ✅
SECTION 1 Axios Fundamentals       ✅
SECTION 2 TypeScript + Axios       ✅
SECTION 3 Error Handling            ✅

SECTION 4 Interceptors              🟢
  ├─ 4.1 What is an Interceptor    🟢 ← نحن هنا
  ├─ 4.2 Request Interceptor       ⬜
  ├─ 4.3 Response Interceptor      ⬜
  ├─ 4.4 Authentication            ⬜
  ├─ 4.5 401 Handling              ⬜
  └─ 4.6 Token Refresh + Queue     ⬜
```
</details>
كمل
ChatGPT
**ابدأ `4.1 Axios Interceptor` الآن، ومدته حوالي 20 دقيقة، وهنفهم أولًا الفكرة قبل ما نكتب أي `interceptor` للكود.**

## 🗺️ Roadmap

```text
SECTION 0 Architecture Foundation  ✅
SECTION 1 Axios Fundamentals       ✅
SECTION 2 TypeScript + Axios       ✅
SECTION 3 Error Handling            ✅

SECTION 4 Interceptors              🟢
  ├─ 4.1 What is an Interceptor    🟢 ← نحن هنا
  ├─ 4.2 Request Interceptor       ⬜
  ├─ 4.3 Response Interceptor      ⬜
  ├─ 4.4 Authentication            ⬜
  ├─ 4.5 401 Handling              ⬜
  └─ 4.6 Token Refresh + Queue     ⬜
```