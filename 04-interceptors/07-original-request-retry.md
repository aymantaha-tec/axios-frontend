## 1. المشكلة

عندنا:

```text
Access Token
ACCESS_abc123
```

انتهى.

نعمل Request:

```text
GET /users/me
Authorization: Bearer ACCESS_abc123
```

الـ Backend:

```text
401 Unauthorized ❌
```

بدل ما نقول للمستخدم:

> Login مرة تانية

هنستخدم:

```text
Refresh Token
REFRESH_xyz789
```

عشان نجيب:

```text
New Access Token
ACCESS_new456
```

---

## 2. شكل الـ Refresh Request

الـ Frontend يبعت:

```ts
const response = await api.post("/auth/refresh", {
  refreshToken,
});
```

والـ Backend يرجع:

```json
{
  "accessToken": "ACCESS_new456"
}
```

إذن:

```text
Refresh Token
      ↓
POST /auth/refresh
      ↓
Backend
      ↓
New Access Token
```

---

## 3. نحطه داخل Response Interceptor

```ts
api.interceptors.response.use(
  (response) => {
    return response;
  },

  async (error) => {
    if (error.response?.status === 401) {
      const refreshToken = localStorage.getItem("refreshToken");

      if (refreshToken) {
        const response = await api.post("/auth/refresh", {
          refreshToken,
        });

        const newAccessToken = response.data.accessToken;

        localStorage.setItem(
          "accessToken",
          newAccessToken
        );
      }
    }

    return Promise.reject(error);
  }
);
```

ركز في التسلسل:

```text
401
 ↓
جيب Refresh Token
 ↓
POST /auth/refresh
 ↓
Backend
 ↓
New Access Token
 ↓
خزن الـ Access Token الجديد
```

**لكن لسه عندنا مشكلة مهمة جدًا:** الطلب الأصلي الذي رجع `401` **لم نعيده**.

يعني:

```text
GET /users/me
      ↓
401
      ↓
Refresh
      ↓
New Access Token
```

وبس.

الطلب الأصلي انتهى.

في `Token Refresh` الكامل لازم نعمل:

```text
401
 ↓
Refresh Token
 ↓
New Access Token
 ↓
Retry Original Request
 ↓
200 ✅
```

وده هنوصله في الخطوة التالية.

**الخطوة التالية:** قبل ما نكتب `Retry`، جاوبني: بعد ما نطلع `New Access Token`، ليه لازم نعيد الـ Request الأصلي؟
<details>
<summary>أنا عايز request الأصلي عشان نرجع من الـstatus ميتين عشان هيبقى أربعمية واحد بسبب التوكن.</summary>
user
أنا عايز request الأصلي عشان نرجع من الـstatus ميتين عشان هيبقى أربعمية واحد بسبب التوكن.
ChatGPT
بالضبط، **فهمك صحيح 100%**. المقصود هنا `401` وليس `400`.

الفكرة:

```text
Original Request
GET /users/me
       ↓
Access Token قديم ❌
       ↓
401
       ↓
Refresh Token
       ↓
New Access Token ✅
       ↓
نرجع نعمل Original Request مرة تانية
       ↓
GET /users/me
Authorization: Bearer NEW_TOKEN
       ↓
200 ✅
```

يعني إحنا **مش بنعيد الطلب عبثًا**.

إحنا بنعيده لأن **السبب الوحيد لفشله كان الـ Access Token القديم**.

وده بالضبط معنى:

**Retry Original Request**

مثال:

```ts
const newAccessToken = response.data.accessToken;

// نضع الـ token الجديد في الطلب الأصلي
error.config.headers.Authorization =
  `Bearer ${newAccessToken}`;

// نعيد الطلب الأصلي
return api(error.config);
```

لاحظ:

```ts
error.config
```

ده مهم جدًا لأنه بيمثل **Configuration بتاع الـ Request الأصلي** اللي فشل.

فإحنا بنقول:

> خد نفس الـ Request اللي فشل، لكن استخدم الـ Access Token الجديد.

**الخطوة التالية:** نفكك `error.config` واحدة واحدة ونفهم إزاي Axios محتفظ بالـ Original Request عشان نقدر نعمل `Retry`.
</details>
أنا عايز request الأصلي عشان نرجع من الـstatus ميتين عشان هيبقى أربعمية واحد بسبب التوكن.
ChatGPT
بالضبط، **فهمك صحيح 100%**. المقصود هنا `401` وليس `400`.

الفكرة:

```text
Original Request
GET /users/me
       ↓
Access Token قديم ❌
       ↓
401
       ↓
Refresh Token
       ↓
New Access Token ✅
       ↓
نرجع نعمل Original Request مرة تانية
       ↓
GET /users/me
Authorization: Bearer NEW_TOKEN
       ↓
200 ✅
```

يعني إحنا **مش بنعيد الطلب عبثًا**.

إحنا بنعيده لأن **السبب الوحيد لفشله كان الـ Access Token القديم**.

وده بالضبط معنى:

**Retry Original Request**

مثال:

```ts
const newAccessToken = response.data.accessToken;

// نضع الـ token الجديد في الطلب الأصلي
error.config.headers.Authorization =
  `Bearer ${newAccessToken}`;

// نعيد الطلب الأصلي
return api(error.config);
```

لاحظ:

```ts
error.config
```

ده مهم جدًا لأنه بيمثل **Configuration بتاع الـ Request الأصلي** اللي فشل.

فإحنا بنقول:

> خد نفس الـ Request اللي فشل، لكن استخدم الـ Access Token الجديد.

**الخطوة التالية:** نفكك `error.config` واحدة واحدة ونفهم إزاي Axios محتفظ بالـ Original Request عشان نقدر نعمل `Retry`.
<details>
<summary>كمل</summary>
user
كمل
ChatGPT
كمل معايا **`error.config` ثم `Retry` في 10 دقائق**، لأن دي القطعة اللي بتوصلنا فعليًا من `401` إلى `200`.

### 🗺️ Roadmap

```text
SECTION 4: Interceptors 🟢
├─ 4.1 What is Interceptor       ✅
├─ 4.2 Request Interceptor       ✅
├─ 4.3 Response Interceptor      ✅
├─ 4.4 Authentication            ✅
├─ 4.5 401 Handling              ✅
└─ 4.6 Token Refresh + Queue     🟡
     ├─ Token Refresh             ✅
     ├─ error.config              🟡 ← هنا
     └─ Retry                     ⬜
```

## 1. إيه هو `error.config`؟

لما Axios يعمل Request:

```ts
api.get("/users/me");
```

Axios بيكوّن Configuration للطلب تقريبًا كده:

```ts
{
  method: "get",
  url: "/users/me",
  headers: {
    Authorization: "Bearer OLD_TOKEN"
  }
}
```

لو الـ Backend رجع `401`، الـ AxiosError بيحتفظ بالـ Config ده:

```ts
error.config
```

يعني:

```text
error
 └── config
      ├── url
      ├── method
      ├── headers
      └── ...
```

فـ `error.config` = **معلومات الـ Request اللي فشل**.

---

## 2. ليه ده مهم؟

لأننا عايزين نعمل:

```text
نفس الطلب
+
Token جديد
```

مثلاً الطلب الأصلي:

```text
GET /users/me
Authorization: Bearer OLD_TOKEN
```

بعد الـ Refresh:

```text
GET /users/me
Authorization: Bearer NEW_TOKEN
```

إحنا مش محتاجين نكتب:

```ts
api.get("/users/me");
```

مرة تانية.

Axios بالفعل محتفظ بالطلب في:

```ts
error.config
```

---

## 3. نغير الـ Token في الطلب الأصلي

بعد ما الـ Backend يدينا:

```ts
const newAccessToken = response.data.accessToken;
```

نحط الـ Token الجديد في الـ Request الأصلي:

```ts
error.config.headers.Authorization =
  `Bearer ${newAccessToken}`;
```

دلوقتي:

```text
error.config

قبل:
Authorization: Bearer OLD_TOKEN

بعد:
Authorization: Bearer NEW_TOKEN
```

---

## 4. نعمل Retry

هنا السطر المهم:

```ts
return api(error.config);
```

معناه:

> يا Axios، خد الـ Configuration بتاع الـ Request اللي فشل، ونفّذه تاني.

فتبقى الرحلة:

```text
GET /users/me
      ↓
OLD_TOKEN
      ↓
401 ❌
      ↓
error.config
      ↓
Refresh Token
      ↓
NEW_TOKEN
      ↓
تحديث Authorization
      ↓
api(error.config)
      ↓
GET /users/me
      ↓
NEW_TOKEN
      ↓
200 ✅
```

## 5. الشكل المبسط الكامل

```ts
api.interceptors.response.use(
  (response) => {
    return response;
  },

  async (error) => {
    if (error.response?.status === 401) {
      const refreshToken =
        localStorage.getItem("refreshToken");

      if (refreshToken) {
        const response = await api.post("/auth/refresh", {
          refreshToken,
        });

        const newAccessToken =
          response.data.accessToken;

        localStorage.setItem(
          "accessToken",
          newAccessToken
        );

        error.config.headers.Authorization =
          `Bearer ${newAccessToken}`;

        return api(error.config);
      }
    }

    return Promise.reject(error);
  }
);
```

### ⚠️ لكن عندنا مشكلة خطيرة هنا

تخيل إن:

```text
Access Token expired
       ↓
401
       ↓
Refresh
       ↓
Refresh نفسه رجع 401
```

الـ Response Interceptor ممكن يمسك الـ `401` بتاع الـ Refresh نفسه ويحاول يعمل Refresh تاني!

فتدخل في:

```text
401
 ↓
Refresh
 ↓
401
 ↓
Refresh
 ↓
401
 ↓
Refresh
 ↓
...
```

وده اسمه **Infinite Loop**.

وعشان كده بنحتاج حماية، وبعدها ندخل على الجزء الأهم:

**Queue** لما يكون عندك 10 Requests حصل لهم `401` في نفس اللحظة.

**الخطوة التالية:** نفهم الأول إزاي نمنع `Infinite Refresh Loop` باستخدام `skipAuthRefresh` أو flag مشابه، وبعدها ندخل الـ `Queue`.
</details>
كمل
ChatGPT
كمل معايا **`error.config` ثم `Retry` في 10 دقائق**، لأن دي القطعة اللي بتوصلنا فعليًا من `401` إلى `200`.

### 🗺️ Roadmap

```text
SECTION 4: Interceptors 🟢
├─ 4.1 What is Interceptor       ✅
├─ 4.2 Request Interceptor       ✅
├─ 4.3 Response Interceptor      ✅
├─ 4.4 Authentication            ✅
├─ 4.5 401 Handling              ✅
└─ 4.6 Token Refresh + Queue     🟡
     ├─ Token Refresh             ✅
     ├─ error.config              🟡 ← هنا
     └─ Retry                     ⬜
```

## 1. إيه هو `error.config`؟

لما Axios يعمل Request:

```ts
api.get("/users/me");
```

Axios بيكوّن Configuration للطلب تقريبًا كده:

```ts
{
  method: "get",
  url: "/users/me",
  headers: {
    Authorization: "Bearer OLD_TOKEN"
  }
}
```

لو الـ Backend رجع `401`، الـ AxiosError بيحتفظ بالـ Config ده:

```ts
error.config
```

يعني:

```text
error
 └── config
      ├── url
      ├── method
      ├── headers
      └── ...
```

فـ `error.config` = **معلومات الـ Request اللي فشل**.

---

## 2. ليه ده مهم؟

لأننا عايزين نعمل:

```text
نفس الطلب
+
Token جديد
```

مثلاً الطلب الأصلي:

```text
GET /users/me
Authorization: Bearer OLD_TOKEN
```

بعد الـ Refresh:

```text
GET /users/me
Authorization: Bearer NEW_TOKEN
```

إحنا مش محتاجين نكتب:

```ts
api.get("/users/me");
```

مرة تانية.

Axios بالفعل محتفظ بالطلب في:

```ts
error.config
```

---

## 3. نغير الـ Token في الطلب الأصلي

بعد ما الـ Backend يدينا:

```ts
const newAccessToken = response.data.accessToken;
```

نحط الـ Token الجديد في الـ Request الأصلي:

```ts
error.config.headers.Authorization =
  `Bearer ${newAccessToken}`;
```

دلوقتي:

```text
error.config

قبل:
Authorization: Bearer OLD_TOKEN

بعد:
Authorization: Bearer NEW_TOKEN
```

---

## 4. نعمل Retry

هنا السطر المهم:

```ts
return api(error.config);
```

معناه:

> يا Axios، خد الـ Configuration بتاع الـ Request اللي فشل، ونفّذه تاني.

فتبقى الرحلة:

```text
GET /users/me
      ↓
OLD_TOKEN
      ↓
401 ❌
      ↓
error.config
      ↓
Refresh Token
      ↓
NEW_TOKEN
      ↓
تحديث Authorization
      ↓
api(error.config)
      ↓
GET /users/me
      ↓
NEW_TOKEN
      ↓
200 ✅
```

## 5. الشكل المبسط الكامل

```ts
api.interceptors.response.use(
  (response) => {
    return response;
  },

  async (error) => {
    if (error.response?.status === 401) {
      const refreshToken =
        localStorage.getItem("refreshToken");

      if (refreshToken) {
        const response = await api.post("/auth/refresh", {
          refreshToken,
        });

        const newAccessToken =
          response.data.accessToken;

        localStorage.setItem(
          "accessToken",
          newAccessToken
        );

        error.config.headers.Authorization =
          `Bearer ${newAccessToken}`;

        return api(error.config);
      }
    }

    return Promise.reject(error);
  }
);
```

### ⚠️ لكن عندنا مشكلة خطيرة هنا

تخيل إن:

```text
Access Token expired
       ↓
401
       ↓
Refresh
       ↓
Refresh نفسه رجع 401
```

الـ Response Interceptor ممكن يمسك الـ `401` بتاع الـ Refresh نفسه ويحاول يعمل Refresh تاني!

فتدخل في:

```text
401
 ↓
Refresh
 ↓
401
 ↓
Refresh
 ↓
401
 ↓
Refresh
 ↓
...
```

وده اسمه **Infinite Loop**.

وعشان كده بنحتاج حماية، وبعدها ندخل على الجزء الأهم:

**Queue** لما يكون عندك 10 Requests حصل لهم `401` في نفس اللحظة.

**الخطوة التالية:** نفهم الأول إزاي نمنع `Infinite Refresh Loop` باستخدام `skipAuthRefresh` أو flag مشابه، وبعدها ندخل الـ `Queue`.
<details>
<summary>كمل</summary>
user
كمل
ChatGPT
نكمل **منع الـ Infinite Refresh Loop في 10 دقائق**، وبعدها مباشرة ندخل على الـ Queue.

### 🗺️ Roadmap

```text id="x8c3r1"
SECTION 4: Interceptors 🟢
├─ 4.1 What is Interceptor       ✅
├─ 4.2 Request Interceptor       ✅
├─ 4.3 Response Interceptor      ✅
├─ 4.4 Authentication            ✅
├─ 4.5 401 Handling              ✅
└─ 4.6 Token Refresh + Queue     🟡
     ├─ Token Refresh             ✅
     ├─ error.config              ✅
     ├─ Retry                     ✅
     ├─ Infinite Loop Protection   🟡 ← هنا
     └─ Queue                     ⬜
```

## 1. المشكلة

إحنا عندنا:

```text id="2gjy0d"
GET /users
      ↓
401
      ↓
Refresh
      ↓
POST /auth/refresh
      ↓
401
```

المشكلة إن `POST /auth/refresh` نفسه مرّ على الـ `Response Interceptor`.

فالـ Interceptor يشوف:

```ts
error.response?.status === 401
```

ويقول:

> أعمل Refresh!

فتبقى:

```text id="x8x8v3"
401
 ↓
Refresh
 ↓
401
 ↓
Refresh
 ↓
401
 ↓
Refresh
 ↓
∞
```

وده **Infinite Loop**.

---

# 2. الحل

نقول للـ Interceptor:

> لو الطلب الحالي هو نفسه طلب الـ Refresh، متحاولش تعمل Refresh مرة تانية.

ممكن نضيف Flag على الـ Config:

```ts
skipAuthRefresh: true
```

مثلاً:

```ts id="b5g1p9"
api.post(
  "/auth/refresh",
  { refreshToken },
  {
    skipAuthRefresh: true
  }
);
```

الفكرة:

```text id="q7w3y2"
Request عادي
      ↓
401
      ↓
Refresh Token
      ↓
Request /auth/refresh
      ↓
skipAuthRefresh = true
      ↓
لو حصل 401
      ↓
❌ لا تعمل Refresh مرة أخرى
```

---

## 3. لكن TypeScript عنده مشكلة

Axios مش عارف إن:

```ts
skipAuthRefresh
```

خاصية موجودة في الـ Config.

فلو كتبت:

```ts
api.post("/auth/refresh", data, {
  skipAuthRefresh: true
});
```

TypeScript غالبًا هيقول:

```text
Property 'skipAuthRefresh' does not exist...
```

وده طبيعي.

إحنا أضفنا **Custom Property** مش موجودة في Axios أصلاً.

لكن مش عايزك تدخل دلوقتي في `Module Augmentation` وتفاصيل TypeScript الخاصة بتوسيع Axios.

خلينا نفهم الـ Concept الأول.

---

# 4. طريقة أبسط للتعلم

ممكن نستخدم Flag على الـ Request Config:

```ts
const originalRequest = error.config;
```

وبعدين:

```ts
if (originalRequest._retry) {
  return Promise.reject(error);
}

originalRequest._retry = true;
```

المعنى:

```text id="d1c6x9"
أول مرة 401
    ↓
_retry = false / غير موجود
    ↓
نعمل Refresh
    ↓
_retry = true
    ↓
Retry
```

لو حصل `401` مرة تانية:

```text id="z4u2pk"
_retry = true
    ↓
❌ متعملش Refresh تاني
    ↓
Promise.reject(error)
```

---

# 5. الشكل الذهني

احفظها كده:

```text id="c0v3qy"
Original Request
      ↓
401
      ↓
هل عملنا Refresh قبل كده؟
      │
   ┌──┴──┐
   │     │
  لا     نعم
   │     │
   ↓     ↓
Refresh  Reject
   │
   ↓
New Token
   │
   ↓
Retry
```

وده بيمنع:

```text
401 → Refresh → 401 → Refresh → 401 → Refresh...
```

ويخليها:

```text
401
 ↓
Refresh
 ↓
New Token
 ↓
Retry
 ↓
401 مرة أخرى
 ↓
Reject ❌
```
</details>
كمل
ChatGPT
نكمل **منع الـ Infinite Refresh Loop في 10 دقائق**، وبعدها مباشرة ندخل على الـ Queue.

### 🗺️ Roadmap

```text id="x8c3r1"
SECTION 4: Interceptors 🟢
├─ 4.1 What is Interceptor       ✅
├─ 4.2 Request Interceptor       ✅
├─ 4.3 Response Interceptor      ✅
├─ 4.4 Authentication            ✅
├─ 4.5 401 Handling              ✅
└─ 4.6 Token Refresh + Queue     🟡
     ├─ Token Refresh             ✅
     ├─ error.config              ✅
     ├─ Retry                     ✅
     ├─ Infinite Loop Protection   🟡 ← هنا
     └─ Queue                     ⬜
```

## 1. المشكلة

إحنا عندنا:

```text id="2gjy0d"
GET /users
      ↓
401
      ↓
Refresh
      ↓
POST /auth/refresh
      ↓
401
```

المشكلة إن `POST /auth/refresh` نفسه مرّ على الـ `Response Interceptor`.

فالـ Interceptor يشوف:

```ts
error.response?.status === 401
```

ويقول:

> أعمل Refresh!

فتبقى:

```text id="x8x8v3"
401
 ↓
Refresh
 ↓
401
 ↓
Refresh
 ↓
401
 ↓
Refresh
 ↓
∞
```

وده **Infinite Loop**.

---

# 2. الحل

نقول للـ Interceptor:

> لو الطلب الحالي هو نفسه طلب الـ Refresh، متحاولش تعمل Refresh مرة تانية.

ممكن نضيف Flag على الـ Config:

```ts
skipAuthRefresh: true
```

مثلاً:

```ts id="b5g1p9"
api.post(
  "/auth/refresh",
  { refreshToken },
  {
    skipAuthRefresh: true
  }
);
```

الفكرة:

```text id="q7w3y2"
Request عادي
      ↓
401
      ↓
Refresh Token
      ↓
Request /auth/refresh
      ↓
skipAuthRefresh = true
      ↓
لو حصل 401
      ↓
❌ لا تعمل Refresh مرة أخرى
```

---

## 3. لكن TypeScript عنده مشكلة

Axios مش عارف إن:

```ts
skipAuthRefresh
```

خاصية موجودة في الـ Config.

فلو كتبت:

```ts
api.post("/auth/refresh", data, {
  skipAuthRefresh: true
});
```

TypeScript غالبًا هيقول:

```text
Property 'skipAuthRefresh' does not exist...
```

وده طبيعي.

إحنا أضفنا **Custom Property** مش موجودة في Axios أصلاً.

لكن مش عايزك تدخل دلوقتي في `Module Augmentation` وتفاصيل TypeScript الخاصة بتوسيع Axios.

خلينا نفهم الـ Concept الأول.

---

# 4. طريقة أبسط للتعلم

ممكن نستخدم Flag على الـ Request Config:

```ts
const originalRequest = error.config;
```

وبعدين:

```ts
if (originalRequest._retry) {
  return Promise.reject(error);
}

originalRequest._retry = true;
```

المعنى:

```text id="d1c6x9"
أول مرة 401
    ↓
_retry = false / غير موجود
    ↓
نعمل Refresh
    ↓
_retry = true
    ↓
Retry
```

لو حصل `401` مرة تانية:

```text id="z4u2pk"
_retry = true
    ↓
❌ متعملش Refresh تاني
    ↓
Promise.reject(error)
```

---

# 5. الشكل الذهني

احفظها كده:

```text id="c0v3qy"
Original Request
      ↓
401
      ↓
هل عملنا Refresh قبل كده؟
      │
   ┌──┴──┐
   │     │
  لا     نعم
   │     │
   ↓     ↓
Refresh  Reject
   │
   ↓
New Token
   │
   ↓
Retry
```

وده بيمنع:

```text
401 → Refresh → 401 → Refresh → 401 → Refresh...
```

ويخليها:

```text
401
 ↓
Refresh
 ↓
New Token
 ↓
Retry
 ↓
401 مرة أخرى
 ↓
Reject ❌
```