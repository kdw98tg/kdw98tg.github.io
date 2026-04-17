---
layout: post

title: "[React/Js] useSWR을 사용해서 통신 최적화 하기"

categories:
  -  Web

excerpt_image: /assets/images/posts/2026-04-17-React%20useSWR/file-20260417143911966.png

tags:
  - React
  - hooks
  - network

toc: true
toc_sticky: true

published: true

date: 2026-04-17
last_modified_at: 2026-04-17
---

## useSWR

![](/assets/images/posts/2026-04-17-React%20useSWR/file-20260417143911966.png)

프론트엔드를 구성하다 보면, 서버와 통신하여 데이터를 갱신해야 하는 일이 많습니다. 만약, 10000개의 데이터중에 바뀐게 아무것도 없는데, 갱신을 할때마다, 서버에서 데이터를 가져오게 된다면 불필요한 부하가 일어날 수 있습니다. 원래는 custom hook 을 만들어서 사용자가 캐시에 대한 처리를 했어야 했지만, Vercel 에서 만든useSWR 을 쓰면 그 과정을 개발자가 직접 구축하지 않아도 됩니다.

`useSWR`은 Stale While Revalidate의 약자로, 캐시된 데이터를 우선적으로 사용하면서 fetch 요청을 하고, 백그라운드에서 서버로부터 새 데이터를 가져오는 방식을 말합니다. 이렇게 하면, 빠른 반응성과 최신 데이터를 동시에 제공할 수 있습니다.

`useSWR`의 강력한 기능들은 다음과 같습니다.


- **요청 중복 제거**

만약, 페이지 내의 5개 컴포넌트에서 동시에 같은 API를 호출해도, useSWR은 이를 캐시 키를 기준으로 하나로 병합해서, 서버에는 단 1번만 요청을 보내게 됩니다.

- **화면 포커스 시 재검증**

사용자가 화면 포커스를 변경했다가, 다시 내 화면으로 돌아올 때, 자동으로 백그라운드에서 데이터를 최신화 합니다.

- **네트워크 회복 시 재검증**

사용자의 인터넷이 끊겼다가 다시 연결되었을 때를 감지하여 데이터를 즉시 갱신합니다. 모바일 환경에서 특히 유용합니다.

- **오프티미스틱 UI 지원**

페이스북에서 사용하는 기법으로, 페이스북에서는 좋아요를 누르면, 즉시 서버에 응답이 가지 않고, 로컬 캐시를 먼저 업데이트하여 UI를 즉각적으로 변경한 뒤, 백그라운드에서 서버와 동기화하는 기능을 사용합니다. useSWR은 이 과정을 쉽게 할 수 있도록 지원합니다.


결론적으로 개발자가 직접 복잡한 상태 관리와 캐싱 로직을 작성할 필요 없이, `useSWR`하나로 이 모든 최적화를 구현할 수 있습니다.

## 사용법

### 설치

`useSWR`은 다음의 명령어로 설치할 수 있습니다.

```bash
npm install swr
```

또한, 해당 예제는 axios를 사용해 통신하므로, axios도 설치해 줍니다. (axios 1.14.1 은 해킹 이슈가 있었기 때문에, 주의해서 다운로드 하세요.)

```bash
npm install axios
```

### 사용

#### fetcher  함수 만들기

`useSWR`은 서버와의 통신에서 캐싱처리를 도와주는 라이브러리 입니다. 서버와의 통신에서 도와준다는 말 자체가 서버와 통신을 한다는것이 깔려있기 때문에, axios를 통해 서버와 통신하겠다 라는 내용을 fetcher 함수 안에 담아줍니다.

```jsx
const fetcher = (url) => axios.get(url).then((res) => res.data);
```

#### useSWR 훅 사용하기

fetcher 함수를 등록했다면, useSWR 훅을 사용합니다.

```jsx
function UseSWR1() {
 
  const { data, error, isLoading, mutate } = useSWR(
    "https://jsonplaceholder.typicode.com/users/1",
    fetcher,
{
  // 화면을 다시 볼 때(다른 탭을 보다가 돌아오거나 창을 클릭할 때) 자동으로 데이터를 새로고침 하지 않음
  revalidateOnFocus: false, 
  // 50초(50000ms) 안에는 같은 API 요청이 여러 번 발생해도 중복해서 서버로 보내지 않고 캐시를 씀
  dedupingInterval: 50000, 
  // 3초(3000ms)마다 백그라운드에서 새 데이터를 가져옴  
  refreshInterval: 3000, 
  // 브라우저 탭이 숨겨져 있을 땐(다른 탭 볼 때) 갱신 일시정지 (선택)
  refreshWhenHidden: false, 
  // 에러 발생 시 재시도 할지 말지 (기본값: true) 
  shouldRetryOnError: true, 
  // 최대 3번까지만 재시도 
  errorRetryCount: 3, 
  // 재시도 간격을 5초(5000ms)로 설정
  errorRetryInterval: 5000,
  // 새 데이터가 로드될 때까지 이전 캐시 데이터를 화면에 유지 
  keepPreviousData: true 
}
  );
  
  //로그를 통해 확인
  console.log('화면 리렌더링 됨')
  
  if (error) return <div>데이터를 불러오는데 실패했습니다.</div>;
  if (isLoading) return <div>데이터를 가져오는 중...</div>;
  
  return (
    <div style={{ border: "2px solid #4CAF50", padding: "15px", margin: "10px", borderRadius: "8px" }}>
      <h3>유저 정보 (SWR 캐싱 완료)</h3>
      <p><strong>이름:</strong> {data.name}</p>
      <p><strong>이메일:</strong> {data.email}</p>
      <p><strong>회사:</strong> {data.company.name}</p>
      
      {/* mutate()를 호출하면 해당 캐시 키의 데이터를 백그라운드에서 다시 가져옵니다 */}
      {/* 기존 데이터와 새로 가져온 데이터를 비교해서 변한게 없으면, 리렌더링 안함 */}
      <button onClick={() => mutate()} style={{ cursor: "pointer" }}>
        수동으로 최신 데이터 갱신하기
      </button>
    </div>
  );
  
}
```

`useSWR`의 인자로는 요청을 보낼 url 경로와, 위에서 만들었던 fetcher 를 넣어주면 됩니다.

여기서 `mutate`는 SWR이 관리하는 데이터를 강제로 업데이트할 때 사용할 함수입니다. 즉, 제가 원하는 타이밍에 업데이트를 하고 싶다면, `mutate`함수를 사용하면 됩니다.

`useSWR`에서 데이터를 새로 가져왔는데, 화면이 안바뀌면 안되겠죠? 그래서 `useSWR`의 데이터는 정보가 바뀔 때, 화면도 리렌더링 됩니다.

그리고, error, isLoading 도 사용 할 수 있습니다. 예전에는 커스텀 훅에서 이러한 요소들을 `useState`를 활용해서 만들어줬어야 했는데, `useSWR`은 그럴 필요 없이 간단하게 로딩이나, 오류처리를 할 수 있도록 도와줍니다.

옵션들도 있습니다.

```jsx
    {
  // 화면을 다시 볼 때(다른 탭을 보다가 돌아오거나 창을 클릭할 때) 자동으로 데이터를 새로고침 하지 않음
  revalidateOnFocus: false, 
  
  // 50초(50000ms) 안에는 같은 API 요청이 여러 번 발생해도 중복해서 서버로 보내지 않고 캐시를 씀
  dedupingInterval: 50000, 
  
  // 3초(3000ms)마다 백그라운드에서 새 데이터를 가져옴  
  refreshInterval: 3000, 
  
  // 브라우저 탭이 숨겨져 있을 땐(다른 탭 볼 때) 갱신 일시정지 (선택)
  refreshWhenHidden: false, 
  
  // 에러 발생 시 재시도 할지 말지 (기본값: true) 
  shouldRetryOnError: true, 
  
  // 최대 3번까지만 재시도 
  errorRetryCount: 3, 
  
  // 재시도 간격을 5초(5000ms)로 설정
  errorRetryInterval: 5000,
  
  // 새 데이터가 로드될 때까지 이전 캐시 데이터를 화면에 유지 
  keepPreviousData: true 
}
```

위처럼 작성한 후, api 에서 받아온 데이터는 변함이 없기 때문에, 제가 수동으로 새로고침 버튼을 눌러도 리렌더링이 일어나지 않으면 됩니다.

결과를 확인해보면, 제가 버튼을 3번 누르면, 누를 때 마다, api 요청은 발송이 됩니다.

하지만, console 창을 보시면, 리렌더링은 처음에 마운트 될때 한번, api 요청을 최초로 받아올 때 한번 일어나고, 그 다음부터는 리렌더링이 일어나지 않는것을 확인할 수 있습니다.

![](/assets/images/posts/2026-04-17-React%20useSWR/file-20260417150828770.png)

![](/assets/images/posts/2026-04-17-React%20useSWR/file-20260417150924934.png)