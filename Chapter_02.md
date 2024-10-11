chapter_01 에서 홈페이지에 아무 스타일링이 적용되지 않았었어요. 이제 Next.js 애플리케이셔넹 스타일링 하는 방법을 배워봅시다!

## 이번 챕터에서는...

1. **global CSS 파일 추가하기** : <br/>
global CSS 파일을 추가하려면, 프로젝트의 특정 폴더에 CSS 파일을 만들고, Next.js 에서 이 파일을 불러와야 합니다.
2. **2가지의 스타일링 방법** : <br/>
Tailwind 와 CSS 모듈 두 가지 방식으로 스타일링 할 수 있습니다.
   - **Tailwind** : 유틸리티 기반의 CSS 프레임워크로, 미리 정의된 클래스 이름을 사용하여 스타일을 적용합니다.
   - **CSS 모듈** : CSS 파일을 모듈화하여 각 컴포넌트에만 적용할 수 있는 방법입니다. 이 방식은 스타일 충돌을 방지할 수 있습니다.
3. **clsx 유틸리티 패키지** :<br/>
조건에 따라 클래스 이름을 추가할 수 있도록 도와주는 ``clsx`` 라는 유틸리티 패키지를 사용할 수 있습니다. 
이를 통해 동적으로 클래스 이름을 관리할 수 있습니다.

<hr/>

## Global Styles

``/app/ui`` 폴더 안을 보면 ``global.css`` 라는 파일이 있습니다. <br/>
이 파일을 사용하면 애플리케이션의 모든 경로에 CSS 규칙을 추가할 수 있습니다. <br/>
예를 들어, CSS 리셋 규칙, 링크와 같은 HTML 요소에 대한 사이트 전체 스타일 등을 설정할 수 있습니다. 

``global.css`` 파일은 애플리케이션의 어떤 컴포넌트에서도 불러올 수 있지만, 일반적으로 최상위 컴포넌트에 추가하는 것이 좋습니다.<br/>
Next.js 에서는 이 최상위 컴포넌트가 root layout 입니다. (자세한 내용은 나중에 설명합니다.)

global styles 을 애플리케이션에 추가하려면 <span style="background-color: rgba(0,200,200, 0.2)">``/app/layout.tsx`` 파일로 이동하여 ``global.css``파일을 불러오면 됩니다.</span>

```javascript
import '@/app/ui/global.css';
 
export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}
```

아직 dev 서버가 실행되고 있다면, 변경사항을 저장하고 브라우저에서 미리보기를 볼 수 있어요.
홈페이지는 이제 아래와 같이 보이게 될 거예요.

<img src="https://nextjs.org/_next/image?url=%2Flearn%2Fdark%2Fhome-page-with-tailwind.png&w=1920&q=75" alt="browser preview" />

여기서 잠깐만!! 🤔<br/>
CSS 규칙을 추가하지 않았는데 스타일은 어디에서 온 걸까요?

``global.css`` 파일 안을 들여다보면 몇 가지 ``@tailwind`` 지시문이 있는 것을 알 수 있습니다.

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

<hr/>

## Tailwind

[Tailwind](https://tailwindcss.com/) 는 CSS 프레임워크로, 개발 과정을 빠르게 해주는 도구입니다.
이 프레임워크를 사용하면 TSX 마크업에 [유틸리티 클래스](https://tailwindcss.com/docs/utility-first)를 쉽게 쓸 수 있습니다.

Tailwind 에서는 elements 를 스타일링할 때 클래스 이름을 추가합니다.
예를 들어, 클래스 "text-blue-500" 을 추가하면 ``<h1>`` 텍스트가 파란색으로 변합니다.

```javascript
<h1 className="text-blue-500">I'm blue!</h1>
```

CSS 스타일은 전역적으로 공유되지만, 각 클래스는 각 요소에 개별적으로 적용됩니다.
즉, 요소를 추가하거나 삭제하더라도 별도의 stylesheets 를 유지하거나 스타일 충돌을 걱정할 필요가 없고,
애플리케이션이 커질 수록 CSS 번들이 커지는 것에 대해서도 신경 쓸 필요가 없습니다.

``create-next-app`` 을 사용하여 새 프로젝트를 시작할 때, Next.js는 Tailwind 를 사용할 것인지 물어봅니다.
"yes" 를 선택하면 Next.js가 필요한 패키지를 자동으로 설치하고 애플리케이션에 Tailwind 를 설정해줍니다.

``/app/page.tsx``를 살펴보면 예제에서 Tailwind 클래스를 사용하는 것을 볼 수 있습니다.

```javascript
import AcmeLogo from '@/app/ui/acme-logo';
import { ArrowRightIcon } from '@heroicons/react/24/outline';
import Link from 'next/link';
 
export default function Page() {
  return (
    // These are Tailwind classes:
    <main className="flex min-h-screen flex-col p-6">
      <div className="flex h-20 shrink-0 items-end rounded-lg bg-blue-500 p-4 md:h-52">
    // ...
  )
}
```

Tailwind 를 처음 사용하는 것이라도 걱정하지 마세요.
시간을 절약하기 위해, 여러분이 사용할 모든 컴포넌트는 이미 스타일링이 되어 있습니다.

이제 Tailwind 를 사용해볼까요! 🎨
아래의 코드를 복사해서 ``/app/page.tsx`` 에서 ``<p>`` 요소 위에 붙여넣어 보세요 :
```javascript
<div
  className="relative w-0 h-0 border-l-[15px] border-r-[15px] border-b-[26px] border-l-transparent border-r-transparent border-b-black"
/>
```
<details>
<summary>위의 코드를 붙여 넣으면 어떤 모양을 볼 수 있을까요?</summary>
정답 : 검정색의 삼각형
</details>

전통적인 CSS 규칙을 작성하거나 스타일을 JSX 와 분리하고 싶다면, CSS 모듈은 좋은 대안입니다. <br/>
CSS 모듈을 사용하면 각 스타일이 해당 컴포넌트에만 적용되어, 스타일 충돌을 방지하고 관리하기 쉽게 만들어 줍니다.

이 방식을 통해 각 파일에 맞춤형 CSS 를 작성할 수 있으며, 필요한 경우 클래스 이름을 import 하여 사용할 수 있습니다. <br/>
CSS 모듈을 사용하면 더 큰 프로젝트에서도 스타일을 깔끔하게 유지할 수 있습니다.

<hr/>

## CSS Modules

CSS modules 를 사용하면 CSS 를 특정 컴포넌트에만 적용할 수 있도록 자동으로 고유한 클래스 이름을 만들어주기 때문에 스타일 충돌에 대해 걱정할 필요가 없습니다. <br/>
이번 코스에서는 계속해서 Tailwind 를 사용할 것이지만, 위 퀴즈에서 얻은 결과를 CSS 모듈로 어떻게 구현할 수 있는지 잠깐 살펴보겠습니다.

``/app/ui`` 폴더 안에 ``home.module.css`` 라는 새로운 파일을 만들고, 다음과 같은 CSS 규칙을 추가하세요 :

```css
.shape {
  height: 0;
  width: 0;
  border-bottom: 30px solid black;
  border-left: 20px solid transparent;
  border-right: 20px solid transparent;
}
```

그런 다음, ``/app/page.tsx`` 파일 안에서 스타일을 가져오고, <br/>
추가한 ``<div>`` 의 Tailwind 클래스 이름을 ``styles.shape`` 으로 바꿔주세요.

```javascript
import AcmeLogo from '@/app/ui/acme-logo';
import { ArrowRightIcon } from '@heroicons/react/24/outline';
import Link from 'next/link';
import styles from '@/app/ui/home.module.css';
 
export default function Page() {
  return (
    <main className="flex min-h-screen flex-col p-6">
      <div className={styles.shape} />
    // ...
  )
}
```

변경사항을 저장하고 브라우저에서 미리보기를 확인해보세요. <br/>
이전과 동일한 모양이 나타날 겁니다!

Tailwind 와 CSS Modules 는 Next.js 애플리케이션에서 가장 일반적인 스타일링 방법 두 가지 입니다. <br/>
둘 중 어떤 것을 사용하는지는 개인의 취향 문제이며, 같은 애플리케이션에서 두 가지 모두 사용할 수도 있습니다!

<hr />

## ``clsx`` 라이브러리로 클래스 이름 토클하기

요소를 상태나 다른 조건에 따라 조건부로 스타일링 해야 할 경우가 있을 수 있어요.

``clsx`` 는 클래스 이름을 쉽게 토클할 수 있도록 해주는 라이브러리입니다. <br/>
더 많은 정보는 문ㄴ서를 참고하면 좋지만, 기본 사용법은 다음과 같아요 :

예를 들어, ``InvoiceStatus`` 라는 컴포넌트를 만들고, 이 컴포넌트는 ``status`` 라는 속성을 받는다고 가정해 볼게요. <br/>
``status`` 는 'pending' 또는 'paid' 일 수 있습니다. 만약 'paid' 라면 색깔을 초록색으로 하고, 'pending' 이람ㄴ 회색으로 하고 싶다면, clsx 를 사용하여 클래스를 조건부로 적용할 수 있어요.

```javascript
import clsx from 'clsx';

const InvoiceStatus = ({ status }) => {
  return (
    <span className={clsx({
      'text-green-500': status === 'paid',
      'text-gray-500': status === 'pending',
    })}>
      {status}
    </span>
  );
};
```

<hr/>

## 다른 스타일링 방법

위의 방법 외에도 Next.js 애플리케이션을 스타일링하는 다른 방법들은 아래와 같아요 : 
- **Sass** : .css 파일과 .scss. 파일을 가져와 사용할 수 있어요.
- **CSS-in-JS** 라이브러리 : 예를 들어 ``styled-jsx``, ``styled-components``, ``emotion`` 같은 것들이 있어요.

더 자세한 내용은 CSS 문서를 참고하면 좋아요!

<details>
<summary>
CSS-in-JS 가 선호되지 않는 이유
</summary>
CSS-in-JS 라이브러리와 서버 사이드 렌더링(SSR)을 함께 사용할 때 발생할 수 있는 주요 문제점은 FOUC(Flash of Unstyled Content) 현상이에요. <br />
이 현상은 스타일이 적용되지 않은 상태로 페이지가 잠깐 표시되었다가, 나중에 스타일이 로드되면서 깜빡이는 문제를 말해요. <br /><br />
1. SSR 에서의 스타일 처리 지연 : <br />
서버 측 렌더링은 브라우저에서 페이지를 요청할 때 서버에서 미리 HTML 을 생성해 반환하는 방식이에요. <br /> 
CSS-in-JS 라이브러리는 자바스크립트 코드 안에서 동적으로 스타일을 생성하는데, 이로 인해 서버에서 생성된 HTML 이 브라우저에 도달할 때 스타일이 적용되기 전에 페이지가 먼저 렌더링 될 수 있어요. <br/>
그 결과, 사용자는 잠시 동알 스타일이 없는 페이지를 보게 돼요. 😭 <br /><br />
2. 스타일 로딩의 비동기적 특성 : <br />
CSS-in-JS 라이브러리들은 스타일을 런타임에 생성하고 적용하기 때문에, 스타일이 서버에서 빠르게 렌더링되지 않으면 클라이언트 측에서 스타일이 적용될 때까지 지연이 발생할 수 있어요. <br />
이 과정이 비동기적으로 처리되므로, 스타일이 느리게 로드될 때 FOUC 문제가 발생할ㄹ 가능성이 높아요. 😥 <br /><br />
3. 스타일 서버에서 클라이언트로 전달 어려움 :<br />
SSR 에서는 서버에서 미리 HTML 과 CSS 를 클라이언트로 보내야 하는데, CSS-in-JS 방식은 자바스크립트가 실행되면서 스타일이 생성되기 때문에, SSR 환경에서는 스타일을 서버에서 미리 계산하고 클라이언트에전달하는 것이 더 복잡해져요. 🥺 <br />
<br />
이 문제를 해결하려면, CSS-in-JS 라이브러리를 사용할 때 SSR 에 최적화된 설정을 하거나, <br />
정적인 CSS 파일을 사용하는 방식으로 혼합하여 사용하는 것이 일반적이에요. <br />
Tailwind CSS 나 CSS Modules 처럼 정적 스타일시트 기반의 해결책이 서버 렌더링 환경에서 더 안정적으로 동작할 수 있는 이유예요!

글쓴이 : 보통은 Tailwind 나 PigmentCSS 를 사용하는 추세입니다 😊
</details>
