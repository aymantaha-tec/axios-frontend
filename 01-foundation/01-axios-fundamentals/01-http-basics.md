ابدأ **Lesson 1: HTTP Request/Response**، لأن فهم HTTP هو الأساس اللي هيخليك تفهم Axios بدل ما تحفظ API بتاعته. الزمن: **45 إلى 60 دقيقة**.

## Step 1 of 4: نفهم الـ HTTP قبل Axios

### 🎯 هدف الدرس

بعد الدرس ده لازم تقدر تبص على الكود ده:

```ts
const response = await axios.get("/users");
```

وتفهم بالضبط إيه اللي حصل من أول ما المتصفح بعت الـ Request لحد ما السيرفر رجع الـ Response.

---

# 1. يعني إيه HTTP؟

**HTTP = HyperText Transfer Protocol**

ببساطة جدًا:

> HTTP هو البروتوكول اللي بينظم طريقة التواصل بين الـ Client والـ Server.

عندك:

```text
React App
   │
   │ HTTP Request
   ▼
Backend API
   │
   │ HTTP Response
   ▼
React App
```

مثال حقيقي:

```text
React
  ↓
GET /api/users
  ↓
Backend
  ↓
200 OK
  ↓
[users]
```

Axios **مش هو اللي اخترع التواصل ده**.

Axios مجرد أداة بتساعدك تتعامل مع HTTP بشكل أسهل.

---

# 2. الـ Request

لما React يطلب بيانات من السيرفر، بنسمي ده:

**HTTP Request**

والـ Request بيتكون بشكل أساسي من:

```text
Request
├── Method
├── URL
├── Headers
└── Body
```

مثال:

```http
POST /api/login HTTP/1.1
Content-Type: application/json
Authorization: Bearer xxx

{
  "email": "ayman@example.com",
  "password": "123456"
}
```

ركز في الأربع حاجات دول، لأن Axios كله تقريبًا هيدور حوالينهم.

---

# 3. Method

الـ Method بتقول للسيرفر:

> أنا عايز أعمل إيه؟

أشهر Methods:

| Method | الاستخدام |
|---|---|
| GET | جلب بيانات |
| POST | إنشاء بيانات |
| PUT | استبدال بيانات |
| PATCH | تعديل جزء من البيانات |
| DELETE | حذف بيانات |

مثال:

```http
GET /users
```

معناه:

> هات المستخدمين.

أما:

```http
POST /users
```

معناه:

> أنشئ مستخدم جديد.

---

# 4. URL

مثلاً:

```http
https://api.example.com/users
```

نقدر نقسمه:

```text
https://
   ↓
Protocol

api.example.com
   ↓
Host

/users
   ↓
Path
```

وفي المشاريع الحقيقية غالبًا عندك:

```text
https://api.example.com
```

كـ:

```ts
baseURL
```

وبعدين:

```text
/users
```

كـ endpoint.

وده تحديدًا هيفسرلك بعدين ليه بنعمل:

```ts
axios.create({
  baseURL: "https://api.example.com",
});
```

---

# 5. Headers

الـ Headers عبارة عن **metadata عن الـ Request**.

مثلاً:

```http
Content-Type: application/json
```

معناها:

> الـ Body اللي ببعتُه JSON.

وممكن يكون عندك:

```http
Authorization: Bearer eyJ...
```

وده معناه:

> الطلب ده مصحوب بـ Access Token.

وممكن:

```http
Accept: application/json
```

يعني:

> أنا متوقع Response بصيغة JSON.

---

# 6. Body

الـ Body هو البيانات اللي بتبعتها للسيرفر.

مثلاً Login:

```json
{
  "email": "ayman@example.com",
  "password": "123456"
}
```

ده ممكن يتبعت في:

```http
POST /api/login
```

لكن لاحظ:

**GET غالبًا لا نستخدم معه Body.**

أما:

```text
POST
PUT
PATCH
```

فغالبًا عندهم Body.

---

# 7. Query Parameters

دي نقطة مهمة جدًا في Axios.

لو عندك:

```http
GET /products?category=phones&page=2
```

فالجزء ده:

```text
?category=phones&page=2
```

اسمه:

**Query Parameters**

في Axios بدل ما تكتبها يدويًا:

```ts
axios.get("/products?category=phones&page=2");
```

ممكن:

```ts
axios.get("/products", {
  params: {
    category: "phones",
    page: 2,
  },
});
```

وده من أول الأماكن اللي Axios بيخلي حياتك أسهل فيها.

---

# 8. الـ Response

بعد ما السيرفر يستقبل Request، بيرجع:

**HTTP Response**

مثلاً:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "status": true,
  "payload": [
    {
      "id": 1,
      "name": "Ahmed"
    }
  ]
}
```

الـ Response عنده:

```text
Response
├── Status Code
├── Headers
└── Body
```

---

# 9. Status Code

دي من أهم الحاجات في شغل الـ API.

### 2xx = Success

```text
200 OK
201 Created
204 No Content
```

### 3xx = Redirection

```text
301
302
304
```

### 4xx = Client Error

```text
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
409 Conflict
422 Unprocessable Entity
```

### 5xx = Server Error

```text
500 Internal Server Error
502 Bad Gateway
503 Service Unavailable
```

احفظ التصنيف:

```text
2xx → نجح
3xx → Redirect
4xx → المشكلة غالبًا في الطلب/الصلاحيات
5xx → المشكلة غالبًا في السيرفر
```

---

# 10. نقطة مهمة جدًا في Axios

لما تعمل:

```ts
const response = await axios.get("/users");
```

Axios مش بيرجعلك الـ JSON مباشرة.

بيرجعلك **AxiosResponse**.

تقريبًا:

```ts
response = {
  data,
  status,
  statusText,
  headers,
  config,
  request
};
```

يعني:

```ts
response.data
```

هي البيانات اللي السيرفر رجعها.

مثلاً:

```ts
const response = await axios.get<User[]>("/users");

console.log(response.data);
```

---

# 11. الصورة الكاملة

دلوقتي اربط كل حاجة ببعض:

```text
React
  │
  │ axios.get("/users")
  ▼
Axios
  │
  │ HTTP Request
  │
  │ GET
  │ /users
  │ Headers
  │
  ▼
Backend
  │
  │ Process Request
  ▼
Database
  │
  ▼
Backend
  │
  │ HTTP Response
  │
  │ 200 OK
  │ JSON
  ▼
Axios
  │
  ▼
response.data
  │
  ▼
React
```

**دي أهم صورة في الدرس كله.**

---

# 🧪 التدريب 1

من غير ما تبص للإجابة، حل السيناريو ده:

عندك صفحة Users.

أنت عايز تجيب:

```text
users
page = 2
limit = 10
```

والـ API:

```text
GET /api/users
```

### المطلوب

اكتبلي شكل الـ HTTP Request بنفسك:

```text
Method:
URL:
Query Params:
Headers:
Body:
```

**ما تستخدمش Axios لسه.**

عايزك تفكر في HTTP الأول.

### 🎤 Interview Question

**ليه `page` و `limit` بنحطهم Query Parameters، ومش Body؟**

اكتب إجابتك على السؤالين، وأنا هقيّم إجابتك كـ Interview، وبعدها ندخل **Lesson 2: HTTP Methods بالتفصيل**.
<details>
<summary>GET</summary>
user
GET
api/users
Users?page=2&limit=10
Token access 
contet-type: application/json


2

لانها بتبقي في الheader
ChatGPT
**الهدف: تثبيت فهم HTTP Request قبل ما ندخل Axios.** إجابتك جيدة كبداية، لكن عندك نقطة مفاهيمية مهمة محتاجة تصحيح. التقييم: **7/10**، والوقت لإصلاحها: **5 دقائق**.

## 1. تقييم إجابتك

أنت كتبت:

```text
GET
api/users
Users?page=2&limit=10
Token access
content-type: application/json
```

### ✅ الصح

- `GET` صحيح.
- `/api/users` صحيح.
- `page=2&limit=10` صحيح كـ Query Parameters.
- الـ Access Token يكون عادةً في `Authorization` Header.

الصيغة الاحترافية تكون:

```http
GET /api/users?page=2&limit=10
Authorization: Bearer <access-token>
Accept: application/json
```

### ⚠️ التصحيح المهم

أنت قلت:

> "لانها بتبقى في الـ header"

دي **مش إجابة السؤال**.

السؤال كان:

> لماذا `page` و `limit` يكونوا Query Parameters وليس Body؟

الإجابة:

**لأنهما يمثلان معايير للتحكم في نتيجة الـ GET Request، وليس بيانات نريد إرسالها لإنشاء أو تعديل Resource.**

مثلاً:

```http
GET /api/users?page=2&limit=10
```

معناها:

> هات الـ users، لكن هات الصفحة الثانية، وبحد أقصى 10 عناصر.

أما الـ Body فغالبًا نستخدمه لإرسال **بيانات الـ Resource نفسه**، خصوصًا مع `POST`, `PUT`, `PATCH`.

مثال:

```http
POST /api/users

{
  "name": "Ahmed",
  "email": "ahmed@example.com"
}
```

هنا أنت تقول للسيرفر:

> أنشئ User بهذه البيانات.

---

# 2. نقطة مهمة جدًا: `Content-Type`

أنت كتبت:

```http
Content-Type: application/json
```

مع `GET`.

هنا محتاج نفرق بين:

```text
Content-Type
```

و:

```text
Accept
```

### Content-Type

يقول:

> البيانات الموجودة في الـ Body نوعها إيه؟

مثلاً:

```http
Content-Type: application/json
```

يعني:

> الـ Body الذي أرسلته JSON.

### Accept

يقول:

> أنا عايز الـ Response يكون إيه؟

مثلاً:

```http
Accept: application/json
```

وفي GET بدون Body، غالبًا **`Accept` هو الأنسب** لو عايز تحدد صيغة الـ Response.

---

# 3. الشكل النهائي

```http
GET /api/users?page=2&limit=10 HTTP/1.1
Authorization: Bearer <access-token>
Accept: application/json
```

لاحظ:

```text
GET
 ↓
/api/users
 ↓
?page=2&limit=10
 ↓
Headers
 ↓
Server
```

---

# 🎤 Interview Evaluation

### السؤال:

**ليه `page` و `limit` Query Parameters؟**

إجابتك:

> لأنها في الـ Header.

❌ **0/10**

الإجابة الأقوى:

> لأن `page` و `limit` بيحددوا طريقة استرجاع الـ collection، مثل pagination، sorting، filtering، والـ GET Request عادةً يستخدم Query Parameters لهذه المعايير بدل وضعها في Body.

**دي إجابة Mid-Level أحسن بكتير.**

---

# 🧠 اختبار سريع قبل Lesson 2

عندك:

```text
POST /api/products
```

وعايز تنشئ Product:

```json
{
  "name": "iPhone",
  "price": 50000,
  "category": "phones"
}
```

اكتب فقط:

```text
Method:
URL:
Headers:
Query Params:
Body:
```

**الخطوة التالية: اكتب إجابتك على الاختبار ده.**
</details>
GET
api/users
Users?page=2&limit=10
Token access 
contet-type: application/json


2

لانها بتبقي في الheader
ChatGPT
**الهدف: تثبيت فهم HTTP Request قبل ما ندخل Axios.** إجابتك جيدة كبداية، لكن عندك نقطة مفاهيمية مهمة محتاجة تصحيح. التقييم: **7/10**، والوقت لإصلاحها: **5 دقائق**.

## 1. تقييم إجابتك

أنت كتبت:

```text
GET
api/users
Users?page=2&limit=10
Token access
content-type: application/json
```

### ✅ الصح

- `GET` صحيح.
- `/api/users` صحيح.
- `page=2&limit=10` صحيح كـ Query Parameters.
- الـ Access Token يكون عادةً في `Authorization` Header.

الصيغة الاحترافية تكون:

```http
GET /api/users?page=2&limit=10
Authorization: Bearer <access-token>
Accept: application/json
```

### ⚠️ التصحيح المهم

أنت قلت:

> "لانها بتبقى في الـ header"

دي **مش إجابة السؤال**.

السؤال كان:

> لماذا `page` و `limit` يكونوا Query Parameters وليس Body؟

الإجابة:

**لأنهما يمثلان معايير للتحكم في نتيجة الـ GET Request، وليس بيانات نريد إرسالها لإنشاء أو تعديل Resource.**

مثلاً:

```http
GET /api/users?page=2&limit=10
```

معناها:

> هات الـ users، لكن هات الصفحة الثانية، وبحد أقصى 10 عناصر.

أما الـ Body فغالبًا نستخدمه لإرسال **بيانات الـ Resource نفسه**، خصوصًا مع `POST`, `PUT`, `PATCH`.

مثال:

```http
POST /api/users

{
  "name": "Ahmed",
  "email": "ahmed@example.com"
}
```

هنا أنت تقول للسيرفر:

> أنشئ User بهذه البيانات.

---

# 2. نقطة مهمة جدًا: `Content-Type`

أنت كتبت:

```http
Content-Type: application/json
```

مع `GET`.

هنا محتاج نفرق بين:

```text
Content-Type
```

و:

```text
Accept
```

### Content-Type

يقول:

> البيانات الموجودة في الـ Body نوعها إيه؟

مثلاً:

```http
Content-Type: application/json
```

يعني:

> الـ Body الذي أرسلته JSON.

### Accept

يقول:

> أنا عايز الـ Response يكون إيه؟

مثلاً:

```http
Accept: application/json
```

وفي GET بدون Body، غالبًا **`Accept` هو الأنسب** لو عايز تحدد صيغة الـ Response.

---

# 3. الشكل النهائي

```http
GET /api/users?page=2&limit=10 HTTP/1.1
Authorization: Bearer <access-token>
Accept: application/json
```

لاحظ:

```text
GET
 ↓
/api/users
 ↓
?page=2&limit=10
 ↓
Headers
 ↓
Server
```

---

# 🎤 Interview Evaluation

### السؤال:

**ليه `page` و `limit` Query Parameters؟**

إجابتك:

> لأنها في الـ Header.

❌ **0/10**

الإجابة الأقوى:

> لأن `page` و `limit` بيحددوا طريقة استرجاع الـ collection، مثل pagination، sorting، filtering، والـ GET Request عادةً يستخدم Query Parameters لهذه المعايير بدل وضعها في Body.

**دي إجابة Mid-Level أحسن بكتير.**

---

# 🧠 اختبار سريع قبل Lesson 2

عندك:

```text
POST /api/products
```

وعايز تنشئ Product:

```json
{
  "name": "iPhone",
  "price": 50000,
  "category": "phones"
}
```

اكتب فقط:

```text
Method:
URL:
Headers:
Query Params:
Body:
```

**الخطوة التالية: اكتب إجابتك على الاختبار ده.**
<details>
<summary>بس انا عشان تجاوبك زي كده</summary>
user
بس انا عشان تجاوبك زي كده 
لأن page و limit بيحددوا طريقة استرجاع الـ collection، مثل pagination، sorting، filtering، والـ GET Request عادةً يستخدم Query Parameters لهذه المعايير بدل وضعها في Body.

انت مشرحتوش ده فأنا عجيبة منين
ChatGPT
**الهدف: ما تحكمش على إجابتك بمعلومة أنا ما شرحتهاش، وأنت معاك حق في النقطة دي.** الخطأ هنا مني في طريقة التدريب، لأنني سألتك عن مفهوم لم أدرّسه لك أولًا، والوقت لتصحيح مسار الدرس: **5 دقائق**.

1. **أنت كنت بتجاوب بناءً على اللي اتشرح فعلًا**

أنا شرحت:

```text
Query Parameters
?category=phones&page=2
```

وقلت إنها:

> Query Parameters

لكن **ما شرحتش لماذا** `page` و `limit` يعتبروا Query Parameters، ولا شرحت مفهوم:

```text
pagination
filtering
sorting
```

فطبيعي جدًا إنك ما تعرفش الإجابة.

2. **القاعدة اللي كان المفروض أشرحها**

خلينا نبنيها من الصفر.

عندك API:

```http
GET /api/products
```

أنت بتقول:

> هات المنتجات.

طيب لو عندي **10,000 منتج**؟

مش منطقي أطلبهم كلهم مرة واحدة.

أقول للسيرفر:

```http
GET /api/products?page=2&limit=10
```

معناها:

```text
page = 2
→ عايز الصفحة الثانية

limit = 10
→ عايز 10 منتجات
```

وده اسمه:

**Pagination**

---

3. ونفس الفكرة مع Filtering

مثلاً:

```http
GET /api/products?category=phones
```

معناها:

> هات المنتجات اللي category بتاعها phones.

ده اسمه:

**Filtering**

---

4. ومع Sorting

مثلاً:

```http
GET /api/products?sort=price
```

معناها:

> هات المنتجات مرتبة حسب السعر.

ده:

**Sorting**

---

5. إذن الـ Query Parameters مش مجرد مكان نحط فيه أي بيانات

فكر فيها كده:

```text
GET /products
       │
       ├── page=2       → Pagination
       ├── limit=10     → Pagination
       ├── category=tv  → Filtering
       └── sort=price   → Sorting
```

كلهم بيقولوا للسيرفر:

> **إزاي أريد البيانات؟**

وليس:

> **ما هي البيانات التي أريد إنشاءها؟**

وده فرق مهم جدًا.

---

## قاعدة تحفظها الآن

```text
GET
│
├── Path → أي Resource أريد؟
│
├── Query Params → كيف أريد البيانات؟
│
├── Headers → معلومات إضافية عن الطلب
│
└── Body → غالبًا لا نستخدمه مع GET
```

وبالنسبة لـ POST:

```text
POST
│
├── Path → أين أريد إنشاء الـ Resource؟
│
├── Headers → معلومات عن الطلب
│
└── Body → ما البيانات التي أريد إنشاءها؟
```

مثال:

```http
POST /api/products
Content-Type: application/json

{
  "name": "iPhone",
  "price": 50000
}
```

---

### والأهم في منهجنا

من هنا ورايح **مش هسألك Interview Question على Concept لسه ما شرحناهوش**.

هنمشي:

```text
شرح
 ↓
مثال
 ↓
تطبيق معايا
 ↓
تمرين
 ↓
Interview
```

مش:

```text
معلومة ناقصة
 ↓
سؤال Interview
 ↓
تخمين
```

**الخطوة التالية: أجب عن اختبار `POST /api/products` السابق بعد الشرح الجديد.**
</details>
بس انا عشان تجاوبك زي كده 
لأن page و limit بيحددوا طريقة استرجاع الـ collection، مثل pagination، sorting، filtering، والـ GET Request عادةً يستخدم Query Parameters لهذه المعايير بدل وضعها في Body.

انت مشرحتوش ده فأنا عجيبة منين
ChatGPT
**الهدف: ما تحكمش على إجابتك بمعلومة أنا ما شرحتهاش، وأنت معاك حق في النقطة دي.** الخطأ هنا مني في طريقة التدريب، لأنني سألتك عن مفهوم لم أدرّسه لك أولًا، والوقت لتصحيح مسار الدرس: **5 دقائق**.

1. **أنت كنت بتجاوب بناءً على اللي اتشرح فعلًا**

أنا شرحت:

```text
Query Parameters
?category=phones&page=2
```

وقلت إنها:

> Query Parameters

لكن **ما شرحتش لماذا** `page` و `limit` يعتبروا Query Parameters، ولا شرحت مفهوم:

```text
pagination
filtering
sorting
```

فطبيعي جدًا إنك ما تعرفش الإجابة.

2. **القاعدة اللي كان المفروض أشرحها**

خلينا نبنيها من الصفر.

عندك API:

```http
GET /api/products
```

أنت بتقول:

> هات المنتجات.

طيب لو عندي **10,000 منتج**؟

مش منطقي أطلبهم كلهم مرة واحدة.

أقول للسيرفر:

```http
GET /api/products?page=2&limit=10
```

معناها:

```text
page = 2
→ عايز الصفحة الثانية

limit = 10
→ عايز 10 منتجات
```

وده اسمه:

**Pagination**

---

3. ونفس الفكرة مع Filtering

مثلاً:

```http
GET /api/products?category=phones
```

معناها:

> هات المنتجات اللي category بتاعها phones.

ده اسمه:

**Filtering**

---

4. ومع Sorting

مثلاً:

```http
GET /api/products?sort=price
```

معناها:

> هات المنتجات مرتبة حسب السعر.

ده:

**Sorting**

---

5. إذن الـ Query Parameters مش مجرد مكان نحط فيه أي بيانات

فكر فيها كده:

```text
GET /products
       │
       ├── page=2       → Pagination
       ├── limit=10     → Pagination
       ├── category=tv  → Filtering
       └── sort=price   → Sorting
```

كلهم بيقولوا للسيرفر:

> **إزاي أريد البيانات؟**

وليس:

> **ما هي البيانات التي أريد إنشاءها؟**

وده فرق مهم جدًا.

---

## قاعدة تحفظها الآن

```text
GET
│
├── Path → أي Resource أريد؟
│
├── Query Params → كيف أريد البيانات؟
│
├── Headers → معلومات إضافية عن الطلب
│
└── Body → غالبًا لا نستخدمه مع GET
```

وبالنسبة لـ POST:

```text
POST
│
├── Path → أين أريد إنشاء الـ Resource؟
│
├── Headers → معلومات عن الطلب
│
└── Body → ما البيانات التي أريد إنشاءها؟
```

مثال:

```http
POST /api/products
Content-Type: application/json

{
  "name": "iPhone",
  "price": 50000
}
```

---

### والأهم في منهجنا

من هنا ورايح **مش هسألك Interview Question على Concept لسه ما شرحناهوش**.

هنمشي:

```text
شرح
 ↓
مثال
 ↓
تطبيق معايا
 ↓
تمرين
 ↓
Interview
```

مش:

```text
معلومة ناقصة
 ↓
سؤال Interview
 ↓
تخمين
```

**الخطوة التالية: أجب عن اختبار `POST /api/products` السابق بعد الشرح الجديد.**
<details>
<summary>مش عاوزك تتوهنا احنا مشييين واحده واحده</summary>
user
مش عاوزك تتوهنا احنا مشييين واحده واحده
ChatGPT
**الهدف: نمشي في Axios خطوة خطوة، بدون قفز لمفاهيم لسه ما اتشرحتش.** من هنا هنلتزم بالترتيب، وكل درس لن ننتقل منه إلا بعد ما تفهمه وتطبقه.

1. **Lesson 1 فقط الآن**
   - HTTP Request
   - HTTP Response
   - Method
   - URL
   - Headers
   - Body
   - Query Parameters
   - Status Code

2. **طريقة كل درس**
   ```text
   شرح مفهوم واحد
   ↓
   مثال
   ↓
   أسألك سؤال بسيط على نفس اللي شرحته
   ↓
   تصحح
   ↓
   ننتقل للنقطة التالية
   ```

3. **ممنوع القفز حاليًا**
   
   مش هندخل دلوقتي:
   - Interceptors
   - Authentication architecture
   - TanStack Query
   - Feature-Based Architecture
   - Storybook
   - Refresh Token
   
   دي كلها موجودة في الـ roadmap، لكن **كل واحدة في وقتها**.

4. وبالنسبة لإجابتك السابقة، نعتبرها **مقبولة بناءً على المعلومات اللي كانت متاحة لك**. مش هحاسبك على حاجة ما شرحتهاش.

**الخطوة التالية: نكمل Lesson 1 من النقطة اللي وقفنا عندها، وهي `HTTP Methods` فقط.**
</details>
مش عاوزك تتوهنا احنا مشييين واحده واحده
ChatGPT
**الهدف: نمشي في Axios خطوة خطوة، بدون قفز لمفاهيم لسه ما اتشرحتش.** من هنا هنلتزم بالترتيب، وكل درس لن ننتقل منه إلا بعد ما تفهمه وتطبقه.

1. **Lesson 1 فقط الآن**
   - HTTP Request
   - HTTP Response
   - Method
   - URL
   - Headers
   - Body
   - Query Parameters
   - Status Code

2. **طريقة كل درس**
   ```text
   شرح مفهوم واحد
   ↓
   مثال
   ↓
   أسألك سؤال بسيط على نفس اللي شرحته
   ↓
   تصحح
   ↓
   ننتقل للنقطة التالية
   ```

3. **ممنوع القفز حاليًا**
   
   مش هندخل دلوقتي:
   - Interceptors
   - Authentication architecture
   - TanStack Query
   - Feature-Based Architecture
   - Storybook
   - Refresh Token
   
   دي كلها موجودة في الـ roadmap، لكن **كل واحدة في وقتها**.

4. وبالنسبة لإجابتك السابقة، نعتبرها **مقبولة بناءً على المعلومات اللي كانت متاحة لك**. مش هحاسبك على حاجة ما شرحتهاش.

**الخطوة التالية: نكمل Lesson 1 من النقطة اللي وقفنا عندها، وهي `HTTP Methods` فقط.**
<details>
<summary>كمل</summary>
user
كمل
ChatGPT
**الهدف: نكمل Lesson 1 فقط، ونفهم HTTP Methods قبل ما نلمس Axios.** الوقت: **10 دقائق**.

## Step 1 of 4: HTTP Methods

الـ **HTTP Method** بتحدد للسيرفر أنت عايز تعمل إيه في الـ Request.

أهم 5 Methods:

| Method | المعنى البسيط |
|---|---|
| `GET` | هات بيانات |
| `POST` | أنشئ بيانات |
| `PUT` | استبدل بيانات |
| `PATCH` | عدّل جزء من البيانات |
| `DELETE` | احذف بيانات |

### 1. GET

```http
GET /users
```

يعني:

> هات الـ users.

مثال:

```text
GET /products
```

يعني هات المنتجات.

---

### 2. POST

```http
POST /users
```

يعني:

> أنشئ User جديد.

والبيانات غالبًا بتكون في الـ Body:

```json
{
  "name": "Ahmed",
  "email": "ahmed@example.com"
}
```

---

### 3. PUT

هنا محتاج تفرق بينه وبين PATCH.

```http
PUT /users/10
```

الفكرة الأساسية:

> استبدل تمثيل الـ User رقم 10 بالبيانات الجديدة.

مثلاً:

```json
{
  "name": "Ahmed",
  "email": "new@example.com",
  "age": 30
}
```

يعني أنت بتتعامل مع الـ Resource ككل.

---

### 4. PATCH

```http
PATCH /users/10
```

يعني:

> عدّل جزء من الـ User.

مثلاً عايز تغير الاسم فقط:

```json
{
  "name": "Mohamed"
}
```

مش لازم تبعت باقي البيانات.

**الفرق المختصر:**

```text
PUT
→ استبدال الـ Resource

PATCH
→ تعديل جزء من الـ Resource
```

---

### 5. DELETE

```http
DELETE /users/10
```

يعني:

> احذف User رقم 10.

غالبًا مش محتاج Body.

---

# الصورة اللي عايزك تحفظها

```text
GET
↓
Read

POST
↓
Create

PUT
↓
Replace

PATCH
↓
Partial Update

DELETE
↓
Delete
```

وده مرتبط جدًا بالـ CRUD:

```text
Create → POST
Read   → GET
Update → PUT / PATCH
Delete → DELETE
```

## 🧪 تدريب بسيط جدًا

عندك `/products`.

عايز تعمل العمليات التالية:

1. تجيب كل المنتجات
2. تضيف منتج جديد
3. تغير سعر المنتج رقم 5 فقط
4. تحذف المنتج رقم 5

اكتب الـ Method لكل واحدة فقط.

**الخطوة التالية: اكتب الأربع Methods بالترتيب.**
</details>
كمل
ChatGPT
**الهدف: نكمل Lesson 1 فقط، ونفهم HTTP Methods قبل ما نلمس Axios.** الوقت: **10 دقائق**.

## Step 1 of 4: HTTP Methods

الـ **HTTP Method** بتحدد للسيرفر أنت عايز تعمل إيه في الـ Request.

أهم 5 Methods:

| Method | المعنى البسيط |
|---|---|
| `GET` | هات بيانات |
| `POST` | أنشئ بيانات |
| `PUT` | استبدل بيانات |
| `PATCH` | عدّل جزء من البيانات |
| `DELETE` | احذف بيانات |

### 1. GET

```http
GET /users
```

يعني:

> هات الـ users.

مثال:

```text
GET /products
```

يعني هات المنتجات.

---

### 2. POST

```http
POST /users
```

يعني:

> أنشئ User جديد.

والبيانات غالبًا بتكون في الـ Body:

```json
{
  "name": "Ahmed",
  "email": "ahmed@example.com"
}
```

---

### 3. PUT

هنا محتاج تفرق بينه وبين PATCH.

```http
PUT /users/10
```

الفكرة الأساسية:

> استبدل تمثيل الـ User رقم 10 بالبيانات الجديدة.

مثلاً:

```json
{
  "name": "Ahmed",
  "email": "new@example.com",
  "age": 30
}
```

يعني أنت بتتعامل مع الـ Resource ككل.

---

### 4. PATCH

```http
PATCH /users/10
```

يعني:

> عدّل جزء من الـ User.

مثلاً عايز تغير الاسم فقط:

```json
{
  "name": "Mohamed"
}
```

مش لازم تبعت باقي البيانات.

**الفرق المختصر:**

```text
PUT
→ استبدال الـ Resource

PATCH
→ تعديل جزء من الـ Resource
```

---

### 5. DELETE

```http
DELETE /users/10
```

يعني:

> احذف User رقم 10.

غالبًا مش محتاج Body.

---

# الصورة اللي عايزك تحفظها

```text
GET
↓
Read

POST
↓
Create

PUT
↓
Replace

PATCH
↓
Partial Update

DELETE
↓
Delete
```

وده مرتبط جدًا بالـ CRUD:

```text
Create → POST
Read   → GET
Update → PUT / PATCH
Delete → DELETE
```

## 🧪 تدريب بسيط جدًا

عندك `/products`.

عايز تعمل العمليات التالية:

1. تجيب كل المنتجات
2. تضيف منتج جديد
3. تغير سعر المنتج رقم 5 فقط
4. تحذف المنتج رقم 5

اكتب الـ Method لكل واحدة فقط.

**الخطوة التالية: اكتب الأربع Methods بالترتيب.**
<details>
<summary>كمل</summary>
user
كمل 
ChatGPT
**الهدف: نكمل Lesson 1 فقط، ونثبت HTTP Methods قبل الانتقال لأي جزء جديد.** بما إنك قلت كمل، هنعتبر التدريب السابق مفهوم وننتقل للنقطة التالية. الوقت: **10 دقائق**.

## Step 2 of 4: URL

الـ **URL** هو العنوان اللي الـ Request هيروح له.

مثال:

```http
GET https://api.example.com/users
```

نقسمه:

```text
https://api.example.com/users
│       │               │
│       │               └── Path
│       └────────────────── Host
└────────────────────────── Protocol
```

### 1. Protocol

```text
https://
```

ده بيحدد طريقة الاتصال.

في APIs الحديثة غالبًا هنستخدم:

```text
HTTPS
```

---

### 2. Host

```text
api.example.com
```

ده عنوان السيرفر.

في مشروع React حقيقي ممكن يكون عندك:

```text
https://api.myapp.com
```

---

### 3. Path

```text
/users
```

ده بيحدد الـ Resource اللي بتتعامل معاه.

مثلاً:

```text
/users
/products
/orders
```

ولو User معين:

```text
/users/15
```

هنا:

```text
/users
```

هو نوع الـ Resource.

و:

```text
15
```

هو الـ ID.

---

## نقطة مهمة

شوف الفرق:

```http
GET /users
```

معناه:

> تعامل مع مجموعة الـ Users.

لكن:

```http
GET /users/15
```

معناه:

> هات User محدد، ID بتاعه 15.

يعني الـ URL نفسه ممكن يحدد **أي Resource** أنت بتتعامل معاه.

---

# Step 3 of 4: Headers

الـ Headers عبارة عن معلومات إضافية عن الـ Request.

مثال:

```http
GET /users
Authorization: Bearer abc123
Accept: application/json
```

عندنا هنا Header اسمه:

```text
Authorization
```

وقيمته:

```text
Bearer abc123
```

وHeader تاني:

```text
Accept
```

وقيمته:

```text
application/json
```

خد بالك:

```text
Headers ≠ Body
```

الـ Headers معلومات عن الطلب.

الـ Body هو البيانات نفسها التي يتم إرسالها، عندما يكون للطلب Body.

---

# Step 4 of 4: Body

الـ Body هو البيانات التي يرسلها الـ Client إلى السيرفر.

مثلاً:

```http
POST /users
Content-Type: application/json

{
  "name": "Ahmed",
  "email": "ahmed@example.com"
}
```

هنا:

```text
POST
↓
Method

/users
↓
URL Path

Content-Type
↓
Header

{
  "name": "...",
  "email": "..."
}
↓
Body
```

ودي أهم صورة في Lesson 1.

---

# 🧠 اربط الأربع أجزاء

لما تشوف Request:

```http
POST /users
Authorization: Bearer xxx
Content-Type: application/json

{
  "name": "Ahmed"
}
```

اقرأه كده:

```text
POST
↓
أنا عايز أعمل عملية إنشاء

/users
↓
على Resource اسمه Users

Authorization
↓
معايا بيانات صلاحية

Content-Type
↓
الـ Body بتاعي JSON

Body
↓
دي البيانات اللي عايز أبعتها
```

## 🎯 اختبار Lesson 1

مش Interview، ومش هجيب حاجة جديدة.

عندك:

```http
PATCH /products/10
Authorization: Bearer abc123
Content-Type: application/json

{
  "price": 500
}
```

حدد فقط:

```text
Method =
Path =
Headers =
Body =
```

**الخطوة التالية: اكتب الأربع إجابات من الـ Request اللي فوق.**
</details>
كمل 
ChatGPT
**الهدف: نكمل Lesson 1 فقط، ونثبت HTTP Methods قبل الانتقال لأي جزء جديد.** بما إنك قلت كمل، هنعتبر التدريب السابق مفهوم وننتقل للنقطة التالية. الوقت: **10 دقائق**.

## Step 2 of 4: URL

الـ **URL** هو العنوان اللي الـ Request هيروح له.

مثال:

```http
GET https://api.example.com/users
```

نقسمه:

```text
https://api.example.com/users
│       │               │
│       │               └── Path
│       └────────────────── Host
└────────────────────────── Protocol
```

### 1. Protocol

```text
https://
```

ده بيحدد طريقة الاتصال.

في APIs الحديثة غالبًا هنستخدم:

```text
HTTPS
```

---

### 2. Host

```text
api.example.com
```

ده عنوان السيرفر.

في مشروع React حقيقي ممكن يكون عندك:

```text
https://api.myapp.com
```

---

### 3. Path

```text
/users
```

ده بيحدد الـ Resource اللي بتتعامل معاه.

مثلاً:

```text
/users
/products
/orders
```

ولو User معين:

```text
/users/15
```

هنا:

```text
/users
```

هو نوع الـ Resource.

و:

```text
15
```

هو الـ ID.

---

## نقطة مهمة

شوف الفرق:

```http
GET /users
```

معناه:

> تعامل مع مجموعة الـ Users.

لكن:

```http
GET /users/15
```

معناه:

> هات User محدد، ID بتاعه 15.

يعني الـ URL نفسه ممكن يحدد **أي Resource** أنت بتتعامل معاه.

---

# Step 3 of 4: Headers

الـ Headers عبارة عن معلومات إضافية عن الـ Request.

مثال:

```http
GET /users
Authorization: Bearer abc123
Accept: application/json
```

عندنا هنا Header اسمه:

```text
Authorization
```

وقيمته:

```text
Bearer abc123
```

وHeader تاني:

```text
Accept
```

وقيمته:

```text
application/json
```

خد بالك:

```text
Headers ≠ Body
```

الـ Headers معلومات عن الطلب.

الـ Body هو البيانات نفسها التي يتم إرسالها، عندما يكون للطلب Body.

---

# Step 4 of 4: Body

الـ Body هو البيانات التي يرسلها الـ Client إلى السيرفر.

مثلاً:

```http
POST /users
Content-Type: application/json

{
  "name": "Ahmed",
  "email": "ahmed@example.com"
}
```

هنا:

```text
POST
↓
Method

/users
↓
URL Path

Content-Type
↓
Header

{
  "name": "...",
  "email": "..."
}
↓
Body
```

ودي أهم صورة في Lesson 1.

---

# 🧠 اربط الأربع أجزاء

لما تشوف Request:

```http
POST /users
Authorization: Bearer xxx
Content-Type: application/json

{
  "name": "Ahmed"
}
```

اقرأه كده:

```text
POST
↓
أنا عايز أعمل عملية إنشاء

/users
↓
على Resource اسمه Users

Authorization
↓
معايا بيانات صلاحية

Content-Type
↓
الـ Body بتاعي JSON

Body
↓
دي البيانات اللي عايز أبعتها
```

## 🎯 اختبار Lesson 1

مش Interview، ومش هجيب حاجة جديدة.

عندك:

```http
PATCH /products/10
Authorization: Bearer abc123
Content-Type: application/json

{
  "price": 500
}
```

حدد فقط:

```text
Method =
Path =
Headers =
Body =
```

**الخطوة التالية: اكتب الأربع إجابات من الـ Request اللي فوق.**
<details>
<summary>الدرس ٢</summary>
user
الدرس ٢
ChatGPT
</details>
الدرس ٢
ChatGPT