# Lesson 2.6: Request Configuration

إحنا استخدمنا قبل كده:

```ts id="zyy7hb"
axios.get("/users");
```

واستخدمنا:

```ts id="s9aq8p"
axios.post<User>("/users", userData);
```

لكن أحيانًا بنحتاج نقول لـ Axios تفاصيل إضافية عن الـ Request.

هنا يأتي دور:

```ts id="0s4j0d"
config
```

---

# 1. يعني إيه Config؟

الـ **Configuration** يعني:

> إعدادات بتحدد إزاي Axios تنفذ الـ Request.

مثلاً:

```ts id="f4s2ad"
axios.get("/users", {
  params: {
    page: 2,
  },
});
```

الـ object ده:

```ts id="u8u4ws"
{
  params: {
    page: 2,
  }
}
```

هو **Request Config**.

---

# 2. الشكل العام

بشكل مبسط:

```ts id="8iw8j7"
axios.get(url, config);
```

مثلاً:

```ts id="m5pm6n"
axios.get("/users", {
  params: {
    page: 2,
  },
  headers: {
    Authorization: "Bearer abc123",
  },
});
```

إذن:

```text id="5e7p8r"
"/users"
   ↓
URL

{
  params: ...
  headers: ...
}
   ↓
Config Object
```

---

# 3. لكن POST مختلف قليلًا

في `POST` عندنا:

```ts id="jy4a7a"
axios.post(url, data, config);
```

يعني:

```ts id="e2qj5m"
axios.post(
  "/users",
  userData,
  {
    headers: {
      Authorization: "Bearer abc123",
    },
  }
);
```

هنا عندنا **3 arguments**:

```text id="w50t4q"
1️⃣ URL
2️⃣ Data / Body
3️⃣ Config
```

ودي نقطة مهمة جدًا.

---

# 4. GET vs POST

### GET

```ts id="t3iyzu"
axios.get(
  "/users",
  {
    params: {
      page: 2,
    },
  }
);
```

```text id="w7uw7q"
1 → URL
2 → Config
```

### POST

```ts id="rj4x7p"
axios.post(
  "/users",
  userData,
  {
    headers: {
      Authorization: "Bearer abc123",
    },
  }
);
```

```text id="w8op7h"
1 → URL
2 → Body
3 → Config
```

**وده سبب إن شكل `get()` و `post()` مش متطابق.**

---

# 🧠 أهم نقطة في الدرس

لما تشوف:

```ts id="e3fz8c"
axios.get("/users", {
  params: {...},
  headers: {...},
});
```

ما تقولش:

> الـ object ده Body.

لأ.

في `GET`:

```text id="9s6h8c"
الـ object الثاني
↓
Config
```

أما في `POST`:

```ts id="4thp0f"
axios.post("/users", userData, config);
```

```text id="xq1j6q"
الـ object الثاني
↓
Body

الـ object الثالث
↓
Config
```

---

# 🧪 تدريب

شوف الكود ده:

```ts id="5qtrm4"
const response = await axios.get("/products", {
  params: {
    page: 2,
    limit: 10,
  },
  headers: {
    Authorization: "Bearer abc123",
  },
});
```

عايز منك تحدد فقط:

```text id="6v2t9c"
"/products" = ؟

الـ object الثاني = ؟

params = ؟

headers = ؟
```

**الخطوة التالية: جاوب الأربع نقاط، وبعدها ندخل `params` بالتفصيل.**
<details>
<summary>url</summary>
user
url
config
params: {...},  
 headers: {...},  
ChatGPT
**الهدف: تثبيت مفهوم `Request Config` قبل الدخول في `params` بالتفصيل، وإجابتك صحيحة لكن ناقصك تحديد وظيفة `params` و`headers`.** الوقت: **3 دقائق**.

إجابتك:

```text
"/products" → URL        ✅
الـ object الثاني → Config ✅

params: {...}             ✅
headers: {...}            ✅
```

لكن في السؤال كنت عايز كمان **ماذا يمثل كل واحد؟**

```text
params
↓
Query Parameters

headers
↓
معلومات إضافية عن الـ Request
```

إذن الصورة الكاملة:

```ts id="4fw6yp"
axios.get("/products", {
  params: {
    page: 2,
    limit: 10,
  },
  headers: {
    Authorization: "Bearer abc123",
  },
});
```

```text
"/products"
   ↓
URL

{
  ...
}
   ↓
Config

params
   ↓
Query Parameters

headers
   ↓
Request Headers
```

### 🟢 تقييم

```text
فهم Config Object     ✅
تمييز URL             ✅
تمييز params          ✅
تمييز headers         ✅

النتيجة: 10/10
```

أنت فاهم المطلوب، وكان الناقص فقط **تسمية وظيفة كل Property**.

**الخطوة التالية: نبدأ `params` بالتفصيل، ونشوف كيف Axios تحول الـ object إلى Query String في الـ URL.**
</details>
url
config
params: {...},  
 headers: {...},  
ChatGPT
**الهدف: تثبيت مفهوم `Request Config` قبل الدخول في `params` بالتفصيل، وإجابتك صحيحة لكن ناقصك تحديد وظيفة `params` و`headers`.** الوقت: **3 دقائق**.

إجابتك:

```text
"/products" → URL        ✅
الـ object الثاني → Config ✅

params: {...}             ✅
headers: {...}            ✅
```

لكن في السؤال كنت عايز كمان **ماذا يمثل كل واحد؟**

```text
params
↓
Query Parameters

headers
↓
معلومات إضافية عن الـ Request
```

إذن الصورة الكاملة:

```ts id="4fw6yp"
axios.get("/products", {
  params: {
    page: 2,
    limit: 10,
  },
  headers: {
    Authorization: "Bearer abc123",
  },
});
```

```text
"/products"
   ↓
URL

{
  ...
}
   ↓
Config

params
   ↓
Query Parameters

headers
   ↓
Request Headers
```

### 🟢 تقييم

```text
فهم Config Object     ✅
تمييز URL             ✅
تمييز params          ✅
تمييز headers         ✅

النتيجة: 10/10
```

أنت فاهم المطلوب، وكان الناقص فقط **تسمية وظيفة كل Property**.

**الخطوة التالية: نبدأ `params` بالتفصيل، ونشوف كيف Axios تحول الـ object إلى Query String في الـ URL.**
<details>
<summary>`params`</summary>
user
`params`   
ChatGPT
**الهدف: نفهم `params` في Axios من الصفر، ونشوف علاقتها بالـ Query Parameters التي درسناها، بدون الانتقال إلى `headers` الآن.** الوقت: **10 دقائق**.

# 📍 Roadmap

```text
SECTION 1: Axios Fundamentals 🟢

GET                  ✅
POST                 ✅
PUT                  ✅
PATCH                ✅
DELETE               ✅

Request Configuration 🟢
├── Config Object     ✅
├── params            🟢 ← نحن هنا
├── headers           ⬜
├── data              ⬜
├── timeout           ⬜
└── signal            ⬜
```
</details>
`params`   
ChatGPT
**الهدف: نفهم `params` في Axios من الصفر، ونشوف علاقتها بالـ Query Parameters التي درسناها، بدون الانتقال إلى `headers` الآن.** الوقت: **10 دقائق**.

# 📍 Roadmap

```text
SECTION 1: Axios Fundamentals 🟢

GET                  ✅
POST                 ✅
PUT                  ✅
PATCH                ✅
DELETE               ✅

Request Configuration 🟢
├── Config Object     ✅
├── params            🟢 ← نحن هنا
├── headers           ⬜
├── data              ⬜
├── timeout           ⬜
└── signal            ⬜
```