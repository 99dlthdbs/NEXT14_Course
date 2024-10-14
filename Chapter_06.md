대시보드 작업을 계속하기 전에 데이터가 필요해요. <br />
이번 장에서는 `@vercel/postgress`를 사용하여 `PostgreSQL` 데이터베이스를 설정할 거예요. <br/>
`PostgreSQL`에 이미 익숙하고 자신의 provider(eg. database) 를 사용하고 있다면 이 장을 건너뛰고 직접 설정해도 괜찮아요. <br/>

그럼 계속 해보아요! 😉

## 이번 챕터에서는...

- Github에 프로젝트 push 하기
- Vercel 계정을 만들고 GitHub repository 를 연결하여 즉시 미리보기 및 배포하기
- Postgres 데이터베이스를 생성하고 연결하기
- 데이터베이스에 초기 데이터 채우기

<hr />

## GitHub repository 만들기

먼저, 아직 repository를 GitHub에 올리지 않았다면 올려보세요. <br/>
이렇게 하면 데이터베이스를 설정하고 배포하는 데 더 쉬워질 거예요.

저장소를 설정하는 데 도움이
필요하다면, [GitHub의 가이드](https://docs.github.com/en/repositories/creating-and-managing-repositories/quickstart-for-repositories)
를 참고해 보세요.

> 알아두면 좋은 점
> - GitLab이나 Bitbucket과 같은 다른 Git provider를 사용할 수도 있어요.
> - GitHub이 처음이라면, 간단한 개발 워크플로를 위해 [Github Desktop App](https://github.com/apps/desktop)을 추천해 드려요.

<hr />

## Vercel 계정 만들기

[vercel.com/signup](https://vercel.com/signup) 에 방문하여 계정을 만들어 보세요. <br/>
무료 "취미" 플랜을 선택한 후, GitHub와 Vercel 계정을 연결하기 위해 "Continue with GitHub"를 선택하세요.

<hr/>

## 프로젝트 연결 및 배포



