# 0.4 Feature-Based Architecture

لحد دلوقتي كنا بنتكلم عن:

```text
Separation of Concerns
Coupling
Cohesion
```

دلوقتي هنستخدم المبادئ دي لبناء **هيكل المشروع**.

---

# 1. عندنا مشروع كبير

تخيل تطبيق E-commerce:

```text
Users
Products
Orders
Cart
Authentication
Payments
```

طريقة تقليدية ممكن تكون:

```text id="a5v4fp"
src/
├── components/
├── hooks/
├── services/
├── types/
├── utils/
├── pages/
└── ...
```

في البداية شكلها جميل.

لكن لما المشروع يكبر، يحصل شيء:

```text
components/
   ↓
مئات الـcomponents

services/
   ↓
عشرات الـAPI services

types/
   ↓
مئات الـtypes
```

ويبقى السؤال:

> `UserCard.tsx` بتاع الـUsers موجود فين وسط كل ده؟

ممكن تلاقي نفسك بتدور في 5 folders.

---

# 2. Feature-Based Architecture

الفكرة:

> **نظم المشروع حسب الـFeature، وليس حسب نوع الملف فقط.**

يعني بدل:

```text
components/
services/
types/
```

نعمل:

```text id="m5gt9q"
features/
├── users/
├── products/
├── orders/
└── cart/
```

كل Feature تحتوي الكود الخاص بها.

---

# 3. مثال Users

بدل:

```text id="xj0g0a"
components/
   UserCard.tsx

services/
   userApi.ts

types/
   user.ts
```

نضعهم مع بعض:

```text id="lqg4eb"
features/
└── users/
    ├── components/
    │   └── UserCard.tsx
    │
    ├── api/
    │   └── userApi.ts
    │
    ├── types/
    │   └── user.types.ts
    │
    └── hooks/
        └── useUser.ts
```

دلوقتي كل ما يخص الـUsers موجود في:

```text id="z8lqg8"
features/users/
```

وده هو جوهر الـFeature-Based Architecture.

---

# 4. ليه ده قوي؟

تخيل إن الـUsers Feature عندها:

```text id="u9j5sh"
users/
├── components/
├── api/
├── hooks/
├── types/
└── utils/
```

وProducts عندها:

```text id="0x4f3n"
products/
├── components/
├── api/
├── hooks/
├── types/
└── utils/
```

كل Feature عندها **حدود واضحة**.

```text id="20xxkq"
Users Feature
      │
      │
      └── كل ما يتعلق بالـUsers

Products Feature
      │
      │
      └── كل ما يتعلق بالـProducts
```

ده بيساعدنا في:

- High Cohesion
- Low Coupling
- Scalability
- Maintainability

---

# 5. نربطها بالـAxios

فاكر:

```ts id="vljt2d"
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

في Feature-Based Architecture ممكن تكون:

```text id="k5n7c4"
features/
└── users/
    └── api/
        └── userApi.ts
```

والـComponent:

```text id="j3h1bs"
features/
└── users/
    └── components/
        └── UserCard.tsx
```

والـTypes:

```text id="d5v1ak"
features/
└── users/
    └── types/
        └── user.types.ts
```

كلها تحت:

```text id="9qaxv8"
users/
```

---

# 6. نقطة مهمة جدًا

**Feature-Based Architecture مش معناها إن كل Feature لازم يكون عندها نفس الـfolders بالضبط.**

مثلاً:

```text id="b9h47m"
users/
├── api/
├── components/
└── types/
```

لكن:

```text id="2c4rj5"
auth/
├── api/
├── components/
├── hooks/
└── utils/
```

و:

```text id="0efxgn"
payments/
├── api/
└── types/
```

عادي جدًا.

الـArchitecture تخدم الـDomain، مش العكس.

---

# 7. Feature يعني إيه أصلًا؟

دي نقطة مهمة.

**Feature مش مجرد folder.**

Feature هي **جزء من التطبيق له وظيفة واضحة للمستخدم أو للـbusiness**.

مثلاً:

```text id="04j8my"
Authentication
Users
Products
Orders
Cart
Payments
```

دي Features أو Domains واضحة.

لكن:

```text id="2d6w9p"
Button
Input
Modal
```

دي غالبًا **Shared UI components**، مش Features.

ودي بالضبط النقطة اللي هنبدأ بها في الدرس القادم:

```text
Feature
   VS
Shared
```

---

# 🧠 مثال ذهني

تخيل شركة.

بدل ما يكون عندك مبنى فيه:

```text
كل المحاسبين
كل المهندسين
كل المديرين
كل المبرمجين
```

وتوزعهم على عشرات المشاريع...

ممكن يكون عندك:

```text
Project A
├── Engineers
├── Accounting
└── Management

Project B
├── Engineers
├── Accounting
└── Management
```

المشروع نفسه أصبح **وحدة متماسكة**.

ده تقريبًا تفكير Feature-Based.

---

# 🎯 اختبار

عندك المشروع ده:

```text id="j0g8xx"
src/
├── components/
│   ├── UserCard.tsx
│   ├── ProductCard.tsx
│   └── OrderCard.tsx
│
├── services/
│   ├── userApi.ts
│   ├── productApi.ts
│   └── orderApi.ts
│
└── types/
    ├── user.ts
    ├── product.ts
    └── order.ts
```

لو عايزين نحوله إلى **Feature-Based Architecture**، كيف ستنظم `users`؟

اكتب فقط الشكل التقريبي:

```text
features/
└── users/
    ├── ...
    ├── ...
    └── ...
```

**الخطوة التالية: اكتب Structure الخاص بـ`users`، وأنا هقيّمه قبل ما ندخل `0.5 Shared vs Feature`.**
<details>
<summary>سهلا. هنعمل داخل السورس حاجة اسمها future. والfuture ده هنعمل تلات فولدرات: components, services, types. جوه الـcomponent tabsCard، جوه service useUI، جوه الـtypes use.ts أو DTS.</summary>
user
سهلا. هنعمل داخل السورس حاجة اسمها future. والfuture ده هنعمل تلات فولدرات: components, services, types. جوه الـcomponent tabsCard، جوه service useUI، جوه الـtypes use.ts أو DTS.
ChatGPT
**إجابتك صح في الفكرة، لكن عندك 3 تعديلات في الأسماء والـstructure، وتقييمك 8.5/10.**

أنت فهمت الجوهر: **نجمع كل ما يخص الـUser داخل Feature واحدة**. ✅

لكن الشكل الأدق:

```text
src/
└── features/
    └── users/
        ├── components/
        │   └── UserCard.tsx
        │
        ├── api/
        │   └── userApi.ts
        │
        └── types/
            └── user.types.ts
```

### التعديلات

**1. `future` ❌ → `features` ✅**

اسمها:

```text
features/
```

لأننا بنتكلم عن Features، مش Future.

---

**2. `services` 🟡 → `api` أفضل في مثالنا**

ممكن فعلًا تستخدم:

```text
services/
```

لكن بما إننا بنتكلم عن وظائف بتتعامل مع Backend عبر Axios، أنا عايزك حاليًا تستخدم:

```text
api/
```

مثلاً:

```text
api/userApi.ts
```

---

**3. `useUI` و `use.ts` ❌**

أنت هنا خلطت بين أسماء الـfiles والـconcepts.

مثلاً:

```text
components/
└── UserCard.tsx
```

ده Component.

أما:

```text
api/
└── userApi.ts
```

ده API functions.

والـTypes:

```text
types/
└── user.types.ts
```

ده مكان:

```ts
interface IUser {
  id: number;
  name: string;
}
```

---

### أهم حاجة عندي في إجابتك

أنت وصلت للفكرة:

```text
User Feature
      ↓
كل ما يتعلق بالـUser
      ↓
مكان واحد
```

وده هو المطلوب.

## 🗺️ Roadmap

```text
SECTION 0: Architecture Foundation 🟢
  ├─ 0.1 HTTP Architecture           ✅
  ├─ 0.2 Separation of Concerns      ✅
  ├─ 0.3 Coupling & Cohesion         ✅
  ├─ 0.4 Feature-Based Architecture ✅
  ├─ 0.5 Shared vs Feature           🟢 ← التالي
  ├─ 0.6 Public vs Internal API      ⬜
  └─ 0.7 Dependency Direction        ⬜
```

**الخطوة التالية: نبدأ `0.5 Shared vs Feature` لمدة حوالي 20 دقيقة، لأنها هتجاوب على سؤال مهم جدًا: إمتى أحط `Button` في `shared` وإمتى أحطه داخل `users`؟**
</details>
سهلا. هنعمل داخل السورس حاجة اسمها future. والfuture ده هنعمل تلات فولدرات: components, services, types. جوه الـcomponent tabsCard، جوه service useUI، جوه الـtypes use.ts أو DTS.
ChatGPT
**إجابتك صح في الفكرة، لكن عندك 3 تعديلات في الأسماء والـstructure، وتقييمك 8.5/10.**

أنت فهمت الجوهر: **نجمع كل ما يخص الـUser داخل Feature واحدة**. ✅

لكن الشكل الأدق:

```text
src/
└── features/
    └── users/
        ├── components/
        │   └── UserCard.tsx
        │
        ├── api/
        │   └── userApi.ts
        │
        └── types/
            └── user.types.ts
```

### التعديلات

**1. `future` ❌ → `features` ✅**

اسمها:

```text
features/
```

لأننا بنتكلم عن Features، مش Future.

---

**2. `services` 🟡 → `api` أفضل في مثالنا**

ممكن فعلًا تستخدم:

```text
services/
```

لكن بما إننا بنتكلم عن وظائف بتتعامل مع Backend عبر Axios، أنا عايزك حاليًا تستخدم:

```text
api/
```

مثلاً:

```text
api/userApi.ts
```

---

**3. `useUI` و `use.ts` ❌**

أنت هنا خلطت بين أسماء الـfiles والـconcepts.

مثلاً:

```text
components/
└── UserCard.tsx
```

ده Component.

أما:

```text
api/
└── userApi.ts
```

ده API functions.

والـTypes:

```text
types/
└── user.types.ts
```

ده مكان:

```ts
interface IUser {
  id: number;
  name: string;
}
```

---

### أهم حاجة عندي في إجابتك

أنت وصلت للفكرة:

```text
User Feature
      ↓
كل ما يتعلق بالـUser
      ↓
مكان واحد
```

وده هو المطلوب.

## 🗺️ Roadmap

```text
SECTION 0: Architecture Foundation 🟢
  ├─ 0.1 HTTP Architecture           ✅
  ├─ 0.2 Separation of Concerns      ✅
  ├─ 0.3 Coupling & Cohesion         ✅
  ├─ 0.4 Feature-Based Architecture ✅
  ├─ 0.5 Shared vs Feature           🟢 ← التالي
  ├─ 0.6 Public vs Internal API      ⬜
  └─ 0.7 Dependency Direction        ⬜
```

**الخطوة التالية: نبدأ `0.5 Shared vs Feature` لمدة حوالي 20 دقيقة، لأنها هتجاوب على سؤال مهم جدًا: إمتى أحط `Button` في `shared` وإمتى أحطه داخل `users`؟**
<details>
<summary>Shared vs Feature</summary>
user
Shared vs Feature
ChatGPT
**ابدأ بـ `0.5 Shared vs Feature` الآن، ومدته حوالي 20 دقيقة، وهنحسم قاعدة مهمة جدًا: إمتى الكود يبقى جوه الـFeature وإمتى يبقى `Shared`.**

## 🗺️ Roadmap

```text
SECTION 0: Architecture Foundation 🟢
  ├─ 0.1 HTTP Architecture           ✅
  ├─ 0.2 Separation of Concerns      ✅
  ├─ 0.3 Coupling & Cohesion         ✅
  ├─ 0.4 Feature-Based Architecture  ✅
  ├─ 0.5 Shared vs Feature           🟢 ← نحن هنا
  ├─ 0.6 Public vs Internal API      ⬜
  └─ 0.7 Dependency Direction        ⬜
```
</details>
Shared vs Feature
ChatGPT
**ابدأ بـ `0.5 Shared vs Feature` الآن، ومدته حوالي 20 دقيقة، وهنحسم قاعدة مهمة جدًا: إمتى الكود يبقى جوه الـFeature وإمتى يبقى `Shared`.**

## 🗺️ Roadmap

```text
SECTION 0: Architecture Foundation 🟢
  ├─ 0.1 HTTP Architecture           ✅
  ├─ 0.2 Separation of Concerns      ✅
  ├─ 0.3 Coupling & Cohesion         ✅
  ├─ 0.4 Feature-Based Architecture  ✅
  ├─ 0.5 Shared vs Feature           🟢 ← نحن هنا
  ├─ 0.6 Public vs Internal API      ⬜
  └─ 0.7 Dependency Direction        ⬜
```