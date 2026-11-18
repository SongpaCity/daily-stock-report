# 📈 데일리 증시 리포트 제작 가이드

이 문서는 **매일 증시 리포트 HTML을 만들어 GitHub Pages에 올리는 전체 절차**를 정리한 것이다.
"오늘자 리포트 만들어줘" 한마디로 이 워크플로우 전체가 실행된다.

---

## 0. 한눈에 보는 흐름

```
① 오늘 날짜 확인 (date)
② 데이터 수집 (서브에이전트 2개 병렬: 증시 / 부동산·주요뉴스)
③ HTML 작성 (DailyReport/YYYY-MM-DD.html, 고정 템플릿 + 데이터 주입)
④ 검증 (node로 JSON 유효성 + renderReport 함수 문법)
⑤ 커밋·푸시 (SongpaCity 계정으로 전환 → push → 원래 계정 복구)
⑥ 라이브 반영 확인 (HTTP 200 + 드롭다운 자동 등록)
```

소요: 데이터 수집 2~3분 + 작성/배포 2~3분. Pages 빌드 반영은 푸시 후 약 1분.

---

## 1. 사이트 구조

- **저장소:** `SongpaCity/daily-stock-report` (public) · 로컬 `~/daily-stock-report`
- **배포:** GitHub Pages → https://songpacity.github.io/daily-stock-report/
- **핵심 파일**
  | 파일 | 역할 |
  |---|---|
  | `index.html` | 메인 껍데기. 날짜 **드롭다운** + iframe. **최신 날짜 자동 표시.** 건드릴 일 거의 없음 |
  | `DailyReport/YYYY-MM-DD.html` | **매일 추가하는 리포트 본체.** 파일명에 날짜만 있으면 드롭다운에 자동 등장 |
  | `realestate.html` | 부동산 실거래(전용 59㎡·2년) 하위 페이지 + 상승률 순위. 메인 헤더에서 링크 |
  | `README.md` | 공개용 사용법 |
  | `REPORT-GUIDE.md` | (이 문서) 제작 절차 |

- **드롭다운 동작:** `index.html`의 JS가 GitHub Contents API로 `DailyReport/` 폴더를 읽어 **파일명에서 `YYYY-MM-DD`를 추출** → 날짜 내림차순 정렬 → 최신을 기본 표시. **목록 파일을 따로 관리할 필요 없음.** 파일만 올리면 끝.
- **특정 날짜 공유:** `.../daily-stock-report/#2026-11-06` 처럼 `#날짜` 해시로 바로 열림.

---

## 2. 리포트 프레임 규칙 (가장 중요)

리포트는 항상 **"어제 마감 → 오늘 현시점 → 오늘 밤 미국장"** 하루 기준이다. (다일 구간 ❌)

- 탭은 2개: **`오늘 (M/D)`** (일간) / **`최근 흐름 (구간)`** (맥락용 주간).
- **주말·휴장 처리**
  - 월요일 작성 → "지난 금요일 마감 → 오늘(월) → 오늘 밤 미국장". 간밤 미국은 금요일 세션이 마지막.
  - 미국은 한국보다 하루 늦게 열리므로 "간밤 미국장"은 **직전 미 거래일**을 가리킴. 요일·세션을 반드시 교차 확인.
- 장중에 쓰면 오늘 수치는 **전부 '장중값'**임을 명시. 종가 아님.
- **요일을 매번 확인**한다 (서브에이전트가 2025/2026 혼입이나 요일 착오를 일으킬 수 있음).

---

## 3. 데이터 수집 (서브에이전트 2개 병렬)

`general-purpose` 에이전트 2개를 **한 메시지에서 동시** 호출한다. 각 에이전트는 WebSearch/WebFetch로 교차 확인.

### 3-1. 증시 스냅샷 에이전트 — 수집 항목
1. **코스피 (어제) 최종 종가** — 등락률·pt
2. **코스피 (오늘) 현재값** + **코스닥** (어제 종가 / 오늘 현재)
3. **삼성전자·SK하이닉스** (어제 종가 / 오늘 현재가·등락률)
4. **간밤 미국장** 다우/S&P500/나스닥 종가·등락률
5. **미국 주요 종목** 등락률 (엔비디아·AMD·마이크론·브로드컴·테슬라·메타·애플·아마존·알파벳 등 + 그날의 핵심 종목)
6. **거시** 미 10년물·WTI·필라델피아 반도체지수(SOX)
7. **외국인/기관/개인** 수급 (어제 마감 / 오늘 장중) + **원·달러**
8. **그날의 핵심 이벤트** (FOMC·빅테크 실적·고용보고서·선거 등)
9. **오늘 밤 미국장 관전 포인트** (지수선물 방향 + 예정 실적/지표)
10. **오늘 한국 증시 주요 뉴스** 1~3건 (날짜·언론사·요약)

지시: "추측 금지, 못 찾으면 '미확인', 수치마다 출처(언론사/날짜) 명시, 2025년 혼입 주의, 요일 정확히."

### 3-2. 부동산·주요뉴스 에이전트 — 수집 항목
1. **부동산** — 오늘/어제자 **새** 정책·대출규제·통계·발언이 있는가? 없으면 "오늘자 새 부동산 소식 없음" 명시.
2. **기타 주요 경제/정책 뉴스** (환율·금리·CPI·수출입·대외 이벤트).
3. 마지막에 **"리포트 삽입용 한 줄"** 요청.

> 뉴스 박스 제목은 범용 **"📰 주요 뉴스"**. 부동산은 **새 소식이 있을 때만** 반영하고, 없으면 억지로 넣지 말 것(동향 뉴스만 있으면 그 수준으로).

---

## 4. HTML 템플릿 구조

템플릿은 **고정**이다. 매번 **최근 날짜 리포트 파일을 복제하듯** 같은 구조로 쓰고, `renderReport({...})`의 **데이터 객체**와 하드코딩 텍스트(lead·news·sub·verdict·flow·scenario)만 그날에 맞게 교체한다.

- D3 v7 기반. `lineChart`·`hbar`·`spark` 렌더 함수는 **절대 수정하지 말 것** (그대로 복붙).
- 다크/라이트 자동, 모바일 반응형, 상승=빨강·하락=파랑(국내 관행).
- 구성: `kicker → h1 → lead → 탭 → [일간: 타일·뉴스박스·주가차트·이슈타임라인·미국장·판단] / [주간: 지수·반도체·미국종목·플로우·시나리오·일정]`.

### 데이터 객체 스키마 (`renderReport({...})`)

```js
{
  indices: [ {date:"YYYY-MM-DD", market:"KOSPI"|"NASDAQ"|"SPX"|"KRW", close:"숫자문자열", note:"메모"} ],
  kr_daily: [ {date, name:"삼성전자"|"SK하이닉스", close:"원", note} ],   // 일별 종가(차트용)
  stocks:  [ {name, a:"어제%", b:"오늘장중%"(없으면 ""), note} ],          // 표: 어제 종가 vs 오늘 장중
  us_top:  [ {name, ticker, group, dXX:"등락률"(없으면 "")} ],            // 미국 종목 막대 (전전일/간밤 2일치 필드)
  macro:   [ {key, label, value, unit, note} ],                          // 플로우/타일에서 mv('key')로 참조
  events:  [ {date, event, why} ],                                       // 주간 '체크 일정' 표
  issues:  [ {date, time, category, title, detail, affected, impact:"-3~+3"} ]  // 일간 이슈 타임라인
}
```

#### 필드 작성 규칙
- `indices`: KOSPI는 최근 6거래일 정도(일간 차트 slice(-5) + 주간용). NASDAQ/SPX는 4~5개, KRW는 3개 정도. 오늘(장중) 포인트는 마지막에 넣고 **차트에서 `dashedLast:true`로 점선** 처리.
- `kr_daily`: 삼성전자·SK하이닉스 각각 6개 전후. 가격은 원 단위 정수 문자열.
- `stocks` 표의 `a`/`b`: 어제 종가 등락 / 오늘 장중 등락(%). 못 구하면 `""` → 표에 `—`.
- `us_top`의 `dXX`: **날짜별 필드명**(예: `d5`=11/5, `d6`=간밤). 일간 막대 = 간밤 하루, 주간 막대 = 전전일/간밤 2개. 미확인은 `""`.
- `macro.key`: `ust10y`, `wti`, `sox`, `krw`, `cpi`, `base_rate`, `kr_foreign`, `export` 등 그날 쓰는 것만. 플로우/타일 본문에서 `mv('key')`로 숫자 삽입.
- `issues.impact`: **코스피 기준 주관 점수 -3(강한악재)~+3(강한호재)**. 막대가 가운데선 기준 좌(악재)/우(호재)로 그려짐.
- 차트 `dashedLast`: 오늘이 **장중(미확정)**이면 해당 시계열 true, 어제가 **확정 종가**면 false.

### 교체해야 하는 하드코딩 텍스트
- `kicker` (작성일·기준), `h1` (날짜), `lead` (핵심 요약 문단)
- `d-news` (📰 주요 뉴스 한 줄)
- 각 `h2/sub` 설명문, 탭 라벨, 캡션
- `d-verdict` (오늘 한 줄 / 악재 체크 / 호재 체크)
- `w-flow` (원인 구조 플로우), `w-scen` (3 시나리오)

---

## 5. 검증 (커밋 전 필수)

`node`로 (1) 임베드 JSON 유효성, (2) `renderReport` 함수 문법을 확인한다. (D3는 브라우저 전용이라 데이터·문법까지만 검증 가능)

```bash
cd ~/daily-stock-report && node -e '
const fs=require("fs");
const html=fs.readFileSync("DailyReport/YYYY-MM-DD.html","utf8");
const m=html.match(/renderReport\((\{[\s\S]*\})\);<\/script>/);
if(!m){console.log("❌ 호출 못찾음");process.exit(1);}
try{const D=JSON.parse(m[1]); console.log("✅ JSON 유효 · KOSPI:",D.indices.filter(r=>r.market==="KOSPI").map(r=>r.date).join(", ")); }
catch(e){console.log("❌ JSON:",e.message);process.exit(1);}
const s=html.indexOf("function renderReport(D)"),e=html.indexOf("</script>",s);
try{ new Function(html.slice(s,e)+"\nreturn renderReport;"); console.log("✅ 함수 문법 정상"); }
catch(x){ console.log("❌ 문법:",x.message); process.exit(1);}
'
```

> JSON 안에 한글/따옴표/`\n` 섞을 때 파싱 깨지기 쉬움. 이 검증에서 반드시 ✅ 확인 후 커밋.

---

## 6. 커밋·푸시 (계정 전환 주의)

저장소 주인은 **SongpaCity**인데, 평소 gh 활성 계정은 **TeamSpartaJaewook**(업무용)이다. **푸시만 SongpaCity로 전환 → 끝나면 반드시 복구.**

```bash
cd ~/daily-stock-report && \
git add -A && \
git -c user.name="SongpaCity" -c user.email="songpacity@users.noreply.github.com" commit -q -m "report: YYYY-MM-DD" && \
gh auth switch --user SongpaCity >/dev/null 2>&1 && \
git push -q && echo "PUSH OK" && \
gh auth switch --user TeamSpartaJaewook >/dev/null 2>&1 && \
echo "restored: $(gh api user --jq .login)"
```

- 커밋 메시지: `report: YYYY-MM-DD` (특이 사건 있으면 `report: YYYY-MM-DD 검은 금요일` 식으로 꼬리말).
- **계정 복구 확인 필수** (`restored: TeamSpartaJaewook` 출력).

---

## 7. 라이브 반영 확인

푸시 후 Pages 빌드에 약 1분. HTTP 200 + 드롭다운(폴더 API)에 새 날짜가 떴는지 확인.

```bash
# 서빙 확인
for i in $(seq 1 8); do
  code=$(curl -s -o /dev/null -w "%{http_code}" https://songpacity.github.io/daily-stock-report/DailyReport/YYYY-MM-DD.html)
  echo "try $i: HTTP $code"; [ "$code" = "200" ] && break; sleep 12
done
# 드롭다운 목록(폴더 자동 집계)
curl -s "https://api.github.com/repos/SongpaCity/daily-stock-report/contents/DailyReport?ref=main" | grep -o '"name": *"[^"]*"' | tail -3
```

---

## 8. 주의사항 · 함정 (경험칙)

- **장중 vs 종가:** 작성 시점이 장중이면 오늘 값은 전부 장중. **다음 날 그 종가가 내 장중값과 크게 다를 수 있음**(예: 장 초반 +2% → 종가 -1%). 다음 날 리포트에서 전일 확정 종가로 바로잡아 반영.
- **"지금 기준으로 갱신":** 같은 날 다시 요청 시 → 새 스냅샷만 받아 **기존 파일을 Edit로 수술**(kicker 시각·lead·타일·stocks·issues·indices/kr_daily의 오늘 포인트). 전면 재작성 아님.
- **미확인은 미확인으로.** 추정·창작 금지. 못 구한 값은 `""`/—, 본문엔 "미확인"으로 표기하고 caveat에 적시.
- **데이터 신뢰도:** 서브에이전트가 2025년 자료 혼입·요일 착오·수치 오기를 낼 수 있음. 핵심 수치는 교차 확인하고, 이례적 값(예: 환율 1,476원, 영업이익 83조)은 의심하고 caveat 처리.
- **지수 간 디커플링:** 미 지수 급등인데 코스피만 급락 같은 날이 있음(예: 블루웨이브 AI 규제). 지수만 보지 말고 **반도체/업종·수급**을 봐야 서사가 맞음.
- **FOMC 국면:** 2026년은 **동결 vs 인상** 사이클(인하 아님, 현 3.75~4.00%). 한국 기준금리(3.00%)와 혼동 금지.
- **gh 계정 복구** 빠뜨리면 다음 업무 계정 작업이 꼬임. 매번 확인.
- **렌더 함수 불변:** `lineChart`/`hbar`/CSS는 그대로 복제. 데이터·텍스트만 교체.

---

## 9. 부동산 하위 페이지 (`realestate.html`)

- 국토부 아파트매매 실거래가 API로 **전용 59㎡(58~62㎡), 최근 2년 8분기 중위가**, 구별 대장 2단지를 집계해 카드+차트로 렌더.
- 상단에 **상승률 순위**(카드 데이터를 JS로 자동 집계, TOP20/전체 토글). 카드가 바뀌면 순위도 자동 갱신.
- 도시 추가: `지역코드(LAWD_CD)`로 24개월 조회 → 58~62㎡ 필터 → (아파트,동) 그룹 중위가 상위 2개(최소 3건) → 분기 중위가/건수 → 카드 HTML + `mkChart("cN", ...)` 생성 → 경기 그리드·데이터 스크립트에 삽입.
- **함정:** API 레이트리밋(429) → 재시도·백오프 + 동시성 3. 지역코드는 `region-codes.md`와 실제 API가 다를 수 있어 **동(洞) 이름으로 교차 확인**(예: 화성시는 구 개편으로 41591/41593/41595/41597 4개 코드).

---

## 10. 체크리스트 (복붙용)

```
[ ] date로 오늘 요일·날짜 확인
[ ] 서브에이전트 2개 병렬 (증시 / 부동산·뉴스)
[ ] 프레임 결정 (평일: 어제→오늘→오늘밤 / 월요일: 지난금→오늘→오늘밤)
[ ] DailyReport/YYYY-MM-DD.html 작성 (템플릿 고정, 데이터+텍스트만 교체)
[ ] node 검증 (JSON ✅ / 함수 ✅)
[ ] 커밋 → SongpaCity 전환 → push → TeamSpartaJaewook 복구
[ ] HTTP 200 + 드롭다운 새 날짜 확인
[ ] 사용자에게: 변경사항/핵심요약/공유링크/장중값 주의 보고
```
