---
layout: post

title: "[Claude Code] 5편 - Hook과 Subagent"

# excerpt_image: /assets/images/posts/2026-11-22-Claude Code 5편/

categories:
  - AX

tags:
  - Claude Code
  - AI
  - Hook
  - Subagent

toc: true
toc_sticky: true

published: true

date: 2026-11-22
last_modified_at: 2026-11-22
---

## 들어가며

지금까지 Claude에게 규칙을 알려주는 방법으로 CLAUDE.md와 Skill을 다뤘습니다. 둘 다 결국은 Claude에게 **부탁**하는 방식입니다. Claude가 대부분 잘 따르지만, 100% 지킨다는 보장은 없습니다.

"ProjectSettings 폴더는 절대 건드리지 마" 같은 규칙은 부탁이 아니라 **강제**가 필요합니다. 이럴 때 쓰는 것이 **Hook**입니다.

또 하나, 큰 작업을 하다 보면 컨텍스트가 금방 찹니다. 코드 리뷰처럼 따로 떼어낼 수 있는 일은 다른 Claude에게 맡기고 결과만 받아오는 게 효율적입니다. 이 역할을 하는 것이 **Subagent**입니다.

이번 글에서는 다음 내용을 다룹니다.

* Hook이란 무엇이고 Skill과 어떻게 다른지
* 실습 1: 작업이 끝나면 소리로 알려주는 Hook
* 실습 2: 중요한 폴더 수정을 막는 Hook
* Subagent란 무엇인지
* 실습 3: 코드 리뷰 전용 Subagent
* Skill, Hook, Subagent 비교 정리

## 1. Hook이란?

Hook은 **정해진 시점에 무조건 실행되는 명령**입니다. Claude가 판단하는 것이 아니라, Claude Code 프로그램이 직접 실행합니다.

| 시점 (이벤트) | 언제 실행되나 | 활용 예 |
|---|---|---|
| `SessionStart` | 세션이 시작될 때 | 안내 메시지 띄우기, 환경 정보 주입 |
| `UserPromptSubmit` | 사용자가 메시지를 보낼 때 | 프롬프트 검사 |
| `PreToolUse` | Claude가 도구(파일 수정, 명령 실행 등)를 쓰기 **전** | 위험한 작업 차단 |
| `PostToolUse` | 도구를 쓴 **후** | 파일 수정 후 자동 포맷 |
| `Stop` | Claude가 응답을 마쳤을 때 | 작업 완료 알림 |

사실 우리는 이미 Hook을 써봤습니다. [3편](/ax/2026/11/22/Claude-Code-3%ED%8E%B8-Skill%EA%B3%BC-Plugin-%ED%99%9C%EC%9A%A9%ED%95%98%EA%B8%B0.html)에서 PonyTail을 설치하고 새 세션을 열었을 때 `PONYTAIL MODE ACTIVE`라는 안내가 떴습니다. 이게 PonyTail에 들어 있는 `SessionStart` Hook입니다.

### 설정 위치

Hook은 settings 파일에 적습니다.

| 파일 | 적용 범위 |
|---|---|
| `~/.claude/settings.json` | 내 모든 프로젝트 |
| `프로젝트/.claude/settings.json` | 이 프로젝트, 팀 공유 (git에 올림) |
| `프로젝트/.claude/settings.local.json` | 이 프로젝트, 나만 |

이번 실습에서는 `프로젝트/.claude/settings.json`을 사용합니다.

## 2. 실습 1 - 작업 끝나면 소리로 알려주기

Claude에게 긴 작업을 맡기고 다른 일을 하다 보면, 언제 끝났는지 확인하러 계속 창을 왔다 갔다 하게 됩니다. 응답이 끝나면 소리가 나도록 만들어 보겠습니다.

프로젝트 루트에 `.claude/settings.json` 파일을 만들고 아래 내용을 넣습니다. (이미 파일이 있다면 `hooks` 부분만 추가합니다.)

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          { "type": "command", "command": "powershell -c \"[console]::beep(800,300)\"" }
        ]
      }
    ]
  }
}
```

* `"Stop"`: Claude가 응답을 마쳤을 때 실행합니다.
* `"command"`: 실행할 명령입니다. 여기서는 PowerShell로 삑 소리를 냅니다. (`800`은 음 높이, `300`은 길이(ms)입니다)

> Mac이라면 `"command": "afplay /System/Library/Sounds/Glass.aiff"`처럼 바꾸면 됩니다.

저장한 뒤 **새 세션**을 열고 아무 요청이나 보내봅니다.

```
안녕
```

응답이 끝나는 순간 소리가 나면 성공입니다.

<!-- 📷 스크린샷: settings.json에 Stop Hook을 추가한 모습 -->

여기서 중요한 점은, Claude에게 "끝나면 알려줘"라고 **부탁하지 않았다**는 것입니다. Claude가 알든 모르든, 응답이 끝나면 **무조건** 소리가 납니다. 이것이 Hook과 Skill의 가장 큰 차이입니다.

## 3. 실습 2 - 중요한 폴더 수정 막기

[2편](/ax/2026/11/22/Claude-Code-2%ED%8E%B8-CLAUDE.md%EC%99%80-%EC%BB%A8%ED%85%8D%EC%8A%A4%ED%8A%B8-%EA%B4%80%EB%A6%AC.html)에서 CLAUDE.md에 "ProjectSettings 폴더는 수정하지 말 것"이라고 적었습니다. 하지만 이건 부탁일 뿐입니다. 이번에는 Hook으로 아예 **막아보겠습니다.**

### 동작 원리

1. Claude가 파일을 수정(`Edit`)하거나 만들려고(`Write`) 하면, 실행 **직전**에 `PreToolUse` Hook이 실행됩니다.
2. Hook 스크립트는 "어떤 파일을 수정하려는지" 정보를 JSON으로 받습니다.
3. 수정하려는 파일이 `Library`나 `ProjectSettings` 폴더 안에 있으면 **종료 코드 2**를 반환합니다.
4. 종료 코드가 2이면 Claude Code가 그 작업을 **취소**하고, 스크립트가 출력한 에러 메시지를 Claude에게 전달합니다.

### 스크립트 만들기

`.claude/hooks/protect.sh` 파일을 만듭니다.

```bash
#!/bin/bash
input=$(cat)
if echo "$input" | grep -qE '"file_path": *"[^"]*[\\/](Library|ProjectSettings)[\\/]'; then
  echo "Library, ProjectSettings 폴더는 수정할 수 없습니다." >&2
  exit 2
fi
exit 0
```

한 줄씩 보면 다음과 같습니다.

* `input=$(cat)`: Claude Code가 넘겨주는 JSON을 읽습니다. 안에 `"file_path": "C:\\...\\ProjectSettings\\..."` 같은 정보가 들어 있습니다.
* `grep -qE '...'`: 파일 경로에 `\Library\` 또는 `\ProjectSettings\` 폴더가 들어 있는지 검사합니다. 앞뒤에 폴더 구분자(`\` 또는 `/`)가 있어야 하므로, `LibraryUI.cs` 같은 파일 이름은 막지 않습니다.
* `echo "..." >&2`: 에러 메시지를 출력합니다. 이 메시지가 Claude에게 전달됩니다.
* `exit 2`: 작업을 차단합니다. `exit 0`은 통과입니다.

### settings.json에 등록

실습 1의 `settings.json`에 `PreToolUse` 부분을 추가합니다.

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          { "type": "command", "command": "powershell -c \"[console]::beep(800,300)\"" }
        ]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          { "type": "command", "command": "bash .claude/hooks/protect.sh" }
        ]
      }
    ]
  }
}
```

* `"matcher": "Edit|Write"`: 파일 수정(`Edit`)과 파일 생성(`Write`) 도구를 쓸 때만 실행합니다.

> Windows에서는 `bash` 명령을 쓰려면 **Git for Windows**가 설치되어 있어야 합니다. Claude Code를 Windows에서 쓰고 있다면 대부분 이미 설치되어 있습니다.

### 스크립트만 먼저 테스트하기

Claude에게 시키기 전에, 스크립트가 제대로 동작하는지 터미널(Git Bash)에서 먼저 확인할 수 있습니다.

```bash
echo '{"tool_input":{"file_path":"C:\\P\\ProjectSettings\\a.asset"}}' | bash .claude/hooks/protect.sh; echo "exit=$?"
```

```bash
echo '{"tool_input":{"file_path":"C:\\P\\Assets\\A.cs"}}' | bash .claude/hooks/protect.sh; echo "exit=$?"
```

첫 번째는 에러 메시지와 `exit=2`, 두 번째는 `exit=0`이 나오면 정상입니다.

### Claude로 테스트하기

새 세션을 열고 일부러 금지된 파일을 수정해 달라고 요청합니다.

```
ProjectSettings/ProjectSettings.asset 파일 맨 아래에 주석 한 줄 추가해줘
```

<!-- 📷 스크린샷: Hook에 의해 수정이 차단된 모습 -->

수정이 차단되고, Claude가 "Library, ProjectSettings 폴더는 수정할 수 없습니다"라는 메시지를 받아서 다른 방법을 안내하는 것을 확인할 수 있습니다.

일반 스크립트 수정은 평소처럼 잘 되는지도 확인합니다.

```
Assets/Scripts/PlayerController.cs 맨 위에 주석 한 줄 추가해줘
```

### 주의할 점

* Hook은 내 컴퓨터에서 **명령을 그대로 실행**합니다. 남이 만든 Hook을 받아 쓸 때는 반드시 내용을 읽어보고 사용합니다.
* settings 파일의 Hook을 바꾼 뒤에는 **새 세션**을 열어야 확실하게 적용됩니다.
* 이 스크립트는 파일 수정 도구만 막습니다. Claude가 터미널 명령으로 파일을 바꾸는 경우까지 막으려면 `Bash` 도구에 대한 Hook도 추가로 만들어야 합니다.

## 4. Subagent란?

Subagent는 **특정 역할만 맡는 별도의 Claude**입니다. 메인 Claude가 일을 맡기면, Subagent가 자기만의 컨텍스트에서 작업한 뒤 **결과 요약만** 돌려줍니다.

```
나 ──요청──> 메인 Claude ──"리뷰해줘"──> 리뷰 Subagent
                  ↑                         (파일 20개 읽음)
                  └────── 결과 요약만 ────────┘
```

Subagent를 쓰면 좋은 점은 다음과 같습니다.

* **컨텍스트 절약**: Subagent가 파일 20개를 읽어도, 메인 대화에는 요약만 들어옵니다.
* **역할 제한**: 리뷰어에게는 읽기 권한만 주는 식으로, 쓸 수 있는 도구를 제한할 수 있습니다.
* **모델 선택**: 간단한 일은 빠르고 저렴한 모델에게 맡길 수 있습니다.
* **전문화**: 역할에 맞는 지침을 따로 줄 수 있습니다.

## 5. 실습 3 - 코드 리뷰 전용 Subagent

### 파일 만들기

`.claude/agents/unity-reviewer.md` 파일을 만듭니다.

```markdown
---
name: unity-reviewer
description: Unity C# 코드 리뷰 전문가. 코드를 작성하거나 수정한 뒤 리뷰가 필요할 때 사용.
tools: Read, Grep, Glob
model: sonnet
---

너는 Unity 시니어 개발자다. 코드를 읽기만 하고 수정하지 않는다.

다음을 중점적으로 본다:
1. Update에서 매 프레임 호출되는 비싼 연산 (GetComponent, Find, 메모리 할당)
2. 물리 처리가 FixedUpdate에 있는지
3. null 체크 누락으로 생길 수 있는 NullReferenceException
4. CLAUDE.md의 컨벤션 위반

결과는 "심각 / 개선 / 사소" 세 단계로 나눠서 보고한다.
각 항목에는 파일명과 줄 번호, 왜 문제인지, 어떻게 고치면 되는지를 적는다.
```

Skill과 모양이 비슷하지만 frontmatter에 두 가지가 더 있습니다.

| 항목 | 의미 |
|---|---|
| `tools` | 이 Subagent가 쓸 수 있는 도구. `Read`(파일 읽기), `Grep`(내용 검색), `Glob`(파일 찾기)만 줬으므로 **파일을 수정할 수 없습니다.** |
| `model` | 사용할 모델. `sonnet`, `opus`, `haiku` 중에 고르거나, `inherit`로 메인과 같은 모델을 씁니다. |

> 직접 파일을 만드는 대신 `/agents` 명령으로 대화하면서 만들 수도 있습니다.

<!-- 📷 스크린샷: /agents 화면 -->

### 테스트

새 세션을 열고 요청합니다.

```
unity-reviewer로 Assets/Scripts 전체 리뷰해줘
```

[3편](/ax/2026/11/22/Claude-Code-3%ED%8E%B8-Skill%EA%B3%BC-Plugin-%ED%99%9C%EC%9A%A9%ED%95%98%EA%B8%B0.html)에서 만든 체력 시스템이나 [4편](/ax/2026/11/22/Claude-Code-4%ED%8E%B8-Skill-%EC%A7%81%EC%A0%91-%EB%A7%8C%EB%93%A4%EA%B8%B0.html)에서 만든 적 스크립트가 있다면 좋은 리뷰 대상이 됩니다. 1편의 버그 있는 `PlayerController.cs` 원본을 다시 넣어두고 리뷰를 맡겨 보는 것도 좋습니다.

<!-- 📷 스크린샷: unity-reviewer의 리뷰 결과 -->

확인할 것:

* 결과가 "심각 / 개선 / 사소"로 나뉘어 나오는지
* 리뷰어가 코드를 **직접 고치지 않았는지** (도구를 제한했기 때문에 고칠 수 없습니다)
* 메인 대화에는 리뷰 결과만 들어오고, 리뷰어가 읽은 파일 내용은 들어오지 않았는지 (`/context`로 확인해 볼 수 있습니다)

리뷰 결과를 보고 메인 Claude에게 고치라고 하면 됩니다.

```
리뷰에서 "심각"으로 나온 것만 고쳐줘
```

리뷰하는 Claude와 고치는 Claude를 나누면, 사람 팀에서 코드 리뷰를 하는 것과 같은 효과가 납니다.

## 6. Skill, Hook, Subagent 비교

지금까지 배운 Claude 확장 방법을 한 번에 정리하면 다음과 같습니다.

| | Skill | Hook | Subagent |
|---|---|---|---|
| 누가 실행하나 | Claude가 **판단**해서 | Claude Code가 **무조건** | Claude가 **위임**해서 |
| 용도 | 작업 방법 가르치기 | 규칙 강제하기, 자동화 | 일 떼어 맡기기 |
| 파일 위치 | `.claude/skills/` | `.claude/settings.json` | `.claude/agents/` |
| 이번 시리즈 예시 | 커밋 메시지, 적 생성 | 완료 알림, 폴더 보호 | 코드 리뷰어 |

언제 무엇을 쓸지 고민될 때는 이렇게 판단하면 됩니다.

* **"이렇게 하면 좋겠다"** → Skill
* **"반드시 이렇게 해야 한다", "매번 자동으로"** → Hook
* **"이 일은 따로 해서 결과만 줘"** → Subagent

그리고 이것들을 묶어서 남에게 배포하는 단위가 **Plugin**입니다. PonyTail이 Skill과 Hook을 함께 담고 있던 것처럼요.

## 마치며

이번 글에서는 Claude를 확장하는 나머지 두 방법을 다뤘습니다.

* **Hook**은 정해진 시점에 무조건 실행됩니다. 꼭 지켜야 하는 규칙은 Hook으로 만듭니다.
* **Subagent**는 일을 따로 떼어 맡기고 결과만 받아옵니다. 컨텍스트를 아끼고 역할을 나눌 수 있습니다.

이제 필요한 도구는 모두 갖췄습니다. 다음 글에서는 지금까지 만든 CLAUDE.md, Skill, Hook, Subagent에 Unity MCP까지 모두 써서 **게임 기능 하나를 처음부터 끝까지** 만들어 보겠습니다.
