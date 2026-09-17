## 1. المشكلة اللي الـ Queue بتحلها

تخيل إن عندنا 3 Requests في نفس اللحظة:

```text
Request A → 401
Request B → 401
Request C → 401
```

والـ `Access Token` انتهى.

لو كل واحد عمل Refresh:

```text
A → 401 → Refresh
B → 401 → Refresh
C → 401 → Refresh
```

يبقى عملنا **3 Refresh Requests**.

وده مش مطلوب.

إحنا عايزين:

```text
A → 401
B → 401
C → 401
       ↓
   Refresh واحد
       ↓
  New Access Token
       ↓
 ┌─────┼─────┐
 ↓     ↓     ↓
 A     B     C
Retry Retry Retry
```

---

# 2. يعني إيه Queue؟

`Queue` ببساطة معناها:

> **ناس وصلت، لكن مستنية دورها.**

مثال عادي:

```text
Checkout
   ↓
Customer A
Customer B
Customer C
   ↓
A يدخل الأول
B ينتظر
C ينتظر
```

نفس الفكرة هنا.

```text
Refresh
   ↓
Request A → ينتظر
Request B → ينتظر
Request C → ينتظر
```

لحد ما الـ Refresh يخلص.

---

# 3. أهم Variable

نحتاج نعرف:

> هل فيه Refresh شغال حاليًا؟

ممكن نستخدم:

```ts id="j4yq2p"
let isRefreshing = false;
```

في البداية:

```text
isRefreshing = false
```

يعني:

```text
مفيش Refresh شغال
```

---

# 4. أول Request يحصل له 401

مثلاً:

```text id="5k7c1z"
Request A
   ↓
401
```

نشوف:

```ts id="n3r8x2"
if (!isRefreshing) {
  isRefreshing = true;
}
```

فتصبح:

```text
isRefreshing = true
```

وبالتالي Request A هو اللي يبدأ الـ Refresh.

---

# 5. Request B يحصل له 401

في نفس الوقت:

```text id="m4k9q1"
Request B
   ↓
401
```

يلاقي:

```ts id="h5v2n8"
isRefreshing === true
```

يعني:

> فيه حد بالفعل بيعمل Refresh.

إذن **B لا يعمل Refresh جديد**.

يستنى.

---

# 6. Request C نفس الكلام

```text id="p7s4d6"
Request C
   ↓
401
   ↓
isRefreshing === true
   ↓
استنى
```

دلوقتي:

```text id="r6x1w9"
A → يعمل Refresh
B → ينتظر
C → ينتظر
```

---

# 7. الـ Refresh يخلص

Backend يرجع:

```json id="v2n5m8"
{
  "accessToken": "NEW_TOKEN"
}
```

نخزن:

```ts id="k8w3r1"
localStorage.setItem(
  "accessToken",
  newAccessToken
);
```

وبعدين:

```ts id="j9p4t6"
isRefreshing = false;
```

---

# 8. دلوقتي B و C يكملوا

كانوا مستنيين:

```text id="u3c7m2"
A ───────────→ Refresh ─────→ Retry
B ───────────→ Wait ─────────→ Retry
C ───────────→ Wait ─────────→ Retry
```

والـ Requests الثلاثة يستخدموا:

```text
NEW_TOKEN
```

فتبقى:

```text id="f8n2q5"
A → NEW_TOKEN → 200 ✅
B → NEW_TOKEN → 200 ✅
C → NEW_TOKEN → 200 ✅
```

---

# 9. طيب إزاي نخليهم "يستَنوا" فعلًا؟

هنا بنستخدم مفهوم اسمه:

**Promise Queue**

الفكرة إننا نحط الـ Requests اللي وصلت أثناء الـ Refresh في قائمة انتظار.

ممكن الشكل المفاهيمي يكون:

```ts id="z5m8q2"
let isRefreshing = false;

let failedQueue = [];
```

وتخيل:

```text id="w4k7n1"
failedQueue

[
  Request B,
  Request C,
  Request D
]
```

لما الـ Refresh يخلص:

```text id="e3p6r9"
New Token
   ↓
failedQueue
   ↓
B
C
D
   ↓
Retry
```

---

# 10. لكن فيه تفصيلة مهمة

إحنا **مش بنخزن الـ Request نفسه بشكل عشوائي**.

عادةً بنخزن `Promise` أو `resolve/reject` الخاصة بالـ Requests المنتظرة.

الفكرة:

```text id="c9v2x6"
Request B
   ↓
"استنى لحد ما الـ Refresh يخلص"

Request C
   ↓
"استنى لحد ما الـ Refresh يخلص"
```

ولما الـ Refresh ينجح:

```text id="n7k3p5"
resolve(newAccessToken)
```

فيكملوا.

ولو الـ Refresh فشل:

```text id="a4m8r2"
reject(error)
```

فيتعاملوا مع الفشل.

---

# 11. الشكل الكامل ذهنيًا

احفظ الرسم ده:

```text id="q1w8e4"
             Access Token Expired
                     ↓
          ┌──────────┴──────────┐
          ↓          ↓          ↓
       Req A       Req B       Req C
          ↓          ↓          ↓
         401        401        401
          ↓          ↓          ↓
          └──────┬───┴──────────┘
                 ↓
          هل Refresh شغال؟
                 ↓
          ┌──────┴──────┐
          ↓             ↓
         No            Yes
          ↓             ↓
    Start Refresh     Queue
          ↓             ↓
      New Token        Wait
          ↓             ↓
          └──────┬──────┘
                 ↓
            Retry All
                 ↓
             200 ✅
```

## ⚠️ ملاحظة مهمة

الـ Queue مش معناها إننا بنعمل:

```text
Request A
   ↓
Refresh
   ↓
Request B
   ↓
Request C
```

واحد وراء التاني.

الأفضل:

```text
A → 401
B → 401
C → 401

       ↓

Refresh واحد

       ↓

A → Retry
B → Retry
C → Retry
```

يعني الـ Queue هدفها **تنسيق الـ concurrent requests** ومنع تنفيذ Refresh متعدد في نفس الوقت.

---

### 🎯 Interview Question

عندك:

```text
Request A → 401
Request B → 401
Request C → 401
```

والـ `Access Token` منتهي.

**ليه مينفعش كل Request يعمل `Refresh Token` لوحده؟ وإيه وظيفة الـ Queue هنا؟**

جاوب كأنك في Interview، وبعدها هقيّم إجابتك.
<details>
<summary>والله باشا ما ينفعش أعمل في كل request refresh token يعني دلوقتي هطلق 15 request، هل الـ15 request دول هيجيبولي ربنا أربعة؟ ساعتها كلهم هيطلبوا في نفس الوقت إيه refresh token، فدي كده هتبقى مشكلة. لكن أنا في الحتة دي بعمل حاجة اسمها queue، queue طويل، واللي مستني إنتوا عشان في حد بيعمل إيه refreshing. عن طريق أعمل constant request refresh وخليها بفلوس، ولكن في حالة إن حد بيعمل refresh فوا request تعملها بتروح، ولحد ما يخلصها بيرجع فلوس ساعتها كل واحد يشتغل تاني. بس بقى بعد الكلام ده كله، أنا الكلام ده كله كده نظري، أنا عايز بقى أشوف حاجة إيه حاجة كودينج.</summary>
user
والله باشا ما ينفعش أعمل في كل request refresh token يعني دلوقتي هطلق 15 request، هل الـ15 request دول هيجيبولي ربنا أربعة؟ ساعتها كلهم هيطلبوا في نفس الوقت إيه refresh token، فدي كده هتبقى مشكلة. لكن أنا في الحتة دي بعمل حاجة اسمها queue، queue طويل، واللي مستني إنتوا عشان في حد بيعمل إيه refreshing. عن طريق أعمل constant request refresh وخليها بفلوس، ولكن في حالة إن حد بيعمل refresh فوا request تعملها بتروح، ولحد ما يخلصها بيرجع فلوس ساعتها كل واحد يشتغل تاني. بس بقى بعد الكلام ده كله، أنا الكلام ده كله كده نظري، أنا عايز بقى أشوف حاجة إيه حاجة كودينج.
ChatGPT
أيوه يا باشا، **فهمك للـ Queue ممتاز**، وخلينا دلوقتي ننزل من النظري للكود مباشرة. هنكتب نسخة تعليمية بسيطة الأول، وبعدها نضيف الـ production details.

### 🗺️ Roadmap

```text
SECTION 4: Interceptors 🟢
├─ 4.1 What is Interceptor       ✅
├─ 4.2 Request Interceptor       ✅
├─ 4.3 Response Interceptor      ✅
├─ 4.4 Authentication            ✅
├─ 4.5 401 Handling              ✅
└─ 4.6 Token Refresh + Queue     🟡
     ├─ Token Refresh             ✅
     ├─ error.config              ✅
     ├─ Retry                     ✅
     ├─ Infinite Loop Protection   ✅
     └─ Queue                     🟡 ← هنكود هنا
```
</details>
والله باشا ما ينفعش أعمل في كل request refresh token يعني دلوقتي هطلق 15 request، هل الـ15 request دول هيجيبولي ربنا أربعة؟ ساعتها كلهم هيطلبوا في نفس الوقت إيه refresh token، فدي كده هتبقى مشكلة. لكن أنا في الحتة دي بعمل حاجة اسمها queue، queue طويل، واللي مستني إنتوا عشان في حد بيعمل إيه refreshing. عن طريق أعمل constant request refresh وخليها بفلوس، ولكن في حالة إن حد بيعمل refresh فوا request تعملها بتروح، ولحد ما يخلصها بيرجع فلوس ساعتها كل واحد يشتغل تاني. بس بقى بعد الكلام ده كله، أنا الكلام ده كله كده نظري، أنا عايز بقى أشوف حاجة إيه حاجة كودينج.
ChatGPT
أيوه يا باشا، **فهمك للـ Queue ممتاز**، وخلينا دلوقتي ننزل من النظري للكود مباشرة. هنكتب نسخة تعليمية بسيطة الأول، وبعدها نضيف الـ production details.

### 🗺️ Roadmap

```text
SECTION 4: Interceptors 🟢
├─ 4.1 What is Interceptor       ✅
├─ 4.2 Request Interceptor       ✅
├─ 4.3 Response Interceptor      ✅
├─ 4.4 Authentication            ✅
├─ 4.5 401 Handling              ✅
└─ 4.6 Token Refresh + Queue     🟡
     ├─ Token Refresh             ✅
     ├─ error.config              ✅
     ├─ Retry                     ✅
     ├─ Infinite Loop Protection   ✅
     └─ Queue                     🟡 ← هنكود هنا
```