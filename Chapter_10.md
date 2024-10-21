지금까지 정적 렌더링과 동적 렌더링에 대해 배웠고, 데이터에 따라 동적 콘텐츠를 스트리밍하는 방법도 배웠어요. <br/>
이번 챕터에서는 정적 렌더링, 동적 렌더링, 스트리밍을 결합하여 같은 경로에서 사용할 수 있는 **부분 사전 렌더링 (Partial Prerendering, PPR)** 에 대해 배워볼 거예요.

> 부분 사전 렌더링(Partial Prerendering, PPR)은 Next.js 14에서 도입된 실험적 기능이에요.

## 이번 챕터에서는...

- Partial Prerendering 이 무엇인가
- Partial Prerendering 이 어떻게 동작하는가

<hr />

## 정적(Static) vs. 동적(Dynamic) Routes

최근 대부분의 웹 애플리케이션에서는 전체 애플리케이션이나 특정 경로에 대해 정적(static) 렌더링과 동적(dynamic) 렌더링 중 하나를 선택해요. <br />
Next.js에서 [동적 함수(dynamic function)](https://nextjs.org/docs/app/building-your-application/routing/route-handlers#dynamic-functions)
를 경로에서 호출하면(예를 들어, 데이터베이스 쿼리와 같은) 해당 경로 전체가 동적(dynamic)이 됩니다.

하지만 대부분의 경로는 완전히 정적이거나 동적이지 않아요. <br/>
예를 들어, [ecommerce site](https://www.partialprerendering.com/)를 생각해보면, 대부분의 제품 정보 페이지를 정적으로 렌더링하면서도, 사용자의 장바구니와 추천 제품은
동적으로 가져오는 것이 좋겠죠. <br/>
이렇게 하면 사용자에게 개인화된 콘텐츠를 보여줄 수 있어요.

대시보드 페이지를 다시 생각해보면, 어떤 구성 요소를 정적(static)으로 보고 어떤 것을 동적(dynamic)으로 볼 수 있을까요? <br/>
예를 들어, 사이드바 같은 UI 요소는 정적으로 렌더링할 수 있지만, 사용자에 따라 달라지는 차트나 알림 같은 것은 동적으로 렌더링해야 할 것입니다.

준비가 되면 아래 버튼을 클릭해서 대시보드 경로를 어떻게 나누는지 확인해보세요!

<details>
<summary>the solution</summary>

<img src="https://nextjs.org/_next/image?url=%2Flearn%2Fdark%2Fdashboard-static-dynamic-components.png&w=3840&q=75&dpl=dpl_4kWRRdpV5mEWMp9Zhahs8vP5fgBq" alt="consider static or dynamic rendering in dashboard" />
</details>

- `<SideNav>` 컴포넌트는 데이터에 의존하지 않고 사용자가 개인화되지 않기 때문에 정적(static)으로 만들 수 있어요.
- `<Page>` 안의 컴포넌트들은 자주 변경되는 데이터에 의존하고 사용자에 맞게 개인화되기 때문에 동적(dynamic)으로 만들어야 해요.

<hr />

## `Partial Prerendering`이란?

Next.js 14에서 실험적인 버전인 부분 사전 렌더링(Partial Prerendering)을 도입했어요. <br/>
이 새로운 렌더링 모델은 정적(static) 렌더링과 동적(dynamic) 렌더링의 장점을 같은 경로에서 결합할 수 있게 해준답니다.

<img src="https://nextjs.org/_next/image?url=%2Flearn%2Fdark%2Fthinking-in-ppr.png&w=3840&q=75&dpl=dpl_4kWRRdpV5mEWMp9Zhahs8vP5fgBq" alt="partial prerendering" />

사용자가 route를 방문할 때:

- 내비게이션 바와 제품 정보가 포함된 정적 경로 셸이 제공되어 빠른 초기 로드를 보장해요.
- 셸에는 장바구니와 추천 상품과 같은 동적 콘텐츠가 비동기적으로 로드될 수 있도록 구멍(hole)이 남겨져 있어요.
- 이 비동기 구멍(async hole)은 병렬로 스트리밍되어 페이지의 전체 로드 시간을 줄여준답니다.

<hr />

## `Partial Prerendering`은 어떻게 동작할까?

Partial Prerendering (PPR)은 React의 [Suspense](https://react.dev/reference/react/Suspense)를 사용하여 애플리케이션의 일부를 특정 조건이 충족될
때까지 렌더링을 미루는 방법이에요. <br/>
예를 들어 데이터가 로드될 때까지 기다릴 수 있죠.

PPR의 작동 방식은 다음과 같아요:

1. **정적 콘텐츠와 suspense fallback**: 초기 HTML 파일에 정적 콘텐츠와 함께 Suspense 폴백이 포함되어 있어요. 이를 통해 페이지가 사용자에게 빠르게 보여질 수 있게 해요.

2. **프리렌더링**: 빌드 타임(또는 재검증 시)에 정적 콘텐츠가 프리렌더링되어 정적 셸(static shell)을 생성해요. 이때 실제로 동적 콘텐츠를 렌더링하는 것은 사용자가 경로를 요청할 때까지 미뤄져요.

3. **정적과 동적 코드 경계**: 컴포넌트를 Suspense로 감싸는 것은 그 컴포넌트를 동적으로 만드는 것이 아니라, 정적 코드와 동적 코드의 경계로 사용된답니다.

이제 PPR을 대시보드 경로에 어떻게 구현할 수 있는지 살펴볼까요?

<hr />

## `Partial Prerendering` 구현하기

Next.js 앱에서 PPR을 활성화하려면 `next.config.mjs` 파일에 `ppr` 옵션을 추가해야 해요.

```javascript
/** @type {import('next').NextConfig} */

const nextConfig = {
        experimental: {
            ppr: 'incremental',
        },
    };

export default nextConfig;
```

`'incremental'` 값은 특정 경로에 대해 PPR을 도입할 수 있게 해줘요.

다음으로, 대시보드 레이아웃에 `experimental_ppr` 세그먼트 구성 옵션을 추가해야 해요.

```javascript
import SideNav from '@/app/ui/dashboard/sidenav';

export const experimental_ppr = true;

// ...
```

이제 끝이에요.
개발 중에는 애플리케이션에서 차이를 보지 못할 수도 있지만, 배포에서는 성능 향상을 느낄 수 있을 거예요. <br/>
Next.js는 경로의 정적 부분을 미리 렌더링하고, 동적 부분은 사용자가 요청할 때까지 연기해요.

`Partial Prerendering`의 좋은 점은 사용하기 위해 코드를 변경할 필요가 없다는 거예요. <br />
경로의 동적 부분을 감싸기 위해 `Suspense`를 사용하기만 하면 Next.js는 경로의 어떤 부분이 정적이고 어떤 부분이 동적인지 알 수 있어요.

PPR은 [정적 사이트와 동적 렌더링의 장점을 모두 모은 웹 애플리케이션의 기본 렌더링 모델](https://vercel.com/blog/partial-prerendering-with-next-js-creating-a-new-default-rendering-model)
이 될 가능성이 있다고 생각해요. <br />
그러나 아직 실험 단계에 있어요. 앞으로 안정화하여 Next.js로 빌드하는 기본 방식이 되기를 희망하고 있어요.

<hr />

## 요약

정리하자면, 여러분은 애플리케이션의 데이터 가져오기를 최적화하기 위해 몇 가지 작업을 했어요:

1. **서버와 데이터베이스 간의 지연 시간을 줄이기 위해 애플리케이션 코드와 동일한 지역에 데이터베이스를 만들었어요.**
    2. 애플리케이션이 실행되는 서버와 데이터베이스 서버가 물리적으로 가까운 장소에 위치해 있다는 의미예요. <br/>
       이렇게 하면 데이터 요청과 응답 사이의 지연 시간이 줄어들고, 성능이 향상될 수 있어요. <br/>
       예를 들어, 클라우드 서비스 제공업체는 여러 지역에 데이터 센터를 운영합니다. 애플리케이션을 특정 지역의 데이터 센터에서 호스팅하면, 같은 지역에 위치한 데이터베이스와의 통신이 더 빠르게 이루어져
       사용자의 경험을 개선할 수 있습니다. 이는 특히 대규모 사용자 요청이 있을 때 더욱 중요해요. <br/>
       이렇게 같은 지역에 데이터베이스를 배치하는 것은 지연(latency)을 줄이고, 데이터 전송 속도를 높이며, 전반적인 성능을 향상시키는 데 도움을 줍니다. <br/>
2. **React 서버 컴포넌트를 사용해 서버에서 데이터를 가져왔어요.** 이렇게 하면 비용이 큰(일반적으로 데이터를 가져오는 데 많은 비용이나 시간이 드는 작업을 의미) 데이터 가져오기와 로직을 서버에서 유지할
   수 있고, 클라이언트 측 JavaScript 번들을 줄이며, 데이터베이스
   비밀번호가 클라이언트에 노출되는 것을 방지할 수 있어요.
3. **SQL을 사용해 필요한 데이터만 가져왔어요.** 이렇게 하면 요청마다 전송되는 데이터 양과 메모리 내에서 데이터를 변환하는 데 필요한 JavaScript 양이 줄어들어요.
4. **JavaScript를 사용해 데이터 가져오기를 병렬로 처리했어요.** 이렇게 하면 적절한 경우에 성능을 향상시킬 수 있어요.
5. **스트리밍을 구현해 느린 데이터 요청이 전체 페이지를 막지 않도록 했고, 사용자가 모든 것이 로드되기를 기다리지 않고 UI와 상호작용을 시작할 수 있도록 했어요.**
6. **데이터 패칭이 필요한 컴포넌트로 이동시켜서 경로의 어떤 부분이 동적이어야 하는지를 분리했어요.**

다음 장에서는 데이터를 가져올 때 구현해야 할 수 있는 두 가지 일반적인 패턴인 **검색**과 **페이지네이션**에 대해 살펴볼 거예요.
