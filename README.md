خطة تعلم Axios الاحترافية
user
عاوز اتعلم axios بطريقة بروفيشنال مع التدريب 
وهنراعي تايب اسكريبت تلوند سي اس اس وشاد سي ان 
وكمان feature based arctichure 
and storyboard
عباره عن اقسام رئيسية وفرعية ودروس  وبروجرس تريكر واسئلة انتر فيو كانك عندك خيره 50 سنة 
ChatGPT
ابدأ من **Section 1: Axios Fundamentals**، وهنبني كورس كامل من الصفر لحد مستوى Production، مع **TypeScript + Tailwind CSS + shadcn/ui + Feature-Based Architecture + Storybook + تدريب عملي + Interview Questions + Progress Tracker**. المدة المقترحة: **6 إلى 8 أسابيع، بمعدل 60 إلى 90 دقيقة يوميًا**.

## 🎯 الهدف النهائي

في نهاية المسار تكون قادر تعمل **API Layer احترافية** داخل مشروع React حقيقي، بالشكل ده تقريبًا:

```text
src/
│
├── app/
│   ├── router/
│   ├── providers/
│   └── config/
│
├── features/
│   │
│   ├── auth/
│   │   ├── api/
│   │   │   ├── auth.api.ts
│   │   │   └── auth.types.ts
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── pages/
│   │   ├── schemas/
│   │   └── types/
│   │
│   ├── users/
│   └── products/
│
├── shared/
│   ├── api/
│   │   ├── axios.ts
│   │   ├── interceptors.ts
│   │   └── errors.ts
│   ├── components/
│   ├── hooks/
│   └── types/
│
└── main.tsx
```

والهدف مش إنك تحفظ:

```ts
axios.get()
axios.post()
```

الهدف إنك **تفهم لماذا صممنا الـ API layer بهذه الطريقة، ومتى نستخدم كل abstraction، وما الذي يحدث تحت الغطاء**.

---

# 🗺️ AXIOS PROFESSIONAL ROADMAP

## SECTION 0: Architecture Foundation

### 0.1 HTTP قبل Axios
- HTTP Request / Response
- Methods
- Headers
- Body
- Query Parameters
- Status Codes
- REST API
- JSON
- Authentication

### 0.2 لماذا Axios؟
- Fetch vs Axios
- Axios abstraction
- Axios Request Lifecycle
- Axios Response Lifecycle
- AxiosError

### 0.3 TypeScript Foundation
- Generics
- Union Types
- Intersection Types
- Type Guards
- `unknown`
- `never`
- Utility Types
- `typeof`
- `keyof`
- Indexed Access Types

### 0.4 Architecture
- Layered Architecture
- Feature-Based Architecture
- Shared vs Feature
- Separation of Concerns
- Dependency Direction
- Coupling / Cohesion

### 🎤 Interview

```text
What problem does Axios solve?

Why use Axios instead of fetch?

What is the difference between HTTP errors
and JavaScript errors?

What is separation of concerns?

Why shouldn't every component call Axios directly?
```

---

# SECTION 1: Axios Fundamentals

## Lesson 1: Axios Request

هنتعلم:

```ts
axios.get()
axios.post()
axios.put()
axios.patch()
axios.delete()
```

ونفهم:

```ts
const response = await axios.get("/users");
```

إيه هو:

```text
axios
   ↓
request
   ↓
HTTP
   ↓
server
   ↓
response
   ↓
AxiosResponse
```

---

## Lesson 2: Request Configuration

```ts
axios.get("/users", {
  params: {},
  headers: {},
  timeout: 5000,
});
```

هندرس بالتفصيل:

- `baseURL`
- `headers`
- `params`
- `data`
- `timeout`
- `signal`
- `withCredentials`
- `responseType`

---

## Lesson 3: Axios Instance

هنا هننقل من:

```ts
axios.get(...)
```

إلى:

```ts
const api = axios.create({
  baseURL: "...",
});
```

ونفهم **ليه الـ instance من أهم الحاجات في Architecture**.

---

# SECTION 2: TypeScript + Axios

## Lesson 4: AxiosResponse

```ts
const response = await api.get<User[]>("/users");
```

نفهم:

```ts
AxiosResponse<T>
```

والفرق بين:

```ts
response
response.data
response.status
response.headers
```

---

## Lesson 5: Generic API Responses

هنبني:

```ts
interface SuccessResponse<T> {
  status: true;
  code: number;
  payload: T;
}
```

و:

```ts
interface ErrorResponse<T> {
  status: false;
  code: number;
  message: string;
  errors: T[];
}
```

ثم:

```ts
type ApiResponse<T, E = unknown> =
  | SuccessResponse<T>
  | ErrorResponse<E>;
```

وهنا هندخل في موضوع مهم جدًا:

### Discriminated Unions

وليه:

```ts
status: true
```

ممكن يخلي TypeScript يفهم الـ response تلقائيًا.

---

# SECTION 3: Error Handling

## Lesson 6: AxiosError

هنتعلم:

```ts
AxiosError
```

والفرق بين:

```ts
error.message
error.response
error.request
error.config
```

ثم:

```ts
axios.isAxiosError(error)
```

---

## Lesson 7: Custom ApiError

هنبني:

```ts
class ApiError<T = unknown> extends Error {
  status?: number;
  data?: T;
}
```

وهنا هتعرف **ليه أصلًا نعمل Custom Error** بدل ما نرمي:

```ts
new Error(...)
```

---

# SECTION 4: Interceptors

## Lesson 8: Request Interceptor

```ts
api.interceptors.request.use(...)
```

استخداماته:

```text
Authorization
Request ID
Locale
Headers
Logging
```

مثال:

```ts
config.headers.Authorization = `Bearer ${token}`;
```

---

## Lesson 9: Response Interceptor

```ts
api.interceptors.response.use(...)
```

هندرس:

```text
2xx
4xx
5xx
network errors
```

---

## Lesson 10: Token Refresh

وده من أهم الدروس.

هنتعامل مع:

```text
Access Token
Refresh Token
401
Refresh
Retry original request
```

والـ Queue problem:

```text
Request A → 401
Request B → 401
Request C → 401
```

مش عايزين نعمل:

```text
Refresh
Refresh
Refresh
```

لكن:

```text
Refresh
   ↓
Queue A
Queue B
Queue C
   ↓
Retry
```

دي من الحاجات اللي بتفرق بين Junior implementation و Production implementation.

---

# SECTION 5: API Architecture

## Lesson 11: API Layer

بدل:

```tsx
function Login() {
  axios.post(...)
}
```

هنوصل إلى:

```text
Component
   ↓
Hook
   ↓
Feature API
   ↓
Axios Instance
   ↓
Server
```

---

## Lesson 12: Feature-Based Architecture

مثلاً:

```text
features/
├── auth/
├── users/
├── products/
└── orders/
```

كل Feature عنده:

```text
api
components
hooks
pages
schemas
types
```

وندخل في:

- Public API
- Internal API
- Feature boundaries
- Shared code
- Circular dependencies

---

# SECTION 6: Axios + TanStack Query

بما إنك بتتعلم TanStack Query، هنربطه بالـ Axios صح.

```text
Component
     ↓
useQuery
     ↓
queryFn
     ↓
Feature API
     ↓
Axios
     ↓
Backend
```

هنتعلم:

- Query Functions
- Mutation Functions
- Query Keys
- `staleTime`
- `gcTime`
- Retry
- Error handling
- Invalidation
- Optimistic Updates
- Pagination
- Infinite Queries

---

# SECTION 7: Authentication Architecture

هنبني Authentication حقيقي:

```text
Login
 ↓
Access Token
 ↓
Refresh Token
 ↓
Axios Interceptor
 ↓
401
 ↓
Refresh
 ↓
Retry
```

مع:

- Protected Routes
- Logout
- Session expiration
- Persistent authentication
- Race conditions
- Multiple tabs

---

# SECTION 8: Forms

ربط:

```text
React Hook Form
+
Zod
+
Axios
+
TanStack Query
+
shadcn/ui
```

مثال:

```text
LoginForm
   ↓
React Hook Form
   ↓
Zod
   ↓
Mutation
   ↓
API
   ↓
Axios
   ↓
Server
```

---

# SECTION 9: Tailwind CSS + shadcn/ui

مش هنتعلمهم كـ CSS course منفصل.

هنستخدمهم **داخل المشروع**.

هنبني:

```text
Button
Input
Form
Dialog
Dropdown
Table
Toast
Alert
Skeleton
Pagination
```

ونربطهم بالـ API states:

```text
Loading
Success
Error
Empty
Retry
Disabled
```

---

# SECTION 10: Storybook

هنا هنفصل UI عن الـ Backend.

مثلاً:

```text
UserTable
```

نعمل له Stories:

```text
Default
Loading
Empty
Error
WithData
LongNames
Pagination
```

ونتعلم:

- Component isolation
- Args
- Controls
- Mocking API
- MSW
- Visual states
- Component documentation

---

# SECTION 11: Testing

هنضيف:

```text
Vitest
+
Testing Library
+
MSW
```

ونختبر:

```text
API
Error handling
Hooks
Forms
Components
Authentication
```

---

# SECTION 12: Production Architecture

هنا نبدأ مستوى Senior.

هنتكلم عن:

- Environment Variables
- API Configuration
- Request cancellation
- AbortController
- Race Conditions
- Retry Strategy
- Request Deduplication
- Error normalization
- Logging
- Monitoring
- Security
- CORS
- CSRF
- XSS
- Token Storage
- Performance

---

# SECTION 13: Real Project

هنبني مشروع كامل:

## 🏗️ Admin Dashboard

```text
Authentication
Users
Products
Orders
Dashboard
Profile
Settings
```

Architecture:

```text
React
│
├── TypeScript
├── Axios
├── TanStack Query
├── React Hook Form
├── Zod
├── React Router
├── Tailwind
├── shadcn/ui
├── Storybook
├── Vitest
└── MSW
```

---

# 🧠 التدريب

كل درس هيبقى بالشكل ده:

```text
[CONCEPT]
     ↓
[WHY]
     ↓
[HOW]
     ↓
[CODE]
     ↓
[EXERCISE]
     ↓
[DEBUGGING]
     ↓
[INTERVIEW]
     ↓
[REAL WORLD]
```

يعني مش هقولك:

> اعمل Axios Instance.

هسألك:

> عندك 17 Feature وكل Feature بتكرر baseURL وheaders وtimeout. إيه المشكلة المعمارية هنا؟

وبعدين نخليك **أنت اللي تستنتج الحل**.

---

# 🎯 Progress Tracker

هنستخدم Tracker بالشكل ده:

| Section | Status |
|---|---|
| HTTP Foundation | ⬜ |
| Axios Fundamentals | ⬜ |
| TypeScript + Axios | ⬜ |
| Error Handling | ⬜ |
| Interceptors | ⬜ |
| Token Refresh | ⬜ |
| API Architecture | ⬜ |
| Feature Architecture | ⬜ |
| TanStack Query | ⬜ |
| Authentication | ⬜ |
| Forms | ⬜ |
| Tailwind | ⬜ |
| shadcn/ui | ⬜ |
| Storybook | ⬜ |
| Testing | ⬜ |
| Production | ⬜ |
| Final Project | ⬜ |

وكل درس هيتسجل:

```text
🟢 Completed
🟡 In Progress
🔴 Needs Review
```

---

# 🎤 Interview System

بعد كل Section:

### Junior

```text
What is Axios?
```

### Mid-Level

```text
Why would you create an Axios instance?
```

### Senior

```text
How would you design an HTTP client
for a large React application?
```

### Staff-Level

```text
How would you prevent concurrent
token refresh requests?
```

ثم **مش هديك الإجابة فورًا**.

هخليك تجاوب، وبعدها:

```text
Your Answer
     ↓
Evaluation
     ↓
What's Missing
     ↓
Senior Answer
     ↓
Production Considerations
```

---

# 🧩 الـ Storyboard بتاع الرحلة

```text
HTTP
 │
 ▼
Axios
 │
 ▼
Axios Instance
 │
 ▼
TypeScript
 │
 ▼
Errors
 │
 ▼
Interceptors
 │
 ▼
Authentication
 │
 ▼
API Layer
 │
 ▼
Feature Architecture
 │
 ▼
TanStack Query
 │
 ▼
Forms
 │
 ▼
UI States
 │
 ▼
Storybook
 │
 ▼
Testing
 │
 ▼
Production Architecture
 │
 ▼
REAL PROJECT
```

**أهم قاعدة:** مش هنعلم Axios كـ library منفصلة، هنعلمك **إزاي تصمم HTTP communication layer محترمة في React + TypeScript application**.

**الخطوة التالية: ابدأ بـ `SECTION 0 - Lesson 1: HTTP Request/Response`، واكتب لي: `ابدأ Lesson 1`.**
user
1
ChatGPT

---

# 📊 Progress Tracker

```text
SECTION 1: Axios Fundamentals

GET                  ✅
POST                 ✅
PUT                  ✅
PATCH                ✅
DELETE               ✅

Current:
Request Configuration ⬅️
```

**الخطوة التالية: نبدأ `Request Configuration`، وأول حاجة نفهمها هي الـ `config object` في Axios.**
user
Request Configuration  
ChatGPT
**الهدف: نفهم `Request Configuration` في Axios من الأساس، وأول جزء فقط هو الـ `config object`، بدون الدخول في `headers` أو `params` بالتفصيل لسه.** الوقت: **10 دقائق**.

# 📍 Roadmap

```text
SECTION 1: Axios Fundamentals 🟢

GET                  ✅
POST                 ✅
PUT                  ✅
PATCH                ✅
DELETE               ✅

Request Configuration 🟢 ← نحن هنا
├── Config Object     🟢
├── params            ⬜
├── headers           ⬜
├── data              ⬜
├── timeout           ⬜
└── signal            ⬜
```


---

# SECTION 0: Architecture Foundation

### 0.1 Architecture

تنظيم أجزاء التطبيق وعلاقاتها ببعض.

```text
Component
   ↓
API
   ↓
Backend
```

### 0.2 Separation of Concerns

كل جزء له مسؤولية محددة.

```text
Component → UI
API       → API Communication
Service   → Business Logic
```

### 0.3 Coupling

مدى اعتماد جزء على جزء آخر.

```text
Low Coupling ✅
High Coupling ❌
```

### 0.4 Cohesion

مدى ترابط المسؤوليات داخل نفس الـ Module.

```text
High Cohesion ✅
```

### 0.5 Feature-Based Architecture

تنظيم المشروع حسب الـ Features:

```text
features/
├── users/
├── products/
└── orders/
```

### 0.6 Shared vs Feature

```text
UserCard      → users/components
Button        → shared/components

getUser       → users/api
formatMoney   → shared/utils
```

القاعدة:

> لو الكود يعرف Domain معين، غالبًا Feature.  
> لو Generic وقابل لإعادة الاستخدام، غالبًا Shared.

### 0.7 Public vs Internal API

```text
Public
↓
الحاجات اللي تسمح لباقي المشروع باستخدامها

Internal
↓
Implementation Details
```

### 0.8 Dependency Direction

```text
Feature
   ↓
Shared
```

```text
Shared
   ↓
Feature ❌
```

Feature-to-Feature ممكن، لكن بحذر.

---

# SECTION 1: Axios Fundamentals

## HTTP Request

```text
Method
URL
Headers
Body
```

## HTTP Response

```text
Status
Headers
Body
```

### Methods

```text
GET     → Read
POST    → Create
PUT     → Replace
PATCH   → Partial Update
DELETE  → Delete
```

### Axios

```ts
const response = await axios.get("/users");
```

النتيجة:

```text
response
   ↓
AxiosResponse

response.data
   ↓
Actual Response Data
```

### Generic

```ts
axios.get<IUser>("/users/10");
```

معناه:

> TypeScript يتوقع أن `response.data` يكون `IUser`.

مش Runtime Validation.

### Request Config

```ts
axios.get("/products", {
  params: {
    page: 1,
    limit: 20,
  },
  headers: {
    Authorization: "Bearer token",
  },
  timeout: 5000,
  signal,
});
```

القاعدة:

```text
params  → URL
data    → Request Body
headers → Metadata
```

---

# SECTION 2: TypeScript + Axios

## AxiosResponse

```ts
const response = await axios.get<IUser>("/users/10");
```

```text
response
→ AxiosResponse<IUser>

response.data
→ IUser
```

ولو:

```ts
axios.get<IUser[]>()
```

يبقى:

```text
response.data
→ IUser[]
```

---

## API Response Wrapper

لو Backend يرجع:

```json
{
  "status": true,
  "code": 200,
  "payload": {}
}
```

نعمل:

```ts
interface IApiResponse<T> {
  status: boolean;
  code: number;
  payload: T;
}
```

وبالتالي:

```ts
axios.get<IApiResponse<IUser>>()
```

التدفق:

```text
response
↓
AxiosResponse<IApiResponse<IUser>>

response.data
↓
IApiResponse<IUser>

response.data.payload
↓
IUser
```

---

## Discriminated Union

```ts
type IApiResponse<T, R> =
  | ISuccessResponse<T>
  | IErrorResponse<R>;
```

والـ discriminator:

```ts
status: true
```

مقابل:

```ts
status: false
```

وده يسمح لـ TypeScript يعمل **Narrowing**.

---

## `unknown` vs `any`

```text
unknown
→ لازم تعمل Narrowing

any
→ TypeScript تقريبًا يتوقف عن حمايتك
```

للبيانات الخارجية، `unknown` أكثر أمانًا.

---

## Type-Safe API Layer

بدل ما Component يتعامل مباشرة مع Axios:

```ts
export const getUser = async (
  id: number
): Promise<IUser> => {
  const response =
    await api.get<IApiResponse<IUser>>(
      `/users/${id}`
    );

  return response.data.payload;
};
```

الـ Component يحصل في النهاية على:

```text
IUser
```

مش AxiosResponse.

---

# SECTION 3: Error Handling

## HTTP Error

الـ Backend رد، لكن Status Code يدل على فشل.

```text
404
401
403
500
```

Axios عادةً يرفض الـ Promise مع non-2xx.

---

## Network Error

مفيش HTTP Response usable.

```text
Internet failure
DNS
Server unreachable
```

لذلك:

```ts
error.response
```

ممكن تكون `undefined`.

---

## AxiosError

ممكن يحتوي:

```text
message
response
request
config
code
```

التمييز المهم:

```text
error.response?.status
→ HTTP Status

error.response?.data
→ Backend Response Body

error.response?.data.code
→ Backend/API Code

error.code
→ Axios Error Code
```

---

## Error Normalization

بدل ما كل Component يفهم أنواع Errors مختلفة:

```text
AxiosError
Network Error
Timeout
Abort
Unknown
```

نعمل:

```text
Different Errors
      ↓
normalizeError()
      ↓
INormalizedError
```

مثلاً:

```ts
interface INormalizedError {
  message: string;
  status?: number;
  code?: string;
}
```

---

## Custom ApiError

الفكرة:

```text
AxiosError
    ↓
normalize
    ↓
ApiError
    ↓
Application
```

الهدف تقليل اعتماد الـ Application مباشرة على Axios.

---

# SECTION 4: Interceptors

## Interceptor

نقطة تدخل في رحلة الـ Request أو Response.

```text
Request:

Component
   ↓
Request Interceptor
   ↓
Backend
```

```text
Response:

Backend
   ↓
Response Interceptor
   ↓
Caller
```

---

## Request Interceptor

مثلاً إضافة Authorization:

```ts
api.interceptors.request.use((config) => {
  const token = localStorage.getItem("token");

  if (token) {
    config.headers.Authorization =
      `Bearer ${token}`;
  }

  return config;
});
```

بدل:

```text
50 API Calls
↓
50 مرة Authorization
```

نخليها:

```text
Request Interceptor
↓
Authorization
```

---

## Response Interceptor

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

الجزء الأول:

```text
Success
```

الجزء الثاني:

```text
Error
```

`return response` يعني تمرير الـ Axios Response للمستوى التالي.

---

## Authentication

### Authentication

```text
Who are you?
```

### Authorization

```text
What are you allowed to do?
```

---

## Access Token

يستخدم للوصول إلى API:

```http
Authorization: Bearer <access-token>
```

مش Token جديد لكل Request.

نستخدم الـ Access Token الحالي إلى أن ينتهي أو يصبح غير صالح.

---

## Refresh Token

عند انتهاء Access Token:

```text
Access Token
   ↓
Expired
   ↓
401
   ↓
Refresh Token
   ↓
New Access Token
```

---

# Token Refresh + Queue

لو عندنا:

```text
A → 401
B → 401
C → 401
D → 401
E → 401
```

مش عايزين:

```text
A → Refresh
B → Refresh
C → Refresh
D → Refresh
E → Refresh
```

نريد:

```text
A → يبدأ Refresh

B → ينتظر
C → ينتظر
D → ينتظر
E → ينتظر

        ↓
New Access Token
        ↓
A B C D E
        ↓
Retry
```

## `refreshPromise`

مهم جدًا:

```text
refreshPromise
≠ Access Token

refreshPromise
≠ Refresh Token
```

هو:

> العملية الحالية الخاصة بالـ Refresh.

```ts
if (!refreshPromise) {
  refreshPromise = refreshAccessToken();
}
```

والباقي:

```ts
const newToken = await refreshPromise;
```

---

## Retry Protection

```ts
if (originalRequest._retry) {
  return Promise.reject(error);
}

originalRequest._retry = true;
```

هدفها منع:

```text
401
 ↓
Refresh
 ↓
Retry
 ↓
401
 ↓
Refresh
 ↓
Retry
 ↓
∞
```

---

## Refresh Endpoint Protection

دي مختلفة عن `_retry`.

لو:

```text
/auth/refresh
```

نفسه رجع `401`، لازم نمنع الـ Interceptor من محاولة Refresh مرة أخرى.

```text
Original Request
 ↓
401
 ↓
Refresh
 ↓
401
 ↓
❌ Stop
```

---

## `refreshPromise = null`

بعد انتهاء عملية الـ Refresh:

```ts
refreshPromise = null;
```

معناها:

> لا توجد عملية Refresh شغالة حاليًا.

**مش معناها حذف الـ Access Token.**

---

# SECTION 5: API Architecture

## الطبقات الأساسية

```text
Component
   ↓
Service Layer
   ↓
Feature API
   ↓
API Client
   ↓
Axios
   ↓
Backend
```

---

## API Client

```ts
export const api = axios.create({
  baseURL: "...",
  timeout: 5000,
});
```

مسؤوليته:

```text
baseURL
timeout
interceptors
common HTTP configuration
```

---

## Feature API

مثلاً:

```text
features/
└── users/
    └── api/
        └── userApi.ts
```

وفيه:

```ts
getUser()
getUsers()
createUser()
updateUser()
deleteUser()
```

مسؤوليته:

> API Operations الخاصة بالـ Feature.

---

## Service Layer

مسؤوليته:

> **Business Logic**

مثلاً:

```text
purchaseProduct()
      ↓
getUser()
      ↓
getProduct()
      ↓
Check Balance
      ↓
createOrder()
```

الـ Service لا يتعامل مباشرة مع Axios.

---

# ⭐ الفرق النهائي

```text
Component
→ UI

Service
→ Business Logic

Feature API
→ API Operations

API Client
→ HTTP Configuration

Axios
→ HTTP Library

Backend
→ Server
```

---

# 📊 تقييمك الحالي

بناءً على الاختبارات والإجابات اللي عملناها، مش مجرد عدد الدروس:

```text
Architecture Foundation     🟢 قوي
Axios Fundamentals          🟢 قوي
TypeScript + Axios          🟢 قوي
Error Handling              🟢 قوي
Interceptors                🟢 قوي
API Architecture            🟢 جيد جدًا
```

**أهم حاجة محتاج تركز عليها الآن مش Concepts جديدة، وإنما تحويل الفهم إلى كتابة Code من الذاكرة، خصوصًا `Token Refresh + Queue` والـ Architecture.**

### 📍 موقفك

```text
Section 0  ████████████████████ 100% ✅
Section 1  ████████████████████ 100% ✅
Section 2  ████████████████████ 100% ✅
Section 3  ████████████████████ 100% ✅
Section 4  ████████████████████ 100% ✅
Section 5  ████████████████████ 100% ✅

Overall    ███████░░░░░░░░░░░░░  ~36%
```

**الخطوة التالية: هنبدأ Section 6 `TanStack Query` بعد المراجعة، وأول Concept هو `Server State vs Client State` لمدة 20 دقيقة.**
user
Section 6 TanStack Query
ChatGPT
**خلال 20 دقيقة هنبدأ Section 6 بـ `Server State vs Client State`، وده أهم Concept قبل ما ندخل `useQuery` و`QueryClient`.**

### 🗺️ Roadmap

```text
SECTION 0  Architecture Foundation     ✅
SECTION 1  Axios Fundamentals          ✅
SECTION 2  TypeScript + Axios          ✅
SECTION 3  Error Handling              ✅
SECTION 4  Interceptors                ✅
SECTION 5  API Architecture            ✅

SECTION 6  TanStack Query              🟢
 ├─ 6.1 Server State vs Client State   🟢 ← هنا
 ├─ 6.2 QueryClient                    ⬜
 ├─ 6.3 QueryClientProvider            ⬜
 ├─ 6.4 useQuery                       ⬜
 ├─ 6.5 queryKey                       ⬜
 ├─ 6.6 queryFn                        ⬜
 ├─ 6.7 Loading / Error States         ⬜
 ├─ 6.8 staleTime / gcTime             ⬜
 ├─ 6.9 Refetching                     ⬜
 ├─ 6.10 useMutation                   ⬜
 ├─ 6.11 invalidateQueries              ⬜
 └─ 6.12 Advanced Patterns             ⬜

SECTION 7  Authentication Architecture ⬜
SECTION 8  Forms                        ⬜
SECTION 9  Tailwind + shadcn/ui         ⬜
SECTION 10 Storybook                    ⬜
SECTION 11 Testing                      ⬜
SECTION 12 Production Architecture     ⬜
SECTION 13 Final Project                ⬜
```
