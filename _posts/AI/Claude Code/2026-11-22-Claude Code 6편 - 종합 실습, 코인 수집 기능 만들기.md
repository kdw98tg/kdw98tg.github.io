---
layout: post

title: "[Claude Code] 6편 - 종합 실습, Unity MCP로 코인 수집 기능 만들기"

# excerpt_image: /assets/images/posts/2026-11-22-Claude Code 6편/

categories:
  - AI

tags:
  - Claude Code
  - AI
  - Unity
  - MCP

toc: true
toc_sticky: true

published: true

date: 2026-11-22
last_modified_at: 2026-11-22
---

## 들어가며

지금까지 Claude Code를 잘 쓰기 위한 도구들을 하나씩 다뤘습니다.

| 편 | 내용 | 만든 것 |
|---|---|---|
| [1편](/ai/2026/11/22/Claude-Code-1%ED%8E%B8-%EC%9D%BC-%EC%9E%98-%EC%8B%9C%ED%82%A4%EB%8A%94-%EB%B2%95.html) | 일 잘 시키는 법 | Plan mode, 구체적인 프롬프트 |
| [2편](/ai/2026/11/22/Claude-Code-2%ED%8E%B8-CLAUDE.md%EC%99%80-%EC%BB%A8%ED%85%8D%EC%8A%A4%ED%8A%B8-%EA%B4%80%EB%A6%AC.html) | CLAUDE.md, 컨텍스트 | 프로젝트 설명서 |
| [3편](/ai/2026/11/22/Claude-Code-3%ED%8E%B8-Skill%EA%B3%BC-Plugin-%ED%99%9C%EC%9A%A9%ED%95%98%EA%B8%B0.html) | Skill 활용 | PonyTail 설치 |
| [4편](/ai/2026/11/22/Claude-Code-4%ED%8E%B8-Skill-%EC%A7%81%EC%A0%91-%EB%A7%8C%EB%93%A4%EA%B8%B0.html) | Skill 만들기 | commit-message, new-enemy Skill |
| [5편](/ai/2026/11/22/Claude-Code-5%ED%8E%B8-Hook%EA%B3%BC-Subagent.html) | Hook, Subagent | 폴더 보호 Hook, unity-reviewer |

이번 글에서는 이 도구들을 **모두** 써서 게임 기능 하나를 처음부터 끝까지 만들어 보겠습니다. 그리고 Unity MCP를 이용해서, Claude가 스크립트만 쓰는 것이 아니라 **유니티 에디터를 직접 조작해서** 씬까지 구성하게 해보겠습니다.

만들 기능은 다음과 같습니다.

* 맵에 코인 10개가 흩어져 있습니다.
* 플레이어가 코인에 닿으면 코인이 사라지고 점수가 1 올라갑니다.
* 화면 왼쪽 위에 `Coin: 3/10`처럼 점수가 표시됩니다.
* 10개를 다 모으면 화면 가운데에 `Clear!`가 표시됩니다.

## 1. 준비

### 프로젝트 상태 확인

이전 글들을 따라왔다면 다음이 준비되어 있어야 합니다.

* [ ] `Ground`(Plane)와 `Player`(Capsule + Rigidbody + PlayerController)가 있는 씬 ([1편](/ai/2026/11/22/Claude-Code-1%ED%8E%B8-%EC%9D%BC-%EC%9E%98-%EC%8B%9C%ED%82%A4%EB%8A%94-%EB%B2%95.html))
* [ ] 프로젝트 루트의 `CLAUDE.md` ([2편](/ai/2026/11/22/Claude-Code-2%ED%8E%B8-CLAUDE.md%EC%99%80-%EC%BB%A8%ED%85%8D%EC%8A%A4%ED%8A%B8-%EA%B4%80%EB%A6%AC.html))
* [ ] `.claude/skills/commit-message/`, `.claude/skills/unity-script-convention/` ([4편](/ai/2026/11/22/Claude-Code-4%ED%8E%B8-Skill-%EC%A7%81%EC%A0%91-%EB%A7%8C%EB%93%A4%EA%B8%B0.html))
* [ ] `.claude/settings.json`의 Hook ([5편](/ai/2026/11/22/Claude-Code-5%ED%8E%B8-Hook%EA%B3%BC-Subagent.html))
* [ ] `.claude/agents/unity-reviewer.md` ([5편](/ai/2026/11/22/Claude-Code-5%ED%8E%B8-Hook%EA%B3%BC-Subagent.html))

`Player` 오브젝트의 Tag를 `Player`로 바꿔줍니다. 코인이 플레이어를 알아보는 데 사용합니다.

<!-- 📷 스크린샷: Player 오브젝트의 Tag 설정 -->

### Unity MCP 확인

Unity MCP 연결은 [Unity MCP 서버 연결하기](/game%20engine/2026/10/20/Unity-Mcp-%EC%84%9C%EB%B2%84-%EC%97%B0%EA%B2%B0.html) 글을 참고해 주세요. 연결이 되어 있다면 `Edit > Project Settings > AI > Unity MCP Server`에서 다음을 확인합니다.

* `Unity Bridge - Status`가 `Running`인지
* `Connected Clients`에 Claude Code가 `Accepted` 상태로 있는지
* `Tools`에서 이번 실습에 필요한 툴이 켜져 있는지

기본값으로는 일부 툴만 켜져 있습니다. 이번 실습에서는 **GameObject 생성과 수정, 컴포넌트 추가, 씬 조회** 관련 툴이 필요하니 목록에서 찾아서 켜줍니다.

<!-- 📷 스크린샷: Unity MCP Server의 Tools 목록에서 필요한 툴을 켠 모습 -->

Claude Code에서 연결이 잘 되었는지 확인해 봅니다.

```
현재 씬에 있는 오브젝트 목록 알려줘
```

`Ground`, `Player`, `Main Camera` 등이 나오면 준비 완료입니다.

### git 커밋해두기

라이브로 진행하다 보면 예상과 다르게 흘러갈 수 있습니다. 시작하기 전에 지금 상태를 커밋해 둡니다.

```
지금 상태 커밋해줘
```

[4편](/ai/2026/11/22/Claude-Code-4%ED%8E%B8-Skill-%EC%A7%81%EC%A0%91-%EB%A7%8C%EB%93%A4%EA%B8%B0.html)에서 만든 `commit-message` Skill이 불려서 `[기능] ...` 같은 형식으로 커밋되는지도 함께 확인해 봅니다.

## 2. 1단계 - 계획 세우기 (Plan mode)

[1편](/ai/2026/11/22/Claude-Code-1%ED%8E%B8-%EC%9D%BC-%EC%9E%98-%EC%8B%9C%ED%82%A4%EB%8A%94-%EB%B2%95.html)에서 배운 대로, 큰 작업은 계획부터 받습니다. Plan mode를 켜고 요청합니다.

```
코인 수집 기능을 만들고 싶어.
- 맵에 코인 10개를 랜덤 위치에 배치 (Ground 범위 안)
- 플레이어가 닿으면 코인이 사라지고 점수 +1
- 화면 왼쪽 위에 "Coin: 3/10" 표시
- 10개를 다 모으면 화면 가운데에 "Clear!" 표시

필요한 스크립트와 씬 구성을 계획해줘.
스크립트는 직접 작성하고, 씬 구성은 Unity MCP로 할 거야.
```

<!-- 📷 스크린샷: Claude가 제시한 계획 -->

계획에서 다음을 확인합니다.

* 어떤 스크립트를 만드는지 (예: `Coin.cs`, `CoinManager.cs`, `CoinUI.cs`)
* 각 스크립트가 어느 폴더에 들어가는지 (CLAUDE.md의 폴더 구조를 따르는지)
* 코인과 플레이어가 닿은 것을 어떻게 감지하는지 (보통 Trigger Collider와 `OnTriggerEnter`)
* 점수를 어디서 관리하는지

계획이 너무 복잡하다면 줄여달라고 요청합니다. 예를 들어 이벤트 시스템이나 싱글톤 매니저까지 만들겠다고 하면 이렇게 말해볼 수 있습니다.

```
더 단순하게 해줘. 스크립트 2개 이내로.
```

> PonyTail이 켜져 있다면 처음부터 단순한 계획이 나올 가능성이 높습니다.

계획이 마음에 들면 승인합니다.

## 3. 2단계 - 스크립트 작성

계획을 승인하면 Claude가 스크립트를 작성합니다.

<!-- 📷 스크린샷: 작성된 Coin.cs, CoinManager.cs -->

작성된 코드에서 이전 편에서 만든 규칙들이 지켜지는지 확인해 봅니다.

| 확인할 것 | 어디서 정한 규칙? |
|---|---|
| `Assets/Scripts/` 아래 적절한 폴더에 있는지 | CLAUDE.md (2편) |
| `[SerializeField] private`를 썼는지 | unity-script-convention Skill (4편) |
| `GetComponent`를 `Awake`에서 캐싱했는지 | unity-script-convention Skill (4편) |
| `Update`에서 `Find`를 호출하지 않는지 | unity-script-convention Skill (4편) |

스크립트 작성이 끝나면 **유니티 에디터로 돌아가서 컴파일이 끝날 때까지 기다립니다.** 스크립트가 컴파일되어야 다음 단계에서 컴포넌트로 붙일 수 있습니다. 콘솔에 에러가 있다면 그대로 복사해서 Claude에게 붙여넣습니다.

```
콘솔에 이 에러가 떠. 고쳐줘.
(에러 메시지 붙여넣기)
```

## 4. 3단계 - Unity MCP로 씬 구성하기

이번 실습의 하이라이트입니다. 씬 구성을 Claude에게 맡겨 보겠습니다.

### 코인 만들기

```
Unity MCP로 코인을 만들어줘.
- Cylinder로 만들고, 납작하게 (Scale 0.5, 0.05, 0.5), X축으로 90도 회전
- 노란색 머티리얼 적용
- Collider는 Is Trigger 켜기
- 아까 만든 Coin 스크립트 붙이기
- 프리팹으로 저장 (Assets/Prefabs/Coin.prefab)
```

<!-- 📷 스크린샷: Claude가 Unity MCP 툴을 호출하는 모습 -->

Claude가 Unity MCP 툴을 하나씩 호출하는 과정이 보입니다. 유니티 에디터의 Hierarchy에서 실시간으로 오브젝트가 생기는 것을 확인할 수 있습니다.

<!-- 📷 스크린샷: 유니티 에디터에 생성된 코인 -->

### 코인 배치하기

```
Coin 프리팹을 Ground 위에 10개 배치해줘.
위치는 랜덤으로, 서로 너무 붙지 않게. 높이는 Y=1.
```

### UI 만들기

```
Canvas를 만들고 텍스트 두 개를 추가해줘.
- 왼쪽 위: 점수 텍스트 "Coin: 0/10"
- 가운데: "Clear!" 텍스트, 크게, 처음엔 비활성화
그리고 CoinManager에 두 텍스트를 연결해줘.
```

> TextMeshPro를 처음 쓰는 프로젝트라면 `Import TMP Essentials` 창이 뜰 수 있습니다. 이때는 직접 `Import` 버튼을 눌러줍니다.

<!-- 📷 스크린샷: 완성된 씬 (코인 10개 + UI) -->

### 잘 안 될 때

Unity MCP로 할 수 있는 작업의 범위는 **켜둔 툴**에 따라 달라집니다. Claude가 "이 작업을 할 수 있는 툴이 없다"고 하면 다음을 확인합니다.

* Unity MCP Server 설정에서 해당 기능의 툴이 켜져 있는지
* 툴을 켰다면, Claude Code 세션을 새로 열어 툴 목록을 다시 불러오게 합니다.

그래도 안 되는 작업은 Claude에게 **수동으로 하는 방법을 알려달라고** 하고 직접 하면 됩니다. 모든 것을 자동화하는 게 목표가 아니라, 귀찮은 부분을 맡기는 게 목표입니다.

## 5. 4단계 - 테스트

Play 버튼을 누르고 직접 테스트합니다.

* [ ] WASD로 이동해서 코인에 닿으면 코인이 사라지는지
* [ ] 왼쪽 위 점수가 `Coin: 1/10`, `Coin: 2/10`으로 올라가는지
* [ ] 10개를 다 먹으면 `Clear!`가 뜨는지
* [ ] 콘솔에 에러가 없는지

<!-- 📷 스크린샷: Play 모드에서 Clear! 화면 -->

문제가 있다면 [1편](/ai/2026/11/22/Claude-Code-1%ED%8E%B8-%EC%9D%BC-%EC%9E%98-%EC%8B%9C%ED%82%A4%EB%8A%94-%EB%B2%95.html)에서 배운 대로 **증상을 구체적으로** 말합니다.

| 나쁜 예 | 좋은 예 |
|---|---|
| 코인이 안 돼 | 플레이어가 코인을 통과하는데 코인이 안 사라지고 점수도 안 올라가. 콘솔 에러는 없어 |

Claude가 씬 상태를 직접 확인할 수 있으니, 이렇게 요청하는 것도 좋습니다.

```
Unity MCP로 Player랑 Coin 오브젝트의 컴포넌트 설정을 확인해서 원인을 찾아줘
```

> 코인이 안 먹힐 때 가장 흔한 원인은 **Player의 Tag가 `Player`가 아니거나**, 코인 Collider의 **Is Trigger가 꺼져 있는** 경우입니다.

## 6. 5단계 - 리뷰와 커밋

### Subagent로 리뷰

기능이 동작하면 [5편](/ai/2026/11/22/Claude-Code-5%ED%8E%B8-Hook%EA%B3%BC-Subagent.html)에서 만든 리뷰어에게 맡깁니다.

```
unity-reviewer로 이번에 만든 코인 관련 스크립트 리뷰해줘
```

<!-- 📷 스크린샷: unity-reviewer의 리뷰 결과 -->

리뷰 결과에서 "심각"과 "개선" 항목을 보고, 고칠 것만 골라서 요청합니다.

```
리뷰에서 심각으로 나온 것만 고쳐줘
```

고친 뒤에는 다시 Play 모드로 동작을 확인합니다.

### 커밋

```
코인 수집 기능 커밋해줘
```

`commit-message` Skill이 불려서 다음과 같은 형식으로 커밋되는지 확인합니다.

```
[기능] 코인 수집 기능 추가

맵에 코인 10개를 배치하고, 모두 모으면 클리어되는 기본 게임 루프.
```

### 돌아보기

이번 실습에서 쓴 도구를 정리하면 다음과 같습니다.

| 단계 | 쓴 도구 |
|---|---|
| 계획 | Plan mode (1편) |
| 스크립트 작성 | CLAUDE.md (2편), unity-script-convention Skill (4편), PonyTail (3편) |
| 씬 구성 | Unity MCP |
| 보호 | ProjectSettings 보호 Hook (5편) |
| 리뷰 | unity-reviewer Subagent (5편) |
| 커밋 | commit-message Skill (4편), 완료 알림 Hook (5편) |

## 7. 더 알아보기

시리즈에서 자세히 다루지 않았지만, 익숙해지면 써볼 만한 기능들입니다.

### 병렬 작업 (git worktree)

git worktree를 쓰면 같은 저장소를 폴더 여러 개로 나눠서 동시에 작업할 수 있습니다. 세션 하나는 적 AI를, 다른 세션은 UI를 만드는 식으로 Claude 여러 개를 동시에 돌릴 수 있습니다.

```bash
git worktree add ../my-game-ui -b feature/ui
```

새로 만들어진 `../my-game-ui` 폴더에서 Claude Code를 따로 실행하면 됩니다. 단, Unity는 프로젝트 폴더마다 에디터를 따로 열어야 하고 `Library` 폴더도 따로 만들어지기 때문에, 스크립트 작업 위주일 때 유용합니다.

### Headless 모드 (`claude -p`)

대화창 없이 명령 한 줄로 Claude를 실행할 수 있습니다.

```bash
claude -p "Assets/Scripts 폴더의 스크립트 중 Update에서 GetComponent를 호출하는 곳을 찾아줘"
```

스크립트나 CI에 넣어서 자동화할 수 있습니다. GitHub Actions에서 PR이 올라오면 자동으로 리뷰하게 만드는 것이 대표적인 활용 예입니다.

### 모델 선택

작업에 따라 모델을 바꾸면 속도와 비용을 아낄 수 있습니다. `/model`로 바꿀 수 있습니다.

| 모델 | 적합한 작업 |
|---|---|
| Opus | 복잡한 설계, 어려운 버그 |
| Sonnet | 일반적인 기능 구현, 리뷰 |
| Haiku | 단순 반복 작업, 빠른 검색 |

[5편](/ai/2026/11/22/Claude-Code-5%ED%8E%B8-Hook%EA%B3%BC-Subagent.html)의 unity-reviewer처럼 Subagent마다 다른 모델을 지정하는 것도 좋은 방법입니다.

## 마치며

6편에 걸쳐 Claude Code를 실전에서 쓰는 방법을 정리했습니다.

처음에는 "점프 고쳐줘" 한 마디로 시작했지만, 이제는 계획을 세우고, 팀 규칙을 알려주고, 반복되는 일은 Skill로 만들고, 꼭 지켜야 할 규칙은 Hook으로 강제하고, 리뷰는 Subagent에게 맡기고, 씬 구성은 Unity MCP로 에디터를 직접 조작하게 할 수 있게 되었습니다.

결국 핵심은 하나입니다. **Claude를 잘 쓰는 방법은 좋은 동료에게 일을 맡기는 방법과 같습니다.** 상황을 구체적으로 설명하고, 팀의 규칙을 알려주고, 결과를 확인하는 것입니다. 여기에 Claude Code만의 도구들을 더하면, 같은 설명을 반복하지 않고도 매번 일관된 결과를 얻을 수 있습니다.

이번 시리즈에서 만든 CLAUDE.md, Skill, Hook, Subagent는 모두 `.claude/` 폴더에 들어 있습니다. 이 폴더를 git에 함께 올려두면, 팀원 누구나 같은 환경에서 Claude를 쓸 수 있습니다.
