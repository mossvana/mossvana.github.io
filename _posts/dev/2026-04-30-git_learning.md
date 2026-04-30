---
title: "learngitbranching으로 배운 Git 명령어 정리"
date: 2026-04-30 
categories: [dev]
---

> [learngitbranching.js.org](https://learngitbranching.js.org/?locale=ko)에서 학습한 내용을 정리한 레퍼런스입니다.

---

## 1. 기본 명령어

### git commit
현재 변경사항을 저장소에 기록한다.

```bash
git commit
git commit --amend   # 가장 최근 커밋을 수정 (메시지 변경 or 파일 추가)
```

### git branch
브랜치를 생성하거나 확인한다.

```bash
git branch <브랜치명>         # 브랜치 생성
git branch -f <브랜치명> <위치>  # 브랜치를 특정 커밋으로 강제 이동
git branch -u origin/<브랜치> <로컬브랜치>  # 원격 추적 설정
```

### git merge
두 브랜치를 합친다. merge 커밋이 생성된다.

```bash
git merge <브랜치명>
```

### git rebase
커밋들을 다른 브랜치 위로 옮겨 붙인다. 히스토리가 선형으로 정리된다.

```bash
git rebase <브랜치명>
git rebase -i <위치>   # 인터랙티브 리베이스 (커밋 순서 변경, 삭제, 합치기 등)
```

---

## 2. HEAD와 상대 참조

### HEAD
현재 체크아웃된 커밋을 가리키는 포인터.

```bash
git checkout <커밋해시>   # HEAD를 특정 커밋으로 분리(detach)
```

### 상대 참조

```bash
git checkout HEAD^    # 한 단계 위 부모 커밋으로 이동
git checkout HEAD^^   # 두 단계 위로 이동
git checkout HEAD~3   # 세 단계 위로 이동

# 브랜치에도 동일하게 사용 가능
git checkout main^
git branch -f main HEAD~3   # main 브랜치를 3단계 위로 강제 이동
```

---

## 3. 작업 되돌리기

### git reset
로컬에서 커밋을 되돌린다. 히스토리 자체를 지운다.

```bash
git reset HEAD~1   # 한 커밋 이전으로 되돌리기
```

### git revert
커밋을 되돌리는 새 커밋을 만든다. 원격 저장소에 공유할 때 사용한다.

```bash
git revert HEAD   # 가장 최근 커밋을 되돌리는 새 커밋 생성
```

---

## 4. 커밋 이리저리 옮기기

### git cherry-pick
특정 커밋만 골라서 현재 브랜치에 적용한다.

```bash
git cherry-pick <커밋1> <커밋2> ...
```

### git rebase -i (인터랙티브 리베이스)
커밋 순서 변경, 특정 커밋 삭제, 커밋 합치기 등을 대화형으로 수행한다.

```bash
git rebase -i HEAD~4   # 최근 4개 커밋을 인터랙티브하게 편집
```

---

## 5. 태그와 describe

### git tag
특정 커밋에 영구적인 이름표(태그)를 붙인다.

```bash
git tag <태그명>            # 현재 HEAD에 태그
git tag <태그명> <커밋해시>  # 특정 커밋에 태그
```

### git describe
가장 가까운 태그를 기준으로 현재 위치를 설명한다.

```bash
git describe <ref>
# 출력 형식: <태그>_<커밋수>_g<해시>
# 예: v1_2_gC6 → v1 태그로부터 2커밋 위, 해시 C6
```

---

## 6. 원격 저장소 기본

### git clone
원격 저장소를 로컬로 복사한다.

```bash
git clone <url>
```

### 원격 브랜치
원격 저장소의 상태를 나타내는 읽기 전용 브랜치. `o/main` 또는 `origin/main` 형태로 표시된다.

### git fetch
원격의 변경사항을 로컬로 가져온다. 단, merge는 하지 않는다.

```bash
git fetch
git fetch origin <브랜치>
git fetch origin <source>:<destination>   # 특정 커밋/브랜치를 로컬 브랜치로 가져오기
git fetch origin :<새브랜치명>             # 로컬에 빈 브랜치 생성
```

### git pull
fetch + merge를 한 번에 수행한다.

```bash
git pull
git pull --rebase              # fetch + rebase
git pull origin <브랜치>
git pull origin <source>:<destination>   # 특정 커밋을 로컬 브랜치로 가져와서 merge
```

> **핵심:** `git pull origin bar:foo`는 `git fetch origin bar:foo` + `git merge foo`와 동일하다.
> source에 브랜치명 대신 **커밋 해시**를 쓰면 `o/브랜치`(원격 추적 브랜치)가 업데이트되지 않는다.

### git push
로컬 변경사항을 원격 저장소에 업로드한다.

```bash
git push
git push origin <브랜치>
git push origin <source>:<destination>   # 로컬의 source를 원격의 destination으로 push
git push origin :<원격브랜치명>           # 원격 브랜치 삭제
```

### git fakeTeamwork
learngitbranching에서 팀원의 작업을 시뮬레이션하는 가상 명령어.

```bash
git fakeTeamwork main 2   # 원격 main에 커밋 2개 추가
```

---

## 7. 고급 원격 저장소

### Push Main! — 원격 main에 직접 push하기

로컬 main이 원격과 diverge된 경우, rebase로 정리 후 push한다.

```bash
git fetch
git rebase o/main
git push
# 또는
git pull --rebase
git push
```

### 원격 작업과 merge하기

feature 브랜치를 main에 합치고 push하는 흐름.

```bash
git checkout -b feature
git commit
git pull origin main
git push origin feature
```

### 원격 저장소 추적 설정

로컬 브랜치가 어떤 원격 브랜치를 추적할지 설정한다.

```bash
git branch -u origin/main <브랜치명>
git checkout -b <브랜치명> origin/main   # 생성과 동시에 추적 설정
```

### git push의 인자들

source와 destination을 명시적으로 지정한다 (colon refspec).

```bash
git push origin <source>:<destination>
git push origin foo^:main    # 상대참조도 사용 가능
git push origin main:newBranch  # 원격에 없는 브랜치도 생성 가능
```

### git push 인자 — 확장판

checkout 없이도 원하는 브랜치를 push할 수 있다.

```bash
git push origin <로컬브랜치>:<원격브랜치>
```

### Fetch의 인자들

fetch도 push와 동일한 방식으로 source:destination을 지정할 수 있다. 단, 방향이 반대다 (원격 → 로컬).

```bash
git fetch origin <원격브랜치>:<로컬브랜치>
git fetch origin c3:foo    # 특정 커밋을 로컬 브랜치로 가져오기
```

### Source가 없다 — 빈 source

source를 비워두면 삭제 또는 생성이 된다.

```bash
git push origin :foo    # 원격 브랜치 foo 삭제
git fetch origin :bar   # 로컬에 빈 브랜치 bar 생성
```

### pull 인자들

pull도 fetch와 동일한 인자를 사용한다.

```bash
git pull origin foo          # = git fetch origin foo; git merge o/foo
git pull origin bar:bugFix   # = git fetch origin bar:bugFix; git merge bugFix
git pull origin c3:foo       # 커밋 해시로 지정하면 o/브랜치가 업데이트되지 않음
```

---

## 정리: fetch vs pull vs push 비교

| 명령어 | 동작 | merge 여부 |
|--------|------|-----------|
| `git fetch` | 원격 → 로컬 다운로드 | ❌ |
| `git pull` | 원격 → 로컬 다운로드 | ✅ (자동 merge) |
| `git push` | 로컬 → 원격 업로드 | ❌ |
