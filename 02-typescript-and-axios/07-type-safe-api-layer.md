# 1. المشكلة الأول

تخيل عندك Component:

```ts id="4q7m2a"
function UsersPage() {
  const getUsers = async () => {
    const response = await axios.get<IUser[]>("/users");

    return response.data;
  };

  // باقي الـ Component
}
```

الكود ده شغال.

لكن فيه مشكلة معمارية:

الـ Component أصبح **عارف تفاصيل الـ API**.

يعني الـ Component يعرف:

```text
Axios
Endpoint
HTTP Method
Response Type
```

وده مش أفضل تصميم.

---

# 2. نفصل الـ API في Layer

بدل ما نكتب:

```ts id="w8k3pz"
axios.get<IUser[]>("/users");
```

داخل الـ Component، نعمل Function مسؤولة عن Users API:

```ts id="n5r9vc"
export const getUsers = async (): Promise<IUser[]> => {
  const response = await axios.get<IUser[]>("/users");

  return response.data;
};
```

دلوقتي الـ Component يعمل:

```ts id="h2q6wd"
const users = await getUsers();
```

لاحظ الفرق:

```text
قبل

Component
   ↓
Axios
   ↓
API


بعد

Component
   ↓
getUsers()
   ↓
Axios
   ↓
API
```

الـ Component لم يعد محتاج يعرف تفاصيل Axios.

---

# 3. ليه ده اسمه Type-safe؟

بص على:

```ts id="c4m7xa"
export const getUsers = async (): Promise<IUser[]> => {
```

إحنا بنقول:

> الـ Function دي لازم ترجع `Promise` يحتوي على `IUser[]`.

وبعدين:

```ts id="z7p3kf"
const response = await axios.get<IUser[]>("/users");
```

الـ Axios نفسه عارف إن:

```text
response.data → IUser[]
```

ثم:

```ts id="v8n2qm"
return response.data;
```

إذن:

```text
Axios
 ↓
IUser[]
 ↓
getUsers()
 ↓
Promise<IUser[]>
```

كل حاجة Typed.

---

# 4. طب لو API Wrapper موجود؟

وده الأقرب للـ API اللي كنا بنتكلم عنه.

لو السيرفر يرجع:

```json id="m8q2rx"
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

عندنا:

```ts id="y6w4kp"
interface IUser {
  id: number;
  name: string;
}
```

و:

```ts id="q9c5nv"
interface ISuccessResponse<T> {
  status: true;
  code: number;
  payload: T;
}
```

نعمل API Function:

```ts id="a3f7md"
export const getUsers = async (): Promise<IUser[]> => {
  const response = await axios.get<
    ISuccessResponse<IUser[]>
  >("/users");

  return response.data.payload;
};
```

هنا عندنا:

```text
Axios Response
      ↓
AxiosResponse<ISuccessResponse<IUser[]>>
      ↓
response.data
      ↓
ISuccessResponse<IUser[]>
      ↓
response.data.payload
      ↓
IUser[]
```

وفي النهاية الـ Component يستلم فقط:

```text
IUser[]
```

---

# 5. والـ Component؟

بقى بسيط جدًا:

```ts id="v5k8cx"
const users = await getUsers();
```

هو مش محتاج يعرف:

```text
Axios
GET
/users
IApiResponse
payload
```

هو فقط يعرف:

```text
getUsers()
   ↓
Promise<IUser[]>
```

وده مهم جدًا في الـ Architecture.

---