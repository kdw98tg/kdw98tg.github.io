---
layout: post

title: "[Git] Unity 프로젝트 Git 시작하기 (Sourcetree, gitignore, 브랜치, PR)"

# excerpt_image: /assets/images/posts/2026-11-22-Git Sourcetree/

categories:
  - Git

tags:
  - Git
  - Sourcetree
  - GitHub
  - Unity

toc: true
toc_sticky: true

published: true

date: 2026-11-22
last_modified_at: 2026-11-22
---

## 들어가며

Claude Code에게 코드를 맡기다 보면 "아까 상태로 되돌리고 싶다"는 순간이 꼭 옵니다. Claude Code에도 `/rewind`가 있지만, Claude가 직접 고친 파일만 되돌릴 수 있습니다. 유니티 에디터에서 바꾼 씬이나 에셋까지 안전하게 지키려면 **Git**이 필요합니다.

이번 글에서는 Git을 처음 쓰는 분도 따라 할 수 있도록, Unity 프로젝트를 Git으로 관리하는 방법을 **Sourcetree**(GUI 프로그램)로 정리해 보겠습니다.

* Git 기본 개념 (커밋, 푸시, 풀, 브랜치)
* Unity 프로젝트를 Git에 올리기 전 설정
* `.gitignore` 작성
* Sourcetree로 커밋, 푸시, 풀
* 브랜치 만들고 병합하기
* GitHub에서 Pull Request(PR) 만들기
* 큰 파일을 위한 Git LFS

## 1. Git 기본 개념

Git은 파일의 변경 이력을 저장하는 프로그램입니다. 게임의 **세이브 포인트**라고 생각하면 이해하기 쉽습니다.

| 용어 | 의미 | 비유 |
|---|---|---|
| 저장소 (Repository) | Git이 관리하는 프로젝트 폴더 | 세이브 파일 묶음 |
| 커밋 (Commit) | 현재 상태를 기록으로 남기기 | 세이브 포인트 만들기 |
| 스테이지 (Stage) | 이번 커밋에 넣을 파일 고르기 | 세이브할 항목 고르기 |
| 원격 저장소 (Remote) | GitHub 같은 서버에 있는 저장소 | 클라우드 세이브 |
| 푸시 (Push) | 내 커밋을 원격 저장소로 올리기 | 클라우드에 업로드 |
| 풀 (Pull) | 원격 저장소의 커밋을 내려받기 | 클라우드에서 다운로드 |
| 패치 (Fetch) | 원격에 새 커밋이 있는지 확인만 하기 | 업데이트 확인 |
| 브랜치 (Branch) | 독립된 작업 공간 | 세이브 슬롯 나누기 |
| 병합 (Merge) | 브랜치의 작업을 합치기 | 두 세이브 합치기 |

작업 흐름은 보통 이렇습니다.

```
파일 수정 → 스테이지 → 커밋 → 푸시
                              ↑
              다른 사람 작업 ← 풀
```

## 2. 준비물 설치

### Git

[git-scm.com](https://git-scm.com)에서 설치합니다. 설치 옵션은 기본값 그대로 두면 됩니다. Claude Code를 Windows에서 쓰고 있다면 이미 설치되어 있을 가능성이 높습니다.

설치 확인은 터미널에서 합니다.

```bash
git --version
```

### Sourcetree

[sourcetreeapp.com](https://www.sourcetreeapp.com)에서 설치합니다.

1. 설치 중에 Bitbucket 계정 연결 화면이 나오면 **건너뛰기(Skip)**를 눌러도 됩니다.
2. Mercurial을 찾을 수 없다는 안내가 나오면 **Mercurial 사용 안 함**을 선택합니다.
3. 마지막에 이름과 이메일을 입력합니다. 커밋에 작성자로 기록되는 정보입니다.

<!-- 📷 스크린샷: Sourcetree 설치 - 이름, 이메일 입력 화면 -->

### GitHub 계정 연결

원격 저장소로 GitHub를 쓰겠습니다. [github.com](https://github.com)에 가입되어 있어야 합니다.

Sourcetree에서 `도구 > 옵션 > 인증` 탭으로 가서 `추가`를 누릅니다.

* 호스팅 서비스: `GitHub`
* 인증 방식: `OAuth`
* `OAuth 토큰 새로 고침`을 누르면 브라우저가 열리고, GitHub 로그인 후 승인하면 연결됩니다.

<!-- 📷 스크린샷: Sourcetree 인증 탭에서 GitHub 계정 추가 -->

> 이 글은 Sourcetree 3.4 버전 기준입니다. 메뉴 이름은 버전이나 언어 설정에 따라 조금 다를 수 있습니다.

## 3. Unity 프로젝트 설정 확인

Git에 올리기 전에 유니티에서 두 가지를 확인합니다. Unity 6에서는 둘 다 기본값으로 설정되어 있지만, 오래된 프로젝트라면 꼭 확인해야 합니다.

`Edit > Project Settings`에서:

| 항목 | 값 | 이유 |
|---|---|---|
| Version Control > Mode | `Visible Meta Files` | `.meta` 파일이 보여야 Git에 함께 올라갑니다 |
| Editor > Asset Serialization > Mode | `Force Text` | 씬, 프리팹이 텍스트로 저장되어야 변경 내용을 비교할 수 있습니다 |

<!-- 📷 스크린샷: Project Settings의 Version Control, Asset Serialization 설정 -->

### .meta 파일은 꼭 함께 올립니다

유니티는 에셋마다 `.meta` 파일을 만들고, 그 안에 에셋의 고유 ID(GUID)를 저장합니다. 씬이나 프리팹은 이 ID로 다른 에셋을 참조합니다.

`.meta` 파일을 빼먹고 올리면, 다른 사람의 컴퓨터에서 유니티가 새 ID를 만들어 버립니다. 그러면 **씬에 연결해 둔 스크립트나 머티리얼이 전부 끊어집니다.** 에셋 파일과 `.meta` 파일은 항상 같이 커밋해야 합니다.

## 4. 저장소 만들기

### 로컬 저장소 만들기

Sourcetree에서 `파일 > 새로 만들기/복제`를 누르고 `Create` 탭을 선택합니다.

* 목적지 경로: Unity 프로젝트 폴더 (`Assets` 폴더가 있는 곳)
* 종류: `Git`

폴더가 이미 있다는 경고가 나오면 `예`를 누릅니다. 프로젝트 파일은 그대로 두고, Git 관리 정보가 담긴 `.git` 폴더만 추가됩니다.

<!-- 📷 스크린샷: Sourcetree Create 탭 -->

저장소를 만들면 `파일 상태` 화면에 프로젝트의 모든 파일이 나타납니다. 그런데 **아직 커밋하면 안 됩니다.** `Library` 폴더처럼 올리면 안 되는 파일까지 수만 개가 들어 있기 때문입니다. 먼저 `.gitignore`를 만들어야 합니다.

## 5. .gitignore 작성

### .gitignore란?

`.gitignore`는 **Git이 무시할 파일 목록**입니다. 여기 적힌 파일과 폴더는 커밋 대상에서 빠집니다.

유니티 프로젝트에서 올리면 안 되는 것은 다음과 같습니다.

| 폴더/파일 | 이유 |
|---|---|
| `Library/` | 유니티가 자동으로 만드는 캐시. 수 GB가 되기도 하고, 다른 컴퓨터에서 다시 만들어집니다 |
| `Temp/`, `Logs/`, `obj/` | 임시 파일, 로그 |
| `UserSettings/` | 내 에디터 레이아웃 같은 개인 설정 |
| `Build/`, `Builds/` | 빌드 결과물 |
| `*.csproj`, `*.sln`, `*.slnx` | 유니티가 자동으로 만드는 IDE 프로젝트 파일 |

**반드시 올려야 하는 것**은 `Assets/`, `Packages/`, `ProjectSettings/` 세 폴더입니다.

### 파일 만들기

프로젝트 루트(`Assets` 폴더와 같은 위치)에 `.gitignore` 파일을 만들고 아래 내용을 넣습니다. GitHub에서 제공하는 Unity용 템플릿을 바탕으로 정리한 것입니다.

```gitignore
# Unity 자동 생성 폴더
/[Ll]ibrary/
/[Tt]emp/
/[Oo]bj/
/[Bb]uild/
/[Bb]uilds/
/[Ll]ogs/
/[Uu]ser[Ss]ettings/
/[Mm]emoryCaptures/
/[Rr]ecordings/

# IDE 자동 생성 파일
.vs/
.vscode/
.idea/
*.csproj
*.unityproj
*.sln
*.slnx
*.suo
*.user
*.pidb
*.booproj
*.pdb
*.mdb
*.opendb
*.VC.db

# 빌드 결과물
*.apk
*.aab
*.unitypackage
sysinfo.txt

# OS 파일
.DS_Store
Thumbs.db

# Claude Code 개인 설정 (팀 공유용 .claude/settings.json 은 올립니다)
.claude/settings.local.json
CLAUDE.local.md
```

> Windows 탐색기에서는 이름이 `.`으로 시작하는 파일을 만들기 어려울 수 있습니다. VS Code 같은 에디터에서 만들거나, Claude Code에게 "Unity용 .gitignore 만들어줘"라고 부탁해도 됩니다.

저장하고 Sourcetree로 돌아가면, `파일 상태` 목록이 크게 줄어든 것을 확인할 수 있습니다. `Library` 폴더 파일이 보이지 않으면 성공입니다.

<!-- 📷 스크린샷: .gitignore 적용 후 파일 상태 목록 -->

### 이미 커밋한 파일은 무시되지 않습니다

`.gitignore`는 **아직 Git이 관리하지 않는 파일**에만 적용됩니다. 실수로 `Library` 폴더를 이미 커밋했다면 `.gitignore`에 추가해도 계속 추적됩니다. 이 경우 터미널에서 Git 관리 목록에서만 빼주고(파일은 남아 있습니다) 다시 커밋합니다.

```bash
git rm -r --cached Library
```

## 6. 첫 커밋

`파일 상태` 화면에서 진행합니다.

1. 아래쪽 `스테이지에 올라가지 않은 파일` 목록에서 `모두 스테이지에 올리기`를 누릅니다.
2. 파일들이 위쪽 `스테이지에 올라간 파일`로 이동합니다.
3. 맨 아래 메시지 칸에 커밋 메시지를 적습니다. 예: `[기능] 프로젝트 초기 설정`
4. `커밋` 버튼을 누릅니다.

<!-- 📷 스크린샷: 스테이지 후 커밋 메시지 입력 화면 -->

왼쪽 사이드바의 `히스토리`를 누르면 방금 만든 커밋이 기록된 것을 볼 수 있습니다.

### 파일 일부만 커밋하기

스테이지는 "이번 커밋에 넣을 파일 고르기"입니다. 플레이어 작업과 UI 작업을 동시에 했다면, 플레이어 파일만 스테이지에 올려서 먼저 커밋하고, UI 파일은 따로 커밋할 수 있습니다. 커밋 하나에는 **한 가지 작업**만 담는 것이 나중에 되돌리거나 찾아보기 좋습니다.

## 7. GitHub에 올리기 (Push)

### GitHub 저장소 만들기

[github.com](https://github.com)에서 오른쪽 위 `+` → `New repository`를 누릅니다.

* Repository name: 프로젝트 이름
* Public / Private: 공개 여부 선택
* **`Add a README file`, `Add .gitignore`는 체크하지 않습니다.** 이미 로컬에 파일이 있으니 빈 저장소로 만들어야 충돌이 생기지 않습니다.

`Create repository`를 누르면 저장소 주소(`https://github.com/<아이디>/<저장소>.git`)가 나옵니다. 복사해 둡니다.

<!-- 📷 스크린샷: GitHub 새 저장소 생성 화면 -->

### 원격 저장소 연결

Sourcetree에서 `저장소 > 저장소 설정 > 원격` 탭으로 가서 `추가`를 누릅니다.

* 리모트 이름: `origin` (기본 이름입니다)
* URL / 경로: 복사한 GitHub 저장소 주소

<!-- 📷 스크린샷: 원격 저장소 추가 화면 -->

### 푸시

상단 툴바의 `푸시` 버튼을 누릅니다. 올릴 브랜치(`main` 또는 `master`)를 체크하고 `푸시`를 누릅니다.

<!-- 📷 스크린샷: 푸시 창 -->

GitHub 저장소 페이지를 새로고침하면 `Assets`, `Packages`, `ProjectSettings` 폴더가 올라간 것을 확인할 수 있습니다.

> 커밋 화면 아래의 `origin/main에 바로 변경 사항 푸시` 체크박스를 켜두면 커밋할 때마다 자동으로 푸시됩니다.

## 8. 내려받기 (Pull, Fetch)

다른 사람이 푸시한 커밋이나 다른 컴퓨터에서 작업한 내용을 받아오려면 `풀`을 누릅니다.

| 버튼 | 동작 |
|---|---|
| 패치 | 원격 저장소에 새 커밋이 있는지 **확인만** 합니다. 내 파일은 바뀌지 않습니다 |
| 풀 | 새 커밋을 받아서 내 파일에 **반영**합니다 |

Sourcetree 툴바의 `풀` 버튼 위에 숫자가 표시되면, 그만큼 받아올 커밋이 있다는 뜻입니다.

### 다른 컴퓨터에서 처음 받을 때 (Clone)

다른 컴퓨터에서 이 프로젝트를 처음 받을 때는 `파일 > 새로 만들기/복제 > Clone` 탭에서 GitHub 저장소 주소를 넣으면 됩니다. 받은 뒤 Unity Hub에서 `Add > Add project from disk`로 폴더를 열면, `Library` 폴더가 다시 만들어지면서 프로젝트가 열립니다. (처음 열 때는 시간이 조금 걸립니다.)

### 풀 하기 전에 주의할 점

* **유니티에서 씬을 저장한 뒤** 풀을 합니다. 저장하지 않은 씬이 있으면 받아온 변경 내용과 꼬일 수 있습니다.
* 커밋하지 않은 변경 사항이 있으면 풀이 거부될 수 있습니다. 먼저 커밋하거나, `스태시`로 잠시 치워둡니다.

## 9. 브랜치

### 왜 브랜치를 쓰나

`main` 브랜치에서 바로 작업하다가 실험이 실패하면, 되돌리기가 번거롭습니다. 새 기능은 **브랜치를 따로 만들어서** 작업하고, 완성되면 `main`에 합치는 것이 안전합니다.

```
main     ──●──────────────●── (병합)
            \            /
feature/coin ●──●──●──●─
```

### 브랜치 만들기

툴바의 `브랜치` 버튼을 누르고 이름을 입력합니다.

* 이름 예시: `feature/coin`, `fix/jump-bug`
* `새 브랜치 체크아웃` 체크

<!-- 📷 스크린샷: 새 브랜치 만들기 창 -->

왼쪽 사이드바 `브랜치` 목록에서 **굵게 표시된 것**이 지금 작업 중인 브랜치입니다. 다른 브랜치로 옮기려면 이름을 **더블 클릭**합니다.

> 브랜치를 옮기면 프로젝트 파일이 그 브랜치의 상태로 바뀝니다. 유니티 에디터가 열려 있다면 바뀐 에셋을 다시 불러옵니다. **브랜치를 옮기기 전에 씬을 꼭 저장하세요.**

### 병합하기

`feature/coin` 브랜치에서 작업을 마쳤다면 `main`에 합칩니다.

1. `main` 브랜치를 더블 클릭해서 체크아웃합니다.
2. 사이드바에서 `feature/coin`을 오른쪽 클릭합니다.
3. `현재 브랜치로 feature/coin 병합`을 선택합니다.

<!-- 📷 스크린샷: 브랜치 병합 메뉴 -->

### 충돌이 났을 때

두 브랜치에서 같은 파일의 같은 부분을 고쳤다면 **충돌(Conflict)**이 납니다. Sourcetree의 `파일 상태`에 느낌표 아이콘이 붙은 파일이 충돌 난 파일입니다.

* **스크립트(.cs) 충돌**: 파일을 열면 아래처럼 표시되어 있습니다. 둘 중 하나를 고르거나 합쳐서 고친 뒤, 표시(`<<<<<<<`, `=======`, `>>>>>>>`)를 지우고 저장합니다.

  ```
  <<<<<<< HEAD
  [SerializeField] private float speed = 5f;
  =======
  [SerializeField] private float speed = 8f;
  >>>>>>> feature/coin
  ```

* **씬(.unity), 프리팹(.prefab) 충돌**: 텍스트로 직접 고치기 어렵습니다. 파일을 오른쪽 클릭하고 `충돌 해결 > 내 것을 사용하여 해결` 또는 `그들의 것을 사용하여 해결` 중 하나를 고릅니다.

고친 파일을 스테이지에 올리고 커밋하면 병합이 끝납니다.

### Unity에서 충돌을 줄이는 방법

씬 파일 충돌은 해결하기가 정말 어렵습니다. 미리 피하는 것이 최선입니다.

* **한 씬은 한 사람만** 작업합니다. 동시에 작업해야 한다면 미리 이야기합니다.
* 씬에 직접 배치하기보다 **프리팹으로 나눠서** 작업합니다. 각자 다른 프리팹을 고치면 충돌이 나지 않습니다.
* 작업을 시작하기 전에 항상 **풀**부터 합니다.

## 10. Pull Request (PR)

### PR이란?

PR은 "내 브랜치를 `main`에 합쳐주세요"라는 **요청**입니다. 혼자 바로 병합하는 대신, 팀원이 변경 내용을 보고 리뷰한 뒤 병합합니다.

| | 직접 병합 | Pull Request |
|---|---|---|
| 어디서 | 내 컴퓨터 (Sourcetree) | GitHub 웹사이트 |
| 리뷰 | 없음 | 팀원이 코드를 보고 코멘트 |
| 적합한 경우 | 혼자 작업 | 팀 작업 |

### PR 만들기

1. Sourcetree에서 `feature/coin` 브랜치를 **푸시**합니다.
2. GitHub 저장소 페이지에 가면 노란 안내창에 `Compare & pull request` 버튼이 생깁니다. 누릅니다.
3. 제목과 설명을 적습니다. 무엇을 왜 바꿨는지, 어떻게 테스트했는지 적어두면 리뷰하는 사람이 편합니다.
4. `Create pull request`를 누릅니다.

<!-- 📷 스크린샷: GitHub Compare & pull request 화면 -->

### 리뷰와 병합

* `Files changed` 탭에서 바뀐 내용을 줄 단위로 볼 수 있고, 줄마다 코멘트를 달 수 있습니다.
* 리뷰가 끝나면 `Merge pull request` 버튼으로 병합합니다.
* 병합한 뒤에는 Sourcetree에서 `main` 브랜치로 옮기고 **풀**을 해서 최신 상태로 맞춥니다.

## 11. 큰 파일은 Git LFS로

Git은 텍스트 파일 관리에 맞게 만들어져 있습니다. `.psd`, `.fbx`, `.wav` 같은 큰 파일을 그대로 커밋하면 저장소가 금방 무거워집니다. GitHub는 100MB가 넘는 파일은 아예 받지 않습니다.

**Git LFS**(Large File Storage)를 쓰면 큰 파일은 따로 저장하고, 저장소에는 "파일이 어디 있는지"만 기록합니다.

Sourcetree에서 `저장소 > Git LFS > 저장소 초기화`를 누릅니다. 그다음 프로젝트 루트에 `.gitattributes` 파일을 만들고 LFS로 관리할 확장자를 적습니다.

```gitattributes
*.psd filter=lfs diff=lfs merge=lfs -text
*.fbx filter=lfs diff=lfs merge=lfs -text
*.wav filter=lfs diff=lfs merge=lfs -text
*.mp3 filter=lfs diff=lfs merge=lfs -text
*.png filter=lfs diff=lfs merge=lfs -text
```

> LFS는 **처음부터** 설정하는 것이 좋습니다. 이미 일반 커밋으로 올린 큰 파일을 LFS로 옮기려면 이력을 다시 써야 해서 번거롭습니다. 또 GitHub의 무료 LFS 저장 용량에는 한도가 있으니, 에셋이 많은 프로젝트라면 미리 확인해 두세요.

## 마치며

이번 글에서는 Unity 프로젝트를 Git으로 관리하는 기본기를 정리했습니다.

* **`.gitignore`**로 `Library` 같은 자동 생성 폴더를 빼고, `Assets`, `Packages`, `ProjectSettings`만 올립니다.
* **`.meta` 파일**은 에셋과 항상 함께 커밋합니다.
* **커밋 → 푸시**로 올리고, **풀**로 받습니다. 풀 하기 전에는 씬을 저장합니다.
* 새 기능은 **브랜치**에서 작업하고, 팀 작업이라면 **PR**로 리뷰받은 뒤 병합합니다.
* 큰 파일은 처음부터 **Git LFS**로 관리합니다.

이제 Git의 기본기를 익혔으니, 다음 글에서는 이 작업들을 **Claude Code에게 맡기는 방법**을 다뤄보겠습니다. 커밋 메시지 작성, 브랜치 만들기, PR 작성, 충돌 해결까지 Claude에게 시킬 수 있습니다.
