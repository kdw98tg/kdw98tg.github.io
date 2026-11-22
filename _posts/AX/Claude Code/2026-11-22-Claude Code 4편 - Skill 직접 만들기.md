---
layout: post

title: "[Claude Code] 4편 - Skill 직접 만들기"

# excerpt_image: /assets/images/posts/2026-11-22-Claude Code 4편/

categories:
  - AX

tags:
  - Claude Code
  - AI
  - Skill
  - Unity

toc: true
toc_sticky: true

published: true

date: 2026-11-22
last_modified_at: 2026-11-22
---

## 들어가며

[3편](/ax/2026/11/22/Claude-Code-3%ED%8E%B8-Skill%EA%B3%BC-Plugin-%ED%99%9C%EC%9A%A9%ED%95%98%EA%B8%B0.html)에서는 PonyTail을 설치해서, Skill 하나로 Claude의 작업 방식이 바뀌는 것을 확인했습니다. 이번에는 **우리 팀 전용 Skill**을 직접 만들어 보겠습니다.

Skill로 만들 만한 것을 찾는 방법은 간단합니다. **Claude에게 같은 설명을 두 번 이상 하고 있다면, 그게 Skill 후보입니다.** "커밋 메시지는 이 형식으로 써줘", "적 만들 때는 EnemyBase를 상속해줘" 같은 설명을 매번 반복하고 있다면 Skill로 만들어 두면 됩니다.

이번 글에서는 난이도 순서로 세 가지 Skill을 만들어 봅니다.

* **예제 A (쉬움)**: 커밋 메시지 규칙
* **예제 B (보통)**: Unity 스크립트 컨벤션
* **예제 C (어려움)**: 적 캐릭터 생성 (참고 파일 포함)

그리고 Skill이 제대로 불리는지 테스트하는 방법과, skill-creator로 Skill을 쉽게 만드는 방법도 함께 다룹니다.

## 1. Skill의 구조

Skill은 폴더 하나입니다. 최소 구성은 `SKILL.md` 파일 하나뿐입니다.

```
.claude/skills/
└── commit-message/       ← 폴더 이름 = Skill 이름
    └── SKILL.md
```

`SKILL.md`는 두 부분으로 나뉩니다.

```markdown
---
name: commit-message
description: 무엇을 하는지 + 언제 쓰는지
---

# 여기부터 실제 지침

Claude가 따라야 할 내용을 자유롭게 적습니다.
```

| 부분 | 역할 | 언제 읽히나 |
|---|---|---|
| `name` | Skill 이름. `/commit-message`처럼 직접 부를 때도 씁니다 | 항상 |
| `description` | Claude가 이 Skill을 쓸지 판단하는 기준 | 항상 |
| 본문 | 실제 작업 지침 | Skill이 불렸을 때만 |

### description이 가장 중요합니다

Claude는 평소에 **`description`만** 봅니다. 본문을 아무리 잘 써도 `description`이 애매하면 Skill이 불리지 않습니다.

좋은 `description`에는 두 가지가 들어갑니다.

1. **무엇을 하는지**: "팀 커밋 규칙에 맞춰 커밋 메시지를 작성한다"
2. **언제 쓰는지**: "git 커밋을 하거나 커밋 메시지를 작성할 때 사용"

| 나쁜 예 | 좋은 예 |
|---|---|
| 커밋 도우미 | git 커밋 메시지를 작성할 때 사용. 팀 커밋 규칙에 맞춰 한국어로 작성한다. |
| 유니티 스크립트 | Unity C# 스크립트를 새로 만들거나 수정할 때 사용. 팀 코딩 컨벤션을 적용한다. |

## 2. 예제 A - 커밋 메시지 규칙 (쉬움)

### 파일 만들기

프로젝트 루트에 `.claude/skills/commit-message/` 폴더를 만들고, 그 안에 `SKILL.md`를 만듭니다.

````markdown
---
name: commit-message
description: git 커밋을 하거나 커밋 메시지를 작성할 때 사용. 팀 커밋 규칙에 맞춰 한국어로 작성한다.
---

# 커밋 메시지 규칙

형식: `[태그] 요약 (50자 이내)`

태그:
- [기능] 새 기능 추가
- [수정] 버그 수정
- [리팩터] 동작 변화 없는 코드 정리
- [에셋] 모델, 텍스처, 사운드 등 에셋 변경

본문에는 "왜" 바꿨는지를 적는다. "무엇을" 바꿨는지는 diff로 알 수 있으므로 쓰지 않는다.

여러 종류의 변경이 섞여 있으면 커밋을 나누자고 제안한다.

## 예시

```
[수정] 공중 무한 점프 버그 수정

땅 체크 없이 AddForce를 호출해서 연타 시 계속 점프되던 문제.
```
````

<!-- 📷 스크린샷: 폴더 구조와 SKILL.md -->

> Skill 파일을 새로 만들거나 고친 뒤에는 **새 세션**을 열어야 확실하게 반영됩니다.

### 테스트

프로젝트가 git 저장소가 아니라면 먼저 `git init`으로 만들어 줍니다. 그다음 아무 파일이나 수정하고 요청합니다.

```
지금 변경사항 커밋해줘
```

<!-- 📷 스크린샷: commit-message Skill이 불린 모습 -->

Claude가 `commit-message` Skill을 불러오고, `[수정] ...` 형식으로 커밋 메시지를 작성하는지 확인합니다.

## 3. 예제 B - Unity 스크립트 컨벤션 (보통)

[2편](/ax/2026/11/22/Claude-Code-2%ED%8E%B8-CLAUDE.md%EC%99%80-%EC%BB%A8%ED%85%8D%EC%8A%A4%ED%8A%B8-%EA%B4%80%EB%A6%AC.html)에서 CLAUDE.md에 코딩 컨벤션을 적었습니다. 이 내용을 Skill로 옮겨보겠습니다.

### 왜 옮기나?

CLAUDE.md는 **항상** 읽힙니다. 커밋을 하든, 질문을 하든, 셰이더를 보든 C# 컨벤션이 함께 읽힙니다. 컨벤션을 Skill로 옮기면 **C# 스크립트를 다룰 때만** 읽히니 컨텍스트를 아낄 수 있습니다.

> 프로젝트가 작을 때는 CLAUDE.md에 둬도 충분합니다. CLAUDE.md가 길어지기 시작하면 이렇게 Skill로 나누는 것을 고려하면 됩니다.

### 파일 만들기

`.claude/skills/unity-script-convention/SKILL.md`

````markdown
---
name: unity-script-convention
description: Unity C# 스크립트를 새로 만들거나 수정할 때 사용. MonoBehaviour 작성 시 팀 코딩 컨벤션을 적용한다.
---

# Unity 스크립트 컨벤션

## 필드
- Inspector에 노출할 필드는 `[SerializeField] private`로 선언한다. `public` 필드는 쓰지 않는다.
- 다른 스크립트에서 읽어야 하면 프로퍼티로 공개한다: `public int Hp => hp;`

## 생명주기
- `GetComponent`는 `Awake`에서 한 번만 호출하고 필드에 저장한다.
- `Update`에서 `GetComponent`, `GameObject.Find`, `FindObjectOfType`을 호출하지 않는다.
- 물리 처리(Rigidbody)는 `FixedUpdate`에서 한다. 입력은 `Update`에서 받아서 필드에 저장한다.

## Unity 6
- `Rigidbody.velocity` 대신 `Rigidbody.linearVelocity`를 쓴다.
- 입력은 새 Input System(`UnityEngine.InputSystem`)을 쓴다.

## 파일
- 클래스 하나당 파일 하나. 파일명과 클래스명을 같게 한다.
- 용도에 맞는 폴더에 둔다: `Assets/Scripts/Player/`, `Enemy/`, `UI/`

## 예시

```csharp
using UnityEngine;

public class Mover : MonoBehaviour
{
    [SerializeField] private float speed = 5f;
    private Rigidbody rb;
    private Vector3 input;

    private void Awake() => rb = GetComponent<Rigidbody>();

    private void FixedUpdate() => rb.linearVelocity = input * speed;
}
```
````

예시 코드를 넣어두면 Claude가 규칙을 훨씬 정확하게 따릅니다. 규칙을 글로만 설명하는 것보다 **"이렇게 생긴 코드"**를 보여주는 게 효과적입니다.

### 테스트

CLAUDE.md에서 컨벤션 부분을 지우고 새 세션을 연 뒤 요청합니다.

```
아이템을 줍는 스크립트 만들어줘
```

결과 코드가 `[SerializeField] private`, `Awake`에서 캐싱 같은 규칙을 지키는지 확인합니다.

## 4. 예제 C - 적 캐릭터 생성 (어려움)

이번에는 **참고 파일**이 있는 Skill을 만들어 봅니다. Skill 폴더에는 `SKILL.md` 외에 다른 파일도 넣을 수 있고, 본문에서 그 파일을 읽으라고 안내하면 됩니다.

### 폴더 구조

```
.claude/skills/new-enemy/
├── SKILL.md
└── template.cs
```

### SKILL.md

```markdown
---
name: new-enemy
description: 새로운 적 캐릭터나 몬스터를 추가해달라는 요청에 사용. 스크립트, 폴더 위치, 기본 구조를 팀 표준대로 만든다.
---

# 새 적 추가 절차

1. `Assets/Scripts/Enemy/EnemyBase.cs`가 있는지 확인한다.
   없으면 이 Skill 폴더의 `template.cs`를 참고해서 먼저 만든다.
2. `Assets/Scripts/Enemy/{이름}Enemy.cs`를 만들고 `EnemyBase`를 상속한다.
3. 적마다 다른 행동은 `Move()`와 `Attack()`을 override해서 구현한다.
4. 공통 필드(maxHp, moveSpeed, attackDamage)는 EnemyBase에 있으니 다시 선언하지 않는다.
5. 다 만들고 나면 Inspector에서 설정해야 할 값을 목록으로 알려준다.

## 이름 규칙
- 클래스: `SlimeEnemy`, `GoblinEnemy` (끝에 Enemy)
- 파일: 클래스 이름과 같게
```

### template.cs

```csharp
using UnityEngine;

[RequireComponent(typeof(Rigidbody))]
public abstract class EnemyBase : MonoBehaviour
{
    [SerializeField] protected int maxHp = 10;
    [SerializeField] protected float moveSpeed = 3f;
    [SerializeField] protected int attackDamage = 1;

    protected Rigidbody rb;
    protected Transform target;
    protected int hp;

    protected virtual void Awake()
    {
        rb = GetComponent<Rigidbody>();
        hp = maxHp;
    }

    protected virtual void Start()
    {
        var player = GameObject.FindWithTag("Player");
        if (player != null) target = player.transform;
    }

    private void FixedUpdate()
    {
        if (target != null) Move();
    }

    protected abstract void Move();
    protected abstract void Attack();

    public void TakeDamage(int damage)
    {
        hp -= damage;
        if (hp <= 0) Destroy(gameObject);
    }
}
```

> `template.cs`는 Skill 폴더(`.claude/`) 안에 있어서 Unity가 컴파일하지 않습니다. Claude가 읽고 참고하는 용도입니다.

### 테스트

```
슬라임 적 추가해줘. 통통 튀면서 플레이어를 따라와.
```

<!-- 📷 스크린샷: new-enemy Skill로 만든 SlimeEnemy -->

확인할 것:

* `EnemyBase.cs`가 먼저 만들어졌는지 (처음 실행이라면)
* `SlimeEnemy.cs`가 `Assets/Scripts/Enemy/`에 있고 `EnemyBase`를 상속하는지
* `Move()`에 튀는 동작이 구현되었는지
* 마지막에 Inspector 설정값 목록을 알려줬는지

이어서 하나 더 요청해 봅니다.

```
고블린 적도 추가해줘. 가까이 오면 공격해.
```

이번에는 `EnemyBase`를 새로 만들지 않고 기존 것을 상속하는지 확인합니다.

## 5. Skill이 제대로 불리는지 테스트하기

Skill을 만들었다면 **불려야 할 때 불리고, 불리면 안 될 때 안 불리는지** 둘 다 확인해야 합니다.

`commit-message` Skill을 예로 들면 다음과 같습니다.

| 요청 | 불려야 하나 |
|---|---|
| "지금 변경사항 커밋해줘" | O |
| "커밋 메시지 추천해줘" | O |
| "작업 끝났으니 정리해서 올려줘" | O (애매함) |
| "git이 뭐야?" | X |
| "커밋 히스토리 보여줘" | X |

새 세션마다 하나씩 요청해 보고, Skill이 불렸는지 확인합니다. Skill이 불리면 응답 중에 Skill을 불러오는 과정이 표시됩니다.

### 안 불릴 때

`description`에 그 상황을 추가합니다.

```yaml
description: git 커밋을 하거나, 커밋 메시지를 작성하거나, 작업 내용을 정리해서 올릴 때 사용. 팀 커밋 규칙에 맞춰 한국어로 작성한다.
```

### 엉뚱하게 불릴 때

`description`을 더 좁히거나, 쓰면 안 되는 상황을 적습니다.

```yaml
description: git 커밋 메시지를 새로 작성할 때 사용. 커밋 히스토리 조회나 git 사용법 질문에는 쓰지 않는다.
```

### 직접 부를 때만 쓰게 하기

Claude가 알아서 부르면 곤란한 Skill(배포, 데이터 삭제 등)이라면 frontmatter에 아래 줄을 추가합니다. 그러면 `/이름`으로 직접 부를 때만 동작합니다.

```yaml
disable-model-invocation: true
```

## 6. skill-creator로 쉽게 만들기

Skill을 처음부터 직접 쓰는 대신, Skill 만드는 것을 도와주는 Skill인 **skill-creator**를 쓸 수도 있습니다.

* **Claude Desktop 앱**에는 기본으로 들어 있습니다.
* **CLI**에서는 Anthropic의 공식 Skill 저장소([github.com/anthropics/skills](https://github.com/anthropics/skills))를 마켓플레이스로 추가한 뒤, `/plugin`에서 skill-creator가 들어 있는 Plugin을 설치합니다.

```
/plugin marketplace add anthropics/skills
```

설치가 되었다면 이렇게 요청합니다.

```
우리 팀 커밋 메시지 규칙을 Skill로 만들고 싶어. 도와줘.
```

<!-- 📷 스크린샷: skill-creator와 대화하는 모습 -->

skill-creator는 다음 순서로 진행합니다.

1. 어떤 Skill인지, 언제 써야 하는지 질문합니다.
2. `SKILL.md` 초안을 만듭니다.
3. 테스트용 요청을 만들어서, Skill이 잘 불리는지 확인합니다.
4. 결과를 보고 `description`을 다듬습니다.

5절에서 직접 했던 테스트 과정을 자동으로 해주기 때문에, Skill이 복잡해질수록 유용합니다.

## 7. 좋은 Skill을 만드는 요령

* **하나의 일만** 하게 합니다. "커밋도 하고 PR도 만들고 리뷰도 하는" Skill보다 세 개로 나누는 게 낫습니다.
* **구체적으로** 씁니다. "깔끔하게"가 아니라 "`[SerializeField] private` 사용"처럼 씁니다.
* **예시를 넣습니다.** 규칙 열 줄보다 예시 코드 하나가 더 정확하게 전달됩니다.
* **길어지면 나눕니다.** `SKILL.md`에는 핵심 절차만 쓰고, 긴 참고 자료는 별도 파일로 빼서 "필요하면 reference.md를 읽어라"라고 안내합니다.
* **팀과 공유합니다.** 프로젝트의 `.claude/skills/`에 두고 git에 올리면 팀원 모두가 같은 Skill을 씁니다.

## 마치며

이번 글에서는 Skill을 직접 만들어 봤습니다.

* Skill은 `SKILL.md` 하나로 시작할 수 있습니다.
* Claude는 **`description`만 보고** Skill을 쓸지 정합니다. "무엇을 + 언제"를 꼭 적습니다.
* 참고 파일을 함께 넣어서 더 복잡한 절차도 만들 수 있습니다.
* 만든 뒤에는 **불려야 할 때와 불리면 안 될 때**를 둘 다 테스트합니다.

그런데 Skill에는 한계가 하나 있습니다. Skill은 Claude가 **판단해서** 쓰는 것이라, 꼭 지켜야 하는 규칙이라도 Claude가 놓칠 수 있습니다. "ProjectSettings는 절대 건드리지 마" 같은 규칙은 부탁이 아니라 **강제**가 필요합니다. 다음 글에서는 무조건 실행되는 **Hook**과, 일을 따로 떼어 맡기는 **Subagent**를 다뤄보겠습니다.
