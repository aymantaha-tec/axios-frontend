# Lesson 2.5: `axios.delete()`

`DELETE` في HTTP معناها ببساطة:

> احذف Resource موجود.

مثلاً عندنا User رقمه `10`:

```http
DELETE /users/10
```

معناها:

> احذف الـ User رقم 10.

---

## 1. Axios

في Axios:

```ts
const response = await axios.delete("/users/10");
```

لاحظ إننا غالبًا **مش محتاجين Body**.

الـ ID موجود في الـ URL:

```text
/users/10
       ↑
      ID
```

---

## 2. مع TypeScript

ممكن السيرفر يرجع الـ User المحذوف:

```ts
const response = await axios.delete<User>(
  "/users/10"
);
```

هنا:

```text
<User>
   ↓
Response Type
```

وبالتالي:

```ts
response.data
```

نوعها:

```ts
User
```

لكن مش كل APIs بترجع User بعد الحذف.

ممكن السيرفر يرجع:

```http
204 No Content
```

يعني:

> تم الحذف، لكن مفيش Response Body.

في الحالة دي مش بنحتاج نتوقع `User` كـ response.

---

# 3. الشكل العام

حتى الآن عندنا:

```text
GET
axios.get<T>(url)

POST
axios.post<T>(url, body)

PUT
axios.put<T>(url, body)

PATCH
axios.patch<T>(url, body)

DELETE
axios.delete<T>(url)
```

لاحظ حاجة مهمة:

```text
POST / PUT / PATCH
        ↓
      Body
```

بينما:

```text
GET / DELETE
        ↓
غالبًا بدون Body
```

مش معنى "غالبًا" إن البروتوكول يمنع أي Body في كل حالة، لكن **ده الاستخدام المعتاد في APIs**.

---

# 🧪 التدريب

عندك:

```ts
interface User {
  id: number;
  name: string;
  email: string;
}
```

عايز تحذف:

```text
User ID = 25
```

والـ API:

```text
DELETE /users/25
```

والـ API بترجع:

```ts
User
```

اكتب:

```ts
const response = ...
```

### 🎤 Interview

بعد الكود جاوب:

> **ما الفرق بين `axios.delete()` و `axios.post()` من ناحية الـ Request Body؟**

**الخطوة التالية: اكتب كود `axios.delete()` وإجابة الـ Interview في سطر واحد.**
<details>
<summary>const response =  await axios.delete&lt;User&gt;("/users/25");</summary>
user
const response =  await axios.delete<User>("/users/25");

axios.delete()   Request Body  مش محتاجة
ChatGPT
**الهدف: تثبيت `axios.delete()`، وإجابتك صحيحة بالكامل.** الوقت: **دقيقتان**.

### 🧪 تقييم التدريب

كتبت:

```ts id="3c7m8u"
const response = await axios.delete<User>("/users/25");
```

✅ صحيح.

```text id="1z7j6d"
<User>
↓
نوع الـ Response المتوقع

"/users/25"
↓
الـ Resource الذي سيتم حذفه
```

### 🎤 Interview

إجابتك:

> `axios.delete()` مش محتاجة Request Body.

✅ **صحيحة في سياق الاستخدام المعتاد للـ APIs.**

والفرق مع `POST`:

```text id="c9ux2e"
POST
↓
غالبًا يحتاج Body
↓
بيانات لإنشاء Resource

DELETE
↓
غالبًا بدون Body
↓
الـ Resource محدد في URL
```

مثال:

```ts id="r2tdtq"
axios.post<User>("/users", userData);
```

مقابل:

```ts id="a9s1kf"
axios.delete<User>("/users/25");
```

---
</details>
const response =  await axios.delete<User>("/users/25");

axios.delete()   Request Body  مش محتاجة
ChatGPT
**الهدف: تثبيت `axios.delete()`، وإجابتك صحيحة بالكامل.** الوقت: **دقيقتان**.

### 🧪 تقييم التدريب

كتبت:

```ts id="3c7m8u"
const response = await axios.delete<User>("/users/25");
```

✅ صحيح.

```text id="1z7j6d"
<User>
↓
نوع الـ Response المتوقع

"/users/25"
↓
الـ Resource الذي سيتم حذفه
```

### 🎤 Interview

إجابتك:

> `axios.delete()` مش محتاجة Request Body.

✅ **صحيحة في سياق الاستخدام المعتاد للـ APIs.**

والفرق مع `POST`:

```text id="c9ux2e"
POST
↓
غالبًا يحتاج Body
↓
بيانات لإنشاء Resource

DELETE
↓
غالبًا بدون Body
↓
الـ Resource محدد في URL
```

مثال:

```ts id="r2tdtq"
axios.post<User>("/users", userData);
```

مقابل:

```ts id="a9s1kf"
axios.delete<User>("/users/25");
```

---