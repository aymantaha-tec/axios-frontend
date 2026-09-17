# Lesson 2.4: `axios.patch()`

إحنا قلنا إن:

```text
PUT
↓
Replace

PATCH
↓
Partial Update
```

يعني `PATCH` بنستخدمها لما نريد **تعديل جزء من Resource موجود**.

---

## 1. مثال بسيط

عندك User:

```json
{
  "id": 5,
  "name": "Ahmed",
  "email": "ahmed@example.com",
  "age": 30
}
```

عايز تغير **الاسم فقط**.

مع `PATCH`:

```http
PATCH /users/5
```

Body:

```json
{
  "name": "Mohamed"
}
```

أنت هنا لم ترسل:

```text
email
age
```

لأنك مش عايز تعدلهم.

---

# 2. Axios

نكتب:

```ts
const response = await axios.patch<User>(
  "/users/5",
  {
    name: "Mohamed",
  }
);
```

نفس pattern اللي اتعلمناه:

```text
axios.patch<ResponseType>(
  URL,
  Body
)
```

---

# 3. الفرق الحقيقي بين PUT و PATCH

افترض إن عندنا:

```json
{
  "name": "Ahmed",
  "email": "ahmed@example.com",
  "age": 30
}
```

### PUT

```ts
await axios.put<User>("/users/5", {
  name: "Mohamed",
  email: "mohamed@example.com",
  age: 30,
});
```

الفكرة:

> استبدل الـ representation بالبيانات الجديدة.

---

### PATCH

```ts
await axios.patch<User>("/users/5", {
  name: "Mohamed",
});
```

الفكرة:

> غيّر `name` فقط.

---

# 4. نقطة TypeScript مهمة

ممكن تعمل Type مخصوص للـ PATCH:

```ts
interface UpdateUserRequest {
  name?: string;
  email?: string;
  age?: number;
}
```

لأن التعديل ممكن يحتوي على أي جزء.

مثلاً:

```ts
const userData: UpdateUserRequest = {
  name: "Mohamed",
};
```

أو:

```ts
const userData: UpdateUserRequest = {
  age: 31,
};
```

أو:

```ts
const userData: UpdateUserRequest = {
  name: "Mohamed",
  age: 31,
};
```

لكن **مش لازم نحفظ التصميم ده دلوقتي**. المهم تفهم إن PATCH يسمح بإرسال الجزء الذي تريد تعديله.

---

# 🧪 التدريب

عندك:

```ts
interface User {
  id: number;
  name: string;
  email: string;
  age: number;
}
```

User رقمه `10`:

```json
{
  "id": 10,
  "name": "Ahmed",
  "email": "ahmed@example.com",
  "age": 30
}
```

عايز تغير **الـ email فقط** إلى:

```text
new@example.com
```

اكتب:

```ts
const response = ...
```

باستخدام `axios.patch()`.

### 🎤 Interview

بعد الكود جاوب:

> **متى تستخدم `PATCH` بدل `PUT`؟**

**الخطوة التالية: اكتب كود `axios.patch()` ثم إجابة الـ Interview في سطر واحد.**
<details>
<summary>interface updatePartailUser {   id?: number;   name?: string;   email?: string;   age?: number; }</summary>
user
interface updatePartailUser {   id?: number;   name?: string;   email?: string;   age?: number; }  

updateUser: updatePartailUser ={
email: "ahmed@example.com",
}

const response =  await axios.patch<User>("/users/10", updateUser);
ChatGPT
**الهدف: تصحيح تدريب `PATCH` فقط، وإجابتك من ناحية Axios صحيحة، لكن عندك ملاحظة TypeScript بسيطة في تعريف المتغير.** الوقت: **3 دقائق**.

### ✅ الجزء الأهم عندك صحيح

```ts id="c5rj0v"
const response = await axios.patch<User>(
  "/users/10",
  updateUser
);
```

أنت فهمت:

```text id="6b4r7s"
<User>
   ↓
Response Type

"/users/10"
   ↓
Resource المطلوب تعديله

updateUser
   ↓
الـ Body
```

ممتاز.

---

## 1. الـ Interface

كتبت:

```ts id="7q2y9f"
interface updatePartailUser {
  id?: number;
  name?: string;
  email?: string;
  age?: number;
}
```

الفكرة صحيحة.

لكن Naming convention في TypeScript عادةً نخلي أسماء الـ types تبدأ بحرف Capital:

```ts id="0c47kg"
interface UpdatePartialUser {
  id?: number;
  name?: string;
  email?: string;
  age?: number;
}
```

وكمان عندك typo بسيط:

```text
Partail ❌
Partial ✅
```

لكن ده **مش مؤثر على فهمك**.

---

## 2. تعريف `updateUser`

أنت كتبت:

```ts id="1j2f5s"
updateUser: updatePartailUser = {
  email: "ahmed@example.com",
};
```

لو `updateUser` متغير، لازم:

```ts id="9ndv0e"
const updateUser: UpdatePartialUser = {
  email: "ahmed@example.com",
};
```

لأن:

```text id="2x7k1q"
const updateUser
       ↓
اسم المتغير

: UpdatePartialUser
       ↓
نوع المتغير

= {...}
       ↓
قيمة المتغير
```

---

# ✅ الكود النهائي

```ts id="tkwp3a"
interface UpdatePartialUser {
  id?: number;
  name?: string;
  email?: string;
  age?: number;
}

const updateUser: UpdatePartialUser = {
  email: "ahmed@example.com",
};

const response = await axios.patch<User>(
  "/users/10",
  updateUser
);
```

## 🟢 تقييم التدريب

```text
PATCH Concept          ✅
Request Body           ✅
Response Type          ✅
Resource ID            ✅
Partial Update         ✅
TypeScript Syntax      🟡
Naming                 🟡
```

**التقييم: 9/10**

### 🎤 Interview

السؤال كان:

> متى تستخدم `PATCH` بدل `PUT`؟

إجابتك لسه ناقصة.

**الخطوة التالية: جاوب بجملة واحدة فقط: `PATCH` أستخدمها إمتى؟**
</details>
interface updatePartailUser {   id?: number;   name?: string;   email?: string;   age?: number; }  

updateUser: updatePartailUser ={
email: "ahmed@example.com",
}

const response =  await axios.patch<User>("/users/10", updateUser);
ChatGPT
**الهدف: تصحيح تدريب `PATCH` فقط، وإجابتك من ناحية Axios صحيحة، لكن عندك ملاحظة TypeScript بسيطة في تعريف المتغير.** الوقت: **3 دقائق**.

### ✅ الجزء الأهم عندك صحيح

```ts id="c5rj0v"
const response = await axios.patch<User>(
  "/users/10",
  updateUser
);
```

أنت فهمت:

```text id="6b4r7s"
<User>
   ↓
Response Type

"/users/10"
   ↓
Resource المطلوب تعديله

updateUser
   ↓
الـ Body
```

ممتاز.

---

## 1. الـ Interface

كتبت:

```ts id="7q2y9f"
interface updatePartailUser {
  id?: number;
  name?: string;
  email?: string;
  age?: number;
}
```

الفكرة صحيحة.

لكن Naming convention في TypeScript عادةً نخلي أسماء الـ types تبدأ بحرف Capital:

```ts id="0c47kg"
interface UpdatePartialUser {
  id?: number;
  name?: string;
  email?: string;
  age?: number;
}
```

وكمان عندك typo بسيط:

```text
Partail ❌
Partial ✅
```

لكن ده **مش مؤثر على فهمك**.

---

## 2. تعريف `updateUser`

أنت كتبت:

```ts id="1j2f5s"
updateUser: updatePartailUser = {
  email: "ahmed@example.com",
};
```

لو `updateUser` متغير، لازم:

```ts id="9ndv0e"
const updateUser: UpdatePartialUser = {
  email: "ahmed@example.com",
};
```

لأن:

```text id="2x7k1q"
const updateUser
       ↓
اسم المتغير

: UpdatePartialUser
       ↓
نوع المتغير

= {...}
       ↓
قيمة المتغير
```

---

# ✅ الكود النهائي

```ts id="tkwp3a"
interface UpdatePartialUser {
  id?: number;
  name?: string;
  email?: string;
  age?: number;
}

const updateUser: UpdatePartialUser = {
  email: "ahmed@example.com",
};

const response = await axios.patch<User>(
  "/users/10",
  updateUser
);
```

## 🟢 تقييم التدريب

```text
PATCH Concept          ✅
Request Body           ✅
Response Type          ✅
Resource ID            ✅
Partial Update         ✅
TypeScript Syntax      🟡
Naming                 🟡
```

**التقييم: 9/10**

### 🎤 Interview

السؤال كان:

> متى تستخدم `PATCH` بدل `PUT`؟

إجابتك لسه ناقصة.

**الخطوة التالية: جاوب بجملة واحدة فقط: `PATCH` أستخدمها إمتى؟**
<details>
<summary>`PATCH`</summary>
user
`PATCH`   
عندما اريد فقط استبدل جزء من الرسورس
ChatGPT
**الهدف: تقييم إجابتك على `PATCH` فقط، وإجابتك صحيحة في المعنى، مع تصحيح لفظ واحد مهم.** الوقت: **دقيقة واحدة**.

### 🎤 Interview

إجابتك:

> عندما أريد فقط استبدل جزء من الـ Resource.

**التقييم: 9.5/10 ✅**

الأدق نقول:

> عندما أريد **تعديل جزء من الـ Resource**.

لأن كلمة **استبدال** ارتبطنا بها مع `PUT`، بينما:

```text
PUT
↓
Replace the Resource

PATCH
↓
Partially modify the Resource
```

مثال:

```ts id="y9xq6h"
await axios.patch<User>("/users/10", {
  email: "new@example.com",
});
```

أنت هنا لا تستبدل الـ User كله، أنت تعدل `email` فقط.

---

## 📍 Roadmap

```text id="d2v8qi"
SECTION 1: Axios Fundamentals 🟢

GET                  ✅
POST                 ✅
PUT                  ✅
PATCH                ✅ ← خلصناه
DELETE               ⬅️ التالي
Request Config       ⬜
```

**الخطوة التالية: نبدأ `axios.delete()` فقط، بنفس النظام: شرح → مثال → تدريب → Interview.**
</details>
`PATCH`   
عندما اريد فقط استبدل جزء من الرسورس
ChatGPT
**الهدف: تقييم إجابتك على `PATCH` فقط، وإجابتك صحيحة في المعنى، مع تصحيح لفظ واحد مهم.** الوقت: **دقيقة واحدة**.

### 🎤 Interview

إجابتك:

> عندما أريد فقط استبدل جزء من الـ Resource.

**التقييم: 9.5/10 ✅**

الأدق نقول:

> عندما أريد **تعديل جزء من الـ Resource**.

لأن كلمة **استبدال** ارتبطنا بها مع `PUT`، بينما:

```text
PUT
↓
Replace the Resource

PATCH
↓
Partially modify the Resource
```

مثال:

```ts id="y9xq6h"
await axios.patch<User>("/users/10", {
  email: "new@example.com",
});
```

أنت هنا لا تستبدل الـ User كله، أنت تعدل `email` فقط.

---

## 📍 Roadmap

```text id="d2v8qi"
SECTION 1: Axios Fundamentals 🟢

GET                  ✅
POST                 ✅
PUT                  ✅
PATCH                ✅ ← خلصناه
DELETE               ⬅️ التالي
Request Config       ⬜
```

**الخطوة التالية: نبدأ `axios.delete()` فقط، بنفس النظام: شرح → مثال → تدريب → Interview.**
<details>
<summary>axios.delete()</summary>
user
axios.delete()  
ChatGPT
**الهدف: نتعلم `axios.delete()` فقط، ونكمل Section 1 بالترتيب بدون قفز.** الوقت: **10 دقائق**.

## 📍 Roadmap

```text
SECTION 1: Axios Fundamentals 🟢

GET                  ✅
POST                 ✅
PUT                  ✅
PATCH                ✅
DELETE               🟢 ← نحن هنا
Request Config       ⬜
```
</details>
axios.delete()  
ChatGPT
**الهدف: نتعلم `axios.delete()` فقط، ونكمل Section 1 بالترتيب بدون قفز.** الوقت: **10 دقائق**.

## 📍 Roadmap

```text
SECTION 1: Axios Fundamentals 🟢

GET                  ✅
POST                 ✅
PUT                  ✅
PATCH                ✅
DELETE               🟢 ← نحن هنا
Request Config       ⬜
```