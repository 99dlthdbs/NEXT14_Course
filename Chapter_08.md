이전 장에서는 대시보드 개요 페이지에 대한 데이터를 가져오는 방법을 배웠어요. <br/>
하지만 현재 설정의 두 가지 제한사항에 대해 간단히 이야기했어요:

1. 데이터 요청이 의도하지 않게 워터폴(waterfall)을 만들어내고 있어요.
2. 대시보드가 정적(static)이기 때문에, 데이터가 업데이트되더라도 애플리케이션에 반영되지 않아요.

이런 문제를 해결하기 위해서는 데이터를 더 효율적으로 가져오는 방법을 알아보고, 대시보드를 동적으로 업데이트할 수 있는 방법을 고민해야 해요.

## 이번 챕터에서는...

- 정적 렌더링이란 무엇이며 애플리케이션의 성능을 어떻게 개선할 수 있는지
- 동적 렌더링이란 무엇이며 언제 사용해야 하는지
- 대시보드를 동적으로 만드는 다양한 접근 방식
- 느린 데이터 요청을 시뮬레이션하여 무슨 일이 일어나는지 보기

<hr/>

## 정적 렌더링(static rendering)이란?

정적 렌더링은 데이터 가져오기와 렌더링이 서버에서 빌드 시간(배포할 때)
이나 [데이터 재검증(revalidating data)](https://nextjs.org/docs/app/building-your-application/data-fetching/fetching-caching-and-revalidating#revalidating-data)
시에 이루어지는 방식이에요.<br/>
사용자가 애플리케이션을 방문할 때마다 캐시된 결과가 제공됩니다.

정적 렌더링의 몇 가지 장점은 다음과 같아요:

1. **빠른 웹사이트**: 미리 렌더링된 콘텐츠는 캐시되어 전 세계에 분산될 수 있어요. 이렇게 하면 사용자들이 더 빠르고 안정적으로 웹사이트의 콘텐츠에 접근할 수 있어요.

2. **서버 부하 감소**: 콘텐츠가 캐시되기 때문에 서버는 각 사용자 요청에 대해 동적으로 콘텐츠를 생성할 필요가 없어요.

3. **SEO 향상**: 미리 렌더링된 콘텐츠는 검색 엔진 크롤러가 인덱스하기 쉬워요. 페이지가 로드될 때 콘텐츠가 이미 준비되어 있으므로 검색 엔진 순위가 개선될 수 있어요.

정적 렌더링은 데이터가 없거나 사용자 간에 공유되는 데이터가 있는 UI에 유용해요. <br/>
예를 들어, 정적 블로그 게시물이나 제품 페이지가 이에 해당해요. <br/>
그러나 개인화된 데이터가 자주 업데이트되는 대시보드에는 적합하지 않을 수 있어요.

정적 렌더링의 반대 개념은 **동적 렌더링**이에요.

<hr />

## 동적 렌더링(dynamic rendering)이란?

동적 렌더링은 각 사용자가 페이지를 요청할 때(즉, 사용자가 페이지를 방문할 때) 서버에서 콘텐츠를 렌더링하는 방식이에요.

동적 렌더링의 몇 가지 장점은 다음과 같아요:

- **실시간 데이터**: 동적 렌더링은 애플리케이션이 실시간 또는 자주 업데이트되는 데이터를 표시할 수 있게 해줘요. 이는 데이터가 자주 변경되는 애플리케이션에 이상적이에요.
- **사용자 맞춤형 콘텐츠**: 대시보드나 사용자 프로필과 같은 개인화된 콘텐츠를 제공하기가 더 쉬워요. 사용자 상호작용에 따라 데이터를 업데이트할 수 있습니다.
- **요청 시간 정보**: 동적 렌더링은 요청 시간에만 알 수 있는 정보(예: 쿠키나 URL search params)에 접근할 수 있어요.

<hr />

## 느린 데이터 가져오기 시뮬레이션

우리가 만들고 있는 대시보드 애플리케이션은 동적이에요. <br />
하지만 이전 장에서 언급한 한 가지 문제가 있어요. <br />
만약 하나의 데이터 요청이 다른 요청보다 느리면 어떻게 될까요?

이제 느린 데이터 가져오기를 시뮬레이션해볼까요? <br />
`data.ts` 파일에서 `fetchRevenue()` 안의 `console.log`와 `setTimeout`을 주석 해제해 주세요. <br />
이렇게 하면, 데이터 요청 중 하나가 느리게 응답하는 상황을 만들 수 있어요.

이렇게 설정하면, 느린 데이터 요청으로 인해 대시보드의 다른 데이터가 로드되기를 기다려야 하므로, 사용자 경험에 어떤 영향을 미치는지 확인할 수 있을 거예요.

```javascript
export async function fetchRevenue() {
    try {
        // We artificially delay a response for demo purposes.
        // Don't do this in production :)
        console.log('Fetching revenue data...');
        await new Promise((resolve) => setTimeout(resolve, 3000));

        const data = await sql < Revenue > `SELECT * FROM revenue`;

        console.log('Data fetch completed after 3 seconds.');

        return data.rows;
    } catch (error) {
        console.error('Database Error:', error);
        throw new Error('Failed to fetch revenue data.');
    }
}
```

이제 새 탭에서 http://localhost:3000/dashboard/를 열어보세요. <br/>
페이지가 로드되는 데 시간이 더 걸리는 것을 확인할 수 있을 거예요. <br/>
그리고 터미널에서는 다음과 같은 메시지도 볼 수 있어요:

```shell
Fetching revenue data...
Data fetch completed after 3 seconds.
```

여기서 인위적으로 3초의 지연을 추가하여 느린 데이터 가져오기를 시뮬레이션했어요.<br/>
그 결과로, 데이터가 가져와지는 동안 방문자에게 UI를 보여주는 전체 페이지가 차단되었답니다. <br/>
이는 개발자들이 해결해야 하는 일반적인 문제를 가져오죠:

동적 렌더링을 사용할 경우, 여러분의 애플리케이션은 **가장 느린 데이터 요청 속도**에 맞춰져 있다는 점이에요.


