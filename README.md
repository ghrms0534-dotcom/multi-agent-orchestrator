# MAOS

MAOS는 **Multi-Agent Autonomous Runtime**입니다.

Python, FastAPI, React, 로컬 LLM 인프라를 기반으로 구축된 자율형 Multi-Agent Runtime 플랫폼입니다. MAOS는 사용자 요청을 Planner, Role Agent, Tool 실행, Validation, Memory, Observability 계층으로 전달하며, 로컬 우선 실행과 추적 가능한 동작 흐름을 목표로 합니다.

이 프로젝트는 단순한 챗봇 래퍼가 아니라, 실제 운영형 Agent Runtime 구조를 직접 설계하고 구현하기 위한 AI Engineering 포트폴리오 프로젝트입니다.

---

## 시스템 개요

MAOS는 사용자 요청을 분석하고, 필요한 작업 유형을 분류한 뒤, 적절한 Agent와 Tool을 선택하여 실행합니다. 실행 결과는 Validation Layer를 통해 검증되며, 최종 응답은 사용자에게 전달됩니다.

현재 Runtime은 일반 대화, 코드 분석 및 수정, Git/GitHub 워크플로우, Kubernetes 및 Docker 작업, 파일 조회, 시스템 상태 확인, Persistent Memory, Trace, 안전한 Validation 명령 실행을 지원합니다.

```text
사용자 요청
   |
   v
Frontend Dashboard (React)
   |
   v
FastAPI Backend
   |
   v
Planner Agent
   |
   v
Role Agent Selection
(Chat / Coding / Git / GitHub / Kubernetes / Docker / File / System)
   |
   v
Tool Selection
   |
   v
Tool Execution
   |
   v
Validation Layer
   |
   v
Final Response
```

---

## 시스템 구성 요소

MAOS는 로컬 Backend Runtime, React Dashboard, Tool Adapter, 로컬 모델 실행 환경으로 구성됩니다.

### 기술 스택

| 영역 | 기술 | 역할 |
| --- | --- | --- |
| Backend | Python, FastAPI, PydanticAI, SQLite | API 서버, Agent Runtime, Memory, Trace 저장 |
| Frontend | React, TypeScript, Vite | Chat Dashboard, Agent Activity, Tool 상태 표시 |
| Infrastructure | Docker, Kubernetes, Ollama Local LLM | 로컬 실행 환경, 컨테이너 오케스트레이션, 모델 실행 |
| Tooling | Git, GitHub API, kubectl, Docker CLI | Guarded Tool 기반 외부 작업 실행 |

---

## Agent Architecture

MAOS는 역할 기반 Agent 구조를 사용합니다. 각 요청은 가능한 가장 작은 실행 경로로 라우팅되며, Agent는 공통 Tool Registry, Validation Layer, Memory Store, Trace Pipeline을 재사용합니다.

각 Agent는 독립적인 책임을 가지며, Tool 실행과 최종 응답 생성을 분리하여 디버깅과 확장이 쉬운 구조를 유지합니다.

### Planner Agent

사용자 의도를 분류하고, 대상 Agent와 필요한 Tool을 결정합니다.

예시:

- 일반 대화
- 코드 관련 요청
- Git 또는 GitHub 워크플로우
- Kubernetes 또는 Docker 작업
- 파일 또는 시스템 조회

---

### DevOps Agent

Git, Kubernetes, Docker 등 인프라 성격의 작업을 묶어 처리하는 역할 그룹입니다.

현재 범위:

- 로컬 Git 상태 조회
- 컨테이너 런타임 상태 확인
- Kubernetes 리소스 작업
- Guarded Tool 기반 인프라 명령 라우팅

---

### Coding Agent

코드 분석, 코드 변환, 자동 수정, Validation, Self-Correction, 안전한 코드 작업 흐름을 담당합니다.

현재 기능:

- 코드 설명, 리뷰, 변환
- 프로젝트 파일 안전 조회
- 프로젝트 코드 검색
- 명시적 요청이 있을 때만 파일 수정
- Allowlist 기반 Validation 명령 실행
- Validation 실패 시 최대 1회 Self-Correction
- Diff 및 Validation 결과 요약 반환

---

### Validator Agent

Tool 실행 결과가 최종 응답에 사용 가능한 상태인지 검증합니다.

현재 검증 항목:

- 빈 결과
- Error 문구
- 명령 실패
- Timeout
- 해석 불가능한 출력 형식

---

### Git Agent

로컬 Git 저장소 조회와 안전하게 제한된 Git 작업을 담당합니다.

현재 기능:

- Git status
- Git branch
- 수정 파일 기준 Git diff
- 쓰기 및 파괴적 작업에 대한 Guard 처리

---

### GitHub Agent

기존 GitHub API 구조와 인증 흐름을 재사용하여 GitHub 저장소 관련 작업을 처리합니다.

현재 기능:

- Repository 조회
- Pull Request 생성
- Issue 생성
- Release 생성
- Branch 생성
- 명시적으로 허용된 경우 GitHub API 기반 commit/push

---

### API Agent

등록된 API Tool을 통해 외부 API 성격의 작업을 처리합니다.

현재 범위:

- GitHub Repository API 호출
- Network Utility API 호출
- Shared Registry 기반 API Tool 라우팅

---

### Kubernetes Agent

Kubernetes 리소스 조회와 제한된 운영 명령을 담당합니다.

현재 기능:

- Pod 상태 조회
- Logs 조회
- Exec
- Apply
- Delete
- Scale
- Rollout restart

---

### Docker Agent

Docker 런타임 조회와 제한된 컨테이너 작업을 담당합니다.

현재 기능:

- Docker container status
- Build
- Run
- Logs
- Stop
- Remove
- Compose up/down

---

### File Agent

프로젝트 파일 및 디렉터리 조회를 담당합니다.

현재 기능:

- 프로젝트 파일 목록 조회
- 디렉터리 조회
- 프로젝트 Root 내부로 제한된 안전한 파일 접근

---

### System Agent

로컬 Runtime 및 시스템 상태 요청을 처리합니다.

현재 기능:

- 시스템 상태 조회
- Public IP 조회
- SQLite Memory 상태 조회

---

## Tool Execution System

Agent는 임의 작업을 직접 실행하지 않습니다. Tool Registry에 등록된 Tool을 선택하고 실행한 뒤, 결과를 검증하여 통제된 응답으로 반환합니다.

현재 Tool 그룹:

| 그룹 | 예시 |
| --- | --- |
| Coding Tools | `list_directory`, `read_file`, `search_code`, `write_file`, `replace_in_file`, `run_validation` |
| Git Tools | `get_git_status`, `get_git_branch`, `get_git_diff` |
| GitHub Tools | repository lookup, pull request, issue, release, branch |
| Kubernetes Tools | pods, logs, exec, apply, delete, scale, rollout restart |
| Docker Tools | ps, build, run, logs, stop, rm, compose up/down |
| System Tools | memory status, system status, public IP |

Safety rules:

- 명시적 수정 의도가 없으면 파일 수정 금지
- 프로젝트 Root 밖 접근 차단
- 민감 파일명 차단
- Validation 명령은 Allowlist 기반으로만 실행
- Git commit/push 자동 실행 금지
- Coding Self-Correction은 최대 1회만 수행

---

## Frontend Dashboard

Frontend는 단순 Chat UI가 아니라 Agent Runtime 상태를 확인하기 위한 Dashboard입니다.

현재 Dashboard 기능:

- 실시간 Chat
- Agent Activity 표시
- Tool 상태 표시
- Execution Trace 확인
- Session History
- Settings
- Memory 제어

---

## 프로젝트 구조

```text
backend/
  app/
    agent/
      planner.py
      model_router.py
      role_agents.py
      runner.py
    agents/
      devops_agent.py
      api_agent.py
      orchestrator_agent.py
    tools/
      registry.py
      validation.py
      local_tools.py

frontend/
  src/
    components/
    data/
    utils/

k8s/
```

---

## Environment Setup

### Required

- Python 3.14+
- Node.js 24+
- Docker Desktop
- Kubernetes with kind
- Ollama

### Python Environment

```bash
python -m venv .venv
```

Windows 환경에서 활성화:

```powershell
.venv\Scripts\activate
```

의존성 설치:

```bash
pip install -r requirements.txt
```

---

## Run Guide

### Backend

```bash
uv run uvicorn backend.app.api.main:app --reload --host 0.0.0.0 --port 8000
```

Swagger:

```text
http://localhost:8000/docs
```

### Frontend

```bash
cd frontend
npm install
npm run dev
```

Frontend URL:

```text
http://localhost:5173
```

### Build

```bash
cd frontend
npm run build
```

---

## API

| Method | Endpoint | Description |
| --- | --- | --- |
| GET | `/health` | Backend 상태 확인 |
| GET | `/tools` | Dashboard Tool Discovery용 메타데이터 조회 |
| POST | `/chat` | Chat 요청 처리 |
| POST | `/api/chat` | Dashboard에서 사용하는 API Chat Endpoint |

---

## 핵심 목표

MAOS는 기본적인 LLM Chat Demo를 목표로 하지 않습니다. 작업 라우팅, Tool 실행, 결과 검증, Memory 유지, Trace 노출이 가능한 로컬 Multi-Agent Runtime 구현을 목표로 합니다.

프로젝트가 중점적으로 다루는 Agent Engineering 패턴:

- Planner 기반 라우팅
- 역할 기반 Agent 구조
- Tool Registry 및 Permission Check
- Memory 기반 Request Context
- Validation 및 Self-Correction
- Runtime Observability

---

## 현재 구현 상태

구현된 기능:

- Multi-Agent Architecture
- Planner Agent
- Coding Agent
- Validator Agent
- Git Agent
- GitHub Agent
- Kubernetes Agent
- Docker Agent
- File Agent
- System Agent
- Tool Registry
- Persistent SQLite Memory
- Request Trace 및 Runtime Metrics
- File Read/Write Safety Guard
- Allowlist 기반 Validation Runner
- React Dashboard
- FastAPI Backend
- Ollama Local Model Integration

---

## 개발 목적

이 프로젝트는 Agent Platform이 단순 LLM 호출을 넘어 Planning, Tool Execution, Validation, Memory, Traceability를 갖춘 구조화된 Runtime으로 확장되는 방식을 보여줍니다.

MAOS는 실용적인 Architecture, Local Execution, Safe Automation을 중심으로 설계한 개인 AI Engineering 프로젝트입니다.
