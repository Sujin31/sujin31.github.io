---
layout: post
title: "Git과 GitHub 소개"
date: 2025-03-15
categories:
  - Git
  - Version Control
tags:
  - Git
  - GitHub
  - 버전 관리
  - 협업
---

## Git이란?

**Git**은 **분산형 버전 관리 시스템**입니다. 코드를 버전별로 관리하여 개발자가 코드 변경 사항을 추적하고 기록할 수 있게 도와줍니다.

### 주요 기능
- **버전 관리**: 코드 변경 사항을 추적하고 기록할 수 있습니다.
- **분산 시스템**: 로컬에서도 저장소를 관리할 수 있습니다.
- **브랜치**: 독립적인 작업 공간을 만들어 코드를 변경할 수 있습니다.
- **머지(Merge)**: 여러 브랜치를 합쳐 변경 사항을 반영할 수 있습니다.

---

## GitHub이란?

**GitHub**는 Git 저장소를 **온라인에서 관리**할 수 있는 **클라우드 기반 플랫폼**입니다. GitHub에서는 코드 저장 및 협업을 할 수 있습니다.

### 주요 기능
- **원격 저장소**: 코드 저장 및 공유가 가능합니다.
- **협업 도구**: Pull Request(PR), 코드 리뷰, Issues 등을 통해 팀원과 협업할 수 있습니다.
- **CI/CD 지원**: GitHub Actions 등을 활용하여 자동화 구축이 가능합니다.

---

## Git의 기본 용어와 개념

### Git 영역

- **Working directory (작업 영역)**: 실제 소스코드 작업이 이루어지는 로컬 PC의 디렉토리 파일입니다.
- **Staging area (스테이징 영역)**: 작업한 내용 중 git이 관리하는 로컬 저장소로 저장할 대상을 선택하는 임시 영역입니다. (`git add`로 추가)
- **Repository (저장소)**: Git 프로젝트를 관리하는 공간으로, **로컬 저장소**와 **원격 저장소**로 구분됩니다.
  
  - **Local (로컬 저장소)**: 개인 PC에서 관리하는 Git 저장소입니다. Git의 버전 관리는 이 로컬 저장소에서 이루어집니다.
  - **Remote (원격 저장소)**: GitHub와 같은 원격 서버에서 관리되는 저장소로, 협업을 위해 사용됩니다.

### 브랜치와 마스터

- **Master (마스터)**: 각 저장소의 기본이 되는 코드입니다. 기본적으로 `main`이라는 이름으로 존재합니다.
- **Branch (브랜치)**: 독립적인 작업을 위한 개별적인 코드 흐름입니다. 새로운 기능을 개발할 때 새 브랜치를 만들어서 작업합니다.

---

## Git 시작하기

### 1. 사용자 설정

```bash
git config --global user.name "이름"
git config --global user.email "이메일"
```

**확인**:
```bash
git config user.name
git config user.email
```

### 2. 클론 생성

```bash
git clone 주소
```

### 3. 스테이징 영역에 파일 올리기 (ADD)

```bash
git add .
git add /파일
```

**확인**:
```bash
git status
```

### 4. 로컬 저장소에 커밋 (COMMIT)

```bash
git commit -m "메시지"
```

**확인**:
```bash
git log
```

### 5. 원격 저장소에 푸시 (PUSH)

```bash
git push origin [브랜치명]
```

### 6. 원격 저장소에서 내려받기 (PULL)

```bash
git pull
git pull origin main
```

---

## Git의 모든 변경사항을 초기화하고 로컬과 원격 동기화하는 방법

### 1. 새로 클론하기

```bash
git clone 주소
```

### 2. 기존 폴더 유지: 원격 저장소 상태로 `--hard reset` & GC

```bash
# 1) 원격에서 최신 상태 받아오기
git fetch --all

# 2) 로컬 main(또는 리셋하고자 하는 브랜치)으로 체크아웃
git checkout main

# 3) 원격 main과 동일하게 강제 재설정
git reset --hard origin/main
```

### 3. Reflog & Git GC 정리

```bash
# reflog에서 더 이상 참조되지 않는 모든 커밋을 만료 처리
git reflog expire --expire=now --all

# 불필요한 이력 및 객체를 실제로 정리 (GC)
git gc --prune=now --aggressive
```

---

## Revert / Reset / Restore

### 1. 커밋 되돌리기 (되돌렸다는 커밋 로그 남기기)

```bash
git revert HEAD
:wq
git log
```

### 2. 리셋으로 커밋 삭제하기
- **mixed**: 커밋은 지우지만 워킹디렉토리에는 남기기
- **hard**: 커밋, 워킹디렉토리 다 지우기

```bash
git reset --mixed HEAD~1
git reset --hard HEAD~1
```

### 3. Restore로 작업 되돌리기: add 되돌리기

```bash
git restore --staged 파일명
git restore 파일명
```

---

## 브랜치와 협업

### 새로운 기능을 위한 브랜치 생성 및 이동

```bash
git branch 브랜치명
git branch // 확인
git checkout 브랜치명
```

### 브랜치 이름 규칙

- **기능 개발**: `feature/*`
- **버그 수정**: `bugfix/*` (개발 브랜치), `hotfix/*` (운영 브랜치)
- **배포 준비**: `release/*`
- **긴급 수정**: `hotfix/*`
- **리팩토링, 문서 작업**: `chore/*`, `docs/*`
- **실험적 테스트**: `experiment/*`

### 기능 개발 및 변경 사항 저장

1. **원격 저장소와 로컬 저장소 동기화**

```bash
git checkout main
git pull
```

2. **기존 브랜치 제거**

```bash
git branch -d 브랜치명
```

---

## 메시지 컨벤션

### Commit 메시지 규칙

1. **커밋 타입 지정 (태그 활용)**

| 타입   | 설명                |
|--------|---------------------|
| FEAT   | 새로운 기능 추가     |
| FIX    | 버그 수정            |
| DOCS   | 문서 수정 및 추가    |
| STYLE  | 코드 스타일 변경     |
| REFACTOR | 코드 리팩토링        |
| TEST   | 테스트 코드 추가     |
| CHORE  | 빌드 설정 변경       |

2. **제목과 본문은 빈 줄로 분리**  
   제목과 본문 사이에 빈 줄을 넣어 가독성을 높입니다.
```bash
  FIX: 로그인 시 비밀번호 검증 오류 수정
  
  로그인 시 비밀번호가 공백이 입력될 경우 발생하는 문제를 수정했습니다.
  ```

3. **제목 행은 50자 이내로 제한**  
   - 커밋 메시지의 제목은 50자를 넘지 않도록 작성
   - 너무 긴 메시지는 가독성을 떨어뜨리며, 여러 개의 커밋을 확인할 때 불편함을 초래
   

4. 제목 행의 첫 글자는 대문자로 시작
  - 커밋 메시지의 제목은 항상 대문자로 시작

5. 제목 행 끝에 마침표(.) 사용 금지
6. 제목은 명령형으로 작성 
7. 본문에는 변경 내용과 이유를 작성


etc..
rebase merge? 

→ 컨벤션 차이(rebase 로그가 한줄로 깔끔하게 나ㅏ옴~~)
