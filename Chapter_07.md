이제 데이터베이스를 생성하고 데이터를 넣었으니, 애플리케이션에 데이터를 가져오는 여러 가지 방법에 대해 이야기해 보겠습니다. <br/>
그리고 대시보드 overview 페이지를 만들어 볼 거예요.

## 이번 챕터에서는...

- 데이터 가져오는 방법에 대해 배워요 : API, ORM, SQL 등
- 서버 컴포넌트가 백엔드 지원에 더 안전하게 접근하는 데 어떻게 도움이 되는지 알아봐요.
- 네트워크 waterfall 이 무엇인지 배워요.
- JavaScript pattern 을 사용해 병렬 fetching 을 구현하는 방법을 배워요.

<hr/>

## 데이터 가져오는 방법 선택하기

### API 레이어

API 는 애플리케이션 코드와 데이터베이스 사이의 중간 레이어예요. <br/>
API 를 사용할 수 있는 몇 가지 경우는 다음과 같아요 :

- 3rd party 서비스에서 제공하는 API 를 사용할 때
- 클라이언트에서 데이터를 가져올 때, 데이터베이스 secret 을 클라이언트에 노출하지 않기 위해 서버에서 실행되는 API 레이어가 필요해요. <br/>
  Next.js 에서는 [route handler](https://nextjs.org/docs/app/building-your-application/routing/route-handlers) 를 사용해 API
  엔드포인트를 만들 수 있어요.

### 데이터베이스 쿼리

풀스택 어플리케이션을 만들 때 데이터베이스와 상호작용하는 로직을 작성해야 해요. <br/>
Postgres 와 같은 [관계형 데이터베이스](https://aws.amazon.com/ko/relational-database/)에서는 SQL
또는 [ORM](https://vercel.com/docs/storage/vercel-postgres/using-an-orm) 을 사용해 이 작업을 할 수 있어요.

데이터베이스 쿼리를 작성해야 하는 경우는 다음과 같아요 :

- API 엔드포인트를 생성할 때 데이터베이스와 상호작용하는 로직을 작성해야 해요.
- 리액트 서버 컴포넌트를 사용하는 경우 (서버에서 데이터를 가져올 때) API 레이어를 생략하고 데이터베이스에 직접 쿼리할 수 있어요. <br/>
  이렇게 하면 데이터베이스 비밀이 클라이언트에 노출되는 위험을 피할 수 있어요.

⚠️ 클라이언트에서 직접 데이터베이스에 접근하는 것은 데이터베이스 secret 이 노출될 위험이 있기 때문에 하면 안돼요 !!!

리액트 서버 컴포넌트에 대해 더 알아보아요.

### 서버 컴포넌트를 사용하여 데이터 가져오기

기본적으로 Next.js 애플리케이션은 리액트 서버 컴포넌트를 사용해요. <br/>
서버 컴포넌트를 사용하여 데이터를 가져오는 것은 비교적 새로운 접근 방식이에요.

장점

- 서버 컴포넌트는 Promise 를 지원해요. <br/>
  이는 fetch 와 같은 비동기 작업을 더 간단하게 처리할 수 있도록 도와줘요.<br/>
  `useEffect`, `useState` 또는 fetch 라이브러리를 사용하지 않고도 `async/await` 문법을 사용할 수 있어요.
- 서버 컴포넌트는 서버에서 실행되므로, 비용이 많이 드는 fetch 와 로직을 서버에 두고 결과만 클라이언트에 전송할 수 있어요.
- 앞서 언급했듯, 서버 컴포넌트는 서버에서 실행되기 때문에 추가적인 API 레이어 없이 데이터베이스에 직접 쿼리할 수 있어요.

### SQL 사용하기

대시보드 프로젝트를 위해 Vercel Postgres SDK 와 SQL 을 사용하여 데이터베이스 쿼리를 작성할 거예요.

SQL 을 사용하는 이유:

- SQL 은 관계형 데이터베이스를 쿼리하는 산업 표준이에요 (ORM은 내부적으로 SQL을 생성해요).
- SQL에 대한 기본적인 이해는 관계형 데이터베이스의 기초를 이해하는 데 도움을 주며, 이 지식을 다른 도구에 적용할 수 있게 해줘요.
- SQL은 다목적이어서 특정 데이터를 가져오고 조작할 수 있어요.
- Vercel Postgres SDK는 주입 공격에 대한 보호 기능을 제공해요.

SQL 을 사용해본 적 없다면 걱정하지 마세요. 필요한 쿼리를 미리 제공해드릴 거예요.

`/app/lib/data.ts` 로 가보세요. 여기에서 `@vercel/postgres` 에서 `sql` 함수를 가져오는 것을 볼 수 있어요. <br/>
이 함수는 데이터베이스를 쿼리하는 데 사용돼요.

```javascript
import {sql} from '@vercel/postgres';

// ...
```

서버 컴포넌트 안에서는 `sql` 을 호출할 수 있어요. <br/>
하지만 컴포넌트를 더 쉽게 탐색할 수 있도록, 모든 데이터 쿼리를 `data.ts` 파일에 모아두었고, 이를 컴포넌트에 임포트하여 사용할 수 있도록 했어요.

> 참고: 6장에서 자신의 데이터베이스 제공자를 사용한 경우, 제공자와 호환되도록 데이터베이스 쿼리를 업데이트해야 해요.
> 쿼리는 `/app/lib/data.ts`에서 찾을 수 있어요.

<hr/>

## 대시보드 overview 페이지의 데이터 가져오기

이제 데이터를 가져오는 방법을 이해했으니, 대시보드 overview 페이지에서 데이터를 가져와 볼 거예요. <br/>
`/app/dashboard/page.tsx`로 이동해 다음 코드를 붙여넣고 살펴보아요.

```javascript
import {Card} from '@/app/ui/dashboard/cards';
import RevenueChart from '@/app/ui/dashboard/revenue-chart';
import LatestInvoices from '@/app/ui/dashboard/latest-invoices';
import {lusitana} from '@/app/ui/fonts';

export default async function Page() {
    return (
        <main>
            <h1 className={`${lusitana.className} mb-4 text-xl md:text-2xl`}>
                Dashboard
            </h1>
            <div className="grid gap-6 sm:grid-cols-2 lg:grid-cols-4">
                {/* <Card title="Collected" value={totalPaidInvoices} type="collected" /> */}
                {/* <Card title="Pending" value={totalPendingInvoices} type="pending" /> */}
                {/* <Card title="Total Invoices" value={numberOfInvoices} type="invoices" /> */}
                {/* <Card
          title="Total Customers"
          value={numberOfCustomers}
          type="customers"
        /> */}
            </div>
            <div className="mt-6 grid grid-cols-1 gap-6 md:grid-cols-4 lg:grid-cols-8">
                {/* <RevenueChart revenue={revenue}  /> */}
                {/* <LatestInvoices latestInvoices={latestInvoices} /> */}
            </div>
        </main>
    );
}
```

위 코드에서는:

- `Page`는 비동기 컴포넌트입니다. 이로 인해 `await`을 사용하여 데이터를 가져올 수 있어요.
- 코드에는 데이터를 받는 세 가지 컴포넌트, `<Card>`, `<RevenueChart>`, 그리고 `<LatestInvoices>`가 있어요. <br/>
  현재는 애플리케이션 오류를 방지하기 위해 주석 처리되어 있어요.

<hr/>

## `<RevenueChart />`의 데이터 가져오기

`<RevenueChart />` 컴포넌트에서 데이터를 가져오기 위해, `data.ts` 파일에서 `fetchRevenue` 함수를 가져와서 컴포넌트 안에서 호출하세요:

```javascript
import {Card} from '@/app/ui/dashboard/cards';
import RevenueChart from '@/app/ui/dashboard/revenue-chart';
import LatestInvoices from '@/app/ui/dashboard/latest-invoices';
import {lusitana} from '@/app/ui/fonts';
import {fetchRevenue} from '@/app/lib/data';

export default async function Page() {
    const revenue = await fetchRevenue();
    // ...
}
```

그런 다음, `<RevenueChart/>` 컴포넌트의 주석을 해제하고, 컴포넌트 파일(`/app/ui/dashboard/revenue-chart.tsx`)로 이동하여 내부의 코드 주석도 해제하세요. <br />
로컬호스트에서 확인해 보세요. 이제 `revenue data`를 사용하는 차트를 볼 수 있을 거예요!

<img src="https://nextjs.org/_next/image?url=%2Flearn%2Fdark%2Frecent-revenue.png&w=1920&q=75" alt="revenue data chart" />

그럼 데이터를 더 가져오기 위한 쿼리를 계속해서 불러와 볼까요!

<hr/>

## `<LatestInvoices />`의 데이터 가져오기

`<LatestInvoices />` 컴포넌트를 위해 최신 5개의 청구서를 가져와야 해요. <br/>

모든 데이터를 가져온 후 자바스크립트를 사용하여 정렬할 수도 있지만, 데이터가 많아질수록 이 방법은 성능에 영향을 줄 수 있어요. <br/>
많은 양의 데이터를 전송하고 이를 처리하기 위해 추가적인 자바스크립트가 필요해질 수 있기 때문입니다.

대신, 메모리에서 정렬하는 대신 SQL 쿼리를 사용하여 최신 5개의 데이터만 가져올 수 있습니다.<br/>
예를 들어, 아래는 `data.ts` 파일에서 사용된 SQL 쿼리입니다.

```javascript
// Fetch the last 5 invoices, sorted by date
const data = await sql < LatestInvoiceRaw > `
  SELECT invoices.amount, customers.name, customers.image_url, customers.email
  FROM invoices
  JOIN customers ON invoices.customer_id = customers.id
  ORDER BY invoices.date DESC
  LIMIT 5`;
```

페이지에서 `fetchLatestInvoices` 함수를 가져오세요:

```javascript
import {Card} from '@/app/ui/dashboard/cards';
import RevenueChart from '@/app/ui/dashboard/revenue-chart';
import LatestInvoices from '@/app/ui/dashboard/latest-invoices';
import {lusitana} from '@/app/ui/fonts';
import {fetchRevenue, fetchLatestInvoices} from '@/app/lib/data';

export default async function Page() {
    const revenue = await fetchRevenue();
    const latestInvoices = await fetchLatestInvoices();
    // ...
}
```

그 다음, `<LatestInvoices />` 컴포넌트를 주석 해제하세요. <br/>
또 `/app/ui/dashboard/latest-invoices`에 있는 `<LatestInvoices />` 컴포넌트 안의 관련 코드도 주석 해제해야 해요.

로컬 서버에 접속하면, 데이터베이스에서 마지막 5개의 인보이스만 반환되는 것을 확인할 수 있어요.<br/>
이제 데이터베이스를 직접 쿼리하는 장점을 조금씩 느끼실 수 있을 거예요!

<img src="https://nextjs.org/_next/image?url=%2Flearn%2Fdark%2Flatest-invoices.png&w=1920&q=75" alt="latest invoices" />

<hr/>

## 실습 : `Card` 컴포넌트 데이터 가져오기

이제는 <Card> 컴포넌트에 데이터를 가져오는 연습을 해볼까요?<br/>
이 카드들은 다음 데이터를 보여줄 예정입니다:

- 징수된 인보이스의 총 금액
- 대기 중인 인보이스의 총 금액
- 인보이스의 총 개수
- 고객의 총 수

또 다시 모든 인보이스와 고객 데이터를 가져와서 자바스크립트로 데이터를 처리하고 싶을 수도 있어요.<br/>
예를 들어, 총 인보이스 개수와 고객 수를 구하기 위해 `Array.length`를 사용할 수 있을 거예요.

```javascript
const totalInvoices = allInvoices.length;
const totalCustomers = allCustomers.length;
```

하지만 SQL을 사용하면 필요한 데이터만 가져올 수 있어요. <br/>
`Array.length`를 사용하는 것보다 조금 더 길게 작성해야 할 수 있지만, 이렇게 하면 요청 중에 전송되는 데이터의 양이 줄어들어요.<br/>
SQL로 대체할 수 있는 방법은 다음과 같아요.

```javascript
const invoiceCountPromise = sql`SELECT COUNT(*) FROM invoices`;
const customerCountPromise = sql`SELECT COUNT(*) FROM customers`;
```

필요한 함수는 `fetchCardData`라고 불려요. <br/>
이 함수를 가져올 때 반환된 값을 **디스트럭처링**해서 사용할 필요가 있어요.

> 힌트:
> - 카드 컴포넌트들이 어떤 데이터를 필요로 하는지 확인하세요.
> - `data.ts` 파일을 살펴보고 함수가 반환하는 값을 확인하세요.

<details>
<summary>the solution</summary>

```javascript
import {Card} from '@/app/ui/dashboard/cards';
import RevenueChart from '@/app/ui/dashboard/revenue-chart';
import LatestInvoices from '@/app/ui/dashboard/latest-invoices';
import {lusitana} from '@/app/ui/fonts';
import {
    fetchRevenue,
    fetchLatestInvoices,
    fetchCardData,
} from '@/app/lib/data';

export default async function Page() {
    const revenue = await fetchRevenue();
    const latestInvoices = await fetchLatestInvoices();
    const {
        numberOfInvoices,
        numberOfCustomers,
        totalPaidInvoices,
        totalPendingInvoices,
    } = await fetchCardData();

    return (
        <main>
            <h1 className={`${lusitana.className} mb-4 text-xl md:text-2xl`}>
                Dashboard
            </h1>
            <div className="grid gap-6 sm:grid-cols-2 lg:grid-cols-4">
                <Card title="Collected" value={totalPaidInvoices} type="collected"/>
                <Card title="Pending" value={totalPendingInvoices} type="pending"/>
                <Card title="Total Invoices" value={numberOfInvoices} type="invoices"/>
                <Card
                    title="Total Customers"
                    value={numberOfCustomers}
                    type="customers"
                />
            </div>
            <div className="mt-6 grid grid-cols-1 gap-6 md:grid-cols-4 lg:grid-cols-8">
                <RevenueChart revenue={revenue}/>
                <LatestInvoices latestInvoices={latestInvoices}/>
            </div>
        </main>
    );
}
```

</details>

잘 하셨어요!👍 <br/>
이제 대시보드 개요 페이지에 필요한 모든 데이터를 가져왔습니다. <br/>
여러분의 페이지는 이렇게 보여야 해요:

<img src="https://nextjs.org/_next/image?url=%2Flearn%2Fdark%2Fcomplete-dashboard.png&w=1920&q=75&dpl=dpl_4kWRRdpV5mEWMp9Zhahs8vP5fgBq" alt="dashboard preview" />

하지만... 두 가지 주의해야 할 점이 있어요: ⚠️😭

1. 데이터 요청들이 서로 의도치 않게 차단되고 있어, **요청 폭포 현상(request waterfall)** 이 발생하고 있어요.
2. 기본적으로 Next.js는 성능을 향상시키기 위해 경로를 미리 렌더링하는데, 이를 정적 렌더링이라고 해요. 그래서 데이터가 변경되면 대시보드에 반영되지 않아요.

이번 chapter 에서는 첫 번째 문제에 대해 논의하고, 다음 chapter 에서는 두 번째 문제에 대해 자세히 살펴볼 거예요.

<hr />

## request waterfall 란?

"waterfall"라는 것은 서로 의존하는 네트워크 요청의 순서를 말해요. <br/>
데이터 요청의 경우, 각 요청은 이전 요청이 데이터를 반환한 후에야 시작될 수 있어요.<br/>
즉, 첫 번째 요청이 끝나야 그 다음 요청이 시작되는 거죠. <br/>
이렇게 되면 요청이 순차적으로 진행되어 전체 처리 시간이 길어질 수 있어요.

간단히 말해서, `request waterfall`는 여러 개의 요청이 마치 폭포처럼 흐르는 것처럼 보이며, 이전 요청이 완료되어야만 다음 요청이 진행되는 구조를 가지고 있어요.

<img src="https://nextjs.org/_next/image?url=%2Flearn%2Fdark%2Fsequential-parallel-data-fetching.png&w=3840&q=75&dpl=dpl_4kWRRdpV5mEWMp9Zhahs8vP5fgBq" alt="request waterfall example image" />

예를 들어, `fetchRevenue()`가 실행되는 것을 기다려야만 `fetchLatestInvoices()`가 실행될 수 있어요.<br/>
이런 식으로 요청이 순차적으로 진행되기 때문에 전체적인 데이터 요청 시간이 길어질 수 있답니다.

```javascript
const revenue = await fetchRevenue();
const latestInvoices = await fetchLatestInvoices(); // wait for fetchRevenue() to finish
const {
    numberOfInvoices,
    numberOfCustomers,
    totalPaidInvoices,
    totalPendingInvoices,
} = await fetchCardData(); // wait for fetchLatestInvoices() to finish
```

이런 패턴이 항상 나쁜 것은 아니에요. <br/>
때로는 다음 요청을 하기 전에 조건이 충족되기를 원할 수도 있기 때문이에요. <br/>

예를 들어, 먼저 사용자의 ID와 프로필 정보를 가져오고 싶을 수 있어요. <br/>
ID를 얻은 후에 그 사용자의 친구 목록을 가져올 수 있죠. <br/>
이 경우에는 각 요청이 이전 요청에서 반환된 데이터에 따라 결정되는 것이랍니다.

하지만 이런 방식이 의도치 않게 발생하면 성능에 영향을 미칠 수 있어요.<br/>
즉, 모든 요청이 이전 요청이 끝난 후에야 시작되기 때문에 시간이 오래 걸릴 수 있다는 점을 주의해야 해요.

<hr/>

## 병렬 데이터 가져오기

`waterfall`을 피하는 일반적인 방법은 모든 데이터 요청을 동시에, 즉 병렬로 시작하는 것입니다.

`JavaScript`에서는 `Promise.all()` 또는 `Promise.allSettled()` 함수를 사용하여 모든 프로미스를 동시에 시작할 수 있어요.<br/>
예를 들어, `data.ts` 파일에서는 `fetchCardData()` 함수에서 `Promise.all()`을 사용하고 있답니다.

이렇게 하면 여러 요청을 동시에 진행할 수 있어요.<br/>
즉, 각각의 요청이 서로의 결과를 기다리지 않고 동시에 실행되므로, 페이지의 응답 속도가 빨라지는 효과가 있어요.

이 방식을 활용하면 데이터 요청의 성능을 크게 개선할 수 있습니다!

```javascript
export async function fetchCardData() {
    try {
        const invoiceCountPromise = sql`SELECT COUNT(*) FROM invoices`;
        const customerCountPromise = sql`SELECT COUNT(*) FROM customers`;
        const invoiceStatusPromise = sql`SELECT
         SUM(CASE WHEN status = 'paid' THEN amount ELSE 0 END) AS "paid",
         SUM(CASE WHEN status = 'pending' THEN amount ELSE 0 END) AS "pending"
         FROM invoices`;

        const data = await Promise.all([
            invoiceCountPromise,
            customerCountPromise,
            invoiceStatusPromise,
        ]);
        // ...
    }
}
```

이 패턴을 사용하면 다음과 같은 장점이 있어요:

- 모든 데이터 가져오기를 동시에 실행할 수 있어 성능이 향상될 수 있어요.
- 어떤 라이브러리나 프레임워크에도 적용할 수 있는 `JavaScript`의 기본 패턴을 사용할 수 있어요.

하지만 이 JavaScript 패턴에만 의존할 경우 단점이 하나 있어요. <br/>
만약 한 데이터 요청이 다른 요청보다 느리면 어떻게 될까요?

이런 상황에서는 페이지가 그 느린 요청이 완료될 때까지 기다려야 하기 때문에, 전체 응답 속도가 느려질 수 있어요.<br/>
따라서 병렬 데이터 요청을 사용할 때는 모든 요청이 동시에 끝나는 것이 아니라, 가장 느린 요청에 따라 전체 데이터 가져오기 시간이 결정될 수 있다는 점을 유의해야 해요.
