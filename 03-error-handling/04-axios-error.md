# AxiosError

دلوقتي عندنا سؤال مهم:

**لما Axios يحصل له Error، إيه شكل الـ `error` اللي بيوصل للـ `catch`؟**

مثلاً:

```ts
try {
  const response = await axios.get("/users");
} catch (error) {
  console.log(error);
}
```

الـ `error` هنا ممكن يكون **`AxiosError`**.

يعني Axios عنده Error object خاص بيه، يحتوي على معلومات تساعدنا نعرف **إيه اللي حصل**.

بشكل مبسط، ممكن يكون عنده:

```text
AxiosError
├── message
├── response
├── request
├── config
└── code
```

والأهم بالنسبة لنا حاليًا:

```text
response
request
```

لأنهم بيساعدونا نفرق بين الحالات.

### HTTP Error

مثلاً `404`:

```text
AxiosError
   │
   ├── response ✅
   │     └── status = 404
   │
   └── request
```

عندنا `response` لأن السيرفر **رد**.

### Network Error

```text
AxiosError
   │
   ├── response ❌
   │
   └── request
```

مفيش `response` بالشكل المعتاد لأننا لم نحصل على HTTP response من السيرفر.

---

وهنا هنبدأ نفهم كود مهم جدًا:

```ts
if (axios.isAxiosError(error)) {
  // error is AxiosError
}
```

لكن **مش هنشرحه بالتفصيل لسه**.

الأول لازم نفهم كل property في `AxiosError` واحدة واحدة، وبعدها نستخدم `axios.isAxiosError()` بشكل صحيح.

**الخطوة التالية:** نبدأ بتفكيك `AxiosError`، وأول property هنشرحها هي `error.response`.
<details>
<summary>كمل</summary>
user
كمل
ChatGPT
**خد 10 دقائق: افهم `error.response` أولًا، لأنه أهم property في `AxiosError` لما يحصل `HTTP Error`.**

## 📍 Roadmap

```text
SECTION 0  Architecture Foundation       ⬜
SECTION 1  Axios Fundamentals             ✅
SECTION 2  TypeScript + Axios              ✅
SECTION 3  Error Handling                  🟢 CURRENT
  ├─ Error Handling Fundamentals           ✅
  ├─ HTTP Errors                           ✅
  ├─ Network Errors                        ✅
  ├─ AxiosError
  │    ├─ error.response                   🟡 ← نحن هنا
  │    ├─ error.request                    ⬜
  │    ├─ error.message                    ⬜
  │    └─ error.code                       ⬜
  ├─ Error Normalization                   ⬜
  └─ Custom ApiError                       ⬜
SECTION 4  Interceptors                    ⬜
...
```

# `error.response`

لما يحصل `HTTP Error`، السيرفر **رد فعلًا**.

مثلاً:

```text id="z6h8l3"
GET /users/999
        ↓
      Server
        ↓
       404
```

Axios بيحط الـ HTTP response داخل:

```ts id="m8x2q1"
error.response
```

يعني لو عندنا:

```ts id="j4w7ka"
try {
  await axios.get("/users/999");
} catch (error) {
  console.log(error.response);
}
```

فـ `error.response` ممكن تحتوي على:

```text id="q8r5sd"
response
├── data
├── status
├── statusText
├── headers
└── config
```

لاحظ حاجة مهمة جدًا:

إحنا قبل كده كان عندنا:

```ts id="1x7kz9"
const response = await axios.get(...)
```

وهنا عندنا:

```ts id="n2v6ca"
error.response
```

**الاتنين Axios Response.**

الفرق إن الأول حصل في حالة النجاح، والثاني موجود داخل الـ Error لأن الطلب انتهى بـ HTTP status يعتبره Axios فشلًا.

---

# مثال

السيرفر رجع:

```json id="2p8f1a"
{
  "status": false,
  "code": 404,
  "message": "User not found"
}
```

في `catch` ممكن نوصل إلى:

```ts id="c5h9tr"
try {
  await axios.get("/users/999");
} catch (error) {
  console.log(error.response?.status);
}
```

فتكون:

```text id="r1m7qx"
error.response?.status
        ↓
       404
```

ولو عايزين البيانات اللي السيرفر رجعها:

```ts id="v4k2nm"
error.response?.data
```

فتكون:

```json id="g7p3wc"
{
  "status": false,
  "code": 404,
  "message": "User not found"
}
```

---

# ⚠️ ليه استخدمنا `?.`؟

دي نقطة مهمة جدًا.

إحنا قلنا قبل كده إن:

```text id="e3v8qa"
HTTP Error
→ response موجودة

Network Error
→ response قد لا تكون موجودة
```

فلو كتبت:

```ts id="n6k2pv"
error.response.status
```

ممكن يحصل:

```text
Cannot read properties of undefined
```

لكن:

```ts id="u8s4ld"
error.response?.status
```

معناها:

> لو `response` موجودة، هات `status`. لو مش موجودة، رجّع `undefined` بدل ما تعمل crash.

---

# 🧠 الصورة الكاملة

```text id="b3t7qx"
AxiosError
│
├── response
│    │
│    ├── status
│    ├── data
│    ├── headers
│    └── config
│
├── request
├── message
└── code
```

وفي حالة:

### HTTP Error

```text id="w5k9zn"
error.response
      ↓
   موجودة ✅
```

### Network Error

```text id="a2f6mr"
error.response
      ↓
   غير موجودة ❌
```

وده بالضبط سبب إن `error.response` مهمة جدًا.

---

## 🎯 تمرين

عندك:

```ts id="v9c3pa"
try {
  await axios.get("/users/999");
} catch (error) {
  console.log(error.response?.status);
  console.log(error.response?.data);
}
```

والسيرفر رجع:

```json id="k4n8xs"
{
  "status": false,
  "code": 404,
  "message": "User not found"
}
```

**إيه قيمة السطرين؟**

```ts id="j6q2rt"
error.response?.status

error.response?.data
```

**الخطوة التالية:** اكتب قيمة كل واحد فيهم.
</details>
كمل
ChatGPT
**خد 10 دقائق: افهم `error.response` أولًا، لأنه أهم property في `AxiosError` لما يحصل `HTTP Error`.**

## 📍 Roadmap

```text
SECTION 0  Architecture Foundation       ⬜
SECTION 1  Axios Fundamentals             ✅
SECTION 2  TypeScript + Axios              ✅
SECTION 3  Error Handling                  🟢 CURRENT
  ├─ Error Handling Fundamentals           ✅
  ├─ HTTP Errors                           ✅
  ├─ Network Errors                        ✅
  ├─ AxiosError
  │    ├─ error.response                   🟡 ← نحن هنا
  │    ├─ error.request                    ⬜
  │    ├─ error.message                    ⬜
  │    └─ error.code                       ⬜
  ├─ Error Normalization                   ⬜
  └─ Custom ApiError                       ⬜
SECTION 4  Interceptors                    ⬜
...
```

# `error.response`

لما يحصل `HTTP Error`، السيرفر **رد فعلًا**.

مثلاً:

```text id="z6h8l3"
GET /users/999
        ↓
      Server
        ↓
       404
```

Axios بيحط الـ HTTP response داخل:

```ts id="m8x2q1"
error.response
```

يعني لو عندنا:

```ts id="j4w7ka"
try {
  await axios.get("/users/999");
} catch (error) {
  console.log(error.response);
}
```

فـ `error.response` ممكن تحتوي على:

```text id="q8r5sd"
response
├── data
├── status
├── statusText
├── headers
└── config
```

لاحظ حاجة مهمة جدًا:

إحنا قبل كده كان عندنا:

```ts id="1x7kz9"
const response = await axios.get(...)
```

وهنا عندنا:

```ts id="n2v6ca"
error.response
```

**الاتنين Axios Response.**

الفرق إن الأول حصل في حالة النجاح، والثاني موجود داخل الـ Error لأن الطلب انتهى بـ HTTP status يعتبره Axios فشلًا.

---

# مثال

السيرفر رجع:

```json id="2p8f1a"
{
  "status": false,
  "code": 404,
  "message": "User not found"
}
```

في `catch` ممكن نوصل إلى:

```ts id="c5h9tr"
try {
  await axios.get("/users/999");
} catch (error) {
  console.log(error.response?.status);
}
```

فتكون:

```text id="r1m7qx"
error.response?.status
        ↓
       404
```

ولو عايزين البيانات اللي السيرفر رجعها:

```ts id="v4k2nm"
error.response?.data
```

فتكون:

```json id="g7p3wc"
{
  "status": false,
  "code": 404,
  "message": "User not found"
}
```

---

# ⚠️ ليه استخدمنا `?.`؟

دي نقطة مهمة جدًا.

إحنا قلنا قبل كده إن:

```text id="e3v8qa"
HTTP Error
→ response موجودة

Network Error
→ response قد لا تكون موجودة
```

فلو كتبت:

```ts id="n6k2pv"
error.response.status
```

ممكن يحصل:

```text
Cannot read properties of undefined
```

لكن:

```ts id="u8s4ld"
error.response?.status
```

معناها:

> لو `response` موجودة، هات `status`. لو مش موجودة، رجّع `undefined` بدل ما تعمل crash.

---

# 🧠 الصورة الكاملة

```text id="b3t7qx"
AxiosError
│
├── response
│    │
│    ├── status
│    ├── data
│    ├── headers
│    └── config
│
├── request
├── message
└── code
```

وفي حالة:

### HTTP Error

```text id="w5k9zn"
error.response
      ↓
   موجودة ✅
```

### Network Error

```text id="a2f6mr"
error.response
      ↓
   غير موجودة ❌
```

وده بالضبط سبب إن `error.response` مهمة جدًا.

---

## 🎯 تمرين

عندك:

```ts id="v9c3pa"
try {
  await axios.get("/users/999");
} catch (error) {
  console.log(error.response?.status);
  console.log(error.response?.data);
}
```

والسيرفر رجع:

```json id="k4n8xs"
{
  "status": false,
  "code": 404,
  "message": "User not found"
}
```

**إيه قيمة السطرين؟**

```ts id="j6q2rt"
error.response?.status

error.response?.data
```

**الخطوة التالية:** اكتب قيمة كل واحد فيهم.
<details>
<summary>والله أنت كتبت فوق إن الـ error response وبعدين بيتstatus دي معناها بتديك true أو false، فبتديك false، ولكن أنت هنا قلت إنها ترجع 404، فدي ما أعرفش إزاي، المفروض أقول دوت code عشان ترجع 404. أما بالنسبة للنقطة التانية error response، لو موجود، لو موجود الخطأ، في الـ response رجع الـ data، الـ data بقى اللي هي شاملة كل حاجة: الـ status والكود والـ message.</summary>
user
والله أنت كتبت فوق إن الـ error response وبعدين بيتstatus دي معناها بتديك true أو false، فبتديك false، ولكن أنت هنا قلت إنها ترجع 404، فدي ما أعرفش إزاي، المفروض أقول دوت code عشان ترجع 404. أما بالنسبة للنقطة التانية error response، لو موجود، لو موجود الخطأ، في الـ response رجع الـ data، الـ data بقى اللي هي شاملة كل حاجة: الـ status والكود والـ message.
ChatGPT
**خد 5 دقائق وثبّت الفرق بين `HTTP status` و`API status` و`API code`، لأنك اكتشفت نقطة مهمة جدًا في تصميم الـ API.**

أنت **صح في ملاحظتك**، وأنا كان لازم أوضح الفرق بشكل أدق. 👌

عندنا هنا **مستويين مختلفين تمامًا**:

### المستوى الأول: Axios / HTTP Response

```ts
error.response?.status
```

ده **HTTP status code**.

مثلاً:

```text
404
401
403
500
```

إذن:

```ts
error.response?.status
// 404
```

---

### المستوى الثاني: البيانات التي أرسلها الـ API

```ts
error.response?.data
```

دي البيانات الموجودة داخل الـ HTTP response.

مثلاً السيرفر ممكن يرجع:

```json
{
  "status": false,
  "code": 404,
  "message": "User not found"
}
```

إذن:

```ts
error.response?.data.status
// false
```

و:

```ts
error.response?.data.code
// 404
```

و:

```ts
error.response?.data.message
// "User not found"
```

---

## إذن عندنا 3 حاجات مختلفة

```text
error.response
        │
        ├── status
        │     ↓
        │    404
        │
        └── data
              │
              ├── status
              │     ↓
              │    false
              │
              ├── code
              │     ↓
              │    404
              │
              └── message
                    ↓
               "User not found"
```

وده مهم جدًا.

### `error.response.status`

**HTTP status**

```text
404
```

### `error.response.data.status`

**API status**

```text
false
```

### `error.response.data.code`

**API code**

```text
404
```

---

## وهنا أنت قلت حاجة صحيحة جدًا

أنت قلت إن:

> `data` شاملة الـ status والـ code والـ message.

بالضبط، **لو ده هو الـ API contract بتاع السيرفر**.

وعشان كده الـ TypeScript اللي أنت كنت بتعمله قبل كده مهم جدًا.

مثلاً:

```ts
interface IErrorResponse {
  status: false;
  code: number;
  message: string;
}
```

وبعد كده ممكن نخلي Axios يعرف إن:

```ts
error.response?.data
```

نوعها:

```ts
IErrorResponse
```

وده هيكون جزء مهم جدًا لما نوصل للـ `AxiosError<T>`.

---

### ⚠️ نقطة دقيقة

مش لازم الـ HTTP status والـ API `code` يكونوا نفس الرقم.

ممكن السيرفر يعمل:

```text
HTTP status = 400
API code     = 1001
```

لذلك **ماينفعش نفترض إن `code` = HTTP status**.

---

### 🧠 قاعدة تحفظها

```text
error.response.status
        ↓
HTTP

error.response.data
        ↓
API payload

error.response.data.status
        ↓
API business status
```

وأنت كده بدأت تلمس نقطة مهمة جدًا في الـ API architecture: **وجود أكثر من طبقة من الـ status information**.

**الخطوة التالية:** نكمل `AxiosError` مع `error.request` ونشوف ليه بنحتاجه رغم إن `error.response` ممكن تكون غير موجودة.
</details>
والله أنت كتبت فوق إن الـ error response وبعدين بيتstatus دي معناها بتديك true أو false، فبتديك false، ولكن أنت هنا قلت إنها ترجع 404، فدي ما أعرفش إزاي، المفروض أقول دوت code عشان ترجع 404. أما بالنسبة للنقطة التانية error response، لو موجود، لو موجود الخطأ، في الـ response رجع الـ data، الـ data بقى اللي هي شاملة كل حاجة: الـ status والكود والـ message.
ChatGPT
**خد 5 دقائق وثبّت الفرق بين `HTTP status` و`API status` و`API code`، لأنك اكتشفت نقطة مهمة جدًا في تصميم الـ API.**

أنت **صح في ملاحظتك**، وأنا كان لازم أوضح الفرق بشكل أدق. 👌

عندنا هنا **مستويين مختلفين تمامًا**:

### المستوى الأول: Axios / HTTP Response

```ts
error.response?.status
```

ده **HTTP status code**.

مثلاً:

```text
404
401
403
500
```

إذن:

```ts
error.response?.status
// 404
```

---

### المستوى الثاني: البيانات التي أرسلها الـ API

```ts
error.response?.data
```

دي البيانات الموجودة داخل الـ HTTP response.

مثلاً السيرفر ممكن يرجع:

```json
{
  "status": false,
  "code": 404,
  "message": "User not found"
}
```

إذن:

```ts
error.response?.data.status
// false
```

و:

```ts
error.response?.data.code
// 404
```

و:

```ts
error.response?.data.message
// "User not found"
```

---

## إذن عندنا 3 حاجات مختلفة

```text
error.response
        │
        ├── status
        │     ↓
        │    404
        │
        └── data
              │
              ├── status
              │     ↓
              │    false
              │
              ├── code
              │     ↓
              │    404
              │
              └── message
                    ↓
               "User not found"
```

وده مهم جدًا.

### `error.response.status`

**HTTP status**

```text
404
```

### `error.response.data.status`

**API status**

```text
false
```

### `error.response.data.code`

**API code**

```text
404
```

---

## وهنا أنت قلت حاجة صحيحة جدًا

أنت قلت إن:

> `data` شاملة الـ status والـ code والـ message.

بالضبط، **لو ده هو الـ API contract بتاع السيرفر**.

وعشان كده الـ TypeScript اللي أنت كنت بتعمله قبل كده مهم جدًا.

مثلاً:

```ts
interface IErrorResponse {
  status: false;
  code: number;
  message: string;
}
```

وبعد كده ممكن نخلي Axios يعرف إن:

```ts
error.response?.data
```

نوعها:

```ts
IErrorResponse
```

وده هيكون جزء مهم جدًا لما نوصل للـ `AxiosError<T>`.

---

### ⚠️ نقطة دقيقة

مش لازم الـ HTTP status والـ API `code` يكونوا نفس الرقم.

ممكن السيرفر يعمل:

```text
HTTP status = 400
API code     = 1001
```

لذلك **ماينفعش نفترض إن `code` = HTTP status**.

---

### 🧠 قاعدة تحفظها

```text
error.response.status
        ↓
HTTP

error.response.data
        ↓
API payload

error.response.data.status
        ↓
API business status
```

وأنت كده بدأت تلمس نقطة مهمة جدًا في الـ API architecture: **وجود أكثر من طبقة من الـ status information**.

**الخطوة التالية:** نكمل `AxiosError` مع `error.request` ونشوف ليه بنحتاجه رغم إن `error.response` ممكن تكون غير موجودة.
<details>
<summary>كمل</summary>
user
كمل
ChatGPT
**خد 10 دقائق: افهم `error.request` كالمستوى الثاني داخل `AxiosError`، وبعده هنقدر نفرّق عمليًا بين `response` و`request`.**

## 📍 Roadmap

```text
SECTION 0  Architecture Foundation       ⬜
SECTION 1  Axios Fundamentals             ✅
SECTION 2  TypeScript + Axios              ✅
SECTION 3  Error Handling                 🟢 CURRENT
  ├─ Error Handling Fundamentals          ✅
  ├─ HTTP Errors                          ✅
  ├─ Network Errors                       ✅
  ├─ AxiosError
  │    ├─ error.response                   ✅
  │    ├─ error.request                    🟡 ← نحن هنا
  │    ├─ error.message                    ⬜
  │    └─ error.code                       ⬜
  ├─ Error Normalization                   ⬜
  └─ Custom ApiError                       ⬜
SECTION 4  Interceptors                    ⬜
...
```

# `error.request`

إحنا قلنا إن `AxiosError` ممكن يكون فيه:

```text id="y6k8mp"
AxiosError
├── response
├── request
├── message
└── code
```

وفهمنا إن:

```ts id="d8m3sx"
error.response
```

بتكون موجودة لما **السيرفر رد علينا**.

طيب لو حصل Network Error؟

هنا ممكن يكون:

```ts id="a4q7nz"
error.response === undefined
```

لكن Axios قد يكون عنده معلومات عن **الـ request الذي تم إرساله**.

وهنا يأتي:

```ts id="m5v2kr"
error.request
```

---

# تخيل العملية

```text id="k9s4wd"
Client
   │
   │ Request
   ▼
Network
   │
   X
   │
   ❌ Server Response لم يصل
```

في الحالة دي:

```text id="x3j8qa"
error.response
      ↓
   undefined
```

لكن:

```text id="n7f2bc"
error.request
      ↓
   موجودة
```

بمعنى إن Axios عنده معلومات عن **الطلب الذي حاول إرساله**.

---

# قارن بين الحالتين

## الحالة الأولى: HTTP Error

مثلاً:

```text id="q2m8vz"
GET /users/999
       ↓
    Server
       ↓
     404
```

هنا:

```ts id="h6r4yt"
error.response
```

موجودة.

لأن السيرفر رد.

---

## الحالة الثانية: Network Error

مثلاً الإنترنت فصل:

```text id="k3p9sa"
GET /users/999
       ↓
    Network
       ↓
       ❌
```

هنا:

```ts id="j8c5nx"
error.response
```

مش موجودة.

لكن ممكن يكون:

```ts id="v4m7qd"
error.request
```

موجود.

---

# ⚠️ نقطة مهمة

ما تحفظش إن:

```text
error.request = Network Error
```

لأ.

`error.request` **مش هو الـ Error نفسه**.

هو object يمثل الـ request الذي تم إرساله أو محاولة إرساله.

يعني:

```text id="b9k2wf"
error
  ↓
AxiosError
  │
  ├── response → server response
  │
  └── request  → request information
```

---

# طيب ليه نحتاج `request`؟

لأن أحيانًا نحتاج نعرف:

> هل الطلب وصل لمرحلة إرسال request لكن لم نحصل على response؟

وده يساعدنا في تشخيص نوع المشكلة.

لكن في المرحلة دي **مش مطلوب منك تحفظ تفاصيل `XMLHttpRequest` أو Node.js request objects**.

هنخليها concept فقط.

---

# الصورة التي يجب أن تثبت الآن

```text id="c5v9mz"
                AxiosError
                    │
          ┌─────────┴─────────┐
          ↓                   ↓
     response              request
          │                   │
          ↓                   ↓
  Server Response       Request Info
          │
          ↓
      HTTP Status
```

مثلاً:

```text id="t1w6hr"
HTTP Error
    ↓
response موجودة
```

بينما:

```text id="s8q3ka"
Network Error
    ↓
response غير موجودة
    ↓
request قد تكون موجودة
```

---

## 🎯 سؤال Interview

عندك:

```ts id="f2x7md"
try {
  await axios.get("/users");
} catch (error) {
  console.log(error.response);
  console.log(error.request);
}
```

لو **السيرفر رد بـ `404`**:

أي واحدة تتوقع تكون موجودة؟

```text
error.response
error.request
```

وليه؟

**الخطوة التالية:** جاوب من فهمك، وبعدها ننتقل إلى `error.message`.
</details>
كمل
ChatGPT
**خد 10 دقائق: افهم `error.request` كالمستوى الثاني داخل `AxiosError`، وبعده هنقدر نفرّق عمليًا بين `response` و`request`.**

## 📍 Roadmap

```text
SECTION 0  Architecture Foundation       ⬜
SECTION 1  Axios Fundamentals             ✅
SECTION 2  TypeScript + Axios              ✅
SECTION 3  Error Handling                 🟢 CURRENT
  ├─ Error Handling Fundamentals          ✅
  ├─ HTTP Errors                          ✅
  ├─ Network Errors                       ✅
  ├─ AxiosError
  │    ├─ error.response                   ✅
  │    ├─ error.request                    🟡 ← نحن هنا
  │    ├─ error.message                    ⬜
  │    └─ error.code                       ⬜
  ├─ Error Normalization                   ⬜
  └─ Custom ApiError                       ⬜
SECTION 4  Interceptors                    ⬜
...
```

# `error.request`

إحنا قلنا إن `AxiosError` ممكن يكون فيه:

```text id="y6k8mp"
AxiosError
├── response
├── request
├── message
└── code
```

وفهمنا إن:

```ts id="d8m3sx"
error.response
```

بتكون موجودة لما **السيرفر رد علينا**.

طيب لو حصل Network Error؟

هنا ممكن يكون:

```ts id="a4q7nz"
error.response === undefined
```

لكن Axios قد يكون عنده معلومات عن **الـ request الذي تم إرساله**.

وهنا يأتي:

```ts id="m5v2kr"
error.request
```

---

# تخيل العملية

```text id="k9s4wd"
Client
   │
   │ Request
   ▼
Network
   │
   X
   │
   ❌ Server Response لم يصل
```

في الحالة دي:

```text id="x3j8qa"
error.response
      ↓
   undefined
```

لكن:

```text id="n7f2bc"
error.request
      ↓
   موجودة
```

بمعنى إن Axios عنده معلومات عن **الطلب الذي حاول إرساله**.

---

# قارن بين الحالتين

## الحالة الأولى: HTTP Error

مثلاً:

```text id="q2m8vz"
GET /users/999
       ↓
    Server
       ↓
     404
```

هنا:

```ts id="h6r4yt"
error.response
```

موجودة.

لأن السيرفر رد.

---

## الحالة الثانية: Network Error

مثلاً الإنترنت فصل:

```text id="k3p9sa"
GET /users/999
       ↓
    Network
       ↓
       ❌
```

هنا:

```ts id="j8c5nx"
error.response
```

مش موجودة.

لكن ممكن يكون:

```ts id="v4m7qd"
error.request
```

موجود.

---

# ⚠️ نقطة مهمة

ما تحفظش إن:

```text
error.request = Network Error
```

لأ.

`error.request` **مش هو الـ Error نفسه**.

هو object يمثل الـ request الذي تم إرساله أو محاولة إرساله.

يعني:

```text id="b9k2wf"
error
  ↓
AxiosError
  │
  ├── response → server response
  │
  └── request  → request information
```

---

# طيب ليه نحتاج `request`؟

لأن أحيانًا نحتاج نعرف:

> هل الطلب وصل لمرحلة إرسال request لكن لم نحصل على response؟

وده يساعدنا في تشخيص نوع المشكلة.

لكن في المرحلة دي **مش مطلوب منك تحفظ تفاصيل `XMLHttpRequest` أو Node.js request objects**.

هنخليها concept فقط.

---

# الصورة التي يجب أن تثبت الآن

```text id="c5v9mz"
                AxiosError
                    │
          ┌─────────┴─────────┐
          ↓                   ↓
     response              request
          │                   │
          ↓                   ↓
  Server Response       Request Info
          │
          ↓
      HTTP Status
```

مثلاً:

```text id="t1w6hr"
HTTP Error
    ↓
response موجودة
```

بينما:

```text id="s8q3ka"
Network Error
    ↓
response غير موجودة
    ↓
request قد تكون موجودة
```

---

## 🎯 سؤال Interview

عندك:

```ts id="f2x7md"
try {
  await axios.get("/users");
} catch (error) {
  console.log(error.response);
  console.log(error.request);
}
```

لو **السيرفر رد بـ `404`**:

أي واحدة تتوقع تكون موجودة؟

```text
error.response
error.request
```

وليه؟

**الخطوة التالية:** جاوب من فهمك، وبعدها ننتقل إلى `error.message`.
<details>
<summary>اللي موجودة الـerror response لأنه جاب الرد إن الـerror response status جاب 404، فهنا وصل للسيرفر، فده هيكون HTTP error وليس network error.</summary>
user
اللي موجودة الـerror response لأنه جاب الرد إن الـerror response status جاب 404، فهنا وصل للسيرفر، فده هيكون HTTP error وليس network error.
ChatGPT
**إجابتك 10/10، وفعلاً `error.response` هي النقطة الأساسية هنا.** ✅

عند:

```text id="p7x3lm"
Client
  ↓
Request
  ↓
Server
  ↓
404
  ↓
AxiosError
```

يبقى:

```ts id="s9f2kc"
error.response
```

**موجودة** لأن السيرفر رد.

ومن خلالها نقدر نوصل إلى:

```ts id="w3n8qa"
error.response.status
// 404
```

وكمان:

```ts id="j6v4rs"
error.response.data
```

اللي تحتوي على الـ API response اللي السيرفر رجعه.

### ملاحظة صغيرة

مش محتاج نقول إن `error.request` "مش موجودة" في حالة `404`.

الأدق إن **`error.response` موجودة لأننا حصلنا على response من السيرفر**. وجود `request` من عدمه مش هو معيار التفريق الأساسي هنا.

المعيار اللي عايزين تثبته:

```text id="f2m8vx"
response موجودة
        ↓
HTTP response وصل
        ↓
نقدر نقرأ status
```

أما:

```text id="q5r1az"
response غير موجودة
        ↓
لم نحصل على HTTP response
        ↓
نبحث في أسباب أخرى مثل Network / Timeout / Abort
```

## 📍 Roadmap

```text
SECTION 3  Error Handling                 🟢
  ├─ Error Handling Fundamentals          ✅
  ├─ HTTP Errors                          ✅
  ├─ Network Errors                       ✅
  ├─ AxiosError
  │    ├─ error.response                   ✅
  │    ├─ error.request                    ✅
  │    ├─ error.message                    🟡 ← التالي
  │    └─ error.code                       ⬜
  ├─ Error Normalization                   ⬜
  └─ Custom ApiError                      ⬜
```

**الخطوة التالية:** نبدأ `error.message` ونفهم الفرق بين الـ `message` وبين `error.response.data.message`.
</details>
اللي موجودة الـerror response لأنه جاب الرد إن الـerror response status جاب 404، فهنا وصل للسيرفر، فده هيكون HTTP error وليس network error.
ChatGPT
**إجابتك 10/10، وفعلاً `error.response` هي النقطة الأساسية هنا.** ✅

عند:

```text id="p7x3lm"
Client
  ↓
Request
  ↓
Server
  ↓
404
  ↓
AxiosError
```

يبقى:

```ts id="s9f2kc"
error.response
```

**موجودة** لأن السيرفر رد.

ومن خلالها نقدر نوصل إلى:

```ts id="w3n8qa"
error.response.status
// 404
```

وكمان:

```ts id="j6v4rs"
error.response.data
```

اللي تحتوي على الـ API response اللي السيرفر رجعه.

### ملاحظة صغيرة

مش محتاج نقول إن `error.request` "مش موجودة" في حالة `404`.

الأدق إن **`error.response` موجودة لأننا حصلنا على response من السيرفر**. وجود `request` من عدمه مش هو معيار التفريق الأساسي هنا.

المعيار اللي عايزين تثبته:

```text id="f2m8vx"
response موجودة
        ↓
HTTP response وصل
        ↓
نقدر نقرأ status
```

أما:

```text id="q5r1az"
response غير موجودة
        ↓
لم نحصل على HTTP response
        ↓
نبحث في أسباب أخرى مثل Network / Timeout / Abort
```

## 📍 Roadmap

```text
SECTION 3  Error Handling                 🟢
  ├─ Error Handling Fundamentals          ✅
  ├─ HTTP Errors                          ✅
  ├─ Network Errors                       ✅
  ├─ AxiosError
  │    ├─ error.response                   ✅
  │    ├─ error.request                    ✅
  │    ├─ error.message                    🟡 ← التالي
  │    └─ error.code                       ⬜
  ├─ Error Normalization                   ⬜
  └─ Custom ApiError                      ⬜
```

**الخطوة التالية:** نبدأ `error.message` ونفهم الفرق بين الـ `message` وبين `error.response.data.message`.
<details>
<summary>error.message</summary>
user
error.message
ChatGPT
**خد 7 دقائق: ثبّت الفرق بين `error.message` و`error.response.data.message`، لأن الاتنين ممكن يكون عندهم كلمة `message` لكن معناهم مختلف.**

## 📍 Roadmap

```text
SECTION 0  Architecture Foundation       ⬜
SECTION 1  Axios Fundamentals             ✅
SECTION 2  TypeScript + Axios              ✅
SECTION 3  Error Handling                 🟢 CURRENT
  ├─ Error Handling Fundamentals          ✅
  ├─ HTTP Errors                          ✅
  ├─ Network Errors                       ✅
  ├─ AxiosError
  │    ├─ error.response                   ✅
  │    ├─ error.request                    ✅
  │    ├─ error.message                    🟡 ← نحن هنا
  │    └─ error.code                       ⬜
  ├─ Error Normalization                   ⬜
  └─ Custom ApiError                       ⬜
SECTION 4  Interceptors                    ⬜
...
```

# `error.message`

لما يحصل Error، الـ `AxiosError` نفسه عنده property اسمها:

```ts id="q3h8kd"
error.message
```

دي بتوصف **سبب الخطأ من منظور Axios / العملية نفسها**.

مثلاً ممكن تكون رسالة مثل:

```text
Request failed with status code 404
```

أو في حالة Network Error ممكن تكون:

```text
Network Error
```

أو في Timeout ممكن تكون رسالة مرتبطة بالـ timeout.

---

## لكن عندنا `message` ثانية

تخيل إن السيرفر رجع:

```json id="a7k3pm"
{
  "status": false,
  "code": 404,
  "message": "User not found"
}
```

هنا عندنا:

```ts id="x5r9bc"
error.message
```

وممكن تكون:

```text
"Request failed with status code 404"
```

بينما:

```ts id="n2v6qa"
error.response?.data.message
```

تكون:

```text
"User not found"
```

**الاتنين مختلفين.**

---

# الفرق

```text id="m8f4zx"
error.message
        ↓
Axios Error Message
```

بينما:

```text id="r7k2wd"
error.response?.data.message
        ↓
Server/API Error Message
```

---

## مثال كامل

```ts id="h3q9vn"
try {
  await axios.get("/users/999");
} catch (error) {
  console.log(error.message);

  console.log(
    error.response?.data.message
  );
}
```

لو السيرفر رجع:

```json id="s6d1kp"
{
  "status": false,
  "code": 404,
  "message": "User not found"
}
```

فممكن نشوف:

```text id="c8w5mt"
error.message
→ "Request failed with status code 404"

error.response?.data.message
→ "User not found"
```

لاحظ:

**الأولى بتتكلم عن الـ HTTP/Axios failure.**

**الثانية بتتكلم عن الرسالة التي صممها الـ API للمستخدم أو للـ application.**

---

# طب في Network Error؟

هنا:

```text id="n4j7qx"
error.response
→ undefined
```

إذن:

```ts id="u2f8cs"
error.response?.data.message
```

مش هتنفع للحصول على رسالة السيرفر، لأننا أصلًا **ما عندناش response**.

لكن:

```ts id="k9m3vp"
error.message
```

قد تعطينا رسالة عن المشكلة، مثل:

```text
"Network Error"
```

وده أحد الأسباب اللي بتخلي `error.message` مفيدة في الـ Error Handling.

---

## 🧠 احفظها كده

```text id="q1v7bz"
error.message
        ↓
رسالة الخطأ الخاصة بالعملية / Axios

error.response?.data.message
        ↓
رسالة الخطأ التي أرسلها الـ API
```

وده فرق مهم جدًا، لأن بعد شوية لما نعمل **Error Normalization** هنقرر أي رسالة نعرضها للمستخدم وأي معلومات نستخدمها للـ debugging.

### 🎯 سؤال سريع

لو السيرفر رجع:

```json id="m5x8qa"
{
  "status": false,
  "code": 401,
  "message": "Token expired"
}
```

إيه الفرق بين:

```ts
error.message
```

و:

```ts
error.response?.data.message
```

**الخطوة التالية:** جاوب الفرق بين الاتنين من فهمك.
</details>
error.message
ChatGPT
**خد 7 دقائق: ثبّت الفرق بين `error.message` و`error.response.data.message`، لأن الاتنين ممكن يكون عندهم كلمة `message` لكن معناهم مختلف.**

## 📍 Roadmap

```text
SECTION 0  Architecture Foundation       ⬜
SECTION 1  Axios Fundamentals             ✅
SECTION 2  TypeScript + Axios              ✅
SECTION 3  Error Handling                 🟢 CURRENT
  ├─ Error Handling Fundamentals          ✅
  ├─ HTTP Errors                          ✅
  ├─ Network Errors                       ✅
  ├─ AxiosError
  │    ├─ error.response                   ✅
  │    ├─ error.request                    ✅
  │    ├─ error.message                    🟡 ← نحن هنا
  │    └─ error.code                       ⬜
  ├─ Error Normalization                   ⬜
  └─ Custom ApiError                       ⬜
SECTION 4  Interceptors                    ⬜
...
```

# `error.message`

لما يحصل Error، الـ `AxiosError` نفسه عنده property اسمها:

```ts id="q3h8kd"
error.message
```

دي بتوصف **سبب الخطأ من منظور Axios / العملية نفسها**.

مثلاً ممكن تكون رسالة مثل:

```text
Request failed with status code 404
```

أو في حالة Network Error ممكن تكون:

```text
Network Error
```

أو في Timeout ممكن تكون رسالة مرتبطة بالـ timeout.

---

## لكن عندنا `message` ثانية

تخيل إن السيرفر رجع:

```json id="a7k3pm"
{
  "status": false,
  "code": 404,
  "message": "User not found"
}
```

هنا عندنا:

```ts id="x5r9bc"
error.message
```

وممكن تكون:

```text
"Request failed with status code 404"
```

بينما:

```ts id="n2v6qa"
error.response?.data.message
```

تكون:

```text
"User not found"
```

**الاتنين مختلفين.**

---

# الفرق

```text id="m8f4zx"
error.message
        ↓
Axios Error Message
```

بينما:

```text id="r7k2wd"
error.response?.data.message
        ↓
Server/API Error Message
```

---

## مثال كامل

```ts id="h3q9vn"
try {
  await axios.get("/users/999");
} catch (error) {
  console.log(error.message);

  console.log(
    error.response?.data.message
  );
}
```

لو السيرفر رجع:

```json id="s6d1kp"
{
  "status": false,
  "code": 404,
  "message": "User not found"
}
```

فممكن نشوف:

```text id="c8w5mt"
error.message
→ "Request failed with status code 404"

error.response?.data.message
→ "User not found"
```

لاحظ:

**الأولى بتتكلم عن الـ HTTP/Axios failure.**

**الثانية بتتكلم عن الرسالة التي صممها الـ API للمستخدم أو للـ application.**

---

# طب في Network Error؟

هنا:

```text id="n4j7qx"
error.response
→ undefined
```

إذن:

```ts id="u2f8cs"
error.response?.data.message
```

مش هتنفع للحصول على رسالة السيرفر، لأننا أصلًا **ما عندناش response**.

لكن:

```ts id="k9m3vp"
error.message
```

قد تعطينا رسالة عن المشكلة، مثل:

```text
"Network Error"
```

وده أحد الأسباب اللي بتخلي `error.message` مفيدة في الـ Error Handling.

---

## 🧠 احفظها كده

```text id="q1v7bz"
error.message
        ↓
رسالة الخطأ الخاصة بالعملية / Axios

error.response?.data.message
        ↓
رسالة الخطأ التي أرسلها الـ API
```

وده فرق مهم جدًا، لأن بعد شوية لما نعمل **Error Normalization** هنقرر أي رسالة نعرضها للمستخدم وأي معلومات نستخدمها للـ debugging.

### 🎯 سؤال سريع

لو السيرفر رجع:

```json id="m5x8qa"
{
  "status": false,
  "code": 401,
  "message": "Token expired"
}
```

إيه الفرق بين:

```ts
error.message
```

و:

```ts
error.response?.data.message
```

**الخطوة التالية:** جاوب الفرق بين الاتنين من فهمك.
<details>
<summary>بص بشب، بكل بساطة، الفرق بين الـ error message والـ error message data message: الأولى دي معناها دوت الـ access error، وده الخطأ بتاع access نفسها. أما في حتة الـ error response، هي بقى بتقول لو الـ data موجودة، خشلي جوه الـ data وهاتلي الـ message، ودي خاصة بالـ API إيه، الـ API الخاصة بالسيرفر.</summary>
user
بص بشب، بكل بساطة، الفرق بين الـ error message والـ error message data message: الأولى دي معناها دوت الـ access error، وده الخطأ بتاع access نفسها. أما في حتة الـ error response، هي بقى بتقول لو الـ data موجودة، خشلي جوه الـ data وهاتلي الـ message، ودي خاصة بالـ API إيه، الـ API الخاصة بالسيرفر.
ChatGPT
**خد دقيقتين وثبّت إجابتك، لأنها صحيحة جدًا: 10/10.** ✅

بالضبط، أنت فرّقت بين مستويين:

```text id="z7m2qp"
error.message
     ↓
رسالة الخطأ الخاصة بـ Axios / العملية
```

وبين:

```text id="k4n8vx"
error.response?.data.message
          ↓
      data
          ↓
      message
          ↓
رسالة الخطأ التي أرسلها الـ API
```

مثال:

```json id="q2x6mc"
{
  "status": false,
  "code": 401,
  "message": "Token expired"
}
```

ممكن يكون:

```text id="s9v3ra"
error.message
→ "Request failed with status code 401"
```

بينما:

```text id="p6c1zw"
error.response?.data.message
→ "Token expired"
```

والـ `?.` هنا مهمة، لأن:

```text id="a8k4yt"
Network Error
     ↓
response غير موجودة
     ↓
error.response?.data.message
     ↓
undefined
```

### 🧠 أنت الآن عندك الصورة دي

```text id="j5r8nx"
AxiosError
│
├── response
│    ├── status        → HTTP status
│    └── data
│         ├── status   → API status
│         ├── code     → API code
│         └── message  → API message
│
├── request            → request information
│
└── message            → Axios/operation error message
```

ودي هتكون مهمة جدًا لما نبدأ **Error Normalization**.

لكن لسه عندنا property واحدة في `AxiosError`:

```text
error.code
```

ودي **مختلفة عن**:

```text
error.response.data.code
```

ودي تحديدًا لازم نفهمها قبل ما نبدأ `Error Normalization`.

**الخطوة التالية:** نكمل `error.code` ونفرق بينه وبين `error.response.data.code`.
</details>
بص بشب، بكل بساطة، الفرق بين الـ error message والـ error message data message: الأولى دي معناها دوت الـ access error، وده الخطأ بتاع access نفسها. أما في حتة الـ error response، هي بقى بتقول لو الـ data موجودة، خشلي جوه الـ data وهاتلي الـ message، ودي خاصة بالـ API إيه، الـ API الخاصة بالسيرفر.
ChatGPT
**خد دقيقتين وثبّت إجابتك، لأنها صحيحة جدًا: 10/10.** ✅

بالضبط، أنت فرّقت بين مستويين:

```text id="z7m2qp"
error.message
     ↓
رسالة الخطأ الخاصة بـ Axios / العملية
```

وبين:

```text id="k4n8vx"
error.response?.data.message
          ↓
      data
          ↓
      message
          ↓
رسالة الخطأ التي أرسلها الـ API
```

مثال:

```json id="q2x6mc"
{
  "status": false,
  "code": 401,
  "message": "Token expired"
}
```

ممكن يكون:

```text id="s9v3ra"
error.message
→ "Request failed with status code 401"
```

بينما:

```text id="p6c1zw"
error.response?.data.message
→ "Token expired"
```

والـ `?.` هنا مهمة، لأن:

```text id="a8k4yt"
Network Error
     ↓
response غير موجودة
     ↓
error.response?.data.message
     ↓
undefined
```

### 🧠 أنت الآن عندك الصورة دي

```text id="j5r8nx"
AxiosError
│
├── response
│    ├── status        → HTTP status
│    └── data
│         ├── status   → API status
│         ├── code     → API code
│         └── message  → API message
│
├── request            → request information
│
└── message            → Axios/operation error message
```

ودي هتكون مهمة جدًا لما نبدأ **Error Normalization**.

لكن لسه عندنا property واحدة في `AxiosError`:

```text
error.code
```

ودي **مختلفة عن**:

```text
error.response.data.code
```

ودي تحديدًا لازم نفهمها قبل ما نبدأ `Error Normalization`.

**الخطوة التالية:** نكمل `error.code` ونفرق بينه وبين `error.response.data.code`.
<details>
<summary>error.code</summary>
user
error.code
ChatGPT
**خد 7 دقائق: افهم `error.code` كـ code داخلي مرتبط بالخطأ في Axios، وليس الـ `code` الذي أرسله الـ API.**

## 📍 Roadmap

```text
SECTION 0  Architecture Foundation       ⬜
SECTION 1  Axios Fundamentals             ✅
SECTION 2  TypeScript + Axios             ✅
SECTION 3  Error Handling                 🟢 CURRENT
  ├─ Error Handling Fundamentals          ✅
  ├─ HTTP Errors                          ✅
  ├─ Network Errors                       ✅
  ├─ AxiosError
  │    ├─ error.response                   ✅
  │    ├─ error.request                    ✅
  │    ├─ error.message                    ✅
  │    └─ error.code                       🟡 ← نحن هنا
  ├─ Error Normalization                  ⬜
  └─ Custom ApiError                      ⬜
SECTION 4  Interceptors                   ⬜
...
```

# `error.code`

هنا لازم نفرّق بين **كودين مختلفين**.

### الأول

```ts
error.code
```

ده **code خاص بـ Axios** يصف نوع الخطأ على مستوى Axios.

### الثاني

```ts
error.response?.data.code
```

ده **code أرسله الـ API نفسه** داخل الـ response.

---

## مثال

السيرفر رجع:

```json
{
  "status": false,
  "code": 1004,
  "message": "Token expired"
}
```

وفي نفس الوقت Axios عنده معلوماته الخاصة عن الخطأ.

ممكن يكون:

```text
error.code
→ كود Axios للخطأ
```

بينما:

```text
error.response?.data.code
→ 1004
```

إذن لا تخلط بينهم.

---

# أشهر Axios Error Codes

من الأكواد التي قد تراها:

```text
ERR_BAD_REQUEST
ERR_BAD_RESPONSE
ECONNABORTED
ETIMEDOUT
ERR_NETWORK
ERR_CANCELED
```

مثلاً لو حصل timeout:

```ts id="m4x8pz"
try {
  await axios.get("/users", {
    timeout: 5000
  });
} catch (error) {
  console.log(error.code);
}
```

قد يكون الكود متعلقًا بالـ timeout، مثل:

```text
ETIMEDOUT
```

أو في بعض الإصدارات/الإعدادات قد ترى:

```text
ECONNABORTED
```

المهم الآن ليس حفظ كل الأكواد، بل فهم **مصدر الكود**.

---

# الصورة النهائية لـ `AxiosError`

```text id="x6p2vb"
AxiosError
│
├── message
│      ↓
│   Axios/operation error message
│
├── code
│      ↓
│   Axios error code
│
├── response
│   │
│   ├── status
│   │      ↓
│   │   HTTP status
│   │
│   └── data
│       │
│       ├── status
│       │      ↓
│       │   API status
│       │
│       ├── code
│       │      ↓
│       │   API code
│       │
│       └── message
│              ↓
│           API message
│
└── request
       ↓
   Request information
```

### احفظ المقارنة دي:

```text id="k3v8qa"
error.code
      ↓
Axios

error.response?.data.code
      ↓
API
```

وده نفس النمط اللي شوفناه قبل كده:

```text id="h8q1mz"
error.message
      ↓
Axios

error.response?.data.message
      ↓
API
```

**مهم:** قيمة `error.code` ليست بالضرورة موجودة في كل نوع من الأخطاء، لذلك لا نبني عليها افتراضًا بدون فحص.

## 🎯 Interview Question

عندك:

```json
{
  "status": false,
  "code": 1004,
  "message": "Token expired"
}
```

والـ HTTP status هو `401`.

اشرح الفرق بين الثلاثة:

```ts
error.code
error.response?.status
error.response?.data.code
```

**الخطوة التالية:** اكتب معنى الثلاثة بالترتيب.
</details>
error.code
ChatGPT
**خد 7 دقائق: افهم `error.code` كـ code داخلي مرتبط بالخطأ في Axios، وليس الـ `code` الذي أرسله الـ API.**

## 📍 Roadmap

```text
SECTION 0  Architecture Foundation       ⬜
SECTION 1  Axios Fundamentals             ✅
SECTION 2  TypeScript + Axios             ✅
SECTION 3  Error Handling                 🟢 CURRENT
  ├─ Error Handling Fundamentals          ✅
  ├─ HTTP Errors                          ✅
  ├─ Network Errors                       ✅
  ├─ AxiosError
  │    ├─ error.response                   ✅
  │    ├─ error.request                    ✅
  │    ├─ error.message                    ✅
  │    └─ error.code                       🟡 ← نحن هنا
  ├─ Error Normalization                  ⬜
  └─ Custom ApiError                      ⬜
SECTION 4  Interceptors                   ⬜
...
```

# `error.code`

هنا لازم نفرّق بين **كودين مختلفين**.

### الأول

```ts
error.code
```

ده **code خاص بـ Axios** يصف نوع الخطأ على مستوى Axios.

### الثاني

```ts
error.response?.data.code
```

ده **code أرسله الـ API نفسه** داخل الـ response.

---

## مثال

السيرفر رجع:

```json
{
  "status": false,
  "code": 1004,
  "message": "Token expired"
}
```

وفي نفس الوقت Axios عنده معلوماته الخاصة عن الخطأ.

ممكن يكون:

```text
error.code
→ كود Axios للخطأ
```

بينما:

```text
error.response?.data.code
→ 1004
```

إذن لا تخلط بينهم.

---

# أشهر Axios Error Codes

من الأكواد التي قد تراها:

```text
ERR_BAD_REQUEST
ERR_BAD_RESPONSE
ECONNABORTED
ETIMEDOUT
ERR_NETWORK
ERR_CANCELED
```

مثلاً لو حصل timeout:

```ts id="m4x8pz"
try {
  await axios.get("/users", {
    timeout: 5000
  });
} catch (error) {
  console.log(error.code);
}
```

قد يكون الكود متعلقًا بالـ timeout، مثل:

```text
ETIMEDOUT
```

أو في بعض الإصدارات/الإعدادات قد ترى:

```text
ECONNABORTED
```

المهم الآن ليس حفظ كل الأكواد، بل فهم **مصدر الكود**.

---

# الصورة النهائية لـ `AxiosError`

```text id="x6p2vb"
AxiosError
│
├── message
│      ↓
│   Axios/operation error message
│
├── code
│      ↓
│   Axios error code
│
├── response
│   │
│   ├── status
│   │      ↓
│   │   HTTP status
│   │
│   └── data
│       │
│       ├── status
│       │      ↓
│       │   API status
│       │
│       ├── code
│       │      ↓
│       │   API code
│       │
│       └── message
│              ↓
│           API message
│
└── request
       ↓
   Request information
```

### احفظ المقارنة دي:

```text id="k3v8qa"
error.code
      ↓
Axios

error.response?.data.code
      ↓
API
```

وده نفس النمط اللي شوفناه قبل كده:

```text id="h8q1mz"
error.message
      ↓
Axios

error.response?.data.message
      ↓
API
```

**مهم:** قيمة `error.code` ليست بالضرورة موجودة في كل نوع من الأخطاء، لذلك لا نبني عليها افتراضًا بدون فحص.

## 🎯 Interview Question

عندك:

```json
{
  "status": false,
  "code": 1004,
  "message": "Token expired"
}
```

والـ HTTP status هو `401`.

اشرح الفرق بين الثلاثة:

```ts
error.code
error.response?.status
error.response?.data.code
```

**الخطوة التالية:** اكتب معنى الثلاثة بالترتيب.
<details>
<summary>رقم واحد دي معناها الـ error بتاع الـ access error، الكود لو بتاع الـ access error أو الخطأ مكتوب access. أما التانية اللي هي error response status دي برضه خاصة بالـ error بس بتاع الـ response اللي موجود اللي هو بتاع مين؟ بتاع الـ access، بيجيبلك الـ status بتاعه زي 400 مثلاً واحد أو 404. أما الـ error response.data.code دي بقى اللي جاية من الـ API بتاع السيرفر وبيتحدد الرسالة اللي حددها الراجل في الباك إند.</summary>
user
رقم واحد دي معناها الـ error بتاع الـ access error، الكود لو بتاع الـ access error أو الخطأ مكتوب access. أما التانية اللي هي error response status دي برضه خاصة بالـ error بس بتاع الـ response اللي موجود اللي هو بتاع مين؟ بتاع الـ access، بيجيبلك الـ status بتاعه زي 400 مثلاً واحد أو 404. أما الـ error response.data.code دي بقى اللي جاية من الـ API بتاع السيرفر وبيتحدد الرسالة اللي حددها الراجل في الباك إند.
ChatGPT
**خد 3 دقائق وثبّت المصطلحات: إجابتك في الجوهر صحيحة، لكن عندك خلط بسيط بين `Axios` و`API` في أول نقطة.**

تقييمك: **9/10** ✅

التصحيح:

### 1. `error.code`

ده **مش HTTP status**، ومش الـ code اللي الباك إند بعته.

ده:

```text id="m4q8bz"
Axios error code
```

مثلاً:

```text
ERR_NETWORK
ETIMEDOUT
ERR_CANCELED
```

يعني Axios بيقول لك **نوع الخطأ اللي حصل أثناء تنفيذ العملية**.

---

### 2. `error.response?.status`

ده:

```text id="z7x3kp"
HTTP status
```

وده فعلًا السيرفر رجعه.

مثلاً:

```text
401
404
500
```

فلو:

```text id="k6n2rv"
error.response?.status
```

رجعت:

```text
404
```

فده معناه إن **السيرفر رد بـ HTTP 404**.

---

### 3. `error.response?.data.code`

ده:

```text id="p3w9fd"
API code
```

وده الـ code اللي الـ Backend صممه داخل الـ response.

مثلاً:

```json id="c8q4sa"
{
  "status": false,
  "code": 1004,
  "message": "Token expired"
}
```

إذن:

```text id="h2v6qm"
error.code
→ Axios

error.response?.status
→ HTTP

error.response?.data.code
→ API / Backend
```

ودي أهم جملة في الدرس كله:

> **Axios code ≠ HTTP status ≠ API code**

كل واحد جاي من **طبقة مختلفة**.

---

## 📍 Roadmap

```text
SECTION 3  Error Handling
  ├─ Fundamentals          ✅
  ├─ HTTP Errors           ✅
  ├─ Network Errors        ✅
  ├─ AxiosError
  │    ├─ response         ✅
  │    ├─ request          ✅
  │    ├─ message          ✅
  │    └─ code             ✅
  ├─ Error Normalization   🟡 ← التالي
  └─ Custom ApiError       ⬜
```

دلوقتي خلصنا **تفكيك `AxiosError`**. الخطوة الطبيعية التالية هي **Error Normalization**، وهنا هنبدأ نبني طريقة تجعل كل الأخطاء القادمة من الـ API لها شكل موحّد بدل ما كل Component يتعامل معها بطريقة مختلفة.

**الخطوة التالية:** نبدأ **Error Normalization** بمثال عملي فيه `404` و`Network Error` ونحوّل الاثنين إلى شكل Error موحّد.
</details>
رقم واحد دي معناها الـ error بتاع الـ access error، الكود لو بتاع الـ access error أو الخطأ مكتوب access. أما التانية اللي هي error response status دي برضه خاصة بالـ error بس بتاع الـ response اللي موجود اللي هو بتاع مين؟ بتاع الـ access، بيجيبلك الـ status بتاعه زي 400 مثلاً واحد أو 404. أما الـ error response.data.code دي بقى اللي جاية من الـ API بتاع السيرفر وبيتحدد الرسالة اللي حددها الراجل في الباك إند.
ChatGPT
**خد 3 دقائق وثبّت المصطلحات: إجابتك في الجوهر صحيحة، لكن عندك خلط بسيط بين `Axios` و`API` في أول نقطة.**

تقييمك: **9/10** ✅

التصحيح:

### 1. `error.code`

ده **مش HTTP status**، ومش الـ code اللي الباك إند بعته.

ده:

```text id="m4q8bz"
Axios error code
```

مثلاً:

```text
ERR_NETWORK
ETIMEDOUT
ERR_CANCELED
```

يعني Axios بيقول لك **نوع الخطأ اللي حصل أثناء تنفيذ العملية**.

---

### 2. `error.response?.status`

ده:

```text id="z7x3kp"
HTTP status
```

وده فعلًا السيرفر رجعه.

مثلاً:

```text
401
404
500
```

فلو:

```text id="k6n2rv"
error.response?.status
```

رجعت:

```text
404
```

فده معناه إن **السيرفر رد بـ HTTP 404**.

---

### 3. `error.response?.data.code`

ده:

```text id="p3w9fd"
API code
```

وده الـ code اللي الـ Backend صممه داخل الـ response.

مثلاً:

```json id="c8q4sa"
{
  "status": false,
  "code": 1004,
  "message": "Token expired"
}
```

إذن:

```text id="h2v6qm"
error.code
→ Axios

error.response?.status
→ HTTP

error.response?.data.code
→ API / Backend
```

ودي أهم جملة في الدرس كله:

> **Axios code ≠ HTTP status ≠ API code**

كل واحد جاي من **طبقة مختلفة**.

---

## 📍 Roadmap

```text
SECTION 3  Error Handling
  ├─ Fundamentals          ✅
  ├─ HTTP Errors           ✅
  ├─ Network Errors        ✅
  ├─ AxiosError
  │    ├─ response         ✅
  │    ├─ request          ✅
  │    ├─ message          ✅
  │    └─ code             ✅
  ├─ Error Normalization   🟡 ← التالي
  └─ Custom ApiError       ⬜
```

دلوقتي خلصنا **تفكيك `AxiosError`**. الخطوة الطبيعية التالية هي **Error Normalization**، وهنا هنبدأ نبني طريقة تجعل كل الأخطاء القادمة من الـ API لها شكل موحّد بدل ما كل Component يتعامل معها بطريقة مختلفة.

**الخطوة التالية:** نبدأ **Error Normalization** بمثال عملي فيه `404` و`Network Error` ونحوّل الاثنين إلى شكل Error موحّد.
<details>
<summary>AxiosError تهيائلي كان في config</summary>
user
AxiosError تهيائلي كان في config
ChatGPT
**خد 5 دقائق: اربط `config` داخل `AxiosError` بما درسته سابقًا عن Request Configuration.**

أيوه، **بالضبط**. `AxiosError` ممكن يحتوي على `config`، ودي فعلًا من الحاجات المهمة فيه.

الصورة المبسطة:

```text
AxiosError
├── message
├── name
├── code
├── config        ← أنت فاكرها صح
├── request
└── response
```

### طيب `error.config` إيه؟

دي بتكون **الـ configuration الخاصة بالـ request اللي حصل فيه الخطأ**.

مثلاً أنت عملت:

```ts
await axios.get("/users", {
  timeout: 5000,
  headers: {
    Authorization: "Bearer token"
  },
  params: {
    page: 1
  }
});
```

لو حصل Error، ممكن تلاقي:

```ts
error.config
```

وفيها معلومات عن الـ request، مثل:

```text
config
├── url
├── method
├── headers
├── params
├── timeout
└── ...
```

يعني أنت هنا بتشوف **إزاي الطلب كان متكوّن وقت حدوث الخطأ**.

### اربطها بما درسناه

أنت درست:

```ts
axios.get("/users", {
  params: {...},
  headers: {...},
  timeout: 5000
});
```

الـ object ده هو **Request Configuration**.

وفي حالة الخطأ، Axios بيحتفظ بالـ configuration المرتبطة بالطلب داخل:

```ts
error.config
```

---

### لكن خد بالك

`error.config` **مش هو الـ response**.

```text
error.config
    ↓
معلومات عن الـ Request

error.request
    ↓
معلومات عن الـ Request الذي أُرسل

error.response
    ↓
معلومات عن الـ Response الذي رجع من السيرفر
```

وده فرق مهم.

---

## الصورة الكاملة

```text
                    AxiosError
                        │
        ┌───────────────┼───────────────┐
        ↓               ↓               ↓
     config          request         response
        │               │               │
        ↓               ↓               ↓
 Request Config    Request Info    Server Response
                                    │
                                    ├─ status
                                    └─ data
```

إذن ملاحظتك كانت في محلها، وأنا كنت مركز معاك على أهم properties للـ Error Classification، لكن فعلًا **`config` جزء مهم من `AxiosError`**.

**الخطوة التالية:** نبدأ `Error Normalization`، ونستخدم كل اللي اتعلمناه عن `response`, `request`, `message`, `code`, و`config` لبناء Error موحّد.
</details>
AxiosError تهيائلي كان في config
ChatGPT
**خد 5 دقائق: اربط `config` داخل `AxiosError` بما درسته سابقًا عن Request Configuration.**

أيوه، **بالضبط**. `AxiosError` ممكن يحتوي على `config`، ودي فعلًا من الحاجات المهمة فيه.

الصورة المبسطة:

```text
AxiosError
├── message
├── name
├── code
├── config        ← أنت فاكرها صح
├── request
└── response
```

### طيب `error.config` إيه؟

دي بتكون **الـ configuration الخاصة بالـ request اللي حصل فيه الخطأ**.

مثلاً أنت عملت:

```ts
await axios.get("/users", {
  timeout: 5000,
  headers: {
    Authorization: "Bearer token"
  },
  params: {
    page: 1
  }
});
```

لو حصل Error، ممكن تلاقي:

```ts
error.config
```

وفيها معلومات عن الـ request، مثل:

```text
config
├── url
├── method
├── headers
├── params
├── timeout
└── ...
```

يعني أنت هنا بتشوف **إزاي الطلب كان متكوّن وقت حدوث الخطأ**.

### اربطها بما درسناه

أنت درست:

```ts
axios.get("/users", {
  params: {...},
  headers: {...},
  timeout: 5000
});
```

الـ object ده هو **Request Configuration**.

وفي حالة الخطأ، Axios بيحتفظ بالـ configuration المرتبطة بالطلب داخل:

```ts
error.config
```

---

### لكن خد بالك

`error.config` **مش هو الـ response**.

```text
error.config
    ↓
معلومات عن الـ Request

error.request
    ↓
معلومات عن الـ Request الذي أُرسل

error.response
    ↓
معلومات عن الـ Response الذي رجع من السيرفر
```

وده فرق مهم.

---

## الصورة الكاملة

```text
                    AxiosError
                        │
        ┌───────────────┼───────────────┐
        ↓               ↓               ↓
     config          request         response
        │               │               │
        ↓               ↓               ↓
 Request Config    Request Info    Server Response
                                    │
                                    ├─ status
                                    └─ data
```

إذن ملاحظتك كانت في محلها، وأنا كنت مركز معاك على أهم properties للـ Error Classification، لكن فعلًا **`config` جزء مهم من `AxiosError`**.

**الخطوة التالية:** نبدأ `Error Normalization`، ونستخدم كل اللي اتعلمناه عن `response`, `request`, `message`, `code`, و`config` لبناء Error موحّد.
<details>
<summary>Error Normalization</summary>
user
Error Normalization
ChatGPT
**خد 15 دقيقة: هنفهم `Error Normalization` كمفهوم معماري أولًا، ثم نبني أول `normalizeError` بدون ما ندخل لسه في `Custom ApiError`.**

## 📍 Roadmap

```text
SECTION 0  Architecture Foundation       ⬜
SECTION 1  Axios Fundamentals             ✅
SECTION 2  TypeScript + Axios             ✅
SECTION 3  Error Handling                 🟢 CURRENT
  ├─ Error Handling Fundamentals          ✅
  ├─ HTTP Errors                          ✅
  ├─ Network Errors                       ✅
  ├─ AxiosError                           ✅
  ├─ Error Normalization                  🟡 ← نحن هنا
  └─ Custom ApiError                      ⬜
SECTION 4  Interceptors                   ⬜
...
```
</details>
Error Normalization
ChatGPT
**خد 15 دقيقة: هنفهم `Error Normalization` كمفهوم معماري أولًا، ثم نبني أول `normalizeError` بدون ما ندخل لسه في `Custom ApiError`.**

## 📍 Roadmap

```text
SECTION 0  Architecture Foundation       ⬜
SECTION 1  Axios Fundamentals             ✅
SECTION 2  TypeScript + Axios             ✅
SECTION 3  Error Handling                 🟢 CURRENT
  ├─ Error Handling Fundamentals          ✅
  ├─ HTTP Errors                          ✅
  ├─ Network Errors                       ✅
  ├─ AxiosError                           ✅
  ├─ Error Normalization                  🟡 ← نحن هنا
  └─ Custom ApiError                      ⬜
SECTION 4  Interceptors                   ⬜
...
```