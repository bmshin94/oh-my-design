# oh-my-design 분석 & 수익화 검토 (2026-09-16)

> 이 문서는 "GitHub에서 받은 이 레포가 뭔지"를 폴더 실물 기준으로 분석하고,
> 로컬 에이전트 구축 관점의 활용법과 수익화 아이디어까지 정리한 세션 기록이다.
> 채팅에만 있던 맥락을 파일로 고정하기 위해 작성됐다.

## 링크

| 대상 | 주소 |
|---|---|
| 이 저장소 (작업 중인 포크) | https://github.com/bmshin94/oh-my-design |
| 업스트림 원본 | https://github.com/kwakseongjae/oh-my-design |
| npm 패키지 | https://www.npmjs.com/package/oh-my-design-cli |
| 공식 문서 | https://oh-my-design.kr/docs/en |
| CLI 안내 | https://oh-my-design.kr/cli |
| 업스트림 이슈 | https://github.com/kwakseongjae/oh-my-design/issues |

- `package.json`의 `repository`는 `kwakseongjae/oh-my-design`, `origin` 리모트는 `bmshin94/oh-my-design` → **포크(또는 클론) 상태**.
- 라이선스: **MIT** (`LICENSE`, Copyright (c) 2026 oh-my-design)
- 패키지 버전: `oh-my-design-cli@2.0.1`

---

## 1. 한 줄 요약

**AI 코딩 에이전트에게 "디자인 감각"을 파일로 심어주는 도구.**

AI에게 UI를 맡기면 매번 비슷한 결과(이 프로젝트는 이를 **"AI slop"**이라 부른다)가 나오는 문제를,
`DESIGN.md`라는 **프로젝트 소유의 계약 파일** 하나를 깔고 이후 모든 UI 작업이 그 파일을 근거로만
움직이게 만들어 해결한다.

핵심 주장: **화면은 첫 단계가 아니라 마지막 단계다.**

```
철학 → 결정표 → 토큰 → 컴포넌트 계약 → 레이아웃 문법 → 빌드 → 렌더 비평 → DESIGN.md
```

각 단계가 다음 단계를 구속하고, 모든 토큰은 자신을 만든 결정 ID(`D-<원칙>-<번호>`)를 역참조한다.
근거 없는 토큰 값은 취향 차이가 아니라 **게이트 실패**로 처리된다(`GS7`, `GS8`).

---

## 2. 폴더 지도 (실물 확인 기준)

| 경로 | 내용 |
|---|---|
| `skills/` | 스킬 **29개** — `omd-init`, `omd-apply`, `omd-autopilot`, `omd-harness`, `omd-landing`, `omd-feel`, `omd-slop-audit` 등 |
| `.claude/agents/` | 전문 서브에이전트 **20개** — UX 리서처, a11y 감사관, 마이크로카피, 페르소나 테스터, 비평가 등 |
| `.claude/hooks/` | 훅 **4개** (`skill-activation`, `session-state-loader`, `post-edit-watch`, `session-end-foldin`) |
| `web/references/<id>/DESIGN.md` | **기업 레퍼런스 440개** — 카탈로그의 단일 진실 소스 |
| `design-md/`, `packages/mcp/data/references/` | 위에서 **파생된 미러**. 편집 금지, `web/references/`만 수정 |
| `spec/` | `design-md-core-v2.md` (벤더 중립 7섹션 계약) + JSON 스키마 |
| `src/cli/`, `bin/` | TypeScript CLI 본체 (`omd`) |
| `test-v2/tools/` | **결정론적 검사기** — `render-integrity.mjs`, `landing-integrity.mjs`(82KB), `text-contrast.mjs`, `showcase.mjs` |
| `benchmarks/ui-resolve-bench/` | 효과 측정 벤치마크 + 하네스가 생성한 완성 시스템 3건 |
| `web/` | Next.js 사이트(oh-my-design.kr) — `/builder`, `/reference/[id]`, `/presets`, `/benchmarks`, 블로그 |
| `docs/` | 메인테이너 R&D 기록(아프로디테 실험 등). 일반 사용자에겐 노이즈 |
| `data/` | `reference-quality.json`, `reference-demand.json`, `reference-fingerprints.json` 등 |

빌드/검증: `npm run build`(tsup) · `npm test`(vitest) · `npm run lint`(tsc --noEmit)

---

## 3. 설치 및 사용법

### 설치

```bash
# UI 작업할 프로젝트 폴더에서
npx oh-my-design-cli@latest
```

설치 시 두 가지를 묻는다.

- **범위**: `Project`(기본) / `Global`
- **채널**: `claude-code` / `codex` / `opencode` / `cursor` (2.4+)

설치 후 **에이전트 재시작**(Claude Code는 Cmd+Q 후 재실행) → 진단:

```bash
npx oh-my-design-cli@latest doctor
```

### 사용

CLI는 **설치기·진단기·안내기**일 뿐이며 **UI를 직접 생성하지 않는다.**
실제 작업은 에이전트 채팅에서 자연어로 한다.

| 말하면 | 트리거되는 스킬 |
|---|---|
| "디자인 시스템 세팅해줘" | `omd:init` |
| "이 버튼 좀 더 따뜻하게" | `omd:apply` |
| "이 가격 페이지 검수해줘" | `omd:slop-audit` |
| "앞으로 그림자 쓰지 마" | `omd:remember` → 이후 `omd:learn`으로 DESIGN.md 반영 |

무엇을 물어볼지 모를 때 라우팅:

```bash
npx oh-my-design-cli@latest workflows "이 가격 페이지를 검수하고 고쳐줘" --lang ko
```

시스템 열람:

```bash
npx oh-my-design-cli@latest book            # 로컬 포트에 디자인 시스템 문서 사이트
npx oh-my-design-cli@latest book --static   # 핸드오프용 단일 HTML
```

> **주의**: 이 저장소 자체는 "제품의 소스코드"이지 설치 결과물이 아니다.
> 실제 사용은 **다른 프로젝트 폴더**에서 `npx`로 설치해 쓴다.

---

## 4. 플러그인? 스킬? MCP? → **스킬 번들**

### 결론: 스킬 + 서브에이전트 + 훅 (MCP 아님)

설치 시 프로젝트에 **파일이 복사되는** 구조다.

| 구성 | 위치 | 개수 |
|---|---|---|
| 스킬 (`SKILL.md` 마크다운) | `.claude/skills/` | 29 |
| 서브에이전트 | `.claude/agents/` | 20 |
| 훅 (Node 스크립트) | `.claude/hooks/` | 4 |
| 레퍼런스 카탈로그 | 로컬 | 440 |

### MCP는 폐기됨

`packages/mcp/package.json`:

> "Archived catalog MCP transport. Retained for history; **no longer built, published, or required** by oh-my-design skills."

루트 `.mcp.json`에 있는 것은 **Playwright**(브라우저 자동화)이며, 이 저장소 **개발용**이지 사용자용이 아니다.

### 훅 4종 (`.claude/settings.json`)

| 시점 | 스크립트 | 역할 |
|---|---|---|
| `UserPromptSubmit` | `skill-activation.cjs` | 어떤 스킬을 켤지 판단 |
| `SessionStart` | `session-state-loader.cjs` + `scripts/context_restore.sh` | 이전 상태 복원 |
| `PostToolUse` (Edit/Write) | `post-edit-watch.cjs` | 수정 시 디자인 규칙 감시 |
| `Stop` | `session-end-foldin.cjs` | 세션 종료 시 기억 정리 |

---

## 5. API 토큰 필요 여부

### 핵심 워크플로 = **키 0개**

DESIGN.md 생성/적용/검수, 레퍼런스 440개 열람 전부 **API 키 없이** 동작한다.
이미 쓰고 있는 코딩 에이전트가 실행 주체이고, 이 도구는 지시서를 건넬 뿐이다.
별도 API 키·데몬·MCP 서버가 필요 없다.

### 선택 기능 = 이미지/영상 생성 시에만

`scripts/omd-setup-detect.mjs` 기준:

| 채널 | 환경변수 | 비용 | 특징 |
|---|---|---|---|
| Recraft | `RECRAFT_API_KEY` | SVG $0.088/장 | 유일한 네이티브 SVG, style_id 재사용 |
| xAI (Grok) | `XAI_API_KEY` | $0.04/장 | CLI 없이 REST |
| OpenAI | `OPENAI_API_KEY` | ≈$0.03–0.08/장 | gpt-image-2 |
| Gemini / Veo | `GEMINI_API_KEY` | 영상 $0.05–0.60/초 | 영상 |

```bash
npx oh-my-design-cli@latest setup detect   # 읽기 전용, 키 값은 출력하지 않음
```

키가 없으면 스톡 이미지로 대체하지 않고 **프롬프트 팩 + 수동 큐**로 넘긴다.

---

## 6. 왜 GitHub에서 주목받는가 (분석)

> 별(star) 수치는 이 세션에서 확인하지 못했다. 업스트림 저장소는 세션의 GitHub 접근 범위 밖이다.
> 아래는 수치 근거가 아니라 **구조 관찰에 기반한 해석**이다.

1. **통증이 보편적** — AI로 UI를 만들어 본 사람 대부분이 겪는 문제.
2. **"AI slop"이라는 네이밍** — 적을 규정하고 이름을 붙여 공유 가능한 밈이 됨.
3. **진입장벽이 낮음** — `npx` 한 줄, API 키 0개, 가입 없음.
4. **데이터 해자** — 440개 레퍼런스가 단순 색상 복사가 아니라 출처·방법·날짜를 동반한 관측 기록.
5. **결과물이 파일** — 블랙박스 SaaS가 아니라 레포 안의 마크다운이라 검증 가능.
6. **멀티 채널 + 4개 국어** — Claude Code/Codex/OpenCode/Cursor, README 한·영·일·중(번체).
7. **자기 주장을 벤치마크로 검증하려는 태도** — `benchmarks/`로 효과를 측정.

레퍼런스 증거 구조 예시(`web/references/toss/DESIGN.md`, 356줄):

```yaml
"tokens.colors.primary":
  surface_id: tds-button
  url: "https://tossmini-docs.toss.im/tds-mobile/components/button/"
  method: computed-style-and-official-doc
  captured: "2026-07-11"
```

---

## 7. 로컬 에이전트 구축에 주는 가치

**도구로 쓰는 것보다 레퍼런스 구현으로 읽는 가치가 크다.** 훔쳐올 패턴:

1. **스킬 트리거 설계** — `description`에 4개 국어 트리거 문구를 넣어 발동 조건을 명시.
2. **역할 분리** — 20개 서브에이전트. 특히 `omd-critic`은 쓰기 권한을 비평문 하나로 제한("the constraint is intentional") → **권한 축소로 역할을 강제**.
3. **훅 라이프사이클 4종** — 에이전트가 잊지 않게 만드는 실전 구현.
4. **결정론적 검사기** — AI 결과물을 AI로 검사하지 않고 **코드로** 검사. 같은 입력 = 같은 출력, API 비용 0.
5. **세션 연속성 프로토콜** — `docs/CURRENT_STATE.md`(단일 복원 지점) + `docs/JOURNAL.md`(일지).
6. **실패 기록의 공개** — `docs/APHRODITE_HANDOVER.md`:
   > 검사기 40종을 **전부 통과한** 산출물이 사람 채점에서 30점·10점.
   > → **"검사기는 품질 모델이 아니다."**

6번은 에이전트 설계자에게 특히 값진 교훈이다. 기계 게이트는 **하한선**을 지킬 뿐 **상한선**을 만들지 못한다.

---

## 8. 수익화 검토

### 8.0 레포에서 발견한 "돈 될 신호" 2개

#### 신호 ① 레퍼런스에 유통기한이 있다 (`data/reference-quality.json`)

| 등급 | 개수 |
|---|---|
| `verified_v2` | 140 |
| `partial` | 160 |
| `legacy_snapshot` | 140 |
| **합계** | **440** |

각 항목에 `verified_at`과 **`next_reverify_at`**이 있다(토스: 2026-07-14 검증 → 2026-10-11 재검증 예정, 분기 주기).

- 기업 디자인은 계속 바뀌므로 이 데이터는 **반드시 썩는다**.
- 썩는 것을 막는 **반복 노동** = **구독료의 명분**. SaaS의 교과서적 조건.
- 미검증 300개(partial+legacy) = **채울 여백**이자 차별화 지점.

#### 신호 ② 수요 실측 데이터가 있다 (`data/reference-demand.json`)

`reference_select_count` 기준 상위(2026-07-10까지, GA4/Upstash 합성 스냅샷):

| 순위 | 브랜드 | 선택 수 |
|---|---|---|
| 1 | **토스** | 4,378 |
| 2 | 애플 | 1,774 |
| 3 | **당근** | 1,687 |
| 4 | **배민** | 1,287 |
| 5 | **카카오** | 1,140 |
| 6 | **KRDS(공공)** | 612 |
| 7 | 네이버 | 458 |

- 상위 20개 중 **15개가 한국 브랜드**, 1위 토스가 2위 애플의 약 2.5배.
- 해석: 실제 사용자층은 한국 개발자이며, **한국 시장에서 PMF 신호**가 보인다.
- 단, 이는 **업스트림 사이트의 집계 스냅샷**이며 우리 데이터가 아니다. 방향 지표로만 사용.

### 8.1 전제 (법/윤리 체크)

- MIT라 상업적 이용·수정·재배포는 **합법**(저작권 고지 유지 필요).
- 그러나 **원본이 무료 공개**이므로 "같은 것을 유료로 되팔기"는 사업이 되지 않는다.
  → 전략은 **"원본이 하지 않는 레이어를 얹어 판다"**: 호스팅·자동화·팀 협업·데이터 신선도·사람 손.
- 레퍼런스는 **타사 브랜드 관측 데이터**다.
  - 괜찮음: 색/간격/타이포 **수치**, 공식 문서 **URL 출처 표기**
  - 위험: **로고·폰트 파일·상표 자산**을 유료 상품에 포함, "공식 제휴"로 오해할 표현
- 커뮤니티 리스크 관리: 업스트림 **크레딧 + 링크 명시**, 개선은 **업스트림 기여**, 유료는 **서비스 레이어**에 배치.

### 8.2 아이디어 6종

#### A. PR 디자인 QA 봇 — 1순위 추천

- **무엇**: PR 올리면 봇이 디자인 일관성을 검사해 인라인 코멘트.
- **근거**: 검사기가 이미 CLI로 노출돼 있음.
  ```bash
  omd check render   <html>   # 오버플로·뷰포트 이탈·텍스트 클립·폰트 폴백·깨진 이미지
  omd check landing  <html>   # 랜딩 규칙 LI-1…LI-23
  omd check contrast <html>   # 사진/그라데이션 위 텍스트 대비 실측, 포커스 링, no-JS 폴백
  ```
  결정론적이라 **결과가 일정하고 AI 비용이 0** → 마진이 높고 신뢰가 쌓인다.
- **대상**: 디자이너 없는 스타트업, 외주 품질 편차가 고민인 팀, 디자인 시스템을 아무도 안 지키는 팀.
- **가격**: Free ₩0 / Team ₩39,000월 / Business ₩150,000월
- **난이도**: 낮음 (MVP 3–4주) — GitHub App + 프리뷰 URL + 검사기 + 코멘트 API
- **리스크**: 프리뷰 배포가 없는 팀은 검사 대상이 없음(Vercel/Netlify 연동 필수). 잔소리가 많으면 즉시 꺼진다 → **기본은 BLOCK만 노출**.

#### B. 레퍼런스 신선도 구독 — 방어력 최고

- **무엇**: `next_reverify_at`을 상품화. "참고한 토스 디자인이 3개월 전 기준입니다" 알림 + 갱신 제공.
- **왜**: 데이터는 반드시 썩고, 막는 것은 반복 노동이며, 반복 노동은 복제가 어렵다(진짜 해자).
- **부가 상품**: 브랜드 진화 리포트(`web/src/app/api/reference-evolution` 라우트 기반), 워치 알림(Slack), 업계 트렌드 리포트.
- **가격**: 개인 ₩9,900월 / 팀 ₩49,000월 / 연간 리포트 단건 ₩99,000~
- **난이도**: 중간 — 기술보다 **운영 지속성**이 관건.
- **리스크**: 자동화 없으면 인력이 갈린다 → 크롤링 + 자동 diff → 사람은 확인만.

#### C. 팀용 디자인 시스템 허브 (`omd book` 호스팅)

- **무엇**: 로컬 전용인 `omd book`을 팀 공용 URL로. 토큰 + 결정 근거 + 상태 매트릭스 + 실측 대비 + 변경 이력.
- **차별점**: Storybook/Zeroheight에 없는 **"왜 이 값인가"가 값 옆에 붙어 있음**.
- **가격**: ₩8,000/인/월 (최소 5인), 5인 + 공개 문서는 무료
- **난이도**: 중간 (6–8주) — `src/cli/book.ts` 렌더링 재사용 가능
- **리스크**: 기존 강자와 정면 충돌, 팀 도구라 도입 사이클이 길다.

#### D. 에이전시/컨설팅 — 제품 0개, 당장 매출

- **무엇**: "브랜드 → AI가 지킬 수 있는 디자인 시스템, 2주 완성". 납품물 = DESIGN.md + 에이전트 설치 + CI 연동 + 2시간 교육 + 1개월 A/S.
- **왜**: **오늘 시작 가능**, 고객과 직접 대화하며 진짜 통증을 학습 → 이후 제품의 설계도가 됨.
- **가격**: 스타트업 패키지 ₩300만/2주 · 에이전시 리테이너 ₩150만/월
- **난이도**: 가장 낮음
- **리스크**: 시간을 파는 구조라 확장이 안 됨 → 반복 부분을 제품으로 추출하는 것이 목표.

#### E. 한국 특화 레퍼런스 팩

- **근거**: 위 수요 데이터. 상위는 채워져 있고 **롱테일이 비어 있다**.
  - 커머스: 무신사, 컬리, 오늘의집, 지그재그
  - 금융: 케이뱅크, 토스증권, 삼성증권
  - 여행: 야놀자, 마이리얼트립, 트리플
  - B2B SaaS: 채널톡, 스티비, 플렉스, 두들린
  - 공공: KRDS 파생(수요 612인데 공급 부족)
- **가격**: 업종 팩 ₩49,000–99,000 · 공공/기업 커스텀 ₩300만~
- **난이도**: 낮음–중간. 단, "언제 어느 URL에서 어떤 방법으로 확인"을 지키지 않으면 가치가 0.
- **리스크**: 브랜드 자산 재배포 금지 원칙 엄수.

#### F. 스크롤 데모 영상 자동화 (유입용 미끼)

- **근거**: CLI에 이미 존재.
  ```bash
  omd showcase page.html --compare --labels "기존|개선" --gif
  # --seconds --fps --dpr --width --height --hold --label
  ```
- **왜**: 마케터·인디해커가 매번 고생하는 작업(Product Hunt, 트위터, 포트폴리오).
- **포지션**: 메인 수익 아님. **무료 공개 → 트래픽/이메일 확보 → A·B·C로 전환**.
- **난이도**: 낮음–중간 (서버 브라우저 비용 관리만)

### 8.3 실행 순서 (돈 먼저, 제품 나중)

```
0–1개월   D 에이전시 서비스        → 매출 + 시장 학습
1–2개월   F showcase 무료 공개     → 트래픽 유입 + 이메일 수집
2–4개월   A PR QA 봇 MVP           → D에서 만난 고객을 첫 유료 전환
4–8개월   B 신선도 데이터 축적      → 방어력 구축 (일찍 시작할수록 유리)
6–12개월  C 팀 허브                → A 고객 업셀
```

### 8.4 가격표 요약

| 상품 | 가격 | 반복 매출 | 난이도 |
|---|---|---|---|
| D 에이전시 세팅 | ₩300만/건 | ✕ | 낮음 |
| D 리테이너 | ₩150만/월 | ○ | 낮음 |
| F 데모 영상 Pro | ₩9,900/월 | ○ | 낮음–중 |
| A QA 봇 Team | ₩39,000/월 | ○ | 낮음 |
| A QA 봇 Business | ₩150,000/월 | ○ | 낮음 |
| B 신선도 구독 | ₩9,900–49,000/월 | ○ | 중간 |
| C 팀 허브 | ₩8,000/인/월 | ○ | 중간 |
| E 레퍼런스 팩 | ₩49,000–99,000 | ✕ | 낮음–중 |

### 8.5 리스크 체크리스트

- [ ] 브랜드 로고/폰트 파일을 유료 상품에 포함하지 않는다
- [ ] 업스트림 크레딧 + 링크 명시
- [ ] MIT 저작권 고지 유지
- [ ] "공식 제휴/인증" 오해 표현 금지
- [ ] 개선사항은 업스트림에 기여
- [ ] 고객 코드/데이터를 학습에 쓰지 않음을 약관에 명시
- [ ] 검사 결과를 자동 차단으로 강제하지 않음

### 8.6 검증 먼저 (만들기 전 1주)

| 요일 | 할 일 |
|---|---|
| 월–화 | `omd check contrast` / `omd showcase` 실행 → 결과 캡처 확보(영업 자료) |
| 수–목 | 캡처로 랜딩 1장 → 커뮤니티 공유(GeekNews/디스콰이엇 등) |
| 금 | 아는 스타트업 5곳에 "디자인 일관성 문제 되세요?" DM |

**GO 기준**: 이메일 20개 확보 또는 5곳 중 3곳이 통증을 확인.

---

## 9. React / PHP로 만들 수 있나

| 덩어리 | React/Node | PHP | 비고 |
|---|---|---|---|
| ① DESIGN.md 규격 | ○ | ○ | YAML 없는 순수 마크다운 7섹션. 언어 무관 |
| ② 카탈로그 + 뷰어 | ◎ | ○ | `web/`이 이미 Next.js+React+Tailwind. PHP도 파싱만 하면 됨 |
| ③ 검사기 | ○ | △ | 렌더링 픽셀 측정이라 **헤드리스 브라우저(Playwright) 필요** |
| ④ 에이전트 연동 | ✕ | ✕ | 각 CLI가 정한 규격(`.claude/skills`, `.cursor/skills`)을 따라야 함 |

- ③은 PHP 단독 불가 → **Playwright를 외부 프로세스로 호출**하거나 **검사 전용 Node 마이크로서비스**를 분리.
- ④는 언어 선택 문제가 아니라 규격 문제. 다만 그 **파일을 생성하는 도구**는 어떤 언어로 만들어도 된다.

권장 구조:

```
[PHP 또는 Next.js]   ← 웹 대시보드 / 카탈로그 / 팀 협업
        ↕
[DESIGN.md]          ← 마크다운, 언어 무관 (핵심 자산)
        ↕
[Node 검사기]        ← Playwright 필요한 부분만 분리
        ↕
[마크다운 스킬 파일]  ← 각 에이전트 규격에 맞춰 생성
```

결론: **React면 단일 언어로 깔끔하고, PHP여도 문제없다.** 검사기만 Node로 떼어내면 된다.

---

## 10. 열린 질문 / 다음 단계

- 어떤 수익화 트랙을 먼저 잡을지 결정 필요 (A: 만들기 / B: 오래 가기 / D: 당장 매출)
- 선택 시 다음 산출물: 상세 설계도(예: A → GitHub App 구조, 검사기 연동, PR 코멘트 포맷)
- 업스트림 별 수치·npm 다운로드 등 **정량 트랙션은 미확인** → 별도 확인 필요
