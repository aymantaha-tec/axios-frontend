# 🧠 Section 3 Final Review

هنراجع **الفهم**، مش الحفظ.

### السؤال 1

عندك:

```ts
try {
  const user = await getUser(10);
} catch (error) {
  // ...
}
```

لو الـ API رجع:

```text
404 Not Found
```

جاوب:

1. هل ده `HTTP Error` ولا `Network Error`؟
2. هل `error.response` موجودة؟
3. فين نقدر نلاقي الـ HTTP status؟
4. فين نلاقي رسالة الـ API لو السيرفر رجع:

```json
{
  "status": false,
  "code": 1001,
  "message": "User not found"
}
```

---

### السؤال 2

إيه الفرق بين:

```ts
error.message
```

و:

```ts
error.response?.data.message
```

---

### السؤال 3

إيه الفرق بين:

```ts
error.code
```

و:

```ts
error.response?.data.code
```

---

### السؤال 4

لو حصل:

```text
Network Error
```

ليه:

```ts
error.response?.status
```

مش هتدينا HTTP status؟

---

### السؤال 5

إيه الهدف من:

```ts
axios.isAxiosError(error)
```

وليه بدأنا بـ:

```ts
error: unknown
```

بدل:

```ts
error: any
```

---

### السؤال 6

بكلامك أنت، إيه معنى:

```text
Error Normalization
```

وماذا المشكلة التي يحلها؟

---

### السؤال 7

عندنا:

```text
AxiosError
     ↓
normalizeError()
     ↓
INormalizedError
```

ليه ما نرميش `AxiosError` مباشرة للـ Components؟

---

### السؤال 8

عندنا:

```ts
interface INormalizedError {
  message: string;
  status?: number;
  code?: string;
}
```

لو عندنا `404` والـ API رجع:

```json
{
  "status": false,
  "code": 1001,
  "message": "User not found"
}
```

إيه الشكل المتوقع للـ normalized error؟

---

## 🎯 التقييم

هقيّمك على:

```text
HTTP vs Network          /10
AxiosError               /10
message & code           /10
unknown & narrowing      /10
Normalization            /10
Architecture             /10
```

وبعدها أقول لك بصراحة:

**جاهز لـ Section 0 أم محتاج تثبيت نقطة معينة.**

**الخطوة التالية:** جاوب الأسئلة الـ8 بالترتيب، حتى لو إجابات قصيرة.
<details>
<summary>١-١</summary>
user
١-١
HTTP Error
١-2
موجود 
1-3
error.response?.stauts
1-⁴
error.response?.data.message
2
error.message=== axios error
error.response?.data.message===api error server
3
نفس اجابه السؤال الثاني بس مع التبديل ال 
message اليcode
4
لانه لم تصل لي السيرفر اي لايوجد response
5
عشان التأكد أن الخطأ تبع axioserror

unknown and any 
الاولي أمن وافضل لأنها تائب اسكربت لاتعرف النوع لا لما تتأكد وده عن طريق قاعده if typeof === "string"
اما التانيه هتخلي تائب اسكربت تتعامل مع أي نوع وتشهد اي خصائص أو ميثود حتي لو خطأ
٦
توحيد الخطا منفصله عن المكونات 
عشان لو عندنا مثلا اكتر من كمبوناتت الموضوع هيبقي صعب وكود كتير هيتكرر
٧
تم الاجابه في فوق
٨ApiError
9



ChatGPT
### نتيجة مراجعة Section 3

**النتيجة: 9.5/10 🔥**

أنت فاهم الـconcepts فعلاً، والأخطاء الموجودة أغلبها **syntax / naming** وليست مشكلة فهم.

| النقطة | التقييم |
|---|---:|
| HTTP Error vs Network Error | ✅ 10/10 |
| `error.response` | ✅ 10/10 |
| `error.message` vs API message | ✅ 10/10 |
| Axios `error.code` vs API `code` | ✅ 9.5/10 |
| Network Error | ✅ 10/10 |
| `axios.isAxiosError()` | ✅ 9.5/10 |
| `unknown` vs `any` | ✅ 10/10 |
| Error Normalization | ✅ 10/10 |
| Custom `ApiError` | 🟡 مفهومه موجود، لكن محتاج تطبيق |

### تصحيح صغير مهم

في السؤال 3 قلت:

> نفس إجابة السؤال الثاني مع تبديل message إلى code

المعنى صحيح، لكن لازم تفرق بين الطبقتين:

```ts
error.code
```

ده **Axios code**.

مثلاً:

```ts
"ERR_NETWORK"
```

بينما:

```ts
error.response?.data.code
```

ده **Backend/API code**.

مثلاً:

```ts
1001
```

يعني عندنا:

```text
HTTP Status
    ↓
404

Axios Error Code
    ↓
ERR_BAD_REQUEST

API Code
    ↓
1001
```

كل واحد له معنى مختلف.

---

### وبالنسبة لـ Error Normalization

إجابتك ممتازة:

> توحيد الخطأ منفصلة عن المكونات

بالضبط.

بدل ما كل Component يعرف:

```ts
AxiosError
Network Error
Timeout
404
500
API message
```

نعمل:

```text
Axios / Network / Unknown
          ↓
   normalizeError()
          ↓
 INormalizedError
          ↓
      Component
```

وده يقلل **duplication** ويقلل **coupling** بين الـUI و Axios.

---

### ملاحظة على إجابة 8

أنت كتبت:

> ApiError

لو السؤال كان: **ما هو الـapplication-level error الذي ممكن نستخدمه بعد الـnormalization؟**

فالإجابة صحيحة:

```ts
ApiError
```

لكن لو السؤال عن الـnormalized object نفسه، فمثلاً:

```ts
{
  message: "User not found",
  status: 404
}
```

---

## 🗺️ Roadmap

```text
SECTION 0 Architecture Foundation ⬜
SECTION 1 Axios Fundamentals    ✅
SECTION 2 TypeScript + Axios    ✅
SECTION 3 Error Handling        ✅ 100%
SECTION 4 Interceptors          ⬜
SECTION 5 API Architecture      ⬜
SECTION 6 TanStack Query        ⬜
SECTION 7 Authentication        ⬜
SECTION 8 Forms                ⬜
SECTION 9 Tailwind + shadcn/ui ⬜
SECTION 10 Storybook           ⬜
SECTION 11 Testing             ⬜
SECTION 12 Production          ⬜
SECTION 13 Final Project       ⬜
```

**Section 3 اتقفلت.**  
والخطوة التالية في المنهج هي **Section 0: Architecture Foundation**، وليس Interceptors، لأننا اتفقنا إن الـArchitecture Foundation لازم تتاخد قبل الدخول في الـInterceptors والـAPI Architecture.

**الخطوة التالية: اكتب `ابدأ Section 0` لنبدأ أول درس في الـArchitecture Foundation.**
</details>
١-١
HTTP Error
١-2
موجود 
1-3
error.response?.stauts
1-⁴
error.response?.data.message
2
error.message=== axios error
error.response?.data.message===api error server
3
نفس اجابه السؤال الثاني بس مع التبديل ال 
message اليcode
4
لانه لم تصل لي السيرفر اي لايوجد response
5
عشان التأكد أن الخطأ تبع axioserror

unknown and any 
الاولي أمن وافضل لأنها تائب اسكربت لاتعرف النوع لا لما تتأكد وده عن طريق قاعده if typeof === "string"
اما التانيه هتخلي تائب اسكربت تتعامل مع أي نوع وتشهد اي خصائص أو ميثود حتي لو خطأ
٦
توحيد الخطا منفصله عن المكونات 
عشان لو عندنا مثلا اكتر من كمبوناتت الموضوع هيبقي صعب وكود كتير هيتكرر
٧
تم الاجابه في فوق
٨ApiError
9



ChatGPT
### نتيجة مراجعة Section 3

**النتيجة: 9.5/10 🔥**

أنت فاهم الـconcepts فعلاً، والأخطاء الموجودة أغلبها **syntax / naming** وليست مشكلة فهم.

| النقطة | التقييم |
|---|---:|
| HTTP Error vs Network Error | ✅ 10/10 |
| `error.response` | ✅ 10/10 |
| `error.message` vs API message | ✅ 10/10 |
| Axios `error.code` vs API `code` | ✅ 9.5/10 |
| Network Error | ✅ 10/10 |
| `axios.isAxiosError()` | ✅ 9.5/10 |
| `unknown` vs `any` | ✅ 10/10 |
| Error Normalization | ✅ 10/10 |
| Custom `ApiError` | 🟡 مفهومه موجود، لكن محتاج تطبيق |

### تصحيح صغير مهم

في السؤال 3 قلت:

> نفس إجابة السؤال الثاني مع تبديل message إلى code

المعنى صحيح، لكن لازم تفرق بين الطبقتين:

```ts
error.code
```

ده **Axios code**.

مثلاً:

```ts
"ERR_NETWORK"
```

بينما:

```ts
error.response?.data.code
```

ده **Backend/API code**.

مثلاً:

```ts
1001
```

يعني عندنا:

```text
HTTP Status
    ↓
404

Axios Error Code
    ↓
ERR_BAD_REQUEST

API Code
    ↓
1001
```

كل واحد له معنى مختلف.

---

### وبالنسبة لـ Error Normalization

إجابتك ممتازة:

> توحيد الخطأ منفصلة عن المكونات

بالضبط.

بدل ما كل Component يعرف:

```ts
AxiosError
Network Error
Timeout
404
500
API message
```

نعمل:

```text
Axios / Network / Unknown
          ↓
   normalizeError()
          ↓
 INormalizedError
          ↓
      Component
```

وده يقلل **duplication** ويقلل **coupling** بين الـUI و Axios.

---

### ملاحظة على إجابة 8

أنت كتبت:

> ApiError

لو السؤال كان: **ما هو الـapplication-level error الذي ممكن نستخدمه بعد الـnormalization؟**

فالإجابة صحيحة:

```ts
ApiError
```

لكن لو السؤال عن الـnormalized object نفسه، فمثلاً:

```ts
{
  message: "User not found",
  status: 404
}
```

---

## 🗺️ Roadmap

```text
SECTION 0 Architecture Foundation ⬜
SECTION 1 Axios Fundamentals    ✅
SECTION 2 TypeScript + Axios    ✅
SECTION 3 Error Handling        ✅ 100%
SECTION 4 Interceptors          ⬜
SECTION 5 API Architecture      ⬜
SECTION 6 TanStack Query        ⬜
SECTION 7 Authentication        ⬜
SECTION 8 Forms                ⬜
SECTION 9 Tailwind + shadcn/ui ⬜
SECTION 10 Storybook           ⬜
SECTION 11 Testing             ⬜
SECTION 12 Production          ⬜
SECTION 13 Final Project       ⬜
```

**Section 3 اتقفلت.**  
والخطوة التالية في المنهج هي **Section 0: Architecture Foundation**، وليس Interceptors، لأننا اتفقنا إن الـArchitecture Foundation لازم تتاخد قبل الدخول في الـInterceptors والـAPI Architecture.

**الخطوة التالية: اكتب `ابدأ Section 0` لنبدأ أول درس في الـArchitecture Foundation.**
<details>
<summary>Section 0</summary>
user
Section 0
ChatGPT
**ابدأ الآن بالدرس 1 من Section 0، ومدته حوالي 20 دقيقة: HTTP Architecture قبل ما نبني Axios Architecture.**

## 🗺️ Roadmap

```text
SECTION 0: Architecture Foundation 🟢
  ├─ 0.1 HTTP Architecture        🟢 ← نحن هنا
  ├─ 0.2 Separation of Concerns    ⬜
  ├─ 0.3 Coupling & Cohesion       ⬜
  ├─ 0.4 Feature-Based Architecture⬜
  ├─ 0.5 Shared vs Feature        ⬜
  ├─ 0.6 Public vs Internal API    ⬜
  └─ 0.7 Dependency Direction      ⬜

SECTION 1 Axios Fundamentals       ✅
SECTION 2 TypeScript + Axios       ✅
SECTION 3 Error Handling           ✅
SECTION 4 Interceptors             ⬜
SECTION 5 API Architecture         ⬜
SECTION 6 TanStack Query            ⬜
...
```
</details>
Section 0
ChatGPT
**ابدأ الآن بالدرس 1 من Section 0، ومدته حوالي 20 دقيقة: HTTP Architecture قبل ما نبني Axios Architecture.**

## 🗺️ Roadmap

```text
SECTION 0: Architecture Foundation 🟢
  ├─ 0.1 HTTP Architecture        🟢 ← نحن هنا
  ├─ 0.2 Separation of Concerns    ⬜
  ├─ 0.3 Coupling & Cohesion       ⬜
  ├─ 0.4 Feature-Based Architecture⬜
  ├─ 0.5 Shared vs Feature        ⬜
  ├─ 0.6 Public vs Internal API    ⬜
  └─ 0.7 Dependency Direction      ⬜

SECTION 1 Axios Fundamentals       ✅
SECTION 2 TypeScript + Axios       ✅
SECTION 3 Error Handling           ✅
SECTION 4 Interceptors             ⬜
SECTION 5 API Architecture         ⬜
SECTION 6 TanStack Query            ⬜
...
```