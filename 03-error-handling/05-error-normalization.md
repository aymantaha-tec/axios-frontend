# ما هو Error Normalization؟

تخيل عندك Components كثيرة:

```text
Profile
Users
Orders
Products
```

وكل واحد بيتعامل مع الأخطاء بطريقة مختلفة:

```text
Profile
→ error.response?.data.message

Users
→ error.message

Orders
→ error.response?.status

Products
→ error.code
```

ده هيعمل فوضى.

إحنا عايزين **مكان واحد** يستقبل أي Error ويحوّله إلى **شكل موحّد**.

وده هو:

> **Error Normalization**

يعني:

```text
Different Errors
      ↓
normalizeError()
      ↓
One Standard Error Shape
```

---

# مثال

ممكن يحصل عندنا:

### HTTP Error

```text
404
```

أو:

### Network Error

```text
ERR_NETWORK
```

أو:

### Timeout

```text
ETIMEDOUT
```

أو Error عادي:

```ts
new Error("Something went wrong")
```

بدل ما الـ Component يعرف كل الاحتمالات دي، نخلي `normalizeError` تتعامل معها.

---

# الشكل الموحد

ممكن نقرر إن كل الأخطاء عندنا يكون شكلها:

```ts
interface INormalizedError {
  message: string;
  status?: number;
  code?: string;
}
```

مثلاً HTTP Error:

```ts
{
  message: "User not found",
  status: 404,
  code: "1004"
}
```

Network Error:

```ts
{
  message: "Network error",
  status: undefined,
  code: "ERR_NETWORK"
}
```

لاحظ إن الـ Component مش محتاج يعرف:

```text
AxiosError
error.response
error.response.data
error.code
```

هو يستقبل:

```ts
INormalizedError
```

---

# ليه ده مهم؟

لأننا بنعمل **Boundary** بين Axios والـ application.

```text
              Axios
                │
                ↓
           AxiosError
                │
                ↓
        normalizeError()
                │
                ↓
       INormalizedError
                │
        ┌───────┼───────┐
        ↓       ↓       ↓
     Profile  Users   Orders
```

الـ Components تتعامل مع **شكل واحد فقط**.

---

# أول خطوة في الكود

نحتاج نعرف هل الـ Error أصلًا من Axios.

هنا نستخدم:

```ts
axios.isAxiosError(error)
```

مثلاً:

```ts
try {
  await axios.get("/users");
} catch (error) {
  if (axios.isAxiosError(error)) {
    console.log("Axios error");
  }
}
```

ليه بنعمل كده؟

لأن TypeScript لا يفترض إن:

```ts
catch (error)
```

هو `AxiosError`.

الـ `error` ممكن يكون أي شيء.

```text
error
├── AxiosError
├── Error
├── string
├── unknown object
└── ...
```

فـ:

```ts
axios.isAxiosError(error)
```

تعمل لنا **Type Narrowing**.

---

# الآن نبني أول نسخة

```ts
import axios from "axios";

interface INormalizedError {
  message: string;
  status?: number;
  code?: string;
}

export function normalizeError(
  error: unknown
): INormalizedError {

  if (axios.isAxiosError(error)) {
    return {
      message: error.message,
      status: error.response?.status,
      code: error.code,
    };
  }

  return {
    message: "Something went wrong",
  };
}
```

لاحظ حاجة مهمة جدًا:

```ts
error: unknown
```

وليس:

```ts
error: any
```

لأننا لا نثق في الـ Error القادم.

نحن أولًا نعمل:

```ts
axios.isAxiosError(error)
```

وبعدها نسمح لأنفسنا باستخدام خصائص Axios.

---

# لكن عندنا مشكلة

في المثال السابق:

```ts
message: error.message
```

لو السيرفر رجع:

```json
{
  "status": false,
  "code": 1004,
  "message": "User not found"
}
```

ممكن نكون عايزين رسالة الـ API:

```text
User not found
```

وليس:

```text
Request failed with status code 404
```

إذن `normalizeError` لازم تكون أذكى.

وهنا هنبدأ في الدرس القادم نبني **API Error Extraction**:

```text
AxiosError
    ↓
هل عندي response؟
    ↓
هل عندي data؟
    ↓
هل عندي API message؟
    ↓
استخدمها
```

وده هيقودنا مباشرة إلى `Custom ApiError`.

---

## 🧠 الفكرة التي أريدك تفهمها الآن

**Error Normalization مش مجرد function اسمها `normalizeError`.**

الفكرة المعمارية هي:

```text
Many Error Shapes
       ↓
      Normalize
       ↓
One Application Error Shape
```

وبالتالي الـ Components لا تتعامل مباشرة مع تفاصيل Axios.

### 🎯 سؤال Interview

ليه نكتب:

```ts
function normalizeError(error: unknown)
```

بدل:

```ts
function normalizeError(error: any)
```

**الخطوة التالية:** اشرح لي ليه `unknown` أفضل من `any` هنا.
<details>
<summary>final non هنا دايمًا بتبقى أفضل من any، لأن any بتبقى مفتوحة. يعني هنا بتقدر تستقبل أي حاجة، تستقبل أي error، سواء الـerror دوت متعرف مرة string مرة boolean مرة أي شيء بيتعرف بيه. أما الـunknown أنا ما أعرفش الـerror، ففي الحالة دي الـTypeScript مش هديلكم autocomplete أي حاجة خالص، حتى لو إن كان string، مش هت... مثلاً قلته string مثلاً، مش هتشتغل، لأن هي مش هتعمل إيه؟ لازم أأكد لازم من نوع الخطأ، وهي ما تعرفوش، وبما إن هي ما تعرفوش، حتى لو متحدد برضه مش هتوافق بيه. فلازم أعمل قاعدة if وأقول لو الـtype بتاع الـdata بيequal equal مثلا string ساعتها هشغل معاه الـmiss of string.</summary>
user
final non هنا دايمًا بتبقى أفضل من any، لأن any بتبقى مفتوحة. يعني هنا بتقدر تستقبل أي حاجة، تستقبل أي error، سواء الـerror دوت متعرف مرة string مرة boolean مرة أي شيء بيتعرف بيه. أما الـunknown أنا ما أعرفش الـerror، ففي الحالة دي الـTypeScript مش هديلكم autocomplete أي حاجة خالص، حتى لو إن كان string، مش هت... مثلاً قلته string مثلاً، مش هتشتغل، لأن هي مش هتعمل إيه؟ لازم أأكد لازم من نوع الخطأ، وهي ما تعرفوش، وبما إن هي ما تعرفوش، حتى لو متحدد برضه مش هتوافق بيه. فلازم أعمل قاعدة if وأقول لو الـtype بتاع الـdata بيequal equal مثلا string ساعتها هشغل معاه الـmiss of string.
ChatGPT
**خد 5 دقائق وثبّت إجابتك: الفكرة الأساسية عندك صحيحة جدًا، لكن في نقطة واحدة محتاجة ضبط، ودرجتك 9.5/10.** ✅

أنت شرحت أهم فرق بين `any` و`unknown` بشكل صحيح:

```text id="p7x4ka"
any
↓
TypeScript يثق فيك
↓
يسمح لك تعمل أي حاجة تقريبًا
```

بينما:

```text id="m3q8vz"
unknown
↓
TypeScript لا يعرف النوع
↓
لا يسمح لك باستخدامه
↓
إلا بعد Type Narrowing
```

### التصحيح الصغير

أنت قلت:

> حتى لو كان `string` مش هتشتغل.

الأدق:

لو المتغير **معلن كـ `unknown`**، TypeScript لا يسمح لك باستخدامه كـ `string` **إلا بعد ما تثبت أنه `string`**.

مثلاً:

```ts id="7x3pqm"
function test(error: unknown) {
  error.toUpperCase(); // ❌
}
```

لكن:

```ts id="n8v2ka"
function test(error: unknown) {
  if (typeof error === "string") {
    error.toUpperCase(); // ✅
  }
}
```

لأن:

```text id="q6w9dz"
unknown
   ↓
typeof error === "string"
   ↓
TypeScript عرف إنه string
   ↓
string methods متاحة
```

---

## وده بالضبط اللي بنعمله مع Axios

عندنا:

```ts id="k5c1vr"
catch (error) {
```

الـ `error` بالنسبة لنا غير موثوق.

فنقول:

```ts id="b9m4xf"
if (axios.isAxiosError(error)) {
```

هنا Axios بيعمل لنا **Type Narrowing**:

```text id="h7z2qp"
unknown
   ↓
axios.isAxiosError(error)
   ↓
AxiosError
   ↓
error.response
error.request
error.message
error.code
```

وده سبب مهم جدًا لاستخدام:

```ts id="w3n8sa"
error: unknown
```

في `normalizeError`.

---

# 🎯 والآن ننتقل للنقطة الأهم

إحنا عندنا:

```ts id="c8q4nm"
interface INormalizedError {
  message: string;
  status?: number;
  code?: string;
}
```

وعندنا:

```ts id="m5v8kp"
AxiosError
```

لكن إحنا لسه بنرجع:

```ts id="x1q7rz"
message: error.message
```

وممكن نكون عايزين بدلها رسالة الـ API:

```ts id="j6n2wc"
error.response?.data.message
```

**وهنا يبدأ الجزء الحقيقي من `Error Normalization`: استخراج المعلومات الصحيحة من الـ `AxiosError` وتحويلها للشكل الموحد.**

**الخطوة التالية:** هنكتب `normalizeError` مع `IErrorResponse` بتاعتك، ونخليها تستخرج `message`, `status`, و`code` من الـ API بشكل type-safe.
</details>
final non هنا دايمًا بتبقى أفضل من any، لأن any بتبقى مفتوحة. يعني هنا بتقدر تستقبل أي حاجة، تستقبل أي error، سواء الـerror دوت متعرف مرة string مرة boolean مرة أي شيء بيتعرف بيه. أما الـunknown أنا ما أعرفش الـerror، ففي الحالة دي الـTypeScript مش هديلكم autocomplete أي حاجة خالص، حتى لو إن كان string، مش هت... مثلاً قلته string مثلاً، مش هتشتغل، لأن هي مش هتعمل إيه؟ لازم أأكد لازم من نوع الخطأ، وهي ما تعرفوش، وبما إن هي ما تعرفوش، حتى لو متحدد برضه مش هتوافق بيه. فلازم أعمل قاعدة if وأقول لو الـtype بتاع الـdata بيequal equal مثلا string ساعتها هشغل معاه الـmiss of string.
ChatGPT
**خد 5 دقائق وثبّت إجابتك: الفكرة الأساسية عندك صحيحة جدًا، لكن في نقطة واحدة محتاجة ضبط، ودرجتك 9.5/10.** ✅

أنت شرحت أهم فرق بين `any` و`unknown` بشكل صحيح:

```text id="p7x4ka"
any
↓
TypeScript يثق فيك
↓
يسمح لك تعمل أي حاجة تقريبًا
```

بينما:

```text id="m3q8vz"
unknown
↓
TypeScript لا يعرف النوع
↓
لا يسمح لك باستخدامه
↓
إلا بعد Type Narrowing
```

### التصحيح الصغير

أنت قلت:

> حتى لو كان `string` مش هتشتغل.

الأدق:

لو المتغير **معلن كـ `unknown`**، TypeScript لا يسمح لك باستخدامه كـ `string` **إلا بعد ما تثبت أنه `string`**.

مثلاً:

```ts id="7x3pqm"
function test(error: unknown) {
  error.toUpperCase(); // ❌
}
```

لكن:

```ts id="n8v2ka"
function test(error: unknown) {
  if (typeof error === "string") {
    error.toUpperCase(); // ✅
  }
}
```

لأن:

```text id="q6w9dz"
unknown
   ↓
typeof error === "string"
   ↓
TypeScript عرف إنه string
   ↓
string methods متاحة
```

---

## وده بالضبط اللي بنعمله مع Axios

عندنا:

```ts id="k5c1vr"
catch (error) {
```

الـ `error` بالنسبة لنا غير موثوق.

فنقول:

```ts id="b9m4xf"
if (axios.isAxiosError(error)) {
```

هنا Axios بيعمل لنا **Type Narrowing**:

```text id="h7z2qp"
unknown
   ↓
axios.isAxiosError(error)
   ↓
AxiosError
   ↓
error.response
error.request
error.message
error.code
```

وده سبب مهم جدًا لاستخدام:

```ts id="w3n8sa"
error: unknown
```

في `normalizeError`.

---

# 🎯 والآن ننتقل للنقطة الأهم

إحنا عندنا:

```ts id="c8q4nm"
interface INormalizedError {
  message: string;
  status?: number;
  code?: string;
}
```

وعندنا:

```ts id="m5v8kp"
AxiosError
```

لكن إحنا لسه بنرجع:

```ts id="x1q7rz"
message: error.message
```

وممكن نكون عايزين بدلها رسالة الـ API:

```ts id="j6n2wc"
error.response?.data.message
```

**وهنا يبدأ الجزء الحقيقي من `Error Normalization`: استخراج المعلومات الصحيحة من الـ `AxiosError` وتحويلها للشكل الموحد.**

**الخطوة التالية:** هنكتب `normalizeError` مع `IErrorResponse` بتاعتك، ونخليها تستخرج `message`, `status`, و`code` من الـ API بشكل type-safe.
<details>
<summary>interface INormalizeError{</summary>
user
interface INormalizeError{
message: string,
code?:number,
stauts?: boolean,
}
export function normalizeError(error: unknown):INormalizeError{
if(axios.isAxiosError(error)){
return{
message: error.message,
status: error.response?.status,
code: error.code
}}
return {
message:"something went wrong",
}
}
بصراحه نقلت شويه من الكود اللي فوق
ChatGPT
**خد 7 دقائق، صحّح الكود بنفسك ذهنيًا ثم قارن بالنسخة الصحيحة، لأنك فاهم الـ flow لكن عندك 3 أخطاء TypeScript/Design واضحة.**

وأيوه، كونك قلت إنك نقلت جزء من الكود مش مشكلة. المهم إننا نفهم **ليه** الكود بالشكل ده.

## تقييمك: 8.5/10

الكود عندك:

```ts
interface INormalizeError {
  message: string,
  code?: number,
  stauts?: boolean,
}

export function normalizeError(
  error: unknown
): INormalizeError {

  if (axios.isAxiosError(error)) {
    return {
      message: error.message,
      status: error.response?.status,
      code: error.code
    }
  }

  return {
    message: "something went wrong",
  }
}
```

### ❌ المشكلة الأولى: `stauts`

أنت كتبت:

```ts
stauts?: boolean
```

في الـ interface.

لكن رجعت:

```ts
status: error.response?.status
```

فيه مشكلتان:

```text
stauts ❌
status  ✅
```

لكن الأهم إننا أصلًا **مش عايزين `status: boolean` هنا**.

لأن:

```ts
error.response?.status
```

ده HTTP status:

```text
404
401
500
```

إذن المفروض:

```ts
status?: number;
```

---

### ❌ المشكلة الثانية: `code`

أنت عرّفت:

```ts
code?: number
```

لكن:

```ts
error.code
```

في Axios غالبًا نوعه:

```ts
string | undefined
```

لأنه ممكن يكون:

```text
ERR_NETWORK
ETIMEDOUT
ERR_CANCELED
```

إذن لو إحنا بنخزن **Axios code**:

```ts
code?: string;
```

---

### ❌ المشكلة الثالثة: `message`

أنت حاليًا تستخدم:

```ts
message: error.message
```

وده **Axios message**.

لكن هدفنا في `Error Normalization` هو إننا نحاول نوصل إلى **رسالة الـ API** لو موجودة.

مثلاً:

```json
{
  "status": false,
  "code": 1004,
  "message": "User not found"
}
```

إحنا عايزين في النهاية:

```ts
{
  message: "User not found",
  status: 404
}
```

وليس فقط:

```ts
{
  message: "Request failed with status code 404"
}
```

لكن هنا لازم نقف.

**لسه ما ينفعش نكتب الوصول إلى `error.response.data.message` مباشرة**، لأننا لم نحدد لـ `AxiosError` شكل الـ `data` بتاعته بشكل TypeScript-safe.

وده هو الجزء التالي.

---

## نقطة مهمة جدًا في تصميم الـ interface

أنت كتبت:

```ts
stauts?: boolean
```

واضح إنك كنت بتفكر في الـ API response اللي عندك:

```ts
status: false
```

وده مفهوم.

لكن لازم نسأل:

**إيه البيانات اللي نريدها من `Normalized Error`؟**

هل نحتاج:

```ts
status: boolean
```

ولا:

```ts
status: number
```

هنا إحنا بنعمل normalization، فممكن نختار الشكل اللي يناسب التطبيق.

في نسختنا الحالية:

```ts
interface INormalizedError {
  message: string;
  status?: number;
  code?: string;
}
```

المعنى:

```text
message
→ رسالة الخطأ

status
→ HTTP status

code
→ Axios error code
```

---

### 🧠 أهم نقطة في الدرس

عندنا الآن **ثلاثة `code/status/message` من أماكن مختلفة**:

```text
AxiosError
│
├── message
├── code
│
└── response
     │
     ├── status       ← HTTP
     │
     └── data
          │
          ├── status  ← API
          ├── code    ← API
          └── message ← API
```

وده بالضبط السبب اللي خلانا نحتاج **Error Normalization**.

**الخطوة التالية:** قبل ما نعدل `normalizeError`، سنعرّف `AxiosError<IErrorResponse>` ونخلي TypeScript يعرف أن `error.response.data` هو الـ `IErrorResponse` بتاع الـ API.
</details>
interface INormalizeError{
message: string,
code?:number,
stauts?: boolean,
}
export function normalizeError(error: unknown):INormalizeError{
if(axios.isAxiosError(error)){
return{
message: error.message,
status: error.response?.status,
code: error.code
}}
return {
message:"something went wrong",
}
}
بصراحه نقلت شويه من الكود اللي فوق
ChatGPT
**خد 7 دقائق، صحّح الكود بنفسك ذهنيًا ثم قارن بالنسخة الصحيحة، لأنك فاهم الـ flow لكن عندك 3 أخطاء TypeScript/Design واضحة.**

وأيوه، كونك قلت إنك نقلت جزء من الكود مش مشكلة. المهم إننا نفهم **ليه** الكود بالشكل ده.

## تقييمك: 8.5/10

الكود عندك:

```ts
interface INormalizeError {
  message: string,
  code?: number,
  stauts?: boolean,
}

export function normalizeError(
  error: unknown
): INormalizeError {

  if (axios.isAxiosError(error)) {
    return {
      message: error.message,
      status: error.response?.status,
      code: error.code
    }
  }

  return {
    message: "something went wrong",
  }
}
```

### ❌ المشكلة الأولى: `stauts`

أنت كتبت:

```ts
stauts?: boolean
```

في الـ interface.

لكن رجعت:

```ts
status: error.response?.status
```

فيه مشكلتان:

```text
stauts ❌
status  ✅
```

لكن الأهم إننا أصلًا **مش عايزين `status: boolean` هنا**.

لأن:

```ts
error.response?.status
```

ده HTTP status:

```text
404
401
500
```

إذن المفروض:

```ts
status?: number;
```

---

### ❌ المشكلة الثانية: `code`

أنت عرّفت:

```ts
code?: number
```

لكن:

```ts
error.code
```

في Axios غالبًا نوعه:

```ts
string | undefined
```

لأنه ممكن يكون:

```text
ERR_NETWORK
ETIMEDOUT
ERR_CANCELED
```

إذن لو إحنا بنخزن **Axios code**:

```ts
code?: string;
```

---

### ❌ المشكلة الثالثة: `message`

أنت حاليًا تستخدم:

```ts
message: error.message
```

وده **Axios message**.

لكن هدفنا في `Error Normalization` هو إننا نحاول نوصل إلى **رسالة الـ API** لو موجودة.

مثلاً:

```json
{
  "status": false,
  "code": 1004,
  "message": "User not found"
}
```

إحنا عايزين في النهاية:

```ts
{
  message: "User not found",
  status: 404
}
```

وليس فقط:

```ts
{
  message: "Request failed with status code 404"
}
```

لكن هنا لازم نقف.

**لسه ما ينفعش نكتب الوصول إلى `error.response.data.message` مباشرة**، لأننا لم نحدد لـ `AxiosError` شكل الـ `data` بتاعته بشكل TypeScript-safe.

وده هو الجزء التالي.

---

## نقطة مهمة جدًا في تصميم الـ interface

أنت كتبت:

```ts
stauts?: boolean
```

واضح إنك كنت بتفكر في الـ API response اللي عندك:

```ts
status: false
```

وده مفهوم.

لكن لازم نسأل:

**إيه البيانات اللي نريدها من `Normalized Error`؟**

هل نحتاج:

```ts
status: boolean
```

ولا:

```ts
status: number
```

هنا إحنا بنعمل normalization، فممكن نختار الشكل اللي يناسب التطبيق.

في نسختنا الحالية:

```ts
interface INormalizedError {
  message: string;
  status?: number;
  code?: string;
}
```

المعنى:

```text
message
→ رسالة الخطأ

status
→ HTTP status

code
→ Axios error code
```

---

### 🧠 أهم نقطة في الدرس

عندنا الآن **ثلاثة `code/status/message` من أماكن مختلفة**:

```text
AxiosError
│
├── message
├── code
│
└── response
     │
     ├── status       ← HTTP
     │
     └── data
          │
          ├── status  ← API
          ├── code    ← API
          └── message ← API
```

وده بالضبط السبب اللي خلانا نحتاج **Error Normalization**.

**الخطوة التالية:** قبل ما نعدل `normalizeError`، سنعرّف `AxiosError<IErrorResponse>` ونخلي TypeScript يعرف أن `error.response.data` هو الـ `IErrorResponse` بتاع الـ API.
<details>
<summary>Normalization</summary>
user
Normalization
محتاج شرح اكثر وضوحا وفهما
ChatGPT
**خد 15 دقيقة، وانسَ الكود مؤقتًا، وافهم `Error Normalization` كفكرة معمارية من خلال مثال واحد واضح.**

## 📍 Roadmap

```text
SECTION 0  Architecture Foundation       ⬜
SECTION 1  Axios Fundamentals             ✅
SECTION 2  TypeScript + Axios             ✅
SECTION 3  Error Handling                 🟢
  ├─ Fundamentals                         ✅
  ├─ HTTP Errors                          ✅
  ├─ Network Errors                       ✅
  ├─ AxiosError                            ✅
  ├─ Error Normalization                  🟡 ← نحن هنا
  └─ Custom ApiError                      ⬜
```

# يعني إيه Normalization أصلًا؟

كلمة **Normalize** معناها ببساطة:

> **أحوّل أشكال مختلفة إلى شكل موحّد.**

مثلاً عندك ناس بتكتب رقم التليفون بأشكال مختلفة:

```text
01012345678
+201012345678
00201012345678
```

لو أنت عايز النظام كله يتعامل مع صيغة واحدة، تعمل:

```text
Different Formats
       ↓
    Normalize
       ↓
One Standard Format
```

نفس الفكرة بالضبط مع الـ Errors.

---

# المشكلة التي نحلها

تخيل أن عندك API.

ممكن تحصل أخطاء كثيرة جدًا:

### الحالة الأولى

السيرفر يرد:

```text
HTTP 404
```

والـ API يقول:

```json
{
  "status": false,
  "code": 1001,
  "message": "User not found"
}
```

---

### الحالة الثانية

الإنترنت يفصل:

```text
Network Error
```

ومفيش `response` أصلًا.

---

### الحالة الثالثة

الـ request يأخذ وقتًا أطول من:

```text
5000 ms
```

فيحدث:

```text
Timeout
```

---

### الحالة الرابعة

المستخدم يلغي الـ request:

```text
Abort
```

---

إذن عندنا أشكال مختلفة جدًا:

```text
HTTP Error
Network Error
Timeout
Abort
Unknown Error
```

ولو سيبنا كل Component يتعامل مع كل نوع بنفسه، هنقع في مشكلة.

---

# شوف الـ Component بدون Normalization

مثلاً:

```tsx
try {
  const user = await getUser(10);
} catch (error) {

  if (axios.isAxiosError(error)) {

    if (error.response) {
      // HTTP Error
    }

    if (error.request) {
      // Network Error
    }

    if (error.code === "ETIMEDOUT") {
      // Timeout
    }

  } else {
    // Unknown Error
  }
}
```

تخيل إن عندك:

```text
20 Components
```

كل Component يعمل الكلام ده.

😵‍💫

هنا الـ Components بدأت تعرف تفاصيل Axios.

---

# هنا يأتي Error Normalization

نقول:

> **أنا مش عايز الـ Component يعرف كل أنواع الـ Errors.**

أعمل function واحدة:

```text
              Different Errors
                     ↓
              normalizeError()
                     ↓
             Standard Error
```

مثلاً:

```ts
interface INormalizedError {
  message: string;
  status?: number;
  code?: string;
}
```

دلوقتي مهما كان الخطأ:

### 404

ندخله:

```text
AxiosError
```

ويخرج:

```ts
{
  message: "User not found",
  status: 404
}
```

### Network Error

يدخل:

```text
AxiosError
```

ويخرج:

```ts
{
  message: "Network error",
  code: "ERR_NETWORK"
}
```

### Timeout

يدخل:

```text
AxiosError
```

ويخرج:

```ts
{
  message: "Request timed out",
  code: "ETIMEDOUT"
}
```

---

# إذن الـ Normalization مش بيحل الـ Error

دي نقطة مهمة جدًا.

لو السيرفر رجع:

```text
404
```

`normalizeError` **مش هتحول 404 إلى 200**.

ولا تصلح المشكلة.

هي فقط تقول:

> "أنا عندي Error شكله كذا، هحوّله للشكل القياسي بتاعي."

يعني:

```text
404
↓
AxiosError
↓
normalizeError()
↓
{
  message: "...",
  status: 404
}
```

---

# طب ليه ده مفيد؟

لأن الـ Component بعد كده يتعامل مع حاجة واحدة:

```tsx
try {
  const user = await getUser(10);
} catch (error) {

  const normalizedError = normalizeError(error);

  console.log(normalizedError.message);
}
```

الـ Component لا يهتم:

```text
هل 404؟
هل Network؟
هل Timeout؟
هل Axios؟
هل response موجودة؟
هل data موجودة؟
```

كل ده مسؤولية:

```text
normalizeError()
```

---

# شوف الفرق المعماري

### ❌ بدون Normalization

```text
Component
   │
   ├── AxiosError
   ├── HTTP Error
   ├── Network Error
   ├── Timeout
   └── Abort
```

الـ Component أصبح يعرف تفاصيل كثيرة.

---

### ✅ مع Normalization

```text
                  Axios
                    ↓
              Different Errors
                    ↓
            normalizeError()
                    ↓
          INormalizedError
                    ↓
               Component
                    ↓
                   UI
```

الـ Component يعرف **شكلًا واحدًا فقط**.

---

# وده اسمه Abstraction

إحنا عملنا **Abstraction** فوق تفاصيل الـ Error.

بدل ما الـ Component يعرف:

```ts
error.response?.data.message
error.response?.status
error.code
error.request
```

نقول له:

```ts
normalizedError.message
normalizedError.status
normalizedError.code
```

فالـ Component يتعامل مع **our application error format** بدل تفاصيل Axios.

---

# مثال من الحياة

تخيل مستشفى فيها:

```text
مريض 1 → صداع
مريض 2 → حرارة
مريض 3 → كسر
مريض 4 → ضغط
```

الدكتور لا يريد أن كل قسم يستخدم نظامًا مختلفًا لتسجيل المرضى.

يتم تحويل كل حالة إلى:

```text
Patient Record
├── name
├── diagnosis
├── severity
└── notes
```

ده نوع من **Normalization**.

نفس الفكرة:

```text
Axios Error
Network Error
Timeout
HTTP Error
       ↓
       ↓
Normalized Error
├── message
├── status
└── code
```

---

# ⚠️ وهنا نقطة مهمة جدًا

أنت ممكن تسأل:

> "طب ليه ما نستخدمش `AxiosError` وخلاص؟"

لأننا لا نريد أن يكون **الـ application كله مربوطًا بـ Axios**.

لو بعد سنة قررت تغير:

```text
Axios
```

إلى:

```text
fetch
```

أو أي HTTP client آخر، مش عايز تعدل عشرات الـ Components.

نريد:

```text
Component
    ↓
Application Error
```

وليس:

```text
Component
    ↓
Axios-specific Error
```

وده سبب معماري قوي جدًا.

---

## 🧠 احفظ الجملة دي

> **Error Normalization = تحويل أنواع وأشكال الأخطاء المختلفة إلى شكل موحّد يستطيع باقي التطبيق التعامل معه بسهولة.**

```text
HTTP Error ───┐
Network Error ├──→ normalizeError() → INormalizedError
Timeout ──────┤
Abort ────────┤
Unknown ──────┘
```

والآن أصبحت فاهم **لماذا** نعمل Normalization، وليس فقط **كيف** نكتب function.

**الخطوة التالية:** سنأخذ `404` حقيقيًا و`Network Error` حقيقيًا، ونمشي بهما داخل `normalizeError` خطوة بخطوة حتى ترى التحويل بعينك.
</details>
Normalization
محتاج شرح اكثر وضوحا وفهما
ChatGPT
**خد 15 دقيقة، وانسَ الكود مؤقتًا، وافهم `Error Normalization` كفكرة معمارية من خلال مثال واحد واضح.**

## 📍 Roadmap

```text
SECTION 0  Architecture Foundation       ⬜
SECTION 1  Axios Fundamentals             ✅
SECTION 2  TypeScript + Axios             ✅
SECTION 3  Error Handling                 🟢
  ├─ Fundamentals                         ✅
  ├─ HTTP Errors                          ✅
  ├─ Network Errors                       ✅
  ├─ AxiosError                            ✅
  ├─ Error Normalization                  🟡 ← نحن هنا
  └─ Custom ApiError                      ⬜
```

# يعني إيه Normalization أصلًا؟

كلمة **Normalize** معناها ببساطة:

> **أحوّل أشكال مختلفة إلى شكل موحّد.**

مثلاً عندك ناس بتكتب رقم التليفون بأشكال مختلفة:

```text
01012345678
+201012345678
00201012345678
```

لو أنت عايز النظام كله يتعامل مع صيغة واحدة، تعمل:

```text
Different Formats
       ↓
    Normalize
       ↓
One Standard Format
```

نفس الفكرة بالضبط مع الـ Errors.

---

# المشكلة التي نحلها

تخيل أن عندك API.

ممكن تحصل أخطاء كثيرة جدًا:

### الحالة الأولى

السيرفر يرد:

```text
HTTP 404
```

والـ API يقول:

```json
{
  "status": false,
  "code": 1001,
  "message": "User not found"
}
```

---

### الحالة الثانية

الإنترنت يفصل:

```text
Network Error
```

ومفيش `response` أصلًا.

---

### الحالة الثالثة

الـ request يأخذ وقتًا أطول من:

```text
5000 ms
```

فيحدث:

```text
Timeout
```

---

### الحالة الرابعة

المستخدم يلغي الـ request:

```text
Abort
```

---

إذن عندنا أشكال مختلفة جدًا:

```text
HTTP Error
Network Error
Timeout
Abort
Unknown Error
```

ولو سيبنا كل Component يتعامل مع كل نوع بنفسه، هنقع في مشكلة.

---

# شوف الـ Component بدون Normalization

مثلاً:

```tsx
try {
  const user = await getUser(10);
} catch (error) {

  if (axios.isAxiosError(error)) {

    if (error.response) {
      // HTTP Error
    }

    if (error.request) {
      // Network Error
    }

    if (error.code === "ETIMEDOUT") {
      // Timeout
    }

  } else {
    // Unknown Error
  }
}
```

تخيل إن عندك:

```text
20 Components
```

كل Component يعمل الكلام ده.

😵‍💫

هنا الـ Components بدأت تعرف تفاصيل Axios.

---

# هنا يأتي Error Normalization

نقول:

> **أنا مش عايز الـ Component يعرف كل أنواع الـ Errors.**

أعمل function واحدة:

```text
              Different Errors
                     ↓
              normalizeError()
                     ↓
             Standard Error
```

مثلاً:

```ts
interface INormalizedError {
  message: string;
  status?: number;
  code?: string;
}
```

دلوقتي مهما كان الخطأ:

### 404

ندخله:

```text
AxiosError
```

ويخرج:

```ts
{
  message: "User not found",
  status: 404
}
```

### Network Error

يدخل:

```text
AxiosError
```

ويخرج:

```ts
{
  message: "Network error",
  code: "ERR_NETWORK"
}
```

### Timeout

يدخل:

```text
AxiosError
```

ويخرج:

```ts
{
  message: "Request timed out",
  code: "ETIMEDOUT"
}
```

---

# إذن الـ Normalization مش بيحل الـ Error

دي نقطة مهمة جدًا.

لو السيرفر رجع:

```text
404
```

`normalizeError` **مش هتحول 404 إلى 200**.

ولا تصلح المشكلة.

هي فقط تقول:

> "أنا عندي Error شكله كذا، هحوّله للشكل القياسي بتاعي."

يعني:

```text
404
↓
AxiosError
↓
normalizeError()
↓
{
  message: "...",
  status: 404
}
```

---

# طب ليه ده مفيد؟

لأن الـ Component بعد كده يتعامل مع حاجة واحدة:

```tsx
try {
  const user = await getUser(10);
} catch (error) {

  const normalizedError = normalizeError(error);

  console.log(normalizedError.message);
}
```

الـ Component لا يهتم:

```text
هل 404؟
هل Network؟
هل Timeout؟
هل Axios؟
هل response موجودة؟
هل data موجودة؟
```

كل ده مسؤولية:

```text
normalizeError()
```

---

# شوف الفرق المعماري

### ❌ بدون Normalization

```text
Component
   │
   ├── AxiosError
   ├── HTTP Error
   ├── Network Error
   ├── Timeout
   └── Abort
```

الـ Component أصبح يعرف تفاصيل كثيرة.

---

### ✅ مع Normalization

```text
                  Axios
                    ↓
              Different Errors
                    ↓
            normalizeError()
                    ↓
          INormalizedError
                    ↓
               Component
                    ↓
                   UI
```

الـ Component يعرف **شكلًا واحدًا فقط**.

---

# وده اسمه Abstraction

إحنا عملنا **Abstraction** فوق تفاصيل الـ Error.

بدل ما الـ Component يعرف:

```ts
error.response?.data.message
error.response?.status
error.code
error.request
```

نقول له:

```ts
normalizedError.message
normalizedError.status
normalizedError.code
```

فالـ Component يتعامل مع **our application error format** بدل تفاصيل Axios.

---

# مثال من الحياة

تخيل مستشفى فيها:

```text
مريض 1 → صداع
مريض 2 → حرارة
مريض 3 → كسر
مريض 4 → ضغط
```

الدكتور لا يريد أن كل قسم يستخدم نظامًا مختلفًا لتسجيل المرضى.

يتم تحويل كل حالة إلى:

```text
Patient Record
├── name
├── diagnosis
├── severity
└── notes
```

ده نوع من **Normalization**.

نفس الفكرة:

```text
Axios Error
Network Error
Timeout
HTTP Error
       ↓
       ↓
Normalized Error
├── message
├── status
└── code
```

---

# ⚠️ وهنا نقطة مهمة جدًا

أنت ممكن تسأل:

> "طب ليه ما نستخدمش `AxiosError` وخلاص؟"

لأننا لا نريد أن يكون **الـ application كله مربوطًا بـ Axios**.

لو بعد سنة قررت تغير:

```text
Axios
```

إلى:

```text
fetch
```

أو أي HTTP client آخر، مش عايز تعدل عشرات الـ Components.

نريد:

```text
Component
    ↓
Application Error
```

وليس:

```text
Component
    ↓
Axios-specific Error
```

وده سبب معماري قوي جدًا.

---

## 🧠 احفظ الجملة دي

> **Error Normalization = تحويل أنواع وأشكال الأخطاء المختلفة إلى شكل موحّد يستطيع باقي التطبيق التعامل معه بسهولة.**

```text
HTTP Error ───┐
Network Error ├──→ normalizeError() → INormalizedError
Timeout ──────┤
Abort ────────┤
Unknown ──────┘
```

والآن أصبحت فاهم **لماذا** نعمل Normalization، وليس فقط **كيف** نكتب function.

**الخطوة التالية:** سنأخذ `404` حقيقيًا و`Network Error` حقيقيًا، ونمشي بهما داخل `normalizeError` خطوة بخطوة حتى ترى التحويل بعينك.
<details>
<summary>كمل</summary>
user
كمل
ChatGPT
**خد 15 دقيقة: هنمشي بـ `404` و`Network Error` داخل `normalizeError` خطوة بخطوة، وبعدها هتكتب النسخة بنفسك.**

## 📍 Roadmap

```text
SECTION 0  Architecture Foundation       ⬜
SECTION 1  Axios Fundamentals             ✅
SECTION 2  TypeScript + Axios             ✅
SECTION 3  Error Handling                 🟢
  ├─ Fundamentals                         ✅
  ├─ HTTP Errors                          ✅
  ├─ Network Errors                       ✅
  ├─ AxiosError                           ✅
  ├─ Error Normalization                  🟡 ← نحن هنا
  └─ Custom ApiError                      ⬜
```

# الحالة الأولى: `404`

افترض أن عندنا:

```ts
try {
  await axios.get("/users/999");
} catch (error) {
  const normalizedError = normalizeError(error);
}
```

والسيرفر رد:

```text
HTTP Status = 404
```

والـ API response:

```json
{
  "status": false,
  "code": 1001,
  "message": "User not found"
}
```

### ماذا يحدث؟

أولًا:

```text
axios.get()
     ↓
Server
     ↓
404
     ↓
AxiosError
     ↓
catch(error)
```

ندخل `error` إلى:

```ts
normalizeError(error)
```

---

# داخل `normalizeError`

أول سؤال:

```ts
axios.isAxiosError(error)
```

هل هو Axios Error؟

```text
YES ✅
```

إذن نقدر نتعامل معه كـ `AxiosError`.

بعدها نسأل:

```ts
error.response
```

هل عندنا response؟

```text
YES ✅
```

لأن السيرفر رد.

إذن نستطيع الوصول إلى:

```ts
error.response.status
```

فتكون:

```text
404
```

ونستطيع الوصول إلى:

```ts
error.response.data
```

فتكون:

```json
{
  "status": false,
  "code": 1001,
  "message": "User not found"
}
```

إذن نقدر نخرج:

```ts
{
  message: "User not found",
  status: 404
}
```

لاحظ:

```text
HTTP status
    ↓
404

API message
    ↓
"User not found"
```

إحنا **اخترنا** المعلومات المهمة وحولناها إلى شكل التطبيق الموحد.

---

# الحالة الثانية: Network Error

نفس الكود:

```ts
try {
  await axios.get("/users");
} catch (error) {
  const normalizedError = normalizeError(error);
}
```

لكن هذه المرة:

```text
Client
  ↓
Network
  ↓
❌
```

مفيش HTTP response.

إذن:

```ts
error.response
```

تكون:

```text
undefined
```

فمينفعش نعمل:

```ts
error.response.data.message
```

لأن:

```text
undefined.data
```

❌

لكن `error.message` ممكن تكون:

```text
"Network Error"
```

إذن نرجع:

```ts
{
  message: "Network Error",
  code: "ERR_NETWORK"
}
```

---

# لاحظ العبقرية البسيطة هنا

نفس الـ function:

```ts
normalizeError(error)
```

تستقبل:

```text
HTTP Error
```

وتخرج:

```ts
{
  message: "User not found",
  status: 404
}
```

وتستقبل:

```text
Network Error
```

وتخرج:

```ts
{
  message: "Network Error",
  code: "ERR_NETWORK"
}
```

والـ Component لا يحتاج يعرف التفاصيل.

---

# لكن عندنا مشكلة TypeScript

لو كتبنا:

```ts
error.response?.data.message
```

TypeScript مش بالضرورة يعرف إن:

```ts
data
```

شكلها:

```ts
IErrorResponse
```

هو ممكن يعرفها كـ:

```text
any
```

أو شكل غير محدد حسب طريقة استخدام `AxiosError`.

وإحنا لا نريد `any`.

إحنا عايزين نقول لـ Axios:

> الـ `data` الموجودة داخل الـ response الخاصة بالـ Error شكلها `IErrorResponse`.

وهنا يظهر مفهوم مهم جدًا:

```ts
AxiosError<IErrorResponse>
```

---

# انتبه للـ Generic

أنت درست قبل كده:

```ts
axios.get<IApiResponse<IUser>>(...)
```

وقلنا:

```text
Generic
   ↓
يحدد نوع response.data
```

نفس الفكرة هنا:

```ts
AxiosError<IErrorResponse>
```

معناها:

```text
AxiosError
    ↓
response.data
    ↓
IErrorResponse
```

فتصبح TypeScript عارفة أن:

```ts
error.response?.data.message
```

هي:

```text
string
```

---

# الصورة الكاملة

```text
                 AxiosError<IErrorResponse>
                         │
                         ↓
                    response?
                         │
                ┌────────┴────────┐
                ↓                 ↓
               YES                NO
                │                 │
                ↓                 ↓
        response.status      Network/etc.
                │                 │
                ↓                 ↓
              404          error.message
                │                 │
                ↓                 ↓
        response.data       "Network Error"
                │
                ↓
       IErrorResponse
                │
                ↓
      data.message
                │
                ↓
        "User not found"
```

وده هو قلب `Error Normalization`.

---

# ليه ما نرجعش الـ AxiosError نفسه؟

ممكن تعمل:

```ts
return error;
```

لكن كده أنت بتقول للـ application:

> اتفضل، تعامل مع Axios وتفاصيله.

وده عكس الهدف.

إحنا عايزين:

```text
Axios-specific Error
        ↓
     Normalize
        ↓
Application Error
```

---

# الآن نقدر نفهم الكود

النسخة المبدئية هتكون بهذا الشكل:

```ts
interface INormalizedError {
  message: string;
  status?: number;
  code?: string;
}
```

ثم:

```ts
function normalizeError(
  error: unknown
): INormalizedError {

  if (axios.isAxiosError(error)) {

    return {
      message: error.message,
      status: error.response?.status,
      code: error.code,
    };
  }

  return {
    message: "Something went wrong",
  };
}
```

لكن **دي لسه نسخة ناقصة**.

لأننا عايزين بدل:

```ts
message: error.message
```

في حالة وجود API message، نستخدم:

```ts
error.response?.data.message
```

وده هو الجزء اللي هنكتبه بعد ما نفهم `AxiosError<IErrorResponse>`.

---

## 🧠 إذن وصلنا لإيه؟

```text
error: unknown
       ↓
isAxiosError()
       ↓
AxiosError
       ↓
هل response موجودة؟
   ↓              ↓
 YES             NO
   ↓              ↓
API message    error.message
HTTP status
       ↓
Normalized Error
```

**الخطوة التالية:** اكتب لي تعريف `IErrorResponse` الذي استخدمناه سابقًا، وبعده سنكتب `AxiosError<IErrorResponse>` ونربطه بـ `normalizeError` خطوة بخطوة.
</details>
كمل
ChatGPT
**خد 15 دقيقة: هنمشي بـ `404` و`Network Error` داخل `normalizeError` خطوة بخطوة، وبعدها هتكتب النسخة بنفسك.**

## 📍 Roadmap

```text
SECTION 0  Architecture Foundation       ⬜
SECTION 1  Axios Fundamentals             ✅
SECTION 2  TypeScript + Axios             ✅
SECTION 3  Error Handling                 🟢
  ├─ Fundamentals                         ✅
  ├─ HTTP Errors                          ✅
  ├─ Network Errors                       ✅
  ├─ AxiosError                           ✅
  ├─ Error Normalization                  🟡 ← نحن هنا
  └─ Custom ApiError                      ⬜
```

# الحالة الأولى: `404`

افترض أن عندنا:

```ts
try {
  await axios.get("/users/999");
} catch (error) {
  const normalizedError = normalizeError(error);
}
```

والسيرفر رد:

```text
HTTP Status = 404
```

والـ API response:

```json
{
  "status": false,
  "code": 1001,
  "message": "User not found"
}
```

### ماذا يحدث؟

أولًا:

```text
axios.get()
     ↓
Server
     ↓
404
     ↓
AxiosError
     ↓
catch(error)
```

ندخل `error` إلى:

```ts
normalizeError(error)
```

---

# داخل `normalizeError`

أول سؤال:

```ts
axios.isAxiosError(error)
```

هل هو Axios Error؟

```text
YES ✅
```

إذن نقدر نتعامل معه كـ `AxiosError`.

بعدها نسأل:

```ts
error.response
```

هل عندنا response؟

```text
YES ✅
```

لأن السيرفر رد.

إذن نستطيع الوصول إلى:

```ts
error.response.status
```

فتكون:

```text
404
```

ونستطيع الوصول إلى:

```ts
error.response.data
```

فتكون:

```json
{
  "status": false,
  "code": 1001,
  "message": "User not found"
}
```

إذن نقدر نخرج:

```ts
{
  message: "User not found",
  status: 404
}
```

لاحظ:

```text
HTTP status
    ↓
404

API message
    ↓
"User not found"
```

إحنا **اخترنا** المعلومات المهمة وحولناها إلى شكل التطبيق الموحد.

---

# الحالة الثانية: Network Error

نفس الكود:

```ts
try {
  await axios.get("/users");
} catch (error) {
  const normalizedError = normalizeError(error);
}
```

لكن هذه المرة:

```text
Client
  ↓
Network
  ↓
❌
```

مفيش HTTP response.

إذن:

```ts
error.response
```

تكون:

```text
undefined
```

فمينفعش نعمل:

```ts
error.response.data.message
```

لأن:

```text
undefined.data
```

❌

لكن `error.message` ممكن تكون:

```text
"Network Error"
```

إذن نرجع:

```ts
{
  message: "Network Error",
  code: "ERR_NETWORK"
}
```

---

# لاحظ العبقرية البسيطة هنا

نفس الـ function:

```ts
normalizeError(error)
```

تستقبل:

```text
HTTP Error
```

وتخرج:

```ts
{
  message: "User not found",
  status: 404
}
```

وتستقبل:

```text
Network Error
```

وتخرج:

```ts
{
  message: "Network Error",
  code: "ERR_NETWORK"
}
```

والـ Component لا يحتاج يعرف التفاصيل.

---

# لكن عندنا مشكلة TypeScript

لو كتبنا:

```ts
error.response?.data.message
```

TypeScript مش بالضرورة يعرف إن:

```ts
data
```

شكلها:

```ts
IErrorResponse
```

هو ممكن يعرفها كـ:

```text
any
```

أو شكل غير محدد حسب طريقة استخدام `AxiosError`.

وإحنا لا نريد `any`.

إحنا عايزين نقول لـ Axios:

> الـ `data` الموجودة داخل الـ response الخاصة بالـ Error شكلها `IErrorResponse`.

وهنا يظهر مفهوم مهم جدًا:

```ts
AxiosError<IErrorResponse>
```

---

# انتبه للـ Generic

أنت درست قبل كده:

```ts
axios.get<IApiResponse<IUser>>(...)
```

وقلنا:

```text
Generic
   ↓
يحدد نوع response.data
```

نفس الفكرة هنا:

```ts
AxiosError<IErrorResponse>
```

معناها:

```text
AxiosError
    ↓
response.data
    ↓
IErrorResponse
```

فتصبح TypeScript عارفة أن:

```ts
error.response?.data.message
```

هي:

```text
string
```

---

# الصورة الكاملة

```text
                 AxiosError<IErrorResponse>
                         │
                         ↓
                    response?
                         │
                ┌────────┴────────┐
                ↓                 ↓
               YES                NO
                │                 │
                ↓                 ↓
        response.status      Network/etc.
                │                 │
                ↓                 ↓
              404          error.message
                │                 │
                ↓                 ↓
        response.data       "Network Error"
                │
                ↓
       IErrorResponse
                │
                ↓
      data.message
                │
                ↓
        "User not found"
```

وده هو قلب `Error Normalization`.

---

# ليه ما نرجعش الـ AxiosError نفسه؟

ممكن تعمل:

```ts
return error;
```

لكن كده أنت بتقول للـ application:

> اتفضل، تعامل مع Axios وتفاصيله.

وده عكس الهدف.

إحنا عايزين:

```text
Axios-specific Error
        ↓
     Normalize
        ↓
Application Error
```

---

# الآن نقدر نفهم الكود

النسخة المبدئية هتكون بهذا الشكل:

```ts
interface INormalizedError {
  message: string;
  status?: number;
  code?: string;
}
```

ثم:

```ts
function normalizeError(
  error: unknown
): INormalizedError {

  if (axios.isAxiosError(error)) {

    return {
      message: error.message,
      status: error.response?.status,
      code: error.code,
    };
  }

  return {
    message: "Something went wrong",
  };
}
```

لكن **دي لسه نسخة ناقصة**.

لأننا عايزين بدل:

```ts
message: error.message
```

في حالة وجود API message، نستخدم:

```ts
error.response?.data.message
```

وده هو الجزء اللي هنكتبه بعد ما نفهم `AxiosError<IErrorResponse>`.

---

## 🧠 إذن وصلنا لإيه؟

```text
error: unknown
       ↓
isAxiosError()
       ↓
AxiosError
       ↓
هل response موجودة؟
   ↓              ↓
 YES             NO
   ↓              ↓
API message    error.message
HTTP status
       ↓
Normalized Error
```

**الخطوة التالية:** اكتب لي تعريف `IErrorResponse` الذي استخدمناه سابقًا، وبعده سنكتب `AxiosError<IErrorResponse>` ونربطه بـ `normalizeError` خطوة بخطوة.
<details>
<summary>حل انت</summary>
user
حل انت
ChatGPT
**خد 10 دقائق، وخلينا نبني الحل كاملًا من الـ `IErrorResponse` لحد `normalizeError` عشان تشوف كل Type رايح فين.**

## 📍 Roadmap

```text
SECTION 0  Architecture Foundation       ⬜
SECTION 1  Axios Fundamentals             ✅
SECTION 2  TypeScript + Axios              ✅
SECTION 3  Error Handling                 🟢
  ├─ Fundamentals                         ✅
  ├─ HTTP Errors                          ✅
  ├─ Network Errors                       ✅
  ├─ AxiosError                           ✅
  ├─ Error Normalization                  🟡 ← نحن هنا
  └─ Custom ApiError                      ⬜
SECTION 4  Interceptors                   ⬜
...
```

## 1. أولًا: شكل Error Response بتاع الـ API

إحنا سبق وعملنا:

```ts
export interface IErrorResponse<R = unknown> {
  status: false;
  code: number;
  message: string;
  errors: R[];
}
```

مثلاً السيرفر ممكن يرجع:

```json
{
  "status": false,
  "code": 1004,
  "message": "User not found",
  "errors": []
}
```

إذن:

```text
error.response.data
        ↓
IErrorResponse
```

---

## 2. نحدد نوع الـ AxiosError

بدل ما نسيب Axios يخمن نوع `data`:

```ts
AxiosError
```

نقول له:

```ts
AxiosError<IErrorResponse>
```

معناها:

```text
AxiosError
   ↓
response
   ↓
data
   ↓
IErrorResponse
```

وبالتالي TypeScript يفهم:

```ts
error.response?.data.message
```

على أنها:

```ts
string
```

---

# 3. نكتب `normalizeError`

```ts
import axios, { type AxiosError } from "axios";

export interface INormalizedError {
  message: string;
  status?: number;
  code?: string;
}

export interface IErrorResponse<R = unknown> {
  status: false;
  code: number;
  message: string;
  errors: R[];
}

export function normalizeError(
  error: unknown
): INormalizedError {

  if (axios.isAxiosError<IErrorResponse>(error)) {

    return {
      message:
        error.response?.data.message ??
        error.message,

      status: error.response?.status,

      code: error.code,
    };
  }

  return {
    message: "Something went wrong",
  };
}
```

خلينا نفك الكود **سطر سطر**.

---

## `error: unknown`

```ts
error: unknown
```

إحنا لا نثق في الـ Error.

ممكن يكون:

```text
AxiosError
Error
string
object
anything
```

فبنبدأ بـ `unknown`.

---

## `axios.isAxiosError<IErrorResponse>(error)`

```ts
if (axios.isAxiosError<IErrorResponse>(error)) {
```

هنا حاجتين حصلوا:

### أولًا

اتأكدنا إن:

```text
error
↓
AxiosError
```

### ثانيًا

قلنا إن الـ API error data شكلها:

```text
IErrorResponse
```

فـ TypeScript أصبح فاهم:

```ts
error.response?.data
```

على أنها:

```ts
IErrorResponse
```

---

# 4. أهم سطر

```ts
message:
  error.response?.data.message ??
  error.message,
```

هنا عندنا **Fallback**.

المعنى:

> لو الـ API بعت `message`، استخدمها.  
> لو مفيش API message، استخدم `error.message`.

مثلاً:

### API رجع 404

```json
{
  "status": false,
  "code": 1004,
  "message": "User not found",
  "errors": []
}
```

إذن:

```text
error.response?.data.message
            ↓
"User not found"
```

فتكون النتيجة:

```ts
{
  message: "User not found",
  status: 404
}
```

---

### Network Error

مفيش:

```ts
error.response
```

إذن:

```ts
error.response?.data.message
```

تكون:

```text
undefined
```

فيستخدم الجزء الثاني:

```ts
error.message
```

مثلاً:

```text
"Network Error"
```

فتكون:

```ts
{
  message: "Network Error",
  code: "ERR_NETWORK"
}
```

---

# 5. ليه استخدمنا `??`؟

```ts
a ?? b
```

معناها:

> استخدم `a` لو لها قيمة، ولو `a` هي `null` أو `undefined` استخدم `b`.

فـ:

```ts
error.response?.data.message ??
error.message
```

يعني:

```text
API message موجودة؟
       │
    ┌──┴──┐
   YES    NO
    ↓      ↓
API msg  Axios msg
```

---

# 6. لاحظ الـ Normalization حصل إزاي

قبل:

```text
AxiosError
├── response
│   ├── status
│   └── data
│       ├── message
│       ├── code
│       └── ...
├── message
├── code
└── request
```

بعد:

```text
INormalizedError
├── message
├── status
└── code
```

إحنا **أخذنا المعلومات اللي application محتاجها فقط**.

وده هو الـ Normalization.

---

## ⚠️ ملاحظة معمارية مهمة

هنا:

```ts
status: error.response?.status
```

إحنا اخترنا **HTTP status**.

وهنا:

```ts
code: error.code
```

إحنا اخترنا **Axios error code**.

لكن ممكن في تصميم مختلف تختار API code:

```ts
code: String(error.response?.data.code)
```

وده قرار معماري حسب الـ error contract اللي عايز التطبيق كله يتعامل معاه.

المهم إنك تكون **واعي أنت اخترت أي code وليه**.

---

## 🧠 الصورة النهائية

```text
                error: unknown
                       ↓
              isAxiosError()
                       ↓
              AxiosError<IErrorResponse>
                       ↓
             ┌─────────┴─────────┐
             ↓                   ↓
       response موجودة      response غير موجودة
             ↓                   ↓
       API message          error.message
       HTTP status           Axios code
             │                   │
             └─────────┬─────────┘
                       ↓
                INormalizedError
```

إحنا كده **فهمنا Error Normalization فعليًا**، ولسه ما دخلناش في `Custom ApiError`.

**الخطوة التالية:** ننتقل إلى **Custom `ApiError`** ونفهم ليه نحتاج نعمل Error class خاصة بالتطبيق بدل ما نرجع `INormalizedError` عادي.
</details>
حل انت
ChatGPT
**خد 10 دقائق، وخلينا نبني الحل كاملًا من الـ `IErrorResponse` لحد `normalizeError` عشان تشوف كل Type رايح فين.**

## 📍 Roadmap

```text
SECTION 0  Architecture Foundation       ⬜
SECTION 1  Axios Fundamentals             ✅
SECTION 2  TypeScript + Axios              ✅
SECTION 3  Error Handling                 🟢
  ├─ Fundamentals                         ✅
  ├─ HTTP Errors                          ✅
  ├─ Network Errors                       ✅
  ├─ AxiosError                           ✅
  ├─ Error Normalization                  🟡 ← نحن هنا
  └─ Custom ApiError                      ⬜
SECTION 4  Interceptors                   ⬜
...
```

## 1. أولًا: شكل Error Response بتاع الـ API

إحنا سبق وعملنا:

```ts
export interface IErrorResponse<R = unknown> {
  status: false;
  code: number;
  message: string;
  errors: R[];
}
```

مثلاً السيرفر ممكن يرجع:

```json
{
  "status": false,
  "code": 1004,
  "message": "User not found",
  "errors": []
}
```

إذن:

```text
error.response.data
        ↓
IErrorResponse
```

---

## 2. نحدد نوع الـ AxiosError

بدل ما نسيب Axios يخمن نوع `data`:

```ts
AxiosError
```

نقول له:

```ts
AxiosError<IErrorResponse>
```

معناها:

```text
AxiosError
   ↓
response
   ↓
data
   ↓
IErrorResponse
```

وبالتالي TypeScript يفهم:

```ts
error.response?.data.message
```

على أنها:

```ts
string
```

---

# 3. نكتب `normalizeError`

```ts
import axios, { type AxiosError } from "axios";

export interface INormalizedError {
  message: string;
  status?: number;
  code?: string;
}

export interface IErrorResponse<R = unknown> {
  status: false;
  code: number;
  message: string;
  errors: R[];
}

export function normalizeError(
  error: unknown
): INormalizedError {

  if (axios.isAxiosError<IErrorResponse>(error)) {

    return {
      message:
        error.response?.data.message ??
        error.message,

      status: error.response?.status,

      code: error.code,
    };
  }

  return {
    message: "Something went wrong",
  };
}
```

خلينا نفك الكود **سطر سطر**.

---

## `error: unknown`

```ts
error: unknown
```

إحنا لا نثق في الـ Error.

ممكن يكون:

```text
AxiosError
Error
string
object
anything
```

فبنبدأ بـ `unknown`.

---

## `axios.isAxiosError<IErrorResponse>(error)`

```ts
if (axios.isAxiosError<IErrorResponse>(error)) {
```

هنا حاجتين حصلوا:

### أولًا

اتأكدنا إن:

```text
error
↓
AxiosError
```

### ثانيًا

قلنا إن الـ API error data شكلها:

```text
IErrorResponse
```

فـ TypeScript أصبح فاهم:

```ts
error.response?.data
```

على أنها:

```ts
IErrorResponse
```

---

# 4. أهم سطر

```ts
message:
  error.response?.data.message ??
  error.message,
```

هنا عندنا **Fallback**.

المعنى:

> لو الـ API بعت `message`، استخدمها.  
> لو مفيش API message، استخدم `error.message`.

مثلاً:

### API رجع 404

```json
{
  "status": false,
  "code": 1004,
  "message": "User not found",
  "errors": []
}
```

إذن:

```text
error.response?.data.message
            ↓
"User not found"
```

فتكون النتيجة:

```ts
{
  message: "User not found",
  status: 404
}
```

---

### Network Error

مفيش:

```ts
error.response
```

إذن:

```ts
error.response?.data.message
```

تكون:

```text
undefined
```

فيستخدم الجزء الثاني:

```ts
error.message
```

مثلاً:

```text
"Network Error"
```

فتكون:

```ts
{
  message: "Network Error",
  code: "ERR_NETWORK"
}
```

---

# 5. ليه استخدمنا `??`؟

```ts
a ?? b
```

معناها:

> استخدم `a` لو لها قيمة، ولو `a` هي `null` أو `undefined` استخدم `b`.

فـ:

```ts
error.response?.data.message ??
error.message
```

يعني:

```text
API message موجودة؟
       │
    ┌──┴──┐
   YES    NO
    ↓      ↓
API msg  Axios msg
```

---

# 6. لاحظ الـ Normalization حصل إزاي

قبل:

```text
AxiosError
├── response
│   ├── status
│   └── data
│       ├── message
│       ├── code
│       └── ...
├── message
├── code
└── request
```

بعد:

```text
INormalizedError
├── message
├── status
└── code
```

إحنا **أخذنا المعلومات اللي application محتاجها فقط**.

وده هو الـ Normalization.

---

## ⚠️ ملاحظة معمارية مهمة

هنا:

```ts
status: error.response?.status
```

إحنا اخترنا **HTTP status**.

وهنا:

```ts
code: error.code
```

إحنا اخترنا **Axios error code**.

لكن ممكن في تصميم مختلف تختار API code:

```ts
code: String(error.response?.data.code)
```

وده قرار معماري حسب الـ error contract اللي عايز التطبيق كله يتعامل معاه.

المهم إنك تكون **واعي أنت اخترت أي code وليه**.

---

## 🧠 الصورة النهائية

```text
                error: unknown
                       ↓
              isAxiosError()
                       ↓
              AxiosError<IErrorResponse>
                       ↓
             ┌─────────┴─────────┐
             ↓                   ↓
       response موجودة      response غير موجودة
             ↓                   ↓
       API message          error.message
       HTTP status           Axios code
             │                   │
             └─────────┬─────────┘
                       ↓
                INormalizedError
```

إحنا كده **فهمنا Error Normalization فعليًا**، ولسه ما دخلناش في `Custom ApiError`.

**الخطوة التالية:** ننتقل إلى **Custom `ApiError`** ونفهم ليه نحتاج نعمل Error class خاصة بالتطبيق بدل ما نرجع `INormalizedError` عادي.
<details>
<summary>Custom ApiError</summary>
user
Custom ApiError
ChatGPT
**خد 15 دقيقة: افهم أولًا لماذا نحتاج `Custom ApiError`، ثم سنبني الـ class فوق `AxiosError` بدون القفز إلى Interceptors.**

## 📍 Roadmap

```text
SECTION 0  Architecture Foundation       ⬜
SECTION 1  Axios Fundamentals             ✅
SECTION 2  TypeScript + Axios              ✅
SECTION 3  Error Handling                 🟢
  ├─ Fundamentals                         ✅
  ├─ HTTP Errors                          ✅
  ├─ Network Errors                       ✅
  ├─ AxiosError                           ✅
  ├─ Error Normalization                  ✅
  └─ Custom ApiError                      🟡 ← نحن هنا
SECTION 4  Interceptors                    ⬜
...
```
</details>
Custom ApiError
ChatGPT
**خد 15 دقيقة: افهم أولًا لماذا نحتاج `Custom ApiError`، ثم سنبني الـ class فوق `AxiosError` بدون القفز إلى Interceptors.**

## 📍 Roadmap

```text
SECTION 0  Architecture Foundation       ⬜
SECTION 1  Axios Fundamentals             ✅
SECTION 2  TypeScript + Axios              ✅
SECTION 3  Error Handling                 🟢
  ├─ Fundamentals                         ✅
  ├─ HTTP Errors                          ✅
  ├─ Network Errors                       ✅
  ├─ AxiosError                           ✅
  ├─ Error Normalization                  ✅
  └─ Custom ApiError                      🟡 ← نحن هنا
SECTION 4  Interceptors                    ⬜
...
```