지금까지 우리의 애플리케이션은 홈페이지만 있어요. <br/>
이제 더 많은 route(경로)를 만들고 레이아웃과 페이지를 추가하는 방법을 배워볼 거예요. 🚏

## 이번 챕터에서는...

- 파일 시스템 라우팅을 사용하여 대시보드 경로 만들기
- 새 route segments 를 만들 때 폴더와 파일의 역할 이해하기
- 여러 대시보드 페이지에서 공유할 수 있는 중첩 레이아웃 만들기
- 콜로케이션(colocation), 부분 렌더링(partial rendering), 루트 레이아웃(root layout)에 대해서 이해하기

<hr/>

## 중첩 레이아웃 (Nested routing)

Next.js 는 파일 시스템 라우팅을 사용하여 폴더를 사용해 중첩된 경로를 생성해요. 각 폴더는 URL의 한 부분에 해당하는 경로 세그먼트를 나타내요.

<img src="https://nextjs.org/_next/image?url=%2Flearn%2Fdark%2Ffolders-to-url-segments.png&w=3840&q=75" alt="route example" />

각 경로에 대한 별도의 사용자 인터페이스(UI)를 만들기 위해 ``layout.tsx``와 ``page.tsx`` 파일을 사용할 수 있어요.

예를 들어, 우리의 애플리케이션에는 이미 ``/app/page.tsx`` 라는 페이지 파일이 있는데, 이는 ``/`` 경로와 연결된 홈페이지에 해당돼요.

중첩된 경로(nested route)를 만들기 위해서는 폴더를 서로 중첩해서 만들고, 그 안에 ``page.tsx`` 파일을 추가하면 돼요. <br/>
예를 들어, ``dashboard`` 라는 폴더 안에 또 다른 폴더인 ``settings`` 를 만들어서 그 안에 ``page.tsx`` 파일을 넣으면, ``/dashboard/settings`` 경로가 생성되는
거죠. <br/>
이렇게 폴더 구조를 활용하면, 각각의 경로에 맞는 UI를 편하게 만들 수 있어요. 😊

<img src='https://nextjs.org/_next/image?url=%2Flearn%2Fdark%2Fdashboard-route.png&w=3840&q=75' alt='nested route example' />

``/app/dashboard/page.tsx``는 ``/dashboard`` 경로와 연결되어 있어요. <br/>
이제 이 페이지를 만들어서 어떻게 작동하는지 살펴볼까요!

<hr />

## dashboard 페이지 만들기

``/app`` 폴더 안에 ``dashboard`` 라는 새로운 폴더를 만들어 주세요. <br/>
그 다음, ``dashboard``폴더 안에 ``page.tsx``라는 새로운 파일을 만들고, 아래의 코드를 넣어주세요 :

```javascript
export default function Page() {
    return <p>Dashboard Page</p>;
}

/* 글쓴이는 화살표 함수를 사용했습니다 !!!! */
const Page = () => {
    return <p>Dashboard Page</p>;
};

export default Page;
```

이제 개발 서버가 실행되고 있는지 확인한 후, http://localhost:3000/dashboard 를 방문해 보세요. <br />
"Dashboard Page" 라는 텍스트가 보일 거예요! ☺️

폴더를 사용하여 새로운 경로 세그먼트(route segment)를 만들고, 그 안에 페이지 파일을 추가하면 페이지를 만들 수 있습니다.

특정 이름을 가진 페이지 파일을 사용함으로써 Next.js 는 UI 컴포넌트, 테스트 파일, 그리고 기타 관련 코드를
경로(route)와 [함께 배치(colocate)](https://nextjs.org/docs/app/building-your-application/routing#colocation)할 수 있게 해줍니다. <br/>
페이지 파일 안의 내용만 공개적으로 접근할 수 있습니다. <br/>
예를 들어, ``/app`` 폴더 안에는 ``/ui``와 ``/lib`` 폴더가 있으며, 이들은 우리 route 와 함께 배치되어 있습니다. <br/>
(그러니까, ``page.tsx`` 파일이 없다면 접근할 수 없다!!! 라는 뜻)

<hr />

## 연습 : dashboard pages 만들기

대시보드에 다음 두 페이지를 추가해주세요 :

1. Customers 페이지 : 이 페이지는 http://localhost:3000/dashboard/customers 에서 접근할 수 있어야 해요. 지금은 `<p>Customers Page</p>` 요소만
   반환하면 돼요.
2. Invoices 페이지 : 이 페이지는 http://localhost:3000/dashboard/invoices 에서 접근할 수 있어야 해요. 이것도 마찬가지로 `<p>Invoices Page</p>` 요소만
   반환하면 돼요.

<details>
<summary>the solution</summary>
<img src="https://nextjs.org/_next/image?url=%2Flearn%2Fdark%2Frouting-solution.png&w=3840&q=75" alt="dashboard pages" />

Customers Page:

```javascript
export default function Page() {
    return <p>Customers Page</p>;
}

/* 화살표 함수 */
const Page = () => {
    return <p>Customers Page</p>;
};

export default Page;
```

Invoices Page:

```javascript
export default function Page() {
    return <p>Invoices Page</p>;
}

/* 화살표 함수 */
const Page = () => {
    return <p>Invoices Page</p>;
};

export default Page;

```

</details>

<hr />

## dashboard layout 만들기

대시보드에는 여러 페이지에서 공유되는 내비게이션(navigation)이 있어요. <br/>
Next.js 에서는 ``layout.tsx`` 라는 특별한 파일을 사용하여 여러 페이지에서 공유되는 UI를 만들 수 있습니다. <br/>
대시보드 페이지의 레이아웃은 만들어 볼까요?

먼저 ``/dashboard``폴더 안에 ``layout.tsx`` 라는 새 파일을 추가하고, 아래의 코드를 붙여 넣어주세요 :

```javascript
import SideNav from '@/app/ui/dashboard/sidenav';

export default function Layout({children}: { children: React.ReactNode }) {
    return (
        <div className="flex h-screen flex-col md:flex-row md:overflow-hidden">
            <div className="w-full flex-none md:w-64">
                <SideNav/>
            </div>
            <div className="flex-grow p-6 md:overflow-y-auto md:p-12">{children}</div>
        </div>
    );
}

/* 화살표 함수 */
import SideNav from "@/app/ui/dashboard/sidenav";

const Layout = ({children}: { children: React.ReactNode }) => (
    <div className="flex h-screen flex-col md:flex-row md:overflow-hidden">
        <div className="w-full flex-none md:w-64">
            <SideNav/>
        </div>
        <div className="flex-grow p-6 md:overflow-y-auto md:p-12">{children}</div>
    </div>
);

export default Layout;
```

이 코드에서 몇 가지 중요한 내용이 있으니, 하나씩 살펴볼게요.

먼저, ``layout.tsx``에서 ``<SideNav />`` 컴포넌트를 가져오고 있어요. <br/>
이 파일에 가져온 모든 컴포넌트는 레이아웃의 일부가 됩니다.

``<Layout />`` 컴포넌트는 ``children`` 일라는 prop을 받아요. <br/>
이 ``children``은 페이지나 다른 레이아웃이 될 수 있습니다. <br/>
우리의 경우, ``/dashboard`` 안에 있는 페이지들은 자동으로 ``<Layout />`` 안에 중첩되어 이렇게 구성됩니다 :

<img src="https://nextjs.org/_next/image?url=%2Flearn%2Fdark%2Fshared-layout.png&w=3840&q=75" alt="layout nested" />

모든 변경사항을 저장한 후, localhost에서 제대로 작동하는지 확인해보세요. <br />
그러면 아래와 같은 내용을 볼 수 있을 거예요.

<img src="https://nextjs.org/_next/image?url=%2Flearn%2Fdark%2Fshared-layout-page.png&w=1920&q=75" alt="apply layout component" />

Next.js 에서 레이아웃을 사용하면 한 가지 큰 장점이 있어요! <br />
navigation 을 할 때, 페이지 컴포넌트만 업데이트 되고 레이아웃은 다시 렌더링되지 않아요. <br />
이를 [**부분 렌더링(partial rendering)
**](https://nextjs.org/docs/app/building-your-application/routing/linking-and-navigating#4-partial-rendering) 이라고 해요.

즉, 사용자가 페이지를 이동할 때 레이아웃은 그대로 유지되면서, 페이지 내용만 바뀌기 때문에 성능이 좋아지고 더 빠르게 반응할 수 있어요. <br/>
이런 방식 덕분에 웹 애플리케이션의 사용자 경험이 더 부드러워지고, 필요하지 않은 렌더링을 줄일 수 있답니다 ☺️

<img src="https://nextjs.org/_next/image?url=%2Flearn%2Fdark%2Fpartial-rendering-dashboard.png&w=3840&q=75" alt="partial rendering structure" />

<hr />

## Root layout

Chapter 3 에서는 다른 레이아웃인 ``/app/layout.tsx`` 에 ``Inter`` font 를 가져왔어요.<br/>
다시 한 번 상기시키자면, 이 font 를 사용하면 애플리케이션 전역에서 일관된 스타일을 유지할 수 있게 되죠.

```javascript
import '@/app/ui/global.css';
import {inter} from '@/app/ui/fonts';

export default function RootLayout({
                                       children,
                                   }: {
    children: React.ReactNode;
}) {
    return (
        <html lang="en">
            <body className={`${inter.className} antialiased`}>{children}</body>
        </html>
    );
}
```

이것을 [root layout](https://nextjs.org/docs/app/building-your-application/routing/pages-and-layouts#root-layout-required)
이라고 하며, 필수적으로 필요해요. <br />
root layout 에 추가하는 모든 UI는 애플리케이션의 모든 페이지에 공유되므로, 모든 페이지에서 일관된 사용자 경험을 제공할 수 있어요. <br />
root layout 을 사용하면 ``<html>``및 ``<body>`` 태그를 수정하고 meta data 를 추가할 수 있어요 (meta data에 대한
내용은 [이후 chapter](https://nextjs.org/learn/dashboard-app/adding-metadata)에서 배울 거예요).

방금 만든 레이아웃인 ``/app/dashboard/layout.tsx``는 대시보드 페이지에만 고유하므로, root layout 위에 UI를 추가할 필요는 없어요.
