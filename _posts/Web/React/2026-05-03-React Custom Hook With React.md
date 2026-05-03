---
layout: post

title: "[React/Js] custom hook 으로 useSWR 사용하기"

categories:
  - Web

excerpt_image: /assets/images/posts/2026-05-03-React%20Custom%20Hook%20With%20React/file-20260503224045029.png

tags:
  - React
  - hooks
  - network

toc: true
toc_sticky: true

published: true

date: 2026-05-03
last_modified_at: 2026-05-03
---


## Custom Hook 과 useSWR 를 사용하여 네트워크 통신하기

![](/assets/images/posts/2026-05-03-React%20Custom%20Hook%20With%20React/file-20260503224045029.png)

React 에서는 함수형 컴포넌트를 사용하면서, 상태를 저장하기 위해 hook 이라는 개념을 도입하였습니다. 이 hook 은 React에서 만들어놓은것을 사용할 수 있고, 사용자가 여러 hook 들을 조합하여 custom hook 을 만들어 사용할 수 있습니다.

이번 포스팅에서는 custom hook 에 useSWR 을 넣어서, 캐싱을 활용한 네트워크 통신을 구현하는 예제를 다룹니다.

우선, 통신 라이브러리는 `axios`를 사용합니다.

왜 Custom Hook과 useSWR 패턴을 사용할까요? 단순히 `useEffect`와 `useState`를 사용하는 대신 이 패턴을 도입하면 다음과 같은 강력한 이점이 있습니다.

1. 서버 상태(Server State) 관리의 분리: 클라이언트의 UI 상태와 서버에서 가져온 데이터를 분리하여 비즈니스 로직의 복잡도를 크게 낮춥니다.
2. 자동 캐싱 및 재검증: 포커스 시 재검증(Revalidate on focus), 중복 요청 제거(Deduplication) 등 강력한 기능을 통해 네트워크 비용을 줄이고 사용자 경험을 향상시킵니다. 
3. 관심사의 분리: UI 컴포넌트는 '데이터를 어떻게 가져올지' 고민할 필요 없이, 오직 '가져온 데이터를 어떻게 보여줄지'에만 집중할 수 있습니다.

구조는 다음과 같습니다.

1. component 에서  custom hook 호출
2. useSWR 의 fetcher로 axios를 사용
3. 통신완료 후, object destructuring을 통해 component 로 전달
4. component에서 받은 데이터 사용

## axios를 사용하여 crud 함수 만들기

우선, useSWR에서 사용할 fetcher 를 만드는 과정 입니다. axios를 사용하여 구현합니다. crud 함수를 모두 구현합니다.

서버의 route를 받는 함수를 만드는 파일과, crud 를 할 수 있는 함수들이 모여있는 파일 2개를 만듭니다.

getDefaultProjectRoute.js
```js
function getDefaultProjectRoute() {
  return "/projects/";
}

export default getDefaultProjectRoute;
```

projectService.js
```js
export async function getProjects() {
  const response = await mainApi.get("/projects/");
  return response.data;
}

export async function createProject(payload) {
  const response = await mainApi.post("/projects/", payload);
  return response.data;
}

export async function updateProject(id, payload) {
  const response = await mainApi.put("/projects/" + id, payload);
  return response.data;
}

export async function deleteProject(id) {
  await mainApi.delete("/projects/" + id);
  return id;
}
```

## CRUD 함수를 사용하여 custom hook 만들기

이제, 위에서 만들어둔 crud 함수들을 `useSWR` 과 합쳐서 통신을 하기 위해서 custom hook 을 만들어서 관리합니다.

custom hook 을 만들때, get과, post/delete/update 는 따로 만들었습니다.

get을 하는 `useProjects` 라는 hook 으로, 그리고 post/delete/update는 `useProjectMutation` 이라는 hook 으로 만들었습니다.

useProjects.js
```js
import useSWR from "swr";
import { getProjects } from "../services/projectService";
import getDefaultProjectRoute from "../services/getDefaultProjectRoute";

function useProjects() {
  const { data, error, isLoading, mutate } = useSWR(
    getDefaultProjectRoute(),
    getProjects,
  );

  return {
    projects: data ?? [],
    isLoading,
    isError: !!error,
    refreshProjects: mutate,
  };
}

export default useProjects;
```

useProjectMutation.js
```js
import { useSWRConfig } from "swr";
import {
  createProject,
  deleteProject,
  updateProject,
} from "../services/projectService";
import getDefaultProjectRoute from "../services/getDefaultProjectRoute";

function useProjectMutations() {
  const { mutate } = useSWRConfig();

  async function addProject(payload) {
    const created = await createProject(payload);
    await mutate(getDefaultProjectRoute());
    return created;
  }

  async function editProject(id, payload) {
    const updated = await updateProject(id, payload);
    await mutate(getDefaultProjectRoute());
    return updated;
  }

  async function removeProject(id) {
    await deleteProject(id);
    await mutate(getDefaultProjectRoute());
  }

  return { addProject, editProject, removeProject };
}

export default useProjectMutations;
```

## custom hook 을 componet 에서 사용하기

목록을 렌더링하고, 새로운 프로젝트를 추가하거나 기존 프로젝트를 삭제하는 UI 컴포넌트를 작성해 보겠습니다.

앞서 분리해 둔 custom hook을 사용하면 컴포넌트 내부에는 복잡한 비동기 통신 로직이나 캐시 업데이트 로직이 남지 않게 됩니다. 오직 **UI 상태 처리**와 **사용자 액션에 따른 함수 호출**에만 집중할 수 있습니다.

```js
import React from "react";
import useProjects from "../hooks/useProjects";
import useProjectMutations from "../hooks/useProjectMutations";

function ProjectList() {
  // 1. 데이터 Fetching Hook 호출
  const { projects, isLoading, isError } = useProjects();
  
  // 2. 데이터 Mutation Hook 호출
  const { addProject, removeProject } = useProjectMutations();

  // 3. 로딩 및 에러 상태 처리
  if (isLoading) return <div>프로젝트 데이터를 불러오는 중입니다...</div>;
  if (isError) return <div>데이터를 불러오는 데 실패했습니다.</div>;

  // 4. 이벤트 핸들러
  const handleAddProject = async () => {
    const newProject = {
      title: "새로운 프로젝트",
      description: "커스텀 훅 테스트 목적입니다.",
    };
    
    try {
      await addProject(newProject);
      // addProject 내부에서 mutate가 호출되므로, 
      // 별도의 데이터 refetch 없이도 화면이 자동으로 최신화됩니다.
    } catch (error) {
      console.error("프로젝트 추가 실패:", error);
    }
  };

  const handleDeleteProject = async (id) => {
    if (window.confirm("정말 삭제하시겠습니까?")) {
      try {
        await removeProject(id);
      } catch (error) {
        console.error("프로젝트 삭제 실패:", error);
      }
    }
  };

  return (
    <div>
      <h2>프로젝트 목록</h2>
      <button onClick={handleAddProject}>새 프로젝트 추가</button>
      
      <ul>
        {projects.length === 0 ? (
          <p>등록된 프로젝트가 없습니다.</p>
        ) : (
          projects.map((project) => (
            <li key={project.id} style={{ margin: "10px 0" }}>
              <strong>{project.title}</strong> - {project.description}
              <button 
                onClick={() => handleDeleteProject(project.id)}
                style={{ marginLeft: "10px" }}
              >
                삭제
              </button>
            </li>
          ))
        )}
      </ul>
    </div>
  );
}

export default ProjectList;
```

## 결론

`useSWR`을 Custom Hook으로 한 번 더 감싸는 패턴은 초기 세팅 시 파일이 많아져 번거로워 보일 수 있습니다. 하지만 프로젝트 규모가 커지고 여러 페이지에서 동일한 데이터를 요구할 때 이 아키텍처는 진가를 발휘합니다.

네트워크 통신과 데이터 캐싱이라는 복잡한 '수단'을 Custom Hook 뒤로 완벽히 숨김으로써, 우리의 React 컴포넌트는 본연의 '목적'인 UI 렌더링과 유저 인터랙션 처리에만 충실할 수 있게 됩니다.