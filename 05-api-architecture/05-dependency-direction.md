## 5.5 Dependency Direction

القاعدة ببساطة:

> **الطبقة الأعلى تعتمد على الطبقة الأقل، وليس العكس.**

عندنا:

```text id="6jv8qk"
Component
    ↓
Service
    ↓
Feature API
    ↓
API Client
    ↓
Axios
```

السهم `↓` معناه:

> اللي فوق **يستخدم / يعتمد على** اللي تحته.

### مثال

الـ `Service`:

```ts id="q8h4m2"
const purchaseProduct = async () => {
  const user = await getUser(10);
  const product = await getProduct(5);

  // Business Logic
};
```

الـ Service يعتمد على `Feature API`.

إذن:

```text id="j3k7p9"
Service
   ↓
Feature API
```

وده صح. ✅

---

## طيب العكس؟

لو `API Client` بدأ يعرف `users`:

```text id="a5r9x2"
API Client
   ↓
Users Feature
```

ده **غلط معماريًا**. ❌

لأن `API Client` موجود في `shared`، والـ `shared` المفروض يكون مستقل عن الـ Features.

يعني:

```text id="z6w1k8"
users
   ↓
shared/api        ✅

shared/api
   ↓
users             ❌
```

---

## ليه؟

تخيل إن عندك:

```text id="f8m2q4"
users
products
orders
```

كلهم يستخدموا:

```text id="r7t3v1"
shared/apiClient
```

لو الـ `apiClient` بدأ يعتمد على `users`:

```text id="c9x5k2"
apiClient
   ↓
users
```

وبعدين `users` يعتمد على `apiClient`:

```text id="n4p8s6"
users
   ↓
apiClient
```

أصبح عندنا:

```text id="v2k7m5"
users
   ↔
apiClient
```

وده ممكن يؤدي إلى **Circular Dependency**.

---

# قاعدة Architecture مهمة

```text id="h3q8w1"
Feature
   ↓
Shared

Shared
   ✕
Feature
```

يعني:

**Feature تقدر تستخدم Shared.**

لكن:

**Shared ما ينفعش يعتمد على Feature.**

---

### مثال كامل

```text id="b7m4z9"
features/users
       ↓
features/users/api
       ↓
shared/api
       ↓
axios
```

وفي نفس الوقت:

```text id="k2n6p8"
features/products
       ↓
shared/api
```

و:

```text id="d5r1x7"
features/orders
       ↓
shared/api
```

كلهم بيتجهوا لنفس الاتجاه.

### 🎯 سؤال Interview

عندك:

```text
users → shared
shared → products
products → users
```

**أي dependency هنا غلط؟ وليه؟**

**الخطوة التالية: جاوب الثلاث علاقات واحدة واحدة.**
<details>
<summary>الثانيه والثالثة غلط مايمغعش</summary>
user
الثانيه والثالثة غلط مايمغعش
ChatGPT
**إجابتك محتاجة تصحيح بسيط: الثانية هي الغلط، أما الثالثة فممكن تكون صحيحة حسب التصميم. 10 دقائق للتثبيت.**

عندنا:

```text id="q4m8z2"
users → shared       ✅
shared → products    ❌
products → users     ⚠️ ممكن
```

### 1. `users → shared` ✅

طبيعي جدًا:

```text
Users Feature
     ↓
Shared
```

الـ Feature تستخدم أدوات مشتركة.

### 2. `shared → products` ❌

دي المشكلة الأساسية.

لأن `shared` لازم يفضل مستقل عن الـ Features.

```text
Shared
  ↓
Products ❌
```

لو حصل كده، أي Feature تانية تستخدم `shared` أصبحت مرتبطة بشكل غير مباشر بـ `products`.

### 3. `products → users` ⚠️

**مش ممنوعة تلقائيًا.**

مثلاً لو الـ Products محتاجة معلومات من Users لسبب حقيقي، ممكن يحصل اعتماد:

```text
Products
   ↓
Users
```

لكن لازم يكون **مقصود ومبرر**، والأفضل تجنب Feature-to-Feature coupling غير الضروري.

المشكلة الخطيرة تكون مثل:

```text id="x7n3k1"
users
  ↓
products
  ↓
users
```

دي **Circular Dependency** ❌

### القاعدة

```text id="m8v2q5"
Feature → Shared      ✅

Shared → Feature      ❌

Feature → Feature     ⚠️ ممكن، لكن بحذر
```

**الخطوة التالية: نعمل 5.6 `Complete API Architecture` ونركب كل اللي اتعلمناه في Structure واحدة كاملة.**
</details>
الثانيه والثالثة غلط مايمغعش
ChatGPT
**إجابتك محتاجة تصحيح بسيط: الثانية هي الغلط، أما الثالثة فممكن تكون صحيحة حسب التصميم. 10 دقائق للتثبيت.**

عندنا:

```text id="q4m8z2"
users → shared       ✅
shared → products    ❌
products → users     ⚠️ ممكن
```

### 1. `users → shared` ✅

طبيعي جدًا:

```text
Users Feature
     ↓
Shared
```

الـ Feature تستخدم أدوات مشتركة.

### 2. `shared → products` ❌

دي المشكلة الأساسية.

لأن `shared` لازم يفضل مستقل عن الـ Features.

```text
Shared
  ↓
Products ❌
```

لو حصل كده، أي Feature تانية تستخدم `shared` أصبحت مرتبطة بشكل غير مباشر بـ `products`.

### 3. `products → users` ⚠️

**مش ممنوعة تلقائيًا.**

مثلاً لو الـ Products محتاجة معلومات من Users لسبب حقيقي، ممكن يحصل اعتماد:

```text
Products
   ↓
Users
```

لكن لازم يكون **مقصود ومبرر**، والأفضل تجنب Feature-to-Feature coupling غير الضروري.

المشكلة الخطيرة تكون مثل:

```text id="x7n3k1"
users
  ↓
products
  ↓
users
```

دي **Circular Dependency** ❌

### القاعدة

```text id="m8v2q5"
Feature → Shared      ✅

Shared → Feature      ❌

Feature → Feature     ⚠️ ممكن، لكن بحذر
```

**الخطوة التالية: نعمل 5.6 `Complete API Architecture` ونركب كل اللي اتعلمناه في Structure واحدة كاملة.**
<details>
<summary>اوكي</summary>
user
اوكي
ChatGPT
**خلال 20 دقيقة هنركب 5.6 `Complete API Architecture` كاملة، من الـ Component لحد الـ Backend، ودي آخر نقطة في Section 5.**

### 🗺️ Roadmap

```text id="q7k2m9"
SECTION 5  API Architecture 🟢
 ├─ 5.1 API Layer vs Component       ✅
 ├─ 5.2 API Client                   ✅
 ├─ 5.3 Feature API                  ✅
 ├─ 5.4 Service Layer                ✅
 ├─ 5.5 Dependency Direction         ✅
 └─ 5.6 Complete API Architecture    🟢 ← هنا
```
</details>
اوكي
ChatGPT
**خلال 20 دقيقة هنركب 5.6 `Complete API Architecture` كاملة، من الـ Component لحد الـ Backend، ودي آخر نقطة في Section 5.**

### 🗺️ Roadmap

```text id="q7k2m9"
SECTION 5  API Architecture 🟢
 ├─ 5.1 API Layer vs Component       ✅
 ├─ 5.2 API Client                   ✅
 ├─ 5.3 Feature API                  ✅
 ├─ 5.4 Service Layer                ✅
 ├─ 5.5 Dependency Direction         ✅
 └─ 5.6 Complete API Architecture    🟢 ← هنا
```