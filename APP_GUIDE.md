# Junho Fitness Tracker — 앱 기능 가이드

> 최종 업데이트: 2026-05-14  
> 단일 파일 React 18 PWA (`index.html`) + Supabase 백엔드

---

## 아키텍처 개요

| 항목 | 내용 |
|---|---|
| 프론트엔드 | 단일 HTML 파일 (`index.html`) — CDN Babel Standalone으로 JSX 트랜스파일 |
| 리액트 버전 | React 18 (CDN, no bundler) |
| 백엔드 | Supabase REST API (`junho_fitness` 테이블) |
| 배포 | Netlify (`netlify deploy --prod`) |
| 오프라인 | PWA 미지원 (단순 웹앱) |

### Supabase 테이블 스키마

```
junho_fitness (
  date_key   TEXT,       -- 날짜키 ('2026-05-14') 또는 'global'
  store_key  TEXT,       -- 데이터 종류 키
  value      JSONB,      -- JSON 데이터
  updated_at TIMESTAMPTZ
  PRIMARY KEY (date_key, store_key)
)
```

### 주요 store_key 목록

| store_key | date_key | 내용 |
|---|---|---|
| `checks` | 날짜 | 식단·영양제·타임라인 체크 상태 |
| `workout_v2` | 날짜 | 운동별 세트 배열 (신규 포맷) |
| `weights` | `global` | 운동별 마지막 사용 무게 |

---

## KST 타임존 처리

한국 시간(UTC+9) 기준으로 날짜 키를 생성한다. UTC를 그대로 쓰면 오후 9시 이후 날짜가 바뀌는 버그 발생.

```js
function kstDateKey() {
  return new Date(Date.now() + 9*60*60*1000).toISOString().slice(0,10);
}
const DATE_KEY = kstDateKey();
const TODAY_KST = new Date(Date.now() + 9*60*60*1000);
const TODAY_DAY = DAYS[TODAY_KST.getDay()]; // '월', '화', ...
```

앱 마운트 시 `visibilitychange` 이벤트로 날짜 변경 감지 → 자동 리로드.

---

## 운동 탭 — 세트별 기록 (`workout_v2`)

### 데이터 포맷

```json
{
  "w3": [
    { "weight": 20, "reps": 15, "isWarmup": true,  "done": true  },
    { "weight": 80, "reps": 8,  "isWarmup": false, "done": true  },
    { "weight": 80, "reps": 8,  "isWarmup": false, "done": false }
  ],
  "h2": [...]
}
```

- 키: 운동 ID (`w3`, `h2` 등 — WORKOUT 상수의 `id` 필드)
- 각 세트: `weight`(kg), `reps`(회), `isWarmup`(웜업 여부), `done`(완료 여부)

### 세트 UI 구조

```
[GIF 썸네일]  운동 이름
볼륨 X,XXX kg
──────────────────────────
세트#  [무게]  [횟수]  [완료]  [···]
+ 세트 추가
```

- `···` 메뉴: 웜업 지정 토글 / 세트 삭제
- 웜업 세트: 회색 표시, 볼륨 계산에서 제외
- 세트 추가: 마지막 세트의 무게/횟수 복사하여 추가

### 볼륨 계산

```js
// 운동별 볼륨
exerciseVolume = sum(weight × reps) for sets where done=true AND isWarmup=false

// 일일 총 볼륨
dailyVolume = sum of all exerciseVolume
```

볼륨은 앰버 색상으로 운동 카드 헤더에 표시. 일일 총 볼륨은 상단 헤더에 표시.

### Supabase 저장

세트 데이터 변경 시 500ms 디바운스 후 자동 저장:
```js
dbSet(DATE_KEY, 'workout_v2', workoutSets)
```

무게 변경 시 `global/weights`에도 저장 (다음 날 초기값으로 활용).

---

## 운동 GIF 미리보기

### 데이터 소스

ExRx.net의 운동 GIF를 각 운동에 매핑. WORKOUT 상수의 `gif` 필드에 URL 저장.

### 핫링크 차단 우회

ExRx.net은 Referer 헤더로 핫링크를 차단한다. `referrerPolicy="no-referrer"` 속성으로 Referer 헤더를 전송하지 않아 차단 우회:

```jsx
<img src={ex.gif} referrerPolicy="no-referrer"
  onError={()=>setGifErr(true)} loading="lazy"/>
```

### GIF 표시 방식

- 카드 헤더: 60×60 썸네일 (로드 실패 시 자동 숨김)
- 탭하면: 200×200 모달 오버레이로 확대

### 운동별 GIF URL 매핑 (주요 운동)

| 운동명 | GIF 경로 |
|---|---|
| 바벨 백스쿼트 | `…/Quadriceps/BBSquat.gif` |
| 컨벤셔널 데드리프트 | `…/ErectorSpinae/BBDeadlift.gif` |
| 바벨 벤치프레스 | `…/PectoralSternal/BBBenchPress.gif` |
| 바벨 OHP | `…/DeltoidAnterior/BBMilitaryPress.gif` |
| 웨이티드 풀업 | `…/LatissimusDorsi/WtPullup.gif` |
| 루마니안 데드리프트 | `…/HamstringGrace/BBRomanianDeadlift.gif` |
| 레그 프레스 | `…/Quadriceps/SMLegPress45.gif` |
| 불가리안 스플릿 스쿼트 | `…/Quadriceps/DBBulgarianSplitSquat.gif` |
| 와이드 그립 랫풀다운 | `…/LatissimusDorsi/CBWideGripLAPulldown.gif` |
| 원암 덤벨 로우 | `…/LatissimusDorsi/DBBentOverRow.gif` |
| 덤벨 사이드 레터럴 레이즈 | `…/DeltoidLateral/DBLateralRaise.gif` |
| 바벨 컬 (EZ바) | `…/BicepsBrachii/EZBarbellCurl.gif` |
| 케이블 로프 푸시다운 | `…/TricepsBrachii/CBPushdownV.gif` |

(`…` = `https://exrx.net/AnimatedEx`)

---

## 날짜 탭 네비게이션

상단 일~토 7일 탭으로 요일을 전환. 앱 시작 시 오늘 요일로 초기화.

```js
const [wDay, setWDay] = useState(TODAY_DAY); // '일'~'토'
```

### 탭 스타일 (Bold Minimal — 필 스타일)

- 선택된 요일: 앰버(`#EF9F27`) 배경, 검은 텍스트, bold
- 오늘이면서 미선택: 흰 텍스트 + 하단 앰버 점
- 나머지: `#1a1a1a` 배경, `var(--text3)` 색

### 서브탭 (언더라인 스타일)

운동 / 식단 / 영양제 3개 탭. 선택된 탭은 앰버 2px 하단 보더.

---

## 일요일 휴식일 처리

```js
const isRestDay = wDay === '일';
```

- **운동 탭**: 🧘 아이콘 + "휴식일" 텍스트 표시, 운동 목록 없음
- **식단 탭**: `MEALS_REST` (일요일용 식단) 적용
- **영양제 탭**: 평소와 동일

### 일요일 식단 (`MEALS_REST`)

PDF v10 기준 일요일 식단 그대로:

| 끼니 | 시간 | 내용 |
|---|---|---|
| 아침 | 07:00 | 오트밀 + 우유, 바나나 + 땅콩버터, 단백이 1개 |
| 점심 (외식) | 12:30 | 외식 자유 (kcal null — 칼로리 표시 없음) |
| 간식 | 16:00 | 그릭요거트 150g + 블루베리 |
| 저녁 | 19:00 | 밀프랩 + 김치 |
| 야식 | 취침 전 | 그릭요거트 + 블루베리 |

`kcal: null` 끼니는 칼로리/매크로 표시를 건너뜀 (`meal.kcal != null` 가드).

---

## 식단 탭 — 매크로 목표

```js
const macroTarget = { kcal: 2350, carbs: 300, protein: 120, fat: 75 };
```

- **탄수화물**: 300g (운동일·휴식일 동일)
- **순단백**: 120g
- **지방**: 75g
- **칼로리**: 2,350 kcal

상단 프로그레스 바에 탄·단·지 달성률 표시. 끼니 체크 시 실시간 업데이트.

### 운동일 식단 (`MEALS_WORKOUT`) — PDF v10 원본

| 끼니 | 시간 | kcal | 탄 | 단 | 지 |
|---|---|---|---|---|---|
| 아침 | 07:00 | 530 | 70g | 25g | 16g |
| 점심 | 12:30 | 590 | 78g | 35g | 15g |
| 오후 간식 | 16:30 | 210 | 42g | 10g | 0g |
| 저녁 (운동 전) | 19:30 | 515 | 65g | 30g | 15g |
| 운동 후 | 23:30 | 150 | 5g | 30g | 1g |

---

## 캘린더/기록 탭

### 달력 뷰

- 이전달·다음달 `< >` 이동
- 7열 그리드 (일~토)
- 오늘: 앰버 테두리 강조
- 데이터 있는 날: 틸 점(운동) / 회색 점(체크)

### 과거 기록 드릴다운

날짜 탭하면 패널 열림:
- 운동: 각 운동별 세트 목록, 볼륨 (읽기 전용)
- 식단: 체크 상태 (요일 기준으로 MEALS_WORKOUT/MEALS_REST 자동 분기)
- 영양제: 체크 상태
- 타임라인: 체크 상태

과거 데이터는 편집 불가 (읽기 전용 뷰).

---

## 디자인 시스템 — Bold Minimal

### 컬러 팔레트

```css
:root {
  --bg:    #0f0f0f;  /* 배경 (순수 블랙) */
  --bg2:   #141414;  /* 카드 배경 */
  --bg3:   #1e1e1e;  /* 입력 필드 */
  --bg4:   #272727;  /* 서브 엘리먼트 */
  --text:  #ffffff;  /* 본문 텍스트 */
  --text2: #888888;  /* 보조 텍스트 */
  --text3: #555555;  /* 비활성 텍스트 */
  --border:  rgba(255,255,255,0.08);
  --border2: rgba(255,255,255,0.14);
}

/* 포인트 컬러 */
const A = '#EF9F27'; /* 앰버 — 주 포인트 */
const T = '#1D9E75'; /* 틸  — 완료/성공 */
```

### 타이포그래피

- 폰트: `-apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif` (시스템 폰트)
- 헤더 날짜: `font-size:22px, font-weight:800`
- 브랜드명: `font-size:11px, letter-spacing:3px, font-weight:800, text-transform:uppercase`
- 입력 필드: `font-size:16px, font-weight:700` (iOS 줌 방지)

### 레이아웃 원칙

- 최대 너비: `480px` 중앙 정렬
- 카드: `border-radius:12px`, `background:var(--bg2)`
- 간격: 카드 간 `12px`, 패딩 `16px`
- 강제 다크 모드: `@media` 쿼리 없이 루트에 고정

---

## 영양제 탭

PDF v10 영양제 프로토콜 그대로 반영. 시간대별 그룹화:

| 시간 | 영양제 |
|---|---|
| 기상 직후 | 멀티비타민, 오메가-3, 비타민D3+K2, 아연, 아슈와간다 |
| 운동 전 | 크레아틴 5g, 베타알라닌 3g, 카페인 200mg |
| 운동 후 | 크레아틴 5g, 단백이 셰이크 |
| 취침 전 | 마그네슘 |

각 항목 개별 체크 → Supabase `checks` 저장.

---

## 타임라인 탭

하루 루틴 체크리스트:

- 07:00 기상 + 아침 루틴
- 07:30 아침 식사
- 09:00 ~ 17:00 일과
- 19:30 ~ 22:00 운동 (웜업 → 본 운동 → 쿨다운)
- 22:00 운동 후 셰이크
- 23:00 취침 전 루틴
- 23:30 취침

---

## 배포

```bash
# Netlify 배포
netlify deploy --prod

# Netlify 사이트 ID
# .netlify/state.json → siteId: "1349aeb3-70af-4930-9e56-d0fc4212d38c"
```

---

## 참고 파일

| 파일 | 용도 |
|---|---|
| `index.html` | 전체 앱 소스 (단일 파일) |
| `Junho_근력증강_플랜_v10.md` | PDF v10 운동·식단 플랜 (마크다운 변환본) |
| `Junho_근력증강_플랜_v10.pdf` | 원본 PDF |
| `APP_GUIDE.md` | 이 파일 — 앱 기능 및 구현 가이드 |
| `.netlify/state.json` | Netlify 사이트 ID |
