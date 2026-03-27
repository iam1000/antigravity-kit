# Antigravity 환경 완벽 이전 가이드 (Migration Guide)

현재 로컬 PC에 세팅된 `antigravity-kit` 워크스페이스와 AI 에이전트(천재소년) 글로벌 환경을 **다른 PC로 똑같이(Mirrored)** 옮기기 위한 총정리 점검표와 복사/백업 자동화 스크립트입니다.

새 PC에서 똑같이 작업하시기 위해 반드시 검토하고 챙겨야 할 **핵심 체크리스트 5가지**를 정리해 드립니다!
## 요약 가이드
1. C:\AI_DEV\antigravity-kit 폴더 전체 복사
2. C:\Users\1\.gemini 사용자 환경 폴더 복사
3. 새 PC에서 파이썬/Node.js 구성 후 디펜던시 설치
4. Supabase 및 외부 MCP 계정 재로그인

---

## 📋 핵심 체크리스트 5가지

### 1️⃣ 프로젝트 폴더 및 Git 이관
현재 워크스페이스에는 `.agent` (에이전트 스킬/스크립트), `.git` 및 방금 추가하신 문서 등이 존재합니다.
* **[🚨 필수 선행 작업] Git 미설치 PC:** 새 PC에 Git 자체가 없다면 코드를 연동하거나 버전 관리를 이어갈 수 없으므로, **먼저 [Git 공식 사이트(git-scm.com)](https://git-scm.com/downloads)에서 Windows용 Git을 다운로드 및 설치해 주셔야 합니다.**
* **현재 Git 리모트 URL 확인 스크립트:** 기존 환경(현재 PC)에서 워크스페이스가 어디로 연동되어 있는지 확인하고 싶다면 아래 명령어를 입력하세요.
  ```bash
  git config --get remote.origin.url
  ```
* **이동 방법 1 (Git Clone 권장):** 리모트 저장소에 코드를 `Push` 한 뒤, 새 PC의 터미널에서 아래 명령어로 프로젝트를 복제합니다.
  ```bash
  git clone https://github.com/iam1000/antigravity-kit.git
  ```
* **이동 방법 2 (직접 복사):** 보안 등 이슈로 Git을 쓸 수 없을 땐, 물리적으로 전체 폴더(`C:\AI_DEV\antigravity-kit`)를 압축해 넘기셔도 무방합니다. **이 폴더 안에는 숨김 처리된 `.git` 폴더가 포함되어 있기 때문에 기존 연결된 GitHub 주소(Remote URL), 브랜치, 커밋 히스토리가 새 PC로 동일하게 유지됩니다.**
* **주의사항 (직접 복사 시 필독):** 
  1. 물리적으로 복사를 진행할 때에는 `.temp_ag_kit` 폴더와 같이 사용 중 락(lock)이 걸리는 임시 폴더 및 무거운 로컬 의존성 폴더(`node_modules` 등)는 제외하는 것이 깔끔합니다.
  2. 소스와 깃 연결 히스토리는 통째로 이동되지만 **새 PC 시스템의 Git 글로벌 설정 및 자격 증명(인증)**은 복사되지 않으므로, 새 PC에서 아래와 같은 글로벌 계정 세팅 작업을 1회 진행해야 `git push`가 정상 동작합니다 (`git config --global user.name "내 이름"`, `git config --global user.email "내 이메일"` 입력 후 첫 `푸시(Push)` 시 GitHub 권한 로그인 팝업 확인 필요).

### 2️⃣ 전역(Global) AI 스킬 및 아티팩트 보존
아까 확인하셨듯이 `word-report`, `docx`, `pptx` 같은 전역 스킬은 현재 프로젝트 폴더가 아닌 사용자 계정 폴더에 저장되어 있습니다.
* **백업 경로:** `C:\Users\[사용자명]\.gemini\` 폴더 전체
* **작업:** 이 폴더 안에는 아티팩트(다이어그램 등)의 히스토리와 글로벌 스킬 디렉터리가 포함되어 있습니다. 새 PC에 동일하게 해당 폴더를 복사해 주셔야 똑같은 AI 확장 기능(전역 스킬)을 이용할 수 있습니다.

### 3️⃣ MCP(Model Context Protocol) 및 인증 설정
이 프로젝트는 Supabase와 NotebookLM 등 외부 MCP 서버와 통신하고 있습니다.
* **Supabase:** 새 PC에서 터미널을 열고 `supabase login` 명령어를 통해 다시 인증(Token 발급)을 해야 데이터를 정상적으로 조회할 수 있습니다.
* **NotebookLM 및 기타 MCP:** `mcp_config.json` 관련 설정이 있다면 새 PC의 경로에 맞게 점검이 필요하며, 인증 토큰(`notebooklm-mcp-auth`)도 기기가 바뀌면 새로 발급/로그인해야 동작합니다.

### 4️⃣ 개발 언어 및 런타임 환경 (Python / Node.js)
현재 에이전트가 `.agent/scripts/*.py` 스크립트를 백그라운드에서 구동(예: `session_manager.py`)하거나, 포트 5173 등에서 Node.js 기반 로컬 테스트 서버를 띄우고 있습니다. 
**이러한 자바스크립트/파이썬 런타임(구동 엔진)은 폴더를 복사한다고 해서 넘어가는 것이 아니므로 새 PC 운영체제에 반드시 새로 설치되어 있어야 합니다!**

* **[🚨 필수 설치] 파이썬(Python) 3.x:** 새 PC에 설치되어 있어야 에이전트 자동 점검(Audit) 스크립트 및 다양한 분석 스크립트가 멈추지 않고 동작합니다. 안 깔려있다면 [Python 공식 웹사이트](https://www.python.org/downloads/)에서 설치해 주세요. (⭐설치 시 `Add Python to PATH` 옵션 체크 필수)
* **[🚨 필수 설치] Node.js (npm):** 코드를 실행하기 위해서는 새 PC에도 Node.js 런타임이 있어야 합니다. 미설치 상태라면 [Node.js 공식 웹사이트](https://nodejs.org/)에서 안정화(LTS) 버전을 다운로드하고 설치하세요. 설치 후 터미널 창(프로젝트 폴더 안)에서 `npm install`을 치면 필요한 모듈 조각들이 알아서 채워집니다.

### 5️⃣ 환경 변수 파일(`.env`) 복사
Git 리포지토리로 소스를 옮기신다면 보안상 `.env` 등 중요 키가 들어있는 파일은 푸시되지 않았을 확률이 큽니다.
* **작업:** `C:\AI_DEV\antigravity-kit`의 최상단이나 내부 서브 프로젝트에 있는 `.env`, `.env.local` 파일들을 수동으로 복사해서 새 PC의 같은 위치에 붙여넣어 주세요.

---

## 🚀 자동화 백업 스크립트 (PowerShell)

기존 PC에서 아래 스크립트를 관리자 권한 PowerShell에서 실행하면 지정한 백업용 드라이브(예: `D:\Antigravity_Backup`)로 필요 파일만 압축/복사해 줍니다. 백업 경로(`$BackupDir`)는 본인 환경(USB 등) 경로에 맞게 수정하세요.

```powershell
# 1. 백업 대상 디렉터리 변수 설정
$ProjectDir = "C:\AI_DEV\antigravity-kit"
$GlobalGeminiDir = "C:\Users\1\.gemini"
$BackupDir = "D:\Antigravity_Backup" # 대상 백업 폴더 경로 (USB 등 지정)

# 백업 폴더 생성
if (-Not (Test-Path "$BackupDir")) {
    New-Item -ItemType Directory -Force -Path "$BackupDir"
}

Write-Host "[1/2] 진행 중: 프로젝트 폴더 백업 중 (무거운 임시 파일 제외)..." -ForegroundColor Cyan
# .git 및 node_modules 등을 제외하고 싶다면 아래 Exclude 옵션 추가
# Copy-Item -Path $ProjectDir -Destination "$BackupDir" -Recurse -Force -Exclude "node_modules", ".temp_ag_kit"
Copy-Item -Path $ProjectDir -Destination "$BackupDir" -Recurse -Force

Write-Host "[2/2] 진행 중: 전역 AI 환경 폴더(.gemini) 백업 중..." -ForegroundColor Cyan
Copy-Item -Path $GlobalGeminiDir -Destination "$BackupDir" -Recurse -Force

Write-Host "✅ 백업이 완료되었습니다! $BackupDir 폴더를 새 PC로 옮겨주세요." -ForegroundColor Green
```

---

## 💻 새 PC에서의 복구 스크립트 (PowerShell)

백업본을 대상 PC의 `D:\Antigravity_Backup` (예시)에 가져오신 후, 새 시의 PowerShell 창에서 아래 명령어를 순차적으로 실행해 주세요. (미리 설치된 Node.js, Python 환경 기준)

```powershell
$BackupDir = "D:\Antigravity_Backup"
$TargetProjectDir = "C:\AI_DEV\antigravity-kit"
$TargetGeminiDir = "C:\Users\$ENV:USERNAME\.gemini"

# 1. 기존 백업 폴더를 복사
Copy-Item -Path "$BackupDir\antigravity-kit" -Destination $TargetProjectDir -Recurse -Force
Copy-Item -Path "$BackupDir\.gemini" -Destination $TargetGeminiDir -Recurse -Force

# 2. 프로젝트 폴더로 진입 후 필수 디펜던시 재설치
cd $TargetProjectDir
Write-Host "Node.js 패키지 및 Python 환경 재구성..." -ForegroundColor Cyan
npm install 
# (pip 등 패키지 설치용 파일이 존재할 경우: pip install -r requirements.txt 실행)

# 3. Supabase 및 외부 MCP 재인증 (필요시)
Write-Host "Supabase CLI 토큰 갱신 로그인을 진행합니다." -ForegroundColor Yellow
supabase login

Write-Host "✅ 복구가 완료되었습니다. 이제 에이전트와 완벽하게 동일한 환경에서 작업하실 수 있습니다!" -ForegroundColor Green
```
