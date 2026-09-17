# 1. يعني إيه `data`؟

لما تبعت Request للسيرفر، ممكن تحتاج تبعت **بيانات فعلية**.

مثلاً عايز تعمل User جديد:

```ts
const userData = {
  name: "Ahmed",
  email: "ahmed@example.com",
};
```

البيانات دي لازم تروح للسيرفر.

مع Axios:

```ts
await axios.post("/users", userData);
```

هنا:

```text
"/users"      → URL
userData      → Request Body
```

وفي Axios بنسمي الـ Body غالبًا:

```text
data
```

---

# 2. طيب فين `data` في Axios؟

بص على الشكل ده:

```ts
axios.post(url, data, config);
```

مثلاً:

```ts
axios.post(
  "/users",
  {
    name: "Ahmed",
    email: "ahmed@example.com",
  }
);
```

الـ object التاني هو الـ `data`.

يعني:

```ts
axios.post("/users", data);
```

ممكن نفهمها كده:

```text
POST
  ↓
URL
  ↓
"/users"

DATA
  ↓
{
  name: "Ahmed",
  email: "ahmed@example.com"
}
```

---

# 3. الفرق المهم جدًا بين `params` و `data`

دي من أهم الحاجات اللي لازم تثبت.

### `params`

بتتحول إلى Query Parameters في الـ URL:

```ts
axios.get("/users", {
  params: {
    page: 2,
    limit: 10,
  },
});
```

الطلب النهائي تقريبًا:

```text
GET /users?page=2&limit=10
```

يعني:

```text
params → URL
```

---

### `data`

بتتحط في Request Body:

```ts
axios.post("/users", {
  name: "Ahmed",
  email: "ahmed@example.com",
});
```

تقريبًا:

```text
POST /users

Body:
{
  "name": "Ahmed",
  "email": "ahmed@example.com"
}
```

يعني:

```text
data → Body
```

احفظ الجملة دي:

> **params في الـ URL، data في الـ Body.**

---

# 4. طب ليه بنستخدم `data` مع POST؟

لأن POST غالبًا بيستخدم لإنشاء Resource جديد.

مثلاً:

```ts
const userData = {
  name: "Ahmed",
  email: "ahmed@example.com",
};

const response = await axios.post<User>(
  "/users",
  userData
);
```

هنا عندنا حاجتين مختلفتين:

```ts
<User>
```

دي بتقول لـ TypeScript:

> أنا متوقع الـ Response يرجع User.

أما:

```ts
userData
```

دي البيانات اللي أنا ببعتها للسيرفر.

يعني:

```text
<User>       → Response
userData     → Request Body
```

وده فرق مهم جدًا.

---

# 5. `data` مش بس مع POST

ممكن تستخدم Body مع:

```text
POST
PUT
PATCH
```

مثلاً PATCH:

```ts
await axios.patch(
  "/users/10",
  {
    email: "new@example.com",
  }
);
```

هنا:

```text
URL
/users/10

DATA / BODY
{
  email: "new@example.com"
}
```

إنت بتقول للسيرفر:

> عدل الـ email الخاص بالـ User رقم 10.

---

# 6. الشكل الكامل مع `config`

دلوقتي نقدر نشوف الصورة كاملة:

```ts
axios.post(
  "/users",
  {
    name: "Ahmed",
    email: "ahmed@example.com",
  },
  {
    headers: {
      Authorization: "Bearer token",
    },
  }
);
```

عندنا 3 أجزاء:

```text
axios.post(
   ↓
URL
   ↓
DATA
   ↓
CONFIG
)
```

يعني:

```text
"/users"
   ↓
URL

{
  name: "Ahmed",
  email: "..."
}
   ↓
DATA / BODY

{
  headers: {...}
}
   ↓
CONFIG
```

وده الشكل اللي عايزك تشوفه في دماغك لما تشوف Axios.

## 🎯 تدريب سريع

اكتب Request يعمل `POST` على:

```text
/users
```

ويبعت:

```text
name = "Mohamed"
email = "mohamed@example.com"
age = 30
```

**مهم:** متستخدمش `params` هنا. البيانات دي لازم تكون في `data`.

**الخطوة التالية:** اكتب الكود بنفسك لـ `axios.post("/users", ...)`.
<details>
<summary>Convert to writing block</summary>
user
Convert to writing block
ChatGPT
:::writing{variant="standard" id="58321"}
axios
:::
</details>
Convert to writing block
ChatGPT
:::writing{variant="standard" id="58321"}
axios
:::
<details>
<summary>Convert to writing block</summary>
user
Convert to writing block
ChatGPT
:::writing{variant="standard" id="74126"}
```json
{
  "name": "Ahmed",
  "email": "ahmed@example.com"
}
```
:::
</details>
Convert to writing block
ChatGPT
:::writing{variant="standard" id="74126"}
```json
{
  "name": "Ahmed",
  "email": "ahmed@example.com"
}
```
:::
<details>
<summary>Convert to writing block</summary>
user
Convert to writing block
ChatGPT
:::writing{variant="standard" id="39217"}
name: "Ahmed",
email: "ahmed@example.com",
:::
</details>
Convert to writing block
ChatGPT
:::writing{variant="standard" id="39217"}
name: "Ahmed",
email: "ahmed@example.com",
:::
<details>
<summary>interface ICreateUser{</summary>
user
interface ICreateUser{
name: string,
email: string,
age = number,
}

interface IUser{
name: string,
email: string,
age: number,
id: number,


const CreateUser: ICreateUser ={
name = "Mohamed"
email = "mohamed@example.com"
age = 30
}

const response=await axios.post<IUser>("/users",CreateUser)  
ChatGPT
عدّل الكود ده أولًا، لأن عندك 3 أخطاء Syntax، وبعدها هنثبت الفرق بين `ICreateUser` و `IUser`. ⏱️ 7 دقائق.

### 1. الـ Interfaces

عندك هنا:

```ts
age = number
```

لازم تكون:

```ts
age: number
```

وكمان لازم تقفل `IUser` بـ `}`.

الصحيح:

```ts
interface ICreateUser {
  name: string;
  email: string;
  age: number;
}

interface IUser {
  name: string;
  email: string;
  age: number;
  id: number;
}
```

### 2. الـ Object

داخل الـ object استخدمت `=`، والصحيح `:`:

```ts
const createUser: ICreateUser = {
  name: "Mohamed",
  email: "mohamed@example.com",
  age: 30,
};
```

لاحظ:

```text
Interface definition  → :
Object value           → :
```

### 3. الـ Axios Request

الكود بتاعك هنا فكرته **صحيحة تمامًا**:

```ts
const response = await axios.post<IUser>(
  "/users",
  createUser
);
```

وهنا عندنا مفهومين مختلفين جدًا:

```text
ICreateUser
     ↓
البيانات التي أنا هبعتها للسيرفر

IUser
     ↓
البيانات التي أتوقع السيرفر يرجعها
```

يعني لو السيرفر رجع:

```json
{
  "id": 15,
  "name": "Mohamed",
  "email": "mohamed@example.com",
  "age": 30
}
```

فـ:

```ts
response.data
```

هيكون TypeScript عارف إنه:

```ts
IUser
```

والـ `id` موجود في الـ Response، لكن مش موجود في البيانات اللي أنت أنشأت بيها المستخدم.

**تقييمك هنا: 9.5/10.** الفكرة الأساسية عندك صحيحة، والأخطاء كانت Syntax فقط.

**الخطوة التالية:** جاوب على السؤال ده: **ليه استخدمنا `ICreateUser` للـ `createUser`، لكن استخدمنا `<IUser>` مع `axios.post`؟**
</details>
interface ICreateUser{
name: string,
email: string,
age = number,
}

interface IUser{
name: string,
email: string,
age: number,
id: number,


const CreateUser: ICreateUser ={
name = "Mohamed"
email = "mohamed@example.com"
age = 30
}

const response=await axios.post<IUser>("/users",CreateUser)  
ChatGPT
عدّل الكود ده أولًا، لأن عندك 3 أخطاء Syntax، وبعدها هنثبت الفرق بين `ICreateUser` و `IUser`. ⏱️ 7 دقائق.

### 1. الـ Interfaces

عندك هنا:

```ts
age = number
```

لازم تكون:

```ts
age: number
```

وكمان لازم تقفل `IUser` بـ `}`.

الصحيح:

```ts
interface ICreateUser {
  name: string;
  email: string;
  age: number;
}

interface IUser {
  name: string;
  email: string;
  age: number;
  id: number;
}
```

### 2. الـ Object

داخل الـ object استخدمت `=`، والصحيح `:`:

```ts
const createUser: ICreateUser = {
  name: "Mohamed",
  email: "mohamed@example.com",
  age: 30,
};
```

لاحظ:

```text
Interface definition  → :
Object value           → :
```

### 3. الـ Axios Request

الكود بتاعك هنا فكرته **صحيحة تمامًا**:

```ts
const response = await axios.post<IUser>(
  "/users",
  createUser
);
```

وهنا عندنا مفهومين مختلفين جدًا:

```text
ICreateUser
     ↓
البيانات التي أنا هبعتها للسيرفر

IUser
     ↓
البيانات التي أتوقع السيرفر يرجعها
```

يعني لو السيرفر رجع:

```json
{
  "id": 15,
  "name": "Mohamed",
  "email": "mohamed@example.com",
  "age": 30
}
```

فـ:

```ts
response.data
```

هيكون TypeScript عارف إنه:

```ts
IUser
```

والـ `id` موجود في الـ Response، لكن مش موجود في البيانات اللي أنت أنشأت بيها المستخدم.

**تقييمك هنا: 9.5/10.** الفكرة الأساسية عندك صحيحة، والأخطاء كانت Syntax فقط.

**الخطوة التالية:** جاوب على السؤال ده: **ليه استخدمنا `ICreateUser` للـ `createUser`، لكن استخدمنا `<IUser>` مع `axios.post`؟**
<details>
<summary>استخدمنا `ICreateUser` للـ `createUser`</summary>
user
استخدمنا `ICreateUser` للـ `createUser`  
عشان تقطتين الاولي اني بعرف اكسيوس نو عالبيانات اللي انا هرسلها 
واتو كمبوليت اثناء التطوير 
<IUser> لو انا مستني وعد عندما ينجح البيانات هتكون من نوع IUser
ChatGPT
إجابتك **صحيحة جدًا من ناحية الفكرة**، وفيها نقطة مهمة جدًا أنت فاهمها صح. ⏱️ دقيقتان

خلينا نصيغها بشكل أدق تقنيًا:

### 1. `ICreateUser`

```ts
const createUser: ICreateUser = {
  name: "Mohamed",
  email: "mohamed@example.com",
  age: 30,
};
```

استخدمنا `ICreateUser` عشان نحدد **شكل البيانات اللي إحنا هنرسلها في Request Body**.

وده بيساعد TypeScript وبيوفر:

- Type safety
- Autocomplete
- اكتشاف الأخطاء أثناء التطوير

مثلاً لو كتبت:

```ts
const createUser: ICreateUser = {
  name: "Mohamed",
  email: "mohamed@example.com",
  age: "30",
};
```

TypeScript هيقولك إن `"30"` مش `number`.

---

### 2. `<IUser>`

```ts
const response = await axios.post<IUser>(
  "/users",
  createUser
);
```

هنا `<IUser>` **مش معناها إن Axios هيبعت `IUser`**.

هي معناها:

> أنا متوقع إن الـ response data اللي هترجع من السيرفر تكون من النوع `IUser`.

وبالتالي:

```ts
response.data
```

هيكون TypeScript عارف إنها:

```ts
IUser
```

---

### نقطة صغيرة جدًا في إجابتك

قلت:

> لو أنا مستني Promise عندما ينجح البيانات هتكون من نوع IUser

المعنى قريب، لكن الأدق:

```ts
axios.post<IUser>()
```

يرجع:

```ts
Promise<AxiosResponse<IUser>>
```

ولما تعمل:

```ts
await
```

تحصل على:

```ts
AxiosResponse<IUser>
```

والبيانات نفسها:

```ts
response.data
```

نوعها:

```ts
IUser
```

وده مهم جدًا لأننا بعد كده هنفهم **AxiosResponse والـ Generics** بشكل أعمق.

### 📍 Progress

```text
Request Configuration

├── Config Object     ✅
├── params            ✅
├── headers           ✅
├── data              ✅ ← خلصنا
├── timeout           ⬜ ← التالي
└── signal            ⬜
```

**الخطوة التالية:** نبدأ `timeout` ونفهم بالضبط ماذا يحدث عندما السيرفر يتأخر في الرد.
</details>
استخدمنا `ICreateUser` للـ `createUser`  
عشان تقطتين الاولي اني بعرف اكسيوس نو عالبيانات اللي انا هرسلها 
واتو كمبوليت اثناء التطوير 
<IUser> لو انا مستني وعد عندما ينجح البيانات هتكون من نوع IUser
ChatGPT
إجابتك **صحيحة جدًا من ناحية الفكرة**، وفيها نقطة مهمة جدًا أنت فاهمها صح. ⏱️ دقيقتان

خلينا نصيغها بشكل أدق تقنيًا:

### 1. `ICreateUser`

```ts
const createUser: ICreateUser = {
  name: "Mohamed",
  email: "mohamed@example.com",
  age: 30,
};
```

استخدمنا `ICreateUser` عشان نحدد **شكل البيانات اللي إحنا هنرسلها في Request Body**.

وده بيساعد TypeScript وبيوفر:

- Type safety
- Autocomplete
- اكتشاف الأخطاء أثناء التطوير

مثلاً لو كتبت:

```ts
const createUser: ICreateUser = {
  name: "Mohamed",
  email: "mohamed@example.com",
  age: "30",
};
```

TypeScript هيقولك إن `"30"` مش `number`.

---

### 2. `<IUser>`

```ts
const response = await axios.post<IUser>(
  "/users",
  createUser
);
```

هنا `<IUser>` **مش معناها إن Axios هيبعت `IUser`**.

هي معناها:

> أنا متوقع إن الـ response data اللي هترجع من السيرفر تكون من النوع `IUser`.

وبالتالي:

```ts
response.data
```

هيكون TypeScript عارف إنها:

```ts
IUser
```

---

### نقطة صغيرة جدًا في إجابتك

قلت:

> لو أنا مستني Promise عندما ينجح البيانات هتكون من نوع IUser

المعنى قريب، لكن الأدق:

```ts
axios.post<IUser>()
```

يرجع:

```ts
Promise<AxiosResponse<IUser>>
```

ولما تعمل:

```ts
await
```

تحصل على:

```ts
AxiosResponse<IUser>
```

والبيانات نفسها:

```ts
response.data
```

نوعها:

```ts
IUser
```

وده مهم جدًا لأننا بعد كده هنفهم **AxiosResponse والـ Generics** بشكل أعمق.

### 📍 Progress

```text
Request Configuration

├── Config Object     ✅
├── params            ✅
├── headers           ✅
├── data              ✅ ← خلصنا
├── timeout           ⬜ ← التالي
└── signal            ⬜
```

**الخطوة التالية:** نبدأ `timeout` ونفهم بالضبط ماذا يحدث عندما السيرفر يتأخر في الرد.
<details>
<summary>`timeout`</summary>
user
`timeout`   
ChatGPT
خلّي هدف الدرس ده واحد: افهم `timeout` كـ **حد زمني للـ request**، ومتى Axios يعتبر الطلب فشل بسبب التأخير. ⏱️ 8 دقائق.

## 📍 Roadmap

```text
SECTION 0: Architecture Foundation       ⬜

SECTION 1: Axios Fundamentals             🟢

GET                                      ✅
POST                                     ✅
PUT                                      ✅
PATCH                                    ✅
DELETE                                   ✅

Request Configuration
├── Config Object                        ✅
├── params                               ✅
├── headers                              ✅
├── data                                 ✅
├── timeout                              🟢 ← إحنا هنا
└── signal                               ⬜

SECTION 2: TypeScript + Axios             ⬜
SECTION 3: Error Handling                 ⬜
SECTION 4: Interceptors                   ⬜
SECTION 5: API Architecture               ⬜
SECTION 6: TanStack Query                 ⬜
...
```
</details>
`timeout`   
ChatGPT
خلّي هدف الدرس ده واحد: افهم `timeout` كـ **حد زمني للـ request**، ومتى Axios يعتبر الطلب فشل بسبب التأخير. ⏱️ 8 دقائق.

## 📍 Roadmap

```text
SECTION 0: Architecture Foundation       ⬜

SECTION 1: Axios Fundamentals             🟢

GET                                      ✅
POST                                     ✅
PUT                                      ✅
PATCH                                    ✅
DELETE                                   ✅

Request Configuration
├── Config Object                        ✅
├── params                               ✅
├── headers                              ✅
├── data                                 ✅
├── timeout                              🟢 ← إحنا هنا
└── signal                               ⬜

SECTION 2: TypeScript + Axios             ⬜
SECTION 3: Error Handling                 ⬜
SECTION 4: Interceptors                   ⬜
SECTION 5: API Architecture               ⬜
SECTION 6: TanStack Query                 ⬜
...
```