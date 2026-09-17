# 0.5 Shared vs Feature

أول قاعدة:

> **Feature-specific code يعيش داخل الـFeature، والـcode العام القابل لإعادة الاستخدام يعيش في `shared`.**

خلينا نفهمها بدل ما نحفظها.

---

# 1. Feature-specific

عندنا:

```text
features/
└── users/
```

أي حاجة مرتبطة بالـUsers تحديدًا:

```text id="f4ah4r"
users/
├── api/
│   └── userApi.ts
├── components/
│   └── UserCard.tsx
├── hooks/
│   └── useUser.ts
└── types/
    └── user.types.ts
```

دي كلها **User-specific**.

مثلاً:

```tsx id="1n9g0w"
<UserCard user={user} />
```

الـ`UserCard` يعرف حاجة عن User.

إذن:

```text id="8sh8gc"
features/users/components/UserCard.tsx
```

---

# 2. Shared

دلوقتي عندنا:

```tsx id="b6ed51"
<Button>Save</Button>
```

هل الـButton خاص بالـUsers؟

❌ لا.

ممكن تستخدمه في:

```text id="2y0u8q"
Users
Products
Orders
Login
Settings
```

إذن:

```text id="u8p4sq"
shared/
└── components/
    └── Button.tsx
```

---

# 3. مثال مهم جدًا

عندك:

```tsx id="y9z44r"
<UserCard />
```

و:

```tsx id="5l7g4h"
<Button />
```

الفرق:

```text id="gq1p3e"
UserCard
↓
خاص بالـUsers

Button
↓
عام
```

إذن:

```text id="q2yy6s"
features/
└── users/
    └── components/
        └── UserCard.tsx
```

لكن:

```text id="g7j1az"
shared/
└── components/
    └── Button.tsx
```

---

# 4. مش معنى Shared إنه نستخدمه مرتين فقط

دي نقطة مهمة.

ممكن Component تستخدمه مرة واحدة حاليًا، لكن تصميمه **عام بطبيعته**.

مثلاً:

```tsx id="x7d3tr"
<Button>Save</Button>
```

هو مش مرتبط بـUser.

إذن مكانه منطقي في:

```text id="d2ij6n"
shared/components
```

لكن ما تعملش كل حاجة `shared` لمجرد احتمال إنك تستخدمها مستقبلًا.

---

# 5. مثال على خطأ شائع

عندك:

```text id="2d3h5n"
shared/
├── components/
│   ├── UserCard.tsx
│   ├── ProductCard.tsx
│   └── OrderCard.tsx
```

ليه ده غالبًا غلط؟

لأن:

```text id="m0w4p4"
UserCard    → User-specific
ProductCard → Product-specific
OrderCard   → Order-specific
```

يبقى الأفضل:

```text id="s7p6g0"
features/
├── users/
│   └── components/
│       └── UserCard.tsx
│
├── products/
│   └── components/
│       └── ProductCard.tsx
│
└── orders/
    └── components/
        └── OrderCard.tsx
```

---

# 6. Shared مش بس Components

ممكن يكون عندك:

```text id="kq5h7f"
shared/
├── components/
├── hooks/
├── utils/
├── types/
└── constants/
```

مثلاً:

```text id="1qv8by"
shared/
└── utils/
    └── formatCurrency.ts
```

لو:

```ts id="g7mxz1"
formatCurrency(1500)
```

تنفع مع:

```text
Products
Orders
Payments
Invoices
```

فهي عامة.

---

# 7. قاعدة عملية قوية

اسأل السؤال ده:

> **هل الكود ده يعرف حاجة عن Feature معينة؟**

لو نعم:

```text id="e5k0xs"
Feature
```

لو لا، وهو reusable بشكل عام:

```text id="qkqj0a"
Shared
```

مثال:

```ts id="q6gqgc"
getUser()
```

يعرف User:

```text
User
```

إذن:

```text id="1f7b0c"
features/users/api
```

لكن:

```ts id="h4q1jk"
formatCurrency()
```

لا يعرف User ولا Product ولا Order.

إذن:

```text id="h6t4s2"
shared/utils
```

---

# 8. ⚠️ فخ مهم

ممكن واحد يقول:

> "أي حاجة ممكن أستخدمها في Feature تانية أحطها Shared."

مش بالضرورة.

مثلاً:

```text id="k7w3n4"
features/users/
└── components/
    └── UserTable.tsx
```

حتى لو بعد فترة احتجت تستخدم `UserTable` في `AdminUsers`، ده لا يعني بالضرورة إنها أصبحت Shared.

اسأل:

> هل الـUserTable نفسها مرتبطة بالـUser domain؟

نعم.

إذن ممكن تظل داخل:

```text id="9v7h4s"
features/users
```

---

# 🧠 الخلاصة

```text id="3n6t9k"
Feature
= مرتبط بـDomain/Feature معينة

Shared
= عام وغير مرتبط بـFeature معينة
```

مثال:

```text id="5x3g2a"
features/
├── users/
│   ├── api/
│   ├── components/
│   └── types/
│
└── products/
    ├── api/
    ├── components/
    └── types/

shared/
├── components/
│   ├── Button.tsx
│   └── Modal.tsx
│
├── utils/
│   └── formatCurrency.ts
│
└── hooks/
    └── useDebounce.ts
```

وده بيحقق اللي اتعلمناه:

```text id="j5w8m2"
Feature-Based Architecture
        ↓
High Cohesion
        +
Low Coupling
        +
Clear Boundaries
```

## 🎯 اختبار

عندك هذه الأشياء:

```text
1. UserCard
2. Button
3. getUser()
4. formatDate()
5. ProductTable
6. useDebounce()
```

حدد لكل واحد:

```text
Feature
أو
Shared
```

ولو Feature، اكتب تبع أنهي Feature.

**الخطوة التالية: صنّف الـ6 عناصر، مثال: `1. Feature → users`.**
<details>
<summary>الإجابة رقم واحد: الـ use card دي future, future user. بعد كده عندك الـ button دي بتبقى shared ورقم get user برضه نقدر نحطها في الـ future بتاع الـ user جوه الـ component. وعندنا الـ form date دي نقدر نحطها في الـ login أو الـ future بتاع الـ login أو الـ register بتاعنا جوه future يعني future login، بعد كده إيه الـ component. بعد كده الـ product table دي ممكن نحطها في الـ shared لو كان التابل ده موجود عندي في الـ dashboard في كل حتة، أو نحطها في الـ future في الـ product. والـ use dependency دي تقريبًا في الـ shared، أنا ما عارفش إيه تقريبًا في الـ shared.</summary>
user
الإجابة رقم واحد: الـ use card دي future, future user. بعد كده عندك الـ button دي بتبقى shared ورقم get user برضه نقدر نحطها في الـ future بتاع الـ user جوه الـ component. وعندنا الـ form date دي نقدر نحطها في الـ login أو الـ future بتاع الـ login أو الـ register بتاعنا جوه future يعني future login، بعد كده إيه الـ component. بعد كده الـ product table دي ممكن نحطها في الـ shared لو كان التابل ده موجود عندي في الـ dashboard في كل حتة، أو نحطها في الـ future في الـ product. والـ use dependency دي تقريبًا في الـ shared، أنا ما عارفش إيه تقريبًا في الـ shared.
ChatGPT
**إجابتك كويسة جدًا، وتقييمك 8.5/10، وأهم حاجة عندك إنك بدأت تفكر في الـownership بدل الحفظ.**

## تصحيح نقطة نقطة

### 1. `UserCard`

قلت:

> Feature → users

✅ **صحيح 100%**

```text
features/
└── users/
    └── components/
        └── UserCard.tsx
```

لأنها مرتبطة مباشرة بـUser.

---

### 2. `Button`

قلت:

> Shared

✅ **صحيح**

```text
shared/
└── components/
    └── Button.tsx
```

لأنها مش عارفة حاجة عن Users أو Products.

---

### 3. `getUser()`

قلت:

> Feature → users

✅ **صحيح في النتيجة.**

لكن قلت:

> جوه الـcomponent

❌ هنا التصحيح المهم.

مش:

```text
features/users/components/
└── getUser()
```

الأفضل:

```text
features/
└── users/
    └── api/
        └── userApi.ts
```

لأن `getUser()` **API function**، مش UI Component.

---

### 4. `formatDate()`

هنا عندك لخبطة بسيطة.

أنت قلت إنها ممكن تكون خاصة بـLogin/Register.

لكن `formatDate()` بشكل عام:

```ts
formatDate(date)
```

لو بتتعامل مع أي تاريخ في التطبيق، فهي:

```text
Shared
```

مثلاً:

```text
shared/
└── utils/
    └── formatDate.ts
```

لكن لو عندك function مخصوصة بمنطق الـLogin، مثلاً:

```ts
formatLoginAttemptDate()
```

ومرتبطة بالـLogin domain، ساعتها ممكن تكون داخل:

```text
features/auth/
```

**المعيار هو: هل الوظيفة تعرف حاجة عن الـFeature؟**

---

### 5. `ProductTable`

هنا إجابتك كانت ممتازة لأنك لاحظت إن فيه احتمالين.

لو:

```text
ProductTable
```

تعرف تفاصيل الـProduct:

```text
Product name
Product price
Product stock
Product category
```

فهي:

```text
features/products/components/
```

لكن لو عندك Table عام جدًا:

```tsx
<DataTable columns={columns} data={data} />
```

ومش عارف حاجة عن Product أو User أو Order:

```text
shared/components/
```

🔥 ودي نقطة Architecture مهمة جدًا.

---

### 6. `useDebounce()`

قلت:

> Shared

✅ **صحيح.**

غالبًا:

```text
shared/
└── hooks/
    └── useDebounce.ts
```

لأنه Hook عام ممكن تستخدمه مع:

```text
Search
Users
Products
Orders
Filters
```

---

# 🧠 عندك قاعدة أهم من كل الكلام السابق

قبل ما تسأل:

> "أحطه فين؟"

اسأل:

> **الكود ده بيعرف إيه؟**

مثلاً:

```ts
getUser()
```

يعرف:

```text
User
```

إذن:

```text
users
```

لكن:

```ts
useDebounce()
```

لا يعرف:

```text
User
Product
Order
```

إذن:

```text
shared
```

---

## النتيجة النهائية

```text
1. UserCard
   → Feature / users ✅

2. Button
   → Shared ✅

3. getUser()
   → Feature / users / api ✅

4. formatDate()
   → Shared / utils ✅

5. ProductTable
   → Feature / products
   أو Shared لو Generic Table ✅

6. useDebounce()
   → Shared / hooks ✅
```

وأحب جدًا إنك في `ProductTable` قلت **يعتمد على طبيعة الـComponent**. دي طريقة تفكير صحيحة معماريًا.

## 🗺️ Roadmap

```text
SECTION 0: Architecture Foundation 🟢
  ├─ 0.1 HTTP Architecture           ✅
  ├─ 0.2 Separation of Concerns      ✅
  ├─ 0.3 Coupling & Cohesion         ✅
  ├─ 0.4 Feature-Based Architecture  ✅
  ├─ 0.5 Shared vs Feature           ✅
  ├─ 0.6 Public vs Internal API      🟢 ← التالي
  └─ 0.7 Dependency Direction        ⬜
```

**الخطوة التالية: نبدأ `0.6 Public vs Internal API`، ومدته حوالي 15 دقيقة، وهتتعلم إزاي تحدد إيه اللي مسموح لباقي المشروع يستخدمه وإيه المفروض يفضل Internal.**
</details>
الإجابة رقم واحد: الـ use card دي future, future user. بعد كده عندك الـ button دي بتبقى shared ورقم get user برضه نقدر نحطها في الـ future بتاع الـ user جوه الـ component. وعندنا الـ form date دي نقدر نحطها في الـ login أو الـ future بتاع الـ login أو الـ register بتاعنا جوه future يعني future login، بعد كده إيه الـ component. بعد كده الـ product table دي ممكن نحطها في الـ shared لو كان التابل ده موجود عندي في الـ dashboard في كل حتة، أو نحطها في الـ future في الـ product. والـ use dependency دي تقريبًا في الـ shared، أنا ما عارفش إيه تقريبًا في الـ shared.
ChatGPT
**إجابتك كويسة جدًا، وتقييمك 8.5/10، وأهم حاجة عندك إنك بدأت تفكر في الـownership بدل الحفظ.**

## تصحيح نقطة نقطة

### 1. `UserCard`

قلت:

> Feature → users

✅ **صحيح 100%**

```text
features/
└── users/
    └── components/
        └── UserCard.tsx
```

لأنها مرتبطة مباشرة بـUser.

---

### 2. `Button`

قلت:

> Shared

✅ **صحيح**

```text
shared/
└── components/
    └── Button.tsx
```

لأنها مش عارفة حاجة عن Users أو Products.

---

### 3. `getUser()`

قلت:

> Feature → users

✅ **صحيح في النتيجة.**

لكن قلت:

> جوه الـcomponent

❌ هنا التصحيح المهم.

مش:

```text
features/users/components/
└── getUser()
```

الأفضل:

```text
features/
└── users/
    └── api/
        └── userApi.ts
```

لأن `getUser()` **API function**، مش UI Component.

---

### 4. `formatDate()`

هنا عندك لخبطة بسيطة.

أنت قلت إنها ممكن تكون خاصة بـLogin/Register.

لكن `formatDate()` بشكل عام:

```ts
formatDate(date)
```

لو بتتعامل مع أي تاريخ في التطبيق، فهي:

```text
Shared
```

مثلاً:

```text
shared/
└── utils/
    └── formatDate.ts
```

لكن لو عندك function مخصوصة بمنطق الـLogin، مثلاً:

```ts
formatLoginAttemptDate()
```

ومرتبطة بالـLogin domain، ساعتها ممكن تكون داخل:

```text
features/auth/
```

**المعيار هو: هل الوظيفة تعرف حاجة عن الـFeature؟**

---

### 5. `ProductTable`

هنا إجابتك كانت ممتازة لأنك لاحظت إن فيه احتمالين.

لو:

```text
ProductTable
```

تعرف تفاصيل الـProduct:

```text
Product name
Product price
Product stock
Product category
```

فهي:

```text
features/products/components/
```

لكن لو عندك Table عام جدًا:

```tsx
<DataTable columns={columns} data={data} />
```

ومش عارف حاجة عن Product أو User أو Order:

```text
shared/components/
```

🔥 ودي نقطة Architecture مهمة جدًا.

---

### 6. `useDebounce()`

قلت:

> Shared

✅ **صحيح.**

غالبًا:

```text
shared/
└── hooks/
    └── useDebounce.ts
```

لأنه Hook عام ممكن تستخدمه مع:

```text
Search
Users
Products
Orders
Filters
```

---

# 🧠 عندك قاعدة أهم من كل الكلام السابق

قبل ما تسأل:

> "أحطه فين؟"

اسأل:

> **الكود ده بيعرف إيه؟**

مثلاً:

```ts
getUser()
```

يعرف:

```text
User
```

إذن:

```text
users
```

لكن:

```ts
useDebounce()
```

لا يعرف:

```text
User
Product
Order
```

إذن:

```text
shared
```

---

## النتيجة النهائية

```text
1. UserCard
   → Feature / users ✅

2. Button
   → Shared ✅

3. getUser()
   → Feature / users / api ✅

4. formatDate()
   → Shared / utils ✅

5. ProductTable
   → Feature / products
   أو Shared لو Generic Table ✅

6. useDebounce()
   → Shared / hooks ✅
```

وأحب جدًا إنك في `ProductTable` قلت **يعتمد على طبيعة الـComponent**. دي طريقة تفكير صحيحة معماريًا.

## 🗺️ Roadmap

```text
SECTION 0: Architecture Foundation 🟢
  ├─ 0.1 HTTP Architecture           ✅
  ├─ 0.2 Separation of Concerns      ✅
  ├─ 0.3 Coupling & Cohesion         ✅
  ├─ 0.4 Feature-Based Architecture  ✅
  ├─ 0.5 Shared vs Feature           ✅
  ├─ 0.6 Public vs Internal API      🟢 ← التالي
  └─ 0.7 Dependency Direction        ⬜
```

**الخطوة التالية: نبدأ `0.6 Public vs Internal API`، ومدته حوالي 15 دقيقة، وهتتعلم إزاي تحدد إيه اللي مسموح لباقي المشروع يستخدمه وإيه المفروض يفضل Internal.**
<details>
<summary>Public vs Internal API</summary>
user
Public vs Internal API
ChatGPT
**ابدأ بـ `0.6 Public vs Internal API` الآن، ومدته حوالي 15 دقيقة، وهنفهم ليه مش كل function داخل الـFeature المفروض باقي المشروع يقدر يستوردها.**

## 🗺️ Roadmap

```text
SECTION 0: Architecture Foundation 🟢
  ├─ 0.1 HTTP Architecture           ✅
  ├─ 0.2 Separation of Concerns      ✅
  ├─ 0.3 Coupling & Cohesion         ✅
  ├─ 0.4 Feature-Based Architecture  ✅
  ├─ 0.5 Shared vs Feature           ✅
  ├─ 0.6 Public vs Internal API      🟢 ← نحن هنا
  └─ 0.7 Dependency Direction        ⬜
```
</details>
Public vs Internal API
ChatGPT
**ابدأ بـ `0.6 Public vs Internal API` الآن، ومدته حوالي 15 دقيقة، وهنفهم ليه مش كل function داخل الـFeature المفروض باقي المشروع يقدر يستوردها.**

## 🗺️ Roadmap

```text
SECTION 0: Architecture Foundation 🟢
  ├─ 0.1 HTTP Architecture           ✅
  ├─ 0.2 Separation of Concerns      ✅
  ├─ 0.3 Coupling & Cohesion         ✅
  ├─ 0.4 Feature-Based Architecture  ✅
  ├─ 0.5 Shared vs Feature           ✅
  ├─ 0.6 Public vs Internal API      🟢 ← نحن هنا
  └─ 0.7 Dependency Direction        ⬜
```