# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 이 폴더는 무엇인가
AI 네이티브 커리어캠프 2기 6일차(D6) 수업 폴더입니다. 빌드·린트·테스트 도구와 git 저장소가 없습니다. 모든 결과물은 **파일 하나짜리 HTML**이고, 더블클릭(`file://`)으로 열어서 눈으로 확인합니다.

- `D06_1021.html`: 수업 워크북("무료 API로 나만의 서비스 만들기"). 110KB짜리 읽기용 자료라서 과제 작업 중에는 건드리지 않습니다.
- `d6-api/`: 과제 화면들. 모두 키 없는 공개 API를 브라우저에서 `fetch`로 부릅니다.
  - `weather.html`: Open-Meteo 날씨 (단일 카드)
  - `rate.html`: Frankfurter 환율 계산기 (금액 + USD/JPY/EUR/CNY → KRW)
  - `city.html`: 도시 이름 → 지오코딩 → 좌표로 기온과 미세먼지
  - `morning.html`: 카드 4장 대시보드 (날씨·미세먼지, 환율, Nager.Date 공휴일 D-day, Dog CEO 사진)
- `.claude/rules/api.md`: API 화면 규칙. **새 화면을 만들거나 고치기 전에 읽습니다.** 맨 아래 `## 실수에서 배운 규칙`에 AI 실수가 생길 때마다 한 줄씩 추가하는 칸이 있습니다.
- `.claude/skills/api-check/SKILL.md`: `/api-check @d6-api\파일.html`로 규칙을 검사만 하는 스킬(고치지 않음).

## 작업 방식
- 실행·검증은 브라우저에서 파일을 직접 엽니다. 이 환경에서는 화면을 열어 볼 수 없으므로, API 응답 모양과 CORS(`Origin: null` 헤더로 `curl -D -`)만 `curl`로 확인하고 "브라우저에서 확인하지 못했다"고 알립니다.
- 커밋·푸시는 하지 않고 커밋 메시지만 제안합니다. 형식은 `"[직군] D6: <과제 이름>"`이고 뒤에 Co-Authored-By 줄을 붙입니다.
- 화면을 새로 만든 뒤에는 `/api-check`로 검사해 보라고 안내합니다. 이 스킬은 `api.md`와 같은 규칙으로 검사하므로, 규칙을 바꾸면 스킬 결과도 달라집니다.

## 화면 파일 공통 구조
`weather.html`을 제외한 세 화면(`rate`, `city`, `morning`)은 같은 골격이라, 새 화면은 이 중 하나를 복사해서 시작하는 것이 빠릅니다.

- 맨 위 `const API = { ... }`에 주소를 모읍니다. `morning.html`은 쿼리 문자열까지 통째로 넣어 두어서(좌표·통화·금액 포함), 주소 한 줄만 틀리게 바꾸면 그 카드만 실패하는지 시험할 수 있습니다. `rate.html`과 `city.html`은 기본 주소만 `API`에 두고 쿼리는 코드에서 붙입니다.
- `fetch`는 파일에서 한 곳(`// 🔌 API 호출`)뿐입니다. 카드마다 이 헬퍼(`getJson`, `getRate`, `load`)를 따로 호출하므로 카드 하나가 실패해도 다른 카드는 영향이 없습니다.
- `showUsed(id, 라벨, url)`이 화면 맨 아래 "🔌 이 화면이 쓰는 API" 상자에 실제로 부른 주소와 원본 JSON 링크를 갱신합니다. 호출 지점마다 이 함수를 같이 불러야 상자가 실제 호출과 맞습니다.
- 입력으로 다시 부르는 화면(`rate`, `city`)은 `seq` 카운터로 오래된 응답이 새 응답을 덮어쓰지 않게 합니다.
- 응답 값을 `innerHTML`에 넣을 때는 `esc()`로 이스케이프합니다.

## 도메인 지식 (코드만 봐서는 안 보임)
- **미세먼지 등급은 한국 환경부 기준(㎍/㎥)**: PM10은 좋음 ≤30, 보통 ≤80, 나쁨 ≤150, 그 위 매우나쁨. PM2.5는 좋음 ≤15, 보통 ≤35, 나쁨 ≤75, 그 위 매우나쁨. `LIMITS`/`grade()`가 `city.html`과 `morning.html`에 중복되어 있으니 기준을 바꿀 때 둘 다 고칩니다. Open-Meteo 값은 모델 추정값이라 에어코리아 측정값과 다르다고 화면에 밝힙니다.
- **환율은 "실시간"이 아님**: Frankfurter는 유럽중앙은행이 하루 한 번 발표하는 기준 환율입니다. 화면에 기준 날짜(`data.date`)를 보여 주고 "실시간"이라는 말은 쓰지 않습니다.
- **API 모양**: 지오코딩은 `results[0].latitude/longitude`(결과가 없으면 `results` 자체가 없음). Frankfurter는 `rates.KRW`와 `date`, 주소는 `https://api.frankfurter.dev/v1/latest?base=…&symbols=KRW&amount=…`. Nager.Date `NextPublicHolidays/KR`은 날짜 순 배열이고 첫 항목이 가장 가까운 공휴일입니다. Dog CEO는 `{message: 이미지URL, status: "success"}`.
- **키가 필요한 API는 만들지 않습니다.** 주소에 `key=`/`token=`/`apikey=`가 필요하거나 문서에 Authentication이 있으면 "이 API는 키가 필요해서 D8 방식이 필요해요"라고 알려 줍니다.
