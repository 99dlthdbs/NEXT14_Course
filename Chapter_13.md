이전 챕터에서는 서버 작업을 사용하여 데이터를 변경하는 방법을 배웠어요. <br />
이제는 자바스크립트의 `try/catch` 문과 Next.js API를 사용하여 오류를 우아하게 처리하는 방법을 알아볼 거예요.

즉, 데이터 변경 중에 문제가 발생했을 때, 어떻게 안전하게 오류를 잡고 처리할 수 있는지를 배우는 거예요.

## 이번 챕터에서는...

- 어떻게 특별한 error.tsx 파일을 사용하여 경로에서 발생하는 오류를 잡고, 사용자에게 대체(fallback) UI를 보여줄 수 있는지 배워요.
- `resource`가 존재하지 않을 때 404 오류를 처리하기 위해 notFound 함수를 사용하고, `not-found` 파일을 사용하는 방법을 알아봐요.

<hr />

## Server Action 에 `try/catch` 추가하기

먼저, 서버 작업에 JavaScript의 `try/catch` 문을 추가해서 오류를 우아하게 처리할 수 있도록 해요. <br />
만약 이 방법을 이미 알고 있다면 몇 분 동안 서버 작업을 업데이트해보거나, 아래 코드를 복사해 사용할 수 있어요.

`/app/lib/actions.ts`

```javascript
export async function createInvoice(formData: FormData) {
    const {customerId, amount, status} = CreateInvoice.parse({
        customerId: formData.get('customerId'),
        amount: formData.get('amount'),
        status: formData.get('status'),
    });

    const amountInCents = amount * 100;
    const date = new Date().toISOString().split('T')[0];

    try {
        await sql`
      INSERT INTO invoices (customer_id, amount, status, date)
      VALUES (${customerId}, ${amountInCents}, ${status}, ${date})
    `;
    } catch (error) {
        return {
            message: 'Database Error: Failed to Create Invoice.',
        };
    }

    revalidatePath('/dashboard/invoices');
    redirect('/dashboard/invoices');
}
```

`/app/lib/actions.ts`

```javascript
export async function updateInvoice(id: string, formData: FormData) {
    const {customerId, amount, status} = UpdateInvoice.parse({
        customerId: formData.get('customerId'),
        amount: formData.get('amount'),
        status: formData.get('status'),
    });

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

`/app/lib/actions.ts`

```javascript
export async function deleteInvoice(id: string) {
    try {
        await sql`DELETE FROM invoices WHERE id = ${id}`;
        revalidatePath('/dashboard/invoices');
        return {message: 'Deleted Invoice.'};
    } catch (error) {
        return {message: 'Database Error: Failed to Delete Invoice.'};
    }
}
```

리다이렉트는 `try/catch` 블록 밖에서 호출되고 있다는 점에 주목해요. <br />
이는 리다이렉트가 오류를 발생시켜서 `catch` 블록에서 잡히기 때문이에요. <br />
이를 피하려면 `try/catch` 다음에 리다이렉트를 호출할 수 있어요. <br />
리다이렉트는 `try`가 성공적으로 실행된 경우에만 도달할 수 있답니다.

> **왜 redirect 가 오류를 발생할까?**
>
> `redirect` 함수가 오류를 발생시키는 이유는 Next.js가 내부적으로 이 함수를 사용해 리다이렉션을 처리하기 때문이에요.
>
> `redirect`는 실제로 특수한 오류를 발생시켜 현재 코드의 실행을 중단하고, 사용자에게 새로운 페이지로 이동하도록 지시해요.
>
> 따라서, 이 `redirect` 함수가 `try/catch` 블록 안에 있을 경우, 이 오류는 `catch` 블록에 의해 포착되어 리다이렉션이 정상적으로 실행되지 않고 `catch`가 처리되게 돼요.
>
> 이를 방지하려면, `redirect` 함수를 `try/catch` 블록 밖에서 호출해서 성공적으로 실행된 후 리다이렉션이 이루어지도록 하는 거예요.
>
>쉽게 말하면, `redirect`는 Next.js가 페이지 이동을 위해 의도적으로 발생시키는 오류이기 때문에, 이를 `catch` 블록에 잡히지 않게 해주어야 한다는 거죠.

이제 서버 작업에서 오류가 발생했을 때 어떤 일이 일어나는지 확인해볼까요. <br />
예를 들어, `deleteInvoice` 작업에서 함수의 맨 위에 오류를 발생시키는 코드를 추가하면 돼요.

```javascript
export async function deleteInvoice(id: string) {
    throw new Error('Failed to Delete Invoice');

    // Unreachable code block
    try {
        await sql`DELETE FROM invoices WHERE id = ${id}`;
        revalidatePath('/dashboard/invoices');
        return {message: 'Deleted Invoice'};
    } catch (error) {
        return {message: 'Database Error: Failed to Delete Invoice'};
    }
}
```

invoice를 삭제하려고 할 때, localhost에서 오류를 볼 수 있어요. <br />
테스트 후에는 이 오류를 제거해야 해요. <br />
개발 중에 이러한 오류를 보는 것은 유용하지만, 갑작스러운 실패를 방지하고 애플리케이션이 계속 실행될 수 있도록 사용자에게도 오류를 보여주는 것이 중요해요.<br/>

이를 위해 Next.js의 `error.tsx` 파일을 사용해요.

<hr />

## `error.tsx` 를 이용하여 error 핸들링하기

`error.tsx` 파일은 경로 구간에서 UI 경계 역할을 하며, 예상치 못한 오류를 잡아 사용자에게 대체 UI를 보여줄 수 있어요. <br />
`dashboard/invoices` 폴더 안에 새로운 `error.tsx` 파일을 만들고 다음 코드를 붙여넣으면 돼요.

```javascript
'use client';

import {useEffect} from 'react';

export default function Error({error, reset}: {
    error: Error & { digest?: string };
    reset: () => void;
}) {
    useEffect(() => {
        // Optionally log the error to an error reporting service
        console.error(error);
    }, [error]);

    return (
        <main className="flex h-full flex-col items-center justify-center">
            <h2 className="text-center">Something went wrong!</h2>
            <button
                className="mt-4 rounded-md bg-blue-500 px-4 py-2 text-sm text-white transition-colors hover:bg-blue-400"
                onClick={
                    // Attempt to recover by trying to re-render the invoices route
                    () => reset()
                }
            >
                Try again
            </button>
        </main>
    );
}
```

위의 코드에서 몇 가지를 볼 수 있어요:

- `"use client"`는 `error.tsx`가 클라이언트 컴포넌트여야 한다는 것을 의미해요.
- 두 가지 props를 받아요:
    - `error`: JavaScript의 기본 Error 객체의 인스턴스를 나타내요.
    - `reset`: 오류 경계를 재설정하는 함수로, 실행되면 경로 구간을 다시 렌더링하려고 시도해요.

다시 invoice를 삭제하려고 하면 해당 UI가 표시될 거예요.

<img src="https://nextjs.org/_next/image?url=%2Flearn%2Fdark%2Ferror-page.png&w=1920&q=75&dpl=dpl_4JAF45ujf7Jjix56w1X5xA6vEZtg" alt="error page" />

<hr/>

## `not found` 함수를 사용하여 404 errors 핸들링하기

오류를 처리하는 또 다른 방법은 `notFound` 함수를 사용하는 것이에요. <br />
`error.tsx`는 모든 오류를 잡아줄 수 있지만, `notFound`는 찾으려는 자원이 존재하지 않을 때 사용할 수 있어요.

예를 들어, http://localhost:3000/dashboard/invoices/2e94d1ed-d220-449f-9f11-f0bbceed9645/edit 주소로 가보세요. <br />
이 UUID는 데이터베이스에 존재하지 않는 가짜 값이에요.

이 경로는 `/invoices`의 자식 경로이기 때문에 즉시 `error.tsx`가 작동할 거예요.

하지만, 더 구체적으로 처리하고 싶다면, 사용자가 찾고 있는 자원이 없다는 것을 알리는 404 오류를 표시할 수 있어요.

자원이 없다는 것을 확인하려면 `data.ts` 파일에 있는 `fetchInvoiceById` 함수에서 반환된 청구서를 `console.log`로 확인할 수 있어요.

```javascript
export async function fetchInvoiceById(id: string) {
    noStore();
    try {
        // ...

        console.log(invoice); // Invoice is an empty array []
        return invoice[0];
    } catch (error) {
        console.error('Database Error:', error);
        throw new Error('Failed to fetch invoice.');
    }
}
```

이제 invoice가 데이터베이스에 존재하지 않는다는 것을 알았으니, `notFound`를 사용해 처리해볼게요. <br />
`/dashboard/invoices/[id]/edit/page.tsx` 파일로 이동한 후, `next/navigation`에서 `{ notFound }`를 불러와요.

그런 다음, 조건문을 사용해서 청구서가 없을 때 `notFound` 함수를 호출하도록 설정할 수 있어요.

```javascript
import {fetchInvoiceById, fetchCustomers} from '@/app/lib/data';
import {updateInvoice} from '@/app/lib/actions';
import {notFound} from 'next/navigation';

export default async function Page(props: { params: Promise<{ id: string }> }) {
    const params = await props.params;
    const id = params.id;
    const [invoice, customers] = await Promise.all([
        fetchInvoiceById(id),
        fetchCustomers(),
    ]);

    if (!invoice) {
        notFound();
    }

    // ...
}
```

이제 특정 invoice를 찾을 수 없을 때, `<Page>`가 오류를 발생시키도록 설정되었어요. <br />
사용자에게 오류 화면을 보여주기 위해서 `/edit` 폴더 안에 `not-found.tsx` 파일을 새로 만들어 주세요.

<img src="https://nextjs.org/_next/image?url=%2Flearn%2Fdark%2Fnot-found-file.png&w=3840&q=75&dpl=dpl_4JAF45ujf7Jjix56w1X5xA6vEZtg" alt="not found file route" />

그런 다음, `not-found.tsx` 파일에 아래의 코드를 넣어주세요.

```javascript
import Link from 'next/link';
import {FaceFrownIcon} from '@heroicons/react/24/outline';

export default function NotFound() {
    return (
        <main className="flex h-full flex-col items-center justify-center gap-2">
            <FaceFrownIcon className="w-10 text-gray-400"/>
            <h2 className="text-xl font-semibold">404 Not Found</h2>
            <p>Could not find the requested invoice.</p>
            <Link
                href="/dashboard/invoices"
                className="mt-4 rounded-md bg-blue-500 px-4 py-2 text-sm text-white transition-colors hover:bg-blue-400"
            >
                Go Back
            </Link>
        </main>
    );
}
```

경로를 새로고침하면, 이제 다음과 같은 화면을 볼 수 있을 거예요.

<img src="https://nextjs.org/_next/image?url=%2Flearn%2Fdark%2F404-not-found-page.png&w=1920&q=75&dpl=dpl_4JAF45ujf7Jjix56w1X5xA6vEZtg" alt="not found page preview" />

이 점을 기억해 두면 좋겠어요. `notFound`는 `error.tsx`보다 우선적으로 작동하니까, 더 구체적인 오류를 처리하고 싶을 때 사용할 수 있어요!

<hr />

## 관련 자료

Next.js error 핸들링에 대해 더 공부하고 싶다면 아래의 자료를 살펴보세요.

- [Error Handling](https://nextjs.org/docs/app/building-your-application/routing/error-handling)
- [`error.js` API Reference](https://nextjs.org/docs/app/api-reference/file-conventions/error)
- [`notFound()` API Reference](https://nextjs.org/docs/app/api-reference/functions/not-found)
- [`not-found.js` API Reference](https://nextjs.org/docs/app/api-reference/file-conventions/not-found)
