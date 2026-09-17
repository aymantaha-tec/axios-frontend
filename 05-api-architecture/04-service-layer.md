# 5.4 Service Layer

ركز في الفرق ده:

```text
API Layer
= يتكلم مع الـ Backend

Service Layer
= ينفذ Business Logic
```

مثال بسيط.

عندنا:

```ts
getUser(id)
```

الـ API Layer يعمل:

```ts id="7h4gk1"
export const getUser = async (id: number) => {
  const response = await api.get(`/users/${id}`);

  return response.data;
};
```

هنا إحنا فقط بنقول:

> ابعت Request للـ Backend وهات User.

---

## طيب Service Layer بيعمل إيه؟

تخيل إن عندنا قاعدة:

> المستخدم ممنوع يشتري منتج لو رصيده أقل من السعر.

دي **Business Rule**.

مش مكانها Axios.

مثلاً:

```ts id="q2z7h8"
export const purchaseProduct = async (
  userId: number,
  productId: number
) => {
  const user = await getUser(userId);
  const product = await getProduct(productId);

  if (user.balance < product.price) {
    throw new Error("Insufficient balance");
  }

  return createOrder({
    userId,
    productId,
  });
};
```

هنا عندنا **Service Layer**.

لاحظ إنه استخدم:

```text id="4w4j6s"
getUser()
getProduct()
createOrder()
```

وما تعاملش مباشرة مع Axios.

---

# ليه نحتاج Service Layer؟

لأن الـ Business Logic ممكن تكون محتاجة أكتر من API operation.

مثلاً:

```text id="f1s4jw"
purchaseProduct()
       ↓
getUser()
       ↓
getProduct()
       ↓
Check balance
       ↓
createOrder()
       ↓
Return result
```

لو حطيت كل ده داخل Component:

```text id="q7h3e5"
Component
 ├─ API calls
 ├─ Business rules
 ├─ Validation
 ├─ Calculations
 └─ UI
```

الـ Component هيكبر جدًا ويصعب اختباره وصيانته.

---

# الفرق الحقيقي

### API Layer

يسأل:

> **إزاي أكلم الـ Backend؟**

```ts id="g3k6q1"
getUser()
getProduct()
createOrder()
```

### Service Layer

يسأل:

> **إيه العملية اللي التطبيق محتاج ينفذها؟**

```ts id="m2d8x4"
purchaseProduct()
```

---

## الشكل المعماري

```text id="2c7y8p"
Component
    ↓
Service Layer
    ↓
Feature API
    ↓
API Client
    ↓
Axios
    ↓
Backend
```

مثال:

```text id="a6v8n1"
PurchaseButton
      ↓
purchaseProduct()
      ↓
getUser()
getProduct()
createOrder()
      ↓
   API Layer
      ↓
 API Client
      ↓
  Backend
```

### ⚠️ لكن مهم

**مش كل API محتاجة Service Layer.**

لو العملية بسيطة:

```ts id="j8c3s5"
const user = await getUser(10);
```

مش محتاج تعمل:

```text
Component
 ↓
Service
 ↓
API
```

ممكن مباشرة:

```text
Component
 ↓
API
```

الـ Service Layer تظهر قيمتها لما يكون عندك **Business Logic أو orchestration بين أكثر من operation**.

### 🎯 سؤال Interview

عندك:

```text id="k5x2m7"
createOrder()
```

لكن قبل إنشاء الـ Order لازم:

1. تجيب User.
2. تجيب Product.
3. تتأكد إن الرصيد كافي.
4. تحسب السعر النهائي.
5. تنشئ Order.

**تحط الخطوات دي في `API Layer` ولا `Service Layer`؟ وليه؟**

**الخطوة التالية: جاوب السؤال في سطرين.**
<details>
<summary>Service لأنها بينزنس لوجك مش بتكلم الباك ايند</summary>
user
Service لأنها بينزنس لوجك مش بتكلم الباك ايند
ChatGPT
**صح 100%، 10/10.** ✅

بالضبط:

- `API Layer` → مسؤول عن **التواصل مع Backend**.
- `Service Layer` → مسؤول عن **Business Logic**.
- التحقق من الرصيد والحسابات وترتيب العمليات = Business Logic.
- الـ Service ممكن تستدعي أكتر من API operation وتنسق بينهم.

```text
Component
   ↓
Service Layer
   ↓
Feature API
   ↓
API Client
   ↓
Backend
```

### 📍 التقدم

```text
5.1 API Layer vs Component      ✅
5.2 API Client                  ✅
5.3 Feature API                 ✅
5.4 Service Layer               ✅

5.5 Dependency Direction        🟢 ← التالي
5.6 Complete API Architecture   ⬜
```

**الخطوة التالية: نبدأ 5.5 `Dependency Direction` لمدة 10 دقائق، ونحدد مين مسموح يعتمد على مين في الـ Architecture.**
</details>
Service لأنها بينزنس لوجك مش بتكلم الباك ايند
ChatGPT
**صح 100%، 10/10.** ✅

بالضبط:

- `API Layer` → مسؤول عن **التواصل مع Backend**.
- `Service Layer` → مسؤول عن **Business Logic**.
- التحقق من الرصيد والحسابات وترتيب العمليات = Business Logic.
- الـ Service ممكن تستدعي أكتر من API operation وتنسق بينهم.

```text
Component
   ↓
Service Layer
   ↓
Feature API
   ↓
API Client
   ↓
Backend
```

### 📍 التقدم

```text
5.1 API Layer vs Component      ✅
5.2 API Client                  ✅
5.3 Feature API                 ✅
5.4 Service Layer               ✅

5.5 Dependency Direction        🟢 ← التالي
5.6 Complete API Architecture   ⬜
```

**الخطوة التالية: نبدأ 5.5 `Dependency Direction` لمدة 10 دقائق، ونحدد مين مسموح يعتمد على مين في الـ Architecture.**
<details>
<summary>تماندم</summary>
user
تماندم
ChatGPT
**خلال 10 دقائق هنثبت 5.5 `Dependency Direction` بأمثلة عملية، وهنحدد مين يعتمد على مين ومين ممنوع يعتمد على مين.**

### 🗺️ Roadmap

```text id="x7c2m1"
SECTION 5  API Architecture 🟢
 ├─ 5.1 API Layer vs Component       ✅
 ├─ 5.2 API Client                   ✅
 ├─ 5.3 Feature API                  ✅
 ├─ 5.4 Service Layer                ✅
 ├─ 5.5 Dependency Direction         🟢 ← هنا
 └─ 5.6 Complete API Architecture    ⬜
```
</details>
تماندم
ChatGPT
**خلال 10 دقائق هنثبت 5.5 `Dependency Direction` بأمثلة عملية، وهنحدد مين يعتمد على مين ومين ممنوع يعتمد على مين.**

### 🗺️ Roadmap

```text id="x7c2m1"
SECTION 5  API Architecture 🟢
 ├─ 5.1 API Layer vs Component       ✅
 ├─ 5.2 API Client                   ✅
 ├─ 5.3 Feature API                  ✅
 ├─ 5.4 Service Layer                ✅
 ├─ 5.5 Dependency Direction         🟢 ← هنا
 └─ 5.6 Complete API Architecture    ⬜
```