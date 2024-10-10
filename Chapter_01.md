# 새로운 프로젝트 만들기

우리는 **pnpm**을 패키지 매니저로 사용하는 것을 추천해요. **pnpm**은 **npm**이나 **yarn**보다 더 빠르고 효율적이에요.
만약 **pnpm**이 설치되어 있지 않다면, 아래 명령어를 실행해서 컴퓨터에 **pnpm**을 설치할 수 있어요.

```
npm install -g pnpm
```
이렇게 하면 프로젝트를 더 빠르고 효율적으로 관리할 수 있게 돼요!

<details>
<summary style="color: skyblue">
저도 npm이나 yarn보다 pnpm을 사용하는 것을 더 선호해요! <br/>
그 이유가 해당 문서에 자세히 설명되어 있지 않아 제가 대신 적어볼게요. ☺️
</summary>

### **pnpm**이 npm이나 yarn보다 빠르고 효율적인 이유
1. **효율적인 패키지 저장** : <br/> 
pnpm은 패키지를 저장하는 독특한 방법을 사용해요. 패키지를 매번 복사하는 대신, 모든 패키지를 global 저장소에 보관하고, 필요한 프로젝트에 링크만 걸어요. 이렇게 하면, 디스크 공간을 아끼고 설치 시간을 줄일 수 있어요.
2. **엄격한 패키지 관리** : <br/>
pnpm은 모든 프로젝트가 정확히 같은 패키지 버전을 사용하게 해요. 이렇게 하면 여러 버전의 같은 패키지가 설치되는 "의존성 문제"를 피할 수 있고, 성능도 향상돼요.
3. **더 나은 캐싱** : <br/>
pnpm은 캐싱 시스템이 효율적이에요. 한 번 다운로드한 패키지는 글로벌 저장소에 저장돼서, 다른 프로젝트에서도 재사용할 수 있어요. 그래서 다음에 설치할 때 더 빨라요.
4. **네트워크 요청 감소** : <br/>
pnpm은 이미 다운로드한 패키지를 사용하기 때문에, 새로운 패키지를 다운로드하기 위해 네트워크에 요청할 필요가 없어요. 그래서 설치가 더 빨라져요.
5. **속도와 성능** : <br/>
symlinking(링크 연결)과 캐싱 덕분에 pnpm은 종종 npm이나 yarn보다 더 빠른 의존성을 해결해요. 특히 많은 패키지를 사용하는 큰 프로젝트에서 이 속도 차이가 더 두드러져요.

이러한 이유들 덕분에 pnpm은 프로젝트의 패키지를 관리하는 데 더 효율적이고 빠른 선택이 돼요!

</details>

Next.js 앱을 만들려면, 터미널을 열고 프로젝트를 저장할 폴더로 이동한 후, 다음 명령을 실행하면 돼요 : 
```
npx create-next-app@latest nextjs-dashboard --example "https://github.com/vercel/next-learn/tree/main/dashboard/starter-example" --use-pnpm
```

**create-next-app** : Next.js 애플ㄹ케이션을 자동으로 설정해주는 **명령줄 도구(CLI)** 예요.
**--example** : 이 코스를 위한 예제 스타터 템플릿을 가져와서 프로젝트를 시작하는 거예요.

쉽게 말하면, 이 명령어는 Next.js 프로젝트를 빠르게 만들고, 수업에 맞는  예시 프로젝트를 가져오는 역할을 해요.

<hr />

## 프로젝트 탐색하기

이번 코스는 처음부터 코드를 직접 작성하는 튜토리얼과는 달라요. 코스에 필요한 코드 대부분이 이미 작성되어 있어요.
실제 개발 환경에서는 이미 존재하는 코드베이스에서 작업할 일이 많기 때문에, 더 현실적인 상황을 반영하고 있어요.

우리의 목표는 애플리케이션 전체 코드를 작성하는 대신, **Next.js** 의 주요 기능을 배우는 데 집중할 수 있도록 도와주는 거예요.

설치가 끝나면, 코드를 편집할 수 있는 프로그램(코드 에디터)에서 프로젝트를 열고 **nextjs-dashboard** 폴더로 이동해 보세요.
```
cd nextjs-dashboard
```

프로젝트를 한 번 살펴볼까요?

### 폴더 구조

프로젝트를 열어보면 다음과 같은 폴더 구조가 있다는 걸 알 수 있을 거예요.
<img src='https://nextjs.org/_next/image?url=%2Flearn%2Fdark%2Flearn-folder-structure.png&w=3840&q=75' alt='folder structure' />

- **/app** : 애플리케이션의 모든 경로(페이지), components, 그리고 로직이 담겨있는 폴더예요. 주로 이곳에서 작업하게 될 거예요.
- **/app/lib** : 애플리케이션에서 사용하는 함수들이 들어 있어요. 예를 들어, 재사용할 수 있는 유틸리티 함수나 데이터를 가져오는 함수들이 여기에 있어요.
- **/app/ui** : 애플리케이션의 UI(사용자 인터페이스) components 들이 담긴 폴더예요. cards, tables, forms 같은 것들이 여기에 있고, 시간을 절약할 수 이쏘록 미리 스타일링 되어 있어요.
- **/public** : 이미지 같은 정적인 파일들이 저장되는 곳이에요.
- **Config 파일** : 프로젝트의 root directory 에는 **next.config.js** 같은 설정 파일들도 있어요. 이런 파일들은 새로운 프로젝트를 만들 때 자동으로 생성되고 설정돼요. 이 코드에서는 따로 수정할 필요가 없어요.

폴더를 자유롭게 탐색해도 괜찮고, 아직 모든 코드를 이해하지 못해도 걱정하지 마세요!

### Placeholder 데이터

UI를 만들 때, 미리 사용할 수 있는 데이터를 준비하면 도움이 돼요. <br/>
만약 데이터베이스나 API가 아직 준비되지 않았다면, 다음 방법을 사용할 수 있어요 :
- **JSON** 형식이나 Javascript 객체로 된 placeholder 데이터를 사용해요.
- **mockAPI** 같은 3rd party 서비스를 사용해요.

이번 프로젝트에서는 **app/lib/placeholder-data.ts** 파일에 일부 placeholder 데이터를 준비해두었어요.
이 파일 안에 있는 각 자바스크립트 객체는 데이터베이스의 테이블을 나타내요.
예를 들어, **invoices 테이블**을 위한 데이터가 들어있어요.

```typescript
const invoices = [
  {
    customer_id: customers[0].id,
    amount: 15795,
    status: 'pending',
    date: '2022-12-06',
  },
  {
    customer_id: customers[1].id,
    amount: 20348,
    status: 'pending',
    date: '2022-11-14',
  },
  // ...
];
```

[데이터베이스 설정](https://nextjs.org/learn/dashboard-app/setting-up-your-database)하는 챕터에서, 이 데이터를 사용해서 데이터베이스에 기본 데이터를 채워 넣을 거예요.

### TypeScript

프로젝트 파일 대부분이 ``.ts`` 나 ``.tsx`` 로 끝나는 걸 볼 수 있을 거예요.
이건 이 프로젝트가 **TypeScript** 로 작성되었기 때문이에요. 우리는 현대 웹 개발 환경을 반영한 과정을 만들고 싶었어요.

TypeScript 를 몰라도 괜찮아요. 필요한 경우에는 TypeScript 코드를 제공할 거예요.

지금은 ``/app/lib/definitions.ts`` 파일을 한 번 살펴보세요. 여기에는 데이터베이스에서 반환된 **type** 을 직접 정의해 놓았어요.
예를 들어, **invoices 테이블** 에는 다음과 같은 타입들이 있어요.

```typescript
export type Invoice = {
  id: string;
  customer_id: string;
  amount: number;
  date: string;
  // In TypeScript, this is called a string union type.
  // It means that the "status" property can only be one of the two strings: 'pending' or 'paid'.
  status: 'pending' | 'paid';
};
```

**TypeScript** 를 사용하면 잘못된 데이터 형식을 실수로 전달하지 않도록 할 수 있어요.
예를 들어, **invoice 금액**에 **숫자** 대신 **문자**를 전달하는 실수를 막을 수 잇어요.


>TypeScript 개발자라면 :
>- 우리는 데이터를 수동으로 타입을 지정하고 있지만, 더 안전한 타입 사용을 원한다면 [Prisma](https://www.prisma.io/) 나 [Drizzle](https://orm.drizzle.team/) 같은 도구를 사용하는 걸 추천해요.
이 도구들은 데이터베이스 구조에 따라 자동으로 타입을 만들어줘요.
> <br /> <br />
>- Next.js는 프로젝트에서 TypeScript를 사용하고 있는지 자동으로 감지하고, 필요한 패키지와 설정을 자동으로 설치해줘요.
또한, Next.js는 코드 작성할 때 자동완성 기능과 타입 안전성을 도와주는 [TypeScript 플러그인](https://nextjs.org/docs/app/building-your-application/configuring/typescript#typescript-plugin)도 제공해요.

<hr/>

## development 서버 실행하기

프로젝트의 패키지를 설치하려면 ``pnpm i`` 명령어를 실행하세요.
```
pnpm i
```
그 다음에는 ``pnpm dev`` 명령어를 실행해서 development 서버를 시작하세요.
```
pnpm dev
```
``pnpm dev`` 명령어를 실행하면 Next.js development 서버가 3000번 포트에서 시작됩니다. 
서버가 잘 작동하는지 확인해보세요.

브라우저에서 ``http://localhost:3000`` 을 열어보세요.
스타일은 일부러 적용되지 않은 상태이기 때문에 아래처럼 보일 거예요.

<img src='https://nextjs.org/_next/image?url=%2Flearn%2Fdark%2Facme-unstyled.png&w=3840&q=75' alt='homepage' />
