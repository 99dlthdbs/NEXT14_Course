이전 장에서는 Next.js의 다양한 렌더링 방법에 대해 배웠어요. <br/>
또한, 느린 데이터 요청이 애플리케이션의 성능에 어떤 영향을 미칠 수 있는지도 이야기했어요. <br/>
이제 느린 데이터 요청이 있을 때 사용자 경험을 개선할 수 있는 방법을 살펴볼 거예요.

## 이번 챕터에서는...

- streaming 이란 무엇이고 언제 사용할 수 있는지
- `loading.tsx`와 `Suspense`를 이용한 스트리밍 구현 방법
- loading skeletons 가 무엇인지
- route group 이란 무엇이고 언제 사용할 수 있는지
- 애플리케이션에서 Suspense boundaries 를 어디에 배치하는지

<hr />

## streaming 이란?

스트리밍은 데이터를 전송하는 기술로, 하나의 경로를 더 작은 "조각"으로 나누고 이 조각들을 서버에서 클라이언트로 준비되는 대로 점진적으로 보내는 방법이에요. <br />
이를 통해 사용자는 전체 데이터를 기다리지 않고도 빠르게 콘텐츠를 볼 수 있게 됩니다.

<img src="https://nextjs.org/_next/image?url=%2Flearn%2Fdark%2Fserver-rendering-with-streaming.png&w=3840&q=75&dpl=dpl_4kWRRdpV5mEWMp9Zhahs8vP5fgBq" alt="streaming" />

스트리밍을 사용하면 느린 데이터 요청으로 인해 전체 페이지가 막히는 것을 방지할 수 있어요. <br />
이렇게 하면 사용자는 모든 데이터가 로드될 때까지 기다리지 않고도 페이지의 일부를 보고 상호작용할 수 있게 됩니다. <br/>
즉, 사용자는 필요한 정보를 빠르게 확인하고 사용할 수 있는 장점이 있어요.

<img src="https://nextjs.org/_next/image?url=%2Flearn%2Fdark%2Fserver-rendering-with-streaming-chart.png&w=3840&q=75&dpl=dpl_4kWRRdpV5mEWMp9Zhahs8vP5fgBq" alt="streaming works" />

스트리밍은 리액트의 컴포넌트 모델과 잘 작동해요. 각 컴포넌트를 하나의 *chunk*로 간주할 수 있기 때문이에요.

Next.js에서 스트리밍을 구현하는 방법은 두 가지가 있어요:

1. **페이지 레벨에서**: `loading.tsx` 파일을 사용해 구현할 수 있어요.
2. **특정 컴포넌트에서**: `<Suspense>`를 사용해 구현할 수 있어요.

이제 이것이 어떻게 작동하는지 살펴보아요.

<hr />

## 전체 페이지에서 스트리밍 하기 with `loading.tsx`

`/app/dashboard` 폴더 안에, `loading.tsx` 파일을 만들어봅시다!

```javascript
export default function Loading() {
    return <div>Loading...</div>;
}
```

http://localhost:3000/dashboard 를 새로고침하면 아래의 화면을 볼 수 있어요.

<img src="https://nextjs.org/_next/image?url=%2Flearn%2Fdark%2Floading-page.png&w=1920&q=75&dpl=dpl_4kWRRdpV5mEWMp9Zhahs8vP5fgBq" alt="loading page" />

여기에서 몇 가지 일이 일어나고 있어요:

1. `loading.tsx`는 특별한 Next.js 파일로, Suspense 위에 구축된 것이에요. <br/>
   이 파일을 사용하면 페이지 콘텐츠가 로드되는 동안 보여줄 대체 UI를 만들 수 있어요.
2. `<SideNav>`는 정적이기 때문에 즉시 보여줘요. <br/>
   사용자는 동적 콘텐츠가 로드되는 동안 `<SideNav>`와 상호작용할 수 있어요.
3. 사용자는 페이지 로딩이 완료될 때까지 기다리지 않고도 다른 페이지로 이동할 수 있어요. <br/>
   이것을 **중단 가능한 내비게이션(interruptable navigation)** 이라고 해요.

축하해요! 여러분은 스트리밍을 성공적으로 구현했어요. <br/>
하지만 사용자 경험을 더 개선할 수 있는 방법이 있어요. <br/>
**"Loading…"** 텍스트 대신 로딩 스켈레톤을 보여주도록 해요.

### 로딩 스켈레톤 추가하기

로딩 스켈레톤은 UI의 간소화된 버전이에요. <br/>
많은 웹사이트에서 콘텐츠가 로드되고 있음을 사용자에게 알리기 위해 placeholder(또는 대체 UI(fallback))로 사용해요. <br/>
`loading.tsx` 파일에 추가하는 UI는 정적 파일의 일부로 포함되어 먼저 전송돼요. <br/>
그 후 나머지 동적 콘텐츠가 서버에서 클라이언트로 스트리밍돼요.

`loading.tsx` 파일 안에 `<DashboardSkeleton>`이라는 새 컴포넌트를 가져와야 해요.

```javascript
import DashboardSkeleton from '@/app/ui/skeletons';

export default function Loading() {
    return <DashboardSkeleton/>;
}
```

http://localhost:3000/dashboard 를 새로고침하면 아래의 화면을 볼 수 있어요.

<img src="https://nextjs.org/_next/image?url=%2Flearn%2Fdark%2Floading-page-with-skeleton.png&w=1920&q=75&dpl=dpl_4kWRRdpV5mEWMp9Zhahs8vP5fgBq" alt="skeleton display" />

### 라우트 그룹을 사용하여 로딩 스켈레톤 버그 수정하기

현재 로딩 스켈레톤은 인보이스와 고객 페이지에도 적용되고 있어요. <br />
`loading.tsx`가 파일 시스템에서 `/invoices/page.tsx`와 `/customers/page.tsx`보다 한 단계 높은 위치에 있기 때문에, 이러한 페이지에도 적용되는 거예요.

이 문제를 해결하기 위해 라우트 그룹을 사용할 수 있어요. <br />
dashboard 폴더 안에 `/(overview)`라는 새 폴더를 만들어야 해요. <br />
그런 다음 `loading.tsx`와 `page.tsx` 파일을 이 폴더로 이동시키면 됩니다.

<img src="https://nextjs.org/_next/image?url=%2Flearn%2Fdark%2Froute-group.png&w=3840&q=75&dpl=dpl_4kWRRdpV5mEWMp9Zhahs8vP5fgBq" alt="dashboard folder structure" />



이제 `loading.tsx` 파일은 대시보드 개요 페이지에만 적용될 거예요.

라우트 그룹은 URL 경로 구조에 영향을 주지 않으면서 파일을 논리적인 그룹으로 나눌 수 있게 해줘요. <br />
괄호 `()`를 사용해 새 폴더를 만들면, 해당 폴더 이름은 URL 경로에 포함되지 않아요. <br/>
예를 들어 `/dashboard/(overview)/page.tsx`는 `/dashboard`로 변하게 되는 거죠.

여기서는 로딩 파일을 대시보드 개요 페이지에만 적용하기 위해 라우트 그룹을 사용하고 있지만, 더 큰 애플리케이션에서는 (marketing), (shop)과 같은 경로 그룹을 만들어 팀별로 분리해 관리할 수도 있어요.

### 컴포넌트 스트리밍

지금까지는 전체 페이지를 스트리밍했어요. <br />
하지만 `React`의 `Suspense`를 사용하면 특정 컴포넌트만 스트리밍할 수도 있어요.

`Suspense`는 애플리케이션의 일부가 로드될 때까지 렌더링을 미루는 기능이에요. <br />
동적인 컴포넌트를 `Suspense`로 감싸고, 해당 컴포넌트가 로드되는 동안 보여줄 대체 컴포넌트를 설정할 수 있어요.

이전에 언급한 `fetchRevenue()` 함수가 페이지 전체 로딩을 느리게 했던 것을 기억하나요? <br/>
이 요청 때문에 페이지 전체가 막히는 대신, 이제 `Suspense`를 사용해 이 컴포넌트만 스트리밍하고 나머지 페이지 UI는 즉시 표시할 수 있어요.

그렇게 하려면, 데이터를 컴포넌트로 옮기고, 코드를 업데이트해야 해요. <br />
`/dashboard/(overview)/page.tsx` 파일에서 `fetchRevenue()`와 관련된 코드를 모두 삭제해주세요.

```javascript
import {Card} from '@/app/ui/dashboard/cards';
import RevenueChart from '@/app/ui/dashboard/revenue-chart';
import LatestInvoices from '@/app/ui/dashboard/latest-invoices';
import {lusitana} from '@/app/ui/fonts';
import {fetchLatestInvoices, fetchCardData} from '@/app/lib/data'; // remove fetchRevenue

export default async function Page() {
    const revenue = await fetchRevenue() // delete this line
    const latestInvoices = await fetchLatestInvoices();
    const {
        numberOfInvoices,
        numberOfCustomers,
        totalPaidInvoices,
        totalPendingInvoices,
    } = await fetchCardData();

    return (
        // ...
    );
}
```

그 다음으로, `React`에서 `<Suspense>`를 가져와서 `<RevenueChart />` 컴포넌트를 감싸주세요. <br/>
그리고 로딩 중에 보여줄 대체 컴포넌트로 `<RevenueChartSkeleton>`을 전달할 수 있어요.

```javascript
import {Card} from '@/app/ui/dashboard/cards';
import RevenueChart from '@/app/ui/dashboard/revenue-chart';
import LatestInvoices from '@/app/ui/dashboard/latest-invoices';
import {lusitana} from '@/app/ui/fonts';
import {fetchLatestInvoices, fetchCardData} from '@/app/lib/data';
import {Suspense} from 'react';
import {RevenueChartSkeleton} from '@/app/ui/skeletons';

export default async function Page() {
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
                <Suspense fallback={<RevenueChartSkeleton/>}>
                    <RevenueChart/>
                </Suspense>
                <LatestInvoices latestInvoices={latestInvoices}/>
            </div>
        </main>
    );
}
```

마지막으로, `<RevenueChart>` 컴포넌트를 업데이트하여 데이터를 자체적으로 가져오도록 하고, 이전에 전달되던 prop을 제거해주세요.

```javascript
import {generateYAxis} from '@/app/lib/utils';
import {CalendarIcon} from '@heroicons/react/24/outline';
import {lusitana} from '@/app/ui/fonts';
import {fetchRevenue} from '@/app/lib/data';

// ...

export default async function RevenueChart() { // Make component async, remove the props
    const revenue = await fetchRevenue(); // Fetch data inside the component

    const chartHeight = 350;
    const {yAxisLabels, topLabel} = generateYAxis(revenue);

    if (!revenue || revenue.length === 0) {
        return <p className="mt-4 text-gray-400">No data available.</p>;
    }

    return (
        // ...
    );
}

```

이제 페이지를 새로고침하면 대시보드 정보가 거의 즉시 표시되고, `<RevenueChart>`에 대한 데이터를 불러오는 동안 대체 스켈레톤이 표시될 거예요.

<img src="https://nextjs.org/_next/image?url=%2Flearn%2Fdark%2Floading-revenue-chart.png&w=1920&q=75&dpl=dpl_4kWRRdpV5mEWMp9Zhahs8vP5fgBq" alt="revenueChart suspense" />

### 실습: `<LatestInvoices>` 스트리밍하기

이제 여러분 차례예요! 방금 배운 내용을 연습하며 `<LatestInvoices>` 컴포넌트를 스트리밍해보세요.

`fetchLatestInvoices()` 함수를 페이지에서 `<LatestInvoices>` 컴포넌트로 옮기고, 이 컴포넌트를 `<Suspense>` 경계 안에 감싸면서 대체 컴포넌트로
`<LatestInvoicesSkeleton>`을 설정해보세요.

준비가 되면, 아래의 토글을 확장하여 해결 코드 확인할 수 있어요.

<details>
<summary>the solution</summary>

```javascript
import {Card} from '@/app/ui/dashboard/cards';
import RevenueChart from '@/app/ui/dashboard/revenue-chart';
import LatestInvoices from '@/app/ui/dashboard/latest-invoices';
import {lusitana} from '@/app/ui/fonts';
import {fetchCardData} from '@/app/lib/data'; // Remove fetchLatestInvoices
import {Suspense} from 'react';
import {
    RevenueChartSkeleton,
    LatestInvoicesSkeleton,
} from '@/app/ui/skeletons';

export default async function Page() {
    // Remove `const latestInvoices = await fetchLatestInvoices()`
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
                <Suspense fallback={<RevenueChartSkeleton/>}>
                    <RevenueChart/>
                </Suspense>
                <Suspense fallback={<LatestInvoicesSkeleton/>}>
                    <LatestInvoices/>
                </Suspense>
            </div>
        </main>
    );
}
```

</details>

<hr />

## Grouping components

이제 거의 다 왔어요!

이제 `<Card>` 컴포넌트를 `<Suspense>`로 감싸야 해요. <br />
각 카드를 개별적으로 데이터를 받아오도록 만들 수 있지만, 이렇게 하면 카드들이 하나씩 불쑥 나타나는 '팝업' 효과가 생길 수 있어요. <br />
이런 시각적인 효과는 사용자에게 불편하게 느껴질 수 있어요.

그렇다면 이 문제를 어떻게 해결할 수 있을까요?

좀 더 자연스러운 순차적인 효과를 만들기 위해 카드를 그룹화할 수 있어요. <br/>
즉, 카드들을 감싸는 컴포넌트를 만들고, 이 컴포넌트를 통해 첫 번째로 고정된 `<SideNav />`가 먼저 보여지고, 그 다음에 카드들이 나타나도록 할 수 있어요.

`page.tsx` 파일에서 다음 작업을 진행해 주세요:

1. `<Card>` 컴포넌트들을 삭제하세요.
2. `fetchCardData()` 함수를 삭제하세요.
3. 새로운 래퍼 컴포넌트 `<CardWrapper />`를 import하세요.
4. 새로운 스켈레톤 컴포넌트 `<CardsSkeleton />`을 import하세요.
5. `<CardWrapper />`를 `<Suspense>`로 감싸세요.

이렇게 하면 자연스럽게 카드들이 차례대로 로드되면서 UI가 사용자에게 더 부드럽게 나타날 거예요.

```javascript
import CardWrapper from '@/app/ui/dashboard/cards';
// ...
import {
    RevenueChartSkeleton,
    LatestInvoicesSkeleton,
    CardsSkeleton,
} from '@/app/ui/skeletons';

export default async function Page() {
    return (
        <main>
            <h1 className={`${lusitana.className} mb-4 text-xl md:text-2xl`}>
                Dashboard
            </h1>
            <div className="grid gap-6 sm:grid-cols-2 lg:grid-cols-4">
                <Suspense fallback={<CardsSkeleton/>}>
                    <CardWrapper/>
                </Suspense>
            </div>
            // ...
        </main>
    );
}
```

이제 `/app/ui/dashboard/cards.tsx` 파일로 이동하세요.

`fetchCardData()` 함수를 import한 후, `<CardWrapper />` 컴포넌트 안에서 이 함수를 호출해 주세요. <br/>
또한, 이 컴포넌트 내에서 필요한 주석 처리된 코드를 반드시 주석 해제해 주세요.

```javascript
// ...
import {fetchCardData} from '@/app/lib/data';

// ...

export default async function CardWrapper() {
    const {
        numberOfInvoices,
        numberOfCustomers,
        totalPaidInvoices,
        totalPendingInvoices,
    } = await fetchCardData();

    return (
        <>
            <Card title="Collected" value={totalPaidInvoices} type="collected"/>
            <Card title="Pending" value={totalPendingInvoices} type="pending"/>
            <Card title="Total Invoices" value={numberOfInvoices} type="invoices"/>
            <Card
                title="Total Customers"
                value={numberOfCustomers}
                type="customers"
            />
        </>
    );
}
```

페이지를 새로고침하면, 모든 카드가 동시에 로드되는 것을 볼 수 있을 거예요. <br/>
여러 개의 컴포넌트를 동시에 로드하고 싶을 때 이 패턴을 사용할 수 있어요.

<hr />

## 어디에 Suspense boundaries 를 두어야 할지 결정하기

어디에 **Suspense** boundaries를 두어야 할지는 몇 가지 요소에 따라 달라져요:

1. **사용자가 페이지를 스트리밍하는 동안 어떻게 경험하길 원하는지**
2. **어떤 콘텐츠를 우선적으로 보여주고 싶은지**
3. **컴포넌트가 데이터 패칭에 의존하는지**

대시보드 페이지를 살펴보면서 다르게 했을 부분이 있는지 생각해 보세요. 걱정하지 마세요, 정답은 없답니다.

- **전체 페이지를 스트리밍**할 수 있어요, 하지만 한 컴포넌트가 느리게 로드되면 전체 페이지 로딩 시간이 길어질 수 있어요.
- **각 컴포넌트를 개별적으로 스트리밍**할 수도 있어요, 하지만 이 경우 준비된 컴포넌트가 화면에 팝업처럼 나타나서 사용자에게 어색할 수 있어요.
- **페이지 섹션별로 스트리밍**하여 단계적인 효과를 낼 수 있어요, 하지만 이를 위해서는 래퍼 컴포넌트를 만들어야 해요.

앱에 따라 **Suspense boundaries 를 두는 위치**는 달라질 거예요.<br/>
일반적으로는 데이터 패칭이 필요한 컴포넌트에 직접 내려서 그 컴포넌트를 **Suspense**로 감싸는 것이 좋아요.<br/>
하지만 필요한 경우, 섹션이나 전체 페이지를 스트리밍하는 것도 나쁘지 않아요.

**Suspense**는 사용자 경험을 향상시킬 수 있는 강력한 API니까, 여러 방법을 시도해 보고 어떤 것이 가장 잘 맞는지 찾아보세요!

<hr/>

## 앞으로의 계획

**스트리밍**과 **서버 컴포넌트**는 데이터 패칭과 로딩 상태를 처리하는 새로운 방법을 제공하여 궁극적으로 최종 사용자 경험을 향상시키는 목표를 가지고 있어요.

다음 챕터에서는 **부분 사전 렌더링**에 대해 배울 거예요. 이는 스트리밍을 염두에 두고 설계된 새로운 Next.js 렌더링 모델이에요.

이 새로운 모델을 통해 더 효율적인 데이터 처리와 사용자 인터페이스 로딩 상태를 관리할 수 있는 방법을 배우게 될 거예요.
