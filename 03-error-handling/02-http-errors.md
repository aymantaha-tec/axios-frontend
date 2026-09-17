# HTTP Errors

خلينا نبدأ من حاجة أنت درستها قبل كده:

الـ HTTP Response عنده `status code`.

مثلاً:

```text
200 → Success
201 → Created
400 → Bad Request
401 → Unauthorized
403 → Forbidden
404 → Not Found
500 → Internal Server Error
```

الجزء المهم هنا:

**`4xx` و `5xx` معناهم إن السيرفر رد فعلًا.**

يعني حصل:

```text
Client
  ↓
HTTP Request
  ↓
Server
  ↓
HTTP Response
  ↓
404
```

إذن دي **مش Network Error**.

الشبكة اشتغلت ووصلنا للسيرفر، والسيرفر رد علينا، لكنه قال:

> الطلب ده مش ناجح.

---

## ماذا يفعل Axios؟

هنا نقطة مهمة جدًا.

لو عملت:

```ts
const response = await axios.get("/users/999");
```

والسيرفر رجع:

```text
404 Not Found
```

Axios افتراضيًا **لن يكمل كأن العملية نجحت**.

الـ Promise هتكون rejected، وبالتالي التنفيذ ينتقل إلى `catch`.

```ts
try {
  const response = await axios.get("/users/999");

  console.log(response.data);
} catch (error) {
  console.log(error);
}
```

الـ flow:

```text
axios.get()
     ↓
Server
     ↓
404
     ↓
Axios يعتبرها failure
     ↓
Promise rejected
     ↓
catch
```

---

# مثال عملي

تخيل إن عندنا:

```ts
const getUser = async (id: number): Promise<IUser> => {
  const response = await axios.get<IApiResponse<IUser>>(
    `/users/${id}`
  );

  return response.data.payload;
};
```

والـ Component يعمل:

```ts
try {
  const user = await getUser(999);

  console.log(user.name);
} catch (error) {
  console.log("Failed");
}
```

لو `999` مش موجود:

```text
getUser(999)
     ↓
axios.get()
     ↓
404
     ↓
❌
     ↓
catch
```

وبالتالي:

```ts
console.log(user.name);
```

**لن يتم تنفيذها.**

لأننا لم نحصل على `IUser`.

---

# طيب هل كل `4xx` و `5xx` نفس المعنى؟

لا.

وده مهم جدًا في الـ Error Handling.

### `400 Bad Request`

السيرفر بيقول:

> الـ request نفسه فيه مشكلة.

مثلاً بيانات غير صحيحة.

---

### `401 Unauthorized`

السيرفر بيقول:

> أنت محتاج authentication أو الـ credentials غير صالحة.

مثلاً:

```text
Access Token expired
```

---

### `403 Forbidden`

السيرفر فهم الطلب، وعرف أنت مين، لكن:

> ليس لديك صلاحية لتنفيذ العملية.

---

### `404 Not Found`

المورد المطلوب غير موجود.

```text
GET /users/999
       ↓
404
```

---

### `500 Internal Server Error`

المشكلة من جهة السيرفر.

```text
Client
  ↓
Request
  ↓
Server
  ↓
💥 Internal problem
  ↓
500
```

---

# أهم نقطة في الدرس

فرق بين:

### HTTP Error

```text
Request
   ↓
Server وصل إليه الطلب
   ↓
Server رد
   ↓
4xx / 5xx
```

### Network Error

```text
Request
   ↓
❌ لم نتمكن من الوصول للسيرفر أصلًا
```

يعني:

```text
HTTP Error
= Server responded, but response indicates failure

Network Error
= We couldn't successfully communicate with the server
```

وده الفرق اللي هنشتغل عليه في الدرس القادم.

---

## 🧠 سؤال Interview

**لو عملت:**

```ts
try {
  const response = await axios.get("/users/999");
} catch (error) {
  console.log(error);
}
```

والـ server رجع `404`:

**هل ده `Network Error`؟ وليه؟**

**الخطوة التالية:** جاوب السؤال ده، وبعدها هقيّم إجابتك وننتقل إلى `Network Errors`.
<details>
<summary>Convert to writing block</summary>
user
Convert to writing block
ChatGPT
:::writing{variant="standard" id="58321" title="HTTP 400 Bad Request"}
**400 Bad Request**

السيرفر بيقول إن الـ **request نفسه فيه مشكلة**.

مثلاً البيانات المرسلة غير صحيحة أو ناقصة.
:::
</details>
Convert to writing block
ChatGPT
:::writing{variant="standard" id="58321" title="HTTP 400 Bad Request"}
**400 Bad Request**

السيرفر بيقول إن الـ **request نفسه فيه مشكلة**.

مثلاً البيانات المرسلة غير صحيحة أو ناقصة.
:::
<details>
<summary>لا، هذا الخطأ لا يعتبر socket error أو يعتبر HTTP error، لأن هنا نروح للسيرفر، والسيرفر responded, but response indicate failure.</summary>
user
لا، هذا الخطأ لا يعتبر socket error أو يعتبر HTTP error، لأن هنا نروح للسيرفر، والسيرفر responded, but response indicate failure.
ChatGPT
إجابتك **10/10**. ✅

بالضبط:

```text
404
↓
HTTP Error
```

وليس:

```text
Network Error ❌
```

لأن الاتصال بالسيرفر تم بنجاح:

```text
Client
  ↓
HTTP Request
  ↓
Server
  ↓
HTTP Response: 404
  ↓
Axios rejects Promise
  ↓
catch
```

والجملة الإنجليزية التي استخدمتها ممتازة:

> **Server responded, but the response indicates failure.**

أما **Network Error** فالمشكلة تكون في الوصول أو الاتصال بالسيرفر نفسه، وده اللي هنفصله في الدرس القادم.

**الخطوة التالية:** نبدأ **Network Errors** ونقارنها مباشرةً بـ `HTTP Errors`.
</details>
لا، هذا الخطأ لا يعتبر socket error أو يعتبر HTTP error، لأن هنا نروح للسيرفر، والسيرفر responded, but response indicate failure.
ChatGPT
إجابتك **10/10**. ✅

بالضبط:

```text
404
↓
HTTP Error
```

وليس:

```text
Network Error ❌
```

لأن الاتصال بالسيرفر تم بنجاح:

```text
Client
  ↓
HTTP Request
  ↓
Server
  ↓
HTTP Response: 404
  ↓
Axios rejects Promise
  ↓
catch
```

والجملة الإنجليزية التي استخدمتها ممتازة:

> **Server responded, but the response indicates failure.**

أما **Network Error** فالمشكلة تكون في الوصول أو الاتصال بالسيرفر نفسه، وده اللي هنفصله في الدرس القادم.

**الخطوة التالية:** نبدأ **Network Errors** ونقارنها مباشرةً بـ `HTTP Errors`.
<details>
<summary>**Network Errors**</summary>
user
**Network Errors**  
ChatGPT
**خد 10 دقائق: افهم الفرق الجوهري بين `HTTP Error` و`Network Error`، لأن الفرق بينهم هيفسر لك بعد كده سلوك `AxiosError`.**

## 📍 Roadmap

```text
SECTION 0  Architecture Foundation       ⬜
SECTION 1  Axios Fundamentals             ✅
SECTION 2  TypeScript + Axios             ✅
SECTION 3  Error Handling                 🟢 CURRENT
  ├─ Error Handling Fundamentals          ✅
  ├─ HTTP Errors                          ✅
  ├─ Network Errors                       🟡 ← نحن هنا
  ├─ AxiosError                           ⬜
  ├─ Error Normalization                  ⬜
  └─ Custom ApiError                      ⬜
SECTION 4  Interceptors                   ⬜
...
```
</details>
**Network Errors**  
ChatGPT
**خد 10 دقائق: افهم الفرق الجوهري بين `HTTP Error` و`Network Error`، لأن الفرق بينهم هيفسر لك بعد كده سلوك `AxiosError`.**

## 📍 Roadmap

```text
SECTION 0  Architecture Foundation       ⬜
SECTION 1  Axios Fundamentals             ✅
SECTION 2  TypeScript + Axios             ✅
SECTION 3  Error Handling                 🟢 CURRENT
  ├─ Error Handling Fundamentals          ✅
  ├─ HTTP Errors                          ✅
  ├─ Network Errors                       🟡 ← نحن هنا
  ├─ AxiosError                           ⬜
  ├─ Error Normalization                  ⬜
  └─ Custom ApiError                      ⬜
SECTION 4  Interceptors                   ⬜
...
```