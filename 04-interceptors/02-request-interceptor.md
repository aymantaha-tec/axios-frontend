# 4.2 Request Interceptor

دلوقتي هنكتب أول واحد **فعليًا**.

ابدأ بأبسط شكل:

```ts id="n6r2x8"
axios.interceptors.request.use(
  (config) => {
    return config;
  }
);
```

خلينا نفهمه حرفيًا.

```ts id="q8v4m1"
axios.interceptors.request
```

معناها:

> ادخل على Interceptors الخاصة بالـRequest.

ثم:

```ts id="c5k9p3"
.use(...)
```

معناها:

> سجل Interceptor جديد.

ثم:

```ts id="w2x7n4"
(config) => {
```

Axios بيديك الـRequest Configuration.

فاكر الـConfig اللي درسناه؟

```ts id="r9m3k6"
axios.get("/users", {
  params: {...},
  headers: {...},
  timeout: 5000,
  signal: controller.signal
});
```

كل ده موجود داخل:

```ts id="v6q2s8"
config
```

---

# إضافة الـToken

ممكن نعمل:

```ts id="a4k8p1"
axios.interceptors.request.use((config) => {
  const token = localStorage.getItem("token");

  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }

  return config;
});
```

الرحلة أصبحت:

```text id="h3m7q9"
axios.get("/users")
       ↓
Request Interceptor
       ↓
get token
       ↓
config.headers.Authorization
       ↓
Server
```

---

# ⚠️ لكن ركز

الـInterceptor **مش بياخد الـtoken كـparameter**.

يعني مش:

```ts id="j8p2v6"
(config, token) => {}
```

Axios هو اللي بيمرر:

```ts id="k4n7x3"
config
```

وأنت داخل الـInterceptor تقرر منين تجيب الـtoken:

```ts id="m5q9w2"
const token = localStorage.getItem("token");
```

---

# سؤال مهم جدًا

ليه بنعمل:

```ts id="z7p3c1"
return config;
```

لأن الـInterceptor استلم الـconfig.

لو عايزين الـRequest يكمل:

```text id="n8k2v5"
config
  ↓
Interceptor
  ↓
return config
  ↓
Server
```

لو عملت:

```ts id="y4m9q6"
return;
```

فأنت لم تعد بترجع الـconfig بالشكل المطلوب، وبالتالي الـRequest flow لن يستمر بشكل صحيح.

---

## 🎯 تدريب سريع

عندك:

```ts id="s6q3w8"
axios.interceptors.request.use((config) => {

  const token =
    localStorage.getItem("token");

  // هنا ماذا نكتب؟

  return config;
});
```

عايزك تكتب **السطر المسؤول عن إضافة الـtoken داخل الـheaders**.

**الخطوة التالية: اكتب سطر `config.headers...` فقط.**
<details>
<summary>if(token){.</summary>
user
if(token){.
config.headers.Authorozation ='Bearer${token}'
 }
ChatGPT
**صح في الفكرة، 9/10، وعندك خطأين Syntax صغيرين فقط.**

أنت كتبت:

```ts
if (token) {
  config.headers.Authorozation = 'Bearer${token}'
}
```

الصحيح:

```ts
if (token) {
  config.headers.Authorization = `Bearer ${token}`;
}
```

### الخطأ الأول

كتبت:

```ts
Authorozation
```

والصحيح:

```ts
Authorization
```

### الخطأ الثاني

أنت محتاج **Template Literal** عشان `${token}` تشتغل:

❌:

```ts
'Bearer ${token}'
```

لأن الـsingle quotes مش هتعمل interpolation.

✅:

```ts
`Bearer ${token}`
```

لاحظ إن دي **backticks**:

```text
`
```

والنتيجة مثلًا لو:

```ts
token = "abc123"
```

هتبقى:

```text
Authorization: Bearer abc123
```

---

## الكود كامل

```ts
axios.interceptors.request.use((config) => {
  const token = localStorage.getItem("token");

  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }

  return config;
});
```

وده أول **Request Interceptor** حقيقي عندك. ✅

## 🗺️ Roadmap

```text
SECTION 4: Interceptors 🟢
  ├─ 4.1 What is an Interceptor    ✅
  ├─ 4.2 Request Interceptor       🟢 ← نحن هنا
  ├─ 4.3 Response Interceptor      ⬜
  ├─ 4.4 Authentication            ⬜
  ├─ 4.5 401 Handling              ⬜
  └─ 4.6 Token Refresh + Queue     ⬜
```

**الخطوة التالية: هنضيف `Request Interceptor` داخل `axios.create()` بدل استخدام `axios` مباشرة، ونفهم ليه ده أفضل في Architecture حقيقية.**
</details>
if(token){.
config.headers.Authorozation ='Bearer${token}'
 }
ChatGPT
**صح في الفكرة، 9/10، وعندك خطأين Syntax صغيرين فقط.**

أنت كتبت:

```ts
if (token) {
  config.headers.Authorozation = 'Bearer${token}'
}
```

الصحيح:

```ts
if (token) {
  config.headers.Authorization = `Bearer ${token}`;
}
```

### الخطأ الأول

كتبت:

```ts
Authorozation
```

والصحيح:

```ts
Authorization
```

### الخطأ الثاني

أنت محتاج **Template Literal** عشان `${token}` تشتغل:

❌:

```ts
'Bearer ${token}'
```

لأن الـsingle quotes مش هتعمل interpolation.

✅:

```ts
`Bearer ${token}`
```

لاحظ إن دي **backticks**:

```text
`
```

والنتيجة مثلًا لو:

```ts
token = "abc123"
```

هتبقى:

```text
Authorization: Bearer abc123
```

---

## الكود كامل

```ts
axios.interceptors.request.use((config) => {
  const token = localStorage.getItem("token");

  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }

  return config;
});
```

وده أول **Request Interceptor** حقيقي عندك. ✅

## 🗺️ Roadmap

```text
SECTION 4: Interceptors 🟢
  ├─ 4.1 What is an Interceptor    ✅
  ├─ 4.2 Request Interceptor       🟢 ← نحن هنا
  ├─ 4.3 Response Interceptor      ⬜
  ├─ 4.4 Authentication            ⬜
  ├─ 4.5 401 Handling              ⬜
  └─ 4.6 Token Refresh + Queue     ⬜
```

**الخطوة التالية: هنضيف `Request Interceptor` داخل `axios.create()` بدل استخدام `axios` مباشرة، ونفهم ليه ده أفضل في Architecture حقيقية.**
<details>
<summary>كمل</summary>
user
كمل
ChatGPT
**نقل الـRequest Interceptor إلى `axios.create()` هو الخطوة التالية، ومدته حوالي 15 دقيقة، ودي نقطة مهمة جدًا في بناء Axios Architecture احترافية.**

## 🗺️ Roadmap

```text
SECTION 4: Interceptors 🟢
  ├─ 4.1 What is an Interceptor    ✅
  ├─ 4.2 Request Interceptor       🟢 ← نحن هنا
  ├─ 4.3 Response Interceptor      ⬜
  ├─ 4.4 Authentication            ⬜
  ├─ 4.5 401 Handling              ⬜
  └─ 4.6 Token Refresh + Queue     ⬜
```

# ليه `axios.create()`؟

لحد دلوقتي كنا بنكتب:

```ts id="g4m8z2"
axios.get("/users");
```

وبعدين:

```ts id="q8v2k1"
axios.interceptors.request.use(...)
```

ده يشتغل.

لكن في مشروع حقيقي غالبًا مش عايزين نستخدم Axios global instance في كل مكان.

بدل كده نعمل **Axios Instance** خاصة بتطبيقنا:

```ts id="m7p3x9"
const api = axios.create({
  baseURL: "https://api.example.com",
  timeout: 5000,
});
```

بقى عندنا:

```text id="h2k6w4"
axios
  │
  └── create()
        ↓
       api
```

والـ`api` دي هي اللي هنستخدمها في المشروع.

---

# 1. نضيف Request Interceptor

```ts id="r5n8q3"
const api = axios.create({
  baseURL: "https://api.example.com",
  timeout: 5000,
});

api.interceptors.request.use((config) => {
  const token = localStorage.getItem("token");

  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }

  return config;
});
```

لاحظ حاجة مهمة جدًا:

بدل:

```ts
axios.interceptors.request.use(...)
```

أصبح:

```ts
api.interceptors.request.use(...)
```

لأننا عايزين الـInterceptor يكون مربوط بالـinstance بتاعتنا.

---

# 2. API Layer تستخدم `api`

بدل:

```ts id="c9w5k1"
axios.get("/users");
```

نستخدم:

```ts id="v3m7q2"
api.get("/users");
```

مثلاً:

```ts id="k8p4x6"
export const getUsers = async (): Promise<IUser[]> => {
  const response =
    await api.get<IApiResponse<IUser[]>>("/users");

  return response.data.payload;
};
```

الرحلة أصبحت:

```text id="j6r2m8"
Component
    ↓
getUsers()
    ↓
api
    ↓
Request Interceptor
    ↓
Backend
```

وده Architecture أنضف.

---

# 3. ليه Instance مهمة؟

لأنك ممكن يكون عندك إعدادات مشتركة:

```text id="q7v3n9"
baseURL
timeout
headers
interceptors
authentication
error handling
```

بدل ما تكررهم في كل Request.

مثلاً:

```ts id="s8k2p4"
api.get("/users");
api.get("/products");
api.post("/orders", data);
api.patch("/profile", data);
```

كلهم يستخدموا نفس الـinstance.

---

# 4. أهم نقطة معمارية

ممكن نعمل ملف:

```text id="w4n7x2"
src/
└── shared/
    └── api/
        └── apiClient.ts
```

وفيه:

```ts id="a6m3q8"
import axios from "axios";

export const api = axios.create({
  baseURL: "https://api.example.com",
  timeout: 5000,
});
```

ثم نضيف الـinterceptors في نفس الـAPI infrastructure.

وبعد كده:

```text id="f9k2m5"
features/
└── users/
    └── api/
        └── userApi.ts
```

يستخدم:

```ts id="u3p8r1"
import { api } from "@/shared/api/apiClient";
```

لاحظ الـDependency Direction:

```text id="d5x7q9"
users
  ↓
shared/api
  ↓
axios
```

وده متوافق مع اللي اتعلمناه في Section 0.

---

# 🧠 فرق مهم

### `axios`

هو الـlibrary نفسه.

```ts id="p7m2x4"
axios.get(...)
```

### `api`

دي **Axios Instance** أنت أنشأتها:

```ts id="k9w3s6"
const api = axios.create(...)
```

وبالتالي:

```text id="h4n8q2"
axios
   ↓
create()
   ↓
api instance
```

والـ`api` ممكن يكون لها:

```text id="c6r1m7"
baseURL
timeout
interceptors
defaults
```

خاصة بتطبيقك.

---

# 🎯 اختبار

عندك:

```ts id="r8k4p2"
const api = axios.create({
  baseURL: "https://api.example.com",
});
```

وعندك:

```ts id="n5q7w3"
axios.get("/users");
```

و:

```ts id="z2m9x6"
api.get("/users");
```

**أي واحد نستخدم في Architecture بتاعتنا؟**

وليه؟

اكتب إجابة قصيرة.

**الخطوة التالية: اختر `axios.get` أو `api.get` واشرح السبب في جملة واحدة.**
</details>
كمل
ChatGPT
**نقل الـRequest Interceptor إلى `axios.create()` هو الخطوة التالية، ومدته حوالي 15 دقيقة، ودي نقطة مهمة جدًا في بناء Axios Architecture احترافية.**

## 🗺️ Roadmap

```text
SECTION 4: Interceptors 🟢
  ├─ 4.1 What is an Interceptor    ✅
  ├─ 4.2 Request Interceptor       🟢 ← نحن هنا
  ├─ 4.3 Response Interceptor      ⬜
  ├─ 4.4 Authentication            ⬜
  ├─ 4.5 401 Handling              ⬜
  └─ 4.6 Token Refresh + Queue     ⬜
```

# ليه `axios.create()`؟

لحد دلوقتي كنا بنكتب:

```ts id="g4m8z2"
axios.get("/users");
```

وبعدين:

```ts id="q8v2k1"
axios.interceptors.request.use(...)
```

ده يشتغل.

لكن في مشروع حقيقي غالبًا مش عايزين نستخدم Axios global instance في كل مكان.

بدل كده نعمل **Axios Instance** خاصة بتطبيقنا:

```ts id="m7p3x9"
const api = axios.create({
  baseURL: "https://api.example.com",
  timeout: 5000,
});
```

بقى عندنا:

```text id="h2k6w4"
axios
  │
  └── create()
        ↓
       api
```

والـ`api` دي هي اللي هنستخدمها في المشروع.

---

# 1. نضيف Request Interceptor

```ts id="r5n8q3"
const api = axios.create({
  baseURL: "https://api.example.com",
  timeout: 5000,
});

api.interceptors.request.use((config) => {
  const token = localStorage.getItem("token");

  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }

  return config;
});
```

لاحظ حاجة مهمة جدًا:

بدل:

```ts
axios.interceptors.request.use(...)
```

أصبح:

```ts
api.interceptors.request.use(...)
```

لأننا عايزين الـInterceptor يكون مربوط بالـinstance بتاعتنا.

---

# 2. API Layer تستخدم `api`

بدل:

```ts id="c9w5k1"
axios.get("/users");
```

نستخدم:

```ts id="v3m7q2"
api.get("/users");
```

مثلاً:

```ts id="k8p4x6"
export const getUsers = async (): Promise<IUser[]> => {
  const response =
    await api.get<IApiResponse<IUser[]>>("/users");

  return response.data.payload;
};
```

الرحلة أصبحت:

```text id="j6r2m8"
Component
    ↓
getUsers()
    ↓
api
    ↓
Request Interceptor
    ↓
Backend
```

وده Architecture أنضف.

---

# 3. ليه Instance مهمة؟

لأنك ممكن يكون عندك إعدادات مشتركة:

```text id="q7v3n9"
baseURL
timeout
headers
interceptors
authentication
error handling
```

بدل ما تكررهم في كل Request.

مثلاً:

```ts id="s8k2p4"
api.get("/users");
api.get("/products");
api.post("/orders", data);
api.patch("/profile", data);
```

كلهم يستخدموا نفس الـinstance.

---

# 4. أهم نقطة معمارية

ممكن نعمل ملف:

```text id="w4n7x2"
src/
└── shared/
    └── api/
        └── apiClient.ts
```

وفيه:

```ts id="a6m3q8"
import axios from "axios";

export const api = axios.create({
  baseURL: "https://api.example.com",
  timeout: 5000,
});
```

ثم نضيف الـinterceptors في نفس الـAPI infrastructure.

وبعد كده:

```text id="f9k2m5"
features/
└── users/
    └── api/
        └── userApi.ts
```

يستخدم:

```ts id="u3p8r1"
import { api } from "@/shared/api/apiClient";
```

لاحظ الـDependency Direction:

```text id="d5x7q9"
users
  ↓
shared/api
  ↓
axios
```

وده متوافق مع اللي اتعلمناه في Section 0.

---

# 🧠 فرق مهم

### `axios`

هو الـlibrary نفسه.

```ts id="p7m2x4"
axios.get(...)
```

### `api`

دي **Axios Instance** أنت أنشأتها:

```ts id="k9w3s6"
const api = axios.create(...)
```

وبالتالي:

```text id="h4n8q2"
axios
   ↓
create()
   ↓
api instance
```

والـ`api` ممكن يكون لها:

```text id="c6r1m7"
baseURL
timeout
interceptors
defaults
```

خاصة بتطبيقك.

---

# 🎯 اختبار

عندك:

```ts id="r8k4p2"
const api = axios.create({
  baseURL: "https://api.example.com",
});
```

وعندك:

```ts id="n5q7w3"
axios.get("/users");
```

و:

```ts id="z2m9x6"
api.get("/users");
```

**أي واحد نستخدم في Architecture بتاعتنا؟**

وليه؟

اكتب إجابة قصيرة.

**الخطوة التالية: اختر `axios.get` أو `api.get` واشرح السبب في جملة واحدة.**
<details>
<summary>api.getليبدنا الـ instance اللي خاص بي.</summary>
user
api.getليبدنا الـ instance اللي خاص بي.
ChatGPT
**صح تمامًا، 10/10: نستخدم `api.get()` لأن `api` هو الـAxios Instance الخاص بتطبيقنا، ومدته أقل من دقيقة.**

بالضبط:

```ts id="4y6p2k"
const api = axios.create({
  baseURL: "https://api.example.com",
});
```

بعدها الـFeatures تستخدم:

```ts id="9m3k7q"
api.get("/users");
api.post("/users", data);
api.patch("/users/10", data);
```

بدل ما تستخدم الـglobal `axios` مباشرة.

والفكرة المعمارية:

```text id="f7w2n9"
Feature
   ↓
api instance
   ↓
Interceptors
   ↓
Axios
   ↓
Backend
```

وده بيدينا مكان مركزي نتحكم منه في:

```text id="m8q4r1"
baseURL
timeout
headers
Request Interceptors
Response Interceptors
Authentication
Error Handling
```

## 🗺️ Roadmap

```text id="d5p9x3"
SECTION 4: Interceptors 🟢
  ├─ 4.1 What is an Interceptor    ✅
  ├─ 4.2 Request Interceptor       🟢
  │    ├─ axios.create()            ✅
  │    ├─ Axios Instance            ✅
  │    └─ Auth Header               ✅
  ├─ 4.3 Response Interceptor      ⬜
  ├─ 4.4 Authentication            ⬜
  ├─ 4.5 401 Handling              ⬜
  └─ 4.6 Token Refresh + Queue     ⬜
```

**الخطوة التالية: نعمل `4.2` عمليًا من البداية ونكتب `apiClient.ts` كامل، ثم نضيف له Request Interceptor خطوة خطوة.**
</details>
api.getليبدنا الـ instance اللي خاص بي.
ChatGPT
**صح تمامًا، 10/10: نستخدم `api.get()` لأن `api` هو الـAxios Instance الخاص بتطبيقنا، ومدته أقل من دقيقة.**

بالضبط:

```ts id="4y6p2k"
const api = axios.create({
  baseURL: "https://api.example.com",
});
```

بعدها الـFeatures تستخدم:

```ts id="9m3k7q"
api.get("/users");
api.post("/users", data);
api.patch("/users/10", data);
```

بدل ما تستخدم الـglobal `axios` مباشرة.

والفكرة المعمارية:

```text id="f7w2n9"
Feature
   ↓
api instance
   ↓
Interceptors
   ↓
Axios
   ↓
Backend
```

وده بيدينا مكان مركزي نتحكم منه في:

```text id="m8q4r1"
baseURL
timeout
headers
Request Interceptors
Response Interceptors
Authentication
Error Handling
```

## 🗺️ Roadmap

```text id="d5p9x3"
SECTION 4: Interceptors 🟢
  ├─ 4.1 What is an Interceptor    ✅
  ├─ 4.2 Request Interceptor       🟢
  │    ├─ axios.create()            ✅
  │    ├─ Axios Instance            ✅
  │    └─ Auth Header               ✅
  ├─ 4.3 Response Interceptor      ⬜
  ├─ 4.4 Authentication            ⬜
  ├─ 4.5 401 Handling              ⬜
  └─ 4.6 Token Refresh + Queue     ⬜
```

**الخطوة التالية: نعمل `4.2` عمليًا من البداية ونكتب `apiClient.ts` كامل، ثم نضيف له Request Interceptor خطوة خطوة.**
<details>
<summary>معاك بس راعي التايب اسكربت</summary>
user
معاك بس راعي التايب اسكربت
ChatGPT
**هنبني `apiClient.ts` بـTypeScript من البداية، ومدته حوالي 20 دقيقة، ومش هنتجاوز أي Type قبل ما نفهمه.**

## 🗺️ Roadmap

```text
SECTION 4: Interceptors 🟢
  ├─ 4.1 What is an Interceptor    ✅
  ├─ 4.2 Request Interceptor       🟢 ← نحن هنا
  │    ├─ axios.create()            ✅
  │    ├─ Axios Instance            ✅
  │    ├─ TypeScript typing         🟢
  │    └─ Auth Header               ⬜
  ├─ 4.3 Response Interceptor      ⬜
  ├─ 4.4 Authentication            ⬜
  ├─ 4.5 401 Handling              ⬜
  └─ 4.6 Token Refresh + Queue     ⬜
```

# نبني `apiClient.ts`

خلينا نبدأ بالملف:

```text id="q5m8r2"
src/
└── shared/
    └── api/
        └── apiClient.ts
```

أول سطر:

```ts id="m7x3k9"
import axios from "axios";
```

ده عادي، لأننا بنستورد Axios.

---

## 1. إنشاء الـInstance

```ts id="v4n8p2"
export const api = axios.create({
  baseURL: "https://api.example.com",
  timeout: 5000,
});
```

هنا TypeScript بيعمل حاجة مهمة.

هو بيعرف تلقائيًا إن:

```ts id="a8q2w6"
api
```

هو **AxiosInstance**.

يعني مش محتاج نكتب:

```ts id="z3m7k1"
const api: AxiosInstance = ...
```

إحنا ممكن، لكن مش محتاجين لأن TypeScript يستنتج النوع.

---

# 2. نضيف Request Interceptor

```ts id="n6p4x8"
api.interceptors.request.use(
  (config) => {
    return config;
  }
);
```

ركز هنا.

`config` مش `any`.

Axios عارف نوعه من تعريف المكتبة.

تقريبًا:

```ts id="w2k7m5"
config: InternalAxiosRequestConfig
```

وده نوع Axios نفسه.

إذن TypeScript يعرف إن `config` فيه حاجات مثل:

```text id="r8v3q1"
config.url
config.method
config.headers
config.params
config.data
config.timeout
config.signal
```

ودي نفس الـRequest Configuration اللي درسناها.

---

# 3. نجيب الـToken

```ts id="c5m9x2"
api.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem("token");

    return config;
  }
);
```

وهنا TypeScript مهم جدًا.

`localStorage.getItem()` نوعه:

```ts id="h7q2w4"
string | null
```

ليه؟

لأن الـtoken ممكن يكون موجود:

```ts id="j3p8n5"
"abc123"
```

أو مش موجود:

```ts id="u6k1r9"
null
```

لذلك نعمل:

```ts id="s4x8m2"
if (token) {
  // هنا TypeScript عرف إن token string
}
```

---

# 4. إضافة Authorization

```ts id="p9k3w7"
api.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem("token");

    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }

    return config;
  }
);
```

لاحظ إننا **ما كتبناش**:

```ts id="z8m4q1"
token: string
```

لأن TypeScript عرف بعد:

```ts id="a2v6k9"
if (token)
```

إن `token` أصبح:

```ts id="f5r9x3"
string
```

وده اسمه:

**Type Narrowing**.

---

# 5. ليه ما نكتبش `any`؟

مهم جدًا في الكورس بتاعنا.

❌ ما تعملش:

```ts id="d7q2m8"
const config: any
```

لأن Axios بالفعل أعطاك Type مناسب.

إحنا عايزين نستفيد من TypeScript.

مثلاً لو كتبت بالخطأ:

```ts id="v3k8p5"
config.headers.Authorizationn
```

TypeScript ممكن يساعدك تكتشف إن الاسم غلط.

---

# الشكل النهائي حاليًا

```ts id="n4w7q2"
import axios from "axios";

export const api = axios.create({
  baseURL: "https://api.example.com",
  timeout: 5000,
});

api.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem("token");

    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }

    return config;
  }
);
```

لكن **لسه مش عايزك تحفظ الكود**.

عايزك تفهم الـType flow:

```text id="y6p2m8"
localStorage.getItem("token")
          ↓
     string | null
          ↓
      if (token)
          ↓
        string
          ↓
Bearer ${token}
```

والـconfig:

```text id="q3n8w5"
Axios
  ↓
request interceptor
  ↓
config
  ↓
config.headers
  ↓
Authorization
```

---

## 🎯 اختبار TypeScript

عندك:

```ts id="r7m2x9"
const token = localStorage.getItem("token");
```

ما هو الـType الذي يستنتجه TypeScript للمتغير `token`؟

**A)** `string`

**B)** `null`

**C)** `string | null`

**D)** `any`

**الخطوة التالية: اختر A أو B أو C أو D، وبعدها نكمل Typing الـRequest Interceptor.**
</details>
معاك بس راعي التايب اسكربت
ChatGPT
**هنبني `apiClient.ts` بـTypeScript من البداية، ومدته حوالي 20 دقيقة، ومش هنتجاوز أي Type قبل ما نفهمه.**

## 🗺️ Roadmap

```text
SECTION 4: Interceptors 🟢
  ├─ 4.1 What is an Interceptor    ✅
  ├─ 4.2 Request Interceptor       🟢 ← نحن هنا
  │    ├─ axios.create()            ✅
  │    ├─ Axios Instance            ✅
  │    ├─ TypeScript typing         🟢
  │    └─ Auth Header               ⬜
  ├─ 4.3 Response Interceptor      ⬜
  ├─ 4.4 Authentication            ⬜
  ├─ 4.5 401 Handling              ⬜
  └─ 4.6 Token Refresh + Queue     ⬜
```

# نبني `apiClient.ts`

خلينا نبدأ بالملف:

```text id="q5m8r2"
src/
└── shared/
    └── api/
        └── apiClient.ts
```

أول سطر:

```ts id="m7x3k9"
import axios from "axios";
```

ده عادي، لأننا بنستورد Axios.

---

## 1. إنشاء الـInstance

```ts id="v4n8p2"
export const api = axios.create({
  baseURL: "https://api.example.com",
  timeout: 5000,
});
```

هنا TypeScript بيعمل حاجة مهمة.

هو بيعرف تلقائيًا إن:

```ts id="a8q2w6"
api
```

هو **AxiosInstance**.

يعني مش محتاج نكتب:

```ts id="z3m7k1"
const api: AxiosInstance = ...
```

إحنا ممكن، لكن مش محتاجين لأن TypeScript يستنتج النوع.

---

# 2. نضيف Request Interceptor

```ts id="n6p4x8"
api.interceptors.request.use(
  (config) => {
    return config;
  }
);
```

ركز هنا.

`config` مش `any`.

Axios عارف نوعه من تعريف المكتبة.

تقريبًا:

```ts id="w2k7m5"
config: InternalAxiosRequestConfig
```

وده نوع Axios نفسه.

إذن TypeScript يعرف إن `config` فيه حاجات مثل:

```text id="r8v3q1"
config.url
config.method
config.headers
config.params
config.data
config.timeout
config.signal
```

ودي نفس الـRequest Configuration اللي درسناها.

---

# 3. نجيب الـToken

```ts id="c5m9x2"
api.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem("token");

    return config;
  }
);
```

وهنا TypeScript مهم جدًا.

`localStorage.getItem()` نوعه:

```ts id="h7q2w4"
string | null
```

ليه؟

لأن الـtoken ممكن يكون موجود:

```ts id="j3p8n5"
"abc123"
```

أو مش موجود:

```ts id="u6k1r9"
null
```

لذلك نعمل:

```ts id="s4x8m2"
if (token) {
  // هنا TypeScript عرف إن token string
}
```

---

# 4. إضافة Authorization

```ts id="p9k3w7"
api.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem("token");

    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }

    return config;
  }
);
```

لاحظ إننا **ما كتبناش**:

```ts id="z8m4q1"
token: string
```

لأن TypeScript عرف بعد:

```ts id="a2v6k9"
if (token)
```

إن `token` أصبح:

```ts id="f5r9x3"
string
```

وده اسمه:

**Type Narrowing**.

---

# 5. ليه ما نكتبش `any`؟

مهم جدًا في الكورس بتاعنا.

❌ ما تعملش:

```ts id="d7q2m8"
const config: any
```

لأن Axios بالفعل أعطاك Type مناسب.

إحنا عايزين نستفيد من TypeScript.

مثلاً لو كتبت بالخطأ:

```ts id="v3k8p5"
config.headers.Authorizationn
```

TypeScript ممكن يساعدك تكتشف إن الاسم غلط.

---

# الشكل النهائي حاليًا

```ts id="n4w7q2"
import axios from "axios";

export const api = axios.create({
  baseURL: "https://api.example.com",
  timeout: 5000,
});

api.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem("token");

    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }

    return config;
  }
);
```

لكن **لسه مش عايزك تحفظ الكود**.

عايزك تفهم الـType flow:

```text id="y6p2m8"
localStorage.getItem("token")
          ↓
     string | null
          ↓
      if (token)
          ↓
        string
          ↓
Bearer ${token}
```

والـconfig:

```text id="q3n8w5"
Axios
  ↓
request interceptor
  ↓
config
  ↓
config.headers
  ↓
Authorization
```

---

## 🎯 اختبار TypeScript

عندك:

```ts id="r7m2x9"
const token = localStorage.getItem("token");
```

ما هو الـType الذي يستنتجه TypeScript للمتغير `token`؟

**A)** `string`

**B)** `null`

**C)** `string | null`

**D)** `any`

**الخطوة التالية: اختر A أو B أو C أو D، وبعدها نكمل Typing الـRequest Interceptor.**
<details>
<summary>C</summary>
user
C
ChatGPT
**C صحيحة 100%، وتقييمك 10/10، ومدتها أقل من دقيقة.**

```ts id="x7p3k9"
const token = localStorage.getItem("token");
```

نوع `token` هو:

```ts id="m4q8w2"
string | null
```

لأن القيمة ممكن تكون موجودة:

```ts id="j6r1v5"
"abc123"
```

أو غير موجودة:

```ts id="p8n3k7"
null
```

وعشان كده:

```ts id="z2w6m9"
if (token) {
  config.headers.Authorization = `Bearer ${token}`;
}
```

داخل الـ`if`، TypeScript يعمل **Type Narrowing**:

```text id="a5k8q1"
string | null
      ↓
 if (token)
      ↓
   string
```

ودي نقطة مهمة جدًا في TypeScript.

## 🗺️ Roadmap

```text id="v3n7x2"
SECTION 4: Interceptors 🟢
  ├─ 4.1 What is an Interceptor    ✅
  ├─ 4.2 Request Interceptor       🟢
  │    ├─ axios.create()            ✅
  │    ├─ Axios Instance            ✅
  │    ├─ TypeScript typing         ✅
  │    └─ Auth Header               🟢
  ├─ 4.3 Response Interceptor      ⬜
  ├─ 4.4 Authentication            ⬜
  ├─ 4.5 401 Handling              ⬜
  └─ 4.6 Token Refresh + Queue     ⬜
```

### سؤال TypeScript التالي

في:

```ts id="k9r2m6"
api.interceptors.request.use((config) => {
  return config;
});
```

هل نحتاج نكتب يدويًا:

```ts id="q4w7p1"
(config: InternalAxiosRequestConfig)
```

ولا TypeScript يقدر يعرف نوع `config` تلقائيًا؟

**الخطوة التالية: جاوب `يستنتجه` أو `لازم أكتبه`.**
</details>
C
ChatGPT
**C صحيحة 100%، وتقييمك 10/10، ومدتها أقل من دقيقة.**

```ts id="x7p3k9"
const token = localStorage.getItem("token");
```

نوع `token` هو:

```ts id="m4q8w2"
string | null
```

لأن القيمة ممكن تكون موجودة:

```ts id="j6r1v5"
"abc123"
```

أو غير موجودة:

```ts id="p8n3k7"
null
```

وعشان كده:

```ts id="z2w6m9"
if (token) {
  config.headers.Authorization = `Bearer ${token}`;
}
```

داخل الـ`if`، TypeScript يعمل **Type Narrowing**:

```text id="a5k8q1"
string | null
      ↓
 if (token)
      ↓
   string
```

ودي نقطة مهمة جدًا في TypeScript.

## 🗺️ Roadmap

```text id="v3n7x2"
SECTION 4: Interceptors 🟢
  ├─ 4.1 What is an Interceptor    ✅
  ├─ 4.2 Request Interceptor       🟢
  │    ├─ axios.create()            ✅
  │    ├─ Axios Instance            ✅
  │    ├─ TypeScript typing         ✅
  │    └─ Auth Header               🟢
  ├─ 4.3 Response Interceptor      ⬜
  ├─ 4.4 Authentication            ⬜
  ├─ 4.5 401 Handling              ⬜
  └─ 4.6 Token Refresh + Queue     ⬜
```

### سؤال TypeScript التالي

في:

```ts id="k9r2m6"
api.interceptors.request.use((config) => {
  return config;
});
```

هل نحتاج نكتب يدويًا:

```ts id="q4w7p1"
(config: InternalAxiosRequestConfig)
```

ولا TypeScript يقدر يعرف نوع `config` تلقائيًا؟

**الخطوة التالية: جاوب `يستنتجه` أو `لازم أكتبه`.**
<details>
<summary>بيعرفه تلقائ</summary>
user
بيعرفه تلقائ
ChatGPT
**بالضبط، TypeScript بيستنتج نوع `config` تلقائيًا، 10/10، وده اسمه Contextual Typing، ومدته أقل من دقيقة.**

عندنا:

```ts
api.interceptors.request.use((config) => {
  return config;
});
```

Axios محدد مسبقًا نوع الـcallback اللي `use()` مستنيه، فـTypeScript يعرف إن:

```ts
config
```

نوعه تقريبًا:

```ts
InternalAxiosRequestConfig
```

لذلك **مش محتاج** تكتب:

```ts
(config: InternalAxiosRequestConfig)
```

يدويًا.

### الفرق المهم

في الحالة دي:

```ts
api.interceptors.request.use((config) => {
```

TypeScript عنده **Context** من `use()`، فاستنتج النوع.

وده اسمه:

> **Contextual Typing**

لكن لو كتبت function مستقلة:

```ts
const handleRequest = (config) => {
  return config;
};
```

هنا TypeScript مش عنده نفس الـcontext، وممكن تحتاج تحدد النوع.

---

## 🗺️ Roadmap

```text id="w3k8p2"
SECTION 4: Interceptors 🟢
  ├─ 4.1 What is an Interceptor    ✅
  ├─ 4.2 Request Interceptor       🟢
  │    ├─ axios.create()            ✅
  │    ├─ Axios Instance            ✅
  │    ├─ TypeScript typing         ✅
  │    ├─ Contextual Typing         ✅
  │    └─ Auth Header               🟢
  ├─ 4.3 Response Interceptor      ⬜
  ├─ 4.4 Authentication            ⬜
  ├─ 4.5 401 Handling              ⬜
  └─ 4.6 Token Refresh + Queue     ⬜
```

**الخطوة التالية: هنكتب `apiClient.ts` بنفسك، وأنا هديك الهيكل الناقص وتملأ فقط الجزء الخاص بالـ`Authorization`، في حوالي 5 دقائق.**
</details>
بيعرفه تلقائ
ChatGPT
**بالضبط، TypeScript بيستنتج نوع `config` تلقائيًا، 10/10، وده اسمه Contextual Typing، ومدته أقل من دقيقة.**

عندنا:

```ts
api.interceptors.request.use((config) => {
  return config;
});
```

Axios محدد مسبقًا نوع الـcallback اللي `use()` مستنيه، فـTypeScript يعرف إن:

```ts
config
```

نوعه تقريبًا:

```ts
InternalAxiosRequestConfig
```

لذلك **مش محتاج** تكتب:

```ts
(config: InternalAxiosRequestConfig)
```

يدويًا.

### الفرق المهم

في الحالة دي:

```ts
api.interceptors.request.use((config) => {
```

TypeScript عنده **Context** من `use()`، فاستنتج النوع.

وده اسمه:

> **Contextual Typing**

لكن لو كتبت function مستقلة:

```ts
const handleRequest = (config) => {
  return config;
};
```

هنا TypeScript مش عنده نفس الـcontext، وممكن تحتاج تحدد النوع.

---

## 🗺️ Roadmap

```text id="w3k8p2"
SECTION 4: Interceptors 🟢
  ├─ 4.1 What is an Interceptor    ✅
  ├─ 4.2 Request Interceptor       🟢
  │    ├─ axios.create()            ✅
  │    ├─ Axios Instance            ✅
  │    ├─ TypeScript typing         ✅
  │    ├─ Contextual Typing         ✅
  │    └─ Auth Header               🟢
  ├─ 4.3 Response Interceptor      ⬜
  ├─ 4.4 Authentication            ⬜
  ├─ 4.5 401 Handling              ⬜
  └─ 4.6 Token Refresh + Queue     ⬜
```

**الخطوة التالية: هنكتب `apiClient.ts` بنفسك، وأنا هديك الهيكل الناقص وتملأ فقط الجزء الخاص بالـ`Authorization`، في حوالي 5 دقائق.**
<details>
<summary>تمام</summary>
user
تمام
ChatGPT
اكتب السطر الناقص بنفسك في **5 دقائق**، وركز فقط على الـ `Authorization` بدون ما ندخل في أي concept جديد.

### 🗺️ Roadmap

```text
SECTION 4: Interceptors 🟢
  ├─ 4.1 What is an Interceptor    ✅
  ├─ 4.2 Request Interceptor       🟢 ← هنا
  │    ├─ axios.create()            ✅
  │    ├─ Axios Instance            ✅
  │    ├─ TypeScript typing         ✅
  │    ├─ Contextual Typing         ✅
  │    └─ Auth Header               🟢
  ├─ 4.3 Response Interceptor      ⬜
  ├─ 4.4 Authentication            ⬜
  ├─ 4.5 401 Handling              ⬜
  └─ 4.6 Token Refresh + Queue     ⬜
```

اكتب الكود ده وأكمل الجزء الناقص:

```ts
import axios from "axios";

export const api = axios.create({
  baseURL: "https://api.example.com",
  timeout: 5000,
});

api.interceptors.request.use((config) => {
  const token = localStorage.getItem("token");

  if (token) {
    // اكتب السطر هنا
  }

  return config;
});
```

**المطلوب منك سطر واحد فقط:** إضافة الـ token داخل `Authorization` Header بصيغة `Bearer`.

**الخطوة التالية:** اكتب السطر الناقص فقط.
</details>
تمام
ChatGPT
اكتب السطر الناقص بنفسك في **5 دقائق**، وركز فقط على الـ `Authorization` بدون ما ندخل في أي concept جديد.

### 🗺️ Roadmap

```text
SECTION 4: Interceptors 🟢
  ├─ 4.1 What is an Interceptor    ✅
  ├─ 4.2 Request Interceptor       🟢 ← هنا
  │    ├─ axios.create()            ✅
  │    ├─ Axios Instance            ✅
  │    ├─ TypeScript typing         ✅
  │    ├─ Contextual Typing         ✅
  │    └─ Auth Header               🟢
  ├─ 4.3 Response Interceptor      ⬜
  ├─ 4.4 Authentication            ⬜
  ├─ 4.5 401 Handling              ⬜
  └─ 4.6 Token Refresh + Queue     ⬜
```

اكتب الكود ده وأكمل الجزء الناقص:

```ts
import axios from "axios";

export const api = axios.create({
  baseURL: "https://api.example.com",
  timeout: 5000,
});

api.interceptors.request.use((config) => {
  const token = localStorage.getItem("token");

  if (token) {
    // اكتب السطر هنا
  }

  return config;
});
```

**المطلوب منك سطر واحد فقط:** إضافة الـ token داخل `Authorization` Header بصيغة `Bearer`.

**الخطوة التالية:** اكتب السطر الناقص فقط.
<details>
<summary>config.headers.Authorization  ='Bearer${token}'</summary>
user
config.headers.Authorization  ='Bearer${token}'
ChatGPT
صح جدًا، أنت وصلت للمكان الصح، لكن عندك **غلطة واحدة في الـ syntax**.

أنت كتبت:

```ts
config.headers.Authorization = 'Bearer${token}'
```

المشكلة إن `' '` عبارة عن **String عادي**، وبالتالي `${token}` مش هتتفسر.

لازم تستخدم **Backticks**:

```ts
config.headers.Authorization = `Bearer ${token}`;
```

### ليه؟

```ts
'Bearer ${token}'
```

تتعامل معها JavaScript كنص حرفي:

```text
Bearer ${token}
```

لكن:

```ts
`Bearer ${token}`
```

تستخدم **Template Literal**، وبالتالي لو:

```ts
token = "abc123"
```

الناتج يكون:

```text
Bearer abc123
```

### وTypeScript هنا كمان شغال معاك

قبل الـ `if`:

```ts
const token = localStorage.getItem("token");
```

نوع `token` هو:

```ts
string | null
```

لكن بعد:

```ts
if (token) {
```

TypeScript يعرف إن `token` موجود، فيتعامل معه كـ:

```ts
string
```

وده اسمه **Type Narrowing**.

**تقييمك في الجزئية دي: 10/10 في الفكرة، فقط خطأ Syntax في الـ quotes.**

**الخطوة التالية:** اكتب الـ `apiClient.ts` كاملًا من البداية للنهاية بنفسك.
</details>
config.headers.Authorization  ='Bearer${token}'
ChatGPT
صح جدًا، أنت وصلت للمكان الصح، لكن عندك **غلطة واحدة في الـ syntax**.

أنت كتبت:

```ts
config.headers.Authorization = 'Bearer${token}'
```

المشكلة إن `' '` عبارة عن **String عادي**، وبالتالي `${token}` مش هتتفسر.

لازم تستخدم **Backticks**:

```ts
config.headers.Authorization = `Bearer ${token}`;
```

### ليه؟

```ts
'Bearer ${token}'
```

تتعامل معها JavaScript كنص حرفي:

```text
Bearer ${token}
```

لكن:

```ts
`Bearer ${token}`
```

تستخدم **Template Literal**، وبالتالي لو:

```ts
token = "abc123"
```

الناتج يكون:

```text
Bearer abc123
```

### وTypeScript هنا كمان شغال معاك

قبل الـ `if`:

```ts
const token = localStorage.getItem("token");
```

نوع `token` هو:

```ts
string | null
```

لكن بعد:

```ts
if (token) {
```

TypeScript يعرف إن `token` موجود، فيتعامل معه كـ:

```ts
string
```

وده اسمه **Type Narrowing**.

**تقييمك في الجزئية دي: 10/10 في الفكرة، فقط خطأ Syntax في الـ quotes.**

**الخطوة التالية:** اكتب الـ `apiClient.ts` كاملًا من البداية للنهاية بنفسك.
<details>
<summary>تمام</summary>
user
تمام
ChatGPT
اكتب الـ `apiClient.ts` كاملًا بنفسك في **5 دقائق**، ومن غير ما تبص على الحل السابق.

### 🗺️ Roadmap

```text
SECTION 4: Interceptors 🟢
  ├─ 4.1 What is an Interceptor    ✅
  ├─ 4.2 Request Interceptor       🟢 ← هنا
  │    ├─ axios.create()            ✅
  │    ├─ Axios Instance            ✅
  │    ├─ TypeScript typing         ✅
  │    ├─ Contextual Typing         ✅
  │    └─ Auth Header               🟢
  ├─ 4.3 Response Interceptor      ⬜
  ├─ 4.4 Authentication            ⬜
  ├─ 4.5 401 Handling              ⬜
  └─ 4.6 Token Refresh + Queue     ⬜
```

المطلوب منك تعمل:

1. `import axios`
2. إنشاء `api` باستخدام `axios.create()`
3. تحديد `baseURL`
4. تحديد `timeout`
5. إضافة `request interceptor`
6. قراءة `token` من `localStorage`
7. لو موجود، ضيفه في:
   ```ts
   Authorization: `Bearer ${token}`
   ```
8. في النهاية:
   ```ts
   return config;
   ```

**الخطوة التالية:** ابعتلي الكود كاملًا، وأنا هراجعه معاك سطر سطر.
</details>
تمام
ChatGPT
اكتب الـ `apiClient.ts` كاملًا بنفسك في **5 دقائق**، ومن غير ما تبص على الحل السابق.

### 🗺️ Roadmap

```text
SECTION 4: Interceptors 🟢
  ├─ 4.1 What is an Interceptor    ✅
  ├─ 4.2 Request Interceptor       🟢 ← هنا
  │    ├─ axios.create()            ✅
  │    ├─ Axios Instance            ✅
  │    ├─ TypeScript typing         ✅
  │    ├─ Contextual Typing         ✅
  │    └─ Auth Header               🟢
  ├─ 4.3 Response Interceptor      ⬜
  ├─ 4.4 Authentication            ⬜
  ├─ 4.5 401 Handling              ⬜
  └─ 4.6 Token Refresh + Queue     ⬜
```

المطلوب منك تعمل:

1. `import axios`
2. إنشاء `api` باستخدام `axios.create()`
3. تحديد `baseURL`
4. تحديد `timeout`
5. إضافة `request interceptor`
6. قراءة `token` من `localStorage`
7. لو موجود، ضيفه في:
   ```ts
   Authorization: `Bearer ${token}`
   ```
8. في النهاية:
   ```ts
   return config;
   ```

**الخطوة التالية:** ابعتلي الكود كاملًا، وأنا هراجعه معاك سطر سطر.
<details>
<summary>import axios from "axios";</summary>
user
import axios from "axios";

export const api = axios.create({
baseURL: "https://www.expmple.com/",
timeout:5000,
})

api.interceptor.request.use((config) => {
const token= localsrorage.getItim(token);
if(token){
config.headers.Authorization= `Bearer ${token}`
}
return config;
})
ChatGPT
صحح الكود بنفسك في **3 دقائق**. الفكرة المعمارية عندك صحيحة، لكن عندك **3 أخطاء Syntax/Typing بسيطة**.

### 🗺️ Roadmap

```text
SECTION 4: Interceptors 🟢
  ├─ 4.1 What is an Interceptor    ✅
  ├─ 4.2 Request Interceptor       🟢 ← هنا
  │    ├─ axios.create()            ✅
  │    ├─ Axios Instance            ✅
  │    ├─ TypeScript typing         ✅
  │    ├─ Contextual Typing         ✅
  │    └─ Auth Header               🟢
  ├─ 4.3 Response Interceptor      ⬜
  ├─ 4.4 Authentication            ⬜
  ├─ 4.5 401 Handling              ⬜
  └─ 4.6 Token Refresh + Queue     ⬜
```

### أخطاؤك

**1.**

كتبت:

```ts
api.interceptor.request.use
```

والصح:

```ts
api.interceptors.request.use
```

لاحظ `interceptors` بالـ **s**.

---

**2.**

كتبت:

```ts
localsrorage.getItim(token)
```

والصح:

```ts
localStorage.getItem("token")
```

هنا عندنا خطأين:

```text
localsrorage ❌
localStorage  ✅

getItim      ❌
getItem      ✅
```

وكمان `"token"` لازم تكون String.

---

**3.**

الـ `baseURL` عندك:

```ts
baseURL: "https://www.expmple.com/",
```

`expmple` مجرد typo، المفروض مثلًا:

```ts
baseURL: "https://www.example.com/",
```

### إذن الهيكل الصحيح المفروض يبقى:

```ts
import axios from "axios";

export const api = axios.create({
  baseURL: "https://www.example.com/",
  timeout: 5000,
});

api.interceptors.request.use((config) => {
  const token = localStorage.getItem("token");

  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }

  return config;
});
```

**تقييمك: 9/10** ⭐

أنت فاهم الـ flow بالكامل:

```text
api
 ↓
request interceptor
 ↓
get token
 ↓
if token exists
 ↓
Authorization header
 ↓
return config
 ↓
Backend
```

والأخطاء كلها تقريبًا **spelling / syntax**، مش مشكلة في فهم الـ concept.

**الخطوة التالية:** اكتب الكود مرة ثانية بعد التصحيح، من غير نسخ الحل، عشان أتأكد إن الـ syntax ثبت عندك.
</details>
import axios from "axios";

export const api = axios.create({
baseURL: "https://www.expmple.com/",
timeout:5000,
})

api.interceptor.request.use((config) => {
const token= localsrorage.getItim(token);
if(token){
config.headers.Authorization= `Bearer ${token}`
}
return config;
})
ChatGPT
صحح الكود بنفسك في **3 دقائق**. الفكرة المعمارية عندك صحيحة، لكن عندك **3 أخطاء Syntax/Typing بسيطة**.

### 🗺️ Roadmap

```text
SECTION 4: Interceptors 🟢
  ├─ 4.1 What is an Interceptor    ✅
  ├─ 4.2 Request Interceptor       🟢 ← هنا
  │    ├─ axios.create()            ✅
  │    ├─ Axios Instance            ✅
  │    ├─ TypeScript typing         ✅
  │    ├─ Contextual Typing         ✅
  │    └─ Auth Header               🟢
  ├─ 4.3 Response Interceptor      ⬜
  ├─ 4.4 Authentication            ⬜
  ├─ 4.5 401 Handling              ⬜
  └─ 4.6 Token Refresh + Queue     ⬜
```

### أخطاؤك

**1.**

كتبت:

```ts
api.interceptor.request.use
```

والصح:

```ts
api.interceptors.request.use
```

لاحظ `interceptors` بالـ **s**.

---

**2.**

كتبت:

```ts
localsrorage.getItim(token)
```

والصح:

```ts
localStorage.getItem("token")
```

هنا عندنا خطأين:

```text
localsrorage ❌
localStorage  ✅

getItim      ❌
getItem      ✅
```

وكمان `"token"` لازم تكون String.

---

**3.**

الـ `baseURL` عندك:

```ts
baseURL: "https://www.expmple.com/",
```

`expmple` مجرد typo، المفروض مثلًا:

```ts
baseURL: "https://www.example.com/",
```

### إذن الهيكل الصحيح المفروض يبقى:

```ts
import axios from "axios";

export const api = axios.create({
  baseURL: "https://www.example.com/",
  timeout: 5000,
});

api.interceptors.request.use((config) => {
  const token = localStorage.getItem("token");

  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }

  return config;
});
```

**تقييمك: 9/10** ⭐

أنت فاهم الـ flow بالكامل:

```text
api
 ↓
request interceptor
 ↓
get token
 ↓
if token exists
 ↓
Authorization header
 ↓
return config
 ↓
Backend
```

والأخطاء كلها تقريبًا **spelling / syntax**، مش مشكلة في فهم الـ concept.

**الخطوة التالية:** اكتب الكود مرة ثانية بعد التصحيح، من غير نسخ الحل، عشان أتأكد إن الـ syntax ثبت عندك.
<details>
<summary>بخصوص السنتاكس انا بحاول من الموبيل من input الدردشه</summary>
user
بخصوص السنتاكس انا بحاول من الموبيل من input الدردشه
ChatGPT
فاهمك، وده يفسر الأخطاء الإملائية اللي ظهرت. بما إنك بتكتب الكود من **الموبايل داخل الـ chat input**، مش هاعتبر أخطاء زي `getItim` أو `localsrorage` أو `interceptor` مشكلة في مستواك.

من دلوقتي في تقييم الكود هفصل بين حاجتين:

- 🧠 **Concept / Understanding**: هل فاهم ليه الكود بيتكتب كده؟
- ⌨️ **Syntax / Typing**: هل كتبت الكلمات والرموز صح؟

والتركيز الأساسي معاك هيكون على **الفهم والـ architecture والـ TypeScript reasoning**، خصوصًا إنك بتتعلم مش بتعمل Copy/Paste.

وبالنسبة للكود الأخير، فهمك للـ Request Interceptor ممتاز.

**الخطوة التالية:** نعمل سؤال Interview واحد على `Request Interceptor` ونقيّم فهمك، وبعدها نكمل `4.3 Response Interceptor`.
</details>
بخصوص السنتاكس انا بحاول من الموبيل من input الدردشه
ChatGPT
فاهمك، وده يفسر الأخطاء الإملائية اللي ظهرت. بما إنك بتكتب الكود من **الموبايل داخل الـ chat input**، مش هاعتبر أخطاء زي `getItim` أو `localsrorage` أو `interceptor` مشكلة في مستواك.

من دلوقتي في تقييم الكود هفصل بين حاجتين:

- 🧠 **Concept / Understanding**: هل فاهم ليه الكود بيتكتب كده؟
- ⌨️ **Syntax / Typing**: هل كتبت الكلمات والرموز صح؟

والتركيز الأساسي معاك هيكون على **الفهم والـ architecture والـ TypeScript reasoning**، خصوصًا إنك بتتعلم مش بتعمل Copy/Paste.

وبالنسبة للكود الأخير، فهمك للـ Request Interceptor ممتاز.

**الخطوة التالية:** نعمل سؤال Interview واحد على `Request Interceptor` ونقيّم فهمك، وبعدها نكمل `4.3 Response Interceptor`.
<details>
<summary>تمام</summary>
user
تمام
ChatGPT
جاوب على سؤال الـ **Interview** ده في **دقيقتين**، ومن غير ما تقلق من الـ syntax لأن إجابتك هتكون كلام.

### 🗺️ Roadmap

```text
SECTION 4: Interceptors 🟢
  ├─ 4.1 What is an Interceptor    ✅
  ├─ 4.2 Request Interceptor       🟢
  │    ├─ axios.create()            ✅
  │    ├─ Axios Instance            ✅
  │    ├─ TypeScript typing         ✅
  │    ├─ Contextual Typing         ✅
  │    └─ Auth Header               ✅
  ├─ 4.3 Response Interceptor      ⬜ ← بعد السؤال
  ├─ 4.4 Authentication            ⬜
  ├─ 4.5 401 Handling              ⬜
  └─ 4.6 Token Refresh + Queue     ⬜
```

### 🎯 Interview Question

عندك تطبيق فيه **50 API requests**، وكل Request محتاج يبعث:

```text
Authorization: Bearer <token>
```

ليه نستخدم **Request Interceptor** بدل ما نكتب الـ `Authorization` Header يدويًا في كل API call؟

وأين تضع الـ Interceptor في Architecture بتاعتك؟

جاوب كأنك في Interview، وبعدها هقيّم إجابتك من **10**.
</details>
تمام
ChatGPT
جاوب على سؤال الـ **Interview** ده في **دقيقتين**، ومن غير ما تقلق من الـ syntax لأن إجابتك هتكون كلام.

### 🗺️ Roadmap

```text
SECTION 4: Interceptors 🟢
  ├─ 4.1 What is an Interceptor    ✅
  ├─ 4.2 Request Interceptor       🟢
  │    ├─ axios.create()            ✅
  │    ├─ Axios Instance            ✅
  │    ├─ TypeScript typing         ✅
  │    ├─ Contextual Typing         ✅
  │    └─ Auth Header               ✅
  ├─ 4.3 Response Interceptor      ⬜ ← بعد السؤال
  ├─ 4.4 Authentication            ⬜
  ├─ 4.5 401 Handling              ⬜
  └─ 4.6 Token Refresh + Queue     ⬜
```

### 🎯 Interview Question

عندك تطبيق فيه **50 API requests**، وكل Request محتاج يبعث:

```text
Authorization: Bearer <token>
```

ليه نستخدم **Request Interceptor** بدل ما نكتب الـ `Authorization` Header يدويًا في كل API call؟

وأين تضع الـ Interceptor في Architecture بتاعتك؟

جاوب كأنك في Interview، وبعدها هقيّم إجابتك من **10**.
<details>
<summary>أنا بكتب requesting interceptors بدل ما أكتب authorization headers داخله عشان يكون مركزي، عشان أقدر أستخدمه في كل الـ API calls بدل ما أكتبه ويحصل فيه تكرار للكود، كمان أقدر أغيره وأتغيره مكان واحد. رقم اتنين هتحطه في الملف client مع الـ access instance في الملف shared component ملف client API client.</summary>
user
أنا بكتب requesting interceptors بدل ما أكتب authorization headers داخله عشان يكون مركزي، عشان أقدر أستخدمه في كل الـ API calls بدل ما أكتبه ويحصل فيه تكرار للكود، كمان أقدر أغيره وأتغيره مكان واحد. رقم اتنين هتحطه في الملف client مع الـ access instance في الملف shared component ملف client API client.
ChatGPT
إجابتك **ممتازة: 10/10**. 👏

### تقييم الـ Interview

**1. ليه Request Interceptor؟**  
إجابتك صحيحة تمامًا:

- Centralized
- يمنع تكرار الـ code
- يطبق الـ `Authorization` على كل الـ API calls
- لو غيرنا طريقة الـ authentication، نعدل في مكان واحد

**2. فين مكانه في Architecture؟**  
برضه صح:

```text id="m7v4kz"
src/
└── shared/
    └── api/
        └── apiClient.ts
```

وفي الملف:

```text id="p8m3yq"
Axios Instance
      +
Request Interceptor
```

والـ dependency direction:

```text id="v9w1hx"
Feature
   ↓
shared/api/apiClient
   ↓
axios
```

وده بالضبط اللي عايزينه في **Feature-Based Architecture**.

### نقطة مهمة جدًا

أنت قلت:

> "أقدر أغيره في مكان واحد"

ودي من أهم فوائد الـ **centralization**.

لو بعدين بدلنا:

```text
Authorization: Bearer <token>
```

بحاجة تانية، مش محتاج تلف على 50 API call.

تعدل الـ interceptor فقط.

### 🟢 Section 4.2 مكتملة

```text id="7d2pqa"
SECTION 4: Interceptors 🟢
  ├─ 4.1 What is an Interceptor    ✅
  ├─ 4.2 Request Interceptor       ✅
  │    ├─ axios.create()            ✅
  │    ├─ Axios Instance            ✅
  │    ├─ TypeScript typing         ✅
  │    ├─ Contextual Typing         ✅
  │    └─ Auth Header               ✅
  ├─ 4.3 Response Interceptor      ⬜ ← التالي
  ├─ 4.4 Authentication            ⬜
  ├─ 4.5 401 Handling              ⬜
  └─ 4.6 Token Refresh + Queue     ⬜
```

**الخطوة التالية:** نبدأ `4.3 Response Interceptor` ونفهم أولًا بالضبط **إمتى بيشتغل وإيه الفرق بينه وبين Request Interceptor**.
</details>
أنا بكتب requesting interceptors بدل ما أكتب authorization headers داخله عشان يكون مركزي، عشان أقدر أستخدمه في كل الـ API calls بدل ما أكتبه ويحصل فيه تكرار للكود، كمان أقدر أغيره وأتغيره مكان واحد. رقم اتنين هتحطه في الملف client مع الـ access instance في الملف shared component ملف client API client.
ChatGPT
إجابتك **ممتازة: 10/10**. 👏

### تقييم الـ Interview

**1. ليه Request Interceptor؟**  
إجابتك صحيحة تمامًا:

- Centralized
- يمنع تكرار الـ code
- يطبق الـ `Authorization` على كل الـ API calls
- لو غيرنا طريقة الـ authentication، نعدل في مكان واحد

**2. فين مكانه في Architecture؟**  
برضه صح:

```text id="m7v4kz"
src/
└── shared/
    └── api/
        └── apiClient.ts
```

وفي الملف:

```text id="p8m3yq"
Axios Instance
      +
Request Interceptor
```

والـ dependency direction:

```text id="v9w1hx"
Feature
   ↓
shared/api/apiClient
   ↓
axios
```

وده بالضبط اللي عايزينه في **Feature-Based Architecture**.

### نقطة مهمة جدًا

أنت قلت:

> "أقدر أغيره في مكان واحد"

ودي من أهم فوائد الـ **centralization**.

لو بعدين بدلنا:

```text
Authorization: Bearer <token>
```

بحاجة تانية، مش محتاج تلف على 50 API call.

تعدل الـ interceptor فقط.

### 🟢 Section 4.2 مكتملة

```text id="7d2pqa"
SECTION 4: Interceptors 🟢
  ├─ 4.1 What is an Interceptor    ✅
  ├─ 4.2 Request Interceptor       ✅
  │    ├─ axios.create()            ✅
  │    ├─ Axios Instance            ✅
  │    ├─ TypeScript typing         ✅
  │    ├─ Contextual Typing         ✅
  │    └─ Auth Header               ✅
  ├─ 4.3 Response Interceptor      ⬜ ← التالي
  ├─ 4.4 Authentication            ⬜
  ├─ 4.5 401 Handling              ⬜
  └─ 4.6 Token Refresh + Queue     ⬜
```

**الخطوة التالية:** نبدأ `4.3 Response Interceptor` ونفهم أولًا بالضبط **إمتى بيشتغل وإيه الفرق بينه وبين Request Interceptor**.
<details>
<summary>Response Interceptor</summary>
user
Response Interceptor
ChatGPT
ابدأ بـ **Response Interceptor** في حوالي **10 دقائق**: أهم حاجة دلوقتي تفهم مكانه في الـ request/response lifecycle قبل ما نكتب أي authentication logic.

### 🗺️ Roadmap

```text
SECTION 4: Interceptors 🟢
  ├─ 4.1 What is an Interceptor    ✅
  ├─ 4.2 Request Interceptor       ✅
  ├─ 4.3 Response Interceptor      🟢 ← هنا
  │    ├─ What is it?
  │    ├─ Response flow
  │    ├─ response vs response.data
  │    ├─ success handler
  │    └─ error handler
  ├─ 4.4 Authentication            ⬜
  ├─ 4.5 401 Handling              ⬜
  └─ 4.6 Token Refresh + Queue     ⬜
```
</details>
Response Interceptor
ChatGPT
ابدأ بـ **Response Interceptor** في حوالي **10 دقائق**: أهم حاجة دلوقتي تفهم مكانه في الـ request/response lifecycle قبل ما نكتب أي authentication logic.

### 🗺️ Roadmap

```text
SECTION 4: Interceptors 🟢
  ├─ 4.1 What is an Interceptor    ✅
  ├─ 4.2 Request Interceptor       ✅
  ├─ 4.3 Response Interceptor      🟢 ← هنا
  │    ├─ What is it?
  │    ├─ Response flow
  │    ├─ response vs response.data
  │    ├─ success handler
  │    └─ error handler
  ├─ 4.4 Authentication            ⬜
  ├─ 4.5 401 Handling              ⬜
  └─ 4.6 Token Refresh + Queue     ⬜
```