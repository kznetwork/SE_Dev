# Git 실무 명령어 가이드

이 문서는 Git의 초기 설정부터 저장소 생성, 브랜치 관리, 원격 저장소 연결, Pull Request, Fork 동기화까지 자주 사용하는 작업을 실습 중심으로 정리한 가이드입니다.

> [!IMPORTANT]
> 명령어의 `<사용자명>`, `<저장소명>`, `<브랜치명>`처럼 꺾쇠로 표시된 부분은 실제 값으로 바꾸어 사용하세요.

---

## 목차

1. [Git 설정 확인](#1-git-설정-확인)
2. [사용자 정보 설정](#2-사용자-정보-설정)
3. [기본 브랜치를 main으로 설정](#3-기본-브랜치를-main으로-설정)
4. [새 로컬 저장소 만들기](#4-새-로컬-저장소-만들기)
5. [원격 저장소 연결과 변경](#5-원격-저장소-연결과-변경)
6. [저장소 복제하기](#6-저장소-복제하기)
7. [변경 사항 저장하기](#7-변경-사항-저장하기)
8. [브랜치 관리](#8-브랜치-관리)
9. [원격 브랜치를 로컬로 가져오기](#9-원격-브랜치를-로컬로-가져오기)
10. [모든 원격 브랜치를 로컬에 만들기](#10-모든-원격-브랜치를-로컬에-만들기)
11. [기준 브랜치에서 새 작업 브랜치 만들기](#11-기준-브랜치에서-새-작업-브랜치-만들기)
12. [Pull Request를 사용하는 작업 흐름](#12-pull-request를-사용하는-작업-흐름)
13. [로컬에서 직접 병합하는 작업 흐름](#13-로컬에서-직접-병합하는-작업-흐름)
14. [Fork 저장소 작업 흐름](#14-fork-저장소-작업-흐름)
15. [특정 파일만 스테이징하거나 제외하기](#15-특정-파일만-스테이징하거나-제외하기)
16. [브랜치 생성 위치 확인](#16-브랜치-생성-위치-확인)
17. [자주 발생하는 문제](#17-자주-발생하는-문제)
18. [빠른 명령어 요약](#18-빠른-명령어-요약)

---

## 1. Git 설정 확인

현재 적용 중인 Git 설정 전체를 확인합니다.

```bash
git config --list
```

설정이 어느 파일에서 왔는지 함께 확인하려면 다음 명령이 더 유용합니다.

```bash
git config --list --show-origin
```

특정 설정만 확인할 수도 있습니다.

```bash
git config --global user.name
git config --global user.email
git config --global init.defaultBranch
```

### 설정 범위

Git 설정에는 대표적으로 세 가지 범위가 있습니다.

| 범위 | 옵션 | 적용 대상 |
|---|---|---|
| 시스템 | `--system` | 컴퓨터의 모든 사용자 |
| 전역 | `--global` | 현재 운영체제 사용자 |
| 로컬 | `--local` | 현재 Git 저장소 |

현재 저장소에서만 다른 사용자 정보를 사용하려면 `--global` 없이 설정합니다.

```bash
git config user.name "Project User"
git config user.email "project-user@example.com"
```

---

## 2. 사용자 정보 설정

커밋 작성자 이름과 이메일을 전역으로 설정합니다.

```bash
git config --global user.name "<사용자명>"
git config --global user.email "<이메일주소>"
```

예시:

```bash
git config --global user.name "Git User"
git config --global user.email "git-user@example.com"
```

설정 결과를 확인합니다.

```bash
git config --global user.name
git config --global user.email
```

> [!CAUTION]
> GitHub Personal Access Token, 비밀번호, API 키를 `git config user.password`에 저장하지 마세요.
> 이미 저장했다면 다음 명령으로 삭제하세요.
>
> ```bash
> git config --global --unset user.password
> git config --local --unset user.password
> ```
>
> 인증 정보는 Git Credential Manager나 운영체제의 보안 자격 증명 저장소를 사용하는 것이 안전합니다.

---

## 3. 기본 브랜치를 main으로 설정

새 저장소를 `git init`으로 만들 때 기본 브랜치 이름이 `main`이 되도록 설정합니다.

```bash
git config --global init.defaultBranch main
```

설정을 확인합니다.

```bash
git config --global init.defaultBranch
```

이미 생성된 저장소의 현재 브랜치 이름을 `main`으로 바꾸려면 다음 명령을 사용합니다.

```bash
git branch -M main
```

---

## 4. 새 로컬 저장소 만들기

### 4.1 기본 초기화

```bash
mkdir <프로젝트폴더>
cd <프로젝트폴더>
git init
```

예시:

```bash
mkdir sample-project
cd sample-project
git init
```

### 4.2 첫 번째 커밋 만들기

```bash
git add .
git commit -m "chore: initial commit"
```

### 4.3 전체 초기화 예시

```bash
cd /c/DEV/<프로젝트폴더>

git init
git branch -M main
git add .
git commit -m "chore: initial commit"
```

> [!WARNING]
> 기존 저장소에서 `.git` 폴더를 삭제하면 커밋 기록, 브랜치 정보, 원격 저장소 설정이 모두 사라집니다.
>
> ```bash
> rm -rf .git
> ```
>
> 저장소를 완전히 다시 초기화할 목적이 아니라면 실행하지 마세요.

---

## 5. 원격 저장소 연결과 변경

### 5.1 원격 저장소 추가

```bash
git remote add origin https://github.com/<사용자명>/<저장소명>.git
```

### 5.2 원격 저장소 확인

```bash
git remote -v
```

상세 정보를 확인하려면 다음을 사용합니다.

```bash
git remote show origin
```

### 5.3 원격 저장소 URL 변경

```bash
git remote set-url origin https://github.com/<사용자명>/<새저장소명>.git
```

### 5.4 원격 저장소 삭제 후 다시 연결

```bash
git remote remove origin
git remote add origin https://github.com/<사용자명>/<저장소명>.git
```

### 5.5 최초 푸시

```bash
git push -u origin main
```

`-u` 또는 `--set-upstream` 옵션은 로컬 브랜치와 원격 브랜치의 추적 관계를 설정합니다. 이후부터는 다음처럼 간단히 실행할 수 있습니다.

```bash
git push
git pull
```

---

## 6. 저장소 복제하기

### 6.1 저장소 전체 복제

```bash
git clone https://github.com/<사용자명>/<저장소명>.git
```

복제할 로컬 폴더 이름을 지정할 수도 있습니다.

```bash
git clone https://github.com/<사용자명>/<저장소명>.git <로컬폴더명>
```

### 6.2 특정 브랜치만 복제

```bash
git clone --branch <브랜치명> --single-branch <원격저장소URL>
```

예시:

```bash
git clone --branch release --single-branch \
  https://github.com/<사용자명>/<저장소명>.git
```

옵션의 의미는 다음과 같습니다.

- `--branch <브랜치명>`: 복제 후 지정한 브랜치를 체크아웃합니다.
- `--single-branch`: 지정한 브랜치의 기록만 가져옵니다.

```bash
$ git fetch --all
for branch in $(git branch -r | grep -v HEAD); do
  git checkout -b ${branch#origin/} ${branch}
done

```

### 6.3 `--branch`만 사용

```bash
git clone --branch <브랜치명> <원격저장소URL>
```

지정한 브랜치가 체크아웃되지만 다른 브랜치의 원격 정보도 함께 가져옵니다.

### 6.4 `--single-branch`만 사용

```bash
git clone --single-branch <원격저장소URL>
```

원격 저장소의 기본 브랜치만 복제합니다.

---

## 7. 변경 사항 저장하기

Git의 기본 작업 흐름은 다음과 같습니다.

```text
작업 디렉터리 → 스테이징 영역 → 로컬 저장소 → 원격 저장소
```

### 7.1 상태 확인

```bash
git status
```

### 7.2 변경 사항 스테이징

현재 폴더 아래의 변경 사항을 추가합니다.

```bash
git add .
```

삭제된 파일까지 포함한 모든 변경 사항을 추가합니다.

```bash
git add -A
```

### 7.3 커밋

```bash
git commit -m "feat: 로그인 기능 추가"
```

커밋 메시지 예시:

```text
feat: 새로운 기능 추가
fix: 오류 수정
docs: 문서 수정
style: 코드 동작과 무관한 형식 수정
refactor: 기능 변경 없는 코드 구조 개선
test: 테스트 추가 또는 수정
chore: 빌드·설정·도구 관련 변경
```

### 7.4 원격 저장소로 푸시

추적 관계가 이미 설정되어 있다면:

```bash
git push
```

처음 푸시하는 브랜치라면:

```bash
git push --set-upstream origin <브랜치명>
```

---

## 8. 브랜치 관리

### 8.1 브랜치 목록 확인

```bash
git branch
```

현재 선택된 브랜치 앞에는 `*`가 표시됩니다.

원격 브랜치까지 함께 확인하려면:

```bash
git branch -a
```

### 8.2 새 브랜치 생성

브랜치만 만들고 이동하지 않습니다.

```bash
git branch <브랜치명>
```

예시:

```bash
git branch feature/login
```

### 8.3 브랜치 생성 후 바로 이동

기존 방식:

```bash
git checkout -b <브랜치명>
```

권장 방식:

```bash
git switch -c <브랜치명>
```

예시:

```bash
git switch -c feature/login
```

### 8.4 기존 브랜치로 이동

기존 방식:

```bash
git checkout <브랜치명>
```

권장 방식:

```bash
git switch <브랜치명>
```

### 8.5 브랜치 삭제

병합된 로컬 브랜치를 안전하게 삭제합니다.

```bash
git branch -d <브랜치명>
```

병합 여부와 관계없이 강제로 삭제합니다.

```bash
git branch -D <브랜치명>
```

원격 브랜치를 삭제합니다.

```bash
git push origin --delete <브랜치명>
```

> [!CAUTION]
> `-D`는 병합되지 않은 작업도 삭제할 수 있으므로 신중하게 사용하세요.

---

## 9. 원격 브랜치를 로컬로 가져오기

먼저 원격 저장소의 최신 브랜치 정보를 가져옵니다.

```bash
git fetch origin
```

원격 브랜치를 기준으로 로컬 브랜치를 만들고 이동합니다.

기존 방식:

```bash
git checkout -b <로컬브랜치명> origin/<원격브랜치명>
```

예시:

```bash
git checkout -b develop origin/develop
```

권장 방식:

```bash
git switch --track -c develop origin/develop
```

Git이 자동으로 같은 이름의 원격 브랜치를 찾을 수 있다면 다음처럼 간단히 실행할 수도 있습니다.

```bash
git switch develop
```

---

## 10. 모든 원격 브랜치를 로컬에 만들기

원격 브랜치 정보를 갱신합니다.

```bash
git fetch --all
```

Git Bash에서 모든 원격 브랜치를 로컬 추적 브랜치로 만들려면 다음 스크립트를 사용할 수 있습니다.

```bash
for branch in $(git branch -r | grep -v 'HEAD' | grep -v 'origin/main'); do
  git branch --track "${branch#origin/}" "$branch" 2>/dev/null || true
done
```

한 줄 버전:

```bash
for branch in $(git branch -r | grep -v 'HEAD' | grep -v 'origin/main'); do git branch --track "${branch#origin/}" "$branch" 2>/dev/null || true; done
```

결과를 확인합니다.

```bash
git branch
```

> [!NOTE]
> 기존 로컬 브랜치와 이름이 같으면 `a branch named '...' already exists` 오류가 발생할 수 있습니다. 이미 존재하는 브랜치는 다시 만들 필요가 없습니다.

---

## 11. 기준 브랜치에서 새 작업 브랜치 만들기

특정 기준 브랜치의 최신 상태에서 새 브랜치를 만들 수 있습니다.

```bash
git checkout -b <새브랜치명> <기준브랜치명>
```

권장 방식:

```bash
git switch -c <새브랜치명> <기준브랜치명>
```

예를 들어 `develop`을 기준으로 `fix/ui-bug` 브랜치를 만들려면:

```bash
git fetch origin
git switch develop
git pull origin develop
git switch -c fix/ui-bug
git push --set-upstream origin fix/ui-bug
```

더 짧게 작성하면:

```bash
git fetch origin
git switch -c fix/ui-bug origin/develop
git push -u origin fix/ui-bug
```

이 작업은 다음 순서로 진행됩니다.

1. 원격 저장소의 최신 상태를 가져옵니다.
2. `develop`을 기준으로 `fix/ui-bug`를 만듭니다.
3. 새 브랜치로 이동합니다.
4. 원격 저장소에 같은 이름의 브랜치를 만들고 추적 관계를 설정합니다.

---

## 12. Pull Request를 사용하는 작업 흐름

팀 작업에서는 기능 브랜치를 `develop`이나 `main`에 직접 병합하기보다 Pull Request를 사용하는 것이 일반적입니다.

예시 상황:

```text
현재 브랜치: feature/aidos-search
병합 대상: develop
다음 작업: feature/aidos-decision-desk
```

### 12.1 현재 작업 커밋

```bash
git add -A
git commit -m "feat: AIDOS Search 구현 완료"
```

### 12.2 기능 브랜치 푸시

```bash
git push -u origin feature/aidos-search
```

### 12.3 GitHub에서 Pull Request 생성

GitHub 저장소에서 다음 방향으로 Pull Request를 만듭니다.

```text
feature/aidos-search → develop
```

리뷰어를 지정하고 검토가 끝나면 병합합니다.

### 12.4 병합된 develop 갱신

```bash
git switch develop
git pull origin develop
```

### 12.5 다음 기능 브랜치 생성

```bash
git switch -c feature/aidos-decision-desk
git push -u origin feature/aidos-decision-desk
```

### 12.6 이전 기능 브랜치 삭제

로컬 브랜치 삭제:

```bash
git branch -d feature/aidos-search
```

원격 브랜치 삭제:

```bash
git push origin --delete feature/aidos-search
```

> [!NOTE]
> GitHub에서 PR을 병합했다면 로컬에서 같은 기능 브랜치를 다시 `develop`에 병합하지 않습니다. `develop`을 pull하여 병합 결과를 가져오면 됩니다.

---

## 13. 로컬에서 직접 병합하는 작업 흐름

혼자 작업하거나 Pull Request가 필요 없는 저장소에서는 로컬에서 직접 병합할 수 있습니다.

### 13.1 기능 브랜치 변경 사항 커밋

```bash
git add -A
git commit -m "feat: AIDOS Search 구현 완료"
```

### 13.2 기능 브랜치 백업용 푸시

```bash
git push -u origin feature/aidos-search
```

### 13.3 develop 최신화

```bash
git switch develop
git pull origin develop
```

### 13.4 기능 브랜치 병합

```bash
git merge feature/aidos-search
```

### 13.5 develop 푸시

```bash
git push origin develop
```

### 13.6 다음 작업 브랜치 생성

```bash
git switch -c feature/aidos-decision-desk
git push -u origin feature/aidos-decision-desk
```

---

## 14. Fork 저장소 작업 흐름

Fork 기반 작업에서는 일반적으로 다음과 같이 원격 저장소를 구분합니다.

```text
origin   = 내 Fork 저장소
upstream = 원본 저장소
```

예시:

```text
origin   https://github.com/<내계정>/<저장소명>.git
upstream https://github.com/<원본계정>/<저장소명>.git
```

### 14.1 Fork 저장소 복제

```bash
git clone https://github.com/<내계정>/<저장소명>.git
cd <저장소명>
```

특정 브랜치만 복제하려면:

```bash
git clone -b release --single-branch \
  https://github.com/<내계정>/<저장소명>.git
```

### 14.2 원격 저장소 확인

```bash
git remote -v
```

### 14.3 원본 저장소 등록

```bash
git remote add upstream \
  https://github.com/<원본계정>/<저장소명>.git
```

다시 확인합니다.

```bash
git remote -v
```

### 14.4 upstream 변경 사항 가져오기

```bash
git fetch upstream
```

특정 브랜치만 가져오려면:

```bash
git fetch upstream release
```

### 14.5 로컬 브랜치를 upstream과 동기화

```bash
git switch release
git merge upstream/release
```

히스토리를 직선으로 유지하고 싶다면 rebase를 사용할 수 있습니다.

```bash
git switch release
git rebase upstream/release
```

### 14.6 동기화 결과를 내 Fork에 푸시

```bash
git push origin release
```

### 14.7 간단한 pull 방식

다음 명령은 upstream의 `release`를 가져와 현재 브랜치에 병합합니다.

```bash
git pull upstream release
```

작업 전 현재 브랜치가 `release`인지 반드시 확인하세요.

```bash
git branch --show-current
```

### 14.8 Fork 작업 브랜치 권장 흐름

```bash
git fetch upstream
git switch release
git merge upstream/release
git push origin release

git switch -c feature/my-work
git push -u origin feature/my-work
```

작업을 완료하면 내 Fork의 기능 브랜치에서 원본 저장소의 대상 브랜치로 Pull Request를 만듭니다.

---

## 15. 특정 파일만 스테이징하거나 제외하기

### 15.1 특정 파일만 추가

```bash
git add src/index.js
git commit -m "fix: index.js 수정"
```

### 15.2 특정 폴더 추가

```bash
git add src/
git commit -m "feat: src 디렉터리 변경 사항 추가"
```

### 15.3 추적 중인 파일의 수정·삭제만 추가

새 파일은 제외하고 기존 추적 파일의 수정과 삭제만 스테이징합니다.

```bash
git add -u src/
git commit -m "fix: src의 기존 파일 업데이트"
```

### 15.4 특정 확장자 제외

Git Bash에서는 pathspec을 사용해 특정 파일을 제외할 수 있습니다.

```bash
git add src/ ':!src/*.zip'
```

전체 프로젝트를 추가하면서 모든 ZIP 파일을 제외하려면:

```bash
git add . ':!*.zip' ':!**/*.zip'
```

### 15.5 `.gitignore` 사용

지속적으로 제외할 파일은 `.gitignore`에 등록하는 것이 좋습니다.

```bash
echo "src/frontend.zip" >> .gitignore
git add .gitignore
git commit -m "chore: frontend.zip 제외"
```

예시 `.gitignore`:

```gitignore
# 압축 파일
*.zip

# 의존성
node_modules/

# 환경 변수
.env
.env.*

# 빌드 결과
dist/
build/
```

이미 Git이 추적하는 파일을 `.gitignore`에 추가해도 자동으로 추적이 중단되지는 않습니다. 캐시에서 제거해야 합니다.

```bash
git rm --cached src/frontend.zip
git commit -m "chore: frontend.zip 추적 중단"
```

---

## 16. 브랜치 생성 위치 확인

브랜치의 변경 이력과 생성에 가까운 기록을 확인하려면 reflog를 사용합니다.

```bash
git reflog show <브랜치명>
```

예시:

```bash
git reflog show feature/login
```

현재 브랜치의 reflog:

```bash
git reflog
```

브랜치 간 공통 조상 커밋을 확인하려면:

```bash
git merge-base <브랜치A> <브랜치B>
```

예시:

```bash
git merge-base feature/login develop
```

브랜치의 그래프를 시각적으로 확인하려면:

```bash
git log --oneline --graph --decorate --all
```

---

## 17. 자주 발생하는 문제

### 17.1 `fatal: a branch named 'main' already exists`

같은 이름의 로컬 브랜치가 이미 존재한다는 의미입니다.

브랜치 목록을 확인합니다.

```bash
git branch
```

기존 브랜치로 이동합니다.

```bash
git switch main
```

기존 브랜치가 불필요하고 안전하게 삭제할 수 있을 때만 삭제 후 다시 만듭니다.

```bash
git branch -d main
```

### 17.2 `src refspec <브랜치명> does not match any`

대표적인 원인은 다음과 같습니다.

- 해당 브랜치가 존재하지 않음
- 아직 커밋이 하나도 없음
- 브랜치 이름 오타

확인 명령:

```bash
git branch
git status
git log --oneline -5
```

최초 커밋이 없다면:

```bash
git add .
git commit -m "chore: initial commit"
git push -u origin main
```

### 17.3 `remote origin already exists`

이미 `origin`이 등록되어 있습니다.

```bash
git remote -v
```

URL만 바꾸려면:

```bash
git remote set-url origin <새URL>
```

삭제 후 다시 등록하려면:

```bash
git remote remove origin
git remote add origin <새URL>
```

### 17.4 `Your branch has no upstream branch`

현재 로컬 브랜치와 연결된 원격 브랜치가 없습니다.

```bash
git push --set-upstream origin <현재브랜치명>
```

현재 브랜치 이름을 자동으로 사용하려면 Git Bash에서:

```bash
git push -u origin HEAD
```

### 17.5 `Already up to date`

병합하려는 브랜치의 변경 사항이 이미 현재 브랜치에 반영되어 있다는 의미입니다. 오류가 아닙니다.

### 17.6 `Everything up-to-date`

로컬과 원격 저장소 사이에 새로 푸시할 커밋이 없다는 의미입니다.

### 17.7 푸시가 거부되는 경우

원격 브랜치가 로컬보다 앞서 있을 수 있습니다.

```bash
git pull --rebase origin <브랜치명>
git push origin <브랜치명>
```

충돌이 발생하면 파일을 수정한 후:

```bash
git add <충돌해결파일>
git rebase --continue
git push origin <브랜치명>
```

---

## 18. 빠른 명령어 요약

### 저장소 시작

```bash
git init
git branch -M main
git add .
git commit -m "chore: initial commit"
git remote add origin <원격저장소URL>
git push -u origin main
```

### 새 작업 브랜치

```bash
git switch develop
git pull origin develop
git switch -c feature/my-feature
git push -u origin feature/my-feature
```

### 변경 사항 저장

```bash
git status
git add .
git commit -m "feat: 기능 설명"
git push
```

### 원격 브랜치 가져오기

```bash
git fetch origin
git switch --track -c develop origin/develop
```

### 기능 브랜치 병합

```bash
git switch develop
git pull origin develop
git merge feature/my-feature
git push origin develop
```

### Fork 동기화

```bash
git fetch upstream
git switch main
git merge upstream/main
git push origin main
```

### 브랜치 삭제

```bash
git branch -d feature/my-feature
git push origin --delete feature/my-feature
```

### 전체 이력 그래프

```bash
git log --oneline --graph --decorate --all
```

---

## 권장 실습 순서

처음 Git을 연습한다면 다음 순서가 좋습니다.

1. 사용자 정보와 기본 브랜치를 설정합니다.
2. 빈 프로젝트에서 `git init`을 실행합니다.
3. 파일을 생성하고 `add → commit` 흐름을 연습합니다.
4. GitHub 빈 저장소를 만들고 `remote add → push`를 연습합니다.
5. 기능 브랜치를 만들고 푸시합니다.
6. GitHub에서 Pull Request를 생성하고 병합합니다.
7. 로컬의 `main` 또는 `develop`을 pull하여 최신화합니다.
8. Fork 저장소에서 `origin`과 `upstream`의 차이를 연습합니다.

---

## 핵심 개념 정리

| 개념 | 설명 |
|---|---|
| Working Tree | 실제 파일을 수정하는 작업 공간 |
| Staging Area | 다음 커밋에 포함할 변경 사항을 준비하는 공간 |
| Commit | 변경 사항의 저장 지점 |
| Branch | 독립적인 작업 흐름 |
| `origin` | 일반적으로 현재 복제한 원격 저장소 |
| `upstream` | Fork 작업에서 원본 저장소 |
| Fetch | 원격 변경 사항을 가져오지만 자동 병합하지 않음 |
| Pull | 원격 변경 사항을 가져와 현재 브랜치에 반영 |
| Push | 로컬 커밋을 원격 저장소에 전송 |
| Merge | 두 브랜치의 변경 이력을 합침 |
| Rebase | 현재 브랜치의 커밋 기반을 다른 브랜치 끝으로 이동 |
| Pull Request | 원격 저장소에서 변경 사항 검토와 병합을 요청하는 절차 |

---

이 문서의 명령어는 Git Bash 기준으로 작성했으며, 대부분 macOS와 Linux 터미널에서도 동일하게 사용할 수 있습니다. PowerShell에서는 줄 연결 문자와 셸 스크립트 문법이 다를 수 있으므로 여러 줄 Bash 스크립트는 Git Bash에서 실행하는 것이 안전합니다.
