# 6. طب لو عملنا Function للـ User الواحد؟

```ts id="r3m9qw"
export const getUser = async (
  id: number
): Promise<IUser> => {
  const response = await axios.get<
    ISuccessResponse<IUser>
  >(`/users/${id}`);

  return response.data.payload;
};
```

الاستخدام:

```ts id="k7p4za"
const user = await getUser(10);
```

TypeScript عارف:

```text
user → IUser
```

فتقدر:

```ts id="x2n8vc"
console.log(user.name);
```

---

# 7. وهنا تظهر قوة الـ Layer

بدل:

```ts id="n8m4qa"
const response = await axios.get<
  ISuccessResponse<IUser[]>
>("/users");

const users = response.data.payload;
```

في كل Component، عندك:

```ts id="b6q9wp"
const users = await getUsers();
```

وده بيحقق مبدأ مهم:

> **كل Layer مسؤولة عن حاجة محددة.**

الـ API Layer مسؤولة عن:

```text
HTTP communication
API endpoints
Response mapping
Types
```

والـ Component مسؤولة عن:

```text
UI
Rendering
User interaction
```

لكن لاحظ حاجة مهمة:

**إحنا لسه مش بندرس Architecture بشكل كامل.**

ده مجرد تمهيد للـ `Section 0` اللي هنرجع له لاحقًا، لما ندرس `Separation of Concerns` و`Feature-Based Architecture` بشكل رسمي.

---

# 8. نقطة مهمة جدًا في الـ Type-safe API Layer

بص على:

```ts id="z8r3mw"
export const getUsers = async (): Promise<IUser[]> => {
```

ممكن تقول:

> طب ما أنا أصلًا كتبت `axios.get<IUser[]>`، ليه أكتب `Promise<IUser[]>` تاني؟

سؤال ممتاز.

لأن الاتنين بيحددوا **حاجتين مختلفتين**:

```text id="k3m7vp"
axios.get<IUser[]>
       ↓
نوع Response Data

Promise<IUser[]>
       ↓
نوع القيمة التي ترجعها Function
```

مثلاً:

```ts id="s5q2nb"
const getUsers = async (): Promise<IUser[]> => {
```

الـ `async function` دائمًا بترجع `Promise`.

والـ Promise دي تحتوي في النهاية على:

```text
IUser[]
```

---

# 🧠 الشكل النهائي

```text
API
 ↓
Axios
 ↓
AxiosResponse<ISuccessResponse<IUser[]>>
 ↓
response.data
 ↓
ISuccessResponse<IUser[]>
 ↓
payload
 ↓
IUser[]
 ↓
getUsers()
 ↓
Promise<IUser[]>
 ↓
Component
```

دي أول مرة في الكورس بنبدأ نشوف **رحلة البيانات كاملة**.

### 🎯 Interview Question

عندك:

```ts
const getUsers = async (): Promise<IUser[]> => {
  const response = await axios.get<
    ISuccessResponse<IUser[]>
  >("/users");

  return response.data.payload;
};
```

**ليه `getUsers` بترجع `Promise<IUser[]>`، مع إن `axios.get` نفسه مكتوب فيه `ISuccessResponse<IUser[]>`؟**

جاوب من فهمك، وبعدها هنقيّم إجابتك قبل ما نكمل.
<details>
<summary>Convert to writing block</summary>
user
Convert to writing block
ChatGPT
:::writing{variant="standard" id="61483" title="نوع القيمة التي ترجعها الدالة"}
نوع القيمة التي ترجعها الدالة
:::
</details>
Convert to writing block
ChatGPT
:::writing{variant="standard" id="61483" title="نوع القيمة التي ترجعها الدالة"}
نوع القيمة التي ترجعها الدالة
:::
<details>
<summary>ليه الـget user بترجع promise user من الـuser دي بتاعت الـfunction، يعني الـfunction بستنية promise. الـPromise ده لما ينجح يكون نوع الـban بتاعته IUser، الـreturn الـIUser. أما مع الـget user نفسه مكتوب إن success response IUser، اللي هي بقى لما تنجح، دي البيانات فعلاً الراجعة، البيانات نفسها اللي راجعة.</summary>
user
ليه الـget user بترجع promise user من الـuser دي بتاعت الـfunction، يعني الـfunction بستنية promise. الـPromise ده لما ينجح يكون نوع الـban بتاعته IUser، الـreturn الـIUser. أما مع الـget user نفسه مكتوب إن success response IUser، اللي هي بقى لما تنجح، دي البيانات فعلاً الراجعة، البيانات نفسها اللي راجعة.
ChatGPT
إجابتك **قريبة جدًا، 9/10**، وأنت فاهم الفرق الأساسي، لكن محتاجين نفصل بين `Promise` و`IUser` بشكل أدق. ⏱️ 4 دقائق

عندنا:

```ts
const getUser = async (): Promise<IUser> => {
  const response = await axios.get<ISuccessResponse<IUser>>(
    "/users/10"
  );

  return response.data.payload;
};
```

## 1. `axios.get<ISuccessResponse<IUser>>`

دي بتقول:

```text
Axios
  ↓
Response Data
  ↓
ISuccessResponse<IUser>
```

يعني:

```ts
response.data
```

نوعها:

```ts
ISuccessResponse<IUser>
```

والـ `payload` جواها:

```ts
response.data.payload
```

نوعه:

```ts
IUser
```

---

## 2. طيب `Promise<IUser>`؟

دي مرتبطة **بالـ function نفسها**:

```ts
const getUser = async (): Promise<IUser>
```

معناها:

> الـ function دي asynchronous، وهترجع `Promise`، والـ value الموجودة داخل الـ Promise لما تكتمل هتكون `IUser`.

يعني:

```text
getUser()
   ↓
Promise<IUser>
   ↓ await
IUser
```

وده مختلف عن:

```text
axios.get()
   ↓
Promise<AxiosResponse<ISuccessResponse<IUser>>>
   ↓ await
AxiosResponse<ISuccessResponse<IUser>>
```

ثم:

```text
response.data.payload
   ↓
IUser
```

---

## 3. وهنا أنت كنت صح في أهم نقطة

أنت قلت إن:

> الـ `ISuccessResponse<IUser>` هي البيانات الراجعة من الـ API.

**صح.**

لكن `getUser` مش بترجع الـ API Response بالكامل.

هي بتعمل:

```ts
return response.data.payload;
```

فهي بتاخد:

```text
ISuccessResponse<IUser>
```

وتستخرج منه:

```text
IUser
```

ثم الـ `async function` تلفه تلقائيًا داخل:

```text
Promise<IUser>
```

### 🧠 احفظ المسار ده

```text
API
 ↓
ISuccessResponse<IUser>
 ↓
response.data
 ↓
payload
 ↓
IUser
 ↓
return
 ↓
Promise<IUser>
```

وده سبب إننا كتبنا:

```ts
Promise<IUser>
```

وليس:

```ts
Promise<ISuccessResponse<IUser>>
```

لأن الـ function في النهاية **بترجع `payload` فقط**.

### ⭐ تقييمك

**9/10**

أنت فاهم الفكرة، والتصحيح الأساسي هو إن `Promise` مش معناها "لما ينجح"، بل هي **حاوية لقيمة مستقبلية**، والـ `await` بيطلع القيمة الموجودة بداخلها.

**الخطوة التالية:** اكتب نوع `getUsers` لو الـ function هترجع `response.data.payload` وفيه Array من Users.
</details>
ليه الـget user بترجع promise user من الـuser دي بتاعت الـfunction، يعني الـfunction بستنية promise. الـPromise ده لما ينجح يكون نوع الـban بتاعته IUser، الـreturn الـIUser. أما مع الـget user نفسه مكتوب إن success response IUser، اللي هي بقى لما تنجح، دي البيانات فعلاً الراجعة، البيانات نفسها اللي راجعة.
ChatGPT
إجابتك **قريبة جدًا، 9/10**، وأنت فاهم الفرق الأساسي، لكن محتاجين نفصل بين `Promise` و`IUser` بشكل أدق. ⏱️ 4 دقائق

عندنا:

```ts
const getUser = async (): Promise<IUser> => {
  const response = await axios.get<ISuccessResponse<IUser>>(
    "/users/10"
  );

  return response.data.payload;
};
```

## 1. `axios.get<ISuccessResponse<IUser>>`

دي بتقول:

```text
Axios
  ↓
Response Data
  ↓
ISuccessResponse<IUser>
```

يعني:

```ts
response.data
```

نوعها:

```ts
ISuccessResponse<IUser>
```

والـ `payload` جواها:

```ts
response.data.payload
```

نوعه:

```ts
IUser
```

---

## 2. طيب `Promise<IUser>`؟

دي مرتبطة **بالـ function نفسها**:

```ts
const getUser = async (): Promise<IUser>
```

معناها:

> الـ function دي asynchronous، وهترجع `Promise`، والـ value الموجودة داخل الـ Promise لما تكتمل هتكون `IUser`.

يعني:

```text
getUser()
   ↓
Promise<IUser>
   ↓ await
IUser
```

وده مختلف عن:

```text
axios.get()
   ↓
Promise<AxiosResponse<ISuccessResponse<IUser>>>
   ↓ await
AxiosResponse<ISuccessResponse<IUser>>
```

ثم:

```text
response.data.payload
   ↓
IUser
```

---

## 3. وهنا أنت كنت صح في أهم نقطة

أنت قلت إن:

> الـ `ISuccessResponse<IUser>` هي البيانات الراجعة من الـ API.

**صح.**

لكن `getUser` مش بترجع الـ API Response بالكامل.

هي بتعمل:

```ts
return response.data.payload;
```

فهي بتاخد:

```text
ISuccessResponse<IUser>
```

وتستخرج منه:

```text
IUser
```

ثم الـ `async function` تلفه تلقائيًا داخل:

```text
Promise<IUser>
```

### 🧠 احفظ المسار ده

```text
API
 ↓
ISuccessResponse<IUser>
 ↓
response.data
 ↓
payload
 ↓
IUser
 ↓
return
 ↓
Promise<IUser>
```

وده سبب إننا كتبنا:

```ts
Promise<IUser>
```

وليس:

```ts
Promise<ISuccessResponse<IUser>>
```

لأن الـ function في النهاية **بترجع `payload` فقط**.

### ⭐ تقييمك

**9/10**

أنت فاهم الفكرة، والتصحيح الأساسي هو إن `Promise` مش معناها "لما ينجح"، بل هي **حاوية لقيمة مستقبلية**، والـ `await` بيطلع القيمة الموجودة بداخلها.

**الخطوة التالية:** اكتب نوع `getUsers` لو الـ function هترجع `response.data.payload` وفيه Array من Users.
<details>
<summary>export const getUser =async () : Promise&lt;IUser&gt;{</summary>
user
export const getUser =async () : Promise<IUser>{
const response =axios.get<IUser[]>("/users")
return response.data.payload;
}
ChatGPT
صحح الكود ده في **3 نقاط** فقط، لأن الفكرة الأساسية عندك بدأت تتكون صح. ⏱️ 5 دقائق

أنت كتبت:

```ts
export const getUser = async (): Promise<IUser> {
  const response = axios.get<IUser[]>("/users")
  return response.data.payload;
}
```

## 1. الـ Function بتاعتك اسمها `getUser`

لكن أنت بتطلب:

```ts
/users
```

وده غالبًا معناه **مجموعة Users**، فالأصح في المثال بتاعنا:

```ts
getUsers
```

وكمان النتيجة هتكون:

```ts
Promise<IUser[]>
```

لأننا بنرجع Array.

---

## 2. نسيت `await`

كتبت:

```ts
const response = axios.get<IUser[]>("/users");
```

لكن `axios.get` بترجع:

```text
Promise<AxiosResponse<...>>
```

إحنا محتاجين نستنى الـ Promise:

```ts
const response = await axios.get<...>("/users");
```

---

## 3. الـ Generic مش `IUser[]` هنا

إحنا افترضنا إن الـ API بيرجع Wrapper:

```json
{
  "status": true,
  "code": 200,
  "payload": [
    {
      "id": 1,
      "name": "Ahmed"
    }
  ]
}
```

إذن الـ Generic الصحيح:

```ts
IApiResponse<IUser[]>
```

وليس:

```ts
IUser[]
```

فتصبح:

```ts
export const getUsers = async (): Promise<IUser[]> => {
  const response = await axios.get<IApiResponse<IUser[]>>(
    "/users"
  );

  return response.data.payload;
};
```

### 🧠 فك الكود

```text
axios.get<IApiResponse<IUser[]>>
              ↓
API Response Wrapper
              ↓
payload
              ↓
IUser[]
              ↓
return
              ↓
Promise<IUser[]>
```

وهنا لازم تلاحظ الفرق الجميل:

```ts
axios.get<IApiResponse<IUser[]>>
```

يصف **ما رجع من الـ API**.

بينما:

```ts
Promise<IUser[]>
```

يصف **ما ترجع الدالة نفسها للمكان الذي استدعاها**.

### 📍 Progress

```text
SECTION 2: TypeScript + Axios

├── AxiosResponse                         ✅
├── Generics مع Axios                     ✅
├── Response Types                        ✅
├── API Response Types                    ✅
├── Discriminated Unions                  ✅
├── unknown vs any                        ✅
└── Type-safe API Layer                   🟢
    ├── API Function                      🟢
    └── Separation from Component         ⬜
```

**الخطوة التالية:** اكتب `getUser(id: number)` بحيث ترجع `Promise<IUser>` باستخدام `IApiResponse<IUser>`.
</details>
export const getUser =async () : Promise<IUser>{
const response =axios.get<IUser[]>("/users")
return response.data.payload;
}
ChatGPT
صحح الكود ده في **3 نقاط** فقط، لأن الفكرة الأساسية عندك بدأت تتكون صح. ⏱️ 5 دقائق

أنت كتبت:

```ts
export const getUser = async (): Promise<IUser> {
  const response = axios.get<IUser[]>("/users")
  return response.data.payload;
}
```

## 1. الـ Function بتاعتك اسمها `getUser`

لكن أنت بتطلب:

```ts
/users
```

وده غالبًا معناه **مجموعة Users**، فالأصح في المثال بتاعنا:

```ts
getUsers
```

وكمان النتيجة هتكون:

```ts
Promise<IUser[]>
```

لأننا بنرجع Array.

---

## 2. نسيت `await`

كتبت:

```ts
const response = axios.get<IUser[]>("/users");
```

لكن `axios.get` بترجع:

```text
Promise<AxiosResponse<...>>
```

إحنا محتاجين نستنى الـ Promise:

```ts
const response = await axios.get<...>("/users");
```

---

## 3. الـ Generic مش `IUser[]` هنا

إحنا افترضنا إن الـ API بيرجع Wrapper:

```json
{
  "status": true,
  "code": 200,
  "payload": [
    {
      "id": 1,
      "name": "Ahmed"
    }
  ]
}
```

إذن الـ Generic الصحيح:

```ts
IApiResponse<IUser[]>
```

وليس:

```ts
IUser[]
```

فتصبح:

```ts
export const getUsers = async (): Promise<IUser[]> => {
  const response = await axios.get<IApiResponse<IUser[]>>(
    "/users"
  );

  return response.data.payload;
};
```

### 🧠 فك الكود

```text
axios.get<IApiResponse<IUser[]>>
              ↓
API Response Wrapper
              ↓
payload
              ↓
IUser[]
              ↓
return
              ↓
Promise<IUser[]>
```

وهنا لازم تلاحظ الفرق الجميل:

```ts
axios.get<IApiResponse<IUser[]>>
```

يصف **ما رجع من الـ API**.

بينما:

```ts
Promise<IUser[]>
```

يصف **ما ترجع الدالة نفسها للمكان الذي استدعاها**.

### 📍 Progress

```text
SECTION 2: TypeScript + Axios

├── AxiosResponse                         ✅
├── Generics مع Axios                     ✅
├── Response Types                        ✅
├── API Response Types                    ✅
├── Discriminated Unions                  ✅
├── unknown vs any                        ✅
└── Type-safe API Layer                   🟢
    ├── API Function                      🟢
    └── Separation from Component         ⬜
```

**الخطوة التالية:** اكتب `getUser(id: number)` بحيث ترجع `Promise<IUser>` باستخدام `IApiResponse<IUser>`.
<details>
<summary>export const getusers = async(id: number): Promise&lt;IUser&gt;{</summary>
user
export const getusers = async(id: number): Promise<IUser>{
const response = await axios.get<IApiResponse<IUser>>(`user/${id}`)
return respinse.data.payload;
}
ChatGPT
الكود بتاعك **ممتاز من ناحية الـ Type design**، وعندك خطأين كتابيين فقط. ⏱️ 3 دقائق

أنت كتبت:

```ts id="8n7r2k"
export const getusers = async (id: number): Promise<IUser> {
  const response = await axios.get<IApiResponse<IUser>>(`user/${id}`);
  return respinse.data.payload;
}
```

### 1. ناقص `=>`

لأنك بتستخدم **Arrow Function**:

```ts id="g8q4w2"
export const getUser = async (id: number): Promise<IUser> => {
```

مش:

```ts id="8s5m1c"
Promise<IUser> {
```

### 2. `respinse` خطأ كتابي

كتبت:

```ts id="m3f7qa"
respinse.data
```

الصحيح:

```ts id="k5z2vb"
response.data
```

### الكود الصحيح

```ts id="v8q3nm"
export const getUser = async (
  id: number
): Promise<IUser> => {
  const response = await axios.get<IApiResponse<IUser>>(
    `/users/${id}`
  );

  return response.data.payload;
};
```

### 🧠 والأهم إنك عملت الـ Type Flow صح

```text id="q4m8vx"
id: number
    ↓
نستخدمه في URL

/users/${id}
    ↓
API Request

IApiResponse<IUser>
    ↓
response.data

payload
    ↓
IUser

return
    ↓
Promise<IUser>
```

ودي نقطة ممتازة جدًا:

```ts id="a7k3pf"
axios.get<IApiResponse<IUser>>
```

لا تعني أن الدالة ترجع `IApiResponse<IUser>`.

الدالة نفسها:

```ts id="v2n6cx"
Promise<IUser>
```

لأنك عملت:

```ts id="j9r4qa"
return response.data.payload;
```

### ⭐ التقييم: 9.5/10

الـ architecture والـ types صح، والأخطاء كانت `=>` وtypo في `response`.

**الخطوة التالية:** اكتب Component بسيط يستدعي `getUser(10)` ويخزن النتيجة في متغير اسمه `user`.
</details>
export const getusers = async(id: number): Promise<IUser>{
const response = await axios.get<IApiResponse<IUser>>(`user/${id}`)
return respinse.data.payload;
}
ChatGPT
الكود بتاعك **ممتاز من ناحية الـ Type design**، وعندك خطأين كتابيين فقط. ⏱️ 3 دقائق

أنت كتبت:

```ts id="8n7r2k"
export const getusers = async (id: number): Promise<IUser> {
  const response = await axios.get<IApiResponse<IUser>>(`user/${id}`);
  return respinse.data.payload;
}
```

### 1. ناقص `=>`

لأنك بتستخدم **Arrow Function**:

```ts id="g8q4w2"
export const getUser = async (id: number): Promise<IUser> => {
```

مش:

```ts id="8s5m1c"
Promise<IUser> {
```

### 2. `respinse` خطأ كتابي

كتبت:

```ts id="m3f7qa"
respinse.data
```

الصحيح:

```ts id="k5z2vb"
response.data
```

### الكود الصحيح

```ts id="v8q3nm"
export const getUser = async (
  id: number
): Promise<IUser> => {
  const response = await axios.get<IApiResponse<IUser>>(
    `/users/${id}`
  );

  return response.data.payload;
};
```

### 🧠 والأهم إنك عملت الـ Type Flow صح

```text id="q4m8vx"
id: number
    ↓
نستخدمه في URL

/users/${id}
    ↓
API Request

IApiResponse<IUser>
    ↓
response.data

payload
    ↓
IUser

return
    ↓
Promise<IUser>
```

ودي نقطة ممتازة جدًا:

```ts id="a7k3pf"
axios.get<IApiResponse<IUser>>
```

لا تعني أن الدالة ترجع `IApiResponse<IUser>`.

الدالة نفسها:

```ts id="v2n6cx"
Promise<IUser>
```

لأنك عملت:

```ts id="j9r4qa"
return response.data.payload;
```

### ⭐ التقييم: 9.5/10

الـ architecture والـ types صح، والأخطاء كانت `=>` وtypo في `response`.

**الخطوة التالية:** اكتب Component بسيط يستدعي `getUser(10)` ويخزن النتيجة في متغير اسمه `user`.
<details>
<summary>import getUser    from "@/api/getUser"</summary>
user

import getUser    from "@/api/getUser"
export function ProfileUser(){
const user =getUser(10)  
return <>
<div>{user}</div>
</>
}
ChatGPT
**التمرين ده ياخد 5 دقائق:** صحّح الكود بحيث `user` يكون فعليًا `IUser` وليس `Promise<IUser>`.

### تقييم الكود

أنت فاهم الفكرة الأساسية صح جدًا:

```ts
const user = getUser(10);
```

لكن هنا فيه نقطة مهمة جدًا:

بما إن `getUser` مكتوبة:

```ts
async function getUser(id: number): Promise<IUser>
```

فإن:

```ts
getUser(10)
```

ترجع:

```ts
Promise<IUser>
```

وليس:

```ts
IUser
```

يعني عندك حاليًا:

```ts
const user = getUser(10);
// user: Promise<IUser>
```

ولذلك لا ينفع تتعامل مع `user` كأنه الـ user نفسه.

---

### الحل

لو الـ component يسمح بـ `async`:

```tsx
import getUser from "@/api/getUser";

export async function ProfileUser() {
  const user = await getUser(10);

  return (
    <>
      <div>{user.name}</div>
    </>
  );
}
```

الـ flow هنا مهم جدًا:

```text
getUser(10)
      ↓
Promise<IUser>
      ↓ await
IUser
      ↓
user.name
```

### نقطة مهمة جدًا

أنت كتبت:

```tsx
<div>{user}</div>
```

حتى بعد `await`، `user` عبارة عن object مثل:

```ts
{
  id: 10,
  name: "Ahmed",
  email: "ahmed@example.com"
}
```

React لا نتعامل معه عادةً كده:

```tsx
<div>{user}</div>
```

لكن نعرض property:

```tsx
<div>{user.name}</div>
```

أو:

```tsx
<div>{user.email}</div>
```

---

## 📍 Roadmap

```text
SECTION 0  Architecture Foundation       ⬜
SECTION 1  Axios Fundamentals             ✅
SECTION 2  TypeScript + Axios             🟢 CURRENT
  ├─ AxiosResponse                        ✅
  ├─ Generics                             ✅
  ├─ Response Types                       ✅
  ├─ API Response Types                   ✅
  ├─ Discriminated Unions                 ✅
  ├─ unknown vs any                       ✅
  └─ Type-safe API Layer
       ├─ API Function                    ✅
       └─ Separation from Component       🟡 ← نحن هنا
SECTION 3  Error Handling                  ⬜
SECTION 4  Interceptors                   ⬜
...
```

**الخطوة التالية فقط:** اكتب لي `ProfileUser` مرة ثانية، لكن هذه المرة استخدم `await` واعرض `user.name`.
</details>

import getUser    from "@/api/getUser"
export function ProfileUser(){
const user =getUser(10)  
return <>
<div>{user}</div>
</>
}
ChatGPT
**التمرين ده ياخد 5 دقائق:** صحّح الكود بحيث `user` يكون فعليًا `IUser` وليس `Promise<IUser>`.

### تقييم الكود

أنت فاهم الفكرة الأساسية صح جدًا:

```ts
const user = getUser(10);
```

لكن هنا فيه نقطة مهمة جدًا:

بما إن `getUser` مكتوبة:

```ts
async function getUser(id: number): Promise<IUser>
```

فإن:

```ts
getUser(10)
```

ترجع:

```ts
Promise<IUser>
```

وليس:

```ts
IUser
```

يعني عندك حاليًا:

```ts
const user = getUser(10);
// user: Promise<IUser>
```

ولذلك لا ينفع تتعامل مع `user` كأنه الـ user نفسه.

---

### الحل

لو الـ component يسمح بـ `async`:

```tsx
import getUser from "@/api/getUser";

export async function ProfileUser() {
  const user = await getUser(10);

  return (
    <>
      <div>{user.name}</div>
    </>
  );
}
```

الـ flow هنا مهم جدًا:

```text
getUser(10)
      ↓
Promise<IUser>
      ↓ await
IUser
      ↓
user.name
```

### نقطة مهمة جدًا

أنت كتبت:

```tsx
<div>{user}</div>
```

حتى بعد `await`، `user` عبارة عن object مثل:

```ts
{
  id: 10,
  name: "Ahmed",
  email: "ahmed@example.com"
}
```

React لا نتعامل معه عادةً كده:

```tsx
<div>{user}</div>
```

لكن نعرض property:

```tsx
<div>{user.name}</div>
```

أو:

```tsx
<div>{user.email}</div>
```

---

## 📍 Roadmap

```text
SECTION 0  Architecture Foundation       ⬜
SECTION 1  Axios Fundamentals             ✅
SECTION 2  TypeScript + Axios             🟢 CURRENT
  ├─ AxiosResponse                        ✅
  ├─ Generics                             ✅
  ├─ Response Types                       ✅
  ├─ API Response Types                   ✅
  ├─ Discriminated Unions                 ✅
  ├─ unknown vs any                       ✅
  └─ Type-safe API Layer
       ├─ API Function                    ✅
       └─ Separation from Component       🟡 ← نحن هنا
SECTION 3  Error Handling                  ⬜
SECTION 4  Interceptors                   ⬜
...
```

**الخطوة التالية فقط:** اكتب لي `ProfileUser` مرة ثانية، لكن هذه المرة استخدم `await` واعرض `user.name`.
<details>
<summary>import getUser from "@/api/getUser";</summary>
user
import getUser from "@/api/getUser";

export async function ProfileUser() {
  const user = await getUser(10);

  return (
    <>
      <div>{user.name}</div>
    </>
  );
}

ليه كتبت asunc and await 
رغم انهم استخدمتهم في ملف getusers
ChatGPT
**خد 5 دقائق وركّز في الفرق بين `async` و `await` في كل مستوى، لأن دي نقطة أساسية جدًا في فهم الـ `API Layer`.**

أيوه، سؤالك ممتاز، لأنك لاحظت حاجة مهمة جدًا:

أنت بالفعل كتبت `async` و `await` داخل `getUser`:

```ts
export const getUser = async (id: number): Promise<IUser> => {
  const response = await axios.get<IApiResponse<IUser>>(
    `/users/${id}`
  );

  return response.data.payload;
};
```

فليه كتبناهم تاني في الـ Component؟

## الإجابة ببساطة

لأن عندنا **دالتين مختلفتين**:

```text
getUser()
   ↓
تتعامل مع Axios والـ API

ProfileUser()
   ↓
تتعامل مع نتيجة getUser()
```

كل دالة عندها عملية `async` خاصة بها.

---

### داخل `getUser`

هنا:

```ts
const response = await axios.get(...);
```

نحن ننتظر **Axios**.

بدون `await`:

```ts
axios.get(...)
```

ترجع:

```ts
Promise<AxiosResponse<...>>
```

لكن `await` يحولها إلى:

```ts
AxiosResponse<...>
```

ثم نستخرج:

```ts
return response.data.payload;
```

وبالتالي `getUser` نفسها ترجع:

```ts
Promise<IUser>
```

---

### داخل `ProfileUser`

لما تعمل:

```ts
const user = getUser(10);
```

أنت الآن تستدعي دالة **async**.

إذن:

```ts
getUser(10)
```

ترجع:

```ts
Promise<IUser>
```

ولذلك نحتاج:

```ts
const user = await getUser(10);
```

فيصبح:

```ts
user: IUser
```

---

## أهم نقطة

الـ `await` داخل `getUser` **لا يجعل كل من يستدعي `getUser` يحصل على `IUser` مباشرة**.

هو فقط ينتظر العملية **داخل `getUser`**.

شوفها كده:

```text
ProfileUser
     │
     │ await getUser(10)
     ▼
getUser
     │
     │ await axios.get()
     ▼
   Axios
```

كل مستوى ينتظر العملية التي **هو نفسه** يتعامل معها.

---

### مثال بسيط جدًا

```ts
async function getUser() {
  const response = await axios.get("/user");

  return response.data;
}
```

الدالة دي في النهاية:

```ts
getUser()
```

ما زالت ترجع:

```ts
Promise<IUser>
```

لذلك لو عندك:

```ts
async function ProfileUser() {
  const user = await getUser();
}
```

الـ `await` هنا ينتظر **Promise الخاصة بـ `getUser`**.

### قاعدة احفظها

> **كل `async function` ترجع Promise، وأي مكان يريد الحصول على القيمة الموجودة داخل الـ Promise يحتاج `await` أو يتعامل مع الـ Promise بطريقة أخرى.**

وده بالضبط سبب إننا كتبنا:

```ts
async function ProfileUser()
```

لأننا نريد استخدام:

```ts
await getUser(10)
```

داخلها.

**الخطوة التالية:** اكتب لي ماذا سيكون Type كل متغير في السطرين دول، بدون تشغيل الكود:

```ts
const response = await axios.get<IApiResponse<IUser>>("/users/10");
const user = await getUser(10);
```
</details>
import getUser from "@/api/getUser";

export async function ProfileUser() {
  const user = await getUser(10);

  return (
    <>
      <div>{user.name}</div>
    </>
  );
}

ليه كتبت asunc and await 
رغم انهم استخدمتهم في ملف getusers
ChatGPT
**خد 5 دقائق وركّز في الفرق بين `async` و `await` في كل مستوى، لأن دي نقطة أساسية جدًا في فهم الـ `API Layer`.**

أيوه، سؤالك ممتاز، لأنك لاحظت حاجة مهمة جدًا:

أنت بالفعل كتبت `async` و `await` داخل `getUser`:

```ts
export const getUser = async (id: number): Promise<IUser> => {
  const response = await axios.get<IApiResponse<IUser>>(
    `/users/${id}`
  );

  return response.data.payload;
};
```

فليه كتبناهم تاني في الـ Component؟

## الإجابة ببساطة

لأن عندنا **دالتين مختلفتين**:

```text
getUser()
   ↓
تتعامل مع Axios والـ API

ProfileUser()
   ↓
تتعامل مع نتيجة getUser()
```

كل دالة عندها عملية `async` خاصة بها.

---

### داخل `getUser`

هنا:

```ts
const response = await axios.get(...);
```

نحن ننتظر **Axios**.

بدون `await`:

```ts
axios.get(...)
```

ترجع:

```ts
Promise<AxiosResponse<...>>
```

لكن `await` يحولها إلى:

```ts
AxiosResponse<...>
```

ثم نستخرج:

```ts
return response.data.payload;
```

وبالتالي `getUser` نفسها ترجع:

```ts
Promise<IUser>
```

---

### داخل `ProfileUser`

لما تعمل:

```ts
const user = getUser(10);
```

أنت الآن تستدعي دالة **async**.

إذن:

```ts
getUser(10)
```

ترجع:

```ts
Promise<IUser>
```

ولذلك نحتاج:

```ts
const user = await getUser(10);
```

فيصبح:

```ts
user: IUser
```

---

## أهم نقطة

الـ `await` داخل `getUser` **لا يجعل كل من يستدعي `getUser` يحصل على `IUser` مباشرة**.

هو فقط ينتظر العملية **داخل `getUser`**.

شوفها كده:

```text
ProfileUser
     │
     │ await getUser(10)
     ▼
getUser
     │
     │ await axios.get()
     ▼
   Axios
```

كل مستوى ينتظر العملية التي **هو نفسه** يتعامل معها.

---

### مثال بسيط جدًا

```ts
async function getUser() {
  const response = await axios.get("/user");

  return response.data;
}
```

الدالة دي في النهاية:

```ts
getUser()
```

ما زالت ترجع:

```ts
Promise<IUser>
```

لذلك لو عندك:

```ts
async function ProfileUser() {
  const user = await getUser();
}
```

الـ `await` هنا ينتظر **Promise الخاصة بـ `getUser`**.

### قاعدة احفظها

> **كل `async function` ترجع Promise، وأي مكان يريد الحصول على القيمة الموجودة داخل الـ Promise يحتاج `await` أو يتعامل مع الـ Promise بطريقة أخرى.**

وده بالضبط سبب إننا كتبنا:

```ts
async function ProfileUser()
```

لأننا نريد استخدام:

```ts
await getUser(10)
```

داخلها.

**الخطوة التالية:** اكتب لي ماذا سيكون Type كل متغير في السطرين دول، بدون تشغيل الكود:

```ts
const response = await axios.get<IApiResponse<IUser>>("/users/10");
const user = await getUser(10);
```
<details>
<summary>response   =IUser</summary>
user
response   =IUser  
user   =IUser  
ChatGPT
**صحّح السطر الأول فقط، وخد 3 دقائق على الفرق بين `response` و `response.data`.**

إجابتك الثانية **صح**، لكن الأولى محتاجة تصحيح بسيط.

### 1. `response`

عندنا:

```ts
const response = await axios.get<IApiResponse<IUser>>("/users/10");
```

الـ `<IApiResponse<IUser>>` بتحدد **نوع `response.data`**، مش نوع `response` نفسه.

إذن:

```ts
response
// AxiosResponse<IApiResponse<IUser>>
```

وليس:

```ts
IUser ❌
```

والـ flow:

```text
axios.get<IApiResponse<IUser>>()
            ↓
Promise<AxiosResponse<IApiResponse<IUser>>>
            ↓ await
AxiosResponse<IApiResponse<IUser>>
            ↓
response.data
            ↓
IApiResponse<IUser>
            ↓
response.data.payload
            ↓
IUser
```

### 2. `user`

عندنا:

```ts
const user = await getUser(10);
```

و`getUser`:

```ts
async function getUser(id: number): Promise<IUser>
```

إذن:

```text
getUser(10)
   ↓
Promise<IUser>
   ↓ await
IUser
```

فإجابتك:

```text
response = IUser       ❌
user     = IUser       ✅
```

والتصحيح النهائي:

```text
response = AxiosResponse<IApiResponse<IUser>>
user     = IUser
```

**الخطوة التالية:** اشرح لي بنفسك لماذا `response` ليس `IUser` رغم أننا كتبنا `<IApiResponse<IUser>>` في `axios.get`.
</details>
response   =IUser  
user   =IUser  
ChatGPT
**صحّح السطر الأول فقط، وخد 3 دقائق على الفرق بين `response` و `response.data`.**

إجابتك الثانية **صح**، لكن الأولى محتاجة تصحيح بسيط.

### 1. `response`

عندنا:

```ts
const response = await axios.get<IApiResponse<IUser>>("/users/10");
```

الـ `<IApiResponse<IUser>>` بتحدد **نوع `response.data`**، مش نوع `response` نفسه.

إذن:

```ts
response
// AxiosResponse<IApiResponse<IUser>>
```

وليس:

```ts
IUser ❌
```

والـ flow:

```text
axios.get<IApiResponse<IUser>>()
            ↓
Promise<AxiosResponse<IApiResponse<IUser>>>
            ↓ await
AxiosResponse<IApiResponse<IUser>>
            ↓
response.data
            ↓
IApiResponse<IUser>
            ↓
response.data.payload
            ↓
IUser
```

### 2. `user`

عندنا:

```ts
const user = await getUser(10);
```

و`getUser`:

```ts
async function getUser(id: number): Promise<IUser>
```

إذن:

```text
getUser(10)
   ↓
Promise<IUser>
   ↓ await
IUser
```

فإجابتك:

```text
response = IUser       ❌
user     = IUser       ✅
```

والتصحيح النهائي:

```text
response = AxiosResponse<IApiResponse<IUser>>
user     = IUser
```

**الخطوة التالية:** اشرح لي بنفسك لماذا `response` ليس `IUser` رغم أننا كتبنا `<IApiResponse<IUser>>` في `axios.get`.
<details>
<summary>وإحنا هنا قلنا الـ response هو ليس all user إحنا جبنا API response مش all user، لأن الـ response بيبقى عبارة عن API headers، بيبقى فيه الـ API response full came، زي فيه status، وحاجة وفيها نفسها، الـ data نفسها، وهو ده اللي إحنا اتكلمنا عن الـ API response.</summary>
user
وإحنا هنا قلنا الـ response هو ليس all user إحنا جبنا API response مش all user، لأن الـ response بيبقى عبارة عن API headers، بيبقى فيه الـ API response full came، زي فيه status، وحاجة وفيها نفسها، الـ data نفسها، وهو ده اللي إحنا اتكلمنا عن الـ API response.
ChatGPT
**ثبّت الفكرة في جملة واحدة: `response` هو الـ Axios Response الكامل، و`response.data` هو الـ API Response الذي رجعه السيرفر.**

أيوه، أنت قريب جدًا من الصح، وفيه **تصحيح اصطلاحي مهم** عشان المفاهيم ما تختلطش عليك.

عندنا مستويين:

### 1. `response` نفسه

```ts
const response = await axios.get<IApiResponse<IUser>>("/users/10");
```

هنا:

```ts
response
```

هو **AxiosResponse** الكامل.

يعني فيه حاجات زي:

```text
response
├── data
├── status
├── statusText
├── headers
└── config
```

وده اللي أنت تقصده لما قلت إن فيه `headers` و`status` وغيرها. ✅

---

### 2. `response.data`

ده **الـ JSON اللي السيرفر رجّعه**.

وبحسب الـ API بتاعنا:

```json
{
  "status": true,
  "code": 200,
  "payload": {
    "id": 10,
    "name": "Ahmed"
  }
}
```

إذن:

```ts
response.data
```

نوعه:

```ts
IApiResponse<IUser>
```

وليس `IUser`.

---

### 3. `response.data.payload`

هنا وصلنا أخيرًا للـ User:

```ts
response.data.payload
```

نوعه:

```ts
IUser
```

فالصورة الكاملة:

```text
Axios
  ↓
response
  ↓
AxiosResponse<IApiResponse<IUser>>
  ↓
response.data
  ↓
IApiResponse<IUser>
  ↓
response.data.payload
  ↓
IUser
```

وده بالضبط سبب إننا في `getUser` عملنا:

```ts
return response.data.payload;
```

وبالتالي أخفينا كل تفاصيل Axios والـ API wrapper عن الـ Component.

فالـ Component يقول فقط:

```ts
const user = await getUser(10);
```

ويحصل على:

```ts
IUser
```

**الخطوة التالية:** اكتب لي الفرق بين `response` و`response.data` و`response.data.payload` في 3 سطور من عندك.
</details>
وإحنا هنا قلنا الـ response هو ليس all user إحنا جبنا API response مش all user، لأن الـ response بيبقى عبارة عن API headers، بيبقى فيه الـ API response full came، زي فيه status، وحاجة وفيها نفسها، الـ data نفسها، وهو ده اللي إحنا اتكلمنا عن الـ API response.
ChatGPT
**ثبّت الفكرة في جملة واحدة: `response` هو الـ Axios Response الكامل، و`response.data` هو الـ API Response الذي رجعه السيرفر.**

أيوه، أنت قريب جدًا من الصح، وفيه **تصحيح اصطلاحي مهم** عشان المفاهيم ما تختلطش عليك.

عندنا مستويين:

### 1. `response` نفسه

```ts
const response = await axios.get<IApiResponse<IUser>>("/users/10");
```

هنا:

```ts
response
```

هو **AxiosResponse** الكامل.

يعني فيه حاجات زي:

```text
response
├── data
├── status
├── statusText
├── headers
└── config
```

وده اللي أنت تقصده لما قلت إن فيه `headers` و`status` وغيرها. ✅

---

### 2. `response.data`

ده **الـ JSON اللي السيرفر رجّعه**.

وبحسب الـ API بتاعنا:

```json
{
  "status": true,
  "code": 200,
  "payload": {
    "id": 10,
    "name": "Ahmed"
  }
}
```

إذن:

```ts
response.data
```

نوعه:

```ts
IApiResponse<IUser>
```

وليس `IUser`.

---

### 3. `response.data.payload`

هنا وصلنا أخيرًا للـ User:

```ts
response.data.payload
```

نوعه:

```ts
IUser
```

فالصورة الكاملة:

```text
Axios
  ↓
response
  ↓
AxiosResponse<IApiResponse<IUser>>
  ↓
response.data
  ↓
IApiResponse<IUser>
  ↓
response.data.payload
  ↓
IUser
```

وده بالضبط سبب إننا في `getUser` عملنا:

```ts
return response.data.payload;
```

وبالتالي أخفينا كل تفاصيل Axios والـ API wrapper عن الـ Component.

فالـ Component يقول فقط:

```ts
const user = await getUser(10);
```

ويحصل على:

```ts
IUser
```

**الخطوة التالية:** اكتب لي الفرق بين `response` و`response.data` و`response.data.payload` في 3 سطور من عندك.
<details>
<summary>`response`   === AxiosResponse كامل</summary>
user
`response`   === AxiosResponse كامل 
response.data  ===IUser اللي راجه من السيرفر 
response.data.payload   دي دخلت جواه خصائص ال data
ChatGPT
**خد دقيقتين وثبّت التصحيح ده: أول سطر صح، لكن السطرين الثاني والثالث محتاجين تعديل بسيط.**

أنت فاهم الـ hierarchy، لكن حصل خلط بين **API Response** و **User**.

### الصحيح

```text id="1kq9h3"
response
= AxiosResponse<IApiResponse<IUser>>
```

✅ ده **Axios Response الكامل**.

---

```text id="f4s8ka"
response.data
= IApiResponse<IUser>
```

⚠️ مش `IUser`.

لأن `data` هنا تحتوي الـ API wrapper:

```json id="8p4m2c"
{
  "status": true,
  "code": 200,
  "payload": {
    "id": 10,
    "name": "Ahmed"
  }
}
```

---

```text id="m7q2vd"
response.data.payload
= IUser
```

✅ هنا وصلنا فعلًا للـ User.

يعني:

```text id="w6x1na"
response
    ↓
AxiosResponse
    ↓
response.data
    ↓
IApiResponse<IUser>
    ↓
response.data.payload
    ↓
IUser
```

### والجملة المهمة جدًا

أنت قلت:

> `response.data.payload` دي دخلت جواه خصائص الـ data

الأدق:

**`payload` هي الـ property التي تحتوي على الـ `IUser` نفسه.**

مثلاً:

```ts id="c9t2ka"
response.data.payload.name
```

هنا:

```text
response.data.payload → IUser
                       ↓
                    .name
                       ↓
                    string
```

وده السبب إننا في `getUser` نعمل:

```ts id="x3r7pm"
return response.data.payload;
```

فنحوّل:

```text id="q8m4zs"
IApiResponse<IUser>
        ↓
      payload
        ↓
      IUser
```

وبالتالي الـ Component يستقبل:

```ts id="v2k6pd"
const user = await getUser(10);

// user: IUser
```

**الخطوة التالية:** اكتب من ذاكرتك الـ 3 مستويات: `response` ثم `response.data` ثم `response.data.payload` مع الـ Type لكل واحد.
</details>
`response`   === AxiosResponse كامل 
response.data  ===IUser اللي راجه من السيرفر 
response.data.payload   دي دخلت جواه خصائص ال data
ChatGPT
**خد دقيقتين وثبّت التصحيح ده: أول سطر صح، لكن السطرين الثاني والثالث محتاجين تعديل بسيط.**

أنت فاهم الـ hierarchy، لكن حصل خلط بين **API Response** و **User**.

### الصحيح

```text id="1kq9h3"
response
= AxiosResponse<IApiResponse<IUser>>
```

✅ ده **Axios Response الكامل**.

---

```text id="f4s8ka"
response.data
= IApiResponse<IUser>
```

⚠️ مش `IUser`.

لأن `data` هنا تحتوي الـ API wrapper:

```json id="8p4m2c"
{
  "status": true,
  "code": 200,
  "payload": {
    "id": 10,
    "name": "Ahmed"
  }
}
```

---

```text id="m7q2vd"
response.data.payload
= IUser
```

✅ هنا وصلنا فعلًا للـ User.

يعني:

```text id="w6x1na"
response
    ↓
AxiosResponse
    ↓
response.data
    ↓
IApiResponse<IUser>
    ↓
response.data.payload
    ↓
IUser
```

### والجملة المهمة جدًا

أنت قلت:

> `response.data.payload` دي دخلت جواه خصائص الـ data

الأدق:

**`payload` هي الـ property التي تحتوي على الـ `IUser` نفسه.**

مثلاً:

```ts id="c9t2ka"
response.data.payload.name
```

هنا:

```text
response.data.payload → IUser
                       ↓
                    .name
                       ↓
                    string
```

وده السبب إننا في `getUser` نعمل:

```ts id="x3r7pm"
return response.data.payload;
```

فنحوّل:

```text id="q8m4zs"
IApiResponse<IUser>
        ↓
      payload
        ↓
      IUser
```

وبالتالي الـ Component يستقبل:

```ts id="v2k6pd"
const user = await getUser(10);

// user: IUser
```

**الخطوة التالية:** اكتب من ذاكرتك الـ 3 مستويات: `response` ثم `response.data` ثم `response.data.payload` مع الـ Type لكل واحد.
<details>
<summary>حل انت</summary>
user
حل انت 
ChatGPT
**خد دقيقتين واحفظ التسلسل ده، لأنه من أهم الحاجات في Axios + TypeScript.**

```ts
const response = await axios.get<IApiResponse<IUser>>("/users/10");
```

### المستويات الثلاثة

```text
1️⃣ response
   ↓
AxiosResponse<IApiResponse<IUser>>
```

ده **Axios Response الكامل**، وفيه:

```text
data
status
statusText
headers
config
```

---

```text
2️⃣ response.data
   ↓
IApiResponse<IUser>
```

ده **الـ API Response اللي رجع من السيرفر**:

```json
{
  "status": true,
  "code": 200,
  "payload": {
    "id": 10,
    "name": "Ahmed"
  }
}
```

---

```text
3️⃣ response.data.payload
   ↓
IUser
```

وده **الـ User نفسه**:

```ts
{
  id: 10,
  name: "Ahmed"
}
```

### احفظها بالشكل ده

```text
response
   ↓
AxiosResponse
   ↓
response.data
   ↓
IApiResponse<IUser>
   ↓
response.data.payload
   ↓
IUser
```

وبالتالي لما كتبنا:

```ts
export const getUser = async (id: number): Promise<IUser> => {
  const response = await axios.get<IApiResponse<IUser>>(
    `/users/${id}`
  );

  return response.data.payload;
};
```

إحنا عملنا **abstraction**:

```text
Component
    ↓
getUser()
    ↓
IUser
```

والـ Component **مش محتاج يعرف** إن Axios عنده `AxiosResponse` أو إن الـ API عنده `payload`.

**الخطوة التالية:** نكمل الآن مفهوم **Separation from Component** ونشوف ليه وضع `axios` داخل الـ Component يعتبر تصميمًا سيئًا.
</details>
حل انت 
ChatGPT
**خد دقيقتين واحفظ التسلسل ده، لأنه من أهم الحاجات في Axios + TypeScript.**

```ts
const response = await axios.get<IApiResponse<IUser>>("/users/10");
```

### المستويات الثلاثة

```text
1️⃣ response
   ↓
AxiosResponse<IApiResponse<IUser>>
```

ده **Axios Response الكامل**، وفيه:

```text
data
status
statusText
headers
config
```

---

```text
2️⃣ response.data
   ↓
IApiResponse<IUser>
```

ده **الـ API Response اللي رجع من السيرفر**:

```json
{
  "status": true,
  "code": 200,
  "payload": {
    "id": 10,
    "name": "Ahmed"
  }
}
```

---

```text
3️⃣ response.data.payload
   ↓
IUser
```

وده **الـ User نفسه**:

```ts
{
  id: 10,
  name: "Ahmed"
}
```

### احفظها بالشكل ده

```text
response
   ↓
AxiosResponse
   ↓
response.data
   ↓
IApiResponse<IUser>
   ↓
response.data.payload
   ↓
IUser
```

وبالتالي لما كتبنا:

```ts
export const getUser = async (id: number): Promise<IUser> => {
  const response = await axios.get<IApiResponse<IUser>>(
    `/users/${id}`
  );

  return response.data.payload;
};
```

إحنا عملنا **abstraction**:

```text
Component
    ↓
getUser()
    ↓
IUser
```

والـ Component **مش محتاج يعرف** إن Axios عنده `AxiosResponse` أو إن الـ API عنده `payload`.

**الخطوة التالية:** نكمل الآن مفهوم **Separation from Component** ونشوف ليه وضع `axios` داخل الـ Component يعتبر تصميمًا سيئًا.
<details>
<summary>كمل</summary>
user
كمل
ChatGPT
**خد 10 دقائق، وطبّق الفرق بين الـ Component والـ API Layer عمليًا، لأن ده آخر جزء في `Type-safe API Layer` قبل ما ننتقل للـ Error Handling.**

## 📍 Roadmap

```text
SECTION 0  Architecture Foundation       ⬜
SECTION 1  Axios Fundamentals             ✅
SECTION 2  TypeScript + Axios             🟢 CURRENT
  ├─ AxiosResponse                        ✅
  ├─ Generics                             ✅
  ├─ Response Types                       ✅
  ├─ API Response Types                   ✅
  ├─ Discriminated Unions                 ✅
  ├─ unknown vs any                       ✅
  └─ Type-safe API Layer
       ├─ API Function                    ✅
       └─ Separation from Component       🟡 ← نحن هنا
SECTION 3  Error Handling                  ⬜
SECTION 4  Interceptors                   ⬜
...
```

# Separation from Component

الفكرة ببساطة:

**الـ Component مسؤول عن UI، والـ API Layer مسؤول عن التواصل مع الـ API.**

### ❌ الشكل السيئ

```tsx
export async function ProfileUser() {
  const response = await axios.get<IApiResponse<IUser>>(
    "/users/10"
  );

  const user = response.data.payload;

  return <div>{user.name}</div>;
}
```

الـ Component هنا أصبح يعرف تفاصيل كثيرة:

```text
Component
   │
   ├── يعرف Axios
   ├── يعرف endpoint
   ├── يعرف IApiResponse
   ├── يعرف payload
   └── يعرف IUser
```

وده بيعمل **coupling** بين الـ UI والـ API implementation.

---

# ✅ الشكل الأفضل

نخلي الـ API في ملف منفصل:

```ts
// api/getUser.ts

export const getUser = async (
  id: number
): Promise<IUser> => {
  const response = await axios.get<IApiResponse<IUser>>(
    `/users/${id}`
  );

  return response.data.payload;
};
```

والـ Component:

```tsx
import { getUser } from "@/api/getUser";

export async function ProfileUser() {
  const user = await getUser(10);

  return <div>{user.name}</div>;
}
```

شوف الفرق:

```text
             API Layer
                │
                │ Axios
                ▼
             Server
                │
                ▼
          IApiResponse<IUser>
                │
                │ payload
                ▼
              IUser
                │
                ▼
           Component
                │
                ▼
                UI
```

الـ Component لا يهتم بالتفاصيل التي حصلت قبل `IUser`.

---

# ليه ده أفضل؟

تخيل عندك 10 Components محتاجين بيانات المستخدم:

```text
ProfileUser
UserCard
UserHeader
UserSidebar
UserSettings
...
```

لو كل واحد يعمل:

```ts
axios.get(...)
```

هيبقى عندك Axios logic متكرر في أماكن كثيرة.

لكن لو عندك:

```ts
getUser()
```

كلهم يستخدموه:

```text
ProfileUser ───┐
UserCard ──────┤
UserHeader ────┼──→ getUser() → API
UserSidebar ───┤
UserSettings ──┘
```

لو غيرت endpoint من:

```ts
/users/10
```

إلى:

```ts
/api/v2/users/10
```

تغيره **في مكان واحد**.

---

## لكن فيه نقطة أعمق

الموضوع مش مجرد:

> "حط Axios في ملف لوحده."

الموضوع هو **Separation of Concerns**.

كل جزء يكون له مسؤولية واضحة.

```text
Component
→ UI

API Function
→ API communication

Types
→ Data shape

Axios
→ HTTP client
```

وده بيخلينا نقدر نغير implementation بدون ما نكسر الـ UI.

مثلاً النهارده:

```ts
getUser()
```

تستخدم Axios.

بكرة ممكن implementation تتغير، والـ Component يفضل:

```ts
const user = await getUser(10);
```

مش فارق معاه إيه اللي حصل تحت.

---

# 🔥 Interview Question

بعد ما شرحنا المفهوم، نقدر نسأل السؤال:

**ليه يعتبر وضع `axios.get()` مباشرة داخل الـ Component تصميمًا أقل جودة من وضعه داخل `API Function` مثل `getUser()`؟**

جاوب من فهمك، وأنا هقيّم إجابتك من **10**.

**الخطوة التالية:** جاوب على سؤال الـ Interview ده في 3 إلى 5 سطور.
</details>
كمل
ChatGPT
**خد 10 دقائق، وطبّق الفرق بين الـ Component والـ API Layer عمليًا، لأن ده آخر جزء في `Type-safe API Layer` قبل ما ننتقل للـ Error Handling.**

## 📍 Roadmap

```text
SECTION 0  Architecture Foundation       ⬜
SECTION 1  Axios Fundamentals             ✅
SECTION 2  TypeScript + Axios             🟢 CURRENT
  ├─ AxiosResponse                        ✅
  ├─ Generics                             ✅
  ├─ Response Types                       ✅
  ├─ API Response Types                   ✅
  ├─ Discriminated Unions                 ✅
  ├─ unknown vs any                       ✅
  └─ Type-safe API Layer
       ├─ API Function                    ✅
       └─ Separation from Component       🟡 ← نحن هنا
SECTION 3  Error Handling                  ⬜
SECTION 4  Interceptors                   ⬜
...
```

# Separation from Component

الفكرة ببساطة:

**الـ Component مسؤول عن UI، والـ API Layer مسؤول عن التواصل مع الـ API.**

### ❌ الشكل السيئ

```tsx
export async function ProfileUser() {
  const response = await axios.get<IApiResponse<IUser>>(
    "/users/10"
  );

  const user = response.data.payload;

  return <div>{user.name}</div>;
}
```

الـ Component هنا أصبح يعرف تفاصيل كثيرة:

```text
Component
   │
   ├── يعرف Axios
   ├── يعرف endpoint
   ├── يعرف IApiResponse
   ├── يعرف payload
   └── يعرف IUser
```

وده بيعمل **coupling** بين الـ UI والـ API implementation.

---

# ✅ الشكل الأفضل

نخلي الـ API في ملف منفصل:

```ts
// api/getUser.ts

export const getUser = async (
  id: number
): Promise<IUser> => {
  const response = await axios.get<IApiResponse<IUser>>(
    `/users/${id}`
  );

  return response.data.payload;
};
```

والـ Component:

```tsx
import { getUser } from "@/api/getUser";

export async function ProfileUser() {
  const user = await getUser(10);

  return <div>{user.name}</div>;
}
```

شوف الفرق:

```text
             API Layer
                │
                │ Axios
                ▼
             Server
                │
                ▼
          IApiResponse<IUser>
                │
                │ payload
                ▼
              IUser
                │
                ▼
           Component
                │
                ▼
                UI
```

الـ Component لا يهتم بالتفاصيل التي حصلت قبل `IUser`.

---

# ليه ده أفضل؟

تخيل عندك 10 Components محتاجين بيانات المستخدم:

```text
ProfileUser
UserCard
UserHeader
UserSidebar
UserSettings
...
```

لو كل واحد يعمل:

```ts
axios.get(...)
```

هيبقى عندك Axios logic متكرر في أماكن كثيرة.

لكن لو عندك:

```ts
getUser()
```

كلهم يستخدموه:

```text
ProfileUser ───┐
UserCard ──────┤
UserHeader ────┼──→ getUser() → API
UserSidebar ───┤
UserSettings ──┘
```

لو غيرت endpoint من:

```ts
/users/10
```

إلى:

```ts
/api/v2/users/10
```

تغيره **في مكان واحد**.

---

## لكن فيه نقطة أعمق

الموضوع مش مجرد:

> "حط Axios في ملف لوحده."

الموضوع هو **Separation of Concerns**.

كل جزء يكون له مسؤولية واضحة.

```text
Component
→ UI

API Function
→ API communication

Types
→ Data shape

Axios
→ HTTP client
```

وده بيخلينا نقدر نغير implementation بدون ما نكسر الـ UI.

مثلاً النهارده:

```ts
getUser()
```

تستخدم Axios.

بكرة ممكن implementation تتغير، والـ Component يفضل:

```ts
const user = await getUser(10);
```

مش فارق معاه إيه اللي حصل تحت.

---

# 🔥 Interview Question

بعد ما شرحنا المفهوم، نقدر نسأل السؤال:

**ليه يعتبر وضع `axios.get()` مباشرة داخل الـ Component تصميمًا أقل جودة من وضعه داخل `API Function` مثل `getUser()`؟**

جاوب من فهمك، وأنا هقيّم إجابتك من **10**.

**الخطوة التالية:** جاوب على سؤال الـ Interview ده في 3 إلى 5 سطور.
<details>
<summary>إحنا دايمًا بنقسم الـaccess اللي بيجي من الـAPI في ملف واحد عن الـcomponents، يعني الـcomponents مسؤولة عن الـUI، والـaccess مسؤولة عن HTTP request. وبحيث وإحنا كمان نقدر نستخدم، لو عندنا عشر components، نقدر نستخدم access فقط من الملف بتاع الـaccess في العشر components، وهذا بيقلل تكرار الكود. لو حد غيّر بعد كده الـmanifest بتاع الـAPI غيره من مكان. فذاك أقدر إن أنا أعمل separation of concerns.</summary>
user
إحنا دايمًا بنقسم الـaccess اللي بيجي من الـAPI في ملف واحد عن الـcomponents، يعني الـcomponents مسؤولة عن الـUI، والـaccess مسؤولة عن HTTP request. وبحيث وإحنا كمان نقدر نستخدم، لو عندنا عشر components، نقدر نستخدم access فقط من الملف بتاع الـaccess في العشر components، وهذا بيقلل تكرار الكود. لو حد غيّر بعد كده الـmanifest بتاع الـAPI غيره من مكان. فذاك أقدر إن أنا أعمل separation of concerns.
ChatGPT
**إجابتك ممتازة، ودرجتك 10/10، وأنت فهمت الـ architectural reason مش مجرد حفظ القاعدة.**

أنت قلت 4 نقاط أساسية صحيحة:

1. **Separation of Concerns**
   ```text
   Component → UI
   API Layer → HTTP/API communication
   ```

2. **إعادة الاستخدام**
   بدل ما 10 Components تكتب:

   ```ts
   axios.get(...)
   ```

   كل واحد يعمل:

   ```ts
   getUser(10)
   ```

3. **تقليل تكرار الكود**
   الـ API logic موجود في مكان واحد.

4. **سهولة التغيير**
   لو الـ endpoint أو طريقة التعامل مع الـ API اتغيرت، نعدل في الـ API Layer بدل ما نلف على كل Components.

والنقطة اللي قلتها عن تغيير الـ API محتاجة فقط تصحيح المصطلح:

> "manifest"

الأدق هنا نقول **API contract** أو **API implementation** حسب اللي اتغير.

---

### 🧠 الخلاصة المعمارية

```text
Component
   │
   │ calls
   ▼
API Function
   │
   │ uses
   ▼
Axios
   │
   │ HTTP Request
   ▼
Server
```

والـ Component لا يعرف تفاصيل:

```text
Axios
headers
endpoint
payload
AxiosResponse
```

هو يعرف فقط:

```ts
const user = await getUser(10);
```

وده هو جوهر **Separation of Concerns**.

### 📊 Section 2

```text
Type-safe API Layer
├── API Function              ✅
└── Separation from Component ✅
```

**Section 2 أصبح مكتملًا بالكامل.** 🎯

والآن حسب الـ roadmap، **لن نقفز إلى Section 4 أو Interceptors**.

لدينا أولًا:

```text
SECTION 3
Error Handling
```

وسنبدأ فيه من الصفر: **ليه الـ API ممكن يفشل أصلًا، وإيه الفرق بين HTTP Error وNetwork Error وAxiosError**.

**الخطوة التالية:** نبدأ Section 3 بـ **Error Handling Fundamentals**.
</details>
إحنا دايمًا بنقسم الـaccess اللي بيجي من الـAPI في ملف واحد عن الـcomponents، يعني الـcomponents مسؤولة عن الـUI، والـaccess مسؤولة عن HTTP request. وبحيث وإحنا كمان نقدر نستخدم، لو عندنا عشر components، نقدر نستخدم access فقط من الملف بتاع الـaccess في العشر components، وهذا بيقلل تكرار الكود. لو حد غيّر بعد كده الـmanifest بتاع الـAPI غيره من مكان. فذاك أقدر إن أنا أعمل separation of concerns.
ChatGPT
**إجابتك ممتازة، ودرجتك 10/10، وأنت فهمت الـ architectural reason مش مجرد حفظ القاعدة.**

أنت قلت 4 نقاط أساسية صحيحة:

1. **Separation of Concerns**
   ```text
   Component → UI
   API Layer → HTTP/API communication
   ```

2. **إعادة الاستخدام**
   بدل ما 10 Components تكتب:

   ```ts
   axios.get(...)
   ```

   كل واحد يعمل:

   ```ts
   getUser(10)
   ```

3. **تقليل تكرار الكود**
   الـ API logic موجود في مكان واحد.

4. **سهولة التغيير**
   لو الـ endpoint أو طريقة التعامل مع الـ API اتغيرت، نعدل في الـ API Layer بدل ما نلف على كل Components.

والنقطة اللي قلتها عن تغيير الـ API محتاجة فقط تصحيح المصطلح:

> "manifest"

الأدق هنا نقول **API contract** أو **API implementation** حسب اللي اتغير.

---

### 🧠 الخلاصة المعمارية

```text
Component
   │
   │ calls
   ▼
API Function
   │
   │ uses
   ▼
Axios
   │
   │ HTTP Request
   ▼
Server
```

والـ Component لا يعرف تفاصيل:

```text
Axios
headers
endpoint
payload
AxiosResponse
```

هو يعرف فقط:

```ts
const user = await getUser(10);
```

وده هو جوهر **Separation of Concerns**.

### 📊 Section 2

```text
Type-safe API Layer
├── API Function              ✅
└── Separation from Component ✅
```

**Section 2 أصبح مكتملًا بالكامل.** 🎯

والآن حسب الـ roadmap، **لن نقفز إلى Section 4 أو Interceptors**.

لدينا أولًا:

```text
SECTION 3
Error Handling
```

وسنبدأ فيه من الصفر: **ليه الـ API ممكن يفشل أصلًا، وإيه الفرق بين HTTP Error وNetwork Error وAxiosError**.

**الخطوة التالية:** نبدأ Section 3 بـ **Error Handling Fundamentals**.
<details>
<summary>نبدأ Section 3 بـ **Error Handling Fundamentals**</summary>
user
نبدأ Section 3 بـ **Error Handling Fundamentals**  
ChatGPT
**ابدأ بـ 10 دقائق: افهم أولًا يعني إيه Error في الـ API قبل ما نلمس `AxiosError` أو نكتب `try/catch`.**

## 📍 Roadmap

```text
SECTION 0  Architecture Foundation       ⬜
SECTION 1  Axios Fundamentals             ✅
SECTION 2  TypeScript + Axios             ✅
SECTION 3  Error Handling                 🟢 CURRENT
  ├─ Error Handling Fundamentals          🟡 ← نحن هنا
  ├─ HTTP Errors
  ├─ Network Errors
  ├─ AxiosError
  ├─ Error Normalization
  └─ Custom ApiError
SECTION 4  Interceptors                   ⬜
...
```
</details>
نبدأ Section 3 بـ **Error Handling Fundamentals**  
ChatGPT
**ابدأ بـ 10 دقائق: افهم أولًا يعني إيه Error في الـ API قبل ما نلمس `AxiosError` أو نكتب `try/catch`.**

## 📍 Roadmap

```text
SECTION 0  Architecture Foundation       ⬜
SECTION 1  Axios Fundamentals             ✅
SECTION 2  TypeScript + Axios             ✅
SECTION 3  Error Handling                 🟢 CURRENT
  ├─ Error Handling Fundamentals          🟡 ← نحن هنا
  ├─ HTTP Errors
  ├─ Network Errors
  ├─ AxiosError
  ├─ Error Normalization
  └─ Custom ApiError
SECTION 4  Interceptors                   ⬜
...
```