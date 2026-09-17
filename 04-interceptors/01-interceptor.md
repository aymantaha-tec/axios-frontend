# 4.1 What is an Axios Interceptor؟

أول حاجة لازم تثبت عندك:

> **Interceptor هو نقطة تدخل في رحلة الـRequest أو الـResponse، تسمح لك تفحص أو تعدل البيانات قبل ما تكمل رحلتها.**

خلينا نشوف الرحلة العادية.

```text id="4g8p2c"
Component
    ↓
axios.get()
    ↓
Backend
    ↓
Response
    ↓
Component
```

لكن باستخدام Interceptors:

```text id="f5q3y8"
Component
    ↓
axios.get()
    ↓
Request Interceptor
    ↓
Backend
    ↓
Response Interceptor
    ↓
Component
```

إذن عندنا نوعين رئيسيين:

```text id="6v8n1a"
Request Interceptor
Response Interceptor
```

---

# 1. Request Interceptor

بيشتغل **قبل إرسال الـRequest إلى الـServer**.

مثلاً:

```text id="m3z7qk"
Component
    ↓
axios.get()
    ↓
Request Interceptor
    ↓
Server
```

ليه نحتاجه؟

مثلاً عندك:

```text id="9x2p4v"
Authorization: Bearer TOKEN
```

ومش عايز تكتب الـHeader ده في كل Request.

بدل:

```ts id="k7c2w9"
axios.get("/users", {
  headers: {
    Authorization: `Bearer ${token}`,
  },
});
```

وبعدين:

```ts id="p8n4q2"
axios.get("/products", {
  headers: {
    Authorization: `Bearer ${token}`,
  },
});
```

وبعدين:

```ts id="j4m6s8"
axios.get("/orders", {
  headers: {
    Authorization: `Bearer ${token}`,
  },
});
```

ده duplication.

ممكن الـRequest Interceptor يتدخل:

```text id="s3k8x1"
أي Request
    ↓
Request Interceptor
    ↓
أضف Authorization
    ↓
Server
```

وبالتالي:

```ts id="v5n2r7"
axios.get("/users");
```

والـInterceptor هو اللي يتولى إضافة الـtoken.

---

# 2. Response Interceptor

بيشتغل **بعد ما الـServer يرد وقبل ما الـResponse يوصل للكود اللي استدعاه**.

الرحلة:

```text id="a7c3m9"
Server
   ↓
Response
   ↓
Response Interceptor
   ↓
Component / API Layer
```

مثلاً السيرفر رجع:

```text id="q2w6e1"
401 Unauthorized
```

بدل ما كل API function تعمل:

```ts id="n5k3p8"
if (response.status === 401) {
   ...
}
```

ممكن الـResponse Interceptor يتعامل مع الـ401 مركزيًا.

وده هنوصله بالتفصيل بعدين.

---

# 3. تشبيه مهم

تخيل المطار:

```text id="z8r4m2"
Passenger
   ↓
Security Check
   ↓
Plane
```

الـSecurity Check بيتعمل لكل Passenger.

نفس الفكرة:

```text id="u6p2x9"
Request
   ↓
Request Interceptor
   ↓
Server
```

والعودة:

```text id="h3q7w5"
Server
   ↓
Response Interceptor
   ↓
Application
```

الـInterceptor كأنه **Checkpoint** في الطريق.

---

# 4. أهم حاجة: هو مش API Layer

دي نقطة مهمة جدًا.

عندنا:

```text id="v2m8k4"
API Layer
    ↓
"إيه الـendpoint اللي أستدعيه؟"
```

مثلاً:

```ts id="q5x7n2"
getUser(10)
```

بينما:

```text id="r9k3c6"
Interceptor
    ↓
"ماذا أفعل بكل Request/Response؟"
```

مثلاً:

```text id="m4w8s1"
كل Request
    ↓
أضف Authorization
```

إذن:

```text id="z1y5p7"
API Layer
= specific API operation

Interceptor
= cross-cutting behavior
```

ودي جملة مهمة جدًا.

---

# 5. يعني إيه Cross-Cutting Behavior؟

معناها حاجة بتحتاجها **أماكن كثيرة مختلفة** في التطبيق.

مثلاً:

```text id="6p3x9m"
Authentication Header
Logging
Error Handling
Request ID
Retry
401 Handling
```

لو كتبتها في كل API function:

```text id="x7n2k4"
getUser()
getProducts()
getOrders()
getProfile()
...
```

هتكرر نفس المنطق.

Interceptor يسمح لنا نحطه في مكان مركزي.

---

# 6. الشكل البرمجي

Axios يوفر:

```ts id="m9k4x2"
axios.interceptors.request.use(...)
```

للـRequest.

و:

```ts id="c7p5z1"
axios.interceptors.response.use(...)
```

للـResponse.

مثال بسيط جدًا:

```ts id="q8v3n6"
axios.interceptors.request.use((config) => {
  console.log("Request is going out");

  return config;
});
```

لاحظ:

```ts id="r4m7k9"
return config;
```

ليه؟

لأننا استلمنا الـRequest Configuration، وعملنا عليه processing، وبعدها لازم نسمح له يكمل الرحلة.

```text id="e2x6b8"
config
  ↓
interceptor
  ↓
return config
  ↓
server
```

---

# 7. Response Interceptor

نفس الفكرة:

```ts id="w5k9q3"
axios.interceptors.response.use(
  (response) => {
    console.log("Response received");

    return response;
  }
);
```

الـResponse وصل.

نعمل processing.

ثم:

```ts id="j8m2v5"
return response;
```

فيكمل طريقه للكود المستدعي.

---

# 8. ⚠️ نقطة مهمة جدًا

الـInterceptor **مش لازم يعدل الحاجة**.

ممكن فقط:

```ts id="d6q1s8"
console.log(...)
return config;
```

أو:

```ts id="n4x7p2"
console.log(response.status);
return response;
```

وممكن يعدل:

```ts id="k3w8m5"
config.headers.Authorization = `Bearer ${token}`;
return config;
```

إذن:

> Interceptor يسمح لك **بالتدخل**، وليس معناه بالضرورة أنك لازم تعدل.

---

# 🧠 الخلاصة

احفظ الرحلة:

```text id="r7p2m4"
REQUEST

Component
   ↓
axios
   ↓
Request Interceptor
   ↓
Server
```

ثم:

```text id="q9k5x1"
RESPONSE

Server
   ↓
Response Interceptor
   ↓
Component / API Layer
```

وأهم فرق:

```text id="t3v8n6"
API Layer
→ ماذا أطلب؟

Interceptor
→ ماذا أفعل بكل Request/Response أثناء الرحلة؟
```

---

## 🎯 سؤال Interview

**أنت عندك 50 API calls في المشروع، وكل Request محتاج `Authorization` Header.**

هل الأفضل إنك تكتب:

```ts id="s6k2p9"
headers: {
  Authorization: `Bearer ${token}`
}
```

داخل كل API call؟

**ولا تستخدم Request Interceptor؟**

وقل لي **ليه**.

**الخطوة التالية: جاوب السؤال ده، وبعدها ندخل `4.2 Request Interceptor` ونكتب أول Interceptor حقيقي.**
<details>
<summary>لا، في الحالة دي يا شوشة هنستخدم اللي هو request interceptor.use عشان خاطر نقدر نستفيد منه في جميع الـcomponents، حتى لو حابين نغيره فيما بعد هنعدل التوكن بس في ناحية واحدة. وده السبب. وكمان عشان ما نكررش كود كتير جدًا.</summary>
user
لا، في الحالة دي يا شوشة هنستخدم اللي هو request interceptor.use عشان خاطر نقدر نستفيد منه في جميع الـcomponents، حتى لو حابين نغيره فيما بعد هنعدل التوكن بس في ناحية واحدة. وده السبب. وكمان عشان ما نكررش كود كتير جدًا.
ChatGPT
**إجابتك ممتازة جدًا، 10/10، وفعلاً أنت فاهم سبب وجود الـRequest Interceptor مش مجرد حافظ الـsyntax.**

أنت ذكرت أهم سببين:

### 1. منع تكرار الكود

بدل:

```ts
axios.get("/users", {
  headers: {
    Authorization: `Bearer ${token}`,
  },
});
```

وفي كل endpoint نكرر نفس الحاجة.

نضعها في مكان مركزي:

```text id="b7v2xk"
Request
   ↓
Request Interceptor
   ↓
إضافة Authorization
   ↓
Server
```

### 2. التعديل من مكان واحد

لو طريقة الـAuthentication تغيرت:

```text id="f5k8q2"
Bearer token
```

إلى طريقة أخرى، مش محتاج تلف على عشرات الـAPI calls.

تعدل الـInterceptor.

وده يحقق:

```text id="p3m7w9"
Centralization
+
DRY
+
Low Coupling
+
Maintainability
```

---
</details>
لا، في الحالة دي يا شوشة هنستخدم اللي هو request interceptor.use عشان خاطر نقدر نستفيد منه في جميع الـcomponents، حتى لو حابين نغيره فيما بعد هنعدل التوكن بس في ناحية واحدة. وده السبب. وكمان عشان ما نكررش كود كتير جدًا.
ChatGPT
**إجابتك ممتازة جدًا، 10/10، وفعلاً أنت فاهم سبب وجود الـRequest Interceptor مش مجرد حافظ الـsyntax.**

أنت ذكرت أهم سببين:

### 1. منع تكرار الكود

بدل:

```ts
axios.get("/users", {
  headers: {
    Authorization: `Bearer ${token}`,
  },
});
```

وفي كل endpoint نكرر نفس الحاجة.

نضعها في مكان مركزي:

```text id="b7v2xk"
Request
   ↓
Request Interceptor
   ↓
إضافة Authorization
   ↓
Server
```

### 2. التعديل من مكان واحد

لو طريقة الـAuthentication تغيرت:

```text id="f5k8q2"
Bearer token
```

إلى طريقة أخرى، مش محتاج تلف على عشرات الـAPI calls.

تعدل الـInterceptor.

وده يحقق:

```text id="p3m7w9"
Centralization
+
DRY
+
Low Coupling
+
Maintainability
```

---