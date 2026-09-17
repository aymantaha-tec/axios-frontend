# مثال حقيقي

لو عملت:

```ts id="q7p3mn"
const response = await api.get("/users");
```

والـ Backend رجع:

```text id="c8x2vw"
200 OK
```

الـ Interceptor يعمل:

```ts id="h1z5kf"
console.log("Response received:", 200);
```

وبعدين:

```ts id="a4m8ys"
return response;
```

فالـ caller يستلم الـ response طبيعي.

---

## ولو حصل Error؟

مثلاً:

```text id="w9c3kx"
404 Not Found
```

يدخل:

```ts id="f5n2qb"
(error) => {
  console.log("API Error:", error);

  return Promise.reject(error);
}
```

فنحن عملنا:

```text id="y7k1mz"
Error
 ↓
Interceptor
 ↓
Log
 ↓
Promise.reject(error)
 ↓
catch
```

إذن الـ Interceptor هنا عمل **Logging**، لكنه لم يغير الـ Error ولم يمنعه من الوصول للـ caller.

---

# طيب ليه ده مفيد؟

تخيل عندك:

```text id="m8q2vx"
getUsers()
getProducts()
getOrders()
getProfile()
createUser()
updateUser()
deleteProduct()
...
```

لو عايز تعمل logging لكل الـ API calls، مش منطقي تعمل:

```ts id="s6d1qa"
console.log(...)
```

في كل function.

بدل كده:

```text id="j4p8cz"
                 API Requests
                      ↓
             Response Interceptor
                      ↓
                   Logging
```

**مكان واحد فقط.**

وده نفس المبدأ اللي استخدمناه مع `Authorization` في الـ Request Interceptor.

---

# Request vs Response

خلي الصورة دي ثابتة في دماغك:

```text id="v6r2na"
              REQUEST
                 ↓
Component
    ↓
api.get()
    ↓
Request Interceptor
    ↓
Backend
    ↓
Response Interceptor
    ↓
Component
              RESPONSE
```

### Request Interceptor

بيتعامل مع **الحاجة وهي رايحة**:

```text id="q9m3kf"
Request
  ↓
ضيف Authorization
  ↓
Backend
```

### Response Interceptor

بيتعامل مع **الحاجة وهي راجعة**:

```text id="t4x7bp"
Backend
  ↓
Response
  ↓
اعمل Logging / Common Handling
  ↓
Caller
```

---

## نقطة TypeScript مهمة

في:

```ts id="b2r6hy"
(response) => {
```

TypeScript يعرف نوع `response` تلقائيًا بسبب **Contextual Typing**.

ونفس الكلام في:

```ts id="k8p4zs"
(error) => {
```

Axios هو اللي بيحدد الـ context للـ callback.

مش محتاج تكتب:

```ts id="m5q1vd"
(response: ...)
```

إلا لو عندك سبب فعلي.

---

### 🎯 Interview Question

عندك 30 API calls في التطبيق، وعايز تعمل **logging لكل الـ responses** في مكان واحد.

هل الأفضل تحط `console.log` داخل كل API function، ولا تستخدم **Response Interceptor**؟ وليه؟

جاوب كأنك في Interview.
<details>
<summary>لا والله في الحالة دي لو أنا عندي EBITDA codes 30 وحده، ففي الحالة دي لأ، الأفضل أعمل response إني سيبه لوحده عشان الـcentralization. فالـcentralized هنا هحط بس الـcodes good look مرة واحدة جوا كل حد مش هيبقوا 30 حتى لو ألف. أول حاجة أمنع التكرار، تاني حاجة لو حبيت أعدل الـconsole log دي هحب أعدلها بطريقة واحدة في الـcentralized. لذلك الـresponse interceptor أفضل بكتير.</summary>
user
لا والله في الحالة دي لو أنا عندي EBITDA codes 30 وحده، ففي الحالة دي لأ، الأفضل أعمل response إني سيبه لوحده عشان الـcentralization. فالـcentralized هنا هحط بس الـcodes good look مرة واحدة جوا كل حد مش هيبقوا 30 حتى لو ألف. أول حاجة أمنع التكرار، تاني حاجة لو حبيت أعدل الـconsole log دي هحب أعدلها بطريقة واحدة في الـcentralized. لذلك الـresponse interceptor أفضل بكتير.
ChatGPT
إجابتك **10/10**. أنت استخدمت بالضبط مبدأ **Centralization + DRY**.

بس واضح إن الـ voice-to-text لخبط شوية في المصطلحات، وأنا فاهم إن قصدك:

> عندي 30 API calls، فبدل ما أكرر `console.log` في كل واحدة، أحطه مرة واحدة في `Response Interceptor`.

وده صحيح.

```text id="h4k7ps"
30 API Calls
     ↓
Response Interceptor
     ↓
Logging
```

بدل:

```text id="x8m2qa"
getUsers()    → console.log ❌
getProducts() → console.log ❌
getOrders()   → console.log ❌
getProfile()  → console.log ❌
...
```

### ليه الـ Interceptor أفضل؟

**1. DRY**

منمنع تكرار نفس الـ logic.

**2. Centralization**

الـ logging موجود في مكان واحد.

**3. Maintainability**

لو غيرت:

```ts id="q6v9xr"
console.log(...)
```

إلى:

```ts id="p2k8md"
logger.info(...)
```

تغيره في مكان واحد.

**4. Consistency**

كل الـ responses تمر بنفس الـ processing.

وده بالضبط من أسباب وجود الـ **Interceptor**.

---

### 🟢 4.3 وصلنا تقريبًا للنهاية

```text id="n5c7vz"
SECTION 4: Interceptors
  ├─ 4.1 What is an Interceptor    ✅
  ├─ 4.2 Request Interceptor       ✅
  └─ 4.3 Response Interceptor      🟢
       ├─ Success handler           ✅
       ├─ Error handler             ✅
       ├─ Promise.reject            ✅
       └─ Centralized processing    ✅
```

باقي نقطة صغيرة جدًا في `4.3` قبل ما نقفلها: **إيه اللي يحصل لو الـ Response Interceptor نفسه عمل `throw` أو رجّع Promise مرفوض؟**

**الخطوة التالية:** نشرح النقطة دي بمثال بسيط، وبعدها نعمل Final Interview لـ `4.3` وننتقل لـ `4.4 Authentication`.
</details>
لا والله في الحالة دي لو أنا عندي EBITDA codes 30 وحده، ففي الحالة دي لأ، الأفضل أعمل response إني سيبه لوحده عشان الـcentralization. فالـcentralized هنا هحط بس الـcodes good look مرة واحدة جوا كل حد مش هيبقوا 30 حتى لو ألف. أول حاجة أمنع التكرار، تاني حاجة لو حبيت أعدل الـconsole log دي هحب أعدلها بطريقة واحدة في الـcentralized. لذلك الـresponse interceptor أفضل بكتير.
ChatGPT
إجابتك **10/10**. أنت استخدمت بالضبط مبدأ **Centralization + DRY**.

بس واضح إن الـ voice-to-text لخبط شوية في المصطلحات، وأنا فاهم إن قصدك:

> عندي 30 API calls، فبدل ما أكرر `console.log` في كل واحدة، أحطه مرة واحدة في `Response Interceptor`.

وده صحيح.

```text id="h4k7ps"
30 API Calls
     ↓
Response Interceptor
     ↓
Logging
```

بدل:

```text id="x8m2qa"
getUsers()    → console.log ❌
getProducts() → console.log ❌
getOrders()   → console.log ❌
getProfile()  → console.log ❌
...
```

### ليه الـ Interceptor أفضل؟

**1. DRY**

منمنع تكرار نفس الـ logic.

**2. Centralization**

الـ logging موجود في مكان واحد.

**3. Maintainability**

لو غيرت:

```ts id="q6v9xr"
console.log(...)
```

إلى:

```ts id="p2k8md"
logger.info(...)
```

تغيره في مكان واحد.

**4. Consistency**

كل الـ responses تمر بنفس الـ processing.

وده بالضبط من أسباب وجود الـ **Interceptor**.

---

### 🟢 4.3 وصلنا تقريبًا للنهاية

```text id="n5c7vz"
SECTION 4: Interceptors
  ├─ 4.1 What is an Interceptor    ✅
  ├─ 4.2 Request Interceptor       ✅
  └─ 4.3 Response Interceptor      🟢
       ├─ Success handler           ✅
       ├─ Error handler             ✅
       ├─ Promise.reject            ✅
       └─ Centralized processing    ✅
```

باقي نقطة صغيرة جدًا في `4.3` قبل ما نقفلها: **إيه اللي يحصل لو الـ Response Interceptor نفسه عمل `throw` أو رجّع Promise مرفوض؟**

**الخطوة التالية:** نشرح النقطة دي بمثال بسيط، وبعدها نعمل Final Interview لـ `4.3` وننتقل لـ `4.4 Authentication`.
<details>
<summary>تمام</summary>
user
تمام
ChatGPT
نثبت النقطة دي في **5 دقائق**: ماذا يحدث لو الـ `Response Interceptor` نفسه عمل `throw` أو رجّع `Promise.reject`؟

### 🗺️ Roadmap

```text id="n3x8qa"
SECTION 4: Interceptors 🟢
  ├─ 4.1 What is an Interceptor    ✅
  ├─ 4.2 Request Interceptor       ✅
  ├─ 4.3 Response Interceptor      🟢
  │    ├─ Success handler            ✅
  │    ├─ Error handler              ✅
  │    ├─ Promise.reject             ✅
  │    ├─ Centralized processing     ✅
  │    └─ throw / reject             🟢 ← هنا
  ├─ 4.4 Authentication             ⬜
  ├─ 4.5 401 Handling               ⬜
  └─ 4.6 Token Refresh + Queue      ⬜
```

### عندنا Success

```ts id="j6t2vp"
api.interceptors.response.use(
  (response) => {
    console.log("SUCCESS");

    return response;
  }
);
```

كل شيء طبيعي:

```text id="c9q4mx"
Backend
   ↓
200
   ↓
Interceptor
   ↓
return response
   ↓
Caller
```

---

### طيب لو عملنا `throw`؟

مثلاً:

```ts id="z7k3nb"
api.interceptors.response.use(
  (response) => {
    throw new Error("Something went wrong");
  }
);
```

هنا أنت **أنتجت Error جديد داخل الـ interceptor**.

فالـ flow يصبح:

```text id="r4m8yk"
Backend
   ↓
200
   ↓
Response Interceptor
   ↓
throw Error
   ↓
❌ Rejected Promise
   ↓
catch
```

يعني حتى لو الـ Backend رجع `200`، أنت قررت داخل الـ interceptor إن العملية تفشل.

---

### ونفس الفكرة مع:

```ts id="a8p2wf"
return Promise.reject(error);
```

أنت بتقول:

> لا تكمل كـ Success، كمل كـ Rejected Promise.

---

### الفرق البسيط

```ts id="x2m7kc"
throw new Error("...");
```

أنت **بتنشئ وترمي Error**.

أما:

```ts id="q5v9dz"
return Promise.reject(error);
```

أنت عندك **Error موجود بالفعل** وبتحافظ على الـ Promise كـ rejected.

---

### مثال واقعي

```ts id="u4h8ns"
api.interceptors.response.use(
  (response) => {
    if (response.data.status !== true) {
      throw new Error("API operation failed");
    }

    return response;
  },

  (error) => {
    return Promise.reject(error);
  }
);
```

هنا الـ interceptor ممكن يعمل validation أو processing، ولو قرر إن فيه مشكلة، يـ `throw`.

لكن خد بالك: **مش عايزين ندخل في API validation وauthentication دلوقتي**. الهدف فقط فهم الـ Promise flow.

### الصورة النهائية

```text id="k9w3pq"
                    Backend
                       ↓
                 Response
                       ↓
             Response Interceptor
                ↙           ↘
           return         throw/reject
              ↓                ↓
          Resolved          Rejected
              ↓                ↓
           Caller             catch
```

**الخطوة التالية:** جاوب سؤال الـ Final Interview لـ `4.3`: لو الـ Backend رجع `200` لكن الـ Response Interceptor عمل `throw new Error(...)`، هل الـ caller يدخل `catch`؟ وليه؟
</details>
تمام
ChatGPT
نثبت النقطة دي في **5 دقائق**: ماذا يحدث لو الـ `Response Interceptor` نفسه عمل `throw` أو رجّع `Promise.reject`؟

### 🗺️ Roadmap

```text id="n3x8qa"
SECTION 4: Interceptors 🟢
  ├─ 4.1 What is an Interceptor    ✅
  ├─ 4.2 Request Interceptor       ✅
  ├─ 4.3 Response Interceptor      🟢
  │    ├─ Success handler            ✅
  │    ├─ Error handler              ✅
  │    ├─ Promise.reject             ✅
  │    ├─ Centralized processing     ✅
  │    └─ throw / reject             🟢 ← هنا
  ├─ 4.4 Authentication             ⬜
  ├─ 4.5 401 Handling               ⬜
  └─ 4.6 Token Refresh + Queue      ⬜
```

### عندنا Success

```ts id="j6t2vp"
api.interceptors.response.use(
  (response) => {
    console.log("SUCCESS");

    return response;
  }
);
```

كل شيء طبيعي:

```text id="c9q4mx"
Backend
   ↓
200
   ↓
Interceptor
   ↓
return response
   ↓
Caller
```

---

### طيب لو عملنا `throw`؟

مثلاً:

```ts id="z7k3nb"
api.interceptors.response.use(
  (response) => {
    throw new Error("Something went wrong");
  }
);
```

هنا أنت **أنتجت Error جديد داخل الـ interceptor**.

فالـ flow يصبح:

```text id="r4m8yk"
Backend
   ↓
200
   ↓
Response Interceptor
   ↓
throw Error
   ↓
❌ Rejected Promise
   ↓
catch
```

يعني حتى لو الـ Backend رجع `200`، أنت قررت داخل الـ interceptor إن العملية تفشل.

---

### ونفس الفكرة مع:

```ts id="a8p2wf"
return Promise.reject(error);
```

أنت بتقول:

> لا تكمل كـ Success، كمل كـ Rejected Promise.

---

### الفرق البسيط

```ts id="x2m7kc"
throw new Error("...");
```

أنت **بتنشئ وترمي Error**.

أما:

```ts id="q5v9dz"
return Promise.reject(error);
```

أنت عندك **Error موجود بالفعل** وبتحافظ على الـ Promise كـ rejected.

---

### مثال واقعي

```ts id="u4h8ns"
api.interceptors.response.use(
  (response) => {
    if (response.data.status !== true) {
      throw new Error("API operation failed");
    }

    return response;
  },

  (error) => {
    return Promise.reject(error);
  }
);
```

هنا الـ interceptor ممكن يعمل validation أو processing، ولو قرر إن فيه مشكلة، يـ `throw`.

لكن خد بالك: **مش عايزين ندخل في API validation وauthentication دلوقتي**. الهدف فقط فهم الـ Promise flow.

### الصورة النهائية

```text id="k9w3pq"
                    Backend
                       ↓
                 Response
                       ↓
             Response Interceptor
                ↙           ↘
           return         throw/reject
              ↓                ↓
          Resolved          Rejected
              ↓                ↓
           Caller             catch
```

**الخطوة التالية:** جاوب سؤال الـ Final Interview لـ `4.3`: لو الـ Backend رجع `200` لكن الـ Response Interceptor عمل `throw new Error(...)`، هل الـ caller يدخل `catch`؟ وليه؟
<details>
<summary>(response.data.status !== true</summary>
user
(response.data.status !== true
ده هيبقي بترو كده كده سواء200او 404
ChatGPT
لا، **مش بالضرورة**. هنا لازم نفرّق بين `HTTP status` وبين `response.data.status`، ودي نقطة مهمة جدًا.

لو الـ Backend عنده الـ API shape اللي إحنا بنستخدمه:

```json id="x9q3kd"
{
  "status": true,
  "code": 200,
  "payload": {}
}
```

فـ:

```ts id="m7c2vp"
response.data.status
```

هتكون:

```text
true
```

حتى لو الـ HTTP status هو `200`.

لكن في حالة `404`، **غالبًا Axios أصلًا هيدخل الـ error handler**، ومش هيكمل للـ success handler، وبالتالي الكود:

```ts id="j4k8ws"
response.data.status !== true
```

مش هو اللي هيحدد إن الـ HTTP request فشل.

يعني عندنا مستويين مختلفين:

```text id="q6r2nm"
HTTP Status
    ↓
200 / 404
```

وده خاص بـ **HTTP**.

أما:

```text id="t8v5za"
response.data.status
    ↓
true / false
```

فده **API-level status**، والـ Backend هو اللي بيحدده داخل الـ JSON.

### مثال

**HTTP 200:**

```json id="p2x7kc"
{
  "status": true,
  "code": 200,
  "payload": {}
}
```

هنا:

```text
HTTP status = 200
API status  = true
```

**HTTP 200 لكن العملية فشلت منطقيًا:**

```json id="h5n9qm"
{
  "status": false,
  "code": 1001,
  "message": "Insufficient balance",
  "errors": []
}
```

هنا:

```text
HTTP status = 200
API status  = false
```

وده مثال يوضح ليه الاتنين **مش نفس الحاجة**.

أما:

**HTTP 404:**

```text
HTTP status = 404
```

Axios عادةً يعتبره rejected ويدخله في:

```ts id="v8c4yp"
(error) => {
  // هنا
}
```

مش:

```ts id="k3m7qw"
(response) => {
  // مش هنا عادةً
}
```

فملاحظتك ممتازة لأنها كشفت إننا كنا بنخلط بين **HTTP status** و **API response status**.

**الخطوة التالية:** أشرح لك الفرق بين `response.status` و`response.data.status` و`response.data.code` في 3 أسطر فقط.
</details>
(response.data.status !== true
ده هيبقي بترو كده كده سواء200او 404
ChatGPT
لا، **مش بالضرورة**. هنا لازم نفرّق بين `HTTP status` وبين `response.data.status`، ودي نقطة مهمة جدًا.

لو الـ Backend عنده الـ API shape اللي إحنا بنستخدمه:

```json id="x9q3kd"
{
  "status": true,
  "code": 200,
  "payload": {}
}
```

فـ:

```ts id="m7c2vp"
response.data.status
```

هتكون:

```text
true
```

حتى لو الـ HTTP status هو `200`.

لكن في حالة `404`، **غالبًا Axios أصلًا هيدخل الـ error handler**، ومش هيكمل للـ success handler، وبالتالي الكود:

```ts id="j4k8ws"
response.data.status !== true
```

مش هو اللي هيحدد إن الـ HTTP request فشل.

يعني عندنا مستويين مختلفين:

```text id="q6r2nm"
HTTP Status
    ↓
200 / 404
```

وده خاص بـ **HTTP**.

أما:

```text id="t8v5za"
response.data.status
    ↓
true / false
```

فده **API-level status**، والـ Backend هو اللي بيحدده داخل الـ JSON.

### مثال

**HTTP 200:**

```json id="p2x7kc"
{
  "status": true,
  "code": 200,
  "payload": {}
}
```

هنا:

```text
HTTP status = 200
API status  = true
```

**HTTP 200 لكن العملية فشلت منطقيًا:**

```json id="h5n9qm"
{
  "status": false,
  "code": 1001,
  "message": "Insufficient balance",
  "errors": []
}
```

هنا:

```text
HTTP status = 200
API status  = false
```

وده مثال يوضح ليه الاتنين **مش نفس الحاجة**.

أما:

**HTTP 404:**

```text
HTTP status = 404
```

Axios عادةً يعتبره rejected ويدخله في:

```ts id="v8c4yp"
(error) => {
  // هنا
}
```

مش:

```ts id="k3m7qw"
(response) => {
  // مش هنا عادةً
}
```

فملاحظتك ممتازة لأنها كشفت إننا كنا بنخلط بين **HTTP status** و **API response status**.

**الخطوة التالية:** أشرح لك الفرق بين `response.status` و`response.data.status` و`response.data.code` في 3 أسطر فقط.
<details>
<summary>باختصار اووي لأننا رحلتها من قبل</summary>
user
باختصار اووي لأننا رحلتها من قبل
ChatGPT
- `response.status` → **HTTP Status** من البروتوكول، مثل `200`, `404`, `500`.
- `response.data.status` → **API Status** الـ Backend حاطه داخل الـ JSON، مثل `true` أو `false`.
- `response.data.code` → **API/Business Code** يحدده الـ Backend، مثل `1001` أو `USER_NOT_FOUND`.

**الخطوة التالية:** نكمل الـ Final Interview لـ `4.3`.
</details>
باختصار اووي لأننا رحلتها من قبل
ChatGPT
- `response.status` → **HTTP Status** من البروتوكول، مثل `200`, `404`, `500`.
- `response.data.status` → **API Status** الـ Backend حاطه داخل الـ JSON، مثل `true` أو `false`.
- `response.data.code` → **API/Business Code** يحدده الـ Backend، مثل `1001` أو `USER_NOT_FOUND`.

**الخطوة التالية:** نكمل الـ Final Interview لـ `4.3`.
<details>
<summary>كمل</summary>
user
كمل
ChatGPT
جاوب على **Final Interview لـ 4.3** في دقيقتين:

### 🗺️ Roadmap

```text id="t8n4kq"
SECTION 4: Interceptors 🟢
  ├─ 4.1 What is an Interceptor    ✅
  ├─ 4.2 Request Interceptor       ✅
  ├─ 4.3 Response Interceptor      🟢 ← Final Interview
  ├─ 4.4 Authentication            ⬜
  ├─ 4.5 401 Handling              ⬜
  └─ 4.6 Token Refresh + Queue     ⬜
```

### 🎯 السؤال

عندك:

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

الـ Backend رجّع **404**.

**اشرح الـ flow من لحظة رجوع الـ 404 لحد ما يوصل للـ `catch`، وليه بنستخدم `Promise.reject(error)`؟**

جاوب كأنك في Interview.
</details>
كمل
ChatGPT
جاوب على **Final Interview لـ 4.3** في دقيقتين:

### 🗺️ Roadmap

```text id="t8n4kq"
SECTION 4: Interceptors 🟢
  ├─ 4.1 What is an Interceptor    ✅
  ├─ 4.2 Request Interceptor       ✅
  ├─ 4.3 Response Interceptor      🟢 ← Final Interview
  ├─ 4.4 Authentication            ⬜
  ├─ 4.5 401 Handling              ⬜
  └─ 4.6 Token Refresh + Queue     ⬜
```

### 🎯 السؤال

عندك:

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

الـ Backend رجّع **404**.

**اشرح الـ flow من لحظة رجوع الـ 404 لحد ما يوصل للـ `catch`، وليه بنستخدم `Promise.reject(error)`؟**

جاوب كأنك في Interview.
<details>
<summary>بسم الله الرحمن الرحيم. أول حاجة الـinterceptor هنا هيخش على الـerror، لأن ده 404. اللي هيحصل إن أول حاجة في interceptor هعمل console.log error، بعد كده هعمل return Promise.rejected. لما أعمل Promise.rejected فأنا بقوله رجع الخطأ كما هو. هيرجعه لمين؟ هيرجعه للـcatch؟ هو مش هيرجعه للـcatch. هو هنا هيقوله اتعامل معاه كـPromise مرفوض، وساعتها الـawait لما نيجي نستخدمه بعد كده في الـcaller هيتعامل مع الـPromise.rejected دي إنها إيه؟ هيحولها لـcatch.</summary>
user
بسم الله الرحمن الرحيم. أول حاجة الـinterceptor هنا هيخش على الـerror، لأن ده 404. اللي هيحصل إن أول حاجة في interceptor هعمل console.log error، بعد كده هعمل return Promise.rejected. لما أعمل Promise.rejected فأنا بقوله رجع الخطأ كما هو. هيرجعه لمين؟ هيرجعه للـcatch؟ هو مش هيرجعه للـcatch. هو هنا هيقوله اتعامل معاه كـPromise مرفوض، وساعتها الـawait لما نيجي نستخدمه بعد كده في الـcaller هيتعامل مع الـPromise.rejected دي إنها إيه؟ هيحولها لـcatch.
ChatGPT
إجابتك **10/10**، وأهم حاجة إنك صححت بنفسك النقطة اللي كنا بنتكلم فيها: `Promise.reject` **مش بيرجع الـ error مباشرةً للـ `catch`**.

الـ flow اللي قلته مضبوط:

```text
Backend
   ↓
404
   ↓
Response Interceptor
   ↓
error handler
   ↓
console.log("ERROR")
   ↓
Promise.reject(error)
   ↓
Rejected Promise
   ↓
await
   ↓
catch
```

وتعبيرك:

> "اتعامل معاه كـ Promise مرفوض"

هو التعبير الأدق.

فقط الـ syntax الصحيح هو:

```ts
Promise.reject(error)
```

مش:

```ts
Promise.rejected(error)
```

### 🟢 4.3 مكتملة

```text
SECTION 4: Interceptors
├─ 4.1 What is an Interceptor    ✅
├─ 4.2 Request Interceptor       ✅
└─ 4.3 Response Interceptor      ✅
```

**الخطوة التالية:** نبدأ `4.4 Authentication`، وأول حاجة هنفهم الفرق بين **Authentication وAuthorization** قبل ما نكتب أي كود.
</details>
بسم الله الرحمن الرحيم. أول حاجة الـinterceptor هنا هيخش على الـerror، لأن ده 404. اللي هيحصل إن أول حاجة في interceptor هعمل console.log error، بعد كده هعمل return Promise.rejected. لما أعمل Promise.rejected فأنا بقوله رجع الخطأ كما هو. هيرجعه لمين؟ هيرجعه للـcatch؟ هو مش هيرجعه للـcatch. هو هنا هيقوله اتعامل معاه كـPromise مرفوض، وساعتها الـawait لما نيجي نستخدمه بعد كده في الـcaller هيتعامل مع الـPromise.rejected دي إنها إيه؟ هيحولها لـcatch.
ChatGPT
إجابتك **10/10**، وأهم حاجة إنك صححت بنفسك النقطة اللي كنا بنتكلم فيها: `Promise.reject` **مش بيرجع الـ error مباشرةً للـ `catch`**.

الـ flow اللي قلته مضبوط:

```text
Backend
   ↓
404
   ↓
Response Interceptor
   ↓
error handler
   ↓
console.log("ERROR")
   ↓
Promise.reject(error)
   ↓
Rejected Promise
   ↓
await
   ↓
catch
```

وتعبيرك:

> "اتعامل معاه كـ Promise مرفوض"

هو التعبير الأدق.

فقط الـ syntax الصحيح هو:

```ts
Promise.reject(error)
```

مش:

```ts
Promise.rejected(error)
```

### 🟢 4.3 مكتملة

```text
SECTION 4: Interceptors
├─ 4.1 What is an Interceptor    ✅
├─ 4.2 Request Interceptor       ✅
└─ 4.3 Response Interceptor      ✅
```

**الخطوة التالية:** نبدأ `4.4 Authentication`، وأول حاجة هنفهم الفرق بين **Authentication وAuthorization** قبل ما نكتب أي كود.
<details>
<summary>Authentication4.4</summary>
user
Authentication4.4
ChatGPT
ابدأ `4.4 Authentication` في حوالي **10 دقائق**، وأول مفهوم عندنا هو الفرق بين **Authentication** و **Authorization**.

### 🗺️ Roadmap

```text id="c4n8vy"
SECTION 4: Interceptors 🟢
  ├─ 4.1 What is an Interceptor    ✅
  ├─ 4.2 Request Interceptor       ✅
  ├─ 4.3 Response Interceptor      ✅
  ├─ 4.4 Authentication            🟢 ← هنا
  │    ├─ Authentication vs Authorization 🟢
  │    ├─ Access Token              ⬜
  │    ├─ Refresh Token             ⬜
  │    └─ Token lifecycle           ⬜
  ├─ 4.5 401 Handling              ⬜
  └─ 4.6 Token Refresh + Queue     ⬜
```

## أولًا: Authentication يعني إيه؟

**Authentication = إثبات هويتك.**

يعني النظام بيسأل:

> **Who are you?**

مثلاً تعمل Login:

```text id="v5c2mz"
Email + Password
       ↓
     Backend
       ↓
"أنت Ahmed"
       ↓
Access Token
```

فالـ Login غالبًا هو جزء من **Authentication**.

---

## Authorization يعني إيه؟

**Authorization = إيه المسموح لك تعمله؟**

يعني بعد ما النظام عرف إنك Ahmed، يسأل:

> **What are you allowed to do?**

مثلاً:

```text id="m7q4kp"
Ahmed
 ↓
Role = Admin
 ↓
Allowed:
  ✅ Delete User
  ✅ Create Product
  ✅ View Dashboard
```

لكن User عادي:

```text id="p8x2na"
Mohamed
 ↓
Role = User
 ↓
Allowed:
  ❌ Delete User
  ✅ View Profile
```

### احفظها كده:

```text id="z4k7qm"
Authentication
      ↓
Who are you?
      ↓
Identity


Authorization
      ↓
What can you do?
      ↓
Permissions
```

### وعلاقتها بالـ Axios

إحنا قبل كده كتبنا:

```ts id="k3m8vp"
api.interceptors.request.use((config) => {
  const token = localStorage.getItem("token");

  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }

  return config;
});
```

الـ `token` هنا بيساعد الـ Backend يعرف:

> الطلب ده جاي من مستخدم authenticated مين؟

وبعد كده الـ Backend ممكن يستخدم معلومات الـ token عشان يقرر:

> المستخدم ده **مسموح له** يعمل العملية دي ولا لأ؟

وده هو المدخل اللي هنحتاجه عشان نفهم بعدين `401` و`403`.

**الخطوة التالية:** جاوبني باختصار: لو المستخدم عمل Login بنجاح، لكن حاول يدخل صفحة Admin وهو مش Admin، دي مشكلة **Authentication** ولا **Authorization**؟
</details>
Authentication4.4
ChatGPT
ابدأ `4.4 Authentication` في حوالي **10 دقائق**، وأول مفهوم عندنا هو الفرق بين **Authentication** و **Authorization**.

### 🗺️ Roadmap

```text id="c4n8vy"
SECTION 4: Interceptors 🟢
  ├─ 4.1 What is an Interceptor    ✅
  ├─ 4.2 Request Interceptor       ✅
  ├─ 4.3 Response Interceptor      ✅
  ├─ 4.4 Authentication            🟢 ← هنا
  │    ├─ Authentication vs Authorization 🟢
  │    ├─ Access Token              ⬜
  │    ├─ Refresh Token             ⬜
  │    └─ Token lifecycle           ⬜
  ├─ 4.5 401 Handling              ⬜
  └─ 4.6 Token Refresh + Queue     ⬜
```

## أولًا: Authentication يعني إيه؟

**Authentication = إثبات هويتك.**

يعني النظام بيسأل:

> **Who are you?**

مثلاً تعمل Login:

```text id="v5c2mz"
Email + Password
       ↓
     Backend
       ↓
"أنت Ahmed"
       ↓
Access Token
```

فالـ Login غالبًا هو جزء من **Authentication**.

---

## Authorization يعني إيه؟

**Authorization = إيه المسموح لك تعمله؟**

يعني بعد ما النظام عرف إنك Ahmed، يسأل:

> **What are you allowed to do?**

مثلاً:

```text id="m7q4kp"
Ahmed
 ↓
Role = Admin
 ↓
Allowed:
  ✅ Delete User
  ✅ Create Product
  ✅ View Dashboard
```

لكن User عادي:

```text id="p8x2na"
Mohamed
 ↓
Role = User
 ↓
Allowed:
  ❌ Delete User
  ✅ View Profile
```

### احفظها كده:

```text id="z4k7qm"
Authentication
      ↓
Who are you?
      ↓
Identity


Authorization
      ↓
What can you do?
      ↓
Permissions
```

### وعلاقتها بالـ Axios

إحنا قبل كده كتبنا:

```ts id="k3m8vp"
api.interceptors.request.use((config) => {
  const token = localStorage.getItem("token");

  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }

  return config;
});
```

الـ `token` هنا بيساعد الـ Backend يعرف:

> الطلب ده جاي من مستخدم authenticated مين؟

وبعد كده الـ Backend ممكن يستخدم معلومات الـ token عشان يقرر:

> المستخدم ده **مسموح له** يعمل العملية دي ولا لأ؟

وده هو المدخل اللي هنحتاجه عشان نفهم بعدين `401` و`403`.

**الخطوة التالية:** جاوبني باختصار: لو المستخدم عمل Login بنجاح، لكن حاول يدخل صفحة Admin وهو مش Admin، دي مشكلة **Authentication** ولا **Authorization**؟
<details>
<summary>Authorization</summary>
user
Authorization
ChatGPT
صح **10/10** ✅

لأن النظام بالفعل **عرف المستخدم**، يعني الـ Authentication نجحت، لكن المستخدم **مش عنده Permission** للوصول للـ Admin.

```text
Login
 ↓
Authentication ✅
"Who are you?"
 ↓
Admin page
 ↓
Authorization ❌
"What are you allowed to do?"
```

والـ HTTP status الشائع هنا:

```text
401 → Authentication problem
403 → Authorization problem
```

مش هنوسع في `401/403` دلوقتي، لأن ده جاي في `4.5`.

**الخطوة التالية:** نبدأ **Access Token** ونفهم بالضبط إيه هو وليه الـ Request Interceptor بيحطه في `Authorization` Header.
</details>
Authorization
ChatGPT
صح **10/10** ✅

لأن النظام بالفعل **عرف المستخدم**، يعني الـ Authentication نجحت، لكن المستخدم **مش عنده Permission** للوصول للـ Admin.

```text
Login
 ↓
Authentication ✅
"Who are you?"
 ↓
Admin page
 ↓
Authorization ❌
"What are you allowed to do?"
```

والـ HTTP status الشائع هنا:

```text
401 → Authentication problem
403 → Authorization problem
```

مش هنوسع في `401/403` دلوقتي، لأن ده جاي في `4.5`.

**الخطوة التالية:** نبدأ **Access Token** ونفهم بالضبط إيه هو وليه الـ Request Interceptor بيحطه في `Authorization` Header.
<details>
<summary>Access Token</summary>
user
Access Token
ChatGPT
افهم **Access Token** في حوالي **7 دقائق** من خلال الـ flow ده، وبعدها نربطه مباشرةً بالـ Axios Interceptor.

### 🗺️ Roadmap

```text id="m8q2vx"
SECTION 4: Interceptors 🟢
  ├─ 4.1 What is an Interceptor    ✅
  ├─ 4.2 Request Interceptor       ✅
  ├─ 4.3 Response Interceptor      ✅
  ├─ 4.4 Authentication            🟢
  │    ├─ Authentication vs Authorization ✅
  │    ├─ Access Token              🟢 ← هنا
  │    ├─ Refresh Token             ⬜
  │    └─ Token lifecycle           ⬜
  ├─ 4.5 401 Handling              ⬜
  └─ 4.6 Token Refresh + Queue     ⬜
```

## يعني إيه Access Token؟

بعد ما المستخدم يعمل Login:

```text id="q6k3mx"
Email + Password
       ↓
     Backend
       ↓
Authentication ✅
       ↓
Access Token
```

الـ Backend يقول للـ Frontend تقريبًا:

> "أنا عرفت أنت مين، وده Token تستخدمه في الطلبات القادمة."

مثلاً:

```text id="w9p4cz"
eyJhbGciOiJIUzI1NiIs...
```

الـ Frontend يحتفظ بالـ token، وبعد كده مع كل API request يحتاج Authentication، يبعته للـ Backend.

---

## إزاي بيتبعت؟

في الـ HTTP Header:

```http
Authorization: Bearer <access-token>
```

وعندنا في Axios:

```ts id="t3m7kp"
api.interceptors.request.use((config) => {
  const token = localStorage.getItem("token");

  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }

  return config;
});
```

فالمسار:

```text id="a7v2nm"
User Login
    ↓
Backend
    ↓
Access Token
    ↓
Frontend stores token
    ↓
Request
    ↓
Request Interceptor
    ↓
Authorization: Bearer Token
    ↓
Backend
```

## طيب ليه اسمه Access Token؟

لأنه Token تستخدمه عشان **تطلب Access إلى resources محمية**.

مثلاً:

```text id="k5r8qx"
GET /profile
GET /orders
POST /orders
GET /dashboard
```

الـ Backend يستقبل الـ token ويقدر يتحقق من هوية صاحب الطلب، ثم يقرر هل الطلب مسموح أم لا.

---

### أهم حاجة حاليًا

**Access Token مش هو Password.**

الـ Password تستخدمه غالبًا أثناء Login.

أما الـ Access Token فتستخدمه **بعد Login** مع الـ API requests.

```text id="n4c7wp"
Password
   ↓
Login
   ↓
Access Token
   ↓
API Requests
```

### نقطة مهمة جدًا

الـ Access Token غالبًا له **Expiration**.

يعني مش بالضرورة يفضل صالح للأبد.

وده السبب اللي هيخلينا بعد كده نحتاج مفهوم:

**Refresh Token**

لكن **مش هنشرحه لسه**.

**الخطوة التالية:** جاوبني: ليه ما نبعتش الـ Email والـ Password مع كل API request بدل ما نستخدم Access Token؟
</details>
Access Token
ChatGPT
افهم **Access Token** في حوالي **7 دقائق** من خلال الـ flow ده، وبعدها نربطه مباشرةً بالـ Axios Interceptor.

### 🗺️ Roadmap

```text id="m8q2vx"
SECTION 4: Interceptors 🟢
  ├─ 4.1 What is an Interceptor    ✅
  ├─ 4.2 Request Interceptor       ✅
  ├─ 4.3 Response Interceptor      ✅
  ├─ 4.4 Authentication            🟢
  │    ├─ Authentication vs Authorization ✅
  │    ├─ Access Token              🟢 ← هنا
  │    ├─ Refresh Token             ⬜
  │    └─ Token lifecycle           ⬜
  ├─ 4.5 401 Handling              ⬜
  └─ 4.6 Token Refresh + Queue     ⬜
```

## يعني إيه Access Token؟

بعد ما المستخدم يعمل Login:

```text id="q6k3mx"
Email + Password
       ↓
     Backend
       ↓
Authentication ✅
       ↓
Access Token
```

الـ Backend يقول للـ Frontend تقريبًا:

> "أنا عرفت أنت مين، وده Token تستخدمه في الطلبات القادمة."

مثلاً:

```text id="w9p4cz"
eyJhbGciOiJIUzI1NiIs...
```

الـ Frontend يحتفظ بالـ token، وبعد كده مع كل API request يحتاج Authentication، يبعته للـ Backend.

---

## إزاي بيتبعت؟

في الـ HTTP Header:

```http
Authorization: Bearer <access-token>
```

وعندنا في Axios:

```ts id="t3m7kp"
api.interceptors.request.use((config) => {
  const token = localStorage.getItem("token");

  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }

  return config;
});
```

فالمسار:

```text id="a7v2nm"
User Login
    ↓
Backend
    ↓
Access Token
    ↓
Frontend stores token
    ↓
Request
    ↓
Request Interceptor
    ↓
Authorization: Bearer Token
    ↓
Backend
```

## طيب ليه اسمه Access Token؟

لأنه Token تستخدمه عشان **تطلب Access إلى resources محمية**.

مثلاً:

```text id="k5r8qx"
GET /profile
GET /orders
POST /orders
GET /dashboard
```

الـ Backend يستقبل الـ token ويقدر يتحقق من هوية صاحب الطلب، ثم يقرر هل الطلب مسموح أم لا.

---

### أهم حاجة حاليًا

**Access Token مش هو Password.**

الـ Password تستخدمه غالبًا أثناء Login.

أما الـ Access Token فتستخدمه **بعد Login** مع الـ API requests.

```text id="n4c7wp"
Password
   ↓
Login
   ↓
Access Token
   ↓
API Requests
```

### نقطة مهمة جدًا

الـ Access Token غالبًا له **Expiration**.

يعني مش بالضرورة يفضل صالح للأبد.

وده السبب اللي هيخلينا بعد كده نحتاج مفهوم:

**Refresh Token**

لكن **مش هنشرحه لسه**.

**الخطوة التالية:** جاوبني: ليه ما نبعتش الـ Email والـ Password مع كل API request بدل ما نستخدم Access Token؟
<details>
<summary>لا، الإيميل والباسورد ده إحنا بنعمله أول مرة بس لما بندخلها التطبيق، وبعد كده يبدأ لما نروح الـbackend، يبدأ الـbackend يعرفك التامين، في الحالة دي بيديك حاجة اسمها access token. من خلال الـaccess token ده اللي هو الكود، بعد كده هو بيبقى موجود، أنت بتحفظه، أو إنت بتستوري التوكن ده في الـfrontend عندك. فأنت من خلال الـinterceptor بتاخد أي... تيجي تبعت أي طلب، بتبعت معاه التوكن. في الحالة دي إنت بس بتعمل access token لكل طلب اللي هو متسجل عندك فين؟ متسجل عندك مثلاً في الـlocal storage. في الحالة دي إنت مش محتاج تبعت كل مرة إيميل وباسورد، هو خلاص كده عرفك من خلال التوكن.</summary>
user
لا، الإيميل والباسورد ده إحنا بنعمله أول مرة بس لما بندخلها التطبيق، وبعد كده يبدأ لما نروح الـbackend، يبدأ الـbackend يعرفك التامين، في الحالة دي بيديك حاجة اسمها access token. من خلال الـaccess token ده اللي هو الكود، بعد كده هو بيبقى موجود، أنت بتحفظه، أو إنت بتستوري التوكن ده في الـfrontend عندك. فأنت من خلال الـinterceptor بتاخد أي... تيجي تبعت أي طلب، بتبعت معاه التوكن. في الحالة دي إنت بس بتعمل access token لكل طلب اللي هو متسجل عندك فين؟ متسجل عندك مثلاً في الـlocal storage. في الحالة دي إنت مش محتاج تبعت كل مرة إيميل وباسورد، هو خلاص كده عرفك من خلال التوكن.
ChatGPT
بالضبط، **10/10**. أنت فاهم الـ lifecycle بشكل صحيح.

الـ flow عندك:

```text id="e6q9pw"
Email + Password
       ↓
      Login
       ↓
    Backend
       ↓
Authentication ✅
       ↓
  Access Token
       ↓
Frontend stores token
       ↓
Request
       ↓
Request Interceptor
       ↓
Authorization: Bearer <token>
       ↓
Backend
       ↓
يتحقق من الـ Token
       ↓
يعرف المستخدم
```

وبالتالي **مش بنبعت الـ Email + Password في كل Request**.

النقطة الوحيدة اللي أصححها في كلامك:

> "بتعمل Access Token لكل طلب"

لا، إحنا **مش بنعمل Access Token جديد لكل طلب**.

إحنا بنستخدم **نفس الـ Access Token الحالي** مع الطلبات، لحد ما ينتهي صلاحيته أو يتم إبطاله.

```text id="p2m7xk"
Access Token واحد
      ↓
GET /profile      → يستخدمه
GET /orders       → يستخدمه
POST /orders      → يستخدمه
GET /dashboard    → يستخدمه
```

ولما الـ Access Token ينتهي، هنا ندخل في مفهوم جديد اسمه **Refresh Token**، وده بالضبط اللي هنشرحه بعد كده.

**الخطوة التالية:** ننتقل إلى `Refresh Token` ونفهم ليه محتاجينه أصلًا رغم إن عندنا `Access Token`.
</details>
لا، الإيميل والباسورد ده إحنا بنعمله أول مرة بس لما بندخلها التطبيق، وبعد كده يبدأ لما نروح الـbackend، يبدأ الـbackend يعرفك التامين، في الحالة دي بيديك حاجة اسمها access token. من خلال الـaccess token ده اللي هو الكود، بعد كده هو بيبقى موجود، أنت بتحفظه، أو إنت بتستوري التوكن ده في الـfrontend عندك. فأنت من خلال الـinterceptor بتاخد أي... تيجي تبعت أي طلب، بتبعت معاه التوكن. في الحالة دي إنت بس بتعمل access token لكل طلب اللي هو متسجل عندك فين؟ متسجل عندك مثلاً في الـlocal storage. في الحالة دي إنت مش محتاج تبعت كل مرة إيميل وباسورد، هو خلاص كده عرفك من خلال التوكن.
ChatGPT
بالضبط، **10/10**. أنت فاهم الـ lifecycle بشكل صحيح.

الـ flow عندك:

```text id="e6q9pw"
Email + Password
       ↓
      Login
       ↓
    Backend
       ↓
Authentication ✅
       ↓
  Access Token
       ↓
Frontend stores token
       ↓
Request
       ↓
Request Interceptor
       ↓
Authorization: Bearer <token>
       ↓
Backend
       ↓
يتحقق من الـ Token
       ↓
يعرف المستخدم
```

وبالتالي **مش بنبعت الـ Email + Password في كل Request**.

النقطة الوحيدة اللي أصححها في كلامك:

> "بتعمل Access Token لكل طلب"

لا، إحنا **مش بنعمل Access Token جديد لكل طلب**.

إحنا بنستخدم **نفس الـ Access Token الحالي** مع الطلبات، لحد ما ينتهي صلاحيته أو يتم إبطاله.

```text id="p2m7xk"
Access Token واحد
      ↓
GET /profile      → يستخدمه
GET /orders       → يستخدمه
POST /orders      → يستخدمه
GET /dashboard    → يستخدمه
```

ولما الـ Access Token ينتهي، هنا ندخل في مفهوم جديد اسمه **Refresh Token**، وده بالضبط اللي هنشرحه بعد كده.

**الخطوة التالية:** ننتقل إلى `Refresh Token` ونفهم ليه محتاجينه أصلًا رغم إن عندنا `Access Token`.
<details>
<summary>Refresh Token</summary>
user
Refresh Token
ChatGPT
افهم **Refresh Token** في حوالي **8 دقائق** من خلال المشكلة اللي بيحلها، لأن ده أهم من حفظ تعريفه.

### 🗺️ Roadmap

```text id="m4k8pz"
SECTION 4: Interceptors 🟢
  ├─ 4.1 What is an Interceptor    ✅
  ├─ 4.2 Request Interceptor       ✅
  ├─ 4.3 Response Interceptor      ✅
  ├─ 4.4 Authentication            🟢
  │    ├─ Authentication vs Authorization ✅
  │    ├─ Access Token              ✅
  │    ├─ Refresh Token             🟢 ← هنا
  │    └─ Token lifecycle           ⬜
  ├─ 4.5 401 Handling              ⬜
  └─ 4.6 Token Refresh + Queue     ⬜
```

## المشكلة

إحنا قلنا إن الـ **Access Token** له expiration.

مثلاً:

```text id="g8m2qx"
Access Token
     ↓
صالح لمدة 15 دقيقة
```

المستخدم دخل التطبيق الساعة 10:00.

الـ token صالح لحد:

```text
10:15
```

المستخدم الساعة 10:20 عمل:

```text
GET /profile
```

الـ Backend يقول:

```text
401 Unauthorized
Token expired
```

طيب هل هنقول للمستخدم:

> "اعمل Login تاني واكتب Email + Password"؟

**لا.**

وهنا يأتي دور الـ **Refresh Token**.

---
</details>
Refresh Token
ChatGPT
افهم **Refresh Token** في حوالي **8 دقائق** من خلال المشكلة اللي بيحلها، لأن ده أهم من حفظ تعريفه.

### 🗺️ Roadmap

```text id="m4k8pz"
SECTION 4: Interceptors 🟢
  ├─ 4.1 What is an Interceptor    ✅
  ├─ 4.2 Request Interceptor       ✅
  ├─ 4.3 Response Interceptor      ✅
  ├─ 4.4 Authentication            🟢
  │    ├─ Authentication vs Authorization ✅
  │    ├─ Access Token              ✅
  │    ├─ Refresh Token             🟢 ← هنا
  │    └─ Token lifecycle           ⬜
  ├─ 4.5 401 Handling              ⬜
  └─ 4.6 Token Refresh + Queue     ⬜
```

## المشكلة

إحنا قلنا إن الـ **Access Token** له expiration.

مثلاً:

```text id="g8m2qx"
Access Token
     ↓
صالح لمدة 15 دقيقة
```

المستخدم دخل التطبيق الساعة 10:00.

الـ token صالح لحد:

```text
10:15
```

المستخدم الساعة 10:20 عمل:

```text
GET /profile
```

الـ Backend يقول:

```text
401 Unauthorized
Token expired
```

طيب هل هنقول للمستخدم:

> "اعمل Login تاني واكتب Email + Password"؟

**لا.**

وهنا يأتي دور الـ **Refresh Token**.

---