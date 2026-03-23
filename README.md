# JANG HQ — Dashboard

개인용 홈페이지 대시보드 프로젝트입니다.

## 주요 기능

- **캘린더** — iframe 기반 캘린더 위젯 연동
- **할 일 관리** — 우선순위(높음/중간/낮음), 마감일, 시간, 상태(진행중/완료/지연/연기) 관리
- **메모** — 카테고리별(업무/가족/개인) 메모 작성 및 관리
- **바로가기** — Google Drive, Gmail, YouTube Music, Notion 등 자주 쓰는 서비스 빠른 접근
- **듀얼 시계** — 두 개 시간대 동시 표시
- **설정** — 테마 색상, 캘린더 URL 등 커스터마이징

## 기술 스택

- HTML / CSS / JavaScript (단일 파일 구성)
- Google Fonts (Outfit, Noto Sans KR, JetBrains Mono)
- LocalStorage 기반 데이터 저장

## 사용 방법

`index.html` 파일을 브라우저에서 열면 바로 사용할 수 있습니다.

## Git 사용법

로컬 저장소와 GitHub가 이미 연결되어 있는 상태입니다.
아래 명령어는 모두 **터미널(명령 프롬프트)** 에서 프로젝트 폴더 안에서 실행합니다.

### 사전 준비: PAT (Personal Access Token)

GitHub에서 발급받은 PAT이 필요합니다. 아직 없다면:

1. https://github.com/settings/tokens?type=beta 접속
2. **Generate new token** 클릭
3. Token name: 아무 이름 (예: `my-token`)
4. Repository access → **Only select repositories** → `GameEtc` 선택
5. Permissions → **Contents** → `Read and write`
6. **Generate token** 클릭 후 `github_pat_xxxx...` 토큰 복사

> 토큰은 생성 직후에만 확인 가능합니다. 반드시 바로 복사해두세요.

---

### 최신 파일 가져오기 (Pull)

GitHub에 올라간 가장 최근 파일을 내 컴퓨터로 받아오는 방법입니다.

```bash
# 1단계: 프로젝트 폴더로 이동
cd GameEtc

# 2단계: 최신 파일 가져오기
git pull https://본인아이디:PAT토큰@github.com/junyoungjang-kor/GameEtc.git main
```

**실제 입력 예시:**
```bash
git pull https://junyoungjang-kor:github_pat_xxxx@github.com/junyoungjang-kor/GameEtc.git main
```

> `github_pat_xxxx` 부분을 본인의 실제 PAT 토큰으로 바꿔주세요.

---

### 내 수정사항 올리기 (Push)

로컬에서 `index.html`을 수정한 뒤 GitHub에 반영하는 방법입니다.

```bash
# 1단계: 프로젝트 폴더로 이동
cd GameEtc

# 2단계: 변경된 파일 확인
git status

# 3단계: 변경된 파일을 커밋 대상으로 추가
git add index.html

# 4단계: 커밋 (수정 내용을 기록)
git commit -m "수정: 변경 내용을 여기에 작성"

# 5단계: GitHub에 올리기
git push https://본인아이디:PAT토큰@github.com/junyoungjang-kor/GameEtc.git main
```

**실제 입력 예시:**
```bash
git add index.html
git commit -m "수정: 바로가기 링크 URL 변경"
git push https://junyoungjang-kor:github_pat_xxxx@github.com/junyoungjang-kor/GameEtc.git main
```

---

### 자주 쓰는 명령어 요약

| 상황 | 명령어 |
|------|--------|
| 변경된 파일 확인 | `git status` |
| 변경 내용 상세 비교 | `git diff` |
| 최신 파일 가져오기 | `git pull https://아이디:PAT@github.com/junyoungjang-kor/GameEtc.git main` |
| 파일 추가 + 커밋 + 푸시 | `git add .` → `git commit -m "메시지"` → `git push https://아이디:PAT@github.com/junyoungjang-kor/GameEtc.git main` |
| 커밋 이력 확인 | `git log --oneline -10` |

---

### 문제 해결

**"rejected - fetch first" 오류가 뜰 때:**
GitHub에 더 최신 파일이 있다는 뜻입니다. pull을 먼저 하세요.
```bash
git pull --rebase https://아이디:PAT@github.com/junyoungjang-kor/GameEtc.git main
# 그 다음 다시 push
git push https://아이디:PAT@github.com/junyoungjang-kor/GameEtc.git main
```

**"CONFLICT" 오류가 뜰 때:**
같은 부분을 양쪽에서 수정한 경우입니다. `index.html`을 열어 `<<<<<<<` 표시를 찾아 원하는 내용만 남기고 저장한 뒤:
```bash
git add index.html
git rebase --continue
# 그 다음 다시 push
```

## 단축키

| 단축키 | 기능 |
|--------|------|
| `Ctrl + M` | 새 메모 작성 |
| `Ctrl + Enter` | 새 할 일 작성 |
| `Esc` | 모달 닫기 |

## 프로젝트 구조

```
GameEtc/
├── index.html    # 전체 애플리케이션 (HTML + CSS + JS)
├── README.md     # 이 파일
└── CLAUDE.md     # AI 작업 가이드
```
