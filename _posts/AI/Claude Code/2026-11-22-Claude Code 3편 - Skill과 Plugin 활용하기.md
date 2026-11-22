---
layout: post

title: "[Claude Code] 3편 - Skill과 Plugin 활용하기 (PonyTail)"

# excerpt_image: /assets/images/posts/2026-11-22-Claude Code 3편/

categories:
  - AI

tags:
  - Claude Code
  - AI
  - Skill

toc: true
toc_sticky: true

published: true

date: 2026-11-22
last_modified_at: 2026-11-22
---

## 들어가며

[2편](/ai/2026/11/22/Claude-Code-2%ED%8E%B8-CLAUDE.md%EC%99%80-%EC%BB%A8%ED%85%8D%EC%8A%A4%ED%8A%B8-%EA%B4%80%EB%A6%AC.html)에서는 CLAUDE.md로 프로젝트 규칙을 알려주는 방법을 다뤘습니다. CLAUDE.md는 편리하지만 **항상** 읽힌다는 한계가 있습니다. "UI 만들 때 규칙", "적 만들 때 규칙", "커밋할 때 규칙"을 전부 넣으면 금방 길어지고, 지금 하는 작업과 상관없는 내용까지 매번 컨텍스트를 차지합니다.

이럴 때 쓰는 것이 **Skill**입니다. Skill은 특정 작업에 대한 지침서로, **필요할 때만** 꺼내 읽힙니다.

이번 글에서는 다른 사람이 만든 Skill을 설치해서 사용해 보겠습니다. 예시로는 Claude를 "최소한의 코드만 쓰는 시니어 개발자"로 바꿔주는 **PonyTail**을 사용합니다.

* Skill과 Plugin이 무엇인지
* PonyTail 설치하기
* PonyTail을 켜고 끄면서 결과 비교하기
* Skill 파일 구조 들여다보기
* Slash command로 직접 부르기

## 1. Skill과 Plugin이란?

### Skill

Skill은 **`SKILL.md` 파일이 들어 있는 폴더**입니다. `SKILL.md`에는 "어떤 작업을, 어떻게 하라"는 지침이 적혀 있습니다.

Skill의 핵심은 **필요할 때만 읽힌다**는 점입니다.

1. 평소에 Claude는 설치된 Skill들의 **이름과 한 줄 설명**만 알고 있습니다.
2. 사용자의 요청이 어떤 Skill의 설명과 맞으면, 그때 그 Skill의 **전체 내용**을 읽습니다.
3. 지침에 따라 작업합니다.

그래서 Skill을 수십 개 설치해도 컨텍스트를 거의 차지하지 않습니다. CLAUDE.md에 전부 적는 것과 가장 크게 다른 점입니다.

| | CLAUDE.md | Skill |
|---|---|---|
| 언제 읽히나 | 매 세션 항상 | 관련 작업이 나올 때만 |
| 적합한 내용 | 항상 지켜야 할 프로젝트 규칙 | 특정 작업에만 필요한 절차, 지식 |
| 공유 | 프로젝트마다 하나 | 폴더째로 복사하거나 Plugin으로 배포 |

### Plugin

Plugin은 **Skill, Slash command, Hook, Subagent 등을 하나로 묶어서 배포하는 단위**입니다. 앱 스토어처럼 **마켓플레이스**에서 찾아 설치할 수 있습니다.

PonyTail도 Plugin입니다. 안에 Skill 여러 개와 Hook이 함께 들어 있습니다.

## 2. PonyTail 설치하기

PonyTail은 GitHub에 공개된 Plugin입니다. 설치는 두 단계로 진행합니다.

먼저 PonyTail의 마켓플레이스를 등록합니다.

```
/plugin marketplace add DietrichGebert/ponytail
```

<!-- 📷 스크린샷: 마켓플레이스 추가 결과 -->

등록이 끝나면 Plugin을 설치합니다.

```
/plugin install ponytail@ponytail
```

`ponytail@ponytail`은 `플러그인이름@마켓플레이스이름` 형식입니다.

> 두 명령은 **따로따로** 입력해야 합니다. 한 번에 붙여넣으면 설치가 제대로 되지 않습니다.

<!-- 📷 스크린샷: 플러그인 설치 완료 -->

Desktop 앱에서는 입력창 옆의 `+` 버튼 → `Plugins` → `Add plugin`에서 목록을 보고 설치할 수도 있습니다.

설치가 끝나면 **새 세션**을 엽니다. 세션 시작 시 `PONYTAIL MODE ACTIVE`라는 안내가 보이면 정상적으로 설치된 것입니다.

> PonyTail은 내부적으로 Node.js로 만든 Hook을 사용합니다. `node: command not found` 같은 에러가 뜬다면 [Node.js](https://nodejs.org)를 설치해 주세요. 에러가 떠도 Skill 자체는 동작합니다.

### 설치 위치 확인

설치된 Plugin은 사용자 폴더 아래에 저장됩니다.

```
C:\Users\<사용자명>\.claude\plugins\
```

Plugin 목록을 보거나 끄고 켜려면 `/plugin`을 입력합니다.

## 3. 실습 1 - PonyTail 없이 만들기

비교를 위해 PonyTail을 먼저 꺼둡니다. 대화 중에 아래처럼 말하면 꺼집니다.

```
stop ponytail
```

그리고 다음과 같이 요청합니다.

```
플레이어 체력 시스템 만들어줘. 데미지 받기, 회복, 죽음 처리.
```

<!-- 📷 스크린샷: PonyTail 없이 만든 체력 시스템 -->

결과를 보고 다음을 메모해 둡니다.

* 만들어진 파일 수
* 전체 줄 수
* 요청하지 않았는데 추가된 기능 (이벤트 시스템, 무적 시간, 인터페이스, 체력바 UI 등)

확인이 끝나면 `/rewind`로 되돌리고, `/clear`로 대화도 비워줍니다.

## 4. 실습 2 - PonyTail 켜고 만들기

이번에는 PonyTail을 켭니다.

```
/ponytail
```

같은 요청을 보냅니다.

```
플레이어 체력 시스템 만들어줘. 데미지 받기, 회복, 죽음 처리.
```

<!-- 📷 스크린샷: PonyTail로 만든 체력 시스템 -->

두 결과를 비교해 봅니다.

| 항목 | PonyTail 끔 | PonyTail 켬 |
|---|---|---|
| 파일 수 | | |
| 줄 수 | | |
| 추가 기능 | | |
| 응답 마지막 | | 생략한 것, 확인하지 않은 것 안내 |

<!-- 실습 결과로 표를 채워주세요 -->

PonyTail을 켜면 보통 다음과 같이 달라집니다.

* 요청한 기능(데미지, 회복, 죽음)만 만듭니다.
* 새 파일이나 추상화(인터페이스, 매니저 클래스)를 잘 만들지 않습니다.
* 응답 끝에 **무엇을 생략했는지, 무엇을 확인하지 않았는지** 한두 줄로 알려줍니다.

### 어느 쪽이 더 좋을까?

정답은 없습니다.

* **프로토타입, 게임잼**: 빠르게 동작하는 것이 중요하니 PonyTail 쪽이 유리합니다.
* **오래 유지할 기반 코드**: 나중에 기능이 붙을 것이 확실하다면 확장성 있는 구조가 나을 수 있습니다.

중요한 것은 **Skill 하나로 Claude의 작업 성향 자체가 바뀐다**는 점입니다. 팀의 일하는 방식을 Skill로 만들어두면, 누가 Claude를 쓰든 같은 방식으로 작업하게 됩니다.

### 강도 조절

PonyTail은 강도를 조절할 수 있습니다.

```
/ponytail lite
/ponytail full
/ponytail ultra
```

`full`이 기본값이고, `lite`는 조금 느슨하게, `ultra`는 더 극단적으로 코드를 줄입니다. 다시 끄려면 `stop ponytail` 또는 `normal mode`라고 말하면 됩니다.

## 5. Skill 파일 들여다보기

다음 글에서 Skill을 직접 만들어볼 예정이니, PonyTail의 Skill 파일이 어떻게 생겼는지 미리 열어보겠습니다.

```
C:\Users\<사용자명>\.claude\plugins\cache\ponytail\ponytail\<버전>\skills\
```

이 폴더 안에 Skill이 여러 개 들어 있습니다.

```
skills/
├── ponytail/          ← 메인 Skill
│   └── SKILL.md
├── ponytail-review/   ← 코드 리뷰
│   └── SKILL.md
├── ponytail-audit/    ← 저장소 전체 점검
├── ponytail-debt/
├── ponytail-gain/
└── ponytail-help/
```

`ponytail/SKILL.md`를 열면 맨 위가 이렇게 생겼습니다.

```markdown
---
name: ponytail
description: >
  Lazy senior dev mode: the smallest change that fully solves the task, ...
  Use on any coding task (writing, fixing, refactoring, reviewing, choosing dependencies)
  and when the user says "ponytail", "be lazy", "simplest solution", "yagni", ...
---

# Ponytail

You are a lazy senior developer. ...
```

* `---` 사이의 부분을 **frontmatter**라고 합니다. `name`(이름)과 `description`(설명)이 들어갑니다.
* 그 아래가 실제 **지침**입니다.

`description`을 자세히 보면 **"무엇을 하는지"**와 함께 **"언제 쓰는지"**(any coding task, when the user says...)가 적혀 있습니다. Claude는 평소에 이 `description`만 보고 Skill을 쓸지 말지 결정합니다. 그래서 Skill을 만들 때 가장 신경 써야 하는 부분이 이 한 줄입니다. 이 내용은 다음 글에서 자세히 다룹니다.

## 6. Slash command로 직접 부르기

Skill이 실행되는 방법은 두 가지입니다.

| 방법 | 설명 | 예시 |
|---|---|---|
| 자동 | Claude가 요청을 보고 알아서 판단 | "코드 리뷰해줘" → ponytail-review가 불림 |
| 직접 | 사용자가 `/이름`으로 호출 | `/ponytail-review` |

입력창에 `/`를 입력하면 사용할 수 있는 명령 목록이 뜹니다. 설치한 Skill도 이 목록에 나타납니다.

<!-- 📷 스크린샷: / 입력 시 나타나는 명령 목록 -->

PonyTail에 들어 있는 다른 Skill도 직접 불러봅니다.

```
/ponytail-help
```

```
/ponytail-review
```

자동으로 불리게 둘지, 직접 부를 때만 쓰게 할지는 Skill을 만드는 사람이 정할 수 있습니다. 예를 들어 "배포하기" 같은 Skill이 Claude 판단으로 멋대로 실행되면 곤란하니, 직접 부를 때만 동작하게 만드는 것이 안전합니다.

## 7. Skill 설치 위치 정리

Plugin이 아닌 Skill 폴더 하나만 받았다면, 아래 위치에 폴더째로 넣으면 됩니다.

| 위치 | 적용 범위 |
|---|---|
| `~/.claude/skills/<스킬이름>/SKILL.md` | 내 모든 프로젝트 |
| `프로젝트/.claude/skills/<스킬이름>/SKILL.md` | 이 프로젝트. git에 올리면 팀 공유 |

## 마치며

이번 글에서는 Skill과 Plugin을 설치해서 사용해 봤습니다.

* **Skill**은 필요할 때만 읽히는 작업 지침서입니다. 그래서 많이 설치해도 부담이 적습니다.
* **Plugin**은 Skill, Hook 등을 묶어 배포하는 단위입니다. `/plugin`으로 관리합니다.
* Skill 하나로 Claude의 작업 방식 자체가 바뀝니다.
* Claude는 **`description`만 보고** Skill을 쓸지 결정합니다.

남이 만든 Skill도 좋지만, 진짜 힘을 발휘하는 건 **우리 팀 전용 Skill**입니다. 다음 글에서는 Skill을 직접 만들고, 제대로 불리는지 테스트하는 방법을 다뤄보겠습니다.
