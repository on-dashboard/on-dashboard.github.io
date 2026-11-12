# ON DASHBOARD 자료 반영 가이드 (Claude용)

https://on-dashboard.github.io/ 는 이 저장소(`on-dashboard/on-dashboard.github.io`, 브랜치 `main`)로 공개되는 DC Cabinet 납기·PO 대시보드입니다.

## 파일 구조 (2026-11 개편)

| 파일 | 역할 | Claude가 수정? |
|---|---|---|
| `dashboard.html` | **대시보드 본체.** Claude가 만드는 단일 HTML 그대로입니다. | **예 — 이 파일만** |
| `index.html` | 웹 편집 껍데기. `dashboard.html`을 불러와 보여 주고, 편집 버튼·편집자 관리·저장을 담당합니다. | 아니요 |
| `editors.json`, `fonts/` | 편집 권한, 글꼴 | 아니요 |

사용자의 PC에는 자동 업로드가 설치돼 있어, Claude가 만든 대시보드 HTML을 **다운로드하면 그대로 `dashboard.html`로 올라갑니다.** 따라서 평소처럼 완성된 단일 HTML 파일을 만들어 주면 됩니다.

## 꼭 지킬 것

1. **작업을 시작하기 전에 최신본을 기준으로 삼으세요.** 웹에서 다른 사람이 고친 내용이 들어 있을 수 있습니다.
   최신본: https://raw.githubusercontent.com/on-dashboard/on-dashboard.github.io/main/dashboard.html
   (가져올 수 없으면 사용자에게 이 파일을 달라고 하세요. 예전 대화의 버전을 그대로 고치면 웹 수정이 사라집니다.)
2. 결과물은 **완성된 단일 HTML 파일 하나**입니다 (`fonts/` 상대 경로 유지, 외부 라이브러리 없이).
3. 아래 "웹 편집 호환 규칙"을 지켜야 웹에서 계속 고칠 수 있습니다. 규칙을 벗어난 부분은 웹 편집 패널에서 "찾지 못함"으로 빠질 뿐 화면은 정상입니다.
4. GitHub 도구(커넥터, Claude Code)로 직접 올릴 수 있으면 `dashboard.html`에 커밋하고, 커밋 메시지는 `클로드 버전 · <반영 자료> · Claude`로 하세요. `index.html`은 절대 덮어쓰지 마세요.
5. 끝나면 바뀐 내용을 한국어로 짧게 정리해 주세요 (항목별 이전 → 새 값).

## 웹 편집 호환 규칙 (형식만 유지, 내용은 자유)

- 데이터 배열은 `<script>` 안에서 **리터럴 배열**로 둡니다:
  `var ITEMS=[…];` (일정·이슈), `var SH=[…];` (항차), `var INV=[…];` (청구·입금), 생산·품질 탭의 `var PR=[…];` (공정 진척).
  배열 안에서 쓰는 상수는 `var A9_10=["A9","A10"];`처럼 문자열 배열로만 정의하세요.
- ITEMS 항목 형식: `{t:[표시탭…],a:"영역",o:"담당",d:"YYYY-MM-DD 또는 \"\"",x:"내용",s:"open|prog|risk|done",p:[프로젝트…],k:1(핵심일 때)}`
  - t: sch 일정, ms 마일스톤, cost 비용, prod 생산·품질, logi 물류·PO, risk 이슈 / a: sch, ms, cost, q(품질), l(물류), risk / o: "ACE" | "대홍" | "On Energy" / p: A9 A10 C1 W1 CF GE EG
- 상단 배너: `<span class="lead">오늘 쟁점</span>` 바로 다음 `<span>…</span>`에 문구 (굵게는 `<b>`만), 그 뒤 `<span class="dd" id="bnDd">`.
- 기준 문서: `<span class="asof">기준 <span id="today"></span> · …</span>` / 출처: `<span>출처: …</span>`
- KPI: `var kp='<div class="kpis">'` 아래 한 줄에 카드 하나씩
  `+'<div class="kpi"><span class="l">제목</span><span class="v">값<small>단위</small></span><span class="s">설명</span></div>'` (빨간 카드는 `class="kpi k-bad"`). 계산식(`'+ddTxt(…)+'`)이 든 칸은 웹에서 잠깁니다.
- 생산품질일보: `var daily=panel("q","생산품질일보 MM/DD","NN주차",'<div class="stats"><div class="stat"><div class="l">…</div><div class="v">값<small>단위</small></div><div class="s">…</div></div>…</div>'+tbl(["첫 열 제목","일정","내용"],[[…],[…]],{}),5);`
- 검사·서류: `var ins=panel(…,tbl([헤더],[["구분","일정",stDot("good|mid|bad|off","상태"),"비고"],…],{}),…);`
- 공정 진척 표 설명: `var prog=panel("q","항차별 공정 진척","설명",…)`

## 자주 하는 반영

- **새 일보**: 일보 패널(제목·주차·숫자 카드·표), PR 진척 %, 관련 KPI, 완료된 ITEMS의 `s`를 `done`, 배너, `asof`의 일보 날짜.
- **입금 확인**: `INV[].paid`에 날짜, 입금 KPI 카드, 관련 ITEMS를 `done`.
- **일정 변경**: `SH[]`의 날짜와 표시용 문자열(`win`, `fat`), 관련 ITEMS.
- **새 이슈**: ITEMS에 추가 (중요하면 `k:1`, 위험이면 `s:"risk"`).


---

# Outlook 메일 트래킹 (Microsoft 365 커넥터)

사용자(김민구, ACE)가 "메일 트래킹 해 줘", "오늘 메일 정리", "회신 대기 확인" 등을 요청하면 Microsoft 365 커넥터로 Outlook을 검색해 아래 3가지를 합니다. 메일을 **보내거나 삭제·이동하지 마세요.** 회신 독촉 메일은 초안만 만들어 사용자에게 보여 줍니다.

## 대상 메일

- 기간: 요청에 기간이 없으면 **최근 7일** (회신 대기는 최근 30일).
- 프로젝트 관련으로 보는 기준 (하나라도 해당):
  - 상대 도메인/회사: On Energy, 대홍전기, Crusoe, KOMERI, KTR, AkzoNobel, 물류 업체
  - 키워드: DC Cabinet, Abilene, Childress, Warrenton, Coffee, GoldenEye, Eagle, PO-0001, 503-, FAT, Audit, UL, IP55, EPB, MCB, SPD, 항차, 출고, 일보, 인보이스, Invoice, Delivery, Release
- 광고·뉴스레터·시스템 알림은 제외.

## 1) 회신 대기 추적

- **내가 보낸 메일** 중 질문·요청·승인 요청이 있고, 같은 대화(Conversation)에 상대 회신이 없는 것.
- **내가 받은 메일** 중 나에게 질문·요청했는데 내가 아직 답하지 않은 것도 따로 표시.
- 표로 정리: `보낸 날 | 상대(사람·회사) | 제목 | 요청 내용 한 줄 | 경과 영업일 | 권장 조치`
- 경과 5영업일 이상은 **위험**, 3영업일 이상은 **주의**. 위험 건은 정중한 리마인드 초안(영어/한국어는 원 메일 언어)을 같이 제시.

## 2) 프로젝트 메일 요약

- 주제별로 묶기: 일정·출고 / FAT·검사·UL / 품질·자재 / 비용·인보이스·입금 / PO·계약 / 물류
- 각 항목: 날짜, 보낸 사람, 핵심 1~2줄, 결정 사항, 내가 해야 할 일(기한)
- 마지막에 "대시보드 반영 제안" 목록 (아래 3번에 넣을 것)

## 3) 대시보드 반영

- 위 규칙(웹 편집 호환 규칙)대로 최신 `dashboard.html`을 가져와 수정합니다.
- 메일에서 나온 일정·이슈는 `ITEMS`에 추가/갱신합니다:
  - 회신 대기: `x:"[회신 대기] 상대 · 제목 요지 (보낸 날 MM/DD)"`, `a:"risk"`, `t:["risk"]`, `o`는 **회신해야 하는 쪽**(On Energy·대홍·ACE), `d`는 보낸 날 + 5영업일, `s`는 경과 5영업일 이상이면 `"risk"` 아니면 `"open"`, 중요하면 `k:1`.
  - 회신이 오면 해당 항목을 `s:"done"`으로 바꾸고 `x` 끝에 `· MM/DD 회신`을 붙입니다.
  - 일정 변경·입금·검사 결과 등은 기존 항목을 찾아 갱신하고, 없으면 새로 추가합니다.
- 반영 전에 **바꿀 목록(이전 → 새 값)을 먼저 보여 주고** 사용자가 확인하면 반영합니다. (사용자가 "바로 반영"이라고 하면 확인 없이 진행)
- 반영 방법: GitHub 도구가 있으면 `dashboard.html`에 직접 커밋(`메일 반영 · MM/DD · Claude`). 없으면 완성된 HTML 파일로 주면 사용자 PC의 자동 업로드가 올립니다.
