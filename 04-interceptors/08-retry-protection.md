## نقطة مهمة جدًا

`_retry` مش معناه:

> الطلب نجح.

هو معناه:

> **إحنا حاولنا بالفعل نعمل Token Refresh لهذا الطلب.**

وده فرق مهم.

---

**الخطوة التالية:** هنعمل السيناريو الأصعب والأهم: **لو 10 Requests حصل لهم `401` في نفس اللحظة، إزاي نخلي Request واحد فقط يعمل Refresh والباقي يستنى الـ New Token باستخدام `Queue`؟**
<details>
<summary>كمل</summary>
user
كمل
ChatGPT
نكمل **Queue في حوالي 20 دقيقة**، ودي أهم نقطة في `Token Refresh` لأنها بتحل مشكلة تعدد الـ Requests في نفس اللحظة.

### 🗺️ Roadmap

```text id="q2k7p1"
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
     └─ Queue                     🟡 ← هنا
```
</details>
كمل
ChatGPT
نكمل **Queue في حوالي 20 دقيقة**، ودي أهم نقطة في `Token Refresh` لأنها بتحل مشكلة تعدد الـ Requests في نفس اللحظة.

### 🗺️ Roadmap

```text id="q2k7p1"
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
     └─ Queue                     🟡 ← هنا
```