# 1. الأول نفهم المشكلة

افترض إن عندك قيمة ومش عارف نوعها:

```ts id="w7x3q2"
let value: unknown;
```

ممكن تكون:

```text
string
number
object
array
boolean
```

إنت مش عارف.

هنا TypeScript بيقولك:

> تمام، مش عارف نوعها، لكن **مش هسمحلك تستخدمها كأنك عارف نوعها**.

وده بالضبط معنى `unknown`.

---

# 2. مثال `unknown`

```ts id="g5n8ka"
let value: unknown = "Ahmed";
```

لو حاولت:

```ts id="m2q7vx"
console.log(value.toUpperCase());
```

TypeScript هيمنعك.

ليه؟

لأن:

```text id="z4p1cw"
value → unknown
```

وممكن تكون:

```text id="3n8bq6"
"Ahmed"     ✅
100         ❌
true        ❌
```

فـ TypeScript مش ضامن إنها String.

لازم تتأكد الأول.

---

# 3. نعمل Type Narrowing

```ts id="a9k4rd"
if (typeof value === "string") {
  console.log(value.toUpperCase());
}
```

هنا TypeScript يقول:

```text id="v6s2mx"
داخل if
value → string
```

وبالتالي:

```ts id="d1w8qp"
value.toUpperCase();
```

أصبحت آمنة.

دي نفس فكرة الـ **Narrowing** اللي شفناها في `Discriminated Unions`.

---

# 4. طيب `any`؟

دلوقتي:

```ts id="r3k7yh"
let value: any = "Ahmed";
```

ممكن تعمل:

```ts id="x8q2mf"
value.toUpperCase();
```

TypeScript مش هيعترض.

بل ممكن تعمل حاجة غلط تمامًا:

```ts id="q4m9va"
value.getUserData();
```

TypeScript برضه غالبًا مش هيعترض.

ليه؟

لأنك قلت له:

> اعتبر القيمة دي أي حاجة، ومتعملش عليها Type Checking.

وده معنى `any`.

---

# 5. الفرق الجوهري

احفظ الجملة دي:

```text id="u6p2ck"
unknown
↓
أنا لا أعرف النوع
↓
إذن لازم أتأكد قبل الاستخدام

any
↓
أنا لا أهتم بالنوع
↓
TypeScript لا يحميني
```

---

# 6. مثال يوضح الفرق جدًا

### `unknown`

```ts id="b7w4nx"
let data: unknown = "Ahmed";

data.toUpperCase(); // ❌
```

لازم:

```ts id="h2c9vp"
if (typeof data === "string") {
  data.toUpperCase(); // ✅
}
```

### `any`

```ts id="n8r3qm"
let data: any = "Ahmed";

data.toUpperCase(); // ✅
data.foo.bar.baz(); // TypeScript مش هيحميك
```

هنا أنت فقدت جزء كبير من فائدة TypeScript.

---

# 7. طب إيه علاقة ده بالـ API؟

دي النقطة اللي تهمنا في الكورس.

لما تستقبل بيانات من API، البيانات القادمة من الخارج **مش المفروض تثق فيها بشكل أعمى**.

مثلاً:

```ts id="k5v9dz"
const data: unknown = await getData();
```

TypeScript بيجبرك تتعامل معها بحذر.

لكن لو كتبت:

```ts id="p2x7mc"
const data: any = await getData();
```

أنت تقريبًا بتقول:

> اعتبر أي حاجة جاية من الـ API صحيحة.

وده خطر من ناحية Type Safety.

---

# 8. لكن خد بالك من نقطة مهمة جدًا

لما نكتب:

```ts id="c8m4qa"
axios.get<IUser>("/users/10");
```

إحنا **مش بنستخدم `unknown`**.

إحنا بنقول لـ TypeScript:

```text id="5r9zkn"
أنا متوقع إن response.data يكون IUser
```

لكن ده لا يعني إن السيرفر **مضمون** إنه رجع `IUser` فعليًا.

وده فرق مهم:

```text id="j6x2vb"
TypeScript Type
      ≠
Runtime Validation
```

الـ Generic:

```ts id="h3q7mp"
axios.get<IUser>()
```

بيساعدك أثناء التطوير.

لكن لو عايز تتحقق فعليًا من البيانات القادمة من API في Runtime، هنحتاج **Runtime Validation**، وده هنوصله بعدين في الـ Type-safe API Architecture.

مش هنقفز له دلوقتي.

---

# 9. مقارنة سريعة

| | `unknown` | `any` |
|---|---|---|
| TypeScript يعرف النوع؟ | لا | لا |
| يسمح بالاستخدام المباشر؟ | ❌ | ✅ |
| يحتاج Narrowing؟ | ✅ | ❌ |
| يحافظ على Type Safety؟ | ✅ | ❌ |
| آمن افتراضيًا؟ | ✅ | ❌ |

### 🧠 الصورة الذهنية

```text id="8q4m1z"
unknown

"أنا مش عارف إيه ده"
        ↓
"اتأكد الأول"
        ↓
استعمله بأمان


any

"أنا مش عارف إيه ده"
        ↓
"اعمل اللي إنت عايزه"
        ↓
TypeScript مش هيتدخل
```

وده سبب إنك لما تكون **مش عارف النوع**، `unknown` غالبًا أفضل من `any`.

### 🎯 Interview Question

عندك:

```ts id="m4v8qx"
const data: unknown = "Ahmed";
```

ليه الكود ده مش مسموح؟

```ts id="r8c2ny"
data.toUpperCase();
```

وكيف تجعل TypeScript يسمح لك باستخدام `toUpperCase()` بشكل آمن؟

**الخطوة التالية:** جاوب بالسطر أو السطرين اللي يعملوا `Type Narrowing` للـ `unknown`.
<details>
<summary>مش مسموح لانو لازم يتاكد من النوع الاول</summary>
user
مش مسموح لانو لازم يتاكد من النوع الاول 
رقم 2
عشان يعمل ادخل عليها قاعده if
if(typeof data=== string)
data.toUpperCase();  
ChatGPT
إجابتك **صحيحة في الفكرة**، وفيه تعديل صغير جدًا في الـ syntax. ⏱️ دقيقتان

أنت قلت:

> لازم يتأكد من النوع الأول.

وده بالضبط سبب استخدام `unknown`.

والـ `Type Narrowing` الصحيح:

```ts id="x8n4pk"
if (typeof data === "string") {
  data.toUpperCase();
}
```

أنت كتبت:

```ts id="m5q7zw"
typeof data === string
```

لازم `"string"` تكون **String literal**، لأنها قيمة:

```ts id="j2c9ra"
typeof data === "string"
```

وليس:

```ts id="9qk3wf"
typeof data === string
```

### ليه الـ `if` حل المشكلة؟

قبل الـ `if`:

```text id="7g4p2m"
data → unknown
```

TypeScript مش عارف نوعها.

بعد:

```ts id="b6x9rv"
typeof data === "string"
```

داخل الـ `if`:

```text id="k2m8qd"
data → string
```

فأصبح:

```ts id="r5v1cx"
data.toUpperCase();
```

مسموح وآمن.

### ⭐ تقييم Interview

**9.5/10**

الفهم ممتاز، والخطأ الوحيد كان كتابة `"string"` بدون quotes.

### 📍 Progress

```text id="e3w7ka"
SECTION 2: TypeScript + Axios

├── AxiosResponse                         ✅
├── Generics مع Axios                     ✅
├── Response Types                        ✅
├── API Response Types                    ✅
├── Discriminated Unions                  ✅
├── unknown vs any                        ✅
└── Type-safe API Layer                   🟢 ← التالي
```

**الخطوة التالية:** نبدأ `Type-safe API Layer` ونبني أول طبقة API بسيطة تفصل Axios عن الـ Components.
</details>
مش مسموح لانو لازم يتاكد من النوع الاول 
رقم 2
عشان يعمل ادخل عليها قاعده if
if(typeof data=== string)
data.toUpperCase();  
ChatGPT
إجابتك **صحيحة في الفكرة**، وفيه تعديل صغير جدًا في الـ syntax. ⏱️ دقيقتان

أنت قلت:

> لازم يتأكد من النوع الأول.

وده بالضبط سبب استخدام `unknown`.

والـ `Type Narrowing` الصحيح:

```ts id="x8n4pk"
if (typeof data === "string") {
  data.toUpperCase();
}
```

أنت كتبت:

```ts id="m5q7zw"
typeof data === string
```

لازم `"string"` تكون **String literal**، لأنها قيمة:

```ts id="j2c9ra"
typeof data === "string"
```

وليس:

```ts id="9qk3wf"
typeof data === string
```

### ليه الـ `if` حل المشكلة؟

قبل الـ `if`:

```text id="7g4p2m"
data → unknown
```

TypeScript مش عارف نوعها.

بعد:

```ts id="b6x9rv"
typeof data === "string"
```

داخل الـ `if`:

```text id="k2m8qd"
data → string
```

فأصبح:

```ts id="r5v1cx"
data.toUpperCase();
```

مسموح وآمن.

### ⭐ تقييم Interview

**9.5/10**

الفهم ممتاز، والخطأ الوحيد كان كتابة `"string"` بدون quotes.

### 📍 Progress

```text id="e3w7ka"
SECTION 2: TypeScript + Axios

├── AxiosResponse                         ✅
├── Generics مع Axios                     ✅
├── Response Types                        ✅
├── API Response Types                    ✅
├── Discriminated Unions                  ✅
├── unknown vs any                        ✅
└── Type-safe API Layer                   🟢 ← التالي
```

**الخطوة التالية:** نبدأ `Type-safe API Layer` ونبني أول طبقة API بسيطة تفصل Axios عن الـ Components.
<details>
<summary>Type-safe API Layer</summary>
user
Type-safe API Layer  
ChatGPT
ابدأ ببناء `Type-safe API Layer` بسيط، والهدف إنك تفهم **ليه بنفصل استدعاء الـ API عن الـ Component** وإزاي `TypeScript` يضمن شكل البيانات في كل طبقة. ⏱️ 20 دقيقة

## 📍 Roadmap

```text
SECTION 0: Architecture Foundation       ⬜
SECTION 1: Axios Fundamentals             ✅

SECTION 2: TypeScript + Axios             🟢
├── AxiosResponse                         ✅
├── Generics مع Axios                     ✅
├── Response Types                        ✅
├── API Response Types                    ✅
├── Discriminated Unions                  ✅
├── unknown vs any                        ✅
└── Type-safe API Layer                   🟢 ← هنا

SECTION 3: Error Handling                 ⬜
SECTION 4: Interceptors                   ⬜
SECTION 5: API Architecture               ⬜
...
```
</details>
Type-safe API Layer  
ChatGPT
ابدأ ببناء `Type-safe API Layer` بسيط، والهدف إنك تفهم **ليه بنفصل استدعاء الـ API عن الـ Component** وإزاي `TypeScript` يضمن شكل البيانات في كل طبقة. ⏱️ 20 دقيقة

## 📍 Roadmap

```text
SECTION 0: Architecture Foundation       ⬜
SECTION 1: Axios Fundamentals             ✅

SECTION 2: TypeScript + Axios             🟢
├── AxiosResponse                         ✅
├── Generics مع Axios                     ✅
├── Response Types                        ✅
├── API Response Types                    ✅
├── Discriminated Unions                  ✅
├── unknown vs any                        ✅
└── Type-safe API Layer                   🟢 ← هنا

SECTION 3: Error Handling                 ⬜
SECTION 4: Interceptors                   ⬜
SECTION 5: API Architecture               ⬜
...
```