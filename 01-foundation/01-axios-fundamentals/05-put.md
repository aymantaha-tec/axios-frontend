# Lesson 2.3: `axios.put()`

إحنا عرفنا:

```ts
axios.post()
```

وكان الهدف منه:

> إنشاء Resource جديد.

أما:

```ts
axios.put()
```

فالفكرة الأساسية:

> **استبدال Resource موجود ببيانات جديدة.**

---

## 1. مثال HTTP

عندك User رقمه `10`:

```http
PUT /users/10
```

أنت بتقول للسيرفر:

> حدّث User رقم 10 بالبيانات اللي هبعتها.

مثلاً:

```json
{
  "name": "Ahmed",
  "email": "ahmed@example.com"
}
```

---

# 2. Axios

نكتب:

```ts
const userData: UpdateUserRequest = {
  name: "Ahmed",
  email: "ahmed@example.com",
};

const response = await axios.put<User>(
  "/users/10",
  userData
);
```

نفس الشكل اللي اتعلمناه في `POST`:

```text
axios.put<ResponseType>(
  URL,
  Body
)
```

---

# 3. نفك السطر

```ts
axios.put<User>(
```

`<User>`:

> نوع الـ Response المتوقع.

---

```ts
"/users/10"
```

ده الـ URL.

لاحظ إننا هنا بنتعامل مع User **محدد**:

```text
/users/10
        ↑
       ID
```

---

```ts
userData
```

ده الـ Request Body.

---

# 4. الفرق بين POST و PUT

ركز في النقطة دي:

### POST

```ts
axios.post<User>(
  "/users",
  userData
);
```

معناه:

```text
أنشئ User جديد
```

السيرفر هو اللي غالبًا بيحدد الـ ID.

مثلاً:

```text
POST /users
       ↓
Server
       ↓
User ID = 10
```

---

### PUT

```ts
axios.put<User>(
  "/users/10",
  userData
);
```

معناه:

```text
تعامل مع User رقم 10
واستبدل بياناته بالبيانات الجديدة
```

---

# 5. PUT vs PATCH

دي نقطة مهمة، لكن **مش هندخل في PATCH دلوقتي**.

فقط خليك فاكر:

```text
PUT
↓
Replace

PATCH
↓
Partial Update
```

هندرس PATCH في الدرس اللي بعده بالتفصيل.

---

# 🧪 التدريب

عندك:

```ts
interface User {
  id: number;
  name: string;
  email: string;
}

interface UpdateUserRequest {
  name: string;
  email: string;
}
```

عايز تعدّل User رقمه `5`:

```text
name = "Mohamed"
email = "mohamed@example.com"
```

والـ API:

```text
PUT /users/5
```

اكتب:

```ts
const userData: ...
```

وبعدها:

```ts
const response = ...
```

### 🎤 Interview Question

**ما الفرق الأساسي بين `POST` و `PUT`؟**

جاوب على الاثنين، وأنا هصححهم، وبعدها ننتقل إلى `PATCH`.
<details>
<summary>const userData:  UpdateUserRequest  ={</summary>
user
const userData:  UpdateUserRequest  ={
name = "Mohamed"
email = "mohamed@example.com"
}
const response = axios.put<User>("users /5", userData);
ChatGPT
**الهدف: نصحح تدريب `PUT` فقط، وإجابتك فاهمة الفكرة، لكن عندك خطأين في الـ syntax والـ URL.** الوقت: **3 دقائق**.

### إجابتك بعد التصحيح

```ts
const userData: UpdateUserRequest = {
  name: "Mohamed",
  email: "mohamed@example.com",
};

const response = await axios.put<User>(
  "/users/5",
  userData
);
```

## التصحيح

### 1. داخل Object

أنت كتبت:

```ts
name = "Mohamed"
email = "mohamed@example.com"
```

الصحيح:

```ts
name: "Mohamed",
email: "mohamed@example.com",
```

لأننا داخل Object بنستخدم:

```text
property: value
```

---

### 2. الـ URL

أنت كتبت:

```ts
"users /5"
```

الصحيح:

```ts
"/users/5"
```

لأن الـ Path هو:

```text
/users/5
```

---

### 3. نسيت `await`

أنت كتبت:

```ts
const response = axios.put<User>(...)
```

والأفضل هنا:

```ts
const response = await axios.put<User>(...)
```

لأن `axios.put()` بترجع Promise.

---

## 🟢 تقييمك

```text
فهم PUT             ✅
Response Type       ✅
Request Body        ✅
User ID             ✅

TypeScript Syntax   🟡
URL Syntax           🟡
await                🟡
```

**8/10**

والأهم إنك فهمت تركيب PUT:

```text
axios.put<User>(
      ↓
Response Type

"/users/5"
      ↓
Resource

userData
      ↓
Request Body
```

## 🎤 Interview

لسه ما جاوبتش الجزء ده:

**ما الفرق الأساسي بين `POST` و `PUT`؟**

جاوب بجملة واحدة، وبعدها نكمل `PATCH`.
</details>
const userData:  UpdateUserRequest  ={
name = "Mohamed"
email = "mohamed@example.com"
}
const response = axios.put<User>("users /5", userData);
ChatGPT
**الهدف: نصحح تدريب `PUT` فقط، وإجابتك فاهمة الفكرة، لكن عندك خطأين في الـ syntax والـ URL.** الوقت: **3 دقائق**.

### إجابتك بعد التصحيح

```ts
const userData: UpdateUserRequest = {
  name: "Mohamed",
  email: "mohamed@example.com",
};

const response = await axios.put<User>(
  "/users/5",
  userData
);
```

## التصحيح

### 1. داخل Object

أنت كتبت:

```ts
name = "Mohamed"
email = "mohamed@example.com"
```

الصحيح:

```ts
name: "Mohamed",
email: "mohamed@example.com",
```

لأننا داخل Object بنستخدم:

```text
property: value
```

---

### 2. الـ URL

أنت كتبت:

```ts
"users /5"
```

الصحيح:

```ts
"/users/5"
```

لأن الـ Path هو:

```text
/users/5
```

---

### 3. نسيت `await`

أنت كتبت:

```ts
const response = axios.put<User>(...)
```

والأفضل هنا:

```ts
const response = await axios.put<User>(...)
```

لأن `axios.put()` بترجع Promise.

---

## 🟢 تقييمك

```text
فهم PUT             ✅
Response Type       ✅
Request Body        ✅
User ID             ✅

TypeScript Syntax   🟡
URL Syntax           🟡
await                🟡
```

**8/10**

والأهم إنك فهمت تركيب PUT:

```text
axios.put<User>(
      ↓
Response Type

"/users/5"
      ↓
Resource

userData
      ↓
Request Body
```

## 🎤 Interview

لسه ما جاوبتش الجزء ده:

**ما الفرق الأساسي بين `POST` و `PUT`؟**

جاوب بجملة واحدة، وبعدها نكمل `PATCH`.
<details>
<summary>post</summary>
user
post
معني انشي شي جديد بوست او منتج او يوزر
put
استبدل user5 بي الريسورس اللي عندي او اي شي اخر حسب المطلوب
ChatGPT
**الهدف: تقييم إجابتك على الفرق بين `POST` و `PUT`، وإجابتك صحيحة في الجوهر.** الوقت: **دقيقتان**.

### 🎤 تقييم Interview

إجابتك:

> POST معناها أنشئ شيء جديد، مثل Post أو Product أو User.  
> PUT استبدل User 5 بالـ Resource اللي عندي.

**التقييم: 9/10 ✅**

الصياغة الأدق تقنيًا:

```text
POST
↓
Create a new Resource

PUT
↓
Replace the representation of an existing Resource
```

مثال:

```http
POST /users
```

يعني:

> أنشئ User جديد.

بينما:

```http
PUT /users/5
```

يعني:

> استبدل تمثيل User رقم 5 بالبيانات التي أرسلتها.

### نقطة صغيرة مهمة

كلمة **استبدل** هي المفتاح في `PUT`.

لو User رقم 5 عنده:

```json
{
  "name": "Ahmed",
  "email": "ahmed@example.com",
  "age": 30
}
```

وأرسلت PUT:

```json
{
  "name": "Mohamed",
  "email": "mohamed@example.com"
}
```

فمفهوم PUT التقليدي هو أن الـ Resource الجديد يصبح بالبيانات المرسلة، ولذلك لا نتعامل معه كـ "تعديل حقل واحد" مثل PATCH.

هنثبت الفرق ده أكثر لما ندرس `PATCH`.

---

## 📍 Roadmap

```text
SECTION 1: Axios Fundamentals 🟢

GET                  ✅
POST                 ✅
PUT                  ✅ ← خلصناه
PATCH                ⬅️ التالي
DELETE               ⬜
Request Config       ⬜
```

**الخطوة التالية: نبدأ `axios.patch()` ونفهم لماذا احتجنا Method مختلفة عن `PUT`.**
</details>
post
معني انشي شي جديد بوست او منتج او يوزر
put
استبدل user5 بي الريسورس اللي عندي او اي شي اخر حسب المطلوب
ChatGPT
**الهدف: تقييم إجابتك على الفرق بين `POST` و `PUT`، وإجابتك صحيحة في الجوهر.** الوقت: **دقيقتان**.

### 🎤 تقييم Interview

إجابتك:

> POST معناها أنشئ شيء جديد، مثل Post أو Product أو User.  
> PUT استبدل User 5 بالـ Resource اللي عندي.

**التقييم: 9/10 ✅**

الصياغة الأدق تقنيًا:

```text
POST
↓
Create a new Resource

PUT
↓
Replace the representation of an existing Resource
```

مثال:

```http
POST /users
```

يعني:

> أنشئ User جديد.

بينما:

```http
PUT /users/5
```

يعني:

> استبدل تمثيل User رقم 5 بالبيانات التي أرسلتها.

### نقطة صغيرة مهمة

كلمة **استبدل** هي المفتاح في `PUT`.

لو User رقم 5 عنده:

```json
{
  "name": "Ahmed",
  "email": "ahmed@example.com",
  "age": 30
}
```

وأرسلت PUT:

```json
{
  "name": "Mohamed",
  "email": "mohamed@example.com"
}
```

فمفهوم PUT التقليدي هو أن الـ Resource الجديد يصبح بالبيانات المرسلة، ولذلك لا نتعامل معه كـ "تعديل حقل واحد" مثل PATCH.

هنثبت الفرق ده أكثر لما ندرس `PATCH`.

---

## 📍 Roadmap

```text
SECTION 1: Axios Fundamentals 🟢

GET                  ✅
POST                 ✅
PUT                  ✅ ← خلصناه
PATCH                ⬅️ التالي
DELETE               ⬜
Request Config       ⬜
```

**الخطوة التالية: نبدأ `axios.patch()` ونفهم لماذا احتجنا Method مختلفة عن `PUT`.**
<details>
<summary>axios.patch()</summary>
user
axios.patch()  
ChatGPT
**الهدف: نفهم `axios.patch()` فقط، ونثبت الفرق بينها وبين `PUT` من خلال مثال وتدريب.** الوقت: **10 إلى 15 دقيقة**.

## 📍 Roadmap

```text
SECTION 1: Axios Fundamentals 🟢

GET                  ✅
POST                 ✅
PUT                  ✅
PATCH                🟢 ← نحن هنا
DELETE               ⬜
Request Config       ⬜
```
</details>
axios.patch()  
ChatGPT
**الهدف: نفهم `axios.patch()` فقط، ونثبت الفرق بينها وبين `PUT` من خلال مثال وتدريب.** الوقت: **10 إلى 15 دقيقة**.

## 📍 Roadmap

```text
SECTION 1: Axios Fundamentals 🟢

GET                  ✅
POST                 ✅
PUT                  ✅
PATCH                🟢 ← نحن هنا
DELETE               ⬜
Request Config       ⬜
```