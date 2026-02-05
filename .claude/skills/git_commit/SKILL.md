---
name: git-commit-workflow
description: Git commit 시 progress.md 업데이트, task.md 최신화, git add/commit/push를 순차적으로 수행하는 워크플로우. 사용자가 "커밋해줘", "git commit", "작업 저장", "푸시해줘" 등 git commit 관련 요청을 할 때 트리거.
---

# Git Commit Workflow

Git commit 요청 시 다음 순서로 진행:

## Step 1: progress.md 업데이트

`./.claude/docs/progress.md` 파일을 열어 다음 내용 추가/수정:

### 1.1 현재 세션 작업 로깅

```markdown
## [YYYY-MM-DD HH:MM] 세션 작업 내역

### 변경된 파일
- `파일경로/파일명.확장자`: 변경 내용 요약
- `파일경로/파일명.확장자`: 변경 내용 요약

### 작업 요약
- 주요 작업 내용 1
- 주요 작업 내용 2
```

### 1.2 다음 스텝 TODO 최신화

```markdown
## 다음 스텝
- [ ] TODO 항목 1
- [ ] TODO 항목 2
- [x] 완료된 항목 (체크 표시)
```

## Step 2: task.md 최신화

`task.md` 파일의 태스크 상태 업데이트:
- 완료된 태스크 체크
- 새로운 태스크 추가 (필요시)
- 우선순위 재정렬 (필요시)

## Step 3: Git 커밋 및 푸시

현재 세션에서 작업한 파일들만 선택적으로 커밋:

```bash
# 1. 변경된 파일 확인
git status

# 2. 현재 세션 작업 파일만 add (progress.md, task.md 포함)
git add <작업한_파일들> progress.md task.md

# 3. 커밋 메시지 작성
git commit -m "feat: 작업 내용 요약

- 상세 변경 내역 1
- 상세 변경 내역 2"

# 4. 푸시
git push origin <현재_브랜치>
```

## 커밋 메시지 컨벤션

```
<type>: <subject>

- 상세 내역 1
- 상세 내역 2
```

**Type 종류:**
- `feat`: 새로운 기능
- `fix`: 버그 수정
- `docs`: 문서 수정
- `style`: 코드 포맷팅
- `refactor`: 리팩토링
- `test`: 테스트 코드
- `chore`: 빌드, 설정 변경