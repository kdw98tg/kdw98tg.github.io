---
layout: post

title: "[Claude Code] 써볼 만한 스킬·플러그인 모음 (토큰 절약, 영상 편집, 보안 검사)"

categories:
  - AX

tags:
  - Claude Code
  - Skill
  - Plugin

toc: true
toc_sticky: true

date: 2026-11-24
last_modified_at: 2026-11-24
---

## 들어가며

유튜브에서 소개된 Claude Code 스킬, 플러그인, 커넥터를 모아 정리했습니다. 실제로 있는지, 설명이 맞는지는 2026년 11월 기준으로 각 저장소 README를 보고 직접 확인했고, 항목마다 바로 밑에 설치 명령을 붙였습니다.

* `/plugin ...` 명령은 **Claude Code 입력창**에, 나머지는 **터미널**에 입력합니다.
* 플러그인은 설치 후 Claude Code를 **다시 시작**해야 적용됩니다.
* `curl ... | bash`처럼 인터넷에서 받은 스크립트를 바로 실행하는 명령은, 실행 전에 스크립트 내용을 한 번 열어보는 것이 안전합니다.

> 남이 만든 스킬은 설치 전에 꼭 내용을 확인하세요. 검사 방법은 [8번 항목](#8-스킬-보안-검사-skillspector)에 정리했습니다.

## 1. 공식 마켓플레이스 인기 플러그인 Top 5

| 이름 | 한 줄 요약 |
|---|---|
| frontend-design | AI 티 안 나는 화면 디자인 |
| superpowers | 계획 → 테스트 → 디버깅 개발 습관 |
| code-review | 여러 에이전트가 코드 리뷰 |
| context7 | 라이브러리 최신 문서 불러오기 |
| skill-creator | 나만의 스킬 만들기 |

앤트로픽 공식 마켓플레이스(`claude-plugins-official`)에 있는 플러그인은 마켓플레이스 등록 없이 바로 설치합니다.

### frontend-design

AI 티 안 나는 화면 디자인을 만들어 줍니다.

```
/plugin install frontend-design@claude-plugins-official
```

### superpowers

계획 → 테스트 → 디버깅 순서로 개발하는 습관을 잡아주는 스킬 묶음입니다.

```
/plugin install superpowers@claude-plugins-official
```

### code-review

여러 에이전트가 코드 변경을 리뷰하고, 확신이 낮은 지적은 걸러줍니다.

```
/plugin install code-review@claude-plugins-official
```

### context7

라이브러리 최신 문서를 불러와서 옛날 방식의 코드를 막아줍니다. 질문이 외부 서버로 가니 민감한 내용은 보내지 마세요.

```
/plugin install context7@claude-plugins-official
```

터미널에서 `npx ctx7 setup`으로 설정할 수도 있습니다.

### skill-creator

나만의 스킬을 만들고 개선하게 도와줍니다.

```
/plugin install skill-creator@claude-plugins-official
```

## 2. 토큰 절약

| 이름 | 한 줄 요약 |
|---|---|
| Code Mode | 도구를 코드로 한 번에 불러 토큰 절약 (실험 기능) |
| graphify | 코드베이스를 지식 그래프로 만들어 탐색 토큰 절약 |
| caveman | Claude의 답변을 짧게 |
| rtk | 명령 출력을 압축 |
| headroom | 컨텍스트 압축 프록시 |
| FreeLLMAPI | 무료 LLM API를 묶어서 연결 (회사 코드 금지) |

### Code Mode

Claude가 JS 코드를 짜서 여러 도구를 한 번에 부릅니다. 불러오는 양이 줄어서 5시간·주간 한도가 오래갑니다.

* Claude Code에 들어 있는 **실험 기능**이라 환경변수로 켭니다.
* 측정 예: 도구 설명이 차지하는 양 24.6k → 11.1k 토큰 (외부 블로그 측정 한 건)

```powershell
# PowerShell
$env:CLAUDE_CODE_EXPERIMENTAL_CODE_MODE=1; claude
```

### graphify

코드베이스를 지식 그래프로 만들어서 탐색에 드는 토큰을 줄입니다.

* "71배 절약"은 제작자가 잰 최대치입니다. 다른 사용자 테스트에서는 5~10배라는 보고가 많고, 오히려 더 든다는 말도 있습니다.
* Claude Code 업데이트 후 Hook이 깨졌다는 보고가 있습니다.

```bash
uv tool install graphifyy
graphify install          # Claude Code 연결
# 이후 Claude Code에서 /graphify .
```

### caveman

Claude의 **답변**을 원시인 말투처럼 짧게 줄여서 출력 토큰을 아낍니다.

```
/plugin marketplace add JuliusBrussee/caveman
/plugin install caveman@caveman
```

### rtk (Rust Token Killer)

git, 테스트 같은 **명령 출력**을 최대 90%까지 압축합니다. caveman과는 별개 도구입니다.

```bash
winget install rtk-ai.rtk # macOS: brew install rtk
rtk init -g               # Claude Code에 Hook 등록
```

### headroom

컨텍스트를 압축해 주는 프록시입니다. 실사용 절감은 보통 50% 정도라는 평가가 많습니다.

```bash
uv tool install --python 3.13 "headroom-ai[all]"
headroom wrap claude      # 되돌리기: headroom unwrap claude
```

### FreeLLMAPI

ChatGPT, Gemini, GLM 등 여러 무료 LLM API를 한 주소로 묶어서 Claude Code에 연결합니다.

* "월 74억 토큰"은 34개 제공처에 전부 가입했을 때의 이론상 최대치입니다.
* 코드가 여러 외부 업체로 나가고, 일부 무료 플랜은 보낸 내용을 학습에 씁니다. **회사 코드에는 쓰지 마세요.**

```bash
curl -fsSL https://freellmapi.co/install.sh | bash
npx freellmapi setup-claude --url http://localhost:3001 --api-key <발급받은 키>
```

## 3. 개발·학습

| 이름 | 한 줄 요약 |
|---|---|
| VibeWise | 되묻는 과외 선생 |
| Matt Pocock 스킬 | grill-me, teach, handoff 등 |
| gstack | Garry Tan의 스킬 묶음 (리뷰, QA, 보안) |
| Everything Claude Code | 스킬·에이전트 수백 개 모음 |
| skill-reviewer | 내가 만든 스킬 리뷰 (공식 plugin-dev) |
| Claude-Code-Game-Studios | AI 에이전트 49명의 게임 스튜디오 |
| CLI-Anything | Blender 같은 프로그램을 CLI로 조작 |

### VibeWise

답부터 알려주지 않고 되묻는 과외 선생입니다. 내 접근법을 먼저 묻고 장단점을 같이 따져봅니다.

```
/plugin install vibe-wise@anthropic-plugin-directory
```

설치 후 `/vibe-wise:learn`으로 시작합니다.

### Matt Pocock 스킬

* `grill-me`: 계획을 질문으로 파고듭니다
* `teach`: 여러 세션에 걸쳐 개념을 가르칩니다
* `handoff`: 대화를 문서로 정리해 다른 에이전트에 넘깁니다
* `wait-what`: 방금 한 말을 쉽게 다시 설명합니다
* `to-questionnaire`: 질문지를 만듭니다

```
/plugin install mattpocock-skills@claude-plugins-official
```

### gstack

Y Combinator 대표 Garry Tan이 만든 스킬 묶음입니다. `office-hours`, `plan-ceo-review`, `review`, `qa`, `cso` 등이 들어 있습니다.

```bash
# Git Bash 또는 macOS/Linux
git clone --single-branch --depth 1 https://github.com/garrytan/gstack.git ~/.claude/skills/gstack
cd ~/.claude/skills/gstack && ./setup
# Windows PowerShell에서는 ./setup 대신 .\setup.ps1
```

### Everything Claude Code (ECC)

스킬과 에이전트 수백 개를 모은 플러그인입니다. v2.0.0 기준 에이전트 64개, 스킬 261개, 명령어·훅 84개이고, 최신 릴리스는 67 / 281 / 94개입니다.

```
/plugin marketplace add https://github.com/affaan-m/ECC
/plugin install ecc@ecc
```

ECC 안에는 프론트엔드 스킬 5종이 들어 있습니다. 따로 설치하는 플러그인이 아닙니다.

| 스킬 | 역할 |
|---|---|
| frontend-design-direction | 코드 짜기 전 목적·타깃·톤을 먼저 정하고, 로딩/에러/빈 화면 처리를 강제, 작업 후 실제 화면을 렌더링해 검증 |
| frontend-patterns | React/Next.js 패턴 (합성 컴포넌트, 커스텀 훅, useMemo/lazy 최적화, 에러 바운더리) |
| frontend-slides | 외부 라이브러리 없이 HTML/CSS/JS로 슬라이드 덱 제작 |
| liquid-glass-design | 애플 리퀴드 글래스 스타일의 반투명·블러 UI |
| frontend-a11y | 키보드 내비게이션, 모달 포커스 트랩, 스크린 리더 대응 등 접근성 |

### skill-reviewer

내가 만든 스킬이 잘 만들어졌는지 리뷰합니다. 앤트로픽 공식 `plugin-dev` 플러그인에 들어 있는 에이전트입니다. 같은 이름의 개인 제작 스킬도 많으니 주의하세요.

```
/plugin install plugin-dev@claude-plugins-official
```

### Claude-Code-Game-Studios

게임 하나를 맡기면 AI 에이전트 49명(48명이라는 곳도 있음)이 기획, 프로그래밍, 아트, QA 등을 나눠 맡습니다.

```bash
# 새 폴더에 템플릿으로 받기
git clone https://github.com/Donchitos/Claude-Code-Game-Studios.git my-game
cd my-game
claude
# Claude Code에서 /start
```

> 유니티를 쓴다면 유니티가 직접 만든 공식 플러그인도 있습니다: `/plugin install unity@claude-plugins-official`

### CLI-Anything

AI가 Blender 같은 프로그램을 CLI로 직접 사용하게 해줍니다.

```
/plugin marketplace add HKUDS/CLI-Anything
/plugin install cli-anything
```

## 4. 디자인·시각화

| 이름 | 한 줄 요약 |
|---|---|
| taste | 디자인 감각을 수치로 관리 |
| anatomy | 시스템 원리를 등축 SVG 도해로 |
| answer-me-with-html | 답변을 한 장짜리 HTML로 |
| logo-design-skill | 로고·브랜드 키트 제작 |
| The Kitchen | 움직이는 UI 코드 사이트 (찾지 못함) |

### taste

디자인 감각을 수치로 관리하는 프론트엔드 스킬입니다.

```
/plugin marketplace add Leonxlnx/taste-skill
/plugin install taste-skill@taste-skill
```

### anatomy

기계나 시스템 원리를 등축(아이소메트릭) SVG 도해로 그립니다. 불꽃이나 물 같은 효과는 WebGL로 덧그리고, 3D 모드는 아직 베타입니다.

```bash
npx skills add wheresryan22/anatomy -g
```

### answer-me-with-html

텍스트 답변 대신 다이어그램과 표가 들어간 한 장짜리 오프라인 HTML로 답합니다. Node.js 20 이상이 필요합니다.

```
/plugin marketplace add QingYunA/answer-me-with-html
/plugin install answer-me-with-html@answer-me-with-html
```

### logo-design-skill

로고 레퍼런스 1,400개 이상과 SVG 점검 도구로 로고와 브랜드 키트를 만듭니다.

```
/plugin marketplace add kaankiziltug/logo-design-skill
/plugin install logo-design@logo-design-skill
```

### The Kitchen

실제 물건이 움직이는 듯한 UI 코드를 제공하는 사이트라고 소개됐는데, 검색으로는 **찾지 못했습니다.** 이름을 다시 확인해 봐야 합니다.

## 5. 영상 편집

| 이름 | 한 줄 요약 |
|---|---|
| video-use | 촬영본을 대본 보고 알아서 편집 |
| Ghost Editor | 말하는 영상을 릴스로, 자막·모션 그래픽 |
| FFmpeg 스킬 | 자르기, 자막, 세로 변환 같은 기본 편집 |
| 에디토리얼 모션 그래픽스 | 그림·도표로 만드는 설명 영상 (서비스형) |
| claude-shorts | 긴 영상에서 숏폼 구간 뽑기 |
| OpenMontage | 대본부터 보이스오버까지 영상 제작 자동화 |

### video-use

찍은 영상을 폴더에 넣고 "런칭 영상으로 편집해줘"라고 하면, 받아쓴 대본을 보고 알아서 편집합니다. Browser Use 팀이 만들었고, ffmpeg와 받아쓰기용 ElevenLabs API 키가 필요합니다.

```bash
git clone https://github.com/browser-use/video-use ~/Developer/video-use
cd ~/Developer/video-use && uv sync
ln -sfn ~/Developer/video-use ~/.claude/skills/video-use
cp .env.example .env      # .env에 ELEVENLABS_API_KEY 입력
```

### Ghost Editor

말하는 촬영본을 릴스로 다듬고 자막과 모션 그래픽을 붙입니다. 참고 영상의 편집 스타일도 따라 합니다. 아직 작은 프로젝트라 설치 전에 코드를 확인하세요. 설치하면 ffmpeg, Node, Python, Whisper 등 약 4GB가 깔립니다.

```bash
git clone https://github.com/kurbaitaev/ghost-editor ~/.claude/skills/ghost-editor
# 이후 docs/INSTALL.md 안내를 따름
```

### FFmpeg 스킬

영상 자르기, 자막 넣기, 세로 화면 변환 같은 기본 편집을 맡길 수 있습니다. 같은 이름의 스킬이 여러 개라 설치 명령은 특정하지 않았습니다.

### 에디토리얼 모션 그래픽스

얼굴 없이 그림, 도표, 전환 효과로 만드는 설명 영상용입니다. Higgsfield 사이트에서 연결하는 서비스형이라 설치 명령이 없습니다.

### claude-shorts

긴 영상에서 숏폼으로 쓸 구간을 골라 자막과 함께 뽑아줍니다. Windows에서는 WSL 2가 필요합니다.

```bash
git clone https://github.com/AgriciDaniel/claude-shorts.git
cd claude-shorts && bash install.sh
# Claude Code에서 /shorts 영상경로
```

### OpenMontage

대본 → 장면 → 보이스오버까지 이어지는 영상 제작 자동화 시스템입니다.

```bash
git clone https://github.com/calesthio/OpenMontage.git
# 이후 README의 설치 단계(Python 가상환경, npm install)를 따름
```

## 6. 사업·기타

| 이름 | 한 줄 요약 |
|---|---|
| founder-skill | 창업 전 사업성 검증 |
| replica-skill | 기존 앱 역설계 후 재구축 |
| day-trader-skill | 트레이딩 자동화 (주식 추천 아님) |
| Agent-Reach | YouTube, X, Reddit 등을 직접 검색·읽기 |

founder-skill, replica-skill, day-trader-skill은 모두 같은 제작자(Jakeschincariol)가 만들었고, Python 3.8 이상이 필요합니다.

### founder-skill

창업 전에 사업성을 검증하는 스킬 11개입니다. 가상 이사회, 가상 소비자 패널, 손익 계산 등이 들어 있습니다.

```
/plugin marketplace add Jakeschincariol/founder-skill
/plugin install founder-skill@founder-skill
```

### replica-skill

기존 앱을 역설계해서 다시 만들고, 버그를 테스트하고 고치는 스킬 11개입니다.

```
/plugin marketplace add Jakeschincariol/replica-skill
/plugin install replica-skill@replica-skill
```

### day-trader-skill

차트 스캔부터 진입, 청산, 리스크 점검, 주문까지 하는 트레이딩 자동화 스킬 11개입니다. **주식 추천 스킬이 아닙니다.**

* 미국 주식 전용이고, 증권사는 Alpaca와 Interactive Brokers만 연결됩니다.
* 기본은 모의투자입니다. 실제 계좌를 연결할 때는 특히 주의하세요.

```
/plugin marketplace add Jakeschincariol/day-trader-skill
/plugin install day-trader-skill@day-trader-skill
```

### Agent-Reach

에이전트가 YouTube 자막, X, Reddit 등을 직접 검색하고 읽게 해주는 도구입니다. 상품 하나를 주면 유튜브 리뷰를 다 보고 장단점을 표로 뽑아오는 식으로 쓸 수 있습니다.

Claude Code에 아래처럼 부탁하면 설치 문서를 읽고 설치합니다. PyPI의 `agent-reach` 패키지는 다른 프로젝트이니 받지 마세요.

```
Agent Reach 설치해줘: https://raw.githubusercontent.com/Panniantong/Agent-Reach/main/docs/install.md
```

## 7. 커넥터

| 이름 | 한 줄 요약 |
|---|---|
| Perplexity | 웹 조사 + 출처 |
| Firecrawl | 웹사이트 내용 읽어오기 |
| PostHog | 내 사이트 이용 데이터 분석 |
| Playwright | 브라우저 직접 조작해서 동작 확인 |

### Perplexity

웹 자료를 조사하고 출처까지 알려줍니다. API 키가 필요합니다.

```bash
claude mcp add perplexity --env PERPLEXITY_API_KEY="발급받은_키" -- npx -y @perplexity-ai/mcp-server
```

### Firecrawl

웹사이트 내용을 직접 읽어옵니다. 경쟁사 페이지의 상품과 문구를 비교할 때 씁니다.

```
/plugin install firecrawl@claude-plugins-official
```

### PostHog

내 사이트에 쌓인 이용 데이터를 분석해서, 사람들이 어느 단계에서 나가는지 확인합니다.

```
/plugin install posthog@claude-plugins-official
```

### Playwright

브라우저를 직접 움직이면서 버튼과 입력창이 잘 동작하는지 확인합니다.

```
/plugin install playwright@claude-plugins-official
```

## 8. 스킬 보안 검사: SkillSpector

* 스킬 3만여 개를 조사한 연구에서 약 26%에 보안 구멍이 나왔습니다.
* 그래서 NVIDIA가 설치 전에 돌리는 무료 검사기 **SkillSpector**를 공개했습니다.
* 점수를 매겨서 안전한지 알려줍니다.

```bash
uv tool install --python 3.13 'skillspector[mcp] @ git+https://github.com/NVIDIA/skillspector.git'
skillspector scan ./검사할-스킬-폴더/

# Claude Code에서 쓰려면 MCP로 연결
claude mcp add skillspector -- skillspector mcp
```

MCP로 연결했다면 Claude Code에 이렇게 부탁하면 됩니다.

```
이 스킬 주소를 SkillSpector 기본 검사로 돌려서 한국어로 설명해줘
```

| 점수 | 판단 |
|---|---|
| 20점 이하 | 설치 |
| 20~50점 | 내용을 직접 확인하고 결정 |
| 50점 초과 | 설치 안 함 |

* 점수 기준은 영상에서 말한 것입니다. 점수 방향과 기준은 공식 README에서 확인하세요.
* 검사를 통과해도 안전이 보장되지는 않습니다. NVIDIA도 그렇게 안내합니다.
* 영어가 아닌 글은 놓칠 수 있습니다. **한국어 스킬은 받기 전에 눈으로 직접 확인**하세요.

## 9. 세팅 정리 프롬프트

세팅에 너무 많은 걸 넣으면 Claude가 지시를 무시할 수 있습니다. 스킬을 깔 때 한 번씩 정리해 주세요.

**설정 점검**

```
지금 내 클로드 세팅을 점검해줘. CLAUDE.md, 설치된 스킬, 플러그인, MCP 목록을 전부 읽고 다음 세 가지를 표로 뽑아줘: 최근 한 달 동안 한 번도 안 쓴 것, 모델이 이미 혼자 잘하는 일을 시키는 규칙, 서로 겹치거나 부딪히는 지시. 그리고 지울 것과 남길 것을 이유와 함께 제안하고, 내가 승인하면 지워. 그다음 내 일에 맞게 폴더 구조를 다시 짜고, 어디에 뭐가 있는지 CLAUDE.md 맨 위에 다섯 줄로 적어줘.
```

**반복 작업 템플릿화**

```
내가 반복해서 하는 일을 찾아줘. 최근 대화와 파일을 보고 일주일에 두 번 이상 하는 작업을 목록으로 뽑아. 작업마다 templates 폴더에 파일을 하나씩 만들고, 각 파일에 순서, 결과물 형식, 잘 된 예시 하나를 넣어줘. 앞으로 그 작업을 시키면 템플릿부터 읽고 시작하라고 CLAUDE.md에 한 줄 추가해.
```

**나를 인터뷰하기**

```
나를 인터뷰해줘. 한 번에 질문 하나씩, 열다섯 개까지. 결과물을 어떤 길이와 말투와 형식으로 받고 싶은지, 일할 때 싫어하는 것, 알아서 결정해도 되는 것과 꼭 물어봐야 하는 것을 알아내. 인터뷰가 끝나면 답을 열 줄 안으로 요약해서 CLAUDE.md의 '일하는 방식' 섹션에 저장해.
```

## 10. 기타

* **D1AL-ogue**: Steam 무료 게임입니다. 다이얼을 돌려 안드로이드를 수리하는 퍼즐 게임으로, Claude와는 관계없습니다.
