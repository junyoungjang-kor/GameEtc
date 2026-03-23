# JANG HQ Dashboard — 코드 감사 보고서

**작성일:** 2026-03-23
**대상 파일:** `index.html` (822줄)
**분석 범위:** 정합성, 함수 호출/도달, 버그, 오류 가능성, 개선 제안

---

## 1. 함수 호출 및 도달 분석

### 전체 함수 목록 (29개)

| # | 함수명 | 호출 위치 | 도달 여부 |
|---|--------|-----------|-----------|
| 1 | `formatClock(tz)` | `updateClock()` | ✅ 도달 |
| 2 | `updateClock()` | 직접 호출 + `setInterval` | ✅ 도달 |
| 3 | `escapeHtml(s)` | `loadTodos`, `loadMemos`, `renderQuickLinks` | ✅ 도달 |
| 4 | `openSettings()` | HTML `onclick` (3곳) + 키보드 미연결 | ✅ 도달 |
| 5 | `closeSettings()` | HTML `onclick` + `Esc` 키 | ✅ 도달 |
| 6 | `saveSettings()` | HTML `onclick` | ✅ 도달 |
| 7 | `extractCalendarId(input)` | `saveSettings()` | ✅ 도달 |
| 8 | `extractAppleMusicUrl(input)` | `saveSettings()` | ✅ 도달 |
| 9 | `applyConnections()` | `saveSettings()` + 페이지 로드 시 | ✅ 도달 |
| 10 | `saveTodos()` | todo 관련 함수 전반 | ✅ 도달 |
| 11 | `getDueDateLabel(dueDate)` | `loadTodos()` | ✅ 도달 |
| 12 | `loadTodos()` | 직접 호출 + todo 변경 후 | ✅ 도달 |
| 13 | `cycleTodoStatus(id)` | `loadTodos()` 내 HTML onclick | ✅ 도달 |
| 14 | `addTodo()` | HTML `onclick` + `Enter` 키 | ✅ 도달 |
| 15 | `openTodoEditor(id, preText)` | `addTodo()` + HTML onclick + `Ctrl+Enter` | ✅ 도달 |
| 16 | `closeTodoEditor()` | HTML `onclick` + `Esc` 키 | ✅ 도달 |
| 17 | `saveTodoItem()` | HTML `onclick` | ✅ 도달 |
| 18 | `deleteTodo(id)` | `loadTodos()` 내 HTML onclick | ✅ 도달 |
| 19 | `deleteTodoFromModal()` | HTML `onclick` | ✅ 도달 |
| 20 | `filterTodo(btn, f)` | HTML `onclick` | ✅ 도달 |
| 21 | `saveMemos()` | memo 관련 함수 전반 | ✅ 도달 |
| 22 | `loadMemos()` | 직접 호출 + memo 변경 후 | ✅ 도달 |
| 23 | `filterMemo(btn, f)` | HTML `onclick` | ✅ 도달 |
| 24 | `openMemoEditor(id)` | HTML `onclick` + `Ctrl+M` | ✅ 도달 |
| 25 | `closeMemoEditor()` | HTML `onclick` + `Esc` 키 | ✅ 도달 |
| 26 | `saveMemo()` | HTML `onclick` | ✅ 도달 |
| 27 | `deleteMemo()` | HTML `onclick` | ✅ 도달 |
| 28 | `renderQuickLinks()` | 직접 호출 + `addQuickLink()` | ✅ 도달 |
| 29 | `addQuickLink()` | HTML `onclick` | ✅ 도달 |

**결과: 미도달 함수 0개. 모든 함수가 정상적으로 호출 경로를 가지고 있음.**

---

## 2. 버그 및 오류

### 🔴 심각 (즉시 수정 필요)

#### BUG-01: `extractCalendarId()` 두 번째 정규식 도달 불가 (데드 코드)
- **위치:** 519~520줄
- **내용:** 첫 번째 정규식 `/[?&]src=([^&]+)/` 이 이미 모든 `src=` 파라미터를 잡으므로, 두 번째 정규식 `/calendar\.google\.com.*[?&]src=([^&]+)/` 은 절대 실행되지 않는다.
- **영향:** 기능상 문제없으나 데드 코드가 유지보수 혼란을 야기함.
- **수정:** 두 번째 정규식 제거, 또는 순서를 바꿔 더 엄격한 것을 먼저 검사.

#### BUG-02: iframe `onerror` 이벤트가 실제로 발동하지 않음
- **위치:** 556줄
- **내용:** iframe의 `onerror`는 cross-origin 로드 실패(X-Frame-Options, CSP 차단) 시 발동하지 않는다. 브라우저 보안 정책상 iframe 로딩 실패를 감지하는 표준 방법이 없다.
- **영향:** 캘린더가 차단되어도 사용자에게 안내 메시지가 표시되지 않고, placeholder도 숨겨진 상태로 빈 영역만 보임.
- **수정:** `onload` 후 타임아웃 기반 확인, 또는 placeholder에 항상 안내 문구를 표시하는 방식으로 변경.

#### BUG-03: `Esc` 키가 모든 모달 닫기 함수를 동시 호출
- **위치:** 813줄
- **내용:** `if(e.key==='Escape'){closeSettings();closeMemoEditor();closeTodoEditor();}` — 열린 모달이 하나여도 3개 함수 모두 호출.
- **영향:** 기능은 동작하나, `closeTodoEditor()`가 `editingTodoId=null` 및 입력필드를 초기화하므로, 설정 모달을 닫을 때 할일 편집 상태가 의도치 않게 리셋될 수 있음.
- **수정:** 열린 모달만 닫도록 조건 분기 추가.

---

### 🟡 경고 (개선 권장)

#### BUG-04: `formatClock()` 내 미사용 변수 `d`
- **위치:** 470줄
- **내용:** `const d=['일','월','화','수','목','금','토'];` 선언 후 사용되지 않음. `weekMap`과 중복.
- **영향:** 메모리 미미하지만, 불필요한 코드.

#### BUG-05: `formatClock()` 내 `weekMap`이 자기 자신으로 매핑
- **위치:** 475줄
- **내용:** `{일:'일',월:'월',...}` — `Intl.DateTimeFormat`의 `ko-KR` 로케일이 이미 한국어 요일을 반환하므로 이 매핑은 무의미.
- **영향:** 없음. 불필요 코드.

#### BUG-06: 바로가기 삭제/편집 불가
- **위치:** 801~808줄
- **내용:** `addQuickLink()`은 있으나, 삭제/편집 함수가 없음. 잘못 추가한 바로가기를 제거하려면 localStorage를 직접 조작해야 함.
- **영향:** 사용자 경험 저하.

#### BUG-07: 빈 URL 바로가기에 시각적 피드백 없음
- **위치:** 773~774줄 (가계부, 운동 기록)
- **내용:** URL이 빈 문자열(`''`)인 바로가기를 클릭하면 아무 반응이 없음. 사용자가 고장으로 오인할 수 있음.
- **영향:** 사용자 혼란.

#### BUG-08: 할 일 시간 검증 없음
- **위치:** 695~707줄
- **내용:** 시작 시간이 종료 시간보다 늦어도 저장됨. 예: 시작 18:00, 종료 09:00.
- **영향:** 데이터 무결성 이슈.

#### BUG-09: `quickLinks` 페이지 로드마다 localStorage 덮어쓰기
- **위치:** 782줄
- **내용:** `localStorage.setItem('jangHQ_links',JSON.stringify(quickLinks));` — 변경 없어도 매번 쓰기.
- **영향:** 성능상 무시할 수준이나 불필요한 연산.

#### BUG-10: `calOpenBtn` 링크가 사용자 캘린더와 무관
- **위치:** 258줄
- **내용:** `href="https://calendar.google.com"` 고정. 캘린더 ID를 설정해도 해당 캘린더 페이지로 연결되지 않음.
- **영향:** 미미하나 개선 가능.

---

### 🟢 참고 (현재 문제없음)

| 항목 | 상태 | 비고 |
|------|------|------|
| XSS 방어 | ✅ 양호 | `escapeHtml()` 일관 적용 |
| LocalStorage 파싱 | ✅ 양호 | `\|\|'[]'` / `\|\|'{}'` 폴백 |
| 구버전 마이그레이션 | ✅ 양호 | `done` → `status` 자동 변환 |
| 샘플 데이터 1회 초기화 | ✅ 양호 | `_initialized` 플래그 정상 |
| 반응형 레이아웃 | ✅ 양호 | 1024px / 640px 대응 |
| 시간대 처리 | ✅ 양호 | `Intl.DateTimeFormat` 사용 |
| ID 생성 (`Date.now()`) | ✅ 허용 | 수동 입력이므로 충돌 가능성 극히 낮음 |

---

## 3. 정합성 종합 평가

| 영역 | 점수 | 평가 |
|------|------|------|
| HTML 구조 | 9/10 | 시맨틱 구조 양호, 접근성(aria) 미적용 |
| CSS 일관성 | 9/10 | 변수 체계 우수, 반응형 적절 |
| JS 함수 도달성 | 10/10 | 미사용 함수 없음 |
| 데이터 무결성 | 7/10 | 시간 검증 부재, 빈 URL 미처리 |
| 보안 | 8/10 | XSS 방어 양호, iframe sandbox 적용, onerror 미작동 |
| UX 완성도 | 7/10 | 바로가기 편집 불가, 빈 URL 피드백 부재 |
| 코드 청결도 | 8/10 | 데드 코드 2건, 불필요 변수 1건 |

**종합: 8.3 / 10** — 단일 파일 대시보드로서 높은 완성도. 핵심 기능 모두 정상 동작.

---

## 4. 발전 제안

### 🥇 우선순위 높음 (실용성 즉시 향상)

| # | 제안 | 설명 |
|---|------|------|
| 1 | **바로가기 편집/삭제** | 우클릭 또는 길게 누르면 편집/삭제 메뉴. 현재는 추가만 가능하여 오입력 시 복구 불가. |
| 2 | **날씨 위젯** | 바그다드/서울 현재 날씨 표시. 파견 환경에서 기온·미세먼지 확인 유용. OpenWeatherMap 무료 API 활용 가능. |
| 3 | **할 일 정렬** | 우선순위·마감일·상태별 정렬 옵션. 현재는 입력 순서 고정. |
| 4 | **D-Day 카운터** | 파견 종료일, 가족 기념일 등 중요 일자까지 남은 날 표시. 헤더 또는 별도 위젯. |

### 🥈 우선순위 중간 (완성도 향상)

| # | 제안 | 설명 |
|---|------|------|
| 5 | **데이터 백업/복원** | LocalStorage 전체를 JSON으로 내보내기/가져오기. 브라우저 변경·캐시 삭제 시 데이터 유실 방지. |
| 6 | **할 일 드래그 정렬** | 마우스 드래그로 할 일 순서 변경. 직관적 우선순위 관리. |
| 7 | **메모 검색** | 메모가 많아지면 제목/본문 키워드 검색 필요. |
| 8 | **알림/리마인더** | 마감일 당일 할 일을 시각적으로 강조하거나, 브라우저 알림(Notification API) 발송. |
| 9 | **환율 정보** | KRW↔IQD 환율 표시. 파견 중 현지 금액 환산에 유용. |

### 🥉 우선순위 낮음 (장기 발전)

| # | 제안 | 설명 |
|---|------|------|
| 10 | **테마 커스터마이징** | 액센트 색상 변경 가능. 현재 파란색 고정. |
| 11 | **PWA 변환** | Service Worker 추가로 오프라인 완전 지원 + 홈 화면 설치. |
| 12 | **위젯 배치 변경** | 드래그로 카드 위치·크기 커스터마이징. |
| 13 | **통계 대시보드** | 할 일 완료율, 주간/월간 달성도 차트. |

---

## 5. 즉시 수정 가능한 항목 요약

| 버그 | 난이도 | 예상 수정 범위 |
|------|--------|----------------|
| BUG-01 데드 코드 | 쉬움 | 2줄 삭제 |
| BUG-03 Esc 키 분기 | 쉬움 | 3줄 → 6줄 |
| BUG-04 미사용 변수 | 쉬움 | 1줄 삭제 |
| BUG-05 무의미 매핑 | 쉬움 | 2줄 삭제 |
| BUG-07 빈 URL 피드백 | 쉬움 | CSS opacity 또는 커서 변경 |
| BUG-02 iframe 오류 감지 | 보통 | onerror → 타임아웃 방식으로 변경 |
| BUG-06 바로가기 편집/삭제 | 보통 | 새 함수 2개 + UI 추가 |

---

*보고서 끝*
