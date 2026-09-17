# 5.6 Complete API Architecture

هنجمع كل الطبقات اللي اتعلمناها.

تخيل عندنا تطبيق فيه:

```text
Users
Products
Orders
```

والـ User يقدر يشتري Product.

الـ Structure ممكن تكون:

```text id="v5r8n2"
src/
│
├── shared/
│   └── api/
│       └── apiClient.ts
│
└── features/
    │
    ├── users/
    │   ├── api/
    │   │   └── userApi.ts
    │   ├── services/
    │   │   └── userService.ts
    │   ├── components/
    │   └── types/
    │
    ├── products/
    │   ├── api/
    │   │   └── productApi.ts
    │   └── types/
    │
    └── orders/
        ├── api/
        │   └── orderApi.ts
        ├── services/
        │   └── orderService.ts
        └── components/
```

## المستوى الأول: API Client

```ts id="p2x6v8"
export const api = axios.create({
  baseURL: "https://api.example.com",
  timeout: 5000,
});
```

مسؤوليته:

```text id="7y3m1q"
HTTP Configuration
Interceptors
Common behavior
```

ولا يعرف أي حاجة عن `users` أو `products` أو `orders`.

---

# المستوى الثاني: Feature API

مثلاً:

```ts id="c8w4n1"
export const getUser = async (id: number) => {
  const response = await api.get(`/users/${id}`);

  return response.data;
};
```

وده موجود داخل:

```text id="e1v5z7"
features/users/api/userApi.ts
```

هو مسؤول عن **Users API operations**.

---

# المستوى الثالث: Service Layer

دلوقتي عندنا عملية شراء:

```ts id="r6k2p9"
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

هنا حصل:

```text id="h3q7m5"
Get User
   ↓
Get Product
   ↓
Business Rule
   ↓
Create Order
```

ده **Business Logic**، ولذلك مكانه `Service Layer`.

---

# المستوى الرابع: Component

الـ Component مش محتاج يعرف كل التفاصيل دي.

يكفي:

```tsx id="u4n8c2"
const handlePurchase = async () => {
  await purchaseProduct(10, 5);
};
```

والـ UI:

```tsx id="j9m3v6"
<button onClick={handlePurchase}>
  Buy
</button>
```

الـ Component يقول:

> "نفذ عملية الشراء."

مش:

> "هات User، هات Product، احسب السعر، ابعت Order."

---

# الشكل النهائي

```text id="k6v2r9"
                    Component
                        │
                        ↓
                  Service Layer
                        │
             ┌──────────┼──────────┐
             ↓          ↓          ↓
          User API  Product API  Order API
             │          │          │
             └──────────┼──────────┘
                        ↓
                  API Client
                        ↓
                      Axios
                        ↓
                    Backend
```

وده أهم رسم في Section 5.

---

## 🔥 احفظ المسؤوليات

```text id="x3m7q1"
Component
↓
UI + User Interaction


Service
↓
Business Logic + Orchestration


Feature API
↓
API Operations


API Client
↓
HTTP Configuration + Cross-cutting behavior


Axios
↓
HTTP Library


Backend
↓
Server-side Logic + Data
```

---

# مثال عملي كامل

لو المستخدم ضغط:

```text
Buy Product
```

يحصل:

```text id="w8n4p2"
Button
  ↓
purchaseProduct()
  ↓
getUser()
  ↓
API Client
  ↓
Axios
  ↓
Backend

getProduct()
  ↓
API Client
  ↓
Axios
  ↓
Backend

Check Balance
  ↓
Business Logic

createOrder()
  ↓
API Client
  ↓
Axios
  ↓
Backend
```

لاحظ حاجة مهمة:

**الـ Service مش بديل للـ API Layer.**

هو يستخدم الـ API Layer.

```text id="c5y9r3"
Service
   ↓
API Layer
   ↓
API Client
```

---

# 📊 Section 5

```text id="n7k4w2"
5.1 API Layer vs Component       ✅
5.2 API Client                   ✅
5.3 Feature API                  ✅
5.4 Service Layer                ✅
5.5 Dependency Direction         ✅
5.6 Complete API Architecture    ✅
```

## 🏁 Section 5 مكتملة

```text id="p4x8m1"
SECTION 5
API Architecture

██████████████████ 100%
```

وأنت الآن جاهز تدخل **Section 6: TanStack Query**، وهنا هنبدأ مرحلة مهمة جدًا لأننا هنفصل بين:

```text
Server State
vs
Client State
```

**الخطوة التالية: نبدأ Section 6.1 `What is TanStack Query?` لمدة 20 دقيقة.**
<details>
<summary>تقدر تعملي مراجعه علي كل اللي فات  بنفس الترتيب</summary>
user
تقدر تعملي مراجعه علي كل اللي فات  بنفس الترتيب
ChatGPT
**خلال 30 إلى 40 دقيقة نعمل مراجعة شاملة لكل اللي فات بنفس ترتيب الخطة، من Section 0 لحد Section 5، بدون إدخال أي Concept جديد.**

## 🗺️ ترتيب المراجعة

```text
SECTION 0  Architecture Foundation     ✅
SECTION 1  Axios Fundamentals          ✅
SECTION 2  TypeScript + Axios          ✅
SECTION 3  Error Handling              ✅
SECTION 4  Interceptors                ✅
SECTION 5  API Architecture            ✅

             ↓
        مراجعة شاملة
             ↓
SECTION 6  TanStack Query              ⬜
```
</details>
تقدر تعملي مراجعه علي كل اللي فات  بنفس الترتيب
ChatGPT
**خلال 30 إلى 40 دقيقة نعمل مراجعة شاملة لكل اللي فات بنفس ترتيب الخطة، من Section 0 لحد Section 5، بدون إدخال أي Concept جديد.**

## 🗺️ ترتيب المراجعة

```text
SECTION 0  Architecture Foundation     ✅
SECTION 1  Axios Fundamentals          ✅
SECTION 2  TypeScript + Axios          ✅
SECTION 3  Error Handling              ✅
SECTION 4  Interceptors                ✅
SECTION 5  API Architecture            ✅

             ↓
        مراجعة شاملة
             ↓
SECTION 6  TanStack Query              ⬜
```