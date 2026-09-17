## 1. يعني إيه Response Interceptor؟

إحنا عندنا:

```text
Component
   ↓
API function
   ↓
api.get()
   ↓
Request Interceptor
   ↓
Backend
   ↓
Response Interceptor
   ↓
API function
   ↓
Component
```

إذن:

**Request Interceptor** بيتدخل وإحنا رايحين للـ Backend.

**Response Interceptor** بيتدخل وإحنا راجعين من الـ Backend.

---

## 2. مثال بسيط

عندنا:

```ts
const response = await api.get("/users");
```

الـ request يروح للـ Backend.

الـ Backend يرجع:

```json
{
  "status": true,
  "code": 200,
  "payload": [
    {
      "id": 1,
      "name": "Ahmed"
    }
  ]
}
```

قبل ما الـ `response` يوصل للكود اللي كتب:

```ts
const response = await api.get("/users");
```

ممكن **Response Interceptor** يتدخل.

---

## 3. شكله إزاي؟

```ts
api.interceptors.response.use(
  (response) => {
    return response;
  },
  (error) => {
    return Promise.reject(error);
  }
);
```

عندنا هنا **وظيفتين**:

```text
(response) => { ... }
```

دي للـ **Success Response**.

و:

```text
(error) => { ... }
```

دي للـ **Error Response**.

---

## 4. الـ Success Handler

لو الـ Backend رجع:

```text
200 OK
```

الـ Axios يدخل هنا:

```ts
(response) => {
  return response;
}
```

يعني:

```text
Backend
   ↓
200 OK
   ↓
Response Interceptor
   ↓
return response
   ↓
api.get()
```

وأهم حاجة حاليًا:

```ts
return response;
```

معناها:

> أنا استلمت الـ Response، معنديش تعديل عليه، كمل طريقك للـ caller.

---

## 5. طب الـ Error Handler؟

لو الـ Backend رجع:

```text
404
```

أو:

```text
500
```

Axios يعتبرها rejected Promise، وبالتالي تدخل هنا:

```ts
(error) => {
  return Promise.reject(error);
}
```

يعني:

```text
Backend
   ↓
404
   ↓
Response Interceptor
   ↓
error handler
   ↓
Promise rejected
   ↓
catch
```

وده مهم جدًا.

الـ Response Interceptor مش معناه "Success فقط".

هو يقدر يتعامل مع:

```text
Success Response
        +
Error Response
```

---

## 6. أهم فرق تحفظه

### Request Interceptor

قبل الـ request:

```text
App
 ↓
Request Interceptor
 ↓
Backend
```

مثلاً:

```ts
config.headers.Authorization = `Bearer ${token}`;
```

### Response Interceptor

بعد الـ response:

```text
Backend
 ↓
Response Interceptor
 ↓
App
```

مثلاً ممكن تستخدمه لاحقًا في:

```text
401 handling
error normalization
logging
token refresh
```

لكن **لسه مش هندخل في دول**.

---

### سؤال صغير قبل ما نكمل

لو عندنا:

```ts
const response = await api.get("/users");
```

والـ Backend رجع:

```text
200 OK
```

مين بيتنفذ الأول؟

**A)** Component  
**B)** Request Interceptor  
**C)** Response Interceptor  
**D)** `response.data`

اختار حرف واحد فقط.
<details>
<summary>A</summary>
user
A
ChatGPT
اختيارك **A غلط**. النتيجة هنا **C: Response Interceptor**.

### ليه؟

الـ flow الصحيح:

```text
Component
   ↓
api.get()
   ↓
Request Interceptor
   ↓
Backend
   ↓
Response Interceptor  ← هنا
   ↓
Component يستلم النتيجة
```

يعني الـ Component هو اللي **بدأ العملية**، لكنه مش أول حاجة تتنفذ بعد رجوع الـ `200`.

أما:

```ts
response.data
```

فده **مش مرحلة بتتنفذ**، ده property داخل الـ response بنقرأه بعد ما الـ response يوصل.

مثال:

```ts
const response = await api.get("/users");

console.log(response.data);
```

الـ `Response Interceptor` بيكون قبل وصول `response` للـ caller.

**تقييم السؤال: 0/10، لكن ده خطأ في ترتيب الـ flow فقط، ومش هنعتبر الـ concept نفسه ضعيف.**

**الخطوة التالية:** جاوب: لو الـ Backend رجّع `404`، هل الـ Response Interceptor يقدر يشوف الـ error قبل ما يوصل للـ `catch`؟ **نعم أم لا؟**
</details>
A
ChatGPT
اختيارك **A غلط**. النتيجة هنا **C: Response Interceptor**.

### ليه؟

الـ flow الصحيح:

```text
Component
   ↓
api.get()
   ↓
Request Interceptor
   ↓
Backend
   ↓
Response Interceptor  ← هنا
   ↓
Component يستلم النتيجة
```

يعني الـ Component هو اللي **بدأ العملية**، لكنه مش أول حاجة تتنفذ بعد رجوع الـ `200`.

أما:

```ts
response.data
```

فده **مش مرحلة بتتنفذ**، ده property داخل الـ response بنقرأه بعد ما الـ response يوصل.

مثال:

```ts
const response = await api.get("/users");

console.log(response.data);
```

الـ `Response Interceptor` بيكون قبل وصول `response` للـ caller.

**تقييم السؤال: 0/10، لكن ده خطأ في ترتيب الـ flow فقط، ومش هنعتبر الـ concept نفسه ضعيف.**

**الخطوة التالية:** جاوب: لو الـ Backend رجّع `404`، هل الـ Response Interceptor يقدر يشوف الـ error قبل ما يوصل للـ `catch`؟ **نعم أم لا؟**
<details>
<summary>كمل</summary>
user
كمل
ChatGPT
كمل معايا **10 دقائق** ونثبت الـ Response Interceptor عمليًا، وبعدها نعمل سؤال Interview قبل ما ننتقل لـ `4.4`.

### 🗺️ Roadmap

```text
SECTION 4: Interceptors 🟢
  ├─ 4.1 What is an Interceptor    ✅
  ├─ 4.2 Request Interceptor       ✅
  ├─ 4.3 Response Interceptor      🟢 ← هنا
  │    ├─ What is it?               ✅
  │    ├─ Response flow             ✅
  │    ├─ Success handler           🟢
  │    └─ Error handler             🟢
  ├─ 4.4 Authentication            ⬜
  ├─ 4.5 401 Handling              ⬜
  └─ 4.6 Token Refresh + Queue     ⬜
```

## 1. شكل الـ Response Interceptor

```ts id="5ajy9d"
api.interceptors.response.use(
  (response) => {
    return response;
  },
  (error) => {
    return Promise.reject(error);
  }
);
```

عندنا هنا **callbackين**.

### الأول: Success

```ts id="n9p3dk"
(response) => {
  return response;
}
```

بيتعمل لما Axios يستلم **Response ناجح**.

مثلاً:

```text id="9t7r4h"
200 OK
201 Created
204 No Content
```

---

### الثاني: Error

```ts id="8w3k7c"
(error) => {
  return Promise.reject(error);
}
```

بيتعمل لما Axios يعتبر الـ response **error**.

مثلاً:

```text id="w9e1k3"
400
401
403
404
500
```

---

## 2. نقطة مهمة جدًا

افترض إن عندنا:

```ts id="9j5p7a"
const response = await api.get("/users");
```

والـ Backend رجع:

```text id="n3w7qa"
200 OK
```

الـ flow:

```text id="q2r5tv"
api.get()
   ↓
Request Interceptor
   ↓
Backend
   ↓
Response Interceptor
   ↓
response
```

لكن لو رجع:

```text id="f6k8ds"
404
```

الـ flow:

```text id="4m2q9x"
api.get()
   ↓
Request Interceptor
   ↓
Backend
   ↓
Response Interceptor
   ↓
Error Handler
   ↓
catch
```

يعني الـ Response Interceptor يقدر يتدخل **قبل الـ catch**.

---

## 3. ليه بنعمل `return Promise.reject(error)`؟

دي نقطة مهمة.

لما الـ error يوصل للـ interceptor:

```ts id="v8q2mw"
(error) => {
  return Promise.reject(error);
}
```

إحنا بنقول:

> أنا شوفت الـ error، ومش هبلعه. خليه يكمل كـ rejected Promise.

وبالتالي الـ caller يقدر يعمل:

```ts id="q4m7sp"
try {
  const response = await api.get("/users");
} catch (error) {
  // هنا نستقبل الـ error
}
```

لو عملت بدلها:

```ts id="g3x9az"
(error) => {
  return error;
}
```

أنت غيرت الـ flow، وممكن تخلي الـ caller يتعامل مع الـ error كأنه **resolved value**، وده غالبًا مش اللي عايزينه.

---

## 4. هل لازم نعمل حاجة في Success؟

لا.

ممكن ببساطة:

```ts id="q5z1nc"
(response) => response
```

وده معناه:

> استلمت الـ response، ومش محتاج أغيره.

لكن لاحقًا ممكن نعمل حاجات زي:

```text id="2g7w8p"
Response
   ↓
Interceptor
   ↓
تحقق / تعديل / Logging
   ↓
Application
```

ومن ضمن الاستخدامات المهمة جدًا لاحقًا:

```text id="2k8r4v"
401
 ↓
Token expired
 ↓
Refresh Token
```

لكن **لسه مش هنشرحها**.

---

## 5. الفرق النهائي

احفظ الصورة دي:

```text id="q1h7mx"
             REQUEST
                ↓
Component → Axios
                ↓
       Request Interceptor
                ↓
             Backend
                ↓
       Response Interceptor
                ↓
             Component
```

**Request Interceptor:**

```text
قبل ما الطلب يروح
```

**Response Interceptor:**

```text
بعد ما الرد يرجع
```

### 🎯 Interview Question

عندك:

```ts id="4q6v8x"
api.interceptors.response.use(
  (response) => {
    return response;
  },
  (error) => {
    return Promise.reject(error);
  }
);
```

لو الـ Backend رجّع `404`، **أي callback هيتنفذ؟ وليه؟**

جاوب بالمفهوم، ومش مهم الـ syntax.
</details>
كمل
ChatGPT
كمل معايا **10 دقائق** ونثبت الـ Response Interceptor عمليًا، وبعدها نعمل سؤال Interview قبل ما ننتقل لـ `4.4`.

### 🗺️ Roadmap

```text
SECTION 4: Interceptors 🟢
  ├─ 4.1 What is an Interceptor    ✅
  ├─ 4.2 Request Interceptor       ✅
  ├─ 4.3 Response Interceptor      🟢 ← هنا
  │    ├─ What is it?               ✅
  │    ├─ Response flow             ✅
  │    ├─ Success handler           🟢
  │    └─ Error handler             🟢
  ├─ 4.4 Authentication            ⬜
  ├─ 4.5 401 Handling              ⬜
  └─ 4.6 Token Refresh + Queue     ⬜
```

## 1. شكل الـ Response Interceptor

```ts id="5ajy9d"
api.interceptors.response.use(
  (response) => {
    return response;
  },
  (error) => {
    return Promise.reject(error);
  }
);
```

عندنا هنا **callbackين**.

### الأول: Success

```ts id="n9p3dk"
(response) => {
  return response;
}
```

بيتعمل لما Axios يستلم **Response ناجح**.

مثلاً:

```text id="9t7r4h"
200 OK
201 Created
204 No Content
```

---

### الثاني: Error

```ts id="8w3k7c"
(error) => {
  return Promise.reject(error);
}
```

بيتعمل لما Axios يعتبر الـ response **error**.

مثلاً:

```text id="w9e1k3"
400
401
403
404
500
```

---

## 2. نقطة مهمة جدًا

افترض إن عندنا:

```ts id="9j5p7a"
const response = await api.get("/users");
```

والـ Backend رجع:

```text id="n3w7qa"
200 OK
```

الـ flow:

```text id="q2r5tv"
api.get()
   ↓
Request Interceptor
   ↓
Backend
   ↓
Response Interceptor
   ↓
response
```

لكن لو رجع:

```text id="f6k8ds"
404
```

الـ flow:

```text id="4m2q9x"
api.get()
   ↓
Request Interceptor
   ↓
Backend
   ↓
Response Interceptor
   ↓
Error Handler
   ↓
catch
```

يعني الـ Response Interceptor يقدر يتدخل **قبل الـ catch**.

---

## 3. ليه بنعمل `return Promise.reject(error)`؟

دي نقطة مهمة.

لما الـ error يوصل للـ interceptor:

```ts id="v8q2mw"
(error) => {
  return Promise.reject(error);
}
```

إحنا بنقول:

> أنا شوفت الـ error، ومش هبلعه. خليه يكمل كـ rejected Promise.

وبالتالي الـ caller يقدر يعمل:

```ts id="q4m7sp"
try {
  const response = await api.get("/users");
} catch (error) {
  // هنا نستقبل الـ error
}
```

لو عملت بدلها:

```ts id="g3x9az"
(error) => {
  return error;
}
```

أنت غيرت الـ flow، وممكن تخلي الـ caller يتعامل مع الـ error كأنه **resolved value**، وده غالبًا مش اللي عايزينه.

---

## 4. هل لازم نعمل حاجة في Success؟

لا.

ممكن ببساطة:

```ts id="q5z1nc"
(response) => response
```

وده معناه:

> استلمت الـ response، ومش محتاج أغيره.

لكن لاحقًا ممكن نعمل حاجات زي:

```text id="2g7w8p"
Response
   ↓
Interceptor
   ↓
تحقق / تعديل / Logging
   ↓
Application
```

ومن ضمن الاستخدامات المهمة جدًا لاحقًا:

```text id="2k8r4v"
401
 ↓
Token expired
 ↓
Refresh Token
```

لكن **لسه مش هنشرحها**.

---

## 5. الفرق النهائي

احفظ الصورة دي:

```text id="q1h7mx"
             REQUEST
                ↓
Component → Axios
                ↓
       Request Interceptor
                ↓
             Backend
                ↓
       Response Interceptor
                ↓
             Component
```

**Request Interceptor:**

```text
قبل ما الطلب يروح
```

**Response Interceptor:**

```text
بعد ما الرد يرجع
```

### 🎯 Interview Question

عندك:

```ts id="4q6v8x"
api.interceptors.response.use(
  (response) => {
    return response;
  },
  (error) => {
    return Promise.reject(error);
  }
);
```

لو الـ Backend رجّع `404`، **أي callback هيتنفذ؟ وليه؟**

جاوب بالمفهوم، ومش مهم الـ syntax.
<details>
<summary>دي مش فاهمها</summary>
user
دي مش فاهمها 
. ليه بنعمل return Promise.reject(error)؟
دي نقطة مهمة.
لما الـ error يوصل للـ interceptor:
(error) => {
  return Promise.reject(error);
}
إحنا بنقول:
أنا شوفت الـ error، ومش هبلعه. خليه يكمل كـ rejected Promise.
وبالتالي الـ caller يقدر يعمل:
try {
  const response = await api.get("/users");
} catch (error) {
  // هنا نستقبل الـ error
}
لو عملت بدلها:
(error) => {
  return error;
}
أنت غيرت الـ flow، وممكن تخلي الـ caller يتعامل مع الـ error كأنه resolved value، وده غالبًا مش اللي عايزينه.
ChatGPT
افهم النقطة دي من خلال **مثال واحد فقط في 5 دقائق**، لأنها فعلًا مهمة في فهم الـ Promise والـ Interceptor.

### الفكرة من البداية

عندنا:

```ts
try {
  const response = await api.get("/users");
} catch (error) {
  console.log(error);
}
```

لو الـ API رجع `404`، Axios عنده **Rejected Promise**.

يعني كأن:

```text
api.get()
   ↓
Promise
   ↓
❌ rejected
```

والـ `await` لما يقابل Promise مرفوض، ينقل التنفيذ إلى `catch`:

```text
await
 ↓
❌ rejected
 ↓
catch
```

---

## طيب دخل الـ Response Interceptor فين؟

الـ Interceptor واقف **في النص**:

```text
Backend
   ↓
404
   ↓
Response Interceptor
   ↓
catch
```

لما نكتب:

```ts
(error) => {
  return Promise.reject(error);
}
```

إحنا بنقول:

> "أنا استلمت الـ error، لكن مش هاعتبر إن المشكلة اتحلت. هرجعه تاني كـ rejected Promise."

فيكمل:

```text
404
 ↓
Interceptor
 ↓
Promise.reject(error)
 ↓
catch
```

وبالتالي:

```ts
try {
  await api.get("/users");
} catch (error) {
  console.log(error);
}
```

الـ `catch` يشتغل عادي.

---

# طب إيه المشكلة لو عملنا `return error`؟

هنا النقطة المهمة.

لما تكتب:

```ts
(error) => {
  return error;
}
```

أنت بتقول للـ Promise chain:

> "أنا تعاملت مع الـ error، وها هو return عادي."

يعني بدل ما تفضل العملية:

```text
❌ Rejected
```

ممكن تتحول إلى:

```text
✅ Resolved
```

والكود اللي بعد `await` ممكن يكمل بشكل غير متوقع.

تخيل:

```ts
const response = await api.get("/users");

console.log("نجح الطلب");
```

أنت **مش عايز** إن `404` يدخل الـ error interceptor وبعدها يرجع كأنه نجاح.

أنت عايز:

```text
404
 ↓
Interceptor
 ↓
❌ rejected
 ↓
catch
```

---

## احفظها بالطريقة دي

```ts
return response;
```

معناها:

> الطلب نجح، كمل.

```ts
return Promise.reject(error);
```

معناها:

> الطلب فشل، كمل كـ Error.

يعني:

```text
SUCCESS
   ↓
return response
   ↓
caller يستلم response


ERROR
   ↓
return Promise.reject(error)
   ↓
caller يدخل catch
```

### نقطة مهمة جدًا

`Promise.reject(error)` **مش بيصلح الـ error**.

ومش معناه إننا عملنا retry.

هو فقط بيحافظ على حالة الـ Promise كـ **rejected** عشان الـ error يكمل في الـ chain ويوصل للـ `catch`.

**الخطوة التالية:** اكتبلي بجملة واحدة الفرق بين `return response` و `return Promise.reject(error)`.
</details>
دي مش فاهمها 
. ليه بنعمل return Promise.reject(error)؟
دي نقطة مهمة.
لما الـ error يوصل للـ interceptor:
(error) => {
  return Promise.reject(error);
}
إحنا بنقول:
أنا شوفت الـ error، ومش هبلعه. خليه يكمل كـ rejected Promise.
وبالتالي الـ caller يقدر يعمل:
try {
  const response = await api.get("/users");
} catch (error) {
  // هنا نستقبل الـ error
}
لو عملت بدلها:
(error) => {
  return error;
}
أنت غيرت الـ flow، وممكن تخلي الـ caller يتعامل مع الـ error كأنه resolved value، وده غالبًا مش اللي عايزينه.
ChatGPT
افهم النقطة دي من خلال **مثال واحد فقط في 5 دقائق**، لأنها فعلًا مهمة في فهم الـ Promise والـ Interceptor.

### الفكرة من البداية

عندنا:

```ts
try {
  const response = await api.get("/users");
} catch (error) {
  console.log(error);
}
```

لو الـ API رجع `404`، Axios عنده **Rejected Promise**.

يعني كأن:

```text
api.get()
   ↓
Promise
   ↓
❌ rejected
```

والـ `await` لما يقابل Promise مرفوض، ينقل التنفيذ إلى `catch`:

```text
await
 ↓
❌ rejected
 ↓
catch
```

---

## طيب دخل الـ Response Interceptor فين؟

الـ Interceptor واقف **في النص**:

```text
Backend
   ↓
404
   ↓
Response Interceptor
   ↓
catch
```

لما نكتب:

```ts
(error) => {
  return Promise.reject(error);
}
```

إحنا بنقول:

> "أنا استلمت الـ error، لكن مش هاعتبر إن المشكلة اتحلت. هرجعه تاني كـ rejected Promise."

فيكمل:

```text
404
 ↓
Interceptor
 ↓
Promise.reject(error)
 ↓
catch
```

وبالتالي:

```ts
try {
  await api.get("/users");
} catch (error) {
  console.log(error);
}
```

الـ `catch` يشتغل عادي.

---

# طب إيه المشكلة لو عملنا `return error`؟

هنا النقطة المهمة.

لما تكتب:

```ts
(error) => {
  return error;
}
```

أنت بتقول للـ Promise chain:

> "أنا تعاملت مع الـ error، وها هو return عادي."

يعني بدل ما تفضل العملية:

```text
❌ Rejected
```

ممكن تتحول إلى:

```text
✅ Resolved
```

والكود اللي بعد `await` ممكن يكمل بشكل غير متوقع.

تخيل:

```ts
const response = await api.get("/users");

console.log("نجح الطلب");
```

أنت **مش عايز** إن `404` يدخل الـ error interceptor وبعدها يرجع كأنه نجاح.

أنت عايز:

```text
404
 ↓
Interceptor
 ↓
❌ rejected
 ↓
catch
```

---

## احفظها بالطريقة دي

```ts
return response;
```

معناها:

> الطلب نجح، كمل.

```ts
return Promise.reject(error);
```

معناها:

> الطلب فشل، كمل كـ Error.

يعني:

```text
SUCCESS
   ↓
return response
   ↓
caller يستلم response


ERROR
   ↓
return Promise.reject(error)
   ↓
caller يدخل catch
```

### نقطة مهمة جدًا

`Promise.reject(error)` **مش بيصلح الـ error**.

ومش معناه إننا عملنا retry.

هو فقط بيحافظ على حالة الـ Promise كـ **rejected** عشان الـ error يكمل في الـ chain ويوصل للـ `catch`.

**الخطوة التالية:** اكتبلي بجملة واحدة الفرق بين `return response` و `return Promise.reject(error)`.
<details>
<summary>يبقى أنا فهمت دلوقتي إن إحنا لما بننوي نروح للـ interceptor أو الـ response interceptor، عندنا حالتين. الحالة الأولى في حالة الـ response، لما يكون success، في الحالة دي برجع response عادي، أو لو عايز أهندل أي حاجة فيها بهندلها، مفيش عندي أي مشكلة في ناحية الـ success. أما في ناحية الـ error إن أنا ما ينفعش أعمل return error. لو عملت return error هتبقى هنا resolved، وساعتها هيبقى هو كأنه تعامل مع الـ error وهيرجعلك error كأنه هو هندله، وإحنا مش عايزين كده. إحنا عايزين نقول له: لا، ارجع للكاتش، لأن الكاتش أنا بهندل فيه error بالفعل. فعشان كده بقول له Promise، بقول له rejected Promise، ليه؟ عشان await لما بتيجي معاها rejected Promise، في الحالة دي هو لما بييجي معايا rejected Promise، آه، لما الـ await لما يقابل Promise مرفوض بينقل التنفيذ إلى الـ catch.</summary>
user
يبقى أنا فهمت دلوقتي إن إحنا لما بننوي نروح للـ interceptor أو الـ response interceptor، عندنا حالتين. الحالة الأولى في حالة الـ response، لما يكون success، في الحالة دي برجع response عادي، أو لو عايز أهندل أي حاجة فيها بهندلها، مفيش عندي أي مشكلة في ناحية الـ success. أما في ناحية الـ error إن أنا ما ينفعش أعمل return error. لو عملت return error هتبقى هنا resolved، وساعتها هيبقى هو كأنه تعامل مع الـ error وهيرجعلك error كأنه هو هندله، وإحنا مش عايزين كده. إحنا عايزين نقول له: لا، ارجع للكاتش، لأن الكاتش أنا بهندل فيه error بالفعل. فعشان كده بقول له Promise، بقول له rejected Promise، ليه؟ عشان await لما بتيجي معاها rejected Promise، في الحالة دي هو لما بييجي معايا rejected Promise، آه، لما الـ await لما يقابل Promise مرفوض بينقل التنفيذ إلى الـ catch.
ChatGPT
أيوه، **أنت فهمت الفكرة الأساسية صح جدًا**. تقييم فهمك هنا **10/10**.

بس عندي تصحيح صغير في التعبير عشان يبقى كلامك مضبوط جدًا في الـ Interview:

### الـ Success

```ts
(response) => {
  return response;
}
```

معناها:

> الـ Response وصل بنجاح، ولو مش محتاج أعمل عليه processing، أرجعه كما هو.

ولو محتاج أعمل processing:

```ts
(response) => {
  // handle something
  return response;
}
```

---

### الـ Error

هنا:

```ts
(error) => {
  return Promise.reject(error);
}
```

أنت **مش بتقول للـ `await` مباشرةً "روح للـ catch"**.

الأدق إنك بتقول:

> "خلّي الـ Promise يفضل في حالة `rejected`."

وبعد كده لما الـ `await` يستقبل الـ rejected Promise، يحصل:

```text id="b7q2km"
Response Error
      ↓
Response Interceptor
      ↓
Promise.reject(error)
      ↓
Rejected Promise
      ↓
await
      ↓
catch
```

وده بالضبط اللي أنت وصفته.

### وأهم نقطة

لو كتبت:

```ts id="p5q8vd"
return error;
```

فالـ Promise chain تعتبر إن الـ callback **رجّع قيمة بشكل طبيعي**، وبالتالي الـ chain ممكن تكمل كـ resolved.

وده خطر لأنك ممكن تخلي كود الـ caller يتعامل مع error كأنه نتيجة ناجحة.

---

### احفظ القاعدة دي

```text id="x5c8jp"
Success
   ↓
return response
   ↓
Resolved


Error
   ↓
return Promise.reject(error)
   ↓
Rejected
   ↓
catch
```

والأجمل إنك دلوقتي فهمت **ليه** بنكتب:

```ts id="1n0j4f"
return Promise.reject(error);
```

مش مجرد إنك حفظت السطر.

**الخطوة التالية:** نعمل تطبيق صغير جدًا على `Response Interceptor` ونشوف بعيننا الـ `200` والـ `404` بيمشوا في أنهي path.
</details>
يبقى أنا فهمت دلوقتي إن إحنا لما بننوي نروح للـ interceptor أو الـ response interceptor، عندنا حالتين. الحالة الأولى في حالة الـ response، لما يكون success، في الحالة دي برجع response عادي، أو لو عايز أهندل أي حاجة فيها بهندلها، مفيش عندي أي مشكلة في ناحية الـ success. أما في ناحية الـ error إن أنا ما ينفعش أعمل return error. لو عملت return error هتبقى هنا resolved، وساعتها هيبقى هو كأنه تعامل مع الـ error وهيرجعلك error كأنه هو هندله، وإحنا مش عايزين كده. إحنا عايزين نقول له: لا، ارجع للكاتش، لأن الكاتش أنا بهندل فيه error بالفعل. فعشان كده بقول له Promise، بقول له rejected Promise، ليه؟ عشان await لما بتيجي معاها rejected Promise، في الحالة دي هو لما بييجي معايا rejected Promise، آه، لما الـ await لما يقابل Promise مرفوض بينقل التنفيذ إلى الـ catch.
ChatGPT
أيوه، **أنت فهمت الفكرة الأساسية صح جدًا**. تقييم فهمك هنا **10/10**.

بس عندي تصحيح صغير في التعبير عشان يبقى كلامك مضبوط جدًا في الـ Interview:

### الـ Success

```ts
(response) => {
  return response;
}
```

معناها:

> الـ Response وصل بنجاح، ولو مش محتاج أعمل عليه processing، أرجعه كما هو.

ولو محتاج أعمل processing:

```ts
(response) => {
  // handle something
  return response;
}
```

---

### الـ Error

هنا:

```ts
(error) => {
  return Promise.reject(error);
}
```

أنت **مش بتقول للـ `await` مباشرةً "روح للـ catch"**.

الأدق إنك بتقول:

> "خلّي الـ Promise يفضل في حالة `rejected`."

وبعد كده لما الـ `await` يستقبل الـ rejected Promise، يحصل:

```text id="b7q2km"
Response Error
      ↓
Response Interceptor
      ↓
Promise.reject(error)
      ↓
Rejected Promise
      ↓
await
      ↓
catch
```

وده بالضبط اللي أنت وصفته.

### وأهم نقطة

لو كتبت:

```ts id="p5q8vd"
return error;
```

فالـ Promise chain تعتبر إن الـ callback **رجّع قيمة بشكل طبيعي**، وبالتالي الـ chain ممكن تكمل كـ resolved.

وده خطر لأنك ممكن تخلي كود الـ caller يتعامل مع error كأنه نتيجة ناجحة.

---

### احفظ القاعدة دي

```text id="x5c8jp"
Success
   ↓
return response
   ↓
Resolved


Error
   ↓
return Promise.reject(error)
   ↓
Rejected
   ↓
catch
```

والأجمل إنك دلوقتي فهمت **ليه** بنكتب:

```ts id="1n0j4f"
return Promise.reject(error);
```

مش مجرد إنك حفظت السطر.

**الخطوة التالية:** نعمل تطبيق صغير جدًا على `Response Interceptor` ونشوف بعيننا الـ `200` والـ `404` بيمشوا في أنهي path.
<details>
<summary>ايه المقصود بي</summary>
user
ايه المقصود بي 
caller بختصار
ChatGPT
**Caller** يعني ببساطة: **الكود أو الدالة اللي استدعت الدالة الحالية**.

مثال:

```ts
const getUser = async () => {
  // ...
};

const user = await getUser();
```

هنا:

- `getUser` = **Called function**
- الكود اللي كتب `getUser()` = **Caller**

وفي مثالنا:

```ts
const response = await api.get("/users");
```

الكود ده هو **Caller** بالنسبة للـ `api.get()`.

يعني باختصار جدًا:

> **Caller = اللي بينادي على الدالة.**

**الخطوة التالية:** نكمل من `Response Interceptor` ونطبق مثال `200` و`404` عمليًا.
</details>
ايه المقصود بي 
caller بختصار
ChatGPT
**Caller** يعني ببساطة: **الكود أو الدالة اللي استدعت الدالة الحالية**.

مثال:

```ts
const getUser = async () => {
  // ...
};

const user = await getUser();
```

هنا:

- `getUser` = **Called function**
- الكود اللي كتب `getUser()` = **Caller**

وفي مثالنا:

```ts
const response = await api.get("/users");
```

الكود ده هو **Caller** بالنسبة للـ `api.get()`.

يعني باختصار جدًا:

> **Caller = اللي بينادي على الدالة.**

**الخطوة التالية:** نكمل من `Response Interceptor` ونطبق مثال `200` و`404` عمليًا.
<details>
<summary>الخطوة التالية: نعمل تطبيق صغير جدًا على Response Interceptor ونشوف بعيننا الـ 200 والـ 404 بيمشوا في أنهي path</summary>
user
الخطوة التالية: نعمل تطبيق صغير جدًا على Response Interceptor ونشوف بعيننا الـ 200 والـ 404 بيمشوا في أنهي path
يلا بينا
ChatGPT
هنعمل تجربة صغيرة تشوف فيها بنفسك مسار `200` ومسار `404` داخل الـ Response Interceptor.

genui{"app_block":{"title":"Response Interceptor Flow","variant":"inline","icon":"study","language":"html","bundle_version":1,"content":"<div id=\"ri-flow\" class=\"w-full max-w-3xl mx-auto space-y-4\">\n  <div class=\"flex flex-wrap gap-2 text-sm\">\n    <button type=\"button\" data-status=\"200\" class=\"rounded-lg border px-4 py-3\">▶ Simulate 200 Success</button>\n    <button type=\"button\" data-status=\"404\" class=\"rounded-lg border px-4 py-3\">▶ Simulate 404 Error</button>\n  </div>\n  <div class=\"grid gap-3 sm:grid-cols-5 items-center text-center text-sm\">\n    <div data-step=\"request\" class=\"rounded-xl border p-3\"><strong>api.get()</strong><div class=\"text-xs opacity-70\">Caller</div></div>\n    <div class=\"text-xl\">↓</div>\n    <div data-step=\"backend\" class=\"rounded-xl border p-3\"><strong>Backend</strong><div class=\"text-xs opacity-70\">Response</div></div>\n    <div class=\"text-xl\">↓</div>\n    <div data-step=\"interceptor\" class=\"rounded-xl border p-3\"><strong>Response Interceptor</strong><div class=\"text-xs opacity-70\">Inspect</div></div>\n  </div>\n  <div data-result class=\"rounded-xl border p-4 text-sm\" aria-live=\"polite\">اضغط على أحد الزرين وشوف الـ path.</div>\n  <div class=\"rounded-xl border p-4 font-mono text-xs whitespace-pre-wrap\">api.interceptors.response.use(\\n  (response) => {\\n    return response;\\n  },\\n  (error) => {\\n    return Promise.reject(error);\\n  }\\n);</div>\n</div>\n<script>\n(() => {\n  const root = document.getElementById('ri-flow');\n  if (!root || root.dataset.ready) return;\n  root.dataset.ready = '1';\n  const result = root.querySelector('[data-result]');\n  const steps = [...root.querySelectorAll('[data-step]')];\n  root.querySelectorAll('button[data-status]').forEach(btn => {\n    btn.addEventListener('click', () => {\n      const status = btn.dataset.status;\n      steps.forEach(s => s.removeAttribute('data-active'));\n      result.textContent = status === '200'\n        ? '200 OK → Response Interceptor success handler → return response → Caller يستلم response'\n        : '404 → Response Interceptor error handler → Promise.reject(error) → catch يستلم الـ error';\n      const order = status === '200' ? ['request','backend','interceptor'] : ['request','backend','interceptor'];\n      order.forEach((name, i) => setTimeout(() => {\n        const el = root.querySelector('[data-step=\"' + name + '\"]');\n        if (el) el.setAttribute('data-active','1');\n      }, i * 250));\n    });\n  });\n})();\n</script>"}}

### دلوقتي ركز في المسارين

**لو `200`:**

```text
api.get()
   ↓
Backend
   ↓
Response Interceptor
   ↓
return response
   ↓
Caller
```

**لو `404`:**

```text
api.get()
   ↓
Backend
   ↓
Response Interceptor
   ↓
Promise.reject(error)
   ↓
catch
```

والنقطة اللي عايزك تثبتها:

> **Response Interceptor هو محطة في النص، والـ `return` اللي جواه هو اللي بيحدد هل الـ Promise هيكمل كـ resolved ولا rejected.**

**الخطوة التالية:** اضغط `Simulate 200` ثم `Simulate 404`، وبعدها قولي لي في حالة `404` مين اللي يستقبل الـ `error` في النهاية؟
</details>
الخطوة التالية: نعمل تطبيق صغير جدًا على Response Interceptor ونشوف بعيننا الـ 200 والـ 404 بيمشوا في أنهي path
يلا بينا
ChatGPT
هنعمل تجربة صغيرة تشوف فيها بنفسك مسار `200` ومسار `404` داخل الـ Response Interceptor.

genui{"app_block":{"title":"Response Interceptor Flow","variant":"inline","icon":"study","language":"html","bundle_version":1,"content":"<div id=\"ri-flow\" class=\"w-full max-w-3xl mx-auto space-y-4\">\n  <div class=\"flex flex-wrap gap-2 text-sm\">\n    <button type=\"button\" data-status=\"200\" class=\"rounded-lg border px-4 py-3\">▶ Simulate 200 Success</button>\n    <button type=\"button\" data-status=\"404\" class=\"rounded-lg border px-4 py-3\">▶ Simulate 404 Error</button>\n  </div>\n  <div class=\"grid gap-3 sm:grid-cols-5 items-center text-center text-sm\">\n    <div data-step=\"request\" class=\"rounded-xl border p-3\"><strong>api.get()</strong><div class=\"text-xs opacity-70\">Caller</div></div>\n    <div class=\"text-xl\">↓</div>\n    <div data-step=\"backend\" class=\"rounded-xl border p-3\"><strong>Backend</strong><div class=\"text-xs opacity-70\">Response</div></div>\n    <div class=\"text-xl\">↓</div>\n    <div data-step=\"interceptor\" class=\"rounded-xl border p-3\"><strong>Response Interceptor</strong><div class=\"text-xs opacity-70\">Inspect</div></div>\n  </div>\n  <div data-result class=\"rounded-xl border p-4 text-sm\" aria-live=\"polite\">اضغط على أحد الزرين وشوف الـ path.</div>\n  <div class=\"rounded-xl border p-4 font-mono text-xs whitespace-pre-wrap\">api.interceptors.response.use(\\n  (response) => {\\n    return response;\\n  },\\n  (error) => {\\n    return Promise.reject(error);\\n  }\\n);</div>\n</div>\n<script>\n(() => {\n  const root = document.getElementById('ri-flow');\n  if (!root || root.dataset.ready) return;\n  root.dataset.ready = '1';\n  const result = root.querySelector('[data-result]');\n  const steps = [...root.querySelectorAll('[data-step]')];\n  root.querySelectorAll('button[data-status]').forEach(btn => {\n    btn.addEventListener('click', () => {\n      const status = btn.dataset.status;\n      steps.forEach(s => s.removeAttribute('data-active'));\n      result.textContent = status === '200'\n        ? '200 OK → Response Interceptor success handler → return response → Caller يستلم response'\n        : '404 → Response Interceptor error handler → Promise.reject(error) → catch يستلم الـ error';\n      const order = status === '200' ? ['request','backend','interceptor'] : ['request','backend','interceptor'];\n      order.forEach((name, i) => setTimeout(() => {\n        const el = root.querySelector('[data-step=\"' + name + '\"]');\n        if (el) el.setAttribute('data-active','1');\n      }, i * 250));\n    });\n  });\n})();\n</script>"}}

### دلوقتي ركز في المسارين

**لو `200`:**

```text
api.get()
   ↓
Backend
   ↓
Response Interceptor
   ↓
return response
   ↓
Caller
```

**لو `404`:**

```text
api.get()
   ↓
Backend
   ↓
Response Interceptor
   ↓
Promise.reject(error)
   ↓
catch
```

والنقطة اللي عايزك تثبتها:

> **Response Interceptor هو محطة في النص، والـ `return` اللي جواه هو اللي بيحدد هل الـ Promise هيكمل كـ resolved ولا rejected.**

**الخطوة التالية:** اضغط `Simulate 200` ثم `Simulate 404`، وبعدها قولي لي في حالة `404` مين اللي يستقبل الـ `error` في النهاية؟
<details>
<summary>بضغط بس مافيش حاجه</summary>
user
بضغط بس مافيش حاجه
ChatGPT
تمام، خلينا نسيب الـ interactive ونشوفها **بشكل مباشر وبسيط**، لأن الهدف هنا فهم الـ flow مش الأداة.

### 🗺️ Roadmap

```text
SECTION 4: Interceptors 🟢
  ├─ 4.1 What is an Interceptor    ✅
  ├─ 4.2 Request Interceptor       ✅
  ├─ 4.3 Response Interceptor      🟢 ← هنا
  │    ├─ What is it?               ✅
  │    ├─ Response flow             ✅
  │    ├─ Success handler           ✅
  │    └─ Error handler             🟢
  ├─ 4.4 Authentication            ⬜
  ├─ 4.5 401 Handling              ⬜
  └─ 4.6 Token Refresh + Queue     ⬜
```

خلينا نفترض إن عندنا:

```ts
api.interceptors.response.use(
  (response) => {
    console.log("SUCCESS");
    return response;
  },
  (error) => {
    console.log("ERROR");
    return Promise.reject(error);
  }
);
```

وعندنا API call:

```ts
try {
  const response = await api.get("/users");
  console.log("CALLER SUCCESS");
} catch (error) {
  console.log("CALLER ERROR");
}
```

### الحالة الأولى: Backend رجع `200`

اللي يحصل:

```text
api.get()
   ↓
Backend
   ↓
200
   ↓
Response Interceptor
   ↓
SUCCESS
   ↓
return response
   ↓
CALLER SUCCESS
```

يعني:

```text
SUCCESS
CALLER SUCCESS
```

---

### الحالة الثانية: Backend رجع `404`

اللي يحصل:

```text
api.get()
   ↓
Backend
   ↓
404
   ↓
Response Interceptor
   ↓
ERROR
   ↓
Promise.reject(error)
   ↓
catch
   ↓
CALLER ERROR
```

يعني:

```text
ERROR
CALLER ERROR
```

### الخلاصة

الـ Response Interceptor **مش بيمنع الـ error من الوصول للـ `catch`**.

هو فقط عنده فرصة يشوف الـ error ويتعامل معاه.

ولو قررنا إننا مش هنعالجه هنا:

```ts
return Promise.reject(error);
```

يبقى الـ error يكمل طريقه:

```text
Interceptor
     ↓
catch
```

وده بالضبط سبب وجود:

```ts
return Promise.reject(error);
```

**الخطوة التالية:** جاوبني: في حالة `404`، ليه مش بنكتب `return response` داخل الـ error handler؟
</details>
بضغط بس مافيش حاجه
ChatGPT
تمام، خلينا نسيب الـ interactive ونشوفها **بشكل مباشر وبسيط**، لأن الهدف هنا فهم الـ flow مش الأداة.

### 🗺️ Roadmap

```text
SECTION 4: Interceptors 🟢
  ├─ 4.1 What is an Interceptor    ✅
  ├─ 4.2 Request Interceptor       ✅
  ├─ 4.3 Response Interceptor      🟢 ← هنا
  │    ├─ What is it?               ✅
  │    ├─ Response flow             ✅
  │    ├─ Success handler           ✅
  │    └─ Error handler             🟢
  ├─ 4.4 Authentication            ⬜
  ├─ 4.5 401 Handling              ⬜
  └─ 4.6 Token Refresh + Queue     ⬜
```

خلينا نفترض إن عندنا:

```ts
api.interceptors.response.use(
  (response) => {
    console.log("SUCCESS");
    return response;
  },
  (error) => {
    console.log("ERROR");
    return Promise.reject(error);
  }
);
```

وعندنا API call:

```ts
try {
  const response = await api.get("/users");
  console.log("CALLER SUCCESS");
} catch (error) {
  console.log("CALLER ERROR");
}
```

### الحالة الأولى: Backend رجع `200`

اللي يحصل:

```text
api.get()
   ↓
Backend
   ↓
200
   ↓
Response Interceptor
   ↓
SUCCESS
   ↓
return response
   ↓
CALLER SUCCESS
```

يعني:

```text
SUCCESS
CALLER SUCCESS
```

---

### الحالة الثانية: Backend رجع `404`

اللي يحصل:

```text
api.get()
   ↓
Backend
   ↓
404
   ↓
Response Interceptor
   ↓
ERROR
   ↓
Promise.reject(error)
   ↓
catch
   ↓
CALLER ERROR
```

يعني:

```text
ERROR
CALLER ERROR
```

### الخلاصة

الـ Response Interceptor **مش بيمنع الـ error من الوصول للـ `catch`**.

هو فقط عنده فرصة يشوف الـ error ويتعامل معاه.

ولو قررنا إننا مش هنعالجه هنا:

```ts
return Promise.reject(error);
```

يبقى الـ error يكمل طريقه:

```text
Interceptor
     ↓
catch
```

وده بالضبط سبب وجود:

```ts
return Promise.reject(error);
```

**الخطوة التالية:** جاوبني: في حالة `404`، ليه مش بنكتب `return response` داخل الـ error handler؟
<details>
<summary>مش بنكتب ليه return response؟ عشان في الحالة دي ساعتها هيكون هو هيتعامل معاك إنه resolved، فساعتها هتروح للcaller، اللي هي الدالة بتاعت الـAPI الأساسية، وهتعتبه resolve، بس ساعتها كأنه الطلب نجح، وهي طبعًا هتبقى مشكلة كبيرة. لكن لما أقوله ده promise rejected، فأنا بقوله هيتعامل معاك كأنه promise مرفوض، ساعتها قويتك هتتعامل معاه، هتتعامل إن الـpromise المرفوض دوت يعمل إيه؟ يخش على catch.</summary>
user
مش بنكتب ليه return response؟ عشان في الحالة دي ساعتها هيكون هو هيتعامل معاك إنه resolved، فساعتها هتروح للcaller، اللي هي الدالة بتاعت الـAPI الأساسية، وهتعتبه resolve، بس ساعتها كأنه الطلب نجح، وهي طبعًا هتبقى مشكلة كبيرة. لكن لما أقوله ده promise rejected، فأنا بقوله هيتعامل معاك كأنه promise مرفوض، ساعتها قويتك هتتعامل معاه، هتتعامل إن الـpromise المرفوض دوت يعمل إيه؟ يخش على catch.
ChatGPT
بالضبط. **إجابتك صحيحة جدًا، 10/10.** أنت فاهم الـ Promise flow مش مجرد حافظ `Promise.reject`.

بس أصحح لك نقطة لفظية صغيرة:

لما تقول:

> "قولي الـ rejected يدخل على catch"

الأدق:

> الـ `Promise.reject(error)` يحافظ على الـ Promise في حالة **rejected**، ولما الـ caller يستخدم `await` داخل `try/catch`، الـ rejected Promise يؤدي إلى انتقال التنفيذ إلى `catch`.

المسار كاملًا:

```text
Backend
   ↓
404
   ↓
Response Interceptor
   ↓
Promise.reject(error)
   ↓
Rejected Promise
   ↓
await
   ↓
catch
```

أما لو الـ error handler عمل:

```ts
return error;
```

فالـ callback نفسه **يرجع قيمة بشكل طبيعي**، وبالتالي الـ Promise chain تكمل كـ resolved، وده ممكن يخلي الـ caller يتعامل مع الخطأ كأنه نتيجة عادية.

### 🟢 4.3 تقدمنا كويس

```text
Response Interceptor
├─ الفكرة                    ✅
├─ Success handler            ✅
├─ Error handler              ✅
├─ return response            ✅
└─ Promise.reject(error)      ✅
```

دلوقتي نقدر ننتقل للجزء التالي داخل `4.3`: **هل نرجع الـ `response` كامل، ولا `response.data` فقط؟** ودي نقطة مهمة جدًا لأنك اتعاملت معاها قبل كده في الـ API Layer.

**الخطوة التالية:** هنشرح الفرق بين `return response` و `return response.data` في الـ Response Interceptor بمثال صغير.
</details>
مش بنكتب ليه return response؟ عشان في الحالة دي ساعتها هيكون هو هيتعامل معاك إنه resolved، فساعتها هتروح للcaller، اللي هي الدالة بتاعت الـAPI الأساسية، وهتعتبه resolve، بس ساعتها كأنه الطلب نجح، وهي طبعًا هتبقى مشكلة كبيرة. لكن لما أقوله ده promise rejected، فأنا بقوله هيتعامل معاك كأنه promise مرفوض، ساعتها قويتك هتتعامل معاه، هتتعامل إن الـpromise المرفوض دوت يعمل إيه؟ يخش على catch.
ChatGPT
بالضبط. **إجابتك صحيحة جدًا، 10/10.** أنت فاهم الـ Promise flow مش مجرد حافظ `Promise.reject`.

بس أصحح لك نقطة لفظية صغيرة:

لما تقول:

> "قولي الـ rejected يدخل على catch"

الأدق:

> الـ `Promise.reject(error)` يحافظ على الـ Promise في حالة **rejected**، ولما الـ caller يستخدم `await` داخل `try/catch`، الـ rejected Promise يؤدي إلى انتقال التنفيذ إلى `catch`.

المسار كاملًا:

```text
Backend
   ↓
404
   ↓
Response Interceptor
   ↓
Promise.reject(error)
   ↓
Rejected Promise
   ↓
await
   ↓
catch
```

أما لو الـ error handler عمل:

```ts
return error;
```

فالـ callback نفسه **يرجع قيمة بشكل طبيعي**، وبالتالي الـ Promise chain تكمل كـ resolved، وده ممكن يخلي الـ caller يتعامل مع الخطأ كأنه نتيجة عادية.

### 🟢 4.3 تقدمنا كويس

```text
Response Interceptor
├─ الفكرة                    ✅
├─ Success handler            ✅
├─ Error handler              ✅
├─ return response            ✅
└─ Promise.reject(error)      ✅
```

دلوقتي نقدر ننتقل للجزء التالي داخل `4.3`: **هل نرجع الـ `response` كامل، ولا `response.data` فقط؟** ودي نقطة مهمة جدًا لأنك اتعاملت معاها قبل كده في الـ API Layer.

**الخطوة التالية:** هنشرح الفرق بين `return response` و `return response.data` في الـ Response Interceptor بمثال صغير.
<details>
<summary>قبل ما ندخل على الجزء الجديد أنا كنت محتاج بس أتأكد على حتة، لما أنا عملت console.success، console.log success وعملها في response، وعملت console.log error في حالة الـerror في الـinterceptor. في الحالة دي أنا كده بقوله خد اللي جاي من الـAPI وزود عليه كذا، يعني خد الـresponse وخد الـresponse الناجح من الـAPI وزود عليه console.log كذا. والحاجة التانية في الـerror بقوله خد الفشل اللي رجع وزود عليه كذا ورجعه في الـcatch.</summary>
user
قبل ما ندخل على الجزء الجديد أنا كنت محتاج بس أتأكد على حتة، لما أنا عملت console.success، console.log success وعملها في response، وعملت console.log error في حالة الـerror في الـinterceptor. في الحالة دي أنا كده بقوله خد اللي جاي من الـAPI وزود عليه كذا، يعني خد الـresponse وخد الـresponse الناجح من الـAPI وزود عليه console.log كذا. والحاجة التانية في الـerror بقوله خد الفشل اللي رجع وزود عليه كذا ورجعه في الـcatch.
ChatGPT
أيوه، **بالضبط**. فهمك هنا صحيح، مع تصحيح بسيط في كلمة "زود عليه".

لو كتبت:

```ts
api.interceptors.response.use(
  (response) => {
    console.log("SUCCESS");
    return response;
  },
  (error) => {
    console.log("ERROR");
    return Promise.reject(error);
  }
);
```

### في حالة الـ Success

الـ API رجّع `response`:

```text
API
 ↓
response
 ↓
Interceptor
```

الـ Interceptor يعمل:

```ts
console.log("SUCCESS");
```

يعني **ينفذ حاجة إضافية**، ثم:

```ts
return response;
```

يرجع نفس الـ response للـ caller.

فمش معنى `console.log` إنه أضاف حاجة داخل الـ response نفسه.

يعني:

```text
response
  ↓
console.log("SUCCESS")  ← فعل إضافي
  ↓
return response         ← رجّع الـ response
```

---

### في حالة الـ Error

الـ API رجّع error:

```text
API
 ↓
error
 ↓
Interceptor
```

الـ Interceptor يعمل:

```ts
console.log("ERROR");
```

يعني ينفذ حاجة إضافية، ثم:

```ts
return Promise.reject(error);
```

يعني:

```text
error
 ↓
console.log("ERROR")        ← فعل إضافي
 ↓
Promise.reject(error)       ← حافظ على حالة الفشل
 ↓
catch
```

### إذن فهمك النهائي:

> **الـ Interceptor يقدر يعمل Side Effect إضافي على الـ response أو error، وبعدها يقرر إزاي يكمل الـ flow.**

مثلاً:

```ts
(response) => {
  console.log("SUCCESS");
  return response;
}
```

**شوف الـ response + اعمل حاجة + كمّل.**

و:

```ts
(error) => {
  console.log("ERROR");
  return Promise.reject(error);
}
```

**شوف الـ error + اعمل حاجة + حافظ على الفشل عشان يكمل للـ catch.**

وده مفهوم مهم جدًا لأن بعد كده هنستخدم نفس الفكرة في `logging` و`401` و`token refresh`.

**الخطوة التالية:** ننتقل للفرق بين `return response` و`return response.data` داخل الـ Response Interceptor.
</details>
قبل ما ندخل على الجزء الجديد أنا كنت محتاج بس أتأكد على حتة، لما أنا عملت console.success، console.log success وعملها في response، وعملت console.log error في حالة الـerror في الـinterceptor. في الحالة دي أنا كده بقوله خد اللي جاي من الـAPI وزود عليه كذا، يعني خد الـresponse وخد الـresponse الناجح من الـAPI وزود عليه console.log كذا. والحاجة التانية في الـerror بقوله خد الفشل اللي رجع وزود عليه كذا ورجعه في الـcatch.
ChatGPT
أيوه، **بالضبط**. فهمك هنا صحيح، مع تصحيح بسيط في كلمة "زود عليه".

لو كتبت:

```ts
api.interceptors.response.use(
  (response) => {
    console.log("SUCCESS");
    return response;
  },
  (error) => {
    console.log("ERROR");
    return Promise.reject(error);
  }
);
```

### في حالة الـ Success

الـ API رجّع `response`:

```text
API
 ↓
response
 ↓
Interceptor
```

الـ Interceptor يعمل:

```ts
console.log("SUCCESS");
```

يعني **ينفذ حاجة إضافية**، ثم:

```ts
return response;
```

يرجع نفس الـ response للـ caller.

فمش معنى `console.log` إنه أضاف حاجة داخل الـ response نفسه.

يعني:

```text
response
  ↓
console.log("SUCCESS")  ← فعل إضافي
  ↓
return response         ← رجّع الـ response
```

---

### في حالة الـ Error

الـ API رجّع error:

```text
API
 ↓
error
 ↓
Interceptor
```

الـ Interceptor يعمل:

```ts
console.log("ERROR");
```

يعني ينفذ حاجة إضافية، ثم:

```ts
return Promise.reject(error);
```

يعني:

```text
error
 ↓
console.log("ERROR")        ← فعل إضافي
 ↓
Promise.reject(error)       ← حافظ على حالة الفشل
 ↓
catch
```

### إذن فهمك النهائي:

> **الـ Interceptor يقدر يعمل Side Effect إضافي على الـ response أو error، وبعدها يقرر إزاي يكمل الـ flow.**

مثلاً:

```ts
(response) => {
  console.log("SUCCESS");
  return response;
}
```

**شوف الـ response + اعمل حاجة + كمّل.**

و:

```ts
(error) => {
  console.log("ERROR");
  return Promise.reject(error);
}
```

**شوف الـ error + اعمل حاجة + حافظ على الفشل عشان يكمل للـ catch.**

وده مفهوم مهم جدًا لأن بعد كده هنستخدم نفس الفكرة في `logging` و`401` و`token refresh`.

**الخطوة التالية:** ننتقل للفرق بين `return response` و`return response.data` داخل الـ Response Interceptor.
<details>
<summary>أنا برضه محتاج أفهم لما بقوله return response، يعني بقوله خد الـresponse اللي رجع من الـAPI، رجعه الدالة تاني، اللي هي الـget، ولا بقوله response بقوله خد الاتنين بقى، يعني خد اللي رجع من الـAPI زائد الـinterceptor.</summary>
user
أنا برضه محتاج أفهم لما بقوله return response، يعني بقوله خد الـresponse اللي رجع من الـAPI، رجعه الدالة تاني، اللي هي الـget، ولا بقوله response بقوله خد الاتنين بقى، يعني خد اللي رجع من الـAPI زائد الـinterceptor.
ChatGPT
أيوه، هنا بالضبط محتاج نفرّق بين **الـ `response` نفسه** وبين **الكود اللي الـ interceptor نفذه**.

لما تكتب:

```ts
(response) => {
  console.log("SUCCESS");
  return response;
}
```

فـ `return response` معناها:

> **رجّع نفس الـ response اللي الـ API رجّعه، من غير ما أضيف عليه حاجة.**

يعني **مش**:

```text
API response + interceptor
```

الـ `console.log` مش بيتضاف للـ response أصلًا.

---

### بص على المسار

الـ API رجّع:

```ts
{
  data: {
    id: 10,
    name: "Ahmed"
  },
  status: 200
}
```

الـ Response Interceptor استلمه:

```ts
(response) => {
  console.log("SUCCESS");

  return response;
}
```

الـ `console.log` حصل **خارج الـ response**.

وبعدين:

```ts
return response;
```

رجّع **نفس الـ object**:

```text
API
 ↓
Response
 ↓
Interceptor
 │
 ├─ console.log("SUCCESS")  ← عملية جانبية
 │
 ↓
return response             ← نفس الـ Response
 ↓
Caller
```

فلو عندك:

```ts
const response = await api.get("/users");
```

فالـ `response` هنا هو **الـ response اللي رجع من الـ API**، بعد ما مرّ على الـ interceptor.

---

### طيب إمتى أقول "أنا عدلت الـ response"؟

لو عملت مثلًا:

```ts
(response) => {
  response.data = response.data.payload;

  return response;
}
```

هنا فعلًا أنت **عدلت الـ response** قبل ما ترجعه.

أما:

```ts
console.log("SUCCESS");
return response;
```

فأنت **لم تعدّل الـ response**.

### قاعدة مهمة جدًا

```text
Interceptor
   ↓
يقدر يعمل حاجة
   ↓
ثم يرجع الـ response
```

لكن الـ `return response` **لا يعني**:

> response + interceptor

بل يعني:

> **خذ الـ response الموجود بالفعل ومرّره للمرحلة التالية.**

**الخطوة التالية:** هنطبق نفس الفكرة على `error` ونفرق بين `return error` و`Promise.reject(error)` بصريًا.
</details>
أنا برضه محتاج أفهم لما بقوله return response، يعني بقوله خد الـresponse اللي رجع من الـAPI، رجعه الدالة تاني، اللي هي الـget، ولا بقوله response بقوله خد الاتنين بقى، يعني خد اللي رجع من الـAPI زائد الـinterceptor.
ChatGPT
أيوه، هنا بالضبط محتاج نفرّق بين **الـ `response` نفسه** وبين **الكود اللي الـ interceptor نفذه**.

لما تكتب:

```ts
(response) => {
  console.log("SUCCESS");
  return response;
}
```

فـ `return response` معناها:

> **رجّع نفس الـ response اللي الـ API رجّعه، من غير ما أضيف عليه حاجة.**

يعني **مش**:

```text
API response + interceptor
```

الـ `console.log` مش بيتضاف للـ response أصلًا.

---

### بص على المسار

الـ API رجّع:

```ts
{
  data: {
    id: 10,
    name: "Ahmed"
  },
  status: 200
}
```

الـ Response Interceptor استلمه:

```ts
(response) => {
  console.log("SUCCESS");

  return response;
}
```

الـ `console.log` حصل **خارج الـ response**.

وبعدين:

```ts
return response;
```

رجّع **نفس الـ object**:

```text
API
 ↓
Response
 ↓
Interceptor
 │
 ├─ console.log("SUCCESS")  ← عملية جانبية
 │
 ↓
return response             ← نفس الـ Response
 ↓
Caller
```

فلو عندك:

```ts
const response = await api.get("/users");
```

فالـ `response` هنا هو **الـ response اللي رجع من الـ API**، بعد ما مرّ على الـ interceptor.

---

### طيب إمتى أقول "أنا عدلت الـ response"؟

لو عملت مثلًا:

```ts
(response) => {
  response.data = response.data.payload;

  return response;
}
```

هنا فعلًا أنت **عدلت الـ response** قبل ما ترجعه.

أما:

```ts
console.log("SUCCESS");
return response;
```

فأنت **لم تعدّل الـ response**.

### قاعدة مهمة جدًا

```text
Interceptor
   ↓
يقدر يعمل حاجة
   ↓
ثم يرجع الـ response
```

لكن الـ `return response` **لا يعني**:

> response + interceptor

بل يعني:

> **خذ الـ response الموجود بالفعل ومرّره للمرحلة التالية.**

**الخطوة التالية:** هنطبق نفس الفكرة على `error` ونفرق بين `return error` و`Promise.reject(error)` بصريًا.
<details>
<summary>خلينا في المسألة اللي قلت دلوقتي. أنت response بالـdata بيساوي response بالـpayload ويرجع response. طيب لما يرجع الـresponse هنا بقى، هيرجع الـdata نفسها الـpayload نفسه ولا هيرجع الـresponse أصلاً.</summary>
user
خلينا في المسألة اللي قلت دلوقتي. أنت response بالـdata بيساوي response بالـpayload ويرجع response. طيب لما يرجع الـresponse هنا بقى، هيرجع الـdata نفسها الـpayload نفسه ولا هيرجع الـresponse أصلاً.
ChatGPT
خلينا نثبتها بمثال واحد في **3 دقائق**: لو كتبت `return response` فأنت ترجع **الـ response كامل**، وليس الـ `payload` فقط.

افترض الـ API رجع:

```json id="u6b5g3"
{
  "status": true,
  "code": 200,
  "payload": {
    "id": 10,
    "name": "Ahmed"
  }
}
```

والـ Axios `response` كامل شكله تقريبًا:

```text id="m1t4bx"
response
├── data
│   ├── status: true
│   ├── code: 200
│   └── payload
│       ├── id: 10
│       └── name: "Ahmed"
├── status: 200
├── statusText
├── headers
└── config
```

### لو عملت:

```ts id="9j3tqf"
return response;
```

هيرجع:

```text id="0k2c7d"
AxiosResponse
```

يعني:

```ts id="4c9f7e"
response.data
```

تفضل هي الـ API response:

```ts id="8a2v7n"
{
  status: true,
  code: 200,
  payload: {
    id: 10,
    name: "Ahmed"
  }
}
```

والـ payload نفسها تظل:

```ts id="x5v2qa"
response.data.payload
```

---

### لكن لو عملت:

```ts id="v8m3kd"
return response.data;
```

ساعتها أنت **مش بترجع AxiosResponse كامل**.

أنت بترجع:

```text id="q7n4zp"
response.data
```

يعني الـ API response wrapper.

ولو عملت:

```ts id="w2r6yc"
return response.data.payload;
```

ساعتها فقط أنت بترجع:

```ts id="z1k8hs"
{
  id: 10,
  name: "Ahmed"
}
```

يعني **الـ payload نفسها**.

### احفظ الثلاثة دول:

```text id="m4h7qc"
return response
        ↓
AxiosResponse


return response.data
        ↓
API Response


return response.data.payload
        ↓
Actual Data
```

وده مهم جدًا لأننا هنستخدمه بعد شوية في تصميم الـ `apiClient`.

**الخطوة التالية:** هوريك ليه غالبًا **مش بنعمل `return response.data` مباشرةً في الـ Response Interceptor**، وإمتى يكون ده قرار صحيح.
</details>
خلينا في المسألة اللي قلت دلوقتي. أنت response بالـdata بيساوي response بالـpayload ويرجع response. طيب لما يرجع الـresponse هنا بقى، هيرجع الـdata نفسها الـpayload نفسه ولا هيرجع الـresponse أصلاً.
ChatGPT
خلينا نثبتها بمثال واحد في **3 دقائق**: لو كتبت `return response` فأنت ترجع **الـ response كامل**، وليس الـ `payload` فقط.

افترض الـ API رجع:

```json id="u6b5g3"
{
  "status": true,
  "code": 200,
  "payload": {
    "id": 10,
    "name": "Ahmed"
  }
}
```

والـ Axios `response` كامل شكله تقريبًا:

```text id="m1t4bx"
response
├── data
│   ├── status: true
│   ├── code: 200
│   └── payload
│       ├── id: 10
│       └── name: "Ahmed"
├── status: 200
├── statusText
├── headers
└── config
```

### لو عملت:

```ts id="9j3tqf"
return response;
```

هيرجع:

```text id="0k2c7d"
AxiosResponse
```

يعني:

```ts id="4c9f7e"
response.data
```

تفضل هي الـ API response:

```ts id="8a2v7n"
{
  status: true,
  code: 200,
  payload: {
    id: 10,
    name: "Ahmed"
  }
}
```

والـ payload نفسها تظل:

```ts id="x5v2qa"
response.data.payload
```

---

### لكن لو عملت:

```ts id="v8m3kd"
return response.data;
```

ساعتها أنت **مش بترجع AxiosResponse كامل**.

أنت بترجع:

```text id="q7n4zp"
response.data
```

يعني الـ API response wrapper.

ولو عملت:

```ts id="w2r6yc"
return response.data.payload;
```

ساعتها فقط أنت بترجع:

```ts id="z1k8hs"
{
  id: 10,
  name: "Ahmed"
}
```

يعني **الـ payload نفسها**.

### احفظ الثلاثة دول:

```text id="m4h7qc"
return response
        ↓
AxiosResponse


return response.data
        ↓
API Response


return response.data.payload
        ↓
Actual Data
```

وده مهم جدًا لأننا هنستخدمه بعد شوية في تصميم الـ `apiClient`.

**الخطوة التالية:** هوريك ليه غالبًا **مش بنعمل `return response.data` مباشرةً في الـ Response Interceptor**، وإمتى يكون ده قرار صحيح.
<details>
<summary>مش هو ده اللي أنا أصله. أنا كل اللي عايز أفهمه إني هنا الـinterceptor. إحنا دلوقتي رجعتلنا بيانات بـX، اللي راجع من الـY X. أنا عايز أقوله U مع الـX Y، مش دي وظيفة الـinterceptor؟ طيب أنا ضفت Y. لما أجي أرجع الـresponse أقوله return the response. طب أنا على كلامك مش هأرجع الـY، ساعتها إيه بقى وظيفة الـinterceptor ساعتها؟</summary>
user
مش هو ده اللي أنا أصله. أنا كل اللي عايز أفهمه إني هنا الـinterceptor. إحنا دلوقتي رجعتلنا بيانات بـX، اللي راجع من الـY X. أنا عايز أقوله U مع الـX Y، مش دي وظيفة الـinterceptor؟ طيب أنا ضفت Y. لما أجي أرجع الـresponse أقوله return the response. طب أنا على كلامك مش هأرجع الـY، ساعتها إيه بقى وظيفة الـinterceptor ساعتها؟
ChatGPT
أنت هنا لمست **النقطة الصح فعلًا**. خلينا نفصلها في دقيقتين:

**الـ Interceptor مش وظيفته بالضرورة إنه يضيف حاجة للـ response.**  
وظيفته الأساسية إنه **يمر على الـ response ويتيح لك تعمل عليه أي processing تريده قبل ما يكمل طريقه**.

مثلاً الـ API رجّع:

```text
X = response
```

والـ Interceptor عمل:

```ts
(response) => {
  console.log("SUCCESS");
  return response;
}
```

هنا أنت **لم تضف Y إلى X**.

أنت عملت:

```text
API
 ↓
X
 ↓
Interceptor
 ↓
اعمل Y = console.log
 ↓
رجّع X
 ↓
Caller
```

فالـ `Y` حصل، لكن **Y مش جزء من X**.

---

### طيب إمتى الـ Interceptor فعلًا يضيف أو يعدل في الـ response؟

لو أنت عايز ده، ممكن تعمل:

```ts
(response) => {
  response.data.extra = "Hello";

  return response;
}
```

هنا فعلًا:

```text
X
 ↓
Interceptor
 ↓
X + extra
 ↓
return response
```

فالـ Caller يستلم الـ response المعدل.

أو ممكن تعمل transformation أكبر:

```ts
(response) => {
  return response.data.payload;
}
```

هنا أنت غيرت **القيمة اللي هتخرج من الـ interceptor** بالكامل.

---

### إذن عندنا 3 استخدامات

```text
1. Observe
   ↓
   console.log(response)
   ↓
   return response


2. Modify
   ↓
   response.data.something = ...
   ↓
   return response


3. Transform
   ↓
   return response.data.payload
```

فأنت كنت فاهم الـ Interceptor على إنه:

> "API رجع X، وأنا لازم أضيف Y عليه."

وده **واحد من الاستخدامات الممكنة، لكنه مش تعريف الـ Interceptor**.

التعريف الأدق:

> **Interceptor هو نقطة تدخل تسمح لك تفحص أو تعدل أو تحول الـ request/response قبل ما يكمل الـ flow.**

وده سبب إن المثال البسيط:

```ts
console.log("SUCCESS");
return response;
```

مفيد للتعلم، لأنه بيورّيك إن الـ interceptor **يمر من خلاله الـ response حتى لو أنت لم تعدله**.

**الخطوة التالية:** نطبق مثال حقيقي نضيف فيه `Y` إلى `X` داخل الـ Response Interceptor، عشان تشوف الفرق بين `return response` و`return modifiedResponse`.
</details>
مش هو ده اللي أنا أصله. أنا كل اللي عايز أفهمه إني هنا الـinterceptor. إحنا دلوقتي رجعتلنا بيانات بـX، اللي راجع من الـY X. أنا عايز أقوله U مع الـX Y، مش دي وظيفة الـinterceptor؟ طيب أنا ضفت Y. لما أجي أرجع الـresponse أقوله return the response. طب أنا على كلامك مش هأرجع الـY، ساعتها إيه بقى وظيفة الـinterceptor ساعتها؟
ChatGPT
أنت هنا لمست **النقطة الصح فعلًا**. خلينا نفصلها في دقيقتين:

**الـ Interceptor مش وظيفته بالضرورة إنه يضيف حاجة للـ response.**  
وظيفته الأساسية إنه **يمر على الـ response ويتيح لك تعمل عليه أي processing تريده قبل ما يكمل طريقه**.

مثلاً الـ API رجّع:

```text
X = response
```

والـ Interceptor عمل:

```ts
(response) => {
  console.log("SUCCESS");
  return response;
}
```

هنا أنت **لم تضف Y إلى X**.

أنت عملت:

```text
API
 ↓
X
 ↓
Interceptor
 ↓
اعمل Y = console.log
 ↓
رجّع X
 ↓
Caller
```

فالـ `Y` حصل، لكن **Y مش جزء من X**.

---

### طيب إمتى الـ Interceptor فعلًا يضيف أو يعدل في الـ response؟

لو أنت عايز ده، ممكن تعمل:

```ts
(response) => {
  response.data.extra = "Hello";

  return response;
}
```

هنا فعلًا:

```text
X
 ↓
Interceptor
 ↓
X + extra
 ↓
return response
```

فالـ Caller يستلم الـ response المعدل.

أو ممكن تعمل transformation أكبر:

```ts
(response) => {
  return response.data.payload;
}
```

هنا أنت غيرت **القيمة اللي هتخرج من الـ interceptor** بالكامل.

---

### إذن عندنا 3 استخدامات

```text
1. Observe
   ↓
   console.log(response)
   ↓
   return response


2. Modify
   ↓
   response.data.something = ...
   ↓
   return response


3. Transform
   ↓
   return response.data.payload
```

فأنت كنت فاهم الـ Interceptor على إنه:

> "API رجع X، وأنا لازم أضيف Y عليه."

وده **واحد من الاستخدامات الممكنة، لكنه مش تعريف الـ Interceptor**.

التعريف الأدق:

> **Interceptor هو نقطة تدخل تسمح لك تفحص أو تعدل أو تحول الـ request/response قبل ما يكمل الـ flow.**

وده سبب إن المثال البسيط:

```ts
console.log("SUCCESS");
return response;
```

مفيد للتعلم، لأنه بيورّيك إن الـ interceptor **يمر من خلاله الـ response حتى لو أنت لم تعدله**.

**الخطوة التالية:** نطبق مثال حقيقي نضيف فيه `Y` إلى `X` داخل الـ Response Interceptor، عشان تشوف الفرق بين `return response` و`return modifiedResponse`.
<details>
<summary>إيه ده يعني أنا بقى أخش جوه الداتا أقوله والله أخش جوه الداتا وأقوله مثلاً أضيف حاجة اسمها إكسترا، أو أضيف حاجة اسمها يوزر تاني، أو في مثلاً النيم أقوله أضيف حاجة اسمها يوزر نيم، وأخلي اليوزر نيم بكذا، مثلاً مفيش ID، أو في ID وعايز أعمل مثلاً حاجة اسمها Name ID حاجة تانية، ينفع؟</summary>
user
إيه ده يعني أنا بقى أخش جوه الداتا أقوله والله أخش جوه الداتا وأقوله مثلاً أضيف حاجة اسمها إكسترا، أو أضيف حاجة اسمها يوزر تاني، أو في مثلاً النيم أقوله أضيف حاجة اسمها يوزر نيم، وأخلي اليوزر نيم بكذا، مثلاً مفيش ID، أو في ID وعايز أعمل مثلاً حاجة اسمها Name ID حاجة تانية، ينفع؟
ChatGPT
أيوه، **ينفع تمامًا**. أنت كده وصلت لفكرة مهمة جدًا: الـ `Response Interceptor` ممكن يعمل **Transformation / Modification** على البيانات قبل ما تسلّم للـ caller.

مثلاً الـ API رجّع:

```ts
response.data = {
  status: true,
  payload: {
    id: 10,
    name: "Ahmed"
  }
}
```

ممكن داخل الـ interceptor تعمل:

```ts
(response) => {
  response.data.payload.username = "Ahmed10";

  return response;
}
```

فيبقى الـ `payload`:

```ts
{
  id: 10,
  name: "Ahmed",
  username: "Ahmed10"
}
```

### وممكن تعمل حاجة محسوبة من بيانات موجودة

مثلاً:

```ts
(response) => {
  const user = response.data.payload;

  user.nameId = `${user.name}_${user.id}`;

  return response;
}
```

فتصبح:

```ts
{
  id: 10,
  name: "Ahmed",
  nameId: "Ahmed_10"
}
```

لكن هنا في **ملاحظة Architecture مهمة جدًا**:

مش معنى إنك تقدر تعمل كل ده إننا **لازم** نعمله في الـ interceptor.

لو الـ `username` أو `nameId` ده **Business Logic خاص بالـ Users**، غالبًا مكانه الأنسب يكون في الـ **Users API layer أو mapper**، مش Global Response Interceptor.

الـ Response Interceptor مناسب أكثر للحاجات **العامة المشتركة**، مثل:

```text
كل الـ responses
     ↓
Response Interceptor
     ↓
logging
normalization
common handling
401
```

أما:

```text
User response
     ↓
username
nameId
user-specific transformation
```

فده غالبًا Feature-specific.

وده بيرجعنا مباشرةً لـ **Feature-Based Architecture** اللي أخدناها في Section 0.

**الخطوة التالية:** هنعمل مثال فيه `response.data.payload` ونضيف عليه `username` داخل الـ interceptor، ونشوف بالضبط إيه اللي الـ caller هيستلمه.
</details>
إيه ده يعني أنا بقى أخش جوه الداتا أقوله والله أخش جوه الداتا وأقوله مثلاً أضيف حاجة اسمها إكسترا، أو أضيف حاجة اسمها يوزر تاني، أو في مثلاً النيم أقوله أضيف حاجة اسمها يوزر نيم، وأخلي اليوزر نيم بكذا، مثلاً مفيش ID، أو في ID وعايز أعمل مثلاً حاجة اسمها Name ID حاجة تانية، ينفع؟
ChatGPT
أيوه، **ينفع تمامًا**. أنت كده وصلت لفكرة مهمة جدًا: الـ `Response Interceptor` ممكن يعمل **Transformation / Modification** على البيانات قبل ما تسلّم للـ caller.

مثلاً الـ API رجّع:

```ts
response.data = {
  status: true,
  payload: {
    id: 10,
    name: "Ahmed"
  }
}
```

ممكن داخل الـ interceptor تعمل:

```ts
(response) => {
  response.data.payload.username = "Ahmed10";

  return response;
}
```

فيبقى الـ `payload`:

```ts
{
  id: 10,
  name: "Ahmed",
  username: "Ahmed10"
}
```

### وممكن تعمل حاجة محسوبة من بيانات موجودة

مثلاً:

```ts
(response) => {
  const user = response.data.payload;

  user.nameId = `${user.name}_${user.id}`;

  return response;
}
```

فتصبح:

```ts
{
  id: 10,
  name: "Ahmed",
  nameId: "Ahmed_10"
}
```

لكن هنا في **ملاحظة Architecture مهمة جدًا**:

مش معنى إنك تقدر تعمل كل ده إننا **لازم** نعمله في الـ interceptor.

لو الـ `username` أو `nameId` ده **Business Logic خاص بالـ Users**، غالبًا مكانه الأنسب يكون في الـ **Users API layer أو mapper**، مش Global Response Interceptor.

الـ Response Interceptor مناسب أكثر للحاجات **العامة المشتركة**، مثل:

```text
كل الـ responses
     ↓
Response Interceptor
     ↓
logging
normalization
common handling
401
```

أما:

```text
User response
     ↓
username
nameId
user-specific transformation
```

فده غالبًا Feature-specific.

وده بيرجعنا مباشرةً لـ **Feature-Based Architecture** اللي أخدناها في Section 0.

**الخطوة التالية:** هنعمل مثال فيه `response.data.payload` ونضيف عليه `username` داخل الـ interceptor، ونشوف بالضبط إيه اللي الـ caller هيستلمه.
<details>
<summary>أنا فهمت قصدك كده خلاص. بس القصة كلها إحنا مش بنعمل الكلام ده. أنا في خيالي إحنا مش بنلعب في البيانات اللي هي جاية من الـ API، عشان لو عدلت فيها مش الـ API نفسه مش هيعرف. فاهم قصدي.</summary>
user
أنا فهمت قصدك كده خلاص. بس القصة كلها إحنا مش بنعمل الكلام ده. أنا في خيالي إحنا مش بنلعب في البيانات اللي هي جاية من الـ API، عشان لو عدلت فيها مش الـ API نفسه مش هيعرف. فاهم قصدي.
ChatGPT
أيوه، **فاهم قصدك بالضبط**، ودي نقطة مهمة جدًا.

أنت بتقول:

> "لو عدلت `response` داخل الـ Interceptor، أنا كده عدلت النسخة اللي عند الـ Frontend، مش البيانات الأصلية الموجودة عند الـ Backend."

**وده صحيح 100%.**

مثلاً الـ Backend رجّع:

```json id="g5r3xn"
{
  "id": 10,
  "name": "Ahmed"
}
```

وعملت في الـ Interceptor:

```ts id="w7k2mz"
response.data.name = "Mohamed";

return response;
```

اللي حصل:

```text id="r8m4qa"
Backend
   │
   │  name = Ahmed
   ↓
Frontend
   │
   ↓
Interceptor
   │
   │  name = Mohamed
   ↓
Caller
```

الـ Backend **لسه عنده**:

```text id="b5c7qy"
name = Ahmed
```

والـ Frontend فقط أصبح عنده:

```text id="z3n6vp"
name = Mohamed
```

لأنك لم تبعت Request جديد للـ Backend.

### ودي نقطة مهمة جدًا

الـ Response Interceptor **لا يعدّل الـ API Database**.

هو يتعامل مع **الـ response الموجود عند الـ client**.

لو أنت عايز تعدل البيانات الأصلية عند الـ Backend، لازم تعمل Request مناسب، مثل:

```ts id="m7k2fp"
await api.patch("/users/10", {
  name: "Mohamed"
});
```

وده شيء مختلف تمامًا.

### لذلك إحساسك الأول كان منطقي

في أغلب تطبيقاتنا، مش هنستخدم Response Interceptor عشان نقعد نغير:

```text
name
id
username
payload
```

لكل Feature.

إحنا غالبًا هنستخدمه في **Cross-cutting concerns**، زي:

```text
Response Interceptor
       ↓
Logging
Error handling
401
Token refresh
Common response processing
```

أما تحويل بيانات `User` تحديدًا، فده غالبًا مكانه الـ **Users feature / mapper**.

**الخطوة التالية:** نكمل `Response Interceptor` ونشوف الاستخدام العملي الحقيقي بتاعه، بدون ما نلعب في بيانات الـ API.
</details>
أنا فهمت قصدك كده خلاص. بس القصة كلها إحنا مش بنعمل الكلام ده. أنا في خيالي إحنا مش بنلعب في البيانات اللي هي جاية من الـ API، عشان لو عدلت فيها مش الـ API نفسه مش هيعرف. فاهم قصدي.
ChatGPT
أيوه، **فاهم قصدك بالضبط**، ودي نقطة مهمة جدًا.

أنت بتقول:

> "لو عدلت `response` داخل الـ Interceptor، أنا كده عدلت النسخة اللي عند الـ Frontend، مش البيانات الأصلية الموجودة عند الـ Backend."

**وده صحيح 100%.**

مثلاً الـ Backend رجّع:

```json id="g5r3xn"
{
  "id": 10,
  "name": "Ahmed"
}
```

وعملت في الـ Interceptor:

```ts id="w7k2mz"
response.data.name = "Mohamed";

return response;
```

اللي حصل:

```text id="r8m4qa"
Backend
   │
   │  name = Ahmed
   ↓
Frontend
   │
   ↓
Interceptor
   │
   │  name = Mohamed
   ↓
Caller
```

الـ Backend **لسه عنده**:

```text id="b5c7qy"
name = Ahmed
```

والـ Frontend فقط أصبح عنده:

```text id="z3n6vp"
name = Mohamed
```

لأنك لم تبعت Request جديد للـ Backend.

### ودي نقطة مهمة جدًا

الـ Response Interceptor **لا يعدّل الـ API Database**.

هو يتعامل مع **الـ response الموجود عند الـ client**.

لو أنت عايز تعدل البيانات الأصلية عند الـ Backend، لازم تعمل Request مناسب، مثل:

```ts id="m7k2fp"
await api.patch("/users/10", {
  name: "Mohamed"
});
```

وده شيء مختلف تمامًا.

### لذلك إحساسك الأول كان منطقي

في أغلب تطبيقاتنا، مش هنستخدم Response Interceptor عشان نقعد نغير:

```text
name
id
username
payload
```

لكل Feature.

إحنا غالبًا هنستخدمه في **Cross-cutting concerns**، زي:

```text
Response Interceptor
       ↓
Logging
Error handling
401
Token refresh
Common response processing
```

أما تحويل بيانات `User` تحديدًا، فده غالبًا مكانه الـ **Users feature / mapper**.

**الخطوة التالية:** نكمل `Response Interceptor` ونشوف الاستخدام العملي الحقيقي بتاعه، بدون ما نلعب في بيانات الـ API.
<details>
<summary>كمل</summary>
user
كمل
ChatGPT
نكمل الآن **4.3 Response Interceptor** في حوالي **10 دقائق** ونثبت استخدامه الحقيقي، من غير تعديل بيانات الـ API.

### 🗺️ Roadmap

```text id="p7h2kx"
SECTION 4: Interceptors 🟢
  ├─ 4.1 What is an Interceptor    ✅
  ├─ 4.2 Request Interceptor       ✅
  ├─ 4.3 Response Interceptor      🟢 ← هنا
  │    ├─ What is it?               ✅
  │    ├─ Response flow             ✅
  │    ├─ Success handler            ✅
  │    ├─ Error handler              ✅
  │    └─ Real-world usage          🟢
  ├─ 4.4 Authentication            ⬜
  ├─ 4.5 401 Handling              ⬜
  └─ 4.6 Token Refresh + Queue     ⬜
```

## الاستخدام الحقيقي

خلينا نعمل `apiClient.ts`:

```ts id="h2k9qp"
import axios from "axios";

export const api = axios.create({
  baseURL: "https://api.example.com",
  timeout: 5000,
});
```

دلوقتي نضيف Response Interceptor:

```ts id="z6c4rx"
api.interceptors.response.use(
  (response) => {
    console.log("Response received:", response.status);

    return response;
  },

  (error) => {
    console.log("API Error:", error);

    return Promise.reject(error);
  }
);
```

هنا إحنا **مش بنغير البيانات**.

إحنا بنراقب الـ response ونعمل processing عام.

---
</details>
كمل
ChatGPT
نكمل الآن **4.3 Response Interceptor** في حوالي **10 دقائق** ونثبت استخدامه الحقيقي، من غير تعديل بيانات الـ API.

### 🗺️ Roadmap

```text id="p7h2kx"
SECTION 4: Interceptors 🟢
  ├─ 4.1 What is an Interceptor    ✅
  ├─ 4.2 Request Interceptor       ✅
  ├─ 4.3 Response Interceptor      🟢 ← هنا
  │    ├─ What is it?               ✅
  │    ├─ Response flow             ✅
  │    ├─ Success handler            ✅
  │    ├─ Error handler              ✅
  │    └─ Real-world usage          🟢
  ├─ 4.4 Authentication            ⬜
  ├─ 4.5 401 Handling              ⬜
  └─ 4.6 Token Refresh + Queue     ⬜
```

## الاستخدام الحقيقي

خلينا نعمل `apiClient.ts`:

```ts id="h2k9qp"
import axios from "axios";

export const api = axios.create({
  baseURL: "https://api.example.com",
  timeout: 5000,
});
```

دلوقتي نضيف Response Interceptor:

```ts id="z6c4rx"
api.interceptors.response.use(
  (response) => {
    console.log("Response received:", response.status);

    return response;
  },

  (error) => {
    console.log("API Error:", error);

    return Promise.reject(error);
  }
);
```

هنا إحنا **مش بنغير البيانات**.

إحنا بنراقب الـ response ونعمل processing عام.

---