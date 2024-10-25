이전 챕터에서는 오류를 잡고(404 오류 포함) 사용자에게 대체 화면을 보여주는 방법에 대해 살펴보았어요. <br/>
하지만 우리는 또 다른 중요한 부분인 폼 유효성 검증에 대해서도 이야기해야 해요. <br />
이제 서버 측에서 유효성 검증을 구현하는 방법과 React의 [`useActionState`](https://react.dev/reference/react/useActionState) 훅을 사용하여 폼 오류를
보여주는 방법을 알아보면서 접근성도 고려해 볼 거예요!

## 이번 챕터에서는...

- Next.js에서 접근성 모범 사례를 구현하기 위해 `eslint-plugin-jsx-a11y`를 사용하는 방법에 대해 배울 거예요.

- 서버 측에서 폼 유효성 검증을 구현하는 방법도 알아볼 거예요.

- React의 `useActionState` 훅을 사용하여 폼 오류를 처리하고 사용자에게 오류를 표시하는 방법을 배울 거예요.

<hr />

## 접근성(accessibility)이란?

접근성(Accessibility)은 모든 사용자가 사용할 수 있도록 웹 애플리케이션을 설계하고 구현하는 것을 의미해요. <br />
여기에는 장애가 있는 사람들을 포함해서 모두가 이용할 수 있도록 하는 것이 중요합니다. <br/>
접근성은 키보드 내비게이션, 의미론적 HTML, 이미지, 색상, 비디오 등 다양한 영역을 포함하는 광범위한 주제예요.

이 과정에서는 접근성에 대해 깊이 다루지는 않지만, Next.js에서 제공하는 접근성 기능과 애플리케이션을 더 접근 가능하게 만들기 위한 일반적인 관행에 대해 논의할 예정이에요. <br />
접근성을 고려한 디자인은 모든 사용자에게 더 나은 경험을 제공하는 데 큰 도움이 됩니다.

> 접근성에 대해 더 배우고 싶으시다면, [web.dev](https://web.dev/?hl=ko)에서
> 제공하는 ["Learn Accessibility"](https://web.dev/learn/accessibility/) 과정을 추천해 드려요.

<hr />

## Next.js에서 ESLint 접근성 플러그인 사용하기

Next.js는 접근성 문제를 조기에 발견할 수 있도록 [eslint-plugin-jsx-a11y](https://www.npmjs.com/package/eslint-plugin-jsx-a11y) 플러그인을
ESLint 설정에 포함하고 있어요. <br />
예를 들어, 이 플러그인은 alt 텍스트가 없는 이미지나 aria-* 및 role 속성을 잘못 사용하는 경우에 경고를 해줘요.

사용해보려면 package.json 파일에 next lint를 스크립트로 추가하면 된답니다.

```
"scripts": {
    "build": "next build",
    "dev": "next dev",
    "start": "next start",
    "lint": "next lint"
},
```

`pnpm lint` 로 실행해보세요.

```shell
pnpm lint
```

이렇게 하면 프로젝트에 ESLint를 설치하고 설정하는 방법을 안내받을 수 있어요. 지금 pnpm lint를 실행하면 다음과 같은 결과를 볼 수 있답니다.

```shell
✔ No ESLint warnings or errors
```

이미지에 alt 속성이 없다면 어떻게 될까요? 한번 확인해 볼까요!

`/app/ui/invoices/table.tsx`로 가서 이미지에서 alt 속성을 제거해 보세요. 에디터의 검색 기능을 사용하면 `<Image>`를 빠르게 찾을 수 있어요.

```javascript
<Image
    src={invoice.image_url}
    className="rounded-full"
    width={28}
    height={28}
    alt={`${invoice.name}'s profile picture`} // Delete this line
/>
```

이제 pnpm lint를 다시 실행해 보세요. 그러면 다음과 같은 경고 메시지를 볼 수 있을 거예요.

```
./app/ui/invoices/table.tsx
45:25  Warning: Image elements must have an alt prop,
either with meaningful text, or an empty string for decorative images. jsx-a11y/alt-text
```

linter를 추가하고 설정하는 것은 필수 단계는 아니지만, 개발 과정에서 접근성 문제를 발견하는 데 유용할 수 있어요.

<hr/>

## 접근성 높이기

우리는 이미 form의 접근성을 개선하기 위해 세 가지 방법을 사용하고 있어요:

1. **시맨틱 HTML**: `<div>` 대신에 `<input>`, `<option>`과 같은 의미 있는 요소를 사용해요. <br />
   이렇게 하면 보조 기술(AT)이 입력 요소에 집중하고 사용자에게 적절한 맥락 정보를 제공해 폼을 더 쉽게 탐색하고 이해할 수 있게 도와줘요.

2. **레이블링**: `<label>`과 `htmlFor` 속성을 포함하면 각 폼 필드에 설명적인 텍스트 레이블이 생겨요. <br/>
   이는 보조 기술의 지원을 개선하고, 사용자가 레이블을 클릭하여 해당 입력 필드에 집중할 수 있도록 하여 사용성을 향상시켜요.

3. **포커스 아울라인**: 필드가 포커스 상태일 때 윤곽선이 표시되도록 제대로 스타일링되어 있어요. 이것은 접근성에 매우 중요해요. <br />
   페이지에서 활성 요소를 시각적으로 나타내어 키보드와 화면 리더 사용자 모두가 폼에서 어디에 있는지를 이해하는 데 도움을 줘요. Tab 키를 눌러 확인할 수 있어요.

이런 실천들은 폼을 더 많은 사용자에게 접근 가능하게 만드는 좋은 기반을 다져요. 그러나 폼 검증 및 오류 처리에 대해서는 다루지 않고 있어요.

<hr />

## Form validation (폼 유효성)

http://localhost:3000/dashboard/invoices/create에 가서 빈 form을 제출해 보세요. 그러면 어떤 일이 발생하나요?

에러가 발생해요! 이는 빈 form 값을 서버 작업에 보내기 때문이에요. 이를 방지하기 위해 클라이언트 또는 서버에서 form을 검증할 수 있어요.

### Client-Side validation

클라이언트에서 form을 검증하는 방법은 여러 가지가 있어요. 가장 간단한 방법은 브라우저에서 제공하는 폼 검증 기능을 사용하는 것이에요. <br />
`<input>` 및 `<select>` 요소에 `required` 속성을 추가하면 돼요.

예를 들어:

```javascript
<input
    id="amount"
    name="amount"
    type="number"
    placeholder="Enter USD amount"
    className="peer block w-full rounded-md border border-gray-200 py-2 pl-10 text-sm outline-2 placeholder:text-gray-500"
    required
/>
```

form을 다시 제출해 보세요. 빈 값으로 form을 제출하면 브라우저에서 경고 메시지가 표시될 거예요.

이 방법은 일반적으로 괜찮아요. 왜냐하면 일부 보조 기술(AT)이 브라우저 검증을 지원하기 때문이에요.

클라이언트 측 검증의 대안으로 서버 측 검증이 있어요. 다음 섹션에서 서버 측 검증을 구현하는 방법을 알아볼 거예요. 지금은 추가한 `required` 속성을 삭제해 주세요.

### Server-Side validation

서버에서 form을 검증하면 다음과 같은 이점이 있어요:

1. 데이터가 데이터베이스에 전송되기 전에 예상하는 형식인지 확인할 수 있어요.
2. 악의적인 사용자가 클라이언트 측 검증을 우회할 위험을 줄일 수 있어요.
3. 유효한 데이터의 기준을 한 곳에서 관리할 수 있어요.

이제 `create-form.tsx` 컴포넌트에서 `useActionState` 훅을 가져오세요. <br />
`useActionState`는 훅이기 때문에, 폼을 "use client" 지시어를 사용하여 클라이언트 컴포넌트로 만들어야 해요.

```javascript
'use client';

// ...
import {useActionState} from 'react';
```

form 컴포넌트 안에서 `useActionState` hook은 다음과 같은 역할을 해요:

1. 두 개의 인자를 받아요: `(action, initialState)`.
2. 두 개의 값을 반환해요: `[state, formAction]` - 폼의 상태와 폼이 제출될 때 호출할 함수예요.

`createInvoice` 액션을 `useActionState`의 인자로 넘기고, `<form action={}>` 속성 안에 `formAction`을 호출하면 돼요.

```javascript
// ...
import {useActionState} from 'react';

export default function Form({customers}: { customers: CustomerField[] }) {
    const [state, formAction] = useActionState(createInvoice, initialState);

    return <form action={formAction}>...</form>;
}
```

`initialState`는 개발자가 자유롭게 정의할 수 있어요. <br />
이 경우, 두 개의 빈 키(`message`와 `errors`)를 가진 객체를 만들면 돼요. <br />
그리고 `actions.ts` 파일에서 `State` 타입을 가져오면 돼요.

```javascript
// ...
import {createInvoice, State} from '@/app/lib/actions';
import {useActionState} from 'react';

export default function Form({customers}: { customers: CustomerField[] }) {
    const initialState: State = {message: null, errors: {}};
    const [state, formAction] = useActionState(createInvoice, initialState);

    return <form action={formAction}>...</form>;
}
```

처음에는 조금 헷갈릴 수 있지만, 서버 액션을 업데이트하면 더 이해가 될 거예요. 이제 그 작업을 시작해봐요.

`action.ts` 파일에서 `Zod`를 사용해 폼 데이터를 검증할 수 있어요. `FormSchema`를 아래와 같이 업데이트하면 돼요.

```typescript
const FormSchema = z.object({
    id: z.string(),
    customerId: z.string({
        invalid_type_error: 'Please select a customer.',
    }),
    amount: z.coerce
        .number()
        .gt(0, {message: 'Please enter an amount greater than $0.'}),
    status: z.enum(['pending', 'paid'], {
        invalid_type_error: 'Please select an invoice status.',
    }),
    date: z.string(),
});
```

- `customerId` - `Zod`는 이미 `customer` 필드에 문자열이 필요하다고 설정되어 있어서, 선택하지 않으면 오류가 발생해요. <br/>
  하지만 사용자가 고객을 선택하지 않았을 때 더 친절한 메시지를 추가할 수 있어요.

- `amount` - `amount` 필드를 문자열에서 숫자로 변경할 때, 빈 문자열이면 기본값이 0으로 설정돼요. <br/>
  여기서 `Zod`의 `.gt()` 기능을 사용해 항상 0보다 큰 값을 요구할 수 있어요.

- `status` - `status` 필드도 "pending" 또는 "paid" 값이 필요하다고 설정되어 있어서, 비워두면 `Zod`에서 오류가 발생해요. <br/>
  하지만 사용자가 상태를 선택하지 않았을 때도 더 친절한 메시지를 추가할 수 있어요.

다음으로 `createInvoice` 액션을 업데이트해서 `prevState`와 `formData` 두 가지 매개변수를 받도록 해보세요.

```typescript
export
type
    State = {
    errors?: {
        customerId?: string[];
        amount?: string[];
        status?: string[];
    };
    message?: string | null;
};

export async function createInvoice(prevState: State, formData: FormData) {
    // ...
}
```

- `formData` - 이전과 동일하게 사용돼요.
- `prevState` - `useActionState` 훅에서 전달되는 상태가 담겨 있어요. 이 예시에서는 액션에서 사용하지 않지만, 필수 매개변수로 필요해요.

그리고 `Zod`의 `parse()` 함수를 `safeParse()`로 변경하세요.

```typescript
export async function createInvoice(prevState: State, formData: FormData) {
    // Validate form fields using Zod
    const validatedFields = CreateInvoice.safeParse({
        customerId: formData.get('customerId'),
        amount: formData.get('amount'),
        status: formData.get('status'),
    });

    // ...
}
```

`safeParse()`는 성공 여부나 오류가 담긴 객체를 반환해요. 이렇게 하면 `try/catch` 블록 안에 검증 논리를 넣지 않아도 돼서 검증을 더 깔끔하게 처리할 수 있어요.

데이터베이스로 정보를 보내기 전에 조건문을 사용해 입력된 폼 필드가 올바르게 검증되었는지 확인해 보세요.

```typescript
export async function createInvoice(prevState: State, formData: FormData) {
    // Validate form fields using Zod
    const validatedFields = CreateInvoice.safeParse({
        customerId: formData.get('customerId'),
        amount: formData.get('amount'),
        status: formData.get('status'),
    });

    // If form validation fails, return errors early. Otherwise, continue.
    if (!validatedFields.success) {
        return {
            errors: validatedFields.error.flatten().fieldErrors,
            message: 'Missing Fields. Failed to Create Invoice.',
        };
    }

    // ...
}
```

`validatedFields`가 성공하지 않으면 `Zod`에서 발생한 오류 메시지를 포함해 함수를 일찍 종료해요.

> **TIP** <br />
> `validatedFields`를 `console.log`로 확인하고 빈 양식을 제출해서 결과가 어떻게 표시되는지 살펴보세요.

마지막으로, 이제 양식 검증을 try/catch 블록 외부에서 따로 처리하므로, 데이터베이스 오류가 발생했을 때는 특정 메시지를 반환할 수 있어요.

최종 코드는 이런 형태가 될 거예요.

```typescript
export async function createInvoice(prevState: State, formData: FormData) {
    // Validate form using Zod
    const validatedFields = CreateInvoice.safeParse({
        customerId: formData.get('customerId'),
        amount: formData.get('amount'),
        status: formData.get('status'),
    });

    // If form validation fails, return errors early. Otherwise, continue.
    if (!validatedFields.success) {
        return {
            errors: validatedFields.error.flatten().fieldErrors,
            message: 'Missing Fields. Failed to Create Invoice.',
        };
    }

    // Prepare data for insertion into the database
    const {customerId, amount, status} = validatedFields.data;
    const amountInCents = amount * 100;
    const date = new Date().toISOString().split('T')[0];

    // Insert data into the database
    try {
        await sql`
      INSERT INTO invoices (customer_id, amount, status, date)
      VALUES (${customerId}, ${amountInCents}, ${status}, ${date})
    `;
    } catch (error) {
        // If a database error occurs, return a more specific error.
        return {
            message: 'Database Error: Failed to Create Invoice.',
        };
    }

    // Revalidate the cache for the invoices page and redirect the user.
    revalidatePath('/dashboard/invoices');
    redirect('/dashboard/invoices');
}
```

좋아요, 이제 오류를 폼 컴포넌트에서 표시해 보도록 할게요.

다시 `create-form.tsx` 컴포넌트로 돌아가면 `form state`를 사용해서 오류를 접근할 수 있어요.

삼항 연산자를 사용해서 특정 오류가 있는지 확인할 수 있어요. 예를 들어, 고객 필드 아래에 다음과 같이 추가할 수 있어요.

```javascript
<form action={formAction}>
    <div className="rounded-md bg-gray-50 p-4 md:p-6">
        {/* Customer Name */}
        <div className="mb-4">
            <label htmlFor="customer" className="mb-2 block text-sm font-medium">
                Choose customer
            </label>
            <div className="relative">
                <select
                    id="customer"
                    name="customerId"
                    className="peer block w-full rounded-md border border-gray-200 py-2 pl-10 text-sm outline-2 placeholder:text-gray-500"
                    defaultValue=""
                    aria-describedby="customer-error"
                >
                    <option value="" disabled>
                        Select a customer
                    </option>
                    {customers.map((name) => (
                        <option key={name.id} value={name.id}>
                            {name.name}
                        </option>
                    ))}
                </select>
                <UserCircleIcon
                    className="pointer-events-none absolute left-3 top-1/2 h-[18px] w-[18px] -translate-y-1/2 text-gray-500"/>
            </div>
            <div id="customer-error" aria-live="polite" aria-atomic="true">
                {state.errors?.customerId &&
                    state.errors.customerId.map((error: string) => (
                        <p className="mt-2 text-sm text-red-500" key={error}>
                            {error}
                        </p>
                    ))}
            </div>
        </div>
        // ...
    </div>
</form>
```

> **TIP** <br />
> 컴포넌트 안에서 `state`를 `console.log`로 확인해서 모든 것이 올바르게 연결되어 있는지 체크할 수 있어요.
>
> 폼이 이제 클라이언트 컴포넌트이므로 개발 도구의 콘솔에서 확인해 보세요.

위 코드에서는 다음과 같은 ARIA 레이블을 추가하고 있어요:

- **aria-describedby="customer-error"**: 이 속성은 선택 요소와 오류 메시지 컨테이너 간의 관계를 설정해 줘요. <br />
  이는 `id="customer-error"`인 컨테이너가 선택 요소를 설명한다고 알려주는 것이에요. <br />
  스크린 리더는 사용자가 선택 상자와 상호작용할 때 이 설명을 읽어서 오류를 알려줍니다.

- **id="customer-error"**: 이 id 속성은 선택 입력을 위한 오류 메시지를 담고 있는 HTML 요소를 고유하게 식별해요. 이는 `aria-describedby`가 관계를 설정하는 데 필요해요.

- **aria-live="polite"**: 이 속성은 스크린 리더가 오류가 업데이트될 때 사용자에게 정중하게 알리도록 해요. 내용이 변경될 때(예를 들어 사용자가 오류를 수정했을 때) 스크린 리더가 이러한 변경
  사항을 발표하지만, 사용자가 한가할 때만 발표해서 방해가 되지 않도록 해요.

## 실습 : aria labels 추가하기

위 예시를 활용하여 나머지 폼 필드에도 오류 메시지를 추가하세요. <br />
또한, 필드에 누락된 부분이 있을 경우 폼 하단에 메시지를 표시해야 해요. 당신의 UI는 이렇게 보여야 해요:

<img src="https://nextjs.org/_next/image?url=%2Flearn%2Fdark%2Fform-validation-page.png&w=1920&q=75&dpl=dpl_4JAF45ujf7Jjix56w1X5xA6vEZtg" alt="adding aria labels" />

준비가 되면 `pnpm lint`를 실행하여 aria 레이블을 올바르게 사용하고 있는지 확인하세요.

도전하고 싶다면, 이 장에서 배운 지식을 바탕으로 `edit-form.tsx` 컴포넌트에 폼 유효성 검사를 추가해 보세요.

해야 할 일은 다음과 같아요:

1. `edit-form.tsx` 컴포넌트에 `useActionStat[edit-form.tsx](nextjs-dashboard/app/ui/invoices/edit-form.tsx)e`를 추가하세요.
2. `updateInvoice` 액션을 수정하여 Zod에서 발생하는 유효성 검사 오류를 처리하세요.
3. 컴포넌트에 오류를 표시하고 접근성을 높이기 위해 aria 레이블을 추가하세요.

준비가 되면 아래의 코드 조각을 확장하여 솔루션을 확인해 보세요:

**Edit Invoice Form**

`/app/ui/invoices/edit-form.tsx`

```javascript
// ...
import {updateInvoice, State} from '@/app/lib/actions';
import {useActionState} from 'react';

export default function EditInvoiceForm({
                                            invoice,
                                            customers,
                                        }: {
    invoice: InvoiceForm;
    customers: CustomerField[];
}) {
    const initialState: State = {message: null, errors: {}};
    const updateInvoiceWithId = updateInvoice.bind(null, invoice.id);
    const [state, formAction] = useActionState(updateInvoiceWithId, initialState);

    return <form action={formAction}></form>;
}
```

**Server Action**

`/app/lib/actions.ts`

```javascript
export async function updateInvoice(
    id: string,
    prevState: State,
    formData: FormData,
) {
    const validatedFields = UpdateInvoice.safeParse({
        customerId: formData.get('customerId'),
        amount: formData.get('amount'),
        status: formData.get('status'),
    });

    if (!validatedFields.success) {
        return {
            errors: validatedFields.error.flatten().fieldErrors,
            message: 'Missing Fields. Failed to Update Invoice.',
        };
    }

    const {customerId, amount, status} = validatedFields.data;
    const amountInCents = amount * 100;

    try {
        await sql`
      UPDATE invoices
      SET customer_id = ${customerId}, amount = ${amountInCents}, status = ${status}
      WHERE id = ${id}
    `;
    } catch (error) {
        return {message: 'Database Error: Failed to Update Invoice.'};
    }

    revalidatePath('/dashboard/invoices');
    redirect('/dashboard/invoices');
}
```

여기서 아쉬운 점... <br/>
edit-form 은 기본적으로 모든 필드가 채워진 상태이기 때문에 create message 와 같이 필드를 채워달라는 경고 메시지 + 이전 데이터와 비교하여 같은 데이터면 경고 메시지를 띄우는 validate 를
추가하면 좋을 것 같다.

