# Git 브랜치 관리 전략 (`dev` ➜ `main` 워크플로우)

## 1. 개요 및 목표
* **현재 상태:** `main` 브랜치 하나만 사용하여 직접 커밋 및 푸시 진행 중.
* **목표 상태:** 개발 및 테스트 전용인 `dev` 브랜치를 분리합니다. 로컬에서 `dev` 브랜치로 작업 후 GitHub 원격 저장소에 푸시하고, 테스트가 완료되면 GitHub 환경에서 `dev` 브랜치를 `main` 브랜치로 병합(Merge/Pull Request)하여 안정적인 배포(Production) 환경을 유지합니다.

---

## 2. 브랜치 전략 구조

| 브랜치 이름 | 역할 및 특징 |
| :--- | :--- |
| **`main`** | **[실제 배포용 / 운영 환경]**<br>가장 안정적인 코드가 유지되어야 하는 브랜치입니다. 개발자가 로컬에서 직접 커밋을 올리지 않으며, 반드시 `dev` 브랜치에서 검증이 끝난 후 병합(Merge)만 이루어집니다. |
| **`dev`** | **[개발 및 테스트 환경]**<br>새로운 기능 개발 로직, 버그 픽스 등이 자유롭게 테스트되는 기본 작업 브랜치입니다. 로컬 작업 후 원격(GitHub)의 `dev` 브랜치로 푸시합니다. |

*(추가 권장사항)* 기능 단위의 작업이 커질 경우 `feature/login`, `feature/ui-fix` 와 같이 `dev`에서 파생된 하위 브랜치를 사용하여 작업 후 `dev`로 병합하는 방식을 도입할 수도 있습니다.

---

## 3. 초기 설정 (Transition Guide)
현재 `main` 브랜치만 있는 상태에서 `dev` 브랜치를 생성하고 적용하는 초기 1회 설정 방법입니다.

1. **로컬에서 최신 `main` 브랜치 확인**
   ```bash
   git checkout main
   git pull origin main
   ```

2. **`dev` 브랜치 생성 및 이동**
   ```bash
   git checkout -b dev
   ```

3. **원격 저장소(GitHub)에 `dev` 브랜치 푸시 및 연동**
   ```bash
   git push -u origin dev
   ```
   > 💡 *-u (또는 --set-upstream) 옵션을 주어야 이후부터는 `git push`와 `git pull`만 입력해도 자동으로 `origin/dev`와 연결됩니다.*

---

## 4. 일상적인 개발 프로세스 (Daily Workflow)

### Step 1. 개발 작업 시작 전
항상 자신이 `dev` 브랜치에 있는지 확인하고, 동료의 최신 업데이트를 내려받습니다.
```bash
git branch       # 현재 브랜치가 * dev 인지 확인
git pull         # 최신 코드 동기화
```

### Step 2. 로컬 코드 작성 및 커밋
코드를 수정/작성한 후 `dev` 브랜치에 커밋과 푸시를 진행합니다.
```bash
git add .
git commit -m "feat: [기능 설명] 추가"
git push
```

---

## 5. GitHub에서 `dev` -> `main` 머지 (배포 프로세스)

`dev` 브랜치에 코드가 쌓이고 기능 테스트가 완료되면 서버(운영 환경)에 반영하기 위해 `main` 브랜치로 병합해야 합니다. 이 작업은 로컬 터미널이 아닌 **GitHub 웹 페이지(Pull Request)** 를 활용하는 것이 가장 안전하고 좋습니다.

1. **GitHub 저장소 웹페이지 접속**
2. 탭 메뉴에서 **[Pull requests]** 클릭
3. 녹색의 **[New pull request]** 버튼 클릭
4. **비교 브랜치 설정 (중요):**
   * **`base`: `main`** (코드가 들어갈 최종 목적지)
   * **`compare`: `dev`** (내가 추가/수정한 작업물)
5. 변경된 파일 목록(Diff)들을 최종적으로 확인합니다.
6. **[Create pull request]** 를 클릭하여 제목을 "Release: v1.1.0 배포"와 같이 작성하고 확인합니다.
7. 리뷰 후 하단의 **[Merge pull request]** 버튼을 눌러 코드를 반영합니다.

> 머지가 완료되면 GitHub의 `main` 브랜치에 새로운 기능이 모두 반영됩니다!

---

## 6. 추가 권장 보안 설정 (Branch Protection Rule)
`main` 브랜치에 실수로 직접 코드를 `git push` 해버리는 휴먼 에러를 막기 위해 GitHub에서 브랜치 보호를 설정해 두는 것을 강력히 권장합니다.

1. GitHub 레포지토리의 **Settings** ➔ **Branches** 이동
2. **Add branch protection rule** 클릭
3. `Branch name pattern`에 **`main`** 입력
4. **"Require a pull request before merging"** 체크 (강제 PR 머지만 허용)
5. Save changes 저장

이제 로컬 터미널에서 실수로 `git push origin main`을 하더라도 권한 에러가 발생하며 서버의 코드를 안전하게 보호할 수 있습니다.
