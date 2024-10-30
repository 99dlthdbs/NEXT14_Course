지난 챕터에서는 인보이스 경로를 완성하고 폼 검증과 접근성을 개선했어요. <br/>
이번 챕터에서는 대시보드에 인증 기능을 추가해볼 거예요.

## 이번 챕터에서는...

- 인증이란 무엇인지
- NextAuth.js를 사용하여 앱에 인증 추가하는 방법
- 미들웨어를 사용하여 사용자를 리디렉션하고 경로를 보호하는 방법
- React의 `useActionState`를 사용하여 대기 상태와 폼 오류 처리하기

<hr />

## 인증(authentication)이란?

인증은 오늘날 많은 웹 애플리케이션의 중요한 부분이에요. <br />
인증은 시스템이 사용자가 자신이 주장하는 사람인지 확인하는 방법이에요.

안전한 웹사이트는 종종 사용자의 신원을 확인하기 위해 여러 가지 방법을 사용해요. <br/>
예를 들어, 사용자가 사용자 이름과 비밀번호를 입력한 후, 사이트는 사용자의 기기로 인증 코드가 포함된 메시지를 보낼 수도 있고, Google Authenticator와 같은 외부 앱을 사용할 수도 있어요.

이렇게 두 가지 인증 방식(2단계 인증, 2FA)을 사용하면 보안이 강화돼요. <br />
누군가가 사용자의 비밀번호를 알게 되더라도, 고유한 토큰이 없으면 계정에 접근할 수 없어요.

### Authentication vs. Authorization

웹 개발에서 인증(authentication)과 권한 부여(authorization)는 서로 다른 역할을 해요:

- **인증(authentication)** 은 사용자가 자신이 주장하는 사람인지 확인하는 과정이에요. 즉, 사용자 이름과 비밀번호 같은 정보를 통해 신원을 증명하는 거예요.

- **권한 부여(authorization)** 는 그 다음 단계에요. 사용자의 신원이 확인되면, 권한 부여는 사용자가 애플리케이션의 어떤 부분을 사용할 수 있는지를 결정해요.

그래서 인증은 당신이 누구인지 확인하고, 권한 부여는 애플리케이션에서 무엇을 할 수 있는지를 정하는 과정이에요.

<hr />

## login route 만들기

`/login` 이라는 새로운 route 를 만들고 아래의 코드를 넣어주세요 :

```javascript
import AcmeLogo from '@/app/ui/acme-logo';
import LoginForm from '@/app/ui/login-form';

export default function LoginPage() {
    return (
        <main className="flex items-center justify-center md:h-screen">
            <div className="relative mx-auto flex w-full max-w-[400px] flex-col space-y-2.5 p-4 md:-mt-32">
                <div className="flex h-20 w-full items-end rounded-lg bg-blue-500 p-3 md:h-36">
                    <div className="w-32 text-white md:w-36">
                        <AcmeLogo/>
                    </div>
                </div>
                <LoginForm/>
            </div>
        </main>
    );
}
```

페이지에는 `<LoginForm />`이 임포트되어 있다는 것을 알 수 있어요. 이 부분은 나중에 다룰 예정이에요.

<hr />

## NextAuth.js

우리는 [NextAuth.js](https://authjs.dev/reference/nextjs)를 사용하여 애플리케이션에 인증 기능을 추가할 거예요.

NextAuth.js는 세션 관리, 로그인 및 로그아웃과 같은 인증 관련 복잡한 부분을 쉽게 처리할 수 있게 도와줍니다. <br />
이러한 기능을 수동으로 구현할 수도 있지만, 그 과정이 시간이 많이 걸리고 오류가 발생하기 쉽거든요.

NextAuth.js는 이러한 과정을 단순화하여 Next.js 애플리케이션에서 사용할 수 있는 통합 솔루션을 제공해요.

<hr />

## NextAuth.js 설정하기

터미널에서 다음 명령어를 실행하여 NextAuth.js를 설치해 주세요.

```shell
pnpm i next-auth@beta
```

여기서는 Next.js 14와 호환되는 NextAuth.js의 베타 버전을 설치하는 중이에요.

다음으로, 애플리케이션을 위한 비밀 키를 생성해야 해요. <br />
이 키는 쿠키를 암호화하는 데 사용되어 사용자 세션의 보안을 보장해 줍니다.

터미널에서 다음 명령어를 실행하여 이 키를 생성할 수 있어요.

```shell
openssl rand -base64 32
```

그런 다음, .env 파일에 생성한 키를 AUTH_SECRET 변수에 추가해야 해요.

```dotenv
AUTH_SECRET=your-secret-key
```

프로덕션에서 인증이 작동하려면, Vercel 프로젝트의 환경 변수를 업데이트해야 해요.

[환경 변수를 추가하는 방법에 대한 가이드](https://vercel.com/docs/projects/environment-variables)를 확인해보면 도움이 될 거예요.

### pages option 추가하기

프로젝트의 루트에 `auth.config.ts`라는 파일을 만들어서, NextAuth.js의 설정 옵션을 포함하는 `authConfig` 객체를 내보내도록 해요.

처음에는 이 객체에 `pages` 옵션만 포함될 거예요.

```typescript
import type {NextAuthConfig} from 'next-auth';

export const authConfig = {
    pages: {
        signIn: '/login',
    },
} satisfies NextAuthConfig;

// 글쓴이는 여기에서 type error 가 발생하여 Partial<NextAuthConfig> 로 사용하였다.
```

`pages` 옵션을 사용하면 사용자 지정 로그인, 로그아웃 및 오류 페이지의 경로를 지정할 수 있어요.

이것은 필수는 아니지만, `signIn: '/login'`을 `pages` 옵션에 추가하면 사용자가 NextAuth.js의 기본 페이지가 아니라 우리가 만든 로그인 페이지로 리디렉션될 거예요.

<hr />

## Next.js 미들웨어로 routes 보호하기

다음으로, 사용자가 로그인하지 않은 경우 대시보드 페이지에 접근할 수 없도록 경로를 보호하는 로직을 추가할 거예요.

이렇게 하면, 로그인한 사용자만 대시보드에 들어갈 수 있게 됩니다.

```typescript
import type {NextAuthConfig} from 'next-auth';

export const authConfig = {
    pages: {
        signIn: '/login',
    },
    callbacks: {
        authorized({auth, request: {nextUrl}}) {
            const isLoggedIn = !!auth?.user;
            const isOnDashboard = nextUrl.pathname.startsWith('/dashboard');
            if (isOnDashboard) {
                if (isLoggedIn) return true;
                return false; // Redirect unauthenticated users to login page
            } else if (isLoggedIn) {
                return Response.redirect(new URL('/dashboard', nextUrl));
            }
            return true;
        },
    },
    providers: [], // Add providers with an empty array for now
} satisfies NextAuthConfig;

// 여기에서 코드를 추가하니 위에 발생했던 type error 가 사라졌다. Partial 없애도 괜찮다.
```

인증된 콜백은 [Next.js 미들웨어](https://nextjs.org/docs/app/building-your-application/routing/middleware)를 통해 요청이 페이지에 접근할 수 있는
권한이 있는지 확인하는 데 사용돼요. <br />
이 콜백은 요청이 완료되기 전에 호출되며, auth와 request 속성이 포함된 객체를 받게 돼요. <br />
auth 속성에는 사용자의 세션 정보가 들어 있고, request 속성에는 들어오는 요청 정보가 담겨 있어요.

providers 옵션은 다양한 로그인 옵션을 나열하는 배열이에요. <br />
현재는 NextAuth 설정을 충족시키기 위해 빈 배열로 두게 돼요. <br/>
[Credentials provider](https://nextjs.org/learn/dashboard-app/adding-authentication#adding-the-credentials-provider)를
추가하는 부분에서 더
자세히 배울 수 있을 거예요.

다음으로, authConfig 객체를 미들웨어 파일에 가져와야 해요. 프로젝트의 루트에 `middleware.ts`라는 파일을 만들고, 아래의 코드를 붙여넣으면 돼요.

```typescript
import NextAuth from 'next-auth';
import {authConfig} from './auth.config';

export default NextAuth(authConfig).auth;

export const config = {
    // https://nextjs.org/docs/app/building-your-application/routing/middleware#matcher
    matcher: ['/((?!api|_next/static|_next/image|.*\\.png$).*)'],
};
```

여기서는 `authConfig` 객체로 NextAuth.js를 초기화하고 `auth` 속성을 내보내고 있어요. <br />
또한 미들웨어의 `matcher` 옵션을 사용하여 특정 경로에서만 실행되도록 설정해요.

미들웨어를 사용하면 인증 확인이 완료될 때까지 보호된 경로가 렌더링을 시작하지 않아요.  <br />
이렇게 하면 애플리케이션의 보안과 성능이 모두 향상돼요.

### 비밀번호 해싱 (Password hashing)

비밀번호를 데이터베이스에 저장하기 전에 해시하는 것이 좋은 습관이에요. <br />
해시는 비밀번호를 고정 길이의 문자열로 변환하여 데이터가 노출되더라도 보안 계층을 추가해줘요.

`seed.js` 파일에서 `bcrypt` 패키지를 사용하여 사용자의 비밀번호를 해시한 다음 데이터베이스에 저장했어요. <br />
이번 챕터에서는 사용자가 입력한 비밀번호가 데이터베이스에 저장된 비밀번호와 일치하는지 비교할 때도 `bcrypt`를 사용할 거예요. <br />
하지만 `bcrypt`는 Next.js 미들웨어에서 사용할 수 없는 Node.js API에 의존하기 때문에 `bcrypt` 패키지를 위한 별도의 파일을 만들어야 해요.

새로운 파일을 `auth.ts`라고 이름 지어 생성하고, 그 안에서 `authConfig` 객체를 확장해 주세요.

### Credentials provider 추가하기

이제 NextAuth.js에 `providers` 옵션을 추가해야 해요. <br />
`providers`는 다양한 로그인 옵션을 목록으로 나열할 수 있는 배열인데, 예를 들어 Google이나 GitHub 같은 외부 인증 서비스가 포함될 수 있어요. <br />
이 과정에서는 [`Credentials provider`](https://authjs.dev/getting-started/providers/credentials)만 사용할 거예요.

`Credentials provider`는 사용자가 사용자 이름과 비밀번호를 사용하여 로그인할 수 있도록 해줘요.

```typescript
import NextAuth from 'next-auth';
import {authConfig} from './auth.config';
import Credentials from 'next-auth/providers/credentials';

export const {auth, signIn, signOut} = NextAuth({
    ...authConfig,
    providers: [Credentials({})],
});
```

> **알아두면 좋은 점**
>
> `Credentials provider`를 사용하고 있지만, 일반적으로는 [OAuth](https://authjs.dev/getting-started/authentication/oauth)
> 나 [이메일](https://authjs.dev/getting-started/authentication/email) 제공자 같은
> 다른 인증 방식을 사용하는 것이 권장돼요.
>
> [NextAuth.js](https://authjs.dev/getting-started/authentication/oauth) 문서를 참고하면 사용할 수 있는 모든 옵션 목록을 확인할 수 있어요.

### 로그인 기능 추가하기

`authorize` 함수를 사용하면 인증 로직을 처리할 수 있어요. <br />
서버 액션과 비슷하게 `zod`를 사용하여 이메일과 비밀번호를 검증하고, 데이터베이스에 해당 사용자가 있는지 확인할 수 있답니다.

```typescript
import NextAuth from 'next-auth';
import {authConfig} from './auth.config';
import Credentials from 'next-auth/providers/credentials';
import {z} from 'zod';

export const {auth, signIn, signOut} = NextAuth({
    ...authConfig,
    providers: [
        Credentials({
            async authorize(credentials) {
                const parsedCredentials = z
                    .object({email: z.string().email(), password: z.string().min(6)})
                    .safeParse(credentials);
            },
        }),
    ],
});
```

자격 증명(validating the credentials)이 확인되면, 데이터베이스에서 사용자를 조회하는 새로운 `getUser` 함수를 만들어보세요.

```typescript
import NextAuth from 'next-auth';
import Credentials from 'next-auth/providers/credentials';
import {authConfig} from './auth.config';
import {z} from 'zod';
import {sql} from '@vercel/postgres';
import type {User} from '@/app/lib/definitions';
import bcrypt from 'bcrypt';

async function getUser(email: string): Promise<User | undefined> {
    try {
        const user = await sql<User>`SELECT * FROM users WHERE email=${email}`;
        return user.rows[0];
    } catch (error) {
        console.error('Failed to fetch user:', error);
        throw new Error('Failed to fetch user.');
    }
}

export const {auth, signIn, signOut} = NextAuth({
    ...authConfig,
    providers: [
        Credentials({
            async authorize(credentials) {
                const parsedCredentials = z
                    .object({email: z.string().email(), password: z.string().min(6)})
                    .safeParse(credentials);

                if (parsedCredentials.success) {
                    const {email, password} = parsedCredentials.data;
                    const user = await getUser(email);
                    if (!user) return null;
                }

                return null;
            },
        }),
    ],
});
```

그런 다음, `bcrypt.compare` 함수를 호출하여 비밀번호가 일치하는지 확인해보세요.

```typescript
import NextAuth from 'next-auth';
import Credentials from 'next-auth/providers/credentials';
import {authConfig} from './auth.config';
import {sql} from '@vercel/postgres';
import {z} from 'zod';
import type {User} from '@/app/lib/definitions';
import bcrypt from 'bcrypt';

// ...

export const {auth, signIn, signOut} = NextAuth({
    ...authConfig,
    providers: [
        Credentials({
            async authorize(credentials) {
                // ...

                if (parsedCredentials.success) {
                    const {email, password} = parsedCredentials.data;
                    const user = await getUser(email);
                    if (!user) return null;
                    const passwordsMatch = await bcrypt.compare(password, user.password);

                    if (passwordsMatch) return user;
                }

                console.log('Invalid credentials');
                return null;
            },
        }),
    ],
});
```

비밀번호가 일치하면 사용자를 반환하고, 그렇지 않으면 `null`을 반환하여 사용자가 로그인하지 못하도록 막아요.

### 로그인 form 업데이트 하기

이제 로그인 폼과 인증 로직을 연결해야 해요.

`actions.ts` 파일에서 `authenticate`라는 새로운 액션을 만들고, 이 액션에서 `auth.ts`에서 `signIn` 함수를 가져와야 해요.

```javascript
'use server';

import {signIn} from '@/auth';
import {AuthError} from 'next-auth';

// ...

export async function authenticate(
    prevState: string | undefined,
    formData: FormData,
) {
    try {
        await signIn('credentials', formData);
    } catch (error) {
        if (error instanceof AuthError) {
            switch (error.type) {
                case 'CredentialsSignin':
                    return 'Invalid credentials.';
                default:
                    return 'Something went wrong.';
            }
        }
        throw error;
    }
}
```

'CredentialsSignin' 오류가 발생하면 적절한 오류 메시지를 표시하고 싶어요. <br />
NextAuth.js의 오류에 대한 자세한 내용은 [문서](https://authjs.dev/reference/core/errors)에서 확인할 수 있어요.

마지막으로, `login-form.tsx` 컴포넌트에서는 React의 `useActionState`를 사용하여 서버 액션을 호출하고, 폼 오류를 처리하며, 폼의 대기 상태를 표시할 수 있어요.

```javascript
'use client';

import {lusitana} from '@/app/ui/fonts';
import {
    AtSymbolIcon,
    KeyIcon,
    ExclamationCircleIcon,
} from '@heroicons/react/24/outline';
import {ArrowRightIcon} from '@heroicons/react/20/solid';
import {Button} from '@/app/ui/button';
import {useActionState} from 'react';
import {authenticate} from '@/app/lib/actions';

export default function LoginForm() {
    const [errorMessage, formAction, isPending] = useActionState(
        authenticate,
        undefined,
    );

    return (
        <form action={formAction} className="space-y-3">
            <div className="flex-1 rounded-lg bg-gray-50 px-6 pb-4 pt-8">
                <h1 className={`${lusitana.className} mb-3 text-2xl`}>
                    Please log in to continue.
                </h1>
                <div className="w-full">
                    <div>
                        <label
                            className="mb-3 mt-5 block text-xs font-medium text-gray-900"
                            htmlFor="email"
                        >
                            Email
                        </label>
                        <div className="relative">
                            <input
                                className="peer block w-full rounded-md border border-gray-200 py-[9px] pl-10 text-sm outline-2 placeholder:text-gray-500"
                                id="email"
                                type="email"
                                name="email"
                                placeholder="Enter your email address"
                                required
                            />
                            <AtSymbolIcon
                                className="pointer-events-none absolute left-3 top-1/2 h-[18px] w-[18px] -translate-y-1/2 text-gray-500 peer-focus:text-gray-900"/>
                        </div>
                    </div>
                    <div className="mt-4">
                        <label
                            className="mb-3 mt-5 block text-xs font-medium text-gray-900"
                            htmlFor="password"
                        >
                            Password
                        </label>
                        <div className="relative">
                            <input
                                className="peer block w-full rounded-md border border-gray-200 py-[9px] pl-10 text-sm outline-2 placeholder:text-gray-500"
                                id="password"
                                type="password"
                                name="password"
                                placeholder="Enter password"
                                required
                                minLength={6}
                            />
                            <KeyIcon
                                className="pointer-events-none absolute left-3 top-1/2 h-[18px] w-[18px] -translate-y-1/2 text-gray-500 peer-focus:text-gray-900"/>
                        </div>
                    </div>
                </div>
                <Button className="mt-4 w-full" aria-disabled={isPending}>
                    Log in
                    <ArrowRightIcon className="ml-auto h-5 w-5 text-gray-50"/>
                </Button>
                <div
                    className="flex h-8 items-end space-x-1"
                    aria-live="polite"
                    aria-atomic="true"
                >
                    {errorMessage && (
                        <>
                            <ExclamationCircleIcon className="h-5 w-5 text-red-500"/>
                            <p className="text-sm text-red-500">{errorMessage}</p>
                        </>
                    )}
                </div>
            </div>
        </form>
    );
}
```

<hr />

## 로그아웃 기능 추가하기

로그아웃 기능을 `<SideNav />`에 추가하려면, `<form>` 요소 안에서 `auth.ts`의 `signOut` 함수를 호출하면 돼요.

```javascript
import Link from 'next/link';
import NavLinks from '@/app/ui/dashboard/nav-links';
import AcmeLogo from '@/app/ui/acme-logo';
import {PowerIcon} from '@heroicons/react/24/outline';
import {signOut} from '@/auth';

export default function SideNav() {
    return (
        <div className="flex h-full flex-col px-3 py-4 md:px-2">
            // ...
            <div className="flex grow flex-row justify-between space-x-2 md:flex-col md:space-x-0 md:space-y-2">
                <NavLinks/>
                <div className="hidden h-auto w-full grow rounded-md bg-gray-50 md:block"></div>
                <form
                    action={async () => {
                        'use server';
                        await signOut();
                    }}
                >
                    <button
                        className="flex h-[48px] grow items-center justify-center gap-2 rounded-md bg-gray-50 p-3 text-sm font-medium hover:bg-sky-100 hover:text-blue-600 md:flex-none md:justify-start md:p-2 md:px-3">
                        <PowerIcon className="w-6"/>
                        <div className="hidden md:block">Sign Out</div>
                    </button>
                </form>
            </div>
        </div>
    );
}
```

<hr />

## 시도해보기

이제 해보세요. 다음 자격 증명을 사용하여 애플리케이션에 로그인하고 로그아웃할 수 있어요:

- 이메일: user@nextmail.com
- 비밀번호: 123456
