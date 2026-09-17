# 1. يعني إيه Error Handling؟

لما تعمل:

```ts
const user = await getUser(10);
```

أنت بتفترض إن العملية هتنجح:

```text
Component
   ↓
getUser(10)
   ↓
API
   ↓
User
```

لكن في الواقع ممكن يحصل:

```text
getUser(10)
     ↓
    ❌
```

مثلاً:

- السيرفر رجع `404`
- السيرفر رجع `500`
- مفيش Internet
- الـ request أخذ وقت طويل جدًا
- الـ request اتعمله `abort`
- حصل خطأ غير متوقع

إذن لازم يكون عندنا طريقة نتعامل بها مع الفشل.

وده هو **Error Handling**.

---

# 2. أول Concept: Promise ممكن تنجح أو تفشل

بما إن:

```ts
getUser(10)
```

ترجع:

```ts
Promise<IUser>
```

فالـ Promise لها حالتين مهمتين بالنسبة لنا:

```text
Success
   ↓
IUser
```

أو:

```text
Failure
   ↓
Error
```

مثلاً:

```ts
const user = await getUser(10);
```

لو نجحت:

```text
user → IUser
```

لو فشلت:

```text
❌ Error
```

وهنا تظهر أهمية `try/catch`.

---

# 3. `try/catch`

الشكل الأساسي:

```ts
try {
  // code that might fail
} catch (error) {
  // handle the error
}
```

مثال:

```ts
try {
  const user = await getUser(10);

  console.log(user.name);
} catch (error) {
  console.log(error);
}
```

معناه:

```text
try
 ↓
نفّذ العملية
 ↓
نجحت؟
 ├── نعم → كمل
 │
 └── لا → catch
```

---

# 4. حاجة مهمة جدًا

`catch` **مش معناها إن الخطأ اتصلح**.

هي معناها:

> "لو حصل Error، أنا هتحكم في إيه اللي يحصل بعده."

مثلاً:

```ts
try {
  const user = await getUser(10);
} catch (error) {
  console.log("Failed to get user");
}
```

أنت هنا لم تصلح الـ API.

أنت فقط **handled the failure**.

ممكن تعرض للمستخدم:

```text
Failed to load user
```

أو تعمل logging:

```ts
console.error(error);
```

أو تعمل retry في تصميم أكبر لاحقًا.

---

# 5. طيب ليه Axios مهم هنا؟

لأن `getUser` بتستخدم Axios:

```ts
const response = await axios.get(...);
```

ولو Axios واجه مشكلة، ممكن الـ Promise تترفض:

```text
axios.get()
     ↓
❌ rejected Promise
     ↓
await
     ↓
throws Error
     ↓
catch
```

مثال:

```ts
try {
  const user = await getUser(10);
} catch (error) {
  console.log(error);
}
```

الـ `error` هنا هو الشيء اللي هنبدأ ندرسه بالتفصيل في الدرس القادم.

وهنا تحديدًا هنبدأ ندخل على:

```text
AxiosError
```

لكن **لسه مش هنستخدمه** قبل ما نفهم أنواع الفشل.

---

# 🧠 أول فرق لازم يثبت

لو كتبت:

```ts
const user = await getUser(10);
```

في حالة النجاح:

```text
user = IUser
```

في حالة الفشل:

```text
throw Error
```

ولذلك:

```ts
try {
  const user = await getUser(10);
} catch (error) {
  // هنا وصلنا للفشل
}
```

---

## 🎯 Interview Question

بعد ما اتعلمنا `try/catch`:

**إيه الفرق بين `try` و`catch` في الكود ده؟**

```ts
try {
  const user = await getUser(10);
} catch (error) {
  console.log(error);
}
```

جاوب بطريقتك، حتى لو بالعربي البسيط.

**الخطوة التالية:** اشرح وظيفة `try` ووظيفة `catch` من فهمك.
<details>
<summary>هنا بقوله لو الكود نجح هتشغل retry وهترجعلي الـ gate user اللي هو الـ ID بتاعه عشرة. أما في حالة لو حصل error وحصل مشكلة في الشبكة أو حصل إن الـ response ما تمشيش أو أي سبب من الأسباب، في الحالة دي هعمل حاجة اسمها caching همسك الـ error وأنا ههندل الـ error أطلع رسالة خطأ أو أعمل retry أو أعمل أي شيء. ولكن أنا هنا مش بعالج... مش بحل المشكلة، ولكن أنا بهندل الـ error.</summary>
user
هنا بقوله لو الكود نجح هتشغل retry وهترجعلي الـ gate user اللي هو الـ ID بتاعه عشرة. أما في حالة لو حصل error وحصل مشكلة في الشبكة أو حصل إن الـ response ما تمشيش أو أي سبب من الأسباب، في الحالة دي هعمل حاجة اسمها caching همسك الـ error وأنا ههندل الـ error أطلع رسالة خطأ أو أعمل retry أو أعمل أي شيء. ولكن أنا هنا مش بعالج... مش بحل المشكلة، ولكن أنا بهندل الـ error.
ChatGPT
**خد دقيقتين: إجابتك 9.5/10، والمفهوم الأساسي عندك صحيح، لكن عندك كلمة واحدة محتاجة تصحيح.**

أنت فهمت النقطة الأهم:

> **Error Handling لا يعني إصلاح المشكلة، بل يعني تحديد ماذا سنفعل عندما يحدث الخطأ.** ✅

### التصحيح الوحيد

أنت قلت:

> "لو الكود نجح هتشغل retry"

لا، `try` لا يشغل `retry`.

`try` معناها ببساطة:

```ts id="q5v2hx"
try {
  const user = await getUser(10);
}
```

**حاول تنفيذ الكود الموجود هنا.**

لو نجح:

```text id="v3b8q1"
try
 ↓
getUser(10)
 ↓
IUser
 ↓
كمل
```

لو فشل:

```text id="e7n2kp"
try
 ↓
getUser(10)
 ↓
❌ Error
 ↓
catch
```

وفي `catch` نقرر نعمل إيه:

```ts id="r8w1zc"
catch (error) {
  // show error message
  // log error
  // retry
  // redirect
  // etc.
}
```

---

### وبالمناسبة، دي نقطة مهمة جدًا

أنت ذكرت **caching** كطريقة للتعامل مع الخطأ.

هنا لازم نفصل بينهم:

```text
Error Handling
→ التعامل مع الفشل

Caching
→ تخزين البيانات لتقليل requests وتحسين الأداء
```

الـ caching **مش هو الـ Error Handling** في حد ذاته.

ممكن في نظام معين تستخدم caching كجزء من استراتيجية أكبر، لكن مش كل Error Handling = caching.

---

## ⭐ النموذج الذهني

```text id="xq5r8k"
try
│
├── Success
│     ↓
│    IUser
│
└── Failure
      ↓
    catch
      ↓
   Handle Error
```

وده أساس كل اللي جاي بعد كده.

لأننا بعد شوية هنفرق بين:

```text
HTTP Error
Network Error
Timeout Error
Abort
AxiosError
```

وهنعرف **إزاي Axios بيمثل كل حالة**.

**الخطوة التالية:** ننتقل إلى **HTTP Errors** ونفهم بالضبط ماذا يحدث عندما يرجع السيرفر `404` أو `401` أو `500`.
</details>
هنا بقوله لو الكود نجح هتشغل retry وهترجعلي الـ gate user اللي هو الـ ID بتاعه عشرة. أما في حالة لو حصل error وحصل مشكلة في الشبكة أو حصل إن الـ response ما تمشيش أو أي سبب من الأسباب، في الحالة دي هعمل حاجة اسمها caching همسك الـ error وأنا ههندل الـ error أطلع رسالة خطأ أو أعمل retry أو أعمل أي شيء. ولكن أنا هنا مش بعالج... مش بحل المشكلة، ولكن أنا بهندل الـ error.
ChatGPT
**خد دقيقتين: إجابتك 9.5/10، والمفهوم الأساسي عندك صحيح، لكن عندك كلمة واحدة محتاجة تصحيح.**

أنت فهمت النقطة الأهم:

> **Error Handling لا يعني إصلاح المشكلة، بل يعني تحديد ماذا سنفعل عندما يحدث الخطأ.** ✅

### التصحيح الوحيد

أنت قلت:

> "لو الكود نجح هتشغل retry"

لا، `try` لا يشغل `retry`.

`try` معناها ببساطة:

```ts id="q5v2hx"
try {
  const user = await getUser(10);
}
```

**حاول تنفيذ الكود الموجود هنا.**

لو نجح:

```text id="v3b8q1"
try
 ↓
getUser(10)
 ↓
IUser
 ↓
كمل
```

لو فشل:

```text id="e7n2kp"
try
 ↓
getUser(10)
 ↓
❌ Error
 ↓
catch
```

وفي `catch` نقرر نعمل إيه:

```ts id="r8w1zc"
catch (error) {
  // show error message
  // log error
  // retry
  // redirect
  // etc.
}
```

---

### وبالمناسبة، دي نقطة مهمة جدًا

أنت ذكرت **caching** كطريقة للتعامل مع الخطأ.

هنا لازم نفصل بينهم:

```text
Error Handling
→ التعامل مع الفشل

Caching
→ تخزين البيانات لتقليل requests وتحسين الأداء
```

الـ caching **مش هو الـ Error Handling** في حد ذاته.

ممكن في نظام معين تستخدم caching كجزء من استراتيجية أكبر، لكن مش كل Error Handling = caching.

---

## ⭐ النموذج الذهني

```text id="xq5r8k"
try
│
├── Success
│     ↓
│    IUser
│
└── Failure
      ↓
    catch
      ↓
   Handle Error
```

وده أساس كل اللي جاي بعد كده.

لأننا بعد شوية هنفرق بين:

```text
HTTP Error
Network Error
Timeout Error
Abort
AxiosError
```

وهنعرف **إزاي Axios بيمثل كل حالة**.

**الخطوة التالية:** ننتقل إلى **HTTP Errors** ونفهم بالضبط ماذا يحدث عندما يرجع السيرفر `404` أو `401` أو `500`.
<details>
<summary>**HTTP Errors**</summary>
user
**HTTP Errors**  
ChatGPT
**خد 10 دقائق: افهم قاعدة واحدة أولًا، وهي أن `HTTP Error` يعني أن السيرفر رد، لكن الـ HTTP status code بيقول إن العملية فشلت.**

## 📍 Roadmap

```text
SECTION 0  Architecture Foundation       ⬜
SECTION 1  Axios Fundamentals             ✅
SECTION 2  TypeScript + Axios             ✅
SECTION 3  Error Handling                 🟢 CURRENT
  ├─ Error Handling Fundamentals          ✅
  ├─ HTTP Errors                          🟡 ← نحن هنا
  ├─ Network Errors                       ⬜
  ├─ AxiosError                           ⬜
  ├─ Error Normalization                  ⬜
  └─ Custom ApiError                      ⬜
SECTION 4  Interceptors                   ⬜
...
```
</details>
**HTTP Errors**  
ChatGPT
**خد 10 دقائق: افهم قاعدة واحدة أولًا، وهي أن `HTTP Error` يعني أن السيرفر رد، لكن الـ HTTP status code بيقول إن العملية فشلت.**

## 📍 Roadmap

```text
SECTION 0  Architecture Foundation       ⬜
SECTION 1  Axios Fundamentals             ✅
SECTION 2  TypeScript + Axios             ✅
SECTION 3  Error Handling                 🟢 CURRENT
  ├─ Error Handling Fundamentals          ✅
  ├─ HTTP Errors                          🟡 ← نحن هنا
  ├─ Network Errors                       ⬜
  ├─ AxiosError                           ⬜
  ├─ Error Normalization                  ⬜
  └─ Custom ApiError                      ⬜
SECTION 4  Interceptors                   ⬜
...
```