이전 장에서 대시보드 레이아웃과 페이지를 만들었죠! <br />
이제 사용자가 대시보드 경로 간에 이동할 수 있도록 링크를 추가해 볼게요.

## 이번 챕터에서는...

아래의 topic 을 공부할 거예요.

- `next/link` 컴포넌트 사용하는 방법
- `usePathname()` hook 을 이용하여 active(활성화) 링크를 표시하는 방법
- Next.js 에서 navigation 이 동작하는 방법

<hr />

## 왜 navigation 을 최적화해야 하나요?

페이지 간 이동을 할 때 전토억으로는 HTML의 `<a>` 요소를 사용합니다. <br />
현재 사이드바 링크는 `<a>` 요소를 사용하고 있지만, 브라우저에서 home, customers, invoices 페이지 사이를 이동할 때 무슨 일이 일어나는지 확인해 보세요.

보셨나요?

페이지를 이동할 때마다 전체 페이지가 새로 고침됩니다! 😥

<hr/>

## `<Link>` 컴포넌트

Next.js 에서는 `<Link />` 컴포넌트를 사용하여 애플리케이션 내 페이지 간 링크를 걸 수 있습니다. <br />
`<Link />` 는 JavaScript 를
사용한 [client-side navigation](https://nextjs.org/docs/app/building-your-application/routing/linking-and-navigating#how-routing-and-navigation-works)
을 가능하게 해줘요.

`/app/ui/dahboard/nav-links.tsx` 파일을 열고, [next/link](https://nextjs.org/docs/app/api-reference/components/link)로부터
`Link` 컴포넌트를 가져올 수 있어요. <br />
그런 다음, `<a>` 태그를 `<Link>` 로 교체해주세요.

```javascript
import {
    UserGroupIcon,
    HomeIcon,
    DocumentDuplicateIcon,
} from '@heroicons/react/24/outline';
import Link from 'next/link';

// ...

export default function NavLinks() {
    return (
        <>
            {links.map((link) => {
                const LinkIcon = link.icon;
                return (
                    <Link
                        key={link.name}
                        href={link.href}
                        className="flex h-[48px] grow items-center justify-center gap-2 rounded-md bg-gray-50 p-3 text-sm font-medium hover:bg-sky-100 hover:text-blue-600 md:flex-none md:justify-start md:p-2 md:px-3"
                    >
                        <LinkIcon className="w-6"/>
                        <p className="hidden md:block">{link.name}</p>
                    </Link>
                );
            })}
        </>
    );
}
```

여러분이 보는 것과 같이, `<Link>` 컴포넌트는 `<a>` 태그와 비슷하게 사용되지만, `<a href="...">` 대신 `<Link href="...">`를 사용해요.

변경 사항을 저장한 후, 로컬 서버에서 작동하는지 확인하세요. <br />
이제 페이지 navigation 시 전체 페이지가 새로 고침되지 않아야 해요. <br/>
비록 애플리케이션의 일부가 서버에서 렌더링 되더라도, 전체 페이지 새로 고침이 일어나지 않아서 웹 애프리케이션처럼 느껴져요. <br/>

왜 그럴까요? 🤔

### 자동 코드 분할과 미리 가져오기 (Automatic code-splitting and prefetching)

navigation 경험을 개선하기 위해, Next.js 는 자동으로 애플리케이션을 route segment 로 코드를 분할해요. <br/>
이것은 전통적인 React [SPA](https://developer.mozilla.org/en-US/docs/Glossary/SPA)와 달리, 브라우저가 처음 로드 시 모든 애플리케이션 코드를 로드하지 않게 돼요.

라우트 별로 코드를 분할하면 페이지가 독립적으로 작동하게 돼요. <br/>
특정 페이지에서 오류가 발생해도, 나머지 애플리케이션은 여전히 정상 작동할 수 있어요.

또한, production 환경에서는 [Link](https://nextjs.org/docs/pages/api-reference/components/link) 컴포넌트가 브라우저 viewport 에 나타날 때마다,
Next.js 가 자동으로 해당 라우트의 코드를 백그라운드에서 미리 가져와요. <br/>
사용자가 링크를 클릭할 때, 대상 페이지의 코드는 이미 로드되어 있어 페이지 전환이 거의 즉각적으로 이루어져요! 🪄

[navigation 이 어떻게 작동하는지](https://nextjs.org/docs/app/building-your-application/routing/linking-and-navigating#how-routing-and-navigation-works)
더 알아보세요.

<hr />

## 활성 링크 표시 패턴 (Pattern: Showing active links)

일반적인 UI 패턴 중 하나는 현재 사용자가 보고 있는 페이지를 나타내기 위해 활성 링크를 표시하는 것이에요. <br/>
이를 위해서는 사용자의 현재 경로를 URL에서 가져와야 해요. <br/>
Next.js 는 이 패턴을 구현할 수 있도록 [usePathname()](https://nextjs.org/docs/app/api-reference/functions/use-pathname) 이라는 hook 을
제공합니다.

`usePathname()`은 hook 이므로 `nav-links.tsx` 파일을 클라이언트 컴포넌트로 바꿔야 해요. 🌟 <br/>
파일 상단에 React 의 `"use client"` 지시문을 추가한 후, `next/navigation` 에서 `usePathname()`을 가져와주세요.

```javascript
'use client';

import {
    UserGroupIcon,
    HomeIcon,
    InboxIcon,
} from '@heroicons/react/24/outline';
import Link from 'next/link';
import {usePathname} from 'next/navigation';

// ...
```

다음으로, `<NavLinks />` 컴포넌트 안에 `pathname` 이라는 변수에 경로를 할당해주세요.

```javascript
export default function NavLinks() {
    const pathname = usePathname();
    // ...
}
```

`clsx` 라이브러리를 사용하여 링크가 활성화되었을 때 조건부로 클래스 이름을 적용할 수
있어요. ([Chapter02. CSS Styling](https://nextjs.org/learn/dashboard-app/css-styling)) <br/>
`link.href` 가 `pathname` 과 일치하는 경우, 해당 링크는 파란색 텍스트와 연한 파란색 배경으로 표시되어야 해요.

`nav-links.tsx` 최종 코드 :

```javascript
'use client';

import {
    UserGroupIcon,
    HomeIcon,
    DocumentDuplicateIcon,
} from '@heroicons/react/24/outline';
import Link from 'next/link';
import {usePathname} from 'next/navigation';
import clsx from 'clsx';

// ...

export default function NavLinks() {
    const pathname = usePathname();

    return (
        <>
            {links.map((link) => {
                const LinkIcon = link.icon;
                return (
                    <Link
                        key={link.name}
                        href={link.href}
                        className={clsx(
                            'flex h-[48px] grow items-center justify-center gap-2 rounded-md bg-gray-50 p-3 text-sm font-medium hover:bg-sky-100 hover:text-blue-600 md:flex-none md:justify-start md:p-2 md:px-3',
                            {
                                'bg-sky-100 text-blue-600': pathname === link.href,
                            },
                        )}
                    >
                        <LinkIcon className="w-6"/>
                        <p className="hidden md:block">{link.name}</p>
                    </Link>
                );
            })}
        </>
    );
}
```

변경 사항을 저장하고 로컬호스트를 확인해 보세요. 이제 활성 링크가 파란색으로 강조 표시되어야 해요!

