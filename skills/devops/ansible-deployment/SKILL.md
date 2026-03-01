---
name: ansible-deployment
description: Automates server configuration and multi-server deployments. Use when writing Ansible playbooks, setting up SSH auth, or checking deployment diffs.
---

# Ansible Deployment Automation

서버 설정 파일 배포 시 안전하고 효율적인 Ansible 패턴을 제공합니다.

**핵심 철학**:
- 배포 전 diff 확인 필수 (사용자가 변경 내용 검토)
- Ansible 네이티브 기능 우선 (shell 스크립트 최소화)
- 단일 비밀번호 입력으로 전체 워크플로우 완료
- 색상 diff로 가독성 향상

## Instructions

### 워크플로우: 요청 분석 및 리소스 선택

사용자 요청을 분석하여 필요한 리소스만 선택적으로 로드합니다.

#### 1. 키워드 매칭

사용자 요청의 키워드를 분석하여 필요한 리소스를 판단:

**Diff 체크** (`resources/01-diff-checking.md`)
- "diff", "차이", "변경 사항"
- "배포 전 확인", "dry-run"
- "file_diff_checker"
- "copy 모듈"

**SSH 인증** (`resources/02-ssh-auth.md`)
- "SSH", "인증", "비밀번호"
- "vars_prompt"
- "ProxyJump", "Gateway"
- "VPN", "접속"

**Copy 모듈 활용** (`resources/03-copy-module.md`)
- "copy 모듈", "copy module"
- "check_mode", "diff"
- "파일 동기화", "파일 배포"

**Playbook 패턴** (`resources/04-playbook-patterns.md`)
- "playbook", "role"
- "배포", "deploy"
- "서비스 reload", "handler"
- "사용자 확인", "pause"

**검증 및 테스트** (`resources/05-testing-validation.md`)
- "syntax check", "문법 검사"
- "dry-run", "테스트"
- "lint", "검증"
- "justfile"

#### 2. 리소스 로딩 전략

**단일 키워드 감지**
- User: "배포 전 diff 확인하고 싶어"
- → Read resources/01-diff-checking.md
- → Read resources/03-copy-module.md

**복합 요청**
- User: "Ansible로 설정 파일 안전하게 배포하는 방법"
- → Read resources/01-diff-checking.md (Diff)
- → Read resources/02-ssh-auth.md (SSH)
- → Read resources/03-copy-module.md (Copy)
- → Read resources/04-playbook-patterns.md (Playbook)

**불명확한 요청**
- User: "Ansible 배포 설정"
- → REFERENCE.md 확인하여 사용자에게 선택지 제시
- → 선택에 따라 리소스 로드

#### 3. 리소스 적용

1. **현재 프로젝트 구조 파악**
   - ansible 디렉토리 위치 확인
   - 기존 playbook, role 확인

2. **리소스 Read**
   - 필요한 리소스만 Read
   - 다른 리소스 참조 필요 시 추가 Read

3. **패턴 적용**
   - Read한 리소스의 패턴을 프로젝트에 적용
   - 기존 설정이 있으면 수정, 없으면 생성
   - 사용자에게 변경 사항 확인

4. **검증**
   - ansible-playbook --syntax-check
   - 가능하면 실행하여 동작 확인
   - 문제 발생 시 사용자에게 보고

### 예시

#### 예시 1: Diff 체크 Role 생성

User: "배포 전에 변경 내용을 diff로 보고 싶어"

1. 키워드 매칭: "diff", "배포 전" → Diff Checking
2. Read resources/01-diff-checking.md
3. Read resources/03-copy-module.md
4. ansible/roles/file_diff_checker/ 생성
5. copy 모듈 check_mode + diff 패턴 적용
6. 사용자에게 사용법 안내

#### 예시 2: SSH 인증 개선

User: "Ansible 실행할 때 비밀번호 여러 번 입력하기 귀찮아"

1. 키워드 매칭: "비밀번호" → SSH Auth
2. Read resources/02-ssh-auth.md
3. vars_prompt 설정 추가
4. ansible_password + ansible_become_password 설정
5. 테스트 실행 확인

#### 예시 3: 안전한 배포 Playbook

User: "서버 설정 파일을 안전하게 배포하는 playbook 만들어줘"

1. 키워드 매칭: "배포", "안전" → 전체
2. Read resources/01-diff-checking.md
3. Read resources/02-ssh-auth.md
4. Read resources/03-copy-module.md
5. Read resources/04-playbook-patterns.md
6. Playbook 생성 (diff → 확인 → 배포 → reload)
7. 실행 테스트

## 중요 원칙

1. **토큰 효율**: 필요한 리소스만 Read
2. **Ansible 네이티브 우선**: shell 스크립트 대신 Ansible 모듈 사용
3. **사용자 확인**: 배포 전 diff 확인 필수
4. **실행 검증**: 가능하면 항상 실행하여 동작 확인
5. **범용성**: 프로젝트 특정 내용 없이 패턴만 제공

## Technical Details

상세한 설정 및 예제는 각 리소스 파일 참조:
- `REFERENCE.md`: 리소스 전체 개요
- `resources/01-diff-checking.md`: 배포 전 diff 확인 패턴
- `resources/02-ssh-auth.md`: SSH 인증 및 vars_prompt
- `resources/03-copy-module.md`: copy 모듈 활용법
- `resources/04-playbook-patterns.md`: 안전한 배포 playbook 구조
- `resources/05-testing-validation.md`: 검증 및 테스트 워크플로우
