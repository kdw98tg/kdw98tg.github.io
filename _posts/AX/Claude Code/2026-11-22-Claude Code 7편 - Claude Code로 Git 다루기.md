---
layout: post

title: "[Claude Code] 7편 - Claude Code로 Git 다루기 (커밋, 브랜치, PR, 충돌)"

# excerpt_image: /assets/images/posts/2026-11-22-Claude Code 7편/

categories:
  - AX

tags:
  - Claude Code
  - Git
  - GitHub
  - Unity

toc: true
toc_sticky: true

published: true

date: 2026-11-22
last_modified_at: 2026-11-22
---

## 들어가며

[Git 시작하기](/git/2026/11/22/Git-Unity-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-Git-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0-%28Sourcetree%29.html) 글에서는 Sourcetree로 커밋, 푸시, 풀, 브랜치, PR을 직접 해봤습니다. 이번 글에서는 같은 작업을 **Claude Code에게 맡기는 방법**을 다룹니다.

Claude Code는 터미널에서 `git` 명령을 직접 실행할 수 있습니다. 그래서 "커밋해줘", "브랜치 만들어줘", "PR 올려줘"라고 말로 시키면 됩니다. 특히 다음 작업에서 큰 도움이 됩니다.

* 변경 내용을 읽고 **커밋 메시지 작성하기**
* 섞여 있는 변경을 **커밋 여러 개로 나누기**
* 변경 내용을 정리해서 **PR 설명 쓰기**
* 스크립트 **충돌 해결하기**
* "이 코드 언제 왜 바뀌었지?" **이력 찾기**

그리고 Claude에게 Git을 맡길 때 **사고를 막는 방법**도 함께 정리합니다.

> Sourcetree는 계속 같이 쓰는 것을 추천합니다. Claude가 한 작업을 Sourcetree의 히스토리 화면에서 눈으로 확인하면, 무슨 일이 일어났는지 이해하기 훨씬 쉽습니다.

## 1. 준비

### Git 저장소 확인

프로젝트가 이미 Git 저장소인지 Claude에게 물어봅니다.

```
이 프로젝트 git 상태 확인해줘
```

저장소가 아니라면 이렇게 요청합니다.

```
git 저장소로 만들고, Unity용 .gitignore를 만들어줘.
Claude Code 개인 설정 파일(.claude/settings.local.json, CLAUDE.local.md)도 무시하게 해줘.
아직 커밋은 하지 마.
```

<!-- 📷 스크린샷: Claude가 만든 .gitignore -->

`.gitignore`가 만들어지면 Library 폴더가 정말 빠졌는지 확인을 시킵니다.

```
지금 커밋하면 올라갈 파일 목록에 Library나 Temp 폴더 파일이 있는지 확인해줘
```

> **"아직 커밋은 하지 마"**처럼 어디까지 할지 정해주는 것이 좋습니다. 1편에서 배운 "범위 정해주기"와 같습니다.

### GitHub CLI (gh) 설치

Claude가 PR을 만들려면 **GitHub CLI**(`gh`)가 필요합니다. PowerShell에서 설치합니다.

```bash
winget install --id GitHub.cli
```

설치 후 터미널을 새로 열고 GitHub에 로그인합니다.

```bash
gh auth login
```

질문이 나오면 `GitHub.com` → `HTTPS` → `Login with a web browser`를 고르고, 화면에 나온 코드를 브라우저에 입력하면 됩니다.

<!-- 📷 스크린샷: gh auth login 진행 화면 -->

> 로그인은 **직접** 하세요. 비밀번호나 토큰을 Claude에게 알려줄 필요가 없고, 알려줘서도 안 됩니다.

## 2. 커밋 맡기기

### 기본

파일을 수정한 뒤 이렇게 요청합니다.

```
지금 변경사항 커밋해줘
```

Claude는 보통 다음 순서로 진행합니다.

1. `git status`로 바뀐 파일을 확인합니다.
2. `git diff`로 무엇이 바뀌었는지 읽습니다.
3. 변경 내용에 맞는 커밋 메시지를 작성합니다.
4. `git add`, `git commit`을 실행합니다.

<!-- 📷 스크린샷: Claude가 커밋하는 과정 -->

[4편](/ax/2026/11/22/Claude-Code-4%ED%8E%B8-Skill-%EC%A7%81%EC%A0%91-%EB%A7%8C%EB%93%A4%EA%B8%B0.html)에서 만든 `commit-message` Skill이 있다면 `[기능]`, `[수정]` 같은 팀 형식으로 메시지가 작성됩니다.

> Claude Code가 만든 커밋에는 메시지 끝에 `Co-Authored-By: Claude ...` 줄이 붙습니다. Claude가 함께 작업했다는 표시입니다. GitHub에서는 공동 작성자로 표시됩니다.

### 커밋 나누기

여러 작업을 한꺼번에 했다면 나눠달라고 할 수 있습니다.

```
변경사항을 보고 성격이 다른 것끼리 커밋을 나눠줘. 커밋하기 전에 어떻게 나눌지 먼저 보여줘.
```

예를 들어 "점프 버그 수정"과 "코인 기능 추가"가 섞여 있다면 커밋 두 개로 나눠서 각각 커밋합니다. 사람이 Sourcetree에서 파일을 하나씩 골라 스테이지에 올리는 작업을 대신해 주는 셈입니다.

### 커밋 전에 확인시키기

```
커밋하기 전에 디버그용 Debug.Log나 주석 처리된 코드가 남아 있는지 확인해줘
```

커밋 전 점검을 습관처럼 시키면 실수가 줄어듭니다. 매번 말하기 귀찮다면 `commit-message` Skill 본문에 이 점검 단계를 추가해 두면 됩니다.

## 3. 브랜치 맡기기

### 브랜치 만들고 작업하기

```
feature/coin 브랜치를 만들어서 거기서 코인 수집 기능을 작업해줘
```

Claude가 `git checkout -b feature/coin`(또는 `git switch -c`)으로 브랜치를 만들고 이동한 뒤 작업을 시작합니다.

<!-- 📷 스크린샷: Sourcetree에서 새 브랜치가 생긴 모습 -->

> 브랜치가 바뀌면 프로젝트 파일이 바뀌고, 유니티가 에셋을 다시 불러옵니다. Claude에게 브랜치 작업을 시키기 전에 **유니티에서 씬을 저장**해 두세요.

### 브랜치 정리

```
main에 이미 병합된 브랜치 목록 보여주고, 지워도 되는 것만 알려줘
```

삭제는 목록을 확인한 뒤 직접 지시합니다.

```
그중에 feature/coin 브랜치만 지워줘
```

## 4. 푸시와 PR 맡기기

### 푸시

```
feature/coin 브랜치 푸시해줘
```

Claude는 **푸시하라고 말하기 전에는 알아서 푸시하지 않습니다.** 푸시는 원격 저장소, 즉 팀원 모두에게 영향을 주는 작업이기 때문입니다. 권한 모드에 따라 명령 실행 전에 허락을 구하기도 합니다.

### PR 만들기

```
feature/coin 브랜치로 main에 PR 만들어줘
```

Claude는 다음을 합니다.

1. `main`과 비교해서 이 브랜치의 커밋과 변경 내용을 읽습니다.
2. PR 제목과 설명(무엇을, 왜, 어떻게 테스트했는지)을 작성합니다.
3. `gh pr create`로 PR을 만들고 링크를 알려줍니다.

<!-- 📷 스크린샷: Claude가 만든 PR (GitHub 화면) -->

PR 설명에 넣을 내용을 정해줄 수도 있습니다.

```
PR 설명에는 변경 요약, 테스트 방법, 유니티에서 따로 설정해야 할 것(Tag, Layer 등)을 넣어줘
```

유니티 프로젝트에서는 **"코드 밖에서 해야 하는 설정"**(Tag 추가, Inspector 값 연결 등)을 PR에 적어두는 것이 특히 중요합니다. 코드만 받아서는 알 수 없는 정보이기 때문입니다.

### PR 리뷰 맡기기

팀원이 올린 PR을 Claude에게 먼저 리뷰받을 수도 있습니다.

```
PR 12번 내용 보고 리뷰해줘. 버그 가능성 위주로.
```

Claude가 `gh pr view`, `gh pr diff`로 PR을 읽고 리뷰합니다. [5편](/ax/2026/11/22/Claude-Code-5%ED%8E%B8-Hook%EA%B3%BC-Subagent.html)에서 만든 `unity-reviewer` Subagent에게 맡겨도 됩니다.

```
unity-reviewer로 PR 12번 리뷰해줘
```

> 리뷰 결과를 PR에 **코멘트로 올리는 것**은 팀원에게 보이는 작업입니다. 먼저 결과를 확인하고, 올릴지는 직접 결정하세요.

## 5. 풀과 충돌 해결 맡기기

### 풀

```
main 최신으로 받아줘
```

커밋하지 않은 변경이 있으면 Claude가 먼저 알려주고 어떻게 할지 물어봅니다. (커밋하기, 스태시로 치워두기 등)

### 충돌 해결

병합하다가 충돌이 나면 이렇게 요청합니다.

```
충돌 난 파일 목록 보여주고, 스크립트 충돌은 양쪽 변경 의도를 살려서 해결해줘.
어떻게 해결했는지 파일별로 설명해줘.
```

<!-- 📷 스크린샷: Claude가 충돌을 해결하고 설명하는 모습 -->

**스크립트(.cs) 충돌**은 Claude가 잘 해결합니다. 양쪽 브랜치의 커밋을 읽고 "한쪽은 속도를 바꿨고, 다른 쪽은 점프를 추가했으니 둘 다 살린다"는 식으로 판단합니다. 해결한 뒤에는 꼭 유니티에서 컴파일 에러가 없는지, Play 모드에서 잘 동작하는지 확인합니다.

**씬(.unity), 프리팹(.prefab) 충돌**은 Claude에게 텍스트로 고치게 하면 안 됩니다. [2편](/ax/2026/11/22/Claude-Code-2%ED%8E%B8-CLAUDE.md%EC%99%80-%EC%BB%A8%ED%85%8D%EC%8A%A4%ED%8A%B8-%EA%B4%80%EB%A6%AC.html)에서 CLAUDE.md에 "씬 파일을 텍스트로 직접 편집하지 말 것"이라고 적어둔 이유입니다. 이런 파일은 어느 쪽을 쓸지만 정하게 합니다.

```
Main.unity 충돌은 main 쪽 버전으로 해결해줘
```

## 6. 이력 찾기

Git 이력은 "이 코드가 언제, 왜 이렇게 됐는지"를 알려주는 기록입니다. 하지만 직접 `git log`를 뒤지는 건 꽤 귀찮습니다. Claude에게 맡기면 편합니다.

```
PlayerController.cs의 점프 코드가 언제, 어떤 커밋에서 바뀌었는지 찾아줘
```

```
어제부터 점프가 이상해졌어. 최근 커밋 중에 점프나 물리에 영향을 준 변경이 있는지 찾아줘
```

```
지난주 금요일 이후 커밋들을 요약해줘. 팀 회의 때 공유할 거야.
```

Claude가 `git log`, `git blame`, `git diff`를 조합해서 찾아줍니다. 커밋 메시지를 잘 써둘수록 이 작업이 정확해집니다. [4편](/ax/2026/11/22/Claude-Code-4%ED%8E%B8-Skill-%EC%A7%81%EC%A0%91-%EB%A7%8C%EB%93%A4%EA%B8%B0.html)의 `commit-message` Skill에서 "왜 바꿨는지"를 적도록 한 이유가 여기에 있습니다.

## 7. 사고를 막는 방법

Claude에게 Git을 맡길 때 가장 조심해야 할 것은 **되돌리기 어려운 명령**입니다.

| 명령 | 위험한 이유 |
|---|---|
| `git push --force` | 원격 저장소의 이력을 덮어씁니다. 팀원의 커밋이 사라질 수 있습니다 |
| `git reset --hard` | 커밋하지 않은 변경을 지웁니다. 되돌릴 수 없습니다 |
| `git clean -fd` | Git이 관리하지 않는 파일을 지웁니다 |
| `git branch -D` | 병합하지 않은 브랜치를 강제로 지웁니다 |

Claude Code는 기본적으로 이런 명령을 쓰기 전에 확인을 구하지만, 확실하게 막고 싶다면 [5편](/ax/2026/11/22/Claude-Code-5%ED%8E%B8-Hook%EA%B3%BC-Subagent.html)에서 배운 **Hook**을 씁니다.

### force push 막는 Hook

`.claude/hooks/no-force-push.sh`

```bash
#!/bin/bash
input=$(cat)
if echo "$input" | grep -qE '"command": *"[^"]*git push[^"]*(--force|-f\b)'; then
  echo "git push --force는 사용할 수 없습니다. 사용자에게 직접 하라고 안내하세요." >&2
  exit 2
fi
exit 0
```

`.claude/settings.json`의 `PreToolUse`에 추가합니다. 5편의 폴더 보호 Hook과 함께 쓰면 다음과 같습니다.

```json
"PreToolUse": [
  {
    "matcher": "Edit|Write",
    "hooks": [
      { "type": "command", "command": "bash .claude/hooks/protect.sh" }
    ]
  },
  {
    "matcher": "Bash",
    "hooks": [
      { "type": "command", "command": "bash .claude/hooks/no-force-push.sh" }
    ]
  }
]
```

* `"matcher": "Bash"`: Claude가 터미널 명령을 실행할 때마다 검사합니다.
* `git push --force`, `git push -f`는 막히고, 일반 `git push`나 `git push --follow-tags`는 통과합니다.
* `--force-with-lease`(조금 더 안전한 강제 푸시)도 함께 막힙니다. 강제 푸시가 정말 필요하면 Sourcetree나 터미널에서 직접 하면 됩니다.

5편처럼 스크립트만 먼저 테스트해 볼 수 있습니다.

```bash
echo '{"tool_input":{"command":"git push --force origin main"}}' | bash .claude/hooks/no-force-push.sh; echo "exit=$?"
```

`exit=2`가 나오면 정상입니다.

### 그 밖의 습관

* **작업 전에 커밋**: Claude에게 큰 작업을 맡기기 전에 지금 상태를 커밋해 두면, 결과가 마음에 안 들 때 언제든 돌아갈 수 있습니다.
* **main에서 바로 작업하지 않기**: "브랜치 만들어서 작업해줘"를 습관처럼 붙입니다. 이 규칙을 CLAUDE.md에 적어둬도 됩니다.
* **푸시는 확인 후에**: Claude가 커밋한 내용을 Sourcetree 히스토리에서 한 번 보고 나서 푸시하라고 시킵니다.

## 마치며

이번 글에서는 Git 작업을 Claude Code에게 맡기는 방법을 정리했습니다.

* **커밋**: 변경 내용을 읽고 메시지를 써주고, 섞인 변경은 나눠서 커밋합니다.
* **브랜치, 푸시**: 말로 시키면 되지만, 푸시는 내가 지시할 때만 합니다.
* **PR**: `gh`로 PR을 만들고 설명까지 써줍니다. 유니티 설정 같은 "코드 밖 정보"를 꼭 넣게 합니다.
* **충돌**: 스크립트는 Claude에게 맡기고, 씬과 프리팹은 어느 쪽을 쓸지만 정하게 합니다.
* **이력**: "언제, 왜 바뀌었는지"를 물어보면 찾아줍니다.
* **안전**: 되돌리기 어려운 명령은 Hook으로 막습니다.

Git 명령을 전부 외우지 않아도, 무엇을 하고 싶은지만 알면 Claude에게 맡길 수 있습니다. 다만 **무슨 일이 일어났는지는 내가 이해하고 있어야** 합니다. Sourcetree로 결과를 눈으로 확인하는 습관을 함께 들이면 Git과 Claude 모두 훨씬 안심하고 쓸 수 있습니다.
