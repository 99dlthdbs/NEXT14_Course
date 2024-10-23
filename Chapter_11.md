이전 장에서 대시보드의 초기 로딩 성능을 스트리밍으로 개선했어요.

이제는 `/invoices` 페이지로 넘어가서 검색과 페이지네이션(목록 나누기)을 추가하는 방법을 배워볼 거예요!

## 이번 챕터에서는...

- Next.js API 어떻게 사용하는지 배우기 : `useSearcParams`, `usePathname`, `useRouter`
- URL search params 를 이용하여 페이지네이션과 검색 구현

<hr />

## Starting code

`/dashboard/invoices/page.tsx` 파일 안에 아래의 코드를 붙여넣으세요.

```javascript
import Pagination from '@/app/ui/invoices/pagination';
import Search from '@/app/ui/search';
import Table from '@/app/ui/invoices/table';
import {CreateInvoice} from '@/app/ui/invoices/buttons';
import {lusitana} from '@/app/ui/fonts';
import {InvoicesTableSkeleton} from '@/app/ui/skeletons';
import {Suspense} from 'react';

export default async function Page() {
    return (
        <div className="w-full">
            <div className="flex w-full items-center justify-between">
                <h1 className={`${lusitana.className} text-2xl`}>Invoices</h1>
            </div>
            <div className="mt-4 flex items-center justify-between gap-2 md:mt-8">
                <Search placeholder="Search invoices..."/>
                <CreateInvoice/>
            </div>
            {/*  <Suspense key={query + currentPage} fallback={<InvoicesTableSkeleton />}>
        <Table query={query} currentPage={currentPage} />
      </Suspense> */}
            <div className="mt-5 flex w-full justify-center">
                {/* <Pagination totalPages={totalPages} /> */}
            </div>
        </div>
    );
}
```

페이지와 작업할 컴포넌트에 익숙해지는 시간을 가져보세요:

1. **<Search/>**: 사용자가 특정 청구서를 검색할 수 있도록 해줍니다.
2. **<Pagination/>**: 사용자가 청구서 페이지 간에 이동할 수 있도록 해줍니다.
3. **<Table/>**: 청구서를 표시합니다.

검색 기능은 클라이언트와 서버 양쪽에서 작동합니다. <br />
사용자가 클라이언트에서 청구서를 검색하면, URL 매개변수가 업데이트되고, 서버에서 데이터가 가져와지며, 새로운 데이터로 서버에서 테이블이 다시 렌더링됩니다.

<hr />

## 왜 URL search params 를 사용할까?

앞서 언급한 것처럼, 검색 상태를 관리하기 위해 URL 검색 매개변수를 사용할 거예요. <br />
만약 클라이언트 측 상태로 작업하는 것에 익숙하다면, 이 패턴은 새로울 수 있어요.

URL 매개변수로 검색을 구현하는 몇 가지 이점이 있어요:

- **북마크 가능하고 공유할 수 있는 URL**: 검색 매개변수가 URL에 있기 때문에, 사용자는 현재 애플리케이션 상태를 북마크할 수 있어요. 검색 쿼리와 필터를 포함하여 나중에 참고하거나 공유할 수 있습니다.

- **서버 측 렌더링 및 초기 로드**: URL 매개변수는 서버에서 직접 사용할 수 있어 초기 상태를 렌더링하는 데 쉽게 처리할 수 있어요.

- **분석 및 추적**: 검색 쿼리와 필터가 URL에 직접 포함되면, 추가적인 클라이언트 측 로직 없이 사용자 행동을 추적하기가 더 쉬워요.

이런 방식으로 검색 기능을 구현하면 더 나은 사용자 경험을 제공할 수 있답니다.

<hr />

## 검색 기능 추가

다음은 검색 기능을 구현하기 위해 사용할 Next.js 클라이언트 훅입니다:

- **useSearchParams**: 현재 URL의 파라미터에 접근할 수 있게 해줘요. 예를 들어, URL이 `/dashboard/invoices?page=1&query=pending`일 때, 이 검색 파라미터는
  `{page: '1', query: 'pending'}`처럼 보일 거예요.
- **usePathname**: 현재 URL의 경로를 읽을 수 있게 해줘요. 예를 들어, 경로가 `/dashboard/invoices`인 경우, usePathname은 `'/dashboard/invoices'`를
  반환해요.
- **useRouter**: 클라이언트 컴포넌트 내에서 라우트 사이를 프로그래밍적으로 탐색할 수 있게 해줘요. 여러 가지 방법을 사용할 수 있어요.

구현 단계는 다음과 같아요:

1. 사용자가 입력한 내용을 받아요.
2. 검색 파라미터로 URL을 업데이트해요.
3. 입력 필드와 URL을 동기화해요.
4. 검색 쿼리를 반영하도록 테이블을 업데이트해요.

이렇게 하면 사용자들이 원하는 검색 결과를 쉽게 찾을 수 있게 도와줄 수 있어요.

### 1. 사용자가 입력한 내용 받기

`<Search>` 컴포넌트(`/app/ui/search.tsx`)로 들어가면 다음과 같은 것들을 볼 수 있어요:

- `"use client"` - 이 부분은 이 컴포넌트가 클라이언트 컴포넌트라는 뜻이에요. 그래서 이벤트 리스너와 훅을 사용할 수 있어요.
- `<input>` - 이건 검색 입력창이에요.

새로운 `handleSearch` 함수를 만들고, `<input>` 요소에 `onChange` 리스너를 추가해요. `onChange`는 입력값이 바뀔 때마다 `handleSearch`를 호출하게 돼요.

```javascript
'use client';

import {MagnifyingGlassIcon} from '@heroicons/react/24/outline';

export default function Search({placeholder}: { placeholder: string }) {
    function handleSearch(term: string) {
        console.log(term);
    }

    return (
        <div className="relative flex flex-1 flex-shrink-0">
            <label htmlFor="search" className="sr-only">
                Search
            </label>
            <input
                className="peer block w-full rounded-md border border-gray-200 py-[9px] pl-10 text-sm outline-2 placeholder:text-gray-500"
                placeholder={placeholder}
                onChange={(e) => {
                    handleSearch(e.target.value);
                }}
            />
            <MagnifyingGlassIcon
                className="absolute left-3 top-1/2 h-[18px] w-[18px] -translate-y-1/2 text-gray-500 peer-focus:text-gray-900"/>
        </div>
    );
}
```

정상적으로 작동하는지 확인하려면 개발자 도구에서 콘솔을 열고, 검색창에 입력해 보세요. 그러면 입력한 검색어가 콘솔에 표시될 거예요.

잘했어요! 이제 사용자의 검색 입력을 잘 받아오고 있네요. 다음으로, 검색어로 URL을 업데이트해야 해요.

### 2. search params 를 활용하여 URL 업데이트 하기

'next/navigation'에서 useSearchParams 훅을 가져와서 변수에 할당해 보세요.

```javascript
'use client';

import {MagnifyingGlassIcon} from '@heroicons/react/24/outline';
import {useSearchParams} from 'next/navigation';

export default function Search() {
    const searchParams = useSearchParams();

    function handleSearch(term: string) {
        console.log(term);
    }

    // ...
}
```

`handleSearch` 안에서, 새로운 `searchParams` 변수를 사용하여 새로운 `URLSearchParams` 인스턴스를 생성해 보세요.

```javascript
'use client';

import {MagnifyingGlassIcon} from '@heroicons/react/24/outline';
import {useSearchParams} from 'next/navigation';

export default function Search() {
    const searchParams = useSearchParams();

    function handleSearch(term: string) {
        const params = new URLSearchParams(searchParams);
    }

    // ...
}
```

`URLSearchParams`는 URL 쿼리 매개변수를 조작하기 위한 유틸리티 메서드를 제공하는 웹 API예요. <br/>
복잡한 문자열을 만드는 대신, 이 API를 사용하여 `?page=1&query=a`와 같은 매개변수 문자열을 쉽게 얻을 수 있어요.

다음으로, 사용자의 입력에 따라 매개변수 문자열을 설정해 보세요. 만약 입력이 비어 있다면, 그 매개변수를 삭제해야 해요.

```javascript
'use client';

import {MagnifyingGlassIcon} from '@heroicons/react/24/outline';
import {useSearchParams} from 'next/navigation';

export default function Search() {
    const searchParams = useSearchParams();

    function handleSearch(term: string) {
        const params = new URLSearchParams(searchParams);
        if (term) {
            params.set('query', term);
        } else {
            params.delete('query');
        }
    }

    // ...
}
```

이제 쿼리 문자열이 생겼으니, Next.js의 `useRouter`와 `usePathname` 훅을 사용하여 URL을 업데이트할 수 있어요.

먼저, `next/navigation`에서 `useRouter`와 `usePathname`을 임포트하고, `handleSearch` 안에서 `useRouter()`의 replace 메서드를 사용하세요. <br/>
이렇게 하면 사용자가 입력한 검색어에 따라 URL이 변경될 거예요.

```javascript
'use client';

import {MagnifyingGlassIcon} from '@heroicons/react/24/outline';
import {useSearchParams, usePathname, useRouter} from 'next/navigation';

export default function Search() {
    const searchParams = useSearchParams();
    const pathname = usePathname();
    const {replace} = useRouter();

    function handleSearch(term: string) {
        const params = new URLSearchParams(searchParams);
        if (term) {
            params.set('query', term);
        } else {
            params.delete('query');
        }
        replace(`${pathname}?${params.toString()}`);
    }
}
```

여기서 일어나는 일을 설명할게요:

- `${pathname}`은 현재 경로를 나타내요. 이 경우에는 "/dashboard/invoices"예요.
- 사용자가 검색창에 입력하면, `params.toString()`이 이 입력을 URL에 적합한 형식으로 변환해줘요.
- `replace(${pathname}?${params.toString()})`는 사용자의 검색 데이터를 포함하여 URL을 업데이트해요. 예를 들어, 사용자가 "Lee"를 검색하면 URL은
  `/dashboard/invoices?query=lee`로 변경될 거예요.
- 페이지를 새로 고침하지 않고도 URL이 업데이트되는데, 이는
  Next.js의 [client side navigation](https://nextjs.org/learn/dashboard-app/navigating-between-pages) 덕분이에요. 이건 페이지 간 이동에
  대한 장에서
  배운 내용이에요.

### 3. URL과 입력을 동기화 하기

URL과 input 필드를 동기화하여 입력 필드가 공유될 때도 채워질 수 있도록 하려면, `searchParams`에서 읽어온 값을 `input`에 `defaultValue`로 전달할 수 있어요.

```javascript
<input
    className="peer block w-full rounded-md border border-gray-200 py-[9px] pl-10 text-sm outline-2 placeholder:text-gray-500"
    placeholder={placeholder}
    onChange={(e) => {
        handleSearch(e.target.value);
    }}
    defaultValue={searchParams.get('query')?.toString()}
/>
```

> **`defaultValue` vs `value`**
>
> **Controlled Component**: 입력 값의 상태를 React의 상태(state)로 관리할 때 사용해요. 이 경우 value 속성을 사용하면 React가 입력 필드의 값을 제어하게 됩니다. 즉, 입력
> 값이 항상 React의 상태와 동기화되므로, React가 입력 필드의 값 변화를 관리해요.
>
> **Uncontrolled Component**: 상태를 사용하지 않고, 입력 필드의 초기값을 설정할 때 defaultValue 속성을 사용해요. 이렇게 하면 입력 필드는 기본값으로 시작하지만, 이후에 사용자가
> 입력한 값은 React가 관리하지 않아요. 대신, 입력 필드 자체가 자신의 상태를 관리하게 됩니다. 이러한 방식은 URL에 검색어를 저장하는 상황에서 적합해요.

### 4. table 업데이트 하기

마지막으로, 검색 쿼리를 반영하기 위해 테이블 컴포넌트를 업데이트해야 해요.

invoices 페이지로 이동해주세요.

페이지 컴포넌트는 `[searchParams`라는 프로퍼티](https://nextjs.org/docs/app/api-reference/file-conventions/page)를 받아서 현재 URL의 파라미터를
`<Table>` 컴포넌트에 전달할 수 있어요.

```javascript
import Pagination from '@/app/ui/invoices/pagination';
import Search from '@/app/ui/search';
import Table from '@/app/ui/invoices/table';
import {CreateInvoice} from '@/app/ui/invoices/buttons';
import {lusitana} from '@/app/ui/fonts';
import {Suspense} from 'react';
import {InvoicesTableSkeleton} from '@/app/ui/skeletons';

export default async function Page({
                                       searchParams,
                                   }: {
    searchParams?: {
        query?: string;
        page?: string;
    };
}) {
    const query = searchParams?.query || '';
    const currentPage = Number(searchParams?.page) || 1;

    return (
        <div className="w-full">
            <div className="flex w-full items-center justify-between">
                <h1 className={`${lusitana.className} text-2xl`}>Invoices</h1>
            </div>
            <div className="mt-4 flex items-center justify-between gap-2 md:mt-8">
                <Search placeholder="Search invoices..."/>
                <CreateInvoice/>
            </div>
            <Suspense key={query + currentPage} fallback={<InvoicesTableSkeleton/>}>
                <Table query={query} currentPage={currentPage}/>
            </Suspense>
            <div className="mt-5 flex w-full justify-center">
                {/* <Pagination totalPages={totalPages} /> */}
            </div>
        </div>
    );
}
```

`<Table>` 컴포넌트로 이동하면, `query`와 `currentPage`라는 두 개의 프로퍼티가 `fetchFilteredInvoices()` 함수에 전달된 것을 볼 수 있어요. 이 함수는 검색 쿼리에 맞는
청구서 목록을 반환해요.

```javascript
// ...
export default async function InvoicesTable({
                                                query,
                                                currentPage,
                                            }: {
    query: string;
    currentPage: number;
}) {
    const invoices = await fetchFilteredInvoices(query, currentPage);
    // ...
}
```

이러한 변경 사항이 적용되면, 이제 테스트해보세요. <br/>
검색어를 입력하면 URL이 업데이트되고, 새로운 요청이 서버로 전송돼요. 그러면 서버에서 데이터를 가져오고, 검색 쿼리에 맞는 청구서만 반환될 거예요.

> ** 언제 `useSearchParams()`, `searchParams` prop 을 사용할까?
>
> 두 가지 방법으로 검색 매개변수를 추출하는 방법이 다르다는 것을 알았을 거예요. 어떤 것을 사용할지는 클라이언트에서 작업 중인지 서버에서 작업 중인지에 따라 달라져요.
> - `<Search>`는 클라이언트 컴포넌트이기 때문에, 클라이언트에서 매개변수에 접근하기 위해 `useSearchParams()` 훅을 사용했어요.
> - `<Table>`는 자신의 데이터를 가져오는 서버 컴포넌트이기 때문에, 페이지에서 해당 컴포넌트로 `searchParams` prop을 전달할 수 있어요.
>
> 일반적으로, 클라이언트에서 매개변수를 읽고 싶다면 `useSearchParams()` 훅을 사용하는 것이 좋아요. 이렇게 하면 서버로 다시 요청할 필요가 없어요.

### Best 실습 : 디바운싱 (Debouncing)

축하해요! Next.js로 검색 기능을 구현했어요! 하지만 더 최적화할 수 있는 방법이 있어요.

handleSearch 함수 안에 다음과 같은 console.log를 추가하세요:

```javascript
function handleSearch(term: string) {
    console.log(`Searching... ${term}`);

    const params = new URLSearchParams(searchParams);
    if (term) {
        params.set('query', term);
    } else {
        params.delete('query');
    }
    replace(`${pathname}?${params.toString()}`);
}
```

그런 다음 검색창에 "Emil"을 입력하고 개발자 도구의 콘솔을 확인해 보세요. 무슨 일이 일어나고 있나요?

```shell
Searching... E
Searching... Em
Searching... Emi
Searching... Emil
```

사용자가 키보드를 칠 때마다 URL이 업데이트되고, 그래서 데이터베이스에 요청을 보내고 있어요!

우리의 애플리케이션은 작기 때문에 큰 문제는 아니지만, 만약 수천 명의 사용자가 각각 키를 누를 때마다 데이터베이스에 새로운 요청을 보내고 있다면 상황이 달라질 수 있어요.

디바운싱(debouncing)은 함수가 호출되는 속도를 제한하는 프로그래밍 기법이에요. 이 경우에는 사용자가 입력을 멈췄을 때만 데이터베이스를 쿼리하도록 설정하고 싶어요.

> 디바운싱 작동 원리:
> 1. **이벤트 발생**: 검색창에서 키를 누르는 것처럼 디바운싱해야 하는 이벤트가 발생하면 타이머가 시작돼요.
> 2. **대기**: 타이머가 끝나기 전에 새로운 이벤트가 발생하면, 타이머가 리셋돼요.
> 3. **실행**: 타이머가 카운트다운이 끝나면 디바운스된 함수가 실행돼요.

디바운싱을 구현하는 방법은 여러 가지가 있지만, 복잡하지 않게 하기 위해서 `use-debounce`라는 라이브러리를 사용할 거예요.<br/>
이 라이브러리를 사용하면 더 쉽게 디바운싱 기능을 추가할 수 있답니다.

`use-debounce` 를 설치하세요.

```shell
pnpm i use-debounce
```

`<Search>` 컴포넌트에서 `useDebouncedCallback`이라는 함수를 임포트해 주세요. 이렇게 하면 디바운싱 기능을 사용할 수 있어요.

```javascript
// ...
import {useDebouncedCallback} from 'use-debounce';

// Inside the Search Component...
const handleSearch = useDebouncedCallback((term) => {
    console.log(`Searching... ${term}`);

    const params = new URLSearchParams(searchParams);
    if (term) {
        params.set('query', term);
    } else {
        params.delete('query');
    }
    replace(`${pathname}?${params.toString()}`);
}, 300);
```

이 함수는 `handleSearch`의 내용을 감싸고, 사용자가 입력을 멈춘 후 특정 시간(300ms)이 지나면 코드가 실행돼요.

이제 다시 검색창에 입력해 보세요. 개발자 도구의 콘솔을 열면, 다음과 같은 내용이 보여야 해요.

```shell
Searching... Emil
```

디바운싱을 사용하면 데이터베이스에 보내는 요청의 수를 줄일 수 있어서 자원을 절약할 수 있어요.

<hr />

## pagination 추가하기

검색 기능을 추가한 후, 테이블에는 한 번에 6개의 청구서만 표시된다는 것을 알 수 있어요. <br />
이는 `data.ts`에 있는 `fetchFilteredInvoices()` 함수가 페이지당 최대 6개의 청구서만 반환하기 때문이에요.

페이징 기능을 추가하면 사용자가 다양한 페이지를 탐색하여 모든 청구서를 볼 수 있게 됩니다. <br />
이제 URL 매개변수를 사용하여 페이징을 구현하는 방법을 살펴보도록 할게요. 검색과 마찬가지로요.

`<Pagination/>` 컴포넌트로 이동하면 이 컴포넌트가 클라이언트 컴포넌트라는 것을 알 수 있어요. <br/>
클라이언트에서 데이터를 가져오고 싶지 않은 이유는 데이터베이스 비밀이 노출될 수 있기 때문이에요 (API 계층을 사용하지 않고 있다는 점을 기억하세요). <br/>
대신 서버에서 데이터를 가져와서 prop으로 컴포넌트에 전달할 수 있어요.

`/dashboard/invoices/page.tsx`에서 `fetchInvoicesPages`라는 새로운 함수를 가져와서 `searchParams`에서 쿼리를 인자로 전달하세요.

```javascript
// ...
import {fetchInvoicesPages} from '@/app/lib/data';

export default async function Page({
                                       searchParams,
                                   }: {
    searchParams?: {
        query?: string,
        page?: string,
    },
}) {
    const query = searchParams?.query || '';
    const currentPage = Number(searchParams?.page) || 1;

    const totalPages = await fetchInvoicesPages(query);

    return (
        // ...
    );
}
```

`fetchInvoicesPages` 함수는 검색 쿼리에 따라 총 페이지 수를 반환해요. <br/>
예를 들어, 검색 쿼리에 맞는 청구서가 12개 있고, 각 페이지에 6개의 청구서가 표시된다면, 총 페이지 수는 2가 될 거예요.

다음으로, `totalPages` prop을 `<Pagination/>` 컴포넌트에 전달하세요.

```javascript
// ...

export default async function Page({
                                       searchParams,
                                   }: {
    searchParams?: {
        query?: string;
        page?: string;
    };
}) {
    const query = searchParams?.query || '';
    const currentPage = Number(searchParams?.page) || 1;

    const totalPages = await fetchInvoicesPages(query);

    return (
        <div className="w-full">
            <div className="flex w-full items-center justify-between">
                <h1 className={`${lusitana.className} text-2xl`}>Invoices</h1>
            </div>
            <div className="mt-4 flex items-center justify-between gap-2 md:mt-8">
                <Search placeholder="Search invoices..."/>
                <CreateInvoice/>
            </div>
            <Suspense key={query + currentPage} fallback={<InvoicesTableSkeleton/>}>
                <Table query={query} currentPage={currentPage}/>
            </Suspense>
            <div className="mt-5 flex w-full justify-center">
                <Pagination totalPages={totalPages}/>
            </div>
        </div>
    );
}
```

`<Pagination/>` 컴포넌트로 이동하여 `usePathname`과 `useSearchParams` 훅을 임포트하세요. <br/>
이 훅들을 사용하여 현재 페이지를 가져오고 새로운 페이지를 설정할 거예요.<br/>
이 컴포넌트의 주석 처리된 코드를 해제하는 것도 잊지 마세요.<br/>
아직 `<Pagination/>` 로직을 구현하지 않았기 때문에 애플리케이션이 일시적으로 멈출 수 있어요. 이제 그 로직을 구현해보아요!

```javascript
'use client';

import {ArrowLeftIcon, ArrowRightIcon} from '@heroicons/react/24/outline';
import clsx from 'clsx';
import Link from 'next/link';
import {generatePagination} from '@/app/lib/utils';
import {usePathname, useSearchParams} from 'next/navigation';

export default function Pagination({totalPages}: { totalPages: number }) {
    const pathname = usePathname();
    const searchParams = useSearchParams();
    const currentPage = Number(searchParams.get('page')) || 1;

    // ...
}
```

`<Pagination>` 컴포넌트 안에 `createPageURL`이라는 새로운 함수를 만들어보세요. <br/>
검색 기능과 비슷하게, 새로운 페이지 번호를 설정하기 위해 `URLSearchParams`를 사용하고, `pathName`을 사용하여 URL 문자열을 생성할 거예요.

```javascript
'use client';

import {ArrowLeftIcon, ArrowRightIcon} from '@heroicons/react/24/outline';
import clsx from 'clsx';
import Link from 'next/link';
import {generatePagination} from '@/app/lib/utils';
import {usePathname, useSearchParams} from 'next/navigation';

export default function Pagination({totalPages}: { totalPages: number }) {
    const pathname = usePathname();
    const searchParams = useSearchParams();
    const currentPage = Number(searchParams.get('page')) || 1;

    const createPageURL = (pageNumber: number | string) => {
        const params = new URLSearchParams(searchParams);
        params.set('page', pageNumber.toString());
        return `${pathname}?${params.toString()}`;
    };

    // ...
}
```

여기서 무슨 일이 일어나고 있는지 설명해 줄게요:

- `createPageURL` 함수는 현재 검색 매개변수들의 인스턴스를 생성해요.
- 그런 다음, 주어진 페이지 번호로 "page" 매개변수를 업데이트해요.
- 마지막으로, `pathname`과 업데이트된 검색 매개변수를 사용해 전체 URL을 구성해요.

`<Pagination>` 컴포넌트의 나머지 부분은 스타일링과 첫 페이지, 마지막 페이지, 활성 상태, 비활성화된 상태 등 다양한 상태를 처리해요. <br />
이 과정에서는 자세하게 다루지 않지만, 코드가 어떻게 동작하는지 궁금하다면 살펴볼 수 있어요.

마지막으로, 사용자가 새로운 검색어를 입력할 때 페이지 번호를 1로 초기화하고 싶어요. <br/>
이 작업은 `<Search>` 컴포넌트에서 `handleSearch` 함수를 업데이트함으로써 할 수 있어요.

```javascript
'use client';

import {MagnifyingGlassIcon} from '@heroicons/react/24/outline';
import {usePathname, useRouter, useSearchParams} from 'next/navigation';
import {useDebouncedCallback} from 'use-debounce';

export default function Search({placeholder}: { placeholder: string }) {
    const searchParams = useSearchParams();
    const {replace} = useRouter();
    const pathname = usePathname();

    const handleSearch = useDebouncedCallback((term) => {
        const params = new URLSearchParams(searchParams);
        params.set('page', '1');
        if (term) {
            params.set('query', term);
        } else {
            params.delete('query');
        }
        replace(`${pathname}?${params.toString()}`);
    }, 300);

```

## 요약

축하해요! 이제 검색과 페이지네이션 기능을 URL 매개변수와 Next.js API를 사용해서 구현했어요.

이번 장에서 배운 내용을 요약해 보면:

- 클라이언트 상태 대신 URL 검색 매개변수를 사용해 검색과 페이지네이션을 처리했어요.
- 데이터를 서버에서 가져왔어요.
- 더 부드러운 클라이언트 사이드 전환을 위해 `useRouter` 후크를 사용했어요.

이 패턴들은 클라이언트 사이드 React에서 작업할 때 익숙할 수 있는 방식과는 다르지만, URL 검색 매개변수를 사용하고 상태를 서버로 올리는 것의 장점을 이제 더 잘 이해하게 되었길 바래요.
