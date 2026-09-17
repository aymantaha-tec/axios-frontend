# 5.2 API Client

خلينا نبدأ بالمشكلة.

ممكن تعمل في كل API:

```ts
axios.get("https://api.example.com/users");
```

وبعدين:

```ts
axios.get("https://api.example.com/products");
```

وبعدين:

```ts
axios.post("https://api.example.com/orders");
```

هنا بدأنا نكرر:

```text
baseURL
timeout
headers
Authorization
interceptors
```

وده مش كويس.

## الحل: Axios Instance

نعمل **API Client واحد**:

```ts
import axios from "axios";

export const api = axios.create({
  baseURL: "https://api.example.com",
  timeout: 5000,
});
```

الـ `api` هنا هو **Axios Instance** خاص بتطبيقنا.

وبدل:

```ts
axios.get("/users");
```

نستخدم:

```ts
api.get("/users");
```

---

# طيب إيه اللي كسبناه؟

### بدون API Client

```text
Component A
   ↓
axios + baseURL + headers

Component B
   ↓
axios + baseURL + headers

Component C
   ↓
axios + baseURL + headers
```

كل مكان ممكن يعمل Configuration مختلف.

---

### باستخدام API Client

```text
             ┌─────────────┐
Users API ──→│             │
             │ API Client  │──→ Backend
Orders API ─→│             │
             │             │
Products ───→│             │
             └─────────────┘
```

والـ Configuration المشتركة موجودة في مكان واحد.

---

# إيه اللي يتحط في API Client؟

الحاجات **المشتركة بين معظم أو كل Requests**.

مثلاً:

```ts
export const api = axios.create({
  baseURL: "https://api.example.com",
  timeout: 5000,
});
```

وبعدها ممكن نضيف:

```text
baseURL
timeout
request interceptors
response interceptors
common headers
authentication behavior
common error behavior
```

لكن خد بالك من حاجة مهمة:

**مش معنى إننا عندنا API Client إن كل Configuration لازم يتحط فيه.**

مثلاً:

```ts
api.get("/users", {
  params: {
    page: 2,
  },
});
```

الـ `page` هنا خاص بالـ Request ده، فمكانه مش الـ API Client.

---

# مكانه في Architecture

بناءً على الـ Feature-Based Architecture اللي اتعلمناها:

```text
src/
│
├── shared/
│   └── api/
│       └── apiClient.ts
│
└── features/
    └── users/
        └── api/
            └── userApi.ts
```

### `apiClient.ts`

يعرف **كيف نتعامل مع HTTP**:

```ts
import axios from "axios";

export const api = axios.create({
  baseURL: "https://api.example.com",
  timeout: 5000,
});
```

### `userApi.ts`

يعرف **إيه الـ API operation الخاصة بالـ Users**:

```ts
import { api } from "@/shared/api/apiClient";

export const getUser = async (id: number) => {
  const response = await api.get(`/users/${id}`);

  return response.data;
};
```

لاحظ الاتجاه:

```text
User Feature
     ↓
Shared API Client
     ↓
Axios
     ↓
Backend
```

وده متوافق مع **Dependency Direction** اللي اتعلمناها في Section 0.

---

## 🔥 أهم فرق في الدرس

لا تخلط بين الاتنين:

```text
API Client
= البنية المشتركة للاتصال بالـ Backend

Feature API
= العمليات الخاصة بالـ Feature
```

مثال:

```text
apiClient.ts
→ baseURL
→ timeout
→ interceptors


userApi.ts
→ getUser()
→ getUsers()
→ createUser()
→ updateUser()
→ deleteUser()
```

### 🎯 سؤال Interview

عندك 30 API مختلفة في المشروع.

ليه **مش الأفضل** إن كل API تعمل:

```ts
axios.get("https://api.example.com/...");
```

بدل ما نعمل Axios Instance واحدة اسمها `api`؟

**جاوبني من ناحية Architecture، مش من ناحية syntax.**
<details>
<summary>جاوب انت بس عباره عن نقاط مختصره</summary>
user
جاوب انت بس عباره عن نقاط مختصره
ChatGPT
- **تجنب تكرار الـ Configuration** مثل `baseURL` و`timeout` و`headers`.
- **مركزية الـ Interceptors** في مكان واحد.
- **توحيد طريقة الاتصال بالـ API** في المشروع.
- **تقليل الـ Coupling** بين الـ Features وAxios.
- **سهولة التعديل**: لو الـ `baseURL` اتغير، تعدله مرة واحدة.
- **سهولة الصيانة والاختبار**.
- كل Feature تستخدم نفس الـ `API Client` بدل ما تعمل Configuration خاص بها.
</details>
جاوب انت بس عباره عن نقاط مختصره
ChatGPT
- **تجنب تكرار الـ Configuration** مثل `baseURL` و`timeout` و`headers`.
- **مركزية الـ Interceptors** في مكان واحد.
- **توحيد طريقة الاتصال بالـ API** في المشروع.
- **تقليل الـ Coupling** بين الـ Features وAxios.
- **سهولة التعديل**: لو الـ `baseURL` اتغير، تعدله مرة واحدة.
- **سهولة الصيانة والاختبار**.
- كل Feature تستخدم نفس الـ `API Client` بدل ما تعمل Configuration خاص بها.
<details>
<summary>كمل</summary>
user
كمل
ChatGPT
**خلال 15 دقيقة هنكمل 5.3 `Feature API` ونربطها عمليًا بالـ `API Client`.**

### 🗺️ Roadmap

```text
SECTION 5  API Architecture 🟢
 ├─ 5.1 API Layer vs Component       ✅
 ├─ 5.2 API Client                   ✅
 ├─ 5.3 Feature API                  🟢 ← هنا
 ├─ 5.4 Service Layer                ⬜
 ├─ 5.5 Dependency Direction         ⬜
 └─ 5.6 Complete API Architecture   ⬜
```
</details>
كمل
ChatGPT
**خلال 15 دقيقة هنكمل 5.3 `Feature API` ونربطها عمليًا بالـ `API Client`.**

### 🗺️ Roadmap

```text
SECTION 5  API Architecture 🟢
 ├─ 5.1 API Layer vs Component       ✅
 ├─ 5.2 API Client                   ✅
 ├─ 5.3 Feature API                  🟢 ← هنا
 ├─ 5.4 Service Layer                ⬜
 ├─ 5.5 Dependency Direction         ⬜
 └─ 5.6 Complete API Architecture   ⬜
```