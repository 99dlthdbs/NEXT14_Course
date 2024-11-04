메타데이터(metadata)는 검색 엔진 최적화(SEO)와 공유 가능성을 높이는 데 매우 중요해요.

이번 장에서는 Next.js 애플리케이션에 메타데이터를 추가하는 방법에 대해 다뤄볼 거예요.

## 이번 챕터에서는...

- 메타데이터란 무엇인지
- 메타데이터의 종류
- 메타데이터를 사용하여 Open Graph 이미지를 추가하는 방법
- 메타데이터를 사용하여 파비콘을 추가하는 방법

<hr />

## metadata 란?

웹 개발에서 메타데이터는 웹페이지에 대한 추가 정보를 제공해요. <br />
메타데이터는 페이지를 방문하는 사용자가 직접 볼 수는 없지만, HTML의 `<head>` 요소 안에 숨겨져 있어요. <br/>
검색 엔진이나 다른 시스템이 웹페이지의 내용을 더 잘 이해할 수 있도록 도움을 주는 중요한 정보랍니다.

<hr />

## 왜 metadata 가 중요할까?

메타데이터는 웹페이지의 검색 순위를 높여주고, 더 접근하기 쉽게 만들어주는 중요한 역할을 해요. <br />
적절한 메타데이터는 검색 엔진이 웹페이지를 더 잘 인덱싱할 수 있도록 도와서 검색 결과에서 페이지의 순위가 높아지게 해줘요. <br />
또한, Open Graph 같은 메타데이터는 소셜 미디어에서 링크를 공유할 때 페이지의 모양을 더 보기 좋고, 사용자에게 유익하게 만들어준답니다.

<hr />

## metadata 종류

메타데이터에는 여러 종류가 있고, 각각의 목적이 다르답니다. 그중에서 자주 쓰이는 메타데이터에는 다음과 같은 것들이 있어요:

**title metadata** :
웹페이지의 제목을 정하는 메타데이터예요. <br />
브라우저 탭에 표시되는 제목을 나타내며, 검색 엔진이 웹페이지가 무엇에 관한 것인지 이해하는 데 중요한 역할을 해요. <br />
SEO에도 아주 중요한 요소예요.

```html
<title>Page Title</title>
```

**description metadata**: 이 메타데이터는 웹페이지의 내용을 간략하게 요약해 보여줘요.  <br/>
주로 검색 엔진 결과에 표시되어 사용자들이 페이지의 내용을 미리 파악할 수 있도록 도와준답니다.

```html

<meta name="description" content="A brief description of the page content."/>
```

**keyword metadata**: 이 메타데이터는 웹페이지 내용과 관련된 주요 키워드들을 포함하고 있어요. <br />
검색 엔진이 페이지를 더 잘 인덱싱할 수 있도록 도와준답니다.

```html

<meta name="keywords" content="keyword1, keyword2, keyword3"/>
```

**open graph metadata**: 이 메타데이터는 웹페이지가 소셜 미디어 플랫폼에서 공유될 때 어떻게 보이는지를 향상시켜줘요. <br />
예를 들어, 제목, 설명, 미리보기 이미지 같은 정보를 제공해서 사람들이 더 쉽게 내용을 이해하고 관심을 가지도록 도와준답니다.

예를 들어, 블로그 글이나 제품 페이지가 소셜 미디어에 공유될 때, Open Graph를 통해 잘 꾸며진 제목, 설명, 이미지가 표시되면 클릭률이 높아질 가능성이 커진답니다.

```html

<meta property="og:title" content="Title Here"/>
<meta property="og:description" content="Description Here"/>
<meta property="og:image" content="image_url_here"/>
```

**favicon metadata**: 이 메타데이터는 파비콘(작은 아이콘)을 웹페이지에 연결해줘요. <br />
이 아이콘은 브라우저의 주소창이나 탭에서 보여져서, 사용자가 여러 탭 중에서도 웹사이트를 쉽게 구별할 수 있도록 도와준답니다.

```html

<link rel="icon" href="path/to/favicon.ico"/>
```

<hr />

## metadata 추가하기

Next.js 에는 애플리케이션의 메타데이터를 정의하기 위한 메타데이터 API가 있어요. <br />
메타데이터를 애플리케이션에 추가하는 방법은 두 가지가 있어요:

1. **구성 기반 (Config-based)**: `layout.js` 또는 `page.js`
   파일에서 [정적 메타데이터 객체(static
   `metadata` object)](https://nextjs.org/docs/app/api-reference/functions/generate-metadata#metadata-object)
   를 내보내거나 동적으로 [메타데이터를 생성하는 함수(
   `generateMetadata` function)](https://nextjs.org/docs/app/api-reference/functions/generate-metadata#generatemetadata-function)
   를 작성할 수 있어요.
2. **파일 기반 (File-based)**: Next.js 에는 메타데이터 전용으로 사용되는 특별한 파일들이 여러 가지 있어요:
    - `favicon.ico`, `apple-icon.jpg`, `icon.jpg`: 파비콘과 아이콘을 위해 사용해요.
    - `opengraph-image.jpg`와 `twitter-image.jpg`: 소셜 미디어 이미지를 위해 사용해요.
    - `robots.txt`: 검색 엔진 크롤링에 대한 지침을 제공해요.
    - `sitemap.xml`: 웹사이트의 구조에 대한 정보를 제공해요.

이 파일들을 사용하여 정적 메타데이터를 사용할 수 있고, 또는 프로젝트 내에서 프로그래밍 방식으로 생성할 수도 있어요. <br />
이 두 가지 방법을 사용하면 Next.js는 페이지에 대한 적절한 `<head>` 요소를 자동으로 생성해줘요.

### Favicon & Open Graph image

1. **/public 폴더에 있는 이미지 확인하기**: 여기에는 `favicon.ico`와 `opengraph-image.jpg` 두 개의 이미지 파일이 있어요.

2. **이미지 이동하기**: 이 두 이미지를 `/app` 폴더의 루트로 옮겨주세요.

3. **자동 인식 확인하기**: 이렇게 하면 Next.js가 자동으로 이 파일들을 파비콘과 OG 이미지로 인식하고 사용해요. <br />
   이 과정이 제대로 이루어졌는지 확인하려면, 개발자 도구에서 애플리케이션의 `<head>`
   요소를 확인해보면 돼요.

> **알아두면 좋은 것**
>
> [`ImageResponse`](https://nextjs.org/docs/app/api-reference/functions/image-response) 생성자를 사용하면 동적인 OG 이미지를 만들 수 있어요.
>
> 이 방법을 사용하면 페이지의 내용에 따라 자동으로 이미지를 생성할 수 있어서, 웹사이트에 더 맞춤화된 메타데이터를 추가할 수 있어요. 이렇게 하면 방문하는 사용자가 보게 되는 이미지를 상황에 맞게 변경할 수
> 있답니다.

### Page title & descriptions

각 페이지에 제목과 설명 같은 추가 정보를 넣으려면, 어떤 layout.js나 page.js 파일에서 메타데이터 객체를 포함할 수 있어요. <br />
layout.js에 있는 메타데이터는 그 레이아웃을 사용하는 모든 페이지에서 상속받아요.

루트 레이아웃에 다음 필드를 포함하는 새로운 메타데이터 객체를 만들어 보세요:

```typescript
import {Metadata} from 'next';

export const metadata: Metadata = {
    title: 'Acme Dashboard',
    description: 'The official Next.js Course Dashboard, built with App Router.',
    metadataBase: new URL('https://next-learn-dashboard.vercel.sh'),
};

export default function RootLayout() {
    // ...
}
```

Next.js는 자동으로 애플리케이션에 제목과 메타데이터를 추가해요. <br />
그런데 특정 페이지에 대해 맞춤 제목을 추가하고 싶다면, 페이지 자체에 메타데이터 객체를 추가하면 돼요. <br />
중첩된 페이지에서의 메타데이터는 부모의 메타데이터를 덮어쓰게 돼요.

예를 들어, `/dashboard/invoices` 페이지에서는 페이지 제목을 업데이트할 수 있어요.

```typescript
import {Metadata} from 'next';

export const metadata: Metadata = {
    title: 'Invoices | Acme Dashboard',
};
```

이 방식은 잘 작동하지만, 매 페이지에서 애플리케이션 제목을 반복하고 있어요. <br />
만약 회사 이름과 같은 정보가 바뀌면, 모든 페이지를 업데이트해야 해요.

대신, 메타데이터 객체의 title.template 필드를 사용하여 페이지 제목의 템플릿을 정의할 수 있어요. <br />
이 템플릿에는 페이지 제목과 함께 포함하고 싶은 다른 정보도 넣을 수 있어요.

루트 레이아웃에서 메타데이터 객체를 업데이트하여 템플릿을 포함하세요.

```typescript
import {Metadata} from 'next';

export const metadata: Metadata = {
    title: {
        template: '%s | Acme Dashboard',
        default: 'Acme Dashboard',
    },
    description: 'The official Next.js Learn Dashboard built with App Router.',
    metadataBase: new URL('https://next-learn-dashboard.vercel.sh'),
};
```

템플릿의 %s는 특정 페이지 제목으로 대체될 거예요.

이제 `/dashboard/invoices` 페이지에서 페이지 제목을 추가할 수 있어요.

```typescript
export const metadata: Metadata = {
    title: 'Invoices',
};
```

`/dashboard/invoices` 페이지로 이동해서 <head> 요소를 확인해 보세요. 이제 페이지 제목이 "Invoices | Acme Dashboard"로 표시될 거예요.

<hr />

## 실습 : metadata 추가하기

이제 메타데이터에 대해 배웠으니, 다른 페이지에 제목을 추가해 보세요:

- /login 페이지
- /dashboard 페이지
- /dashboard/customers 페이지
- /dashboard/invoices/create 페이지
- /dashboard/invoices/[id]/edit 페이지

Next.js 메타데이터 API는 강력하고 유연해서 애플리케이션의 메타데이터를 완전히 제어할 수 있게 해 줘요. 여기서는 기본적인 메타데이터 추가 방법을 보여드렸지만, `keywords`, `robots`,
`canonical` 등 여러 필드를
추가할 수 있어요. [문서](https://nextjs.org/docs/app/api-reference/functions/generate-metadata)를 탐색해 보면서 원하는 추가 메타데이터를 애플리케이션에
추가해 보세요.
