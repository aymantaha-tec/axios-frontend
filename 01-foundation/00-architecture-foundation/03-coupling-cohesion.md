# 0.3 Coupling & Cohesion

عندنا مفهومين لازم تفهمهم مع بعض:

```text
Coupling
Cohesion
```

خلينا نبدأ بـ **Coupling**.

---

# 1. Coupling

**Coupling = درجة اعتماد جزء من الكود على جزء آخر.**

يعني:

> لو غيرت A، قد إيه B هيتأثر؟

### مثال

عندك:

```tsx
function UsersPage() {
  axios.get("/users");
}
```

الـComponent هنا مرتبط مباشرة بـAxios.

```text
UsersPage
    │
    └──────→ Axios
```

ده **Coupling**.

ولو غيرنا طريقة التعامل مع الـAPI، ممكن نضطر نعدل الـComponent.

---

## مثال أفضل

```tsx
function UsersPage() {
  const users = getUsers();
}
```

والـAPI Layer:

```ts
async function getUsers() {
  const response = await axios.get("/users");

  return response.data;
}
```

بقى:

```text
UsersPage
    │
    ↓
getUsers()
    │
    ↓
Axios
```

الـComponent لم يعد يعرف تفاصيل Axios.

إذن قللنا **Coupling**.

---

# 2. ليه Low Coupling مهم؟

تخيل إن عندك 30 Component:

```text
Component 1 → Axios
Component 2 → Axios
Component 3 → Axios
...
Component 30 → Axios
```

لو قررت تغير طريقة الـAPI communication:

```text
Axios
   ↓
Something else
```

المشكلة؟

لازم تعدل أماكن كثيرة.

لكن لو:

```text
Components
     ↓
 API Layer
     ↓
   Axios
```

التغيير غالبًا يتركز في الـAPI Layer.

وده اسمه:

> **Low Coupling**

يعني الأجزاء مش متعلقة ببعض بشكل زائد.

---

# 3. Cohesion

دلوقتي العكس.

**Cohesion = مدى ترابط المسؤوليات الموجودة داخل نفس الوحدة.**

بمعنى أبسط:

> هل الحاجات الموجودة مع بعض فعلاً لها علاقة ببعض؟

---

### مثال سيئ

ملف اسمه:

```text
utils.ts
```

وفيه:

```ts
getUser()
calculateTax()
formatDate()
login()
sendEmail()
getProducts()
```

😵

الحاجات دي مش مرتبطة ببعض بشكل قوي.

إذن:

```text
Low Cohesion ❌
```

---

### مثال أفضل

ملف:

```text
userApi.ts
```

وفيه:

```ts
getUser()
createUser()
updateUser()
deleteUser()
```

كل الوظائف مرتبطة بالـUser API.

إذن:

```text
High Cohesion ✅
```

---

# 4. العلاقة بينهم

الهدف المعماري غالبًا:

```text
Low Coupling
        +
High Cohesion
        ↓
Better Architecture
```

يعني:

### Low Coupling

الأجزاء **تعتمد على بعضها بأقل قدر ممكن**.

### High Cohesion

كل جزء يحتوي على أشياء **مرتبطة ببعضها منطقيًا**.

---

# 5. مثال كامل

عندك مشروع:

```text
src/
```

بدل:

```text
src/
├── everything.ts
```

نعمل:

```text
src/
├── features/
│   └── users/
│       ├── api/
│       │   └── userApi.ts
│       ├── components/
│       │   └── UserCard.tsx
│       └── types/
│           └── user.types.ts
```

هنا:

```text
userApi.ts
```

عنده **High Cohesion** لأنه مركز على User API.

والـComponent لا يحتاج معرفة تفاصيل Axios.

إذن عندنا **Lower Coupling**.

---

# 6. خلي بالك من حاجة مهمة

**Low Coupling لا يعني Zero Coupling.**

مستحيل المشروع يكون كل جزء فيه منفصل 100%.

طبيعي جدًا:

```text
Component
    ↓
API
```

أن الـComponent يعتمد على API function.

المطلوب إن الاعتماد يكون:

> **واضح + منطقي + محدود**

مش:

```text
Component
 ↓
Axios
 ↓
localStorage
 ↓
Auth
 ↓
Database logic
 ↓
Business logic
```

كل ده جوه Component واحد.

---

# 7. مثال من اللي أنت درسته

أنت عندك:

```ts
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

وعندك:

```tsx
const user = await getUser(10);
```

ليه التصميم ده كويس؟

لأن:

```text
Component
   │
   ↓
getUser()
```

والـComponent لا يحتاج يعرف:

```text
Axios
AxiosResponse
IApiResponse
response.data
response.data.payload
```

كل التفاصيل دي مخفية داخل API Layer.

ده مثال ممتاز على:

```text
Separation of Concerns
        +
Low Coupling
        +
High Cohesion
```

---

# 🧠 اختبارك

عندك الكود ده:

```text
src/
├── utils.ts
│
├── components/
│   └── UsersPage.tsx
│
└── api/
    └── users.ts
```

و`utils.ts` يحتوي:

```ts
getUser()
getProduct()
formatDate()
login()
calculateOrderTotal()
```

**السؤال:**

1. هل عندنا High Cohesion ولا Low Cohesion في `utils.ts`؟
2. وليه؟
3. لو `UsersPage` يستخدم Axios مباشرة، هل ده بيزود Coupling ولا بيقلله؟

جاوب على **1 و2 و3**، وأنا هصححهم لك قبل ما ندخل على `0.4 Feature-Based Architecture`.
<details>
<summary>الإجابة هل الـ high cohesion ولا الـ low cohesion في الـ utils.ts. في هذه الحالة لا، دوت هنا بقى low cohesion، لأن الـ function الموجودة جوا الـ utils.ts غير مترابطة أو مالهاش علاقة ببعضها. فيها function فيها logging، فيها calculation، فيها business logic، فيها sanitization، فيها form، فيها get user، حاجات، فهي لا، هي low cohesion. وده بتبقى جبت لوحده. طب الـ user اللي بيستخدم الـ function هل ده بيزود coupling ما بين الـ...؟ لا، ده بيزوده.</summary>
user
الإجابة هل الـ high cohesion ولا الـ low cohesion في الـ utils.ts. في هذه الحالة لا، دوت هنا بقى low cohesion، لأن الـ function الموجودة جوا الـ utils.ts غير مترابطة أو مالهاش علاقة ببعضها. فيها function فيها logging، فيها calculation، فيها business logic، فيها sanitization، فيها form، فيها get user، حاجات، فهي لا، هي low cohesion. وده بتبقى جبت لوحده. طب الـ user اللي بيستخدم الـ function هل ده بيزود coupling ما بين الـ...؟ لا، ده بيزوده.
ChatGPT
**إجابتك ممتازة: 10/10، وخد حوالي دقيقتين فقط لتثبيت الفرق بين المصطلحين.**

### التصحيح

**1. `utils.ts` → Low Cohesion ✅**

تحليلك صحيح جدًا.

لما يكون عندنا:

```ts
getUser()
getProduct()
formatDate()
login()
calculateOrderTotal()
sanitizeInput()
```

كلهم في نفس الملف، لكن مفيش علاقة قوية تجمعهم، يبقى:

```text
Low Cohesion ❌
```

لأن الوحدة الواحدة مش مركزة على **مسؤولية مترابطة**.

---

**2. السبب → صحيح ✅**

أنت قلت إن الـfunctions غير مترابطة، وده بالضبط المقصود.

مثلاً الأفضل:

```text
userApi.ts
├── getUser()
├── createUser()
└── updateUser()
```

ده:

```text
High Cohesion ✅
```

لأن كلهم متعلقين بـUser API.

---

**3. استخدام Axios مباشرة داخل `UsersPage` → يزيد Coupling ✅**

بالضبط.

```text
UsersPage
    │
    └──────→ Axios
```

الـComponent أصبح مرتبطًا بتفاصيل Axios.

الأفضل:

```text
UsersPage
    │
    ↓
getUsers()
    │
    ↓
Axios
```

كده قللنا الـCoupling بين الـUI وHTTP implementation.

---

## 🧠 احفظ الفرق بالطريقة دي

```text
COHESION
"الحاجات جوه الوحدة دي شبه بعض ومترابطة؟"

COUPLING
"الوحدة دي متعلقة قد إيه بوحدات تانية؟"
```

أو بطريقة أسرع:

```text
High Cohesion
= related things together ✅

Low Coupling
= unrelated parts depend on each other as little as possible ✅
```

وده أساس مهم جدًا قبل ما ندخل على **Feature-Based Architecture**.

---

## 🗺️ Roadmap

```text
SECTION 0: Architecture Foundation 🟢
  ├─ 0.1 HTTP Architecture           ✅
  ├─ 0.2 Separation of Concerns      ✅
  ├─ 0.3 Coupling & Cohesion         ✅
  ├─ 0.4 Feature-Based Architecture 🟢 ← التالي
  ├─ 0.5 Shared vs Feature           ⬜
  ├─ 0.6 Public vs Internal API      ⬜
  └─ 0.7 Dependency Direction        ⬜

SECTION 1 Axios Fundamentals         ✅
SECTION 2 TypeScript + Axios         ✅
SECTION 3 Error Handling             ✅
SECTION 4 Interceptors               ⬜
```

**الخطوة التالية: اكتب `كمل` ونبدأ `0.4 Feature-Based Architecture`، ومدته حوالي 20 دقيقة.**
</details>
الإجابة هل الـ high cohesion ولا الـ low cohesion في الـ utils.ts. في هذه الحالة لا، دوت هنا بقى low cohesion، لأن الـ function الموجودة جوا الـ utils.ts غير مترابطة أو مالهاش علاقة ببعضها. فيها function فيها logging، فيها calculation، فيها business logic، فيها sanitization، فيها form، فيها get user، حاجات، فهي لا، هي low cohesion. وده بتبقى جبت لوحده. طب الـ user اللي بيستخدم الـ function هل ده بيزود coupling ما بين الـ...؟ لا، ده بيزوده.
ChatGPT
**إجابتك ممتازة: 10/10، وخد حوالي دقيقتين فقط لتثبيت الفرق بين المصطلحين.**

### التصحيح

**1. `utils.ts` → Low Cohesion ✅**

تحليلك صحيح جدًا.

لما يكون عندنا:

```ts
getUser()
getProduct()
formatDate()
login()
calculateOrderTotal()
sanitizeInput()
```

كلهم في نفس الملف، لكن مفيش علاقة قوية تجمعهم، يبقى:

```text
Low Cohesion ❌
```

لأن الوحدة الواحدة مش مركزة على **مسؤولية مترابطة**.

---

**2. السبب → صحيح ✅**

أنت قلت إن الـfunctions غير مترابطة، وده بالضبط المقصود.

مثلاً الأفضل:

```text
userApi.ts
├── getUser()
├── createUser()
└── updateUser()
```

ده:

```text
High Cohesion ✅
```

لأن كلهم متعلقين بـUser API.

---

**3. استخدام Axios مباشرة داخل `UsersPage` → يزيد Coupling ✅**

بالضبط.

```text
UsersPage
    │
    └──────→ Axios
```

الـComponent أصبح مرتبطًا بتفاصيل Axios.

الأفضل:

```text
UsersPage
    │
    ↓
getUsers()
    │
    ↓
Axios
```

كده قللنا الـCoupling بين الـUI وHTTP implementation.

---

## 🧠 احفظ الفرق بالطريقة دي

```text
COHESION
"الحاجات جوه الوحدة دي شبه بعض ومترابطة؟"

COUPLING
"الوحدة دي متعلقة قد إيه بوحدات تانية؟"
```

أو بطريقة أسرع:

```text
High Cohesion
= related things together ✅

Low Coupling
= unrelated parts depend on each other as little as possible ✅
```

وده أساس مهم جدًا قبل ما ندخل على **Feature-Based Architecture**.

---

## 🗺️ Roadmap

```text
SECTION 0: Architecture Foundation 🟢
  ├─ 0.1 HTTP Architecture           ✅
  ├─ 0.2 Separation of Concerns      ✅
  ├─ 0.3 Coupling & Cohesion         ✅
  ├─ 0.4 Feature-Based Architecture 🟢 ← التالي
  ├─ 0.5 Shared vs Feature           ⬜
  ├─ 0.6 Public vs Internal API      ⬜
  └─ 0.7 Dependency Direction        ⬜

SECTION 1 Axios Fundamentals         ✅
SECTION 2 TypeScript + Axios         ✅
SECTION 3 Error Handling             ✅
SECTION 4 Interceptors               ⬜
```

**الخطوة التالية: اكتب `كمل` ونبدأ `0.4 Feature-Based Architecture`، ومدته حوالي 20 دقيقة.**
<details>
<summary>كمل</summary>
user
كمل
ChatGPT
**ابدأ بـ `0.4 Feature-Based Architecture` الآن، ومدته حوالي 20 دقيقة، وهنبني الـfolder structure خطوة خطوة بدل ما نحفظه.**

## 🗺️ Roadmap

```text id="7nq5mt"
SECTION 0: Architecture Foundation 🟢
  ├─ 0.1 HTTP Architecture           ✅
  ├─ 0.2 Separation of Concerns      ✅
  ├─ 0.3 Coupling & Cohesion         ✅
  ├─ 0.4 Feature-Based Architecture 🟢 ← نحن هنا
  ├─ 0.5 Shared vs Feature           ⬜
  ├─ 0.6 Public vs Internal API      ⬜
  └─ 0.7 Dependency Direction        ⬜

SECTION 1 Axios Fundamentals         ✅
SECTION 2 TypeScript + Axios         ✅
SECTION 3 Error Handling             ✅
SECTION 4 Interceptors               ⬜
```
</details>
كمل
ChatGPT
**ابدأ بـ `0.4 Feature-Based Architecture` الآن، ومدته حوالي 20 دقيقة، وهنبني الـfolder structure خطوة خطوة بدل ما نحفظه.**

## 🗺️ Roadmap

```text id="7nq5mt"
SECTION 0: Architecture Foundation 🟢
  ├─ 0.1 HTTP Architecture           ✅
  ├─ 0.2 Separation of Concerns      ✅
  ├─ 0.3 Coupling & Cohesion         ✅
  ├─ 0.4 Feature-Based Architecture 🟢 ← نحن هنا
  ├─ 0.5 Shared vs Feature           ⬜
  ├─ 0.6 Public vs Internal API      ⬜
  └─ 0.7 Dependency Direction        ⬜

SECTION 1 Axios Fundamentals         ✅
SECTION 2 TypeScript + Axios         ✅
SECTION 3 Error Handling             ✅
SECTION 4 Interceptors               ⬜
```