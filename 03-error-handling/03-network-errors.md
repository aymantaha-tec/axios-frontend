# Network Errors

قبل كده قلنا:

### HTTP Error

السيرفر **وصل له الـ request ورد علينا**:

```text
Client
  ↓
Request
  ↓
Server
  ↓
404 / 401 / 500
  ↓
HTTP Error
```

أما **Network Error** فالمشكلة بتحصل في الاتصال نفسه.

```text
Client
  ↓
Request
  ↓
❌ Communication failed
```

يعني ببساطة:

> **We couldn't successfully communicate with the server.**

---

## مثال

أنت تعمل:

```ts
try {
  const response = await axios.get("/users");
} catch (error) {
  console.log(error);
}
```

ممكن يحصل إن:

```text
Internet connection
       ↓
      ❌
```

أو السيرفر غير قابل للوصول:

```text
Client
   ↓
   ❌
Server
```

في الحالة دي أنت **لم تحصل على HTTP Response طبيعي من السيرفر**.

لذلك مفيش:

```text
404
401
500
```

لازم تكون وصلت لمرحلة إن السيرفر يرد أصلًا عشان نقول عندنا HTTP status.

---

# قارن الاثنين

### HTTP Error

```text
Request
   ↓
Server
   ↓
Response
   ↓
404
```

السيرفر رد، لكن الرد يقول إن العملية فشلت.

### Network Error

```text
Request
   ↓
❌
No usable HTTP Response
```

الاتصال نفسه فشل.

---

# مثال واقعي

تخيل إنك بتطلب:

```ts
axios.get("/users");
```

### الحالة الأولى

السيرفر موجود:

```text
GET /users
      ↓
Server
      ↓
404 Not Found
```

ده:

```text
HTTP Error
```

### الحالة الثانية

الإنترنت فصل:

```text
GET /users
      ↓
   Internet
      ↓
      ❌
```

ده:

```text
Network Error
```

---

# طيب إيه الأسباب الممكنة؟

Network Error ممكن يحصل بسبب:

- انقطاع الإنترنت.
- مشكلة في DNS.
- السيرفر غير قابل للوصول.
- مشكلة في الاتصال بين Client وServer.
- Browser يمنع الاتصال بسبب CORS في بعض الحالات.

المهم هنا إننا **لا نستقبل HTTP response صالحًا من السيرفر بالطريقة المعتادة**.

---

# نقطة مهمة جدًا مع Axios

لما يحصل Network Error:

```ts
try {
  await axios.get("/users");
} catch (error) {
  // Network failure
}
```

الـ `catch` يشتغل.

لكن ماينفعش تفترض إن عندك:

```ts
error.response.status
```

لأن في Network Error ممكن أصلًا **مفيش `response` موجودة**.

وده هيكون مهم جدًا لما نبدأ:

```text
AxiosError
```

لأننا هنبدأ نسأل:

> إزاي أعرف هل عندي `response`؟  
> وهل عندي `request`؟  
> وهل الخطأ Axios أصلًا؟

---

## 🧠 احفظ الجدول ده

| الحالة | وصل للسيرفر؟ | HTTP Response؟ | مثال |
|---|---|---|---|
| Success | ✅ | ✅ | `200` |
| HTTP Error | ✅ | ✅ | `404`, `500` |
| Network Error | ❌ غالبًا | ❌ | Internet/DNS/connection failure |

كلمة **غالبًا** مهمة لأن تفاصيل بعض مشاكل الشبكة تعتمد على البيئة، خصوصًا المتصفح.

### Interview Question

عندك:

```ts
try {
  const response = await axios.get("/users");
} catch (error) {
  console.log(error);
}
```

لو حصل **Network Error**، ليه ماينفعش تفترض إن:

```ts
error.response.status
```

موجود؟

**الخطوة التالية:** جاوب السؤال ده، وبعدها ندخل على **`AxiosError`** ونفكك الـ object نفسه.
<details>
<summary>الـ network error أنا أصلًا ما وصلتش الـ response يعني أنا ما وصلتش أصلًا للسيرفر عشان يحصل فيه response عشان أعرف الـ status، سالك الـ response مش موجودة، فساعتها مش عارف من حالة.</summary>
user
الـ network error أنا أصلًا ما وصلتش الـ response يعني أنا ما وصلتش أصلًا للسيرفر عشان يحصل فيه response عشان أعرف الـ status، سالك الـ response مش موجودة، فساعتها مش عارف من حالة.
ChatGPT
**إجابتك 10/10، وده بالضبط السبب اللي كنا عايزين نوصل له.** ✅

أنت قلت الفكرة الأساسية بشكل صحيح:

> في `Network Error` أنا لم أحصل على `response`، وبالتالي لا أقدر أعتمد على `response.status`.

وده يقودنا مباشرةً للنقطة التالية.

## 📍 Roadmap

```text
SECTION 0  Architecture Foundation       ⬜
SECTION 1  Axios Fundamentals             ✅
SECTION 2  TypeScript + Axios             ✅
SECTION 3  Error Handling                 🟢 CURRENT
  ├─ Error Handling Fundamentals          ✅
  ├─ HTTP Errors                          ✅
  ├─ Network Errors                       ✅
  ├─ AxiosError                           🟡 ← نحن هنا
  ├─ Error Normalization                  ⬜
  └─ Custom ApiError                      ⬜
SECTION 4  Interceptors                   ⬜
...
```
</details>
الـ network error أنا أصلًا ما وصلتش الـ response يعني أنا ما وصلتش أصلًا للسيرفر عشان يحصل فيه response عشان أعرف الـ status، سالك الـ response مش موجودة، فساعتها مش عارف من حالة.
ChatGPT
**إجابتك 10/10، وده بالضبط السبب اللي كنا عايزين نوصل له.** ✅

أنت قلت الفكرة الأساسية بشكل صحيح:

> في `Network Error` أنا لم أحصل على `response`، وبالتالي لا أقدر أعتمد على `response.status`.

وده يقودنا مباشرةً للنقطة التالية.

## 📍 Roadmap

```text
SECTION 0  Architecture Foundation       ⬜
SECTION 1  Axios Fundamentals             ✅
SECTION 2  TypeScript + Axios             ✅
SECTION 3  Error Handling                 🟢 CURRENT
  ├─ Error Handling Fundamentals          ✅
  ├─ HTTP Errors                          ✅
  ├─ Network Errors                       ✅
  ├─ AxiosError                           🟡 ← نحن هنا
  ├─ Error Normalization                  ⬜
  └─ Custom ApiError                      ⬜
SECTION 4  Interceptors                   ⬜
...
```