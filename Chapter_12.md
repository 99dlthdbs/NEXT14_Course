이전 장에서는 `URL Search Params`와 Next.js API를 사용해 검색과 페이지네이션 기능을 구현했어요. <br/>
이번에는 인보이스 페이지에서 인보이스를 생성하고, 수정하고, 삭제할 수 있는 기능을 추가해볼 거예요!

## 이번 장에서는...

- React 서버 액션이 무엇이며, 데이터를 수정(변경)하는 방법

- 폼과 서버 컴포넌트 작업하는 방법

- formData 객체를 사용한 최적의 작업 방법과 타입 검증

- revalidatePath API를 사용해 클라이언트 캐시를 재검증하는 방법

- 특정 ID를 가진 dynamic route segments 생성 방법

<hr />

## Server Action 이란?

`React Server Actions`는 비동기 코드를 서버에서 직접 실행할 수 있게 해주는 기능이에요. <br/>
보통 데이터를 변경할 때 API 엔드포인트를 만들어서 클라이언트에서 서버로 요청을 보내야 했죠. <br/>
하지만 `Server Actions`를 사용하면 API 엔드포인트를 따로 만들 필요 없이, 서버에서 실행되는 비동기 함수를 작성해 클라이언트나 서버 컴포넌트에서 바로 호출할 수 있어요.

이 기능이 중요한 이유는 웹 애플리케이션의 **보안**을 강화해 준다는 점이에요. <br />
웹은 여러 가지 보안 위협에 취약할 수 있거든요. <br/>
`Server Actions`는 **POST 요청, 암호화된 클로저, 엄격한 입력 검증, 에러 메시지 해싱, 호스트 제한** 같은 방법을 사용해 데이터를 보호하고, 허가된 사용자만 접근할 수 있도록
도와줘요. <br />
이 과정들이 합쳐져서 애플리케이션의 보안을 크게 향상시켜 준답니다.

쉽게 말해, `React Server Actions`는 서버에서 실행되는 비동기 작업을 더 안전하고 간편하게 처리할 수 있는 방법이에요!

<hr />

## `Server Action`을 통해 `form` 사용하기

`React`에서는 `<form>` 요소의 `action` 속성을 사용하여 서버로 데이터를 전송할 수 있어요. <br/>
이때 `action` 속성에 지정된 서버 함수는 자동으로 네이티브 [`FormData`](https://developer.mozilla.org/en-US/docs/Web/API/FormData) 객체를
받아요. <br />
이 객체에는 사용자가 폼에 입력한 모든 데이터가 담겨 있어요.

쉽게 말해서, 사용자가 입력한 데이터를 자동으로 모아서 서버로 보내주는 방식이라고 생각하면 돼요. <br />
이렇게 하면 데이터를 쉽게 처리할 수 있고, 서버로 안전하게 전달할 수 있답니다.

```javascript
// Server Component
export default function Page() {
    // Action
    async function create(formData: FormData) {
        'use server';

        // Logic to mutate data...
    }

    // Invoke the action using the "action" attribute
    return <form action={create}>...</form>;
}
```

서버 컴포넌트에서 서버 액션을 호출하는 이점 중 하나는 **점진적 향상(progressive enhancement)** 이에요. <br/>
이 말은 자바스크립트가 클라이언트에서 비활성화되더라도 폼이 정상적으로 작동한다는 뜻이에요.

즉, 사용자의 브라우저에서 자바스크립트가 꺼져 있어도 폼이 데이터를 서버로 안전하게 전송할 수 있어서, 더 많은 사용자에게 안정적으로 기능을 제공할 수 있다는 거예요.

<hr />

## Next.js with `Server Actions`

서버 액션은 Next.js의 [캐싱](https://nextjs.org/docs/app/building-your-application/caching) 시스템과 깊이 통합되어 있어요. <br/>
폼이 서버 액션을 통해 제출되면, 데이터를 변경할 뿐만 아니라 **revalidatePath**나 **revalidateTag**와 같은 API를 사용해서 관련된 캐시를 다시 검증할 수 있어요.

예를 들어, 사용자가 폼을 통해 데이터를 입력하면 서버에서는 그 데이터를 처리하고, 동시에 이 데이터를 기반으로 업데이트된 내용이 화면에 즉시 반영될 수 있도록 캐시를 다시 검증하는 거예요. <br/>
이를 통해 최신 데이터를 유지하면서도 성능을 최적화할 수 있어요.

이제 어떻게 작동하는지 살펴볼까요? 😉

<hr />

## Invoice 만들기

여기서는 새 인보이스를 만들 때 거쳐야 할 단계들을 설명할게요:

1. 사용자의 입력을 받을 폼을 만들어요.
2. 폼에서 서버 액션을 호출해요.
3. 서버 액션 안에서 `formData` 객체로부터 데이터를 추출해요.
4. 데이터를 검증하고 데이터베이스에 저장할 준비를 해요.
5. 데이터를 삽입하고, 오류가 생길 경우 처리해요.
6. 캐시를 다시 검증하고, 사용자에게 인보이스 페이지로 돌아가게 해줘요.

### 1. 새로운 route 와 form 만들기

먼저, `/invoices` 폴더 안에 새로운 경로 세그먼트인 `/create`를 만들고, 그 안에 `page.tsx` 파일을 추가하세요.

<img src="https://nextjs.org/_next/image?url=%2Flearn%2Fdark%2Fcreate-invoice-route.png&w=3840&q=75&dpl=dpl_4kWRRdpV5mEWMp9Zhahs8vP5fgBq" alt="create new segment" />

이 경로는 새 인보이스를 생성하는 데 사용될 거예요. <br />
`page.tsx` 파일 안에 아래 코드를 붙여 넣고, 시간을 들여 코드를 잘 살펴보세요.

```javascript
import Form from '@/app/ui/invoices/create-form';
import Breadcrumbs from '@/app/ui/invoices/breadcrumbs';
import {fetchCustomers} from '@/app/lib/data';

export default async function Page() {
    const customers = await fetchCustomers();

    return (
        <main>
            <Breadcrumbs
                breadcrumbs={[
                    {label: 'Invoices', href: '/dashboard/invoices'},
                    {
                        label: 'Create Invoice',
                        href: '/dashboard/invoices/create',
                        active: true,
                    },
                ]}
            />
            <Form customers={customers}/>
        </main>
    );
}
```

여러분의 페이지는 고객 데이터를 가져와서 `<Form>` 컴포넌트에 전달하는 서버 컴포넌트예요. <br/>
시간을 절약하기 위해 이미 `<Form>` 컴포넌트를 만들어 두었어요.

`<Form>` 컴포넌트로 이동해 보면, 폼에는 다음과 같은 요소들이 있어요:

- 하나의 `<select>` (드롭다운) 요소로 고객 목록이 표시돼요.
- 하나의 `<input>` 요소로 숫자 타입(`type="number"`)의 금액을 입력할 수 있어요.
- 두 개의 `<input>` 요소로 상태를 선택할 수 있어요(`type="radio"`).
- 하나의 버튼이 있고, 이 버튼은 `type="submit"`으로 설정되어 있어요.

http://localhost:3000/dashboard/invoices/create 에 접속하면 아래와 같은 UI를 볼 수 있을 거예요.

<img src="https://nextjs.org/_next/image?url=%2Flearn%2Fdark%2Fcreate-invoice-page.png&w=1920&q=75&dpl=dpl_4kWRRdpV5mEWMp9Zhahs8vP5fgBq" alt="create invoice page preview" />

### Server Action 만들기

이제 폼이 제출되었을 때 호출될 서버 액션을 만들어야 해요.

`lib` 디렉토리로 이동해서 새 파일 `actions.ts`를 만들어요. <br/>
파일의 맨 위에 React 서버 디렉티브인 [`"use server"`](https://react.dev/reference/rsc/use-server)를 추가하세요.

```shell
'use server';
```

`'use server'`라는 키워드를 추가하면, 해당 파일 내에서 내보낸 모든 함수가 서버 액션으로 표시되게 돼요. <br />
이 서버 함수들은 클라이언트와 서버 컴포넌트에서 불러와 사용할 수 있어요.

또한 서버 컴포넌트 내에서 직접 서버 액션을 작성할 수도 있어요. <br />
이때도 `use server`를 해당 액션 안에 직접 추가하면 돼요. <br />
하지만 이 강의에서는 모든 서버 액션을 별도의 파일에 정리해 둘 거예요.

`actions.ts` 파일에 새로운 비동기 함수(async function)를 만들어서 `formData`를 인자로 받도록 해 주세요.

```javascript
'use server';

export async function createInvoice(formData: FormData) {
}
```

이제 `Form` 컴포넌트 안에서, 방금 만든 `actions.ts` 파일에서 `createInvoice` 함수를 가져오세요. <br />
그런 다음, `<form>` 요소에 `action` 속성을 추가하고, `createInvoice` 액션을 호출해 주세요.

이렇게 하면 폼이 제출될 때 `createInvoice` 서버 액션이 실행되어 새 인보이스를 생성하게 돼요.

```javascript
import {customerField} from '@/app/lib/definitions';
import Link from 'next/link';
import {
    CheckIcon,
    ClockIcon,
    CurrencyDollarIcon,
    UserCircleIcon,
} from '@heroicons/react/24/outline';
import {Button} from '@/app/ui/button';
import {createInvoice} from '@/app/lib/actions';

export default function Form({
                                 customers,
                             }: {
    customers: customerField[];
}) {
    return (
        <form action={createInvoice}>
            // ...
            )
            }
```

> ### 알아 두면 좋은 것
> HTML에서는 `action` 속성에 URL을 전달해서 폼 데이터를 제출할 목적지(API 엔드포인트)를 지정해요.
>
> 하지만 React에서는 `action` 속성이 특별한 속성(prop)으로 간주돼서, React가 그 위에 추가 기능을 제공해요. <br />
> 즉, 액션을 호출할 수 있도록 해주는 거예요.

`React Server Action`을 사용하면, 내부적으로 `Server Action`이 자동으로 POST API 엔드포인트를 생성해줘서 따로 API 엔드포인트를 수동으로 만들 필요가 없어요. <br/>
이렇게 하면 폼 제출 시 서버 액션이 실행돼 데이터를 처리할 수 있답니다.

### 3. `formData`에서 데이터를 추출하기

`actions.ts` 파일로 돌아가면, `formData`의 값을 추출해야 해요. <br />
이때 사용할 수 있는 방법이 [몇 가지](https://developer.mozilla.org/en-US/docs/Web/API/FormData/append) 있는데, 예시로 [
`.get(name)`](https://developer.mozilla.org/en-US/docs/Web/API/FormData/get) 메서드를 사용해볼
거예요.

이 내용을 좀 더 쉽게 설명하자면:

1. **formData**는 사용자가 폼에 입력한 데이터를 담고 있어요.
2. `.get(name)` 메서드를 사용하면, 이 데이터에서 특정 이름을 가진 값을 가져올 수 있어요.
3. 예를 들어, 사용자가 입력한 고객의 이름이나 금액 같은 정보를 쉽게 추출할 수 있게 돼요.

이 방법은 데이터를 쉽게 가져올 수 있도록 도와주기 때문에 많이 사용돼요.

```javascript
'use server';

export async function createInvoice(formData: FormData) {
    const rawFormData = {
        customerId: formData.get('customerId'),
        amount: formData.get('amount'),
        status: formData.get('status'),
    };
    // Test it out:
    console.log(rawFormData);
}
```

> ### Tip
> 많은 필드가 있는 폼을 다룰 때는 [`entries()`](https://developer.mozilla.org/en-US/docs/Web/API/FormData/entries) 메서드와 JavaScript의
[
`Object.fromEntries()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/fromEntries)
> 를 사용하는 것이 좋을 수 있어요.
>
> example:
>
> ```javascript
> const rawFormData = Object.fromEntries(formData.entries())
> ```
>
> 이 내용을 쉽게 설명하자면:
>
> 1. **entries() 메서드**: 이 메서드는 폼에 입력된 모든 데이터를 배열 형태로 반환해요. 즉, 각 필드의 이름과 값을 쌍으로 묶어줍니다.
> 2. **Object.fromEntries()**: 이 함수는 배열을 객체로 변환해줘요. 그래서 모든 폼 데이터를 한 번에 객체 형태로 쉽게 사용할 수 있게 돼요.
>
> 예를 들어, 사용자가 입력한 모든 정보를 한 번에 추출하고 싶을 때 이 방법이 유용해요.
>
> 이렇게 하면 코드가 간단해지고, 필드가 많을 때도 편리하게 데이터를 관리할 수 있어요.

모든 것이 제대로 연결되었는지 확인하기 위해 폼을 사용해 보세요. <br />
`submit`한 후, 방금 폼에 입력한 데이터가 터미널에 로그로 표시될 거예요.

이제 데이터가 객체 형태로 되어 있어서 작업하기가 훨씬 수월해질 거예요.

### 4. 데이터를 검증하고 준비하기

제출한 양식 데이터가 올바른 형식과 타입으로 되어 있는지 확인하고 싶어요. <br />
여러분이 앞에서 배운 것처럼, 청구서(invoices) 테이블은 다음과 같은 형식의 데이터를 기대하고 있어요:

1. **고객 ID**: 선택된 고객의 고유 식별자.
2. **금액**: 숫자 형태로, 청구서의 금액.
3. **상태**: 라디오 버튼으로 선택된 상태, 예를 들어 "지불 완료" 또는 "미지급" 등이 될 수 있어요.

이러한 형식과 타입이 맞지 않으면, 데이터베이스에 저장할 수 없으니 반드시 확인해야 해요. <br />
올바른 형식으로 데이터가 준비되면, 데이터베이스에 쉽게 저장할 수 있답니다!

```typescript
export type Invoice = {
    id: string; // Will be created on the database
    customer_id: string;
    amount: number; // Stored in cents
    status: 'pending' | 'paid';
    date: string;
};
```

지금까지 양식(form)에서 가져온 정보는 고객 ID(customer_id), 금액(amount), 그리고 상태(status) 뿐이에요.

이 데이터들은 청구서를 만들기 위해 꼭 필요한 정보인데요, 다른 필요한 정보가 있는지 확인해볼 필요가 있어요. <br />
예를 들어, 만약 날짜나 설명 같은 다른 정보도 필요하다면, 그 부분을 추가로 입력받는 양식을 만들어야 할 수도 있답니다.

**타입 검증 및 변환**

양식에서 가져온 데이터가 데이터베이스에서 기대하는 타입과 일치하는지 확인하는 것은 매우 중요해요. <br />
예를 들어, 액션(action) 내부에 `console.log`를 추가하면, 입력된 데이터의 타입을 확인할 수 있습니다.

이 과정을 통해 입력된 데이터가 올바른 형식인지, 예를 들어 숫자형, 문자열 등인지 확인할 수 있답니다. <br/>
만약 데이터의 타입이 예상과 다르다면, 오류가 발생할 수 있으므로 이를 미리 점검해야 해요.

```javascript
console.log(typeof rawFormData.amount);
```

입력한 금액(amount)이 실제로는 숫자형이 아닌 문자열(string) 타입이라는 점에 주목해야 해요. <br />
이는 `type="number"`로 설정된 입력 요소가 실제로는 문자열을 반환하기 때문이에요!

타입 검증을 처리하는 방법은 여러 가지가 있어요. <br />
수동으로 타입을 검증할 수도 있지만, 타입 검증 라이브러리를 사용하면 시간과 노력을 절약할 수 있어요. <br/>
이번 예제에서는 [Zod](https://zod.dev/)라는 TypeScript 우선의 검증 라이브러리를 사용할 거예요.

이제 `actions.ts` 파일에서 Zod를 가져오고, 폼 객체의 구조와 일치하는 스키마(schema)를 정의해 보세요. <br />
이 스키마는 데이터를 데이터베이스에 저장하기 전에 `formData`를 검증하는 역할을 한답니다.


> **`schema` 란?**
>
> 스키마(schema)는 데이터 구조를 정의하는 청사진으로, 데이터가 어떻게 구성되고 저장될지를 설명해 줘요.
>
> 일반적으로 데이터베이스나 프로그램에서 데이터를 처리할 때 사용되며, 각 필드의 이름, 타입, 제약 조건 등을 포함합니다.
>
> 예를 들어, 사용자 정보를 저장하는 스키마에는 이름, 이메일 주소, 비밀번호 같은 필드가 있을 수 있고, 각각의 필드는 특정 타입(예: 문자열, 숫자)과 제약 조건(예: 필수 입력, 고유성)을 가질 수 있어요.
>
> 이를 통해 데이터의 일관성과 유효성을 보장할 수 있답니다.
>
> 스키마를 사용하는 것은 데이터 관리와 응용 프로그램의 구조를 명확하게 하고, 오류를 줄이며, 데이터 간의 관계를 이해하는 데 도움을 줘요.

```javascript
'use server';

import {z} from 'zod';

const FormSchema = z.object({
    id: z.string(),
    customerId: z.string(),
    amount: z.coerce.number(),
    status: z.enum(['pending', 'paid']),
    date: z.string(),
});

const CreateInvoice = FormSchema.omit({id: true, date: true});

export async function createInvoice(formData: FormData) {
    // ...
}
```

금액(amount) 필드는 문자열에서 숫자로 변환되도록 설정되어 있으며, 동시에 그 타입도 검증하게 돼요.

그런 다음, `rawFormData`를 `CreateInvoice` 함수에 전달하여 타입을 검증할 수 있어요. <br/>

```javascript
// ...
export async function createInvoice(formData: FormData) {
    const {customerId, amount, status} = CreateInvoice.parse({
        customerId: formData.get('customerId'),
        amount: formData.get('amount'),
        status: formData.get('status'),
    });
}
```

**cents 단위로 저장하기**

통화 값을 센트 단위로 저장하는 것은 좋은 습관이에요. <br/>
이렇게 하면 자바스크립트의 부동 소수점 오류를 피하고 더 높은 정확성을 보장할 수 있습니다. <br/>
예를 들어, 10.99달러를 저장할 때, 이를 1099센트로 변환해 저장함으로써 계산의 일관성을 유지할 수 있죠.

금액은 센트로 변환해볼까요?

```javascript
// ...
export async function createInvoice(formData: FormData) {
    const {customerId, amount, status} = CreateInvoice.parse({
        customerId: formData.get('customerId'),
        amount: formData.get('amount'),
        status: formData.get('status'),
    });
    const amountInCents = amount * 100;
}
```

**new Date 만들기**

마지막으로, 인보이스의 생성 날짜를 "YYYY-MM-DD" 형식으로 만들어 볼게요.

```javascript
// ...
export async function createInvoice(formData: FormData) {
    const {customerId, amount, status} = CreateInvoice.parse({
        customerId: formData.get('customerId'),
        amount: formData.get('amount'),
        status: formData.get('status'),
    });
    const amountInCents = amount * 100;
    const date = new Date().toISOString().split('T')[0];
}
```

### 5. `database`에 데이터 넣기

이제 데이터베이스에 필요한 모든 값을 갖추었으니, 새로운 인보이스를 데이터베이스에 추가하는 SQL 쿼리를 만들고 변수를 전달할 수 있습니다.

```javascript
import {z} from 'zod';
import {sql} from '@vercel/postgres';

// ...

export async function createInvoice(formData: FormData) {
    const {customerId, amount, status} = CreateInvoice.parse({
        customerId: formData.get('customerId'),
        amount: formData.get('amount'),
        status: formData.get('status'),
    });
    const amountInCents = amount * 100;
    const date = new Date().toISOString().split('T')[0];

    await sql`
    INSERT INTO invoices (customer_id, amount, status, date)
    VALUES (${customerId}, ${amountInCents}, ${status}, ${date})
  `;
}
```

지금은 오류를 처리하고 있지 않아요. 이건 다음 장에서 다룰 예정이니 이제 다음 단계로 넘어가도록 해요.

### 6. 재검증 및 리디렉션 (Revalidate and redirect)

Next.js는 [클라이언트 측 라우터 캐시](https://nextjs.org/docs/app/building-your-application/caching#router-cache)를 사용하여 사용자의 브라우저에
경로 세그먼트를 일정 시간 동안 저장해 두어요. <br />
이
캐시는 [사전 가져오기(prefetching)](https://nextjs.org/docs/app/building-your-application/routing/linking-and-navigating#1-prefetching)
와 함께 사용되어 사용자가 경로 간 빠르게 탐색할 수 있도록 하며, 서버에 대한 요청 수를 줄여줍니다.

지금 여러분이 인보이스 경로에서 표시되는 데이터를 업데이트하고 있으므로, 이 캐시를 지우고 서버에 새로운 요청을 트리거해야 해요. <br />
Next.js의 [`revalidatePath`](https://nextjs.org/docs/app/api-reference/functions/revalidatePath) 함수를 사용하면 이 작업을 할 수 있어요.

```javascript
'use server';

import {z} from 'zod';
import {sql} from '@vercel/postgres';
import {revalidatePath} from 'next/cache';

// ...

export async function createInvoice(formData: FormData) {
    const {customerId, amount, status} = CreateInvoice.parse({
        customerId: formData.get('customerId'),
        amount: formData.get('amount'),
        status: formData.get('status'),
    });
    const amountInCents = amount * 100;
    const date = new Date().toISOString().split('T')[0];

    await sql`
    INSERT INTO invoices (customer_id, amount, status, date)
    VALUES (${customerId}, ${amountInCents}, ${status}, ${date})
  `;

    revalidatePath('/dashboard/invoices');
}
```

데이터베이스가 업데이트되면 `/dashboard/invoices` 경로가 재검증되고, 서버에서 새로운 데이터가 가져와질 거예요.

또한, 사용자를 `/dashboard/invoices` 페이지로 다시 리다이렉트하고 싶어요. <br />
이 작업은 Next.js의 [`redirect`](https://nextjs.org/docs/app/api-reference/functions/redirect) 함수를 사용하여 수행할 수 있어요.

```javascript
'use server';

import {z} from 'zod';
import {sql} from '@vercel/postgres';
import {revalidatePath} from 'next/cache';
import {redirect} from 'next/navigation';

// ...

export async function createInvoice(formData: FormData) {
    // ...

    revalidatePath('/dashboard/invoices');
    redirect('/dashboard/invoices');
}
```

축하해요! 이제 첫 번째 서버 액션을 구현했어요. 새로운 인보이스를 추가해보면서 제대로 작동하는지 테스트해보세요.

1. 제출 후 `/dashboard/invoices` 경로로 리다이렉트되어야 해요.
2. 테이블 맨 위에서 새로운 인보이스를 확인할 수 있어야 해요.

<hr />

## invoices 업데이트 하기

인보이스를 업데이트하는 폼은 인보이스를 생성하는 폼과 비슷하지만, 데이터베이스에서 레코드를 업데이트하기 위해 인보이스 ID를 전달해야 해요. 인보이스 ID를 어떻게 얻고 전달할 수 있는지 살펴볼게요.

인보이스를 업데이트하기 위해 다음과 같은 단계를 거칠 거예요:

1. 인보이스 ID를 가진 새로운 동적 라우트 세그먼트를 생성해요.
2. 페이지 파라미터에서 인보이스 ID를 읽어와요.
3. 데이터베이스에서 특정 인보이스를 가져와요.
4. 인보이스 데이터로 폼을 미리 채워요.
5. 데이터베이스에서 인보이스 데이터를 업데이트해요.

### 1. 인보이스 ID로 동적 라우트 세그먼트 생성하기

Next.js에서는 정확한 세그먼트 이름을 모르고 데이터에 따라 라우트를 생성하고 싶을
때 [동적 라우트 세그먼트(dynamic route segments)](https://nextjs.org/docs/app/building-your-application/routing/dynamic-routes)를
만들 수 있어요. <br/>
예를 들어 블로그 포스트 제목이나 제품 페이지 같은 경우에요. <br/>
동적 라우트 세그먼트를 만들려면 폴더 이름을 대괄호([])로 감싸면 됩니다. <br />
예를 들어, [id], [post], [slug]와 같이요.

이제 /invoices 폴더에 [id]라는 새로운 동적 라우트를 만들고, 그 안에 `edit`이라는 새로운 라우트를 추가하여 `page.tsx` 파일을 생성해 주세요.<br/>
그러면 파일 구조는 다음과 같아야 해요.

<img src="https://nextjs.org/_next/image?url=%2Flearn%2Fdark%2Fedit-invoice-route.png&w=3840&q=75&dpl=dpl_4kWRRdpV5mEWMp9Zhahs8vP5fgBq" alt="dynamic route segments structure" />

여러분의 `<Table>` 컴포넌트 안에는 `invoice`의 id를 받아오는 `<UpdateInvoice />` 버튼이 있어요.

```javascript
export default async function InvoicesTable({query, currentPage}: {
    query: string;
    currentPage: number;
}) {
    return (
        // ...
        <td className="flex justify-end gap-2 whitespace-nowrap px-6 py-4 text-sm">
            <UpdateInvoice id={invoice.id}/>
            <DeleteInvoice id={invoice.id}/>
        </td>
        // ...
    );
}
```

`<UpdateInvoice />` 컴포넌트로 이동해서, `Link`의 href 속성을 업데이트하여 id prop을 받을 수 있도록 해주세요. <br/>
동적 라우트 세그먼트에 링크를 연결하기 위해 템플릿 리터럴을 사용할 수 있어요.

```javascript
import {PencilIcon, PlusIcon, TrashIcon} from '@heroicons/react/24/outline';
import Link from 'next/link';

// ...

export function UpdateInvoice({id}: { id: string }) {
    return (
        <Link
            href={`/dashboard/invoices/${id}/edit`}
            className="rounded-md border p-2 hover:bg-gray-100"
        >
            <PencilIcon className="w-5"/>
        </Link>
    );
}
```

### 2. 페이지 `params`에서 invoice `id` 읽기

이제 `<Page />` 컴포넌트로 돌아와서 아래의 코드를 넣어보세요.

`/app/dashboard/invoices/[id]/edit/page.tsx` 에 넣으셔야 해요!

```javascript
import Form from '@/app/ui/invoices/edit-form';
import Breadcrumbs from '@/app/ui/invoices/breadcrumbs';
import {fetchCustomers} from '@/app/lib/data';

export default async function Page() {
    return (
        <main>
            <Breadcrumbs
                breadcrumbs={[
                    {label: 'Invoices', href: '/dashboard/invoices'},
                    {
                        label: 'Edit Invoice',
                        href: `/dashboard/invoices/${id}/edit`,
                        active: true,
                    },
                ]}
            />
            <Form invoice={invoice} customers={customers}/>
        </main>
    );
}
```

이제 `/create` invoice 페이지와 비슷한 점을 알아차릴 수 있어요. <br />
하지만 이번에는 edit-form.tsx 파일에서 다른 양식을 가져와야 해요. <br />
이 양식은 고객의 이름, 청구서 금액, 상태를 미리 채워 넣어야 해요. <br />
양식 필드를 미리 채우기 위해서는 id를 사용해서 특정 청구서를 가져와야 해요.

`searchParams` 외에도, 페이지 컴포넌트는 `params`라는 속성을 받아서 id에 접근할 수 있어요. <br />
<Page> 컴포넌트를 업데이트하여 이 속성을 받을 수 있도록 해주세요.

```javascript
import Form from '@/app/ui/invoices/edit-form';
import Breadcrumbs from '@/app/ui/invoices/breadcrumbs';
import {fetchCustomers} from '@/app/lib/data';

export default async function Page(props: { params: Promise<{ id: string }> }) {
    const params = await props.params;
    const id = params.id;
    // ...
}
```

### 3. 특정 invoice 가져오기

그런 다음에:

- `fetchInvoiceById`라는 새로운 함수를 가져와서 id를 인수로 전달하세요.
- 고객 이름을 가져오기 위해 `fetchCustomers`를 가져와야 해요.

두 개의 요청을 동시에 처리하기 위해 `Promise.all`을 사용할 수 있어요. <br />
이렇게 하면 청구서와 고객 데이터를 동시에 가져올 수 있답니다.

```javascript
import Form from '@/app/ui/invoices/edit-form';
import Breadcrumbs from '@/app/ui/invoices/breadcrumbs';
import {fetchInvoiceById, fetchCustomers} from '@/app/lib/data';

export default async function Page(props: { params: Promise<{ id: string }> }) {
    const params = await props.params;
    const id = params.id;
    const [invoice, customers] = await Promise.all([
        fetchInvoiceById(id),
        fetchCustomers(),
    ]);
    // ...
}
```

일시적으로 터미널에서 invoice prop에 대해 TS 오류가 발생할 수 있어요. <br />
이 오류는 invoice가 정의되지 않을 수 있기 때문인데, 걱정하지 마세요. 다음 장에서 오류 처리를 추가할 때 해결할 거예요.

좋아요! 이제 모든 것이 제대로 연결되었는지 테스트해볼까요? <br />
http://localhost:3000/dashboard/invoices를 방문하고 연필 아이콘을 클릭하여 청구서를 편집해 보세요. <br />
이동한 후에는 청구서 세부 정보로 미리 채워진 양식을 볼 수 있어야 해요.

<img src="https://nextjs.org/_next/image?url=%2Flearn%2Fdark%2Fedit-invoice-page.png&w=1920&q=75&dpl=dpl_4JAF45ujf7Jjix56w1X5xA6vEZtg" alt="filled form" />

URL도 다음과 같이 id가 포함되도록 업데이트되어야 해요: http://localhost:3000/dashboard/invoice/uuid/edit

> **UUIDs vs. Auto-incrementing Keys**
> 우리는 고유 식별자(UUID)를 사용해요. 이는 숫자가 1, 2, 3처럼 계속 증가하는 방식 대신에 사용되죠.
>
> 이렇게 하면 URL이 조금 길어지지만, UUID는 ID 충돌의 위험을 없애고, 전 세계에서 유일하며, 시퀀스 공격의 위험을 줄여줘요. <br />
> 그래서 큰 데이터베이스에 적합해요.
>
> 하지만 만약 더 깔끔한 URL을 원하신다면, 자동 증가하는 키를 사용하는 것도 좋은 방법이에요.

### 4. Server Action 으로 id 전달하기

마지막으로, server action 에 ID를 전달하여 데이터베이스에서 올바른 데이터를 업데이트할 수 있도록 해야 해요. <br />
하지만 ID를 이렇게 함수의 인자로 직접 전달할 수는 없어요:

```javascript
// Passing an id as argument won't work
<form action={updateInvoice(id)}>
```

대신, ID를 server action 에 전달하려면 JavaScript의 `bind`를 사용해야 해요. <br />
이렇게 하면 server action에 전달된 값이 인코딩되어 안전하게 전송될 수 있어요.

```javascript
// ...
import {updateInvoice} from '@/app/lib/actions';

export default function EditInvoiceForm({invoice, customers}: {
    invoice: InvoiceForm;
    customers: CustomerField[];
}) {
    const updateInvoiceWithId = updateInvoice.bind(null, invoice.id);

    return (
        <form action={updateInvoiceWithId}>
            <input type="hidden" name="id" value={invoice.id}/>
        </form>
    );
}
```

<details>
<summary>안되는 이유가 무엇일까?</summary>

위 코드에서 `form action={updateInvoice(id)}`와 같이 작성하면, `updateInvoice` 함수가 즉시 실행되어 반환값이 `action`에 할당됩니다. <br />
이렇게 하면 ID를 서버 작업에 전달할 수 없고, 대신 서버 작업이 제대로 연결되지 않을 수 있습니다. 이유는 다음과 같습니다:

1. **즉시 실행**: `updateInvoice(id)`는 즉시 실행되기 때문에, 폼이 제출될 때마다 새로운 요청을 생성하지 않고, 불필요하게 즉시 실행된 결과를 사용하게 됩니다. <br />
   이 결과는 폼의 제출과 관계없이 항상 같은 값을 반환할 수 있습니다.

2. **바인딩**: 반면에, `const updateInvoiceWithId = updateInvoice.bind(null, invoice.id)`와 같이 바인딩하면, `updateInvoice` 함수가 나중에
   호출되도록 설정할 수 있습니다. <br />
   이렇게 하면, 폼이 제출될 때 올바른 ID가 전달됩니다. <br />
   `bind` 메서드는 특정 값으로 함수의 `this`를 고정시키고, 매개변수를 사전 설정하여 나중에 사용할 수 있게 합니다.

3. **폼 데이터 전송**: 또한, 폼에 숨겨진 입력 필드를 추가하여 ID를 제출할 수 있습니다. 이 방법으로 ID를 명시적으로 서버에 전달할 수 있고, 서버 작업은 이 데이터를 사용할 수 있습니다.

결론적으로, `bind`를 사용하면 ID가 올바르게 전달되고, 나중에 폼이 제출될 때마다 `updateInvoice`가 올바른 값으로 호출될 수 있습니다.
</details>

> 폼에서 `hidden` (숨겨진) 입력 필드를 사용해서 데이터를 전달할 수 있어요 (예를 들어, `<input type="hidden" name="id" value={invoice.id} />`).
>
> 하지만 이 방법은 HTML 소스 코드에 그 값이 그대로 나타나기 때문에, `id`와 같은 중요한 정보가 노출될 수 있어요. 그래서 중요한 정보를 숨기는 데는 좋은 방법이 아니랍니다!

그런 다음, `actions.ts` 파일에 `updateInvoice` 라는 새로운 함수를 만들어요 :

```javascript
// Use Zod to update the expected types
const UpdateInvoice = FormSchema.omit({id: true, date: true});

// ...

export async function updateInvoice(id: string, formData: FormData) {
    const {customerId, amount, status} = UpdateInvoice.parse({
        customerId: formData.get('customerId'),
        amount: formData.get('amount'),
        status: formData.get('status'),
    });

    const amountInCents = amount * 100;

    await sql`
    UPDATE invoices
    SET customer_id = ${customerId}, amount = ${amountInCents}, status = ${status}
    WHERE id = ${id}
  `;

    revalidatePath('/dashboard/invoices');
    redirect('/dashboard/invoices');
}
```

다음은 `createInvoice` 액션에서 했던 작업과 비슷한 과정을 통해 진행하는 내용이에요:

1. **formData에서 데이터를 추출**해요. 폼에 입력된 데이터를 서버로 보낼 준비를 해요.
2. **Zod로 타입을 검증**해요. 데이터가 올바른 형태인지 확인해요.
3. **금액을 센트로 변환**해요. 금액을 더 정확하게 저장하기 위해 100배로 변환해요.
4. **SQL 쿼리에 변수를 전달**해요. 데이터를 데이터베이스에 저장해요.
5. **revalidatePath를 호출**해서 클라이언트의 캐시를 지우고 새 서버 요청을 보내요.
6. **redirect를 호출**해 사용자를 청구서 페이지로 다시 보내요.

invoice를 수정한 후 submit하면, invoice 페이지로 이동하고 수정된 invoice가 나타나야 해요.

<hr />

## Invoice 삭제하기

청구서를 삭제하기 위해서, 삭제 버튼을 `<form>` 요소로 감싸고 `bind`를 사용해 서버 작업에 ID를 전달해요. <br />
이렇게 하면 삭제할 청구서의 ID를 안전하게 서버 작업에 넘길 수 있어요.

```javascript
import {deleteInvoice} from '@/app/lib/actions';

// ...

export function DeleteInvoice({id}: { id: string }) {
    const deleteInvoiceWithId = deleteInvoice.bind(null, id);

    return (
        <form action={deleteInvoiceWithId}>
            <button type="submit" className="rounded-md border p-2 hover:bg-gray-100">
                <span className="sr-only">Delete</span>
                <TrashIcon className="w-4"/>
            </button>
        </form>
    );
}
```

`actions.ts` 파일 안에 `deleteInvoice` 함수를 만들어요.

```javascript
export async function deleteInvoice(id: string) {
    await sql`DELETE FROM invoices WHERE id = ${id}`;
    revalidatePath('/dashboard/invoices');
}
```

이 작업은 `/dashboard/invoices` 경로에서 호출되기 때문에, 리다이렉트를 할 필요가 없어요. <br />
`revalidatePath`를 호출하면 새로운 서버 요청이 발생하고 테이블이 다시 렌더링돼요.<br />
이렇게 하면 페이지를 새로 고침하지 않고도 변경된 내용을 바로 볼 수 있게 돼요.

<hr />

## 추가 자료

이번 장에서는 `Server Actions`를 사용하여 데이터를 수정하는 방법을 배웠어요. <br />
또한 `revalidatePath` API를 사용하여 Next.js 캐시를 재검증하고, 사용자를 새로운 페이지로 리다이렉트하는 방법도 배웠답니다.

추가로 [`Server Actions`의 보안](https://nextjs.org/blog/security-nextjs-server-components-actions)에 대해 더 알아보는 것도 좋을
거예요. <br />
이 내용에 대해 더 배우고 싶다면 관련 자료를 찾아보는 것도 추천해요!
