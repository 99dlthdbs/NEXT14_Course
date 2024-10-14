이전 챕터에서는 Next.js 애플리케이션을 스타일링 하는 방법에 대해서 배웠어요. <br/>
이번에는 홈페이지를 더 멋지게 만들기 위해 사용자 정의 폰트와 hero 이미지를 추가하는 방법에 대해서 배워볼 거예요. <br/>

## 이번 챕터에서는...

1. ``next/font`` 를 사용해 사용자 정의 폰트를 추가하는 방법
2. ``next/image`` 를 사용해 이미지를 추가하는 방법
3. Next.js 에서 폰트와 이미지가 최적화되는 방법

## 왜 font 를 최적화해야 할까요?

font 는 웹사이트 디자인에서 중요한 역할을 해요. <br/>
하지만 custom font 를 사용하면 그 파일을 불러오느라 시간이 걸려 성능에 영향을 줄 수 있어요. <br/>

구글에서 사용하는 [**누적 레이아웃 이동(Cumulative Layout Shift, CLS)**](https://vercel.com/blog/how-core-web-vitals-affect-seo) 이라는 기준이
있어요. <br/>
이건 웹사이트가 얼마나 잘 작동하고, 사용자가 편안하게 이용할 수 있는 지 평가하는 지표예요.<br/>

font 가 늦게 불러와지면, 처음에는 기본 font 로 글자를 보여주다가 나중에 custom font 로 바꿔 보여줘요. <br/>
이때 글자의 크기나 위치가 바뀌면서 화면이 갑자기 움직이게 돼요. 이렇게 되면 사용하기 불편하죠.

그래서 Next.js 에서는 font 을 효율적으로 불러오게 도와주는 방법이 있어요. <br/>
이를 통해 이런 문제를 줄이고, 웹사이트가 더 빠르고 안정적으로 보일 수 있게 도와줘요. <br/>

<img src='https://nextjs.org/_next/image?url=%2Flearn%2Fdark%2Ffont-layout-shift.png&w=3840&q=75' alt='cumulative layout shift' />

Next.js 는 ``next/font`` 모듈을 사용할 때 font 를 자동으로 최적화해요. <br/>
이 모듈을 사용하면 빌드할 때 font 파일을 미리 다운로드 하고, 다른 정적 파일과 함께 저장해요. <br/>
그래서 사용자가 웹사이트를 방문할 때, font 를 따로 불러오기 위해 추가적인 네트워크 요청을 할 필요가 없어요. <br/>
이렇게 하면 웹사이트가 더 빨리 로드되고 성능에 영향을 덜 주게 돼요.

## 기본 font 추가하기

구글에서 제공하는 맞춤형 font 를 프로젝트에 추가해봐요!

먼저. ``/app/ui`` 폴더 안에 ``fonts.ts`` 라는 새 파일을 만드세요. <br/>
이 파일은 애플리케이션 전체에서 사용할 font 를 저장하는 곳이에요.

이제 ``next/font/google`` 모듈에서 ``Inter`` font 를 불러올 거예요. <br/>
``Inter`` 는 주로 사용할 기본 font 가 될 거고, ``latin`` [subset](https://fonts.google.com/knowledge/glossary/subsetting) 을 불러오도록
설정할 거예요.

이 과정을 통해 프로젝트에 font 를 쉽게 적용할 수 있어요.

```javascript
import {Inter} from 'next/font/google';

export const inter = Inter({subsets: ['latin']});
```

마지막으로, ``/app/layout.tsx``에 있는 ``<body>`` 엘리먼트에 font 를 추가해요.

```javascript
import '@/app/ui/global.css';
import {inter} from '@/app/ui/fonts';

export default function RootLayout({children}: {
    children: React.ReactNode;
}) {
    return (
        <html lang="en">
            <body className={`${inter.className} antialiased`}>{children}</body>
        </html>
    );
}
```

``Inter`` font 를 ``<body>`` 요소에 추가하면, 애플리케이션 전체에 이 font 가 적용됩니다. <br/>
여기에 Tailwind 의 [antialiased](https://tailwindcss.com/docs/font-smoothing) 클래스도 추가했어요. <br/>
이 클래스는 font 를 부드럽게 처리해주는 역할을 합니다. 꼭 사용해야 하는 건 아니지만, font 가 좀 더 깔끔하게 보이도록 도와줘요.

브라우저에서 개발자 도구(Dev Tools)를 열고, ``<body>`` 요소를 선택해 보세요. <br/>
이제 스타일에서 Inter 와 Inter_Fallback 이 적용된 것을 확인할 수 있을 거예요.

## Secondary font 추가하기

이제 직접 해볼 차례예요! <br/>
``/app/ui/fonts.ts`` 파일에서 ``Lusitana`` 라는 secondary font 를 가져와서, ``/app/page.tsx`` 파일의 ``<p>`` 요소에 적용해보세요. <br/>
이전에 했던 것처럼 font set 을 지정하고, 이번에는 font 의 굵기(font weight) 도 설정해야 해요.

준비가 되면 아래 코드 스니펫을 펼쳐서 정답을 확인해 보세요.

> 힌트
> - 어떤 font weight 를 사용해야 할 지 잘 모르겠다면, 코드 편집기에서 TypeScript 에러를 확인해보세요.
> - [Google Fonts](https://fonts.google.com/) 웹사이트에서 Lusitana 를 검색해 사용 가능ㅇ한 옵션들을 확인해보세요.
> - [여러 font 를 추가하는 방법](https://nextjs.org/docs/app/building-your-application/optimizing/fonts#using-multiple-fonts)과
    [옵션 종류](https://nextjs.org/docs/app/api-reference/components/font#font-function-arguments)은 문서를 참고하세요.

<details>
<summary>
the solution
</summary>

``/app/ui/fonts.ts``

```javascript
import {Inter, Lusitana} from 'next/font/google';

export const inter = Inter({subsets: ['latin']});

export const lusitana = Lusitana({
    weight: ['400', '700'],
    subsets: ['latin'],
});
```

``/app/page.tsx``

```javascript
import AcmeLogo from '@/app/ui/acme-logo';
import {ArrowRightIcon} from '@heroicons/react/24/outline';
import Link from 'next/link';
import {lusitana} from '@/app/ui/fonts';

export default function Page() {
    return (
        // ...
        <p
            className={`${lusitana.className} text-xl text-gray-800 md:text-3xl md:leading-normal`}
        >
            <strong>Welcome to Acme.</strong>
            This is the example for the{' '}
            <a href="https://nextjs.org/learn/" className="text-blue-500">
                Next.js Learn Course
            </a>
            , brought to you by Vercel.
        </p>
        // ...
    );
}
```

</details>

마지막으로, ``<AcmeLogo />`` 컴포넌트도 ``Lusitana`` font 를 사용해요. 오류를 방지하기 위해 주석 처리되어 있었는데, 이제 주석을 해제해볼까요?

```javascript
// ...

export default function Page() {
    return (
        <main className="flex min-h-screen flex-col p-6">
            <div className="flex h-20 shrink-0 items-end rounded-lg bg-blue-500 p-4 md:h-52">
                <AcmeLogo/>
                {/* ... */}
            </div>
        </main>
    );
}
```

좋아요! 이제 애플리케이션에 두 개의 custom font 를 추가했으니, 다음으로 홈페이지에 hero 이미지를 추가해볼게요.

## 왜 이미지를 최적화해야 하나요?

Next.js는 이미지와 같은 ``static asset`` 을 ``/public`` 폴더에서 제공할 수 있어요. <br />
``/public`` 폴더 안에 있는 파일들은 애플리케이션에서 참조할 수 있어요.

일반 HTML 에서는 이미지를 아래와 같이 추가할 수 있어요 :

```html
<img
        src="/hero.png"
        alt="Screenshots of the dashboard project showing desktop version"
/>
```

하지만 이렇게 이미지를 추가하게 되면 수동으로 아래의 작업을 해야 해요 😭:

- 다양한 화면 크기에서 이미지가 반응형으로 보이도록 하기
- 다른 devices 에 맞게 이미지 크기를 지정하기
- 이미지가 로드될 때 레이아웃 이동 방지
- 사용자의 화면에 보이지 않는 이미지에 대해 지연 로딩 (lazy load)

이미지 최적화는 웹 개발에서 큰 주제로, 그것만으로도 전문 분야가 될 수 있습니다. <br/>
이러한 최적화를 수동으로 구현하는 대신, ``next/image`` 컴포넌트를 사용하면 이미지를 자동으로 최적화할 수 있어요.

## ``<Image>`` 컴포넌트

``<Image>`` 컴포넌트는 HTML 의 ``<img>`` 태그를 확장한 것이며, 아래의 자동 이미지 최적화 기능을 제공해요 :

- 이미지가 로드될 때 자동으로 레이아웃 이동 방지
- 작은 화면을 가진 device 로 큰 이미지를 전송하지 않도록 이미지를 크기에 맞게 조정
- 기본적으로 이미지를 지연 로딩(lazy loading) 하여, 사용자가 화면에 들어올 때만 이미지를 로드
- 브라우저가 지원할 경우, WebP 및 AVIF 와 같은 현대적인 형식으로 이미지 제공

## 데스크탑 hero 이미지 추가하기

이제 ``<Image />`` 컴포너트를 사용해볼까요? <br/>
``/public`` 폴더 안에는 ``hero-desktop.png`` 와 ``hero-mobile.png`` 라는 두 개의 이미지가 있어요.<br/>
이 두 이미지는 완전히 다르며, 사용자의 기기가 데스크탑인지 모바일인지에 따라 표시돼요.

이제 ``/app/page.tsx`` 파일에서 ``next/image`` 에서 컴포넌트를 가져옵니다. <br/>
그런 다음 주석 아래에 이미지를 추가해요.

```javascript
import AcmeLogo from '@/app/ui/acme-logo';
import {ArrowRightIcon} from '@heroicons/react/24/outline';
import Link from 'next/link';
import {lusitana} from '@/app/ui/fonts';
import Image from 'next/image';

export default function Page() {
    return (
        // ...
        <div className="flex items-center justify-center p-6 md:w-3/5 md:px-28 md:py-12">
            {/* Add Hero Images Here */}
            <Image
                src="/hero-desktop.png"
                width={1000}
                height={760}
                className="hidden md:block"
                alt="Screenshots of the dashboard project showing desktop version"
            />
        </div>
        //...
    );
}
```

여기서는 이미지를 ``width: 1000px`` ``height: 760px`` 으로 하고 있어요. <br/>
이미지의 너비와 높이를 설정하는 것은 레이아웃 변동을 피하는 좋은 방법이며, 이 값들은 원본 이미지와 동일한 비율을 유지해야 해요.

또한, 모바일 화면에서 이미지를 DOM 에서 제거하기 위해 ``class hidden`` 울 사용하고, 데스크탑 화면에서 이미지를 표시하기 위해 ``md:block`` 클래스를 사용하고 있어요.

이제 우리의 홈페이지는 이렇게 보여야 해요!

<img src='https://nextjs.org/_next/image?url=%2Flearn%2Fdark%2Fhome-page-with-hero.png&w=1920&q=75' alt='homepage' />

<hr />

## 연습: 모바일 hero 이미지 추가하기

이제 여러분의 차례예요! <br />
방금 추가한 이미지 아래에 ``hero-mobile.png`` 에 대한 또 다른 ``<Image>`` 컴포넌트를 추가하세요.

``width: 560px, height: 620px`` 로 설정해야 해요. <br/>
이 이미지는 모바일 화면엣서 보여야 하고, 데스크탑에서는 숨겨져야 해요. <br />
dev tools 를 사용해서 데스크탑과 모바일 이미지가 올바르게 교체되는지 확인할 수 있어요.

준비가 되면 아래 해결책을 확인하세요!

<details>
<summary>
the solutions
</summary>

```javascript
import AcmeLogo from '@/app/ui/acme-logo';
import {ArrowRightIcon} from '@heroicons/react/24/outline';
import Link from 'next/link';
import {lusitana} from '@/app/ui/fonts';
import Image from 'next/image';

export default function Page() {
    return (
        // ...
        <div className="flex items-center justify-center p-6 md:w-3/5 md:px-28 md:py-12">
            {/* Add Hero Images Here */}
            <Image
                src="/hero-desktop.png"
                width={1000}
                height={760}
                className="hidden md:block"
                alt="Screenshots of the dashboard project showing desktop version"
            />
            <Image
                src="/hero-mobile.png"
                width={560}
                height={620}
                className="block md:hidden"
                alt="Screenshot of the dashboard project showing mobile version"
            />
        </div>
        //...
    );
}
```

</details>

좋아요! 👍
이제 우리 홈페이지에는 custom font 와 hero 이미지가 생겼어요.

## 추천하는 자료

이 주제에 대해 배울 것이 정말 많아요 🤓 <br/>
remote 이미지를 최적화하거나 로컬 폰트 파일을 사용하는 것에 대해 더 자세히 알고 싶다면 아래의 자료를 참고해 보세요 :

- [이미지 최적화 문서](https://nextjs.org/docs/app/building-your-application/optimizing/images)
- [폰트 최적화 문서](https://nextjs.org/docs/app/building-your-application/optimizing/fonts)
- [이미지로 웹 성능 향상하기 (MDN)](https://developer.mozilla.org/en-US/docs/Learn/Performance/Multimedia)
- [웹 폰트 (MDN)](https://developer.mozilla.org/en-US/docs/Learn/CSS/Styling_text/Web_fonts)
- [코어 웹 vitals이 SEO에 미치는 영향](https://vercel.com/blog/how-core-web-vitals-affect-seo)


