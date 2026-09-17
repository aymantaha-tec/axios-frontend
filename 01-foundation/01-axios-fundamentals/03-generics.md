ChatGPT
**الهدف: نفهم `axios.get<T>()` بعمق، خصوصًا معنى `<T>` ولماذا نستخدمه، قبل الانتقال لأي Concept جديد.** الوقت: **10 إلى 15 دقيقة**.

# Lesson 2, الجزء التالي: Generic في Axios

أنت وصلت للنقطة دي:

```ts
const response = await axios.get<Product[]>("/products");
```

وقلت:

> أنا متوقع Array من Product.

ممتاز. دلوقتي نفهم **إزاي TypeScript عرف ده**.

---

## 1. يعني إيه `<Product[]>`؟

الجزء ده:

```ts
<Product[]>
```

اسمه **Generic Type Argument**.

ببساطة أنت بتقول لـ Axios:

> الـ Response اللي هترجعهولي، اعتبر الـ `data` بتاعته من النوع `Product[]`.

يعني:

```ts
axios.get<Product[]>()
```

مش معناها:

> هاتلي Products.

هي معناها:

> **أنا بحدد للـ TypeScript نوع البيانات المتوقع رجوعها.**

---

## 2. بدون Generic

لو كتبت:

```ts
const response = await axios.get("/products");
```

Axios مش عارف شكل الـ data بالتحديد.

ممكن السيرفر يرجع:

```json
[
  {
    "id": 1,
    "name": "iPhone"
  }
]
```

أو:

```json
{
  "products": []
}
```

أو أي شكل آخر.

فـ TypeScript مش عنده معلومات كافية عن شكل البيانات.

---

## 3. مع Generic

أنت تقول:

```ts
const response = await axios.get<Product[]>("/products");
```

فتصبح العلاقة:

```text
axios.get<Product[]>
       ↓
AxiosResponse<Product[]>
       ↓
response.data
       ↓
Product[]
```

ودي نقطة مهمة جدًا.

---

# 4. Axios عندها Generic أصلًا

بشكل مبسط، تعريف `get` في Axios قريب من الفكرة دي:

```ts
get<T = any>(
  url: string
): Promise<AxiosResponse<T>>
```

مش مطلوب منك تحفظ التعريف دلوقتي.

المهم تفهم:

```text
T
```

هو **Placeholder للـ Type**.

لما أنت تكتب:

```ts
axios.get<Product[]>()
```

أنت بتقول:

```text
T = Product[]
```

فتصبح النتيجة منطقيًا:

```ts
Promise<AxiosResponse<Product[]>>
```

وبالتالي:

```ts
response.data
```

نوعها:

```ts
Product[]
```

---

# 5. مثال أبسط بعيد عن Axios

عشان تفهم الـ Generic نفسه.

تخيل Function:

```ts
function identity<T>(value: T): T {
  return value;
}
```

لو كتبت:

```ts
const result = identity<string>("Ahmed");
```

هنا:

```text
T = string
```

ولو:

```ts
const result = identity<number>(100);
```

يبقى:

```text
T = number
```

نفس الفكرة في Axios:

```ts
axios.get<Product[]>()
```

يعني:

```text
T = Product[]
```

---

# 6. مهم جدًا: Generic مش Validation

دي نقطة لازم تثبت.

لو كتبت:

```ts
axios.get<Product[]>("/products");
```

ده **لا يعني** إن Axios هتروح للسيرفر وتتأكد إن البيانات فعلًا `Product[]`.

يعني لو السيرفر رجع بالخطأ:

```json
{
  "hello": "world"
}
```

TypeScript مش هيوقف السيرفر ويقول:

> البيانات غلط!

لأن TypeScript موجود أثناء **التطوير والـ compile-time**، وليس runtime validation.

---

# 7. طب لو عايزين Validation؟

هنا لاحقًا هندخل:

```text
Axios
+
Zod
```

وساعتها نقدر نعمل Runtime Validation.

لكن **مش دلوقتي**.

نحن حاليًا بنتعلم:

```text
HTTP
 ↓
Axios
 ↓
TypeScript Generic
```

---

# 🧪 تدريب صغير

عندك:

```ts
interface User {
  id: number;
  name: string;
}
```

والـ API:

```text
GET /users
```

والـ Response:

```json
[
  {
    "id": 1,
    "name": "Ahmed"
  }
]
```

اكتب السطر ده باستخدام Axios:

```ts
const response = ...
```

بحيث TypeScript يعرف إن:

```ts
response.data
```

نوعها:

```ts
User[]
```

**الخطوة التالية: اكتب سطر Axios واحد فقط باستخدام `User[]`.**
<details>
<summary>.</summary>
user
.
ChatGPT
**الهدف: تثبيت الـ Generic في Axios قبل ما نكمل، ومش هنضيف أي Concept جديد.** بما إنك كتبت نقطة فقط، فهعتبرها إشارة إنك عايزني أكمل بدل ما نوقف عند التمرين. الوقت: **5 دقائق**.

## الإجابة

```ts
const response = await axios.get<User[]>("/users");
```

النتيجة:

```text
axios.get<User[]>
      ↓
T = User[]
      ↓
AxiosResponse<User[]>
      ↓
response.data = User[]
```

يعني لو عندك:

```ts
interface User {
  id: number;
  name: string;
}
```

فـ TypeScript يعرف إن:

```ts
response.data
```

هي:

```ts
User[]
```

وبالتالي:

```ts
response.data[0].name
```

صحيحة، وTypeScript يعرف أن `name` من نوع `string`.

### قاعدة الدرس

```ts
axios.get<T>()
```

الـ `T` = **نوع البيانات المتوقع داخل `response.data`**.

مثال:

```ts
axios.get<User[]>()
axios.get<Product>()
axios.get<Order[]>()
```

**الخطوة التالية: ننتقل لنقطة `axios.post()` ونفهم الفرق بين `GET` و`POST` في Axios عمليًا.**
</details>
.
ChatGPT
**الهدف: تثبيت الـ Generic في Axios قبل ما نكمل، ومش هنضيف أي Concept جديد.** بما إنك كتبت نقطة فقط، فهعتبرها إشارة إنك عايزني أكمل بدل ما نوقف عند التمرين. الوقت: **5 دقائق**.

## الإجابة

```ts
const response = await axios.get<User[]>("/users");
```

النتيجة:

```text
axios.get<User[]>
      ↓
T = User[]
      ↓
AxiosResponse<User[]>
      ↓
response.data = User[]
```

يعني لو عندك:

```ts
interface User {
  id: number;
  name: string;
}
```

فـ TypeScript يعرف إن:

```ts
response.data
```

هي:

```ts
User[]
```

وبالتالي:

```ts
response.data[0].name
```

صحيحة، وTypeScript يعرف أن `name` من نوع `string`.

### قاعدة الدرس

```ts
axios.get<T>()
```

الـ `T` = **نوع البيانات المتوقع داخل `response.data`**.

مثال:

```ts
axios.get<User[]>()
axios.get<Product>()
axios.get<Order[]>()
```

**الخطوة التالية: ننتقل لنقطة `axios.post()` ونفهم الفرق بين `GET` و`POST` في Axios عمليًا.**
<details>
<summary>🧪 تدريب صغير</summary>
user
🧪 تدريب صغير  
const response =   await axios.get<User[]>("/users")
ChatGPT
**الهدف: تثبيت أول تدريب في Axios، وإجابتك صحيحة تمامًا.** التقييم: **10/10**، والوقت: **دقيقة واحدة**.

```ts
const response = await axios.get<User[]>("/users");
```

أنت هنا طبقت 3 حاجات صح:

1. `await` لأن `axios.get()` بترجع Promise.
2. `"/users"` هو الـ endpoint.
3. `<User[]>` بتحدد لـ TypeScript إن:

```ts
response.data
```

نوعها:

```ts
User[]
```

يعني فهمك الحالي:

```text
axios.get<User[]>()
       ↓
Promise<AxiosResponse<User[]>>
       ↓
await
       ↓
response
       ↓
response.data
       ↓
User[]
```

### 🟢 Progress

```text
Lesson 1: HTTP              ✅
Lesson 2: Axios GET         🟢
  ├── axios.get()           ✅
  ├── await                 ✅
  ├── response              ✅
  ├── response.data         ✅
  └── Generic <T>           ✅
```

**الخطوة التالية: نكمل `axios.post()` ونفهم أولًا شكل الـ arguments بتاعته.**
</details>
🧪 تدريب صغير  
const response =   await axios.get<User[]>("/users")
ChatGPT
**الهدف: تثبيت أول تدريب في Axios، وإجابتك صحيحة تمامًا.** التقييم: **10/10**، والوقت: **دقيقة واحدة**.

```ts
const response = await axios.get<User[]>("/users");
```

أنت هنا طبقت 3 حاجات صح:

1. `await` لأن `axios.get()` بترجع Promise.
2. `"/users"` هو الـ endpoint.
3. `<User[]>` بتحدد لـ TypeScript إن:

```ts
response.data
```

نوعها:

```ts
User[]
```

يعني فهمك الحالي:

```text
axios.get<User[]>()
       ↓
Promise<AxiosResponse<User[]>>
       ↓
await
       ↓
response
       ↓
response.data
       ↓
User[]
```

### 🟢 Progress

```text
Lesson 1: HTTP              ✅
Lesson 2: Axios GET         🟢
  ├── axios.get()           ✅
  ├── await                 ✅
  ├── response              ✅
  ├── response.data         ✅
  └── Generic <T>           ✅
```

**الخطوة التالية: نكمل `axios.post()` ونفهم أولًا شكل الـ arguments بتاعته.**
<details>
<summary>axios.post()</summary>
user
axios.post()  
ChatGPT
**الهدف: نفهم `axios.post()` فقط، ونربطه بما تعلمناه عن HTTP و `axios.get()`، بدون دخول في أي Concept جديد.** الوقت: **10 دقائق**.
</details>
axios.post()  
ChatGPT
**الهدف: نفهم `axios.post()` فقط، ونربطه بما تعلمناه عن HTTP و `axios.get()`، بدون دخول في أي Concept جديد.** الوقت: **10 دقائق**.