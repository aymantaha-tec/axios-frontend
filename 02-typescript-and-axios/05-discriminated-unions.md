# 1. المشكلة

تخيل إن الـ API ممكن يرجع **نجاح**:

```json id="w5h2a8"
{
  "status": true,
  "code": 200,
  "payload": {
    "id": 10,
    "name": "Ahmed"
  }
}
```

أو **خطأ**:

```json id="q1n9bx"
{
  "status": false,
  "code": 400,
  "message": "Invalid email",
  "errors": [
    "Email is required"
  ]
}
```

لاحظ إن الاتنين عندهم:

```text id="0t8p4v"
status
code
```

لكن باقي البيانات مختلفة.

---

# 2. نعمل Type لكل حالة

### Success

```ts id="e2k7pz"
interface ISuccessResponse<T> {
  status: true;
  code: number;
  payload: T;
}
```

### Error

```ts id="w9c4mx"
interface IErrorResponse<R> {
  status: false;
  code: number;
  message: string;
  errors: R[];
}
```

ركز في:

```ts id="q7f3za"
status: true
```

و:

```ts id="b8m1ck"
status: false
```

مش:

```ts id="a9r2vx"
status: boolean
```

ليه؟

لأننا عايزين `status` يكون هو **العلامة المميزة** اللي تقول لـ TypeScript أي Response عندنا.

---

# 3. هنا يأتي الـ Union

نعمل:

```ts id="f4x8qs"
type IApiResponse<T, R> =
  | ISuccessResponse<T>
  | IErrorResponse<R>;
```

معنى `|`:

> الـ Type ممكن يكون **واحد من النوعين**.

يعني:

```text id="6v3k1p"
IApiResponse
      │
      ├── ISuccessResponse
      │
      └── IErrorResponse
```

---

# 4. طيب فين كلمة Discriminated؟

عندنا property مشتركة بين النوعين:

```ts id="7c2m5n"
status
```

لكن قيمتها مختلفة:

```text id="v8q4zs"
Success → status: true

Error   → status: false
```

الـ `status` هنا اسمه:

**Discriminant**

لأنه بيساعد TypeScript يحدد أي Union Member عندنا.

---

# 5. شوف القوة هنا

مثلاً:

```ts id="n5k7wd"
function handleResponse(
  response: IApiResponse<IUser, string>
) {
  if (response.status === true) {
    console.log(response.payload);
  }
}
```

داخل:

```ts id="2w9p4a"
if (response.status === true)
```

TypeScript يفهم:

```text id="9g5x2m"
response
   ↓
ISuccessResponse<IUser>
```

وبالتالي يسمح لك:

```ts id="k8r3cz"
response.payload
```

لأن `payload` موجود في Success Response.

---

# 6. والـ Error

```ts id="j4v8qn"
function handleResponse(
  response: IApiResponse<IUser, string>
) {
  if (response.status === false) {
    console.log(response.message);
    console.log(response.errors);
  }
}
```

هنا TypeScript يعرف:

```text id="z2c6pf"
response
   ↓
IErrorResponse<string>
```

وبالتالي:

```ts id="b7n1rx"
response.message
```

صحيح.

و:

```ts id="k2m9vw"
response.errors
```

نوعها:

```text id="c8f4az"
string[]
```

---

# 7. ودي نقطة مهمة جدًا

لو كتبنا:

```ts id="m5x7qd"
interface ISuccessResponse<T> {
  status: boolean;
  code: number;
  payload: T;
}

interface IErrorResponse<R> {
  status: boolean;
  code: number;
  message: string;
  errors: R[];
}
```

وبعدين:

```ts id="p9k3wf"
type IApiResponse<T, R> =
  | ISuccessResponse<T>
  | IErrorResponse<R>;
```

هنا `status` مش مميز.

ليه؟

لأن الاتنين:

```text id="z6q2bn"
status: boolean
```

ممكن يكونوا `true` أو `false`.

TypeScript مش قادر يقول:

```text id="8j3vkm"
status === true
   ↓
Success
```

لكن لما نكتب:

```ts id="w1f6rt"
status: true
```

و:

```ts id="n8q2cx"
status: false
```

بقى عندنا Discriminated Union حقيقية.

---

# 8. مثال كامل مع Axios

```ts id="v6m2qa"
interface IUser {
  id: number;
  name: string;
  email: string;
}

interface ISuccessResponse<T> {
  status: true;
  code: number;
  payload: T;
}

interface IErrorResponse<R> {
  status: false;
  code: number;
  message: string;
  errors: R[];
}

type IApiResponse<T, R> =
  | ISuccessResponse<T>
  | IErrorResponse<R>;
```

ثم:

```ts id="q9w5kd"
const response = await axios.get<
  IApiResponse<IUser, string>
>("/users/10");
```

الآن:

```text id="x7c4mp"
response.data
      ↓
IApiResponse<IUser, string>
      │
      ├── Success
      │    payload → IUser
      │
      └── Error
           message → string
           errors  → string[]
```

ولو:

```ts id="p3n8za"
if (response.data.status) {
  console.log(response.data.payload.name);
}
```

TypeScript يفهم إنك في حالة Success.

ولو:

```ts id="h6v2qx"
if (!response.data.status) {
  console.log(response.data.message);
}
```

TypeScript يفهم إنك في حالة Error.

---

# 🧠 احفظ الفكرة دي

**Union:**

```text id="m7c4xq"
A | B
```

يعني:

> ممكن يكون A أو B.

**Discriminated Union:**

```text id="r2k8vn"
A | B

مع property مشتركة
تفرق بينهم
```

مثلاً:

```ts id="w5z9hp"
status: true   → Success
status: false  → Error
```

فالـ `status` هو الـ **Discriminant**.

### 🎯 Interview Question

عندك:

```ts id="j8q3mw"
type Result =
  | { status: true; data: string }
  | { status: false; error: string };
```

ليه `status` هنا يعتبر **Discriminant**؟

**الخطوة التالية:** اشرح في جملة واحدة لماذا `status: boolean` لا يكفي لعمل Discriminated Union حقيقية.
<details>
<summary>هنا الـstatus هو يعتبر discriminated عشان خاطر فيه قيمة أو محددة value اللي هي بتروح وfalse، فهنا فيه مميزة. طيب نرجع للجزئية التانية، حتة إن أنا أخلي الـstatus بـBoolean أنا كده مش عامل discriminated onion حقيقية، ليه؟ لأن هنا أنا ما أعرفش هي هتشتغل بيانات، أنا ما أعرفش هي true ولا false، ومرتين عنده Boolean ممكن تبقى true وممكن false، فساعتها مش هقدر أحقق الشرط اللي هو if response بتاعتي بالـstatus هي بتروح ولا false، مش هقدر أحققها دي ليه؟ عشان مفيش discriminated onion.</summary>
user
هنا الـstatus هو يعتبر discriminated عشان خاطر فيه قيمة أو محددة value اللي هي بتروح وfalse، فهنا فيه مميزة. طيب نرجع للجزئية التانية، حتة إن أنا أخلي الـstatus بـBoolean أنا كده مش عامل discriminated onion حقيقية، ليه؟ لأن هنا أنا ما أعرفش هي هتشتغل بيانات، أنا ما أعرفش هي true ولا false، ومرتين عنده Boolean ممكن تبقى true وممكن false، فساعتها مش هقدر أحقق الشرط اللي هو if response بتاعتي بالـstatus هي بتروح ولا false، مش هقدر أحققها دي ليه؟ عشان مفيش discriminated onion.
ChatGPT
إجابتك **ممتازة جدًا، 10/10 من ناحية الفهم**. أنت وصلت للسبب الحقيقي، مش مجرد حفظ المصطلح. ⏱️ 3 دقائق

بس أصحح لك المصطلح:

```text
Discriminated Union
```

مش `discriminated onion` 😄

### اللي فهمته صح

لما نكتب:

```ts
type Result =
  | { status: true; data: string }
  | { status: false; error: string };
```

الـ `status` هنا هو الـ **Discriminant** لأن عندنا قيمتين محددتين:

```text
status: true
      ↓
Success

status: false
      ↓
Error
```

فلو عملنا:

```ts
if (result.status) {
  result.data;
}
```

TypeScript يقدر يستنتج:

```text
status === true
      ↓
النوع هو Success
      ↓
data موجودة
```

ولو:

```ts
if (!result.status) {
  result.error;
}
```

يفهم:

```text
status === false
      ↓
النوع هو Error
      ↓
error موجودة
```

### طيب ليه `boolean` مش كفاية؟

لو كتبنا:

```ts
type Result =
  | { status: boolean; data: string }
  | { status: boolean; error: string };
```

الـ TypeScript شايف:

```text
الأول:
status → boolean

الثاني:
status → boolean
```

يعني الاتنين ممكن يكونوا:

```text
true
```

أو:

```text
false
```

فلو كتبت:

```ts
if (result.status) {
  result.data;
}
```

الـ TypeScript **مش قادر يستنتج إن `result` هو النوع الأول**، لأن النوع الثاني أيضًا ممكن يكون عنده:

```ts
status: true
```

وده جوهر الـ Discriminated Union:

> لازم يكون عندنا **علامة مميزة بقيمة محددة** تسمح لـ TypeScript يعمل `Narrowing`.

```text
boolean ❌
   ↓
true | false

true / false literals ✅
   ↓
Success / Error
```

### 📍 Progress

```text
SECTION 2: TypeScript + Axios

├── AxiosResponse                         ✅
├── Generics مع Axios                     ✅
├── Response Types                        ✅
├── API Response Types                    ✅
├── Discriminated Unions                  ✅
└── unknown vs any                        🟢 ← التالي
```

**الخطوة التالية:** نبدأ `unknown vs any` ونفهم ليه `unknown` هو الاختيار الأكثر أمانًا لما تكون مش عارف نوع البيانات.
</details>
هنا الـstatus هو يعتبر discriminated عشان خاطر فيه قيمة أو محددة value اللي هي بتروح وfalse، فهنا فيه مميزة. طيب نرجع للجزئية التانية، حتة إن أنا أخلي الـstatus بـBoolean أنا كده مش عامل discriminated onion حقيقية، ليه؟ لأن هنا أنا ما أعرفش هي هتشتغل بيانات، أنا ما أعرفش هي true ولا false، ومرتين عنده Boolean ممكن تبقى true وممكن false، فساعتها مش هقدر أحقق الشرط اللي هو if response بتاعتي بالـstatus هي بتروح ولا false، مش هقدر أحققها دي ليه؟ عشان مفيش discriminated onion.
ChatGPT
إجابتك **ممتازة جدًا، 10/10 من ناحية الفهم**. أنت وصلت للسبب الحقيقي، مش مجرد حفظ المصطلح. ⏱️ 3 دقائق

بس أصحح لك المصطلح:

```text
Discriminated Union
```

مش `discriminated onion` 😄

### اللي فهمته صح

لما نكتب:

```ts
type Result =
  | { status: true; data: string }
  | { status: false; error: string };
```

الـ `status` هنا هو الـ **Discriminant** لأن عندنا قيمتين محددتين:

```text
status: true
      ↓
Success

status: false
      ↓
Error
```

فلو عملنا:

```ts
if (result.status) {
  result.data;
}
```

TypeScript يقدر يستنتج:

```text
status === true
      ↓
النوع هو Success
      ↓
data موجودة
```

ولو:

```ts
if (!result.status) {
  result.error;
}
```

يفهم:

```text
status === false
      ↓
النوع هو Error
      ↓
error موجودة
```

### طيب ليه `boolean` مش كفاية؟

لو كتبنا:

```ts
type Result =
  | { status: boolean; data: string }
  | { status: boolean; error: string };
```

الـ TypeScript شايف:

```text
الأول:
status → boolean

الثاني:
status → boolean
```

يعني الاتنين ممكن يكونوا:

```text
true
```

أو:

```text
false
```

فلو كتبت:

```ts
if (result.status) {
  result.data;
}
```

الـ TypeScript **مش قادر يستنتج إن `result` هو النوع الأول**، لأن النوع الثاني أيضًا ممكن يكون عنده:

```ts
status: true
```

وده جوهر الـ Discriminated Union:

> لازم يكون عندنا **علامة مميزة بقيمة محددة** تسمح لـ TypeScript يعمل `Narrowing`.

```text
boolean ❌
   ↓
true | false

true / false literals ✅
   ↓
Success / Error
```

### 📍 Progress

```text
SECTION 2: TypeScript + Axios

├── AxiosResponse                         ✅
├── Generics مع Axios                     ✅
├── Response Types                        ✅
├── API Response Types                    ✅
├── Discriminated Unions                  ✅
└── unknown vs any                        🟢 ← التالي
```

**الخطوة التالية:** نبدأ `unknown vs any` ونفهم ليه `unknown` هو الاختيار الأكثر أمانًا لما تكون مش عارف نوع البيانات.
<details>
<summary>unknown vs any</summary>
user
unknown vs any  
ChatGPT
ابدأ بالفرق بين `unknown` و`any`، والهدف إنك تفهم **ليه `unknown` بيحافظ على Type Safety بينما `any` ممكن يلغيه**. ⏱️ 12 دقيقة

## 📍 Roadmap

```text
SECTION 0: Architecture Foundation       ⬜
SECTION 1: Axios Fundamentals             ✅

SECTION 2: TypeScript + Axios             🟢
├── AxiosResponse                         ✅
├── Generics مع Axios                     ✅
├── Response Types                         ✅
├── API Response Types                     ✅
├── Discriminated Unions                  ✅
├── unknown vs any                        🟢 ← هنا
└── Type-safe API Layer                   ⬜

SECTION 3: Error Handling                 ⬜
SECTION 4: Interceptors                   ⬜
...
```
</details>
unknown vs any  
ChatGPT
ابدأ بالفرق بين `unknown` و`any`، والهدف إنك تفهم **ليه `unknown` بيحافظ على Type Safety بينما `any` ممكن يلغيه**. ⏱️ 12 دقيقة

## 📍 Roadmap

```text
SECTION 0: Architecture Foundation       ⬜
SECTION 1: Axios Fundamentals             ✅

SECTION 2: TypeScript + Axios             🟢
├── AxiosResponse                         ✅
├── Generics مع Axios                     ✅
├── Response Types                         ✅
├── API Response Types                     ✅
├── Discriminated Unions                  ✅
├── unknown vs any                        🟢 ← هنا
└── Type-safe API Layer                   ⬜

SECTION 3: Error Handling                 ⬜
SECTION 4: Interceptors                   ⬜
...
```