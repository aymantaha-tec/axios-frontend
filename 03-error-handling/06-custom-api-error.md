# لماذا `Custom ApiError`؟

إحنا وصلنا لمرحلة إن عندنا:

```ts
AxiosError
```

لكن فيه مشكلة معمارية:

**`AxiosError` تابع لـ Axios.**

يعني لو كل التطبيق بدأ يتعامل مع:

```ts
AxiosError
```

فالتطبيق أصبح مربوطًا بـ Axios.

إحنا عايزين application-level Error خاص بنا:

```text
AxiosError
    ↓
   normalize
    ↓
ApiError
```

يعني:

> **Axios يعرف تفاصيل الـ HTTP، لكن الـ application يتعامل مع `ApiError`.**

---

# أولًا: ما هو Custom Error؟

JavaScript عندها أصلًا:

```ts
Error
```

مثلاً:

```ts
throw new Error("Something went wrong");
```

لكن ممكن أنت تقول:

> أنا عايز Error خاص بالـ API بتاعي، ويحتوي على معلومات إضافية.

فننشئ:

```ts
class ApiError extends Error {
}
```

دلوقتي عندنا نوع Error خاص بنا.

---

# نضيف المعلومات التي نحتاجها

مثلاً:

```ts
class ApiError extends Error {
  status?: number;
  code?: string;
}
```

فتصبح:

```text
ApiError
├── message
├── status
└── code
```

لاحظ أن `message` أصلًا موجودة في `Error`.

---

# لماذا `extends Error`؟

لأننا نريد `ApiError` تكون **Error حقيقية** في JavaScript، لكن مع معلومات إضافية.

يعني:

```text
Error
   ↑
   │ extends
   │
ApiError
```

وبالتالي نقدر نعمل:

```ts
throw new ApiError("User not found");
```

والـ `catch` يقدر يمسكها كـ Error.

---

# نكتب Constructor

```ts
class ApiError extends Error {
  status?: number;
  code?: string;

  constructor(
    message: string,
    status?: number,
    code?: string
  ) {
    super(message);

    this.status = status;
    this.code = code;
  }
}
```

خلينا نفهم:

```ts
super(message);
```

دي مهمة جدًا.

لأن `ApiError` ورثت من:

```ts
Error
```

فالـ `super()` تستدعي constructor الخاص بالـ parent class.

بمعنى:

```text
ApiError
   ↓
extends Error
   ↓
super(message)
   ↓
Error.message = message
```

---

# استخدامها

```ts
const error = new ApiError(
  "User not found",
  404,
  "USER_NOT_FOUND"
);
```

الـ object أصبح تقريبًا:

```text
ApiError
├── message
│   → "User not found"
│
├── status
│   → 404
│
└── code
    → "USER_NOT_FOUND"
```

وتقدر تعمل:

```ts
console.log(error.message);
console.log(error.status);
console.log(error.code);
```

---

# طيب Axios دخل فين؟

هنا الجزء المهم.

عندنا:

```text
Server
   ↓
HTTP 404
   ↓
AxiosError
```

إحنا لا نريد أن نرمي `AxiosError` إلى باقي التطبيق.

نريد:

```text
AxiosError
    ↓
extract information
    ↓
ApiError
    ↓
Application
```

مثلاً:

```ts
throw new ApiError(
  "User not found",
  404,
  "USER_NOT_FOUND"
);
```

الـ Component أو باقي التطبيق لا يعرف أن الخطأ بدأ كـ `AxiosError`.

---

# مثال كامل

```ts
try {
  await axios.get("/users/999");
} catch (error) {

  if (axios.isAxiosError<IErrorResponse>(error)) {

    throw new ApiError(
      error.response?.data.message ?? error.message,
      error.response?.status,
      error.code
    );
  }

  throw new ApiError("Something went wrong");
}
```

لاحظ الـ flow:

```text
Axios
  ↓
AxiosError
  ↓
isAxiosError()
  ↓
extract information
  ↓
new ApiError(...)
  ↓
throw
```

---

# وهنا سؤال مهم جدًا

ليه نعمل:

```ts
throw new ApiError(...)
```

بدل:

```ts
return {
  message: "...",
  status: 404
};
```

لأن الـ Error المفروض يمثل **حالة فشل**.

إحنا عايزين نستفيد من JavaScript error mechanism:

```ts
try {
  ...
} catch (error) {
  ...
}
```

بدل ما كل function ترجع:

```text
Success
or
Error Object
```

---

# الشكل المعماري

```text
             HTTP/API
                 ↓
              Axios
                 ↓
            AxiosError
                 ↓
       Error Normalization
                 ↓
             ApiError
                 ↓
          Application
                 ↓
           Components
```

وده separation مهم جدًا:

```text
Axios
→ يعرف Axios

ApiError
→ يعرفه Application

Component
→ يعرف ApiError فقط
```

---

## ⚠️ لكن عندنا مشكلة في الكود السابق

إحنا عملنا:

```ts
code?: string;
```

لكن أنت عندك API response فيه:

```ts
code: number;
```

وكمان Axios عنده:

```ts
error.code
```

غالبًا `string`.

إذن عندنا مرة أخرى:

```text
Axios code
≠
API code
```

وده قرار لازم نحدده في تصميم `ApiError`.

مثلاً ممكن نعمل:

```ts
class ApiError<T = unknown> extends Error {
  status?: number;
  code?: string;
  data?: T;
}
```

لكن **مش هنضيف `data` دلوقتي** قبل ما نفهم لماذا نحتاجها.

---

# 🧠 الخلاصة

`Custom ApiError` هو:

> **Error class خاصة بالـ application، نستبدل بها `AxiosError` بعد ما نستخرج منه المعلومات المهمة.**

بدل:

```text
Application
    ↓
AxiosError
```

نريد:

```text
Application
    ↓
ApiError
```

والـ Axios يبقى في الطبقة الخاصة بالـ API.

---

## 🎯 Interview Question

عندك:

```text
AxiosError
    ↓
ApiError
    ↓
Component
```

**ليه مش نخلي الـ Component يتعامل مباشرة مع `AxiosError`؟**

جاوب من فهمك، وبعدها نعمل **Section 3 Final Review** قبل الانتقال إلى Section 0.
<details>
<summary>الصراحه درس ال class ده صعب بنسبلي هل مهم ام لا</summary>
user
الصراحه درس ال class ده صعب بنسبلي هل مهم ام لا
ChatGPT
**خد 5 دقائق فقط: ما توقفش الكورس بسبب `class`، لأنها مهمة لكن مش لازم تتعمق فيها الآن.**

أيوه، **`Custom ApiError` مهم في الشغل الحقيقي**، لكن بالنسبة لمسارك الحالي في Axios، مش محتاج تتقن الـ `class` نفسها بعمق دلوقتي.

### إيه المطلوب منك الآن؟

يكفي تفهم الفكرة:

```text
AxiosError
   ↓
Normalize
   ↓
ApiError
   ↓
Component
```

وأن:

```ts
class ApiError extends Error
```

معناها:

> أنا عملت نوع Error خاص بالتطبيق، مبني على الـ `Error` الأصلي، وأضفت له معلومات أحتاجها مثل `status` و`code`.

مش مطلوب منك حاليًا تحفظ:

```ts
constructor()
super()
extends
this
```

بالتفصيل.

---

### ليه ما أنصحكش نغوص في `class` الآن؟

لأن هدفنا الأساسي هو **Axios Architecture**، وليس دراسة JavaScript OOP من البداية.

ولو دخلنا دلوقتي في:

```text
class
constructor
extends
super
inheritance
prototype
this
static
private fields
```

هنخرج من مسار Axios ونعمل تشتيت.

وأنت أصلًا درست **OOP في C++**، فالمفاهيم دي مش غريبة عليك، لكن JavaScript لها تفاصيل مختلفة، وهنقدر نرجع لها في وقت مناسب.

### إذن نعمل إيه؟

نعتبر:

```text
Custom ApiError
```

🟡 **مفهومه مهم، لكن تفاصيل `class` مؤجلة.**

ونكمل الكورس بدون ما نضيّع وقتنا في نقطة مش هي هدف المرحلة الحالية.

## 📍 Roadmap

```text
SECTION 3  Error Handling
  ├─ Fundamentals          ✅
  ├─ HTTP Errors           ✅
  ├─ Network Errors        ✅
  ├─ AxiosError            ✅
  ├─ Error Normalization   ✅
  └─ Custom ApiError       🟡 مفهوم فقط
```

وبعد كده نعمل **مراجعة Section 3 + أسئلة Interview + تمرين عملي**، ثم نرجع لـ **Section 0 Architecture Foundation** قبل Interceptors، زي ما اتفقنا.

**الخطوة التالية:** نعمل الآن **Section 3 Final Review** بدون دخول أعمق في `class`.
</details>
الصراحه درس ال class ده صعب بنسبلي هل مهم ام لا
ChatGPT
**خد 5 دقائق فقط: ما توقفش الكورس بسبب `class`، لأنها مهمة لكن مش لازم تتعمق فيها الآن.**

أيوه، **`Custom ApiError` مهم في الشغل الحقيقي**، لكن بالنسبة لمسارك الحالي في Axios، مش محتاج تتقن الـ `class` نفسها بعمق دلوقتي.

### إيه المطلوب منك الآن؟

يكفي تفهم الفكرة:

```text
AxiosError
   ↓
Normalize
   ↓
ApiError
   ↓
Component
```

وأن:

```ts
class ApiError extends Error
```

معناها:

> أنا عملت نوع Error خاص بالتطبيق، مبني على الـ `Error` الأصلي، وأضفت له معلومات أحتاجها مثل `status` و`code`.

مش مطلوب منك حاليًا تحفظ:

```ts
constructor()
super()
extends
this
```

بالتفصيل.

---

### ليه ما أنصحكش نغوص في `class` الآن؟

لأن هدفنا الأساسي هو **Axios Architecture**، وليس دراسة JavaScript OOP من البداية.

ولو دخلنا دلوقتي في:

```text
class
constructor
extends
super
inheritance
prototype
this
static
private fields
```

هنخرج من مسار Axios ونعمل تشتيت.

وأنت أصلًا درست **OOP في C++**، فالمفاهيم دي مش غريبة عليك، لكن JavaScript لها تفاصيل مختلفة، وهنقدر نرجع لها في وقت مناسب.

### إذن نعمل إيه؟

نعتبر:

```text
Custom ApiError
```

🟡 **مفهومه مهم، لكن تفاصيل `class` مؤجلة.**

ونكمل الكورس بدون ما نضيّع وقتنا في نقطة مش هي هدف المرحلة الحالية.

## 📍 Roadmap

```text
SECTION 3  Error Handling
  ├─ Fundamentals          ✅
  ├─ HTTP Errors           ✅
  ├─ Network Errors        ✅
  ├─ AxiosError            ✅
  ├─ Error Normalization   ✅
  └─ Custom ApiError       🟡 مفهوم فقط
```

وبعد كده نعمل **مراجعة Section 3 + أسئلة Interview + تمرين عملي**، ثم نرجع لـ **Section 0 Architecture Foundation** قبل Interceptors، زي ما اتفقنا.

**الخطوة التالية:** نعمل الآن **Section 3 Final Review** بدون دخول أعمق في `class`.
<details>
<summary>تمام</summary>
user
تمام
ChatGPT
**خد 20 دقيقة: اعمل مراجعة `Section 3` كلها، وبعدها هنحدد هل أنت جاهز ننتقل لـ `Section 0`.**

## 📍 Roadmap

```text id="f4m8xq"
SECTION 0  Architecture Foundation       ⬜ ← التالي
SECTION 1  Axios Fundamentals             ✅
SECTION 2  TypeScript + Axios             ✅
SECTION 3  Error Handling                 🟢 FINAL REVIEW
  ├─ Fundamentals                         ✅
  ├─ HTTP Errors                          ✅
  ├─ Network Errors                       ✅
  ├─ AxiosError                           ✅
  ├─ Error Normalization                  ✅
  └─ Custom ApiError                      🟡 مفهوم فقط
SECTION 4  Interceptors                   ⬜
...
```
</details>
تمام
ChatGPT
**خد 20 دقيقة: اعمل مراجعة `Section 3` كلها، وبعدها هنحدد هل أنت جاهز ننتقل لـ `Section 0`.**

## 📍 Roadmap

```text id="f4m8xq"
SECTION 0  Architecture Foundation       ⬜ ← التالي
SECTION 1  Axios Fundamentals             ✅
SECTION 2  TypeScript + Axios             ✅
SECTION 3  Error Handling                 🟢 FINAL REVIEW
  ├─ Fundamentals                         ✅
  ├─ HTTP Errors                          ✅
  ├─ Network Errors                       ✅
  ├─ AxiosError                           ✅
  ├─ Error Normalization                  ✅
  └─ Custom ApiError                      🟡 مفهوم فقط
SECTION 4  Interceptors                   ⬜
...
```