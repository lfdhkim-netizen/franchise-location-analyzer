# Franchise Location Analyzer

외식업 프랜차이즈 본사용 Claude 스킬. **프리미엄 국밥(단가 15,000원, 타겟 직장인)** 브랜드에 맞춰 튜닝된 신규 점포 입지 분석 자동화.

후보 점포의 주소·상권·임대조건을 텍스트로 입력하면 한 번의 호출로 6단계 분석 + 8인 멀티 페르소나 평가를 수행하고, **Word 보고서(.docx)** 한 묶음을 생성한다.

---

## 무엇을 하나요

| 단계 | 분석 영역 | 담당 페르소나 |
|---|---|---|
| 1 | 상권 분석 (유동인구·동선·접근성) | 상권 분석가 |
| 2 | 타겟 적합도 (직장인 × 15,000원 단가 수용성) | 직장인 고객 + 마케팅 전문가 |
| 3 | 경쟁 분석 (반경 500m 한식·국밥점) | F&B 운영 전문가 |
| 4 | 재무 시뮬레이션 (3시나리오·BEP·회수기간) | 재무 분석가(CFO 출신) |
| 5 | 리스크 분석 (임대·계약·환기·식자재 동선) | 부동산 감정평가사 + 가맹점주 |
| 6 | 8인 페르소나 종합 평가 + GO/HOLD/NO-GO | 본부 의사결정자(개발본부장) |

---

## 8인 페르소나

1. **상권 분석가** — 도시계획 박사, 15년차
2. **부동산 감정평가사** — F&B 임대차 전문
3. **F&B 운영 전문가** — 15년차 외식 컨설턴트
4. **재무 분석가** — CFO 출신, 외식 IPO 자문
5. **마케팅 전문가** — SNS·배달앱 운영
6. **가맹점주** — 다점포 운영, 현장형
7. **직장인 고객 페르소나** — 30대 사무직, 점심 1.2–1.5만원 허용
8. **본부 의사결정자** — 개발본부장, 최종 GO/NO-GO 권한

---

## 설치 및 사용

### 옵션 A. 단일 프로젝트에 설치
```bash
git clone https://github.com/lfdhkim-netizen/franchise-location-analyzer.git
cd <your-working-project>
cp -r <cloned-repo>/.claude/skills/franchise-location-analyzer .claude/skills/
```

### 옵션 B. 전역(user-level) 설치
```bash
cp -r <cloned-repo>/.claude/skills/franchise-location-analyzer ~/.claude/skills/
```

Claude Code를 재시작하면 스킬이 자동 인식된다.

### 옵션 C. Claude.ai 일반 채팅(웹/데스크탑) 업로드

이 스킬은 [`dist/franchise-location-analyzer.zip`](dist/franchise-location-analyzer.zip)에 claude.ai 업로드 형식(ZIP)으로 미리 패키징되어 있다.

1. `dist/franchise-location-analyzer.zip` 다운로드
2. claude.ai 접속 → 오른쪽 위 프로필 → **Settings → Customize → Skills**
3. **"+ Create skill" → "Upload a skill"** 선택 → ZIP 업로드
4. (사전 요건) **Settings → Features → Code Execution 활성화** 필요
5. 업로드 후 일반 채팅에서 자연어로 호출되면 자동 실행됨
   - 예: "강남 테헤란로 ○○○ 신규점 검토해줘…"

**플랜 요건**: Free / Pro / Max / Team / Enterprise 모두 지원 (단, Code Execution 활성화 필수).
**호출 방식**: claude.ai의 Skills는 슬래시 호출이 아닌 **자연어 매칭(description 기반)** 으로 자동 트리거된다.

### 호출 방법
다음과 같이 자연어로 입력하면 스킬이 트리거된다.

> "강남구 테헤란로 ○○○ 신규점 검토해줘. CBD 오피스, 보증금 1.5억 / 월세 1,200만원 / 권리금 1.5억, 35평 36석, 역삼역 도보 4분."

산출물: 6단계 분석 + 8인 페르소나 평가 매트릭스 + Word 보고서(.docx).

---

## 입력 항목

**필수**
- 후보 점포 주소
- 상권 특성 (CBD / 오피스 외곽 / 혼합 / 주거 / 학원가 / 역세권 등)
- 임대조건: 보증금 / 월세 / 권리금
- 점포 면적 (평수, 좌석수)

**선택 (있으면 정밀도↑)**
- 반경 300m·500m·1km 추정 직장인 인구
- 인근 경쟁점 정보 (상호/단가/평점)
- 임대 계약기간·갱신조건
- 지하철역 도보거리

---

## 산출물

**Word 보고서(.docx)**, 10–15페이지 분량
- 표지
- Executive Summary (1페이지, GO/HOLD/NO-GO 한 줄 결론)
- 6장 본문
- 페르소나 평가 매트릭스
- 최종 추천 및 액션 아이템

생성은 `anthropic-skills:docx` 스킬을 통해 수행한다.

---

## 판정 규칙

| 판정 | 기준 |
|---|---|
| 🟢 GO | 5개 영역 평균 ≥ 75점 + 리스크 점수 ≤ 30점 |
| 🟡 HOLD | 조건부 — 정량적 충족 조건 명시 필요 (예: "권리금 -20% 협상") |
| 🔴 NO-GO | 결정적 리스크 2–3개 명시 |

---

## 디자인 근거

`docs/design-rationale.md` 참조.

- 왜 8인 페르소나인가
- 왜 6단계인가
- 직장인 + 15,000원 단가 특화 분석 항목
- 프리미엄 국밥 특유의 리스크 (시즌·환기·식자재 동선)

---

## 라이선스

내부 사용 (본부·가맹점). 외부 배포 시 본부 허가 필요.
