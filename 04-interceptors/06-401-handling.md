## 4.5 يعني إيه 401 Handling؟

الـ `401 Unauthorized` غالبًا معناها إن الـ `Access Token` اللي بعتناه **مش صالح للمصادقة**، مثلًا انتهى أو غير صالح.

عندنا:

```text
Component
   ↓
api.get("/users/me")
   ↓
Request Interceptor
   ↓
Authorization: Bearer ACCESS_abc123
   ↓
Backend
   ↓
401 ❌
   ↓
Response Interceptor
```

وهنا الـ **Response Interceptor** يقدر يمسك الـ `401`.

مثلاً:

```ts
api.interceptors.response.use(
  (response) => {
    return response;
  },

  (error) => {
    if (error.response?.status === 401) {
      console.log("Access Token expired");
    }

    return Promise.reject(error);
  }
);
```

ركز في الجزء ده:

```ts
if (error.response?.status === 401)
```

معناه:

> لو الـ Backend رجع HTTP Status Code يساوي `401`، اعمل حاجة معينة.

---

### طيب ليه بنستخدم `error.response?.status`؟

لأن:

```text
error
 ↓
response
 ↓
status
```

والـ:

```ts
error.response?.status
```

هنا هو **HTTP Status Code**.

يعني ممكن يكون:

```text
401
403
404
500
```

---

### دلوقتي أهم نقطة

إحنا **لسه مش هنعمل Refresh**.

في `4.5` هدفنا فقط نفهم:

```text
401
 ↓
Response Interceptor
 ↓
اكتشف إن الـ Access Token فيه مشكلة
```

أما:

```text
401
 ↓
Refresh Token
 ↓
New Access Token
 ↓
Retry
```

ده موضوع **4.6 Token Refresh + Queue**.

### سؤال Interview 🎯

لو عندك:

```ts
api.get("/users/me")
```

والـ Backend رجع:

```http
401 Unauthorized
```

**مين اللي يقدر يمسك الـ 401 مركزيًا قبل ما يوصل للـ Component: الـ Request Interceptor ولا الـ Response Interceptor؟ وليه؟**

**الخطوة التالية:** جاوب السؤال ده في سطرين، وبعدها أصحح إجابتك ونكمل 4.5.
<details>
<summary>الـ response أنت ستة هو ليه 4004 وساعدة يبقى معناه إن التوكن غير صالح أو في مشكلة الكلام ده.</summary>
user
الـ response أنت ستة هو ليه 4004 وساعدة يبقى معناه إن التوكن غير صالح أو في مشكلة الكلام ده.
ChatGPT
إجابتك **صح في الفكرة**، ودرجة فهمك هنا **10/10**.

بس نصحح رقم واحد: أنت قلت `4004`، والمقصود **`401`**.

```text
Backend
   ↓
401 Unauthorized
   ↓
Response Interceptor
   ↓
error.response?.status === 401
   ↓
نعرف إن فيه مشكلة في Authentication
```

والـ `401` ممكن يحصل لأن:

- الـ `Access Token` انتهى.
- الـ `Access Token` غير صالح.
- الـ `Access Token` مفقود.

والأهم: **الـ Response Interceptor هو المناسب هنا** لأنه بيتعامل مع الـ Response اللي رجع من الـ Backend.

أما الـ Request Interceptor فشغله قبل الطلب، مثلًا:

```text
Request Interceptor
→ يضيف Access Token

Backend
→ يرجع 401

Response Interceptor
→ يمسك 401
```

**الخطوة التالية:** نبدأ `4.6 Token Refresh` ونكتب أول كود يجعل الـ `401` يستخدم الـ `Refresh Token` للحصول على `Access Token` جديد.
</details>
الـ response أنت ستة هو ليه 4004 وساعدة يبقى معناه إن التوكن غير صالح أو في مشكلة الكلام ده.
ChatGPT
إجابتك **صح في الفكرة**، ودرجة فهمك هنا **10/10**.

بس نصحح رقم واحد: أنت قلت `4004`، والمقصود **`401`**.

```text
Backend
   ↓
401 Unauthorized
   ↓
Response Interceptor
   ↓
error.response?.status === 401
   ↓
نعرف إن فيه مشكلة في Authentication
```

والـ `401` ممكن يحصل لأن:

- الـ `Access Token` انتهى.
- الـ `Access Token` غير صالح.
- الـ `Access Token` مفقود.

والأهم: **الـ Response Interceptor هو المناسب هنا** لأنه بيتعامل مع الـ Response اللي رجع من الـ Backend.

أما الـ Request Interceptor فشغله قبل الطلب، مثلًا:

```text
Request Interceptor
→ يضيف Access Token

Backend
→ يرجع 401

Response Interceptor
→ يمسك 401
```

**الخطوة التالية:** نبدأ `4.6 Token Refresh` ونكتب أول كود يجعل الـ `401` يستخدم الـ `Refresh Token` للحصول على `Access Token` جديد.
<details>
<summary>Token Refresh</summary>
user
Token Refresh
ChatGPT
ابدأ معايا **Token Refresh** في حوالي **15 دقيقة**، وهنبنيه خطوة خطوة من غير ما ندخل `Queue` لسه.

### 🗺️ Roadmap

```text
SECTION 4: Interceptors 🟢
├─ 4.1 What is Interceptor       ✅
├─ 4.2 Request Interceptor       ✅
├─ 4.3 Response Interceptor      ✅
├─ 4.4 Authentication            ✅
├─ 4.5 401 Handling              ✅
└─ 4.6 Token Refresh + Queue     🟡
     ├─ Token Refresh             🟡 ← هنا
     └─ Queue                     ⬜
```
</details>
Token Refresh
ChatGPT
ابدأ معايا **Token Refresh** في حوالي **15 دقيقة**، وهنبنيه خطوة خطوة من غير ما ندخل `Queue` لسه.

### 🗺️ Roadmap

```text
SECTION 4: Interceptors 🟢
├─ 4.1 What is Interceptor       ✅
├─ 4.2 Request Interceptor       ✅
├─ 4.3 Response Interceptor      ✅
├─ 4.4 Authentication            ✅
├─ 4.5 401 Handling              ✅
└─ 4.6 Token Refresh + Queue     🟡
     ├─ Token Refresh             🟡 ← هنا
     └─ Queue                     ⬜
```