# CLAUDE.md — JANG HQ Dashboard 작업 가이드

## 프로젝트 개요

**JANG HQ**는 개인 생산성 대시보드로, 단일 `index.html` 파일에 HTML/CSS/JS가 모두 포함된 올인원 웹 애플리케이션이다.
이라크 파견 중인 작업자가 서울과 바그다드 시간을 동시에 확인하며, 일정·할일·메모·음악·바로가기를 한 화면에서 관리하기 위해 만들었다.

## 설계 원칙

- **단일 파일 유지**: `index.html` 하나로 모든 것을 처리한다. 별도 JS/CSS 파일로 분리하지 않는다. 브라우저에서 파일 하나만 열면 바로 동작해야 한다.
- **프레임워크 없음**: 바닐라 HTML/CSS/JS만 사용한다. 외부 라이브러리나 빌드 도구를 도입하지 않는다.
- **오프라인 우선**: 모든 데이터는 LocalStorage에 저장한다. 서버나 DB 의존성이 없다.
- **다크 테마 고정**: 어두운 배경(`#0a0a0f`) 기반의 모던 UI를 유지한다. 라이트 모드는 고려하지 않는다.
- **한국어 UI**: 모든 사용자 대면 텍스트는 한국어로 작성한다.

## 아키텍처

### 파일 구조
```
GameEtc/
├── index.html    # 전체 애플리케이션 (HTML + CSS + JS)
├── README.md     # 프로젝트 문서
└── CLAUDE.md     # 이 파일
```

### index.html 내부 구조 (순서)
1. **`<head>`** — 메타, Google Fonts 임포트, `<style>` (CSS 변수 → 레이아웃 → 컴포넌트 → 반응형)
2. **`<body>`** — 앰비언트 배경 → 헤더(듀얼시계) → 대시보드 그리드 → 모달들
3. **`<script>`** — CLOCK → UTILS → SETTINGS → TODO → MEMO → QUICK LINKS → KEYBOARD

### 대시보드 그리드 배치
```
┌──────────────────┬──────────────┐
│  Google Calendar  │   할 일 관리  │
│  (2행 차지)       │              │
│                  ├──────────────┤
│                  │   메모        │
├──────────────────┼──────────────┤
│  Apple Music     │  바로가기     │
└──────────────────┴──────────────┘
```

## 코드 컨벤션

### JavaScript
- **절차적 스타일**: 클래스 없이 함수 단위로 구성한다.
- **네이밍**: camelCase 사용. DOM 요소 참조 시 약어 허용 (`t` = todo, `m` = memo).
- **섹션 구분**: `// ── SECTION NAME ──` 형태의 주석으로 코드 영역을 구분한다.
- **HTML 이스케이프**: 사용자 입력을 DOM에 삽입할 때는 반드시 `escapeHtml()` 함수를 사용한다.

### CSS
- **CSS 변수**: 색상은 `:root`에 정의된 변수(`--accent-blue`, `--bg-card` 등)를 사용한다. 하드코딩하지 않는다.
- **트랜지션**: `0.25s cubic-bezier(0.4,0,0.2,1)` 기본 이징을 따른다.
- **반응형**: 1024px / 640px 브레이크포인트. 모바일에서는 1컬럼 레이아웃.

### LocalStorage 키
| 키 | 용도 |
|----|------|
| `jangHQ_config` | 설정 (캘린더 ID, Apple Music URL) |
| `jangHQ_todos` | 할 일 목록 배열 |
| `jangHQ_todos_initialized` | 샘플 데이터 초기화 플래그 |
| `jangHQ_memos` | 메모 목록 배열 |
| `jangHQ_memos_initialized` | 샘플 데이터 초기화 플래그 |
| `jangHQ_links` | 바로가기 링크 배열 |

## 기능별 핵심 사항

### 듀얼 시계
- 서울(`Asia/Seoul`)과 바그다드(`Asia/Baghdad`) 시간을 1초 간격으로 갱신한다.
- `Intl.DateTimeFormat`을 사용하여 로케일 포맷팅한다.

### 할 일 관리
- 우선순위: high(빨강) / medium(주황) / low(초록)
- 상태: active → done / delayed / postponed (순환 토글 지원)
- 마감일 초과 시 자동 시각적 경고
- 빈 제목 입력 차단

### 메모
- 카테고리: personal / work / family
- 탭 기반 필터링
- 본문 미리보기 (50자 이내)

### 외부 서비스 연동
- **Google Calendar**: iframe embed. `extractCalendarId()`로 이메일/URL 양쪽 입력 지원.
- **Apple Music**: iframe embed. `extractAppleMusicUrl()`로 일반 URL → embed URL 자동 변환. `music.apple.com` 외 URL은 거부한다.
- **바로가기**: 이모지 아이콘 + 외부 링크. 사용자 추가 가능.

### iframe 보안
- Google Calendar: `sandbox="allow-scripts allow-same-origin allow-popups"` + 로드 실패 시 fallback 메시지
- Apple Music: `sandbox="allow-forms allow-popups allow-same-origin allow-scripts allow-storage-access-by-user-activation allow-top-navigation-by-user-activation"`

## 작업 시 주의사항

1. **단일 파일 유지**: 새 파일을 만들지 않는다. 모든 변경은 `index.html` 안에서 한다.
2. **샘플 데이터 보존**: `_initialized` 플래그 로직을 건드리지 않는다. 사용자가 삭제한 데이터가 다시 생성되면 안 된다.
3. **XSS 방지**: 사용자 입력 → DOM 삽입 시 `escapeHtml()` 필수. `innerHTML`에 raw 입력을 넣지 않는다.
4. **URL 파싱 함수 수정 시 주의**: `extractCalendarId()`와 `extractAppleMusicUrl()`은 다양한 입력 형태를 처리한다. 수정 시 기존 케이스가 깨지지 않는지 확인한다.
5. **CSS 변수 우선**: 새로운 색상이 필요하면 기존 변수를 활용하거나 `:root`에 추가한다.
6. **한국어**: UI 텍스트, placeholder, alert 메시지 모두 한국어로 작성한다.
7. **키보드 단축키**: 새 기능 추가 시 단축키 충돌 여부를 확인한다. 현재: `Ctrl+M`(메모), `Ctrl+Enter`(할일), `Esc`(모달닫기).

## Git 컨벤션

### 브랜치
- 작업 브랜치: `claude/setup-github-dashboard-jGH1Q`

### 커밋 규칙 (필수)

**모든 작업 단위는 완료 즉시 커밋 + 푸시한다.** 여러 작업을 묶지 않고, 하나의 논리적 변경마다 즉시 반영한다.

#### 커밋 메시지 형식
```
<타입>: <변경 요약> (한글, 1줄)

[변경 내역]
- <수정한 내용 1>
- <수정한 내용 2>
- ...

[사유]
- <왜 이 변경이 필요했는지>
```

#### 타입 목록
| 타입 | 용도 |
|------|------|
| `기능` | 새로운 기능 추가 |
| `수정` | 버그 수정 |
| `개선` | 기존 기능의 동작·성능·UX 향상 |
| `리팩터` | 동작 변경 없는 코드 구조 정리 |
| `문서` | README, CLAUDE.md 등 문서 변경 |
| `스타일` | CSS/디자인 변경 (기능 변화 없음) |

#### 커밋 메시지 예시
```
수정: Apple Music 위젯 플레이리스트 미표시 문제 해결

[변경 내역]
- extractAppleMusicUrl()에서 embed URL 판별을 일반 URL 변환보다 우선 처리
- iframe width를 100%로 고정하여 레이아웃 깨짐 방지

[사유]
- embed.music.apple.com URL 입력 시 이중 변환되어 위젯이 로드되지 않던 문제
```

#### 주의사항
- **커밋 메시지는 반드시 한글로 작성한다.**
- 제목 줄은 50자 이내로 간결하게 쓴다.
- `[변경 내역]`에는 무엇을 바꿨는지, `[사유]`에는 왜 바꿨는지를 구분하여 기록한다.
- 작업 단위가 끝나면 즉시 커밋하고 푸시한다. 작업을 모아두지 않는다.

### Push/Pull
- 이 프로젝트에서는 PAT을 이용한 직접 URL 방식으로 push/pull한다.
- 프록시(`origin`) 대신 아래 형식 사용:
  ```
  git push https://<user>:<PAT>@github.com/junyoungjang-kor/GameEtc.git <branch>
  ```
- push 후 로컬 트래킹 참조를 `git update-ref`로 동기화한다.

## CLAUDE.md 운영 규칙

- 작업 중 새로운 공통 규칙이나 패턴이 발견되면 CLAUDE.md에 반영한다.
- 작업자의 의도와 방향성이 담긴 결정 사항은 해당 섹션에 추가하여 이후 작업의 일관성을 보장한다.
- CLAUDE.md 수정도 별도 커밋으로 관리한다 (타입: `문서`).

## 테스트 방법

빌드 도구가 없으므로 다음으로 검증한다:
1. `index.html`을 브라우저에서 직접 열어 동작 확인
2. LocalStorage를 비운 상태에서 초기 로딩 테스트 (샘플 데이터 생성 확인)
3. 각 모달(설정/할일/메모) CRUD 동작 확인
4. 반응형: 브라우저 창 크기를 줄여가며 레이아웃 확인
