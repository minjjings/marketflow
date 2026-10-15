# MarketFlow AI-DLC 사용 가이드

이 문서는 Codex CLI와 프로젝트에 설치된 AI-DLC를 처음 사용하는 사람을 위한 참고서다. Skill을 전부 외울 필요는 없다. 보통은 Codex 대화창에서 `$aidlc` 뒤에 원하는 일을 설명하면 된다. 개별 Skill 목록은 특정 단계만 실행하고 싶을 때 찾아본다.

확인일: 2026-10-15. 프로젝트의 실제 설치 파일을 기준으로 작성했다. 이 문서를 작성하면서 개발 Workflow, 배포, 설정 변경은 실행하지 않았다. 아래 예시는 사용자가 나중에 입력할 예시이며, 실행 성공을 보장하는 테스트 결과가 아니다.

## 목차

1. [빠른 시작](#1-빠른-시작)
2. [핵심 용어](#2-핵심-용어)
3. [시작 순서와 기본 명령어](#3-시작-순서와-기본-명령어)
4. [초보자가 먼저 알아둘 Skill](#4-초보자가-먼저-알아둘-skill)
5. [설치된 Skill 전체 목록](#5-설치된-skill-전체-목록)
6. [Workflow 선택 기준](#6-workflow-선택-기준)
7. [Hook과 승인](#7-hook과-승인)
8. [질문과 승인 요청에 답하기](#8-질문과-승인-요청에-답하기)
9. [중단하고 다음 세션에서 이어가기](#9-중단하고-다음-세션에서-이어가기)
10. [Windows와 MacBook에서 Git으로 공유하기](#10-windows와-macbook에서-git으로-공유하기)
11. [MarketFlow 개발 예시](#11-marketflow-개발-예시)
12. [문제 해결과 근거 자료](#12-문제-해결과-근거-자료)

## 1. 빠른 시작

처음에는 다음 순서면 충분하다.

1. 터미널에서 MarketFlow 폴더로 이동하고 `codex`를 실행한다.
2. Codex 대화창에서 `$aidlc --doctor`로 설치 상태를 확인한다.
3. `$aidlc --status`로 진행 중인 작업이 있는지 확인한다.
4. 새 작업이면 `$aidlc` 뒤에 목표와 이번에 하지 않을 일을 적는다.
5. 제안받은 작업 방식과 질문을 검토하고, 문서를 읽은 뒤 승인하거나 수정을 요청한다.
6. 그만할 때는 “여기서 중단하고 다음 세션에서 이어가자”라고 요청한다.

첫 요청 예시 — **Codex 대화창**:

```text
$aidlc MarketFlow의 첫 MVP를 만들고 싶다.
미국 금융시장 데이터를 수집하고 검증한 뒤 일일 요약 파일을 생성하는 것이 목표다.
먼저 데이터 범위와 완료 기준을 질문해 줘.
외부 자동 게시와 AWS 배포는 이번 범위에서 제외해 줘.
```

이 요청은 실제 개발을 시작하는 요청이다. 설명만 듣고 싶다면 “아직 실행하지 말고 사용법만 설명해 줘”라고 범위를 분명히 적는다.

### 어디에 입력하는가

| 입력 위치 | 예시 | 의미 |
|---|---|---|
| PowerShell 또는 macOS 터미널 | `codex`, `codex --version`, `git status` | 운영체제가 실행하는 명령 |
| 실행 중인 Codex 대화창 | `$aidlc --status`, `$aidlc-feature ...` | Skill 호출 또는 AI-DLC 요청 |
| 실행 중인 Codex 대화창 | `/skills` | Skill 선택 메뉴 |
| 터미널의 AI-DLC 실행 파일 | `aidlc engine orchestrate help` | 엔진 명령; 보통 Codex가 대신 실행 |

`$aidlc`를 PowerShell에 직접 붙여넣으면 Skill이 실행되지 않는다. PowerShell의 `$`는 변수 문법이다. macOS 셸도 `$`를 해석하므로 동일하게 Codex 대화창에 입력한다.

이 설치본의 일부 내부 설명에는 `/aidlc` 표기가 남아 있다. Codex에서는 onboarding에 명시된 `$aidlc` 또는 `/skills` 선택 방식을 우선 사용한다. `aidlc --help`와 `$aidlc --help`도 입력 위치와 처리 경로가 다르다.

## 2. 핵심 용어

| 용어 | 쉽게 말하면 | MarketFlow에서의 예 |
|---|---|---|
| Codex | 파일을 읽고 수정하며 명령을 실행하는 AI 개발 도구 | 요구사항을 읽고 수집 코드를 작성·검증 |
| Codex CLI | Codex를 터미널에서 사용하는 프로그램 | 프로젝트 폴더에서 `codex` 실행 |
| AI-DLC | AI와 개발할 때 질문·설계·구현·검증·승인을 연결하는 방법과 도구 | 무엇을 만들지 정한 뒤 단계적으로 개발 |
| Workflow | 한 작업을 완료하기 위한 단계의 흐름 | 요구사항 → 설계 → 구현 → 테스트 |
| Agent | 특정 전문 역할로 일하는 AI | 요구사항은 product, 설계는 architect, 구현은 developer |
| Skill | 반복 작업의 방법을 담은 사용 설명과 실행 절차 | `aidlc-bugfix`가 버그 수정 흐름을 시작 |
| Harness | AI가 파일·명령·권한·대화를 다루게 하는 실행 환경 | 여기서는 Codex CLI; 다른 설치에는 다른 환경도 존재 |
| Scope | 작업 목적에 맞는 Workflow 구성 | `mvp`, `feature`, `bugfix` |
| Phase | 여러 단계를 묶은 큰 구간 | Inception, Construction |
| Stage | 한 가지 목적의 개별 단계 | `requirements-analysis`, `build-and-test` |
| Intent | 하나의 개발 목표와 그 작업 기록 | “일일 시장 데이터 수집 MVP” |
| Unit | 따로 설계·구현·검증할 수 있도록 나눈 개발 단위 | 수집기, 검증기, 요약 생성기 |
| Bolt | 실행·전달 계획에서 묶어 진행하는 작업 구간 | 작은 기능 묶음의 개발 순서 |
| Hook | 특정 이벤트에 자동으로 실행되는 처리 | 세션 시작 시 이전 작업 확인 |
| Sensor | 산출물이나 코드의 조건을 검사하는 장치 | 필수 문서 항목, 타입 검사 확인 |

Agent와 Skill은 일대일 관계가 아니다. 하나의 Skill이 여러 전문 역할을 연결할 수 있고, 같은 Agent가 여러 Stage에서 일할 수 있다. Agent를 사용자가 매번 직접 지정할 필요는 없다.

전체 단계의 큰 구분은 다음과 같다. Scope에 따라 일부가 생략되므로 모두를 매번 거치는 것은 아니다.

| Phase | 하는 일 |
|---|---|
| Initialization | 작업 기록 생성, 프로젝트 탐지, 초기 상태 준비 |
| Ideation | 목표·타당성·범위 확인 |
| Inception | 요구사항·사용자 스토리·도메인·실행 계획 정리 |
| Construction | 상세 설계·구현·테스트·CI 구성 |
| Operation | 배포·환경·관측·운영·개선 |

## 3. 시작 순서와 기본 명령어

### 3.1 프로젝트 폴더에서 실행

Windows PowerShell:

```powershell
Set-Location 'C:\Users\mk\IdeaProjects\marketflow'
codex --version
codex
```

MacBook 터미널 — 경로는 실제 클론 위치로 바꾼다:

```bash
cd ~/Projects/marketflow
codex --version
codex
```

설치된 onboarding은 Codex CLI 최소 버전을 `0.145.0`으로 명시한다. 현재 사용 중인 실행 파일 버전은 위 명령으로 확인한다. 이 문서는 해당 기기에 설치된 버전을 실행 측정한 결과가 아니다. Codex 기본 실행은 [OpenAI의 Codex CLI 안내](https://learn.chatgpt.com/docs/codex/cli)에서도 확인할 수 있다.

### 3.2 설치·상태 확인

다음은 모두 **Codex 대화창 입력**이다.

| 요청 | 용도 |
|---|---|
| `$aidlc --help` | 설치본이 지원하는 사용법 확인 |
| `$aidlc --doctor` | 런타임·설정·설치 문제 진단 |
| `$aidlc --status` | 현재 Scope, 단계, 승인 상태, Guard Policy 확인 |
| `$aidlc intent` | 작업 기록 목록 확인 |
| `$aidlc intent <name>` | 목록에 나온 기존 작업 선택 |
| `$aidlc` | 선택된 작업 이어가기 |
| `$aidlc --resume` | 선택된 작업 재개 |
| `$aidlc compose "작업 설명"` | 작업에 맞는 단계 구성을 제안받기 |

`<name>` 같은 꺾쇠괄호는 자리표시자다. 그대로 입력하지 말고 실제 목록에 표시된 이름으로 바꾼다.

### 3.3 새 작업 시작

```text
$aidlc --scope mvp 미국 시장 일일 데이터 수집·검증 MVP를 만들고 싶다.
```

또는 같은 목적의 바로가기를 사용한다.

```text
$aidlc-mvp 미국 시장 일일 데이터 수집·검증 MVP를 만들고 싶다.
```

Scope를 고르기 어렵다면 `$aidlc` 뒤에 목표를 설명한다. 제안받은 구성과 설명을 검토해서 선택하면 된다. 이미 진행 중인 작업이 있으면 기존 작업의 연장인지 별도 작업인지 질문할 수 있다.

`$aidlc-init --scope mvp ...`는 작업 기록을 명시적으로 만드는 선택지다. 이 경로는 초기화 후 멈추므로 `$aidlc`로 이어간다. 일반적인 시작에는 필수가 아니다.

### 3.4 작업 파일 위치

```text
marketflow/
  AGENTS.md                         프로젝트 공통 지침과 한국어 규칙
  aidlc.settings.json               공유 AI-DLC 설정
  .agents/skills/                    프로젝트에 설치된 Skill
  .codex/                           Codex용 설정·Agent·Hook·단계 정의
  aidlc/
    active-space                    이 클론에서 선택한 공간: Git 제외
    spaces/default/
      memory/                       조직·팀·프로젝트 규칙
      knowledge/                    참고 문서와 문서 카탈로그
      intents/
        active-intent               이 클론에서 선택한 작업: Git 제외
        <YYMMDD>-<label>/            요구사항·설계·감사 기록 등
  docs/AI-DLC-사용-가이드.md          이 문서
```

실제 업무 산출물은 선택한 Intent 아래에 생성된다. 애플리케이션 코드는 프로젝트 루트나 지정된 애플리케이션 위치에 작성된다. `aidlc-state.md`나 감사 기록을 직접 고쳐 진행 단계를 바꾸지 않는다.

현재 `aidlc.settings.json`에는 `balanced` 모델 프리셋이 있다. 이것만으로 실제 모델 공급자·인증·모델이 모두 정해지는 것은 아니다. onboarding에 따르면 사용자 Codex 설정도 상속한다. 한국어 기본 규칙은 `AGENTS.md`에 있다.

## 4. 초보자가 먼저 알아둘 Skill

다음 8개부터 알아두면 된다. 나머지 단계는 보통 Workflow가 연결한다.

| 우선 Skill | 먼저 알아둘 이유 | 예시 |
|---|---|---|
| `aidlc` | 시작·상태·재개를 한곳에서 처리 | `$aidlc --status` |
| `aidlc-compose` | 작업 방식이 막막할 때 구성안을 제안받음 | `$aidlc-compose MarketFlow 수집 MVP에 맞는 계획을 제안해 줘` |
| `aidlc-mvp` | 핵심 결과를 먼저 만드는 작업 | `$aidlc-mvp 일일 시장 데이터 수집과 검증을 구현하고 싶다` |
| `aidlc-feature` | 운영까지 고려하는 새 기능 | `$aidlc-feature 검증 실패 알림 기능을 추가하고 싶다` |
| `aidlc-bugfix` | 재현되는 특정 오류 수정 | `$aidlc-bugfix 동일 날짜 데이터가 중복 저장되는 오류를 고쳐 줘` |
| `aidlc-knowledge` | 내가 가진 참고 자료를 작업 근거로 사용 | `$aidlc-knowledge list` |
| `aidlc-replay` | 이전에 한 일과 결정 이유를 다시 확인 | `$aidlc-replay` |
| `aidlc-outcomes-pack` | 완료한 작업의 인계 문서 생성 | `$aidlc-outcomes-pack` |

아직 코드가 없다면 `bugfix`로 시작하지 않는다. 기능을 추가하는 예시도 해당 코드가 먼저 존재한다는 전제다.

## 5. 설치된 Skill 전체 목록

확인한 대상은 이 저장소의 `.agents/skills/*/SKILL.md` **42개**다. 사용자 전역 Skill이나 다른 플러그인의 Skill은 이 목록에 포함하지 않는다. 각 이름의 링크는 실제 설명 파일로 연결한다.

### 5.1 작업 시작·구성·Scope 바로가기: 8개

| Skill | 한국어 설명 | 사용 시점 | Codex 대화창 호출 예시 |
|---|---|---|---|
| [aidlc](../.agents/skills/aidlc/SKILL.md) | 전체 작업 진행을 조정하는 기본 진입점 | 시작·상태·재개·작업 전환 | `$aidlc --status` |
| [aidlc-init](../.agents/skills/aidlc-init/SKILL.md) | 작업 기록과 초기 상태 준비 | Scope를 정하고 기록부터 만들 때 | `$aidlc-init --scope mvp 시장 데이터 수집 MVP` |
| [aidlc-compose](../.agents/skills/aidlc-compose/SKILL.md) | 작업에 맞는 단계 조합 제안 | 기존 Scope가 적절한지 모르거나 구성을 바꿀 때 | `$aidlc-compose 시장 데이터 수집 MVP 계획을 제안해 줘` |
| [aidlc-mvp](../.agents/skills/aidlc-mvp/SKILL.md) | 핵심 결과를 먼저 만드는 흐름 | 첫 제품·기능의 핵심 검증 | `$aidlc-mvp 일일 시장 데이터 수집과 검증` |
| [aidlc-feature](../.agents/skills/aidlc-feature/SKILL.md) | 새 기능의 전체 개발 흐름 | 설계부터 운영까지 고려할 기능 | `$aidlc-feature 수집 실패 알림 추가` |
| [aidlc-bugfix](../.agents/skills/aidlc-bugfix/SKILL.md) | 특정 오류를 고치는 흐름 | 기존 코드에서 재현 가능한 버그 | `$aidlc-bugfix 휴장일에 수집기가 실패하는 오류 수정` |
| [aidlc-security-patch](../.agents/skills/aidlc-security-patch/SKILL.md) | 보안 취약점 대응 흐름 | 취약한 의존성·코드 보완 | `$aidlc-security-patch 확인된 의존성 취약점 수정` |
| [aidlc-express](../.agents/skills/aidlc-express/SKILL.md) | 설계·리뷰 절차를 줄인 가장 가벼운 흐름 | 요구사항이 명확한 작은 작업 | `$aidlc-express 요약 파일명 형식 변경` |

### 5.2 단계 단독 실행: 30개

아래 Skill은 해당 Stage를 `--single` 방식으로 실행한다. **주 Workflow의 Current Stage를 다음으로 넘기지 않는다.** 그러나 실행 자체가 읽기 전용이라는 뜻은 아니다. 산출물을 생성하고, 단계에 따라 코드·파이프라인·클라우드 환경을 변경할 수 있다. 필요한 선행 문서나 기존 작업 맥락이 없으면 질문하거나 진행이 막힐 수 있다.

예를 들어 `$aidlc-domain-design`은 `$aidlc --stage domain-design --single`의 바로가기다. 반면 `$aidlc --stage domain-design`은 주 작업의 단계를 지정하는 요청이므로 목적이 다르다. 처음에는 주 Workflow를 따라가고, 특정 단계를 따로 수행해야 할 이유가 있을 때만 아래를 사용한다.

| Skill | 한국어 설명 | 사용 시점 | 호출 예시 |
|---|---|---|---|
| [aidlc-intent-capture](../.agents/skills/aidlc-intent-capture/SKILL.md) | 해결할 문제와 목표 정리 | 아이디어를 작업 목표로 구체화 | `$aidlc-intent-capture` |
| [aidlc-market-research](../.agents/skills/aidlc-market-research/SKILL.md) | 시장·대안·사용자 맥락 조사 | 개발 필요성과 기존 대안을 조사 | `$aidlc-market-research` |
| [aidlc-feasibility](../.agents/skills/aidlc-feasibility/SKILL.md) | 기술·비용·제약의 실현 가능성 검토 | 데이터 접근·예산·운영 제약 판단 | `$aidlc-feasibility` |
| [aidlc-scope-definition](../.agents/skills/aidlc-scope-definition/SKILL.md) | 포함할 일과 제외할 일 확정 | MVP 범위와 완료 조건 결정 | `$aidlc-scope-definition` |
| [aidlc-rough-mockups](../.agents/skills/aidlc-rough-mockups/SKILL.md) | 초기 화면·사용 흐름 초안 | 화면 방향을 빠르게 검토 | `$aidlc-rough-mockups` |
| [aidlc-team-formation](../.agents/skills/aidlc-team-formation/SKILL.md) | 역할·담당·협업 방식 구성 | 팀 단위 수행을 준비 | `$aidlc-team-formation` |
| [aidlc-approval-handoff](../.agents/skills/aidlc-approval-handoff/SKILL.md) | 기획 결과 승인과 다음 구간 인계 | 기획을 개발 준비로 넘김 | `$aidlc-approval-handoff` |
| [aidlc-practices-discovery](../.agents/skills/aidlc-practices-discovery/SKILL.md) | 기존 개발·테스트·배포 관행 확인 | 프로젝트 작업 방식을 합의 | `$aidlc-practices-discovery` |
| [aidlc-reverse-engineering](../.agents/skills/aidlc-reverse-engineering/SKILL.md) | 기존 코드와 구조 분석 | 이미 있는 구현을 이해 | `$aidlc-reverse-engineering` |
| [aidlc-requirements-analysis](../.agents/skills/aidlc-requirements-analysis/SKILL.md) | 요구사항과 수용 조건 정리 | 무엇을 구현하고 검증할지 명확화 | `$aidlc-requirements-analysis` |
| [aidlc-user-stories](../.agents/skills/aidlc-user-stories/SKILL.md) | 사용자 관점의 기능·조건 정리 | 사용자 가치와 동작을 연결 | `$aidlc-user-stories` |
| [aidlc-domain-design](../.agents/skills/aidlc-domain-design/SKILL.md) | 핵심 개념과 업무 규칙 설계 | 데이터·기능의 개념 모델 정리 | `$aidlc-domain-design` |
| [aidlc-units-generation](../.agents/skills/aidlc-units-generation/SKILL.md) | 작업 단위와 의존 관계 분해 | 구현할 부분과 선후 관계 결정 | `$aidlc-units-generation` |
| [aidlc-contract-design](../.agents/skills/aidlc-contract-design/SKILL.md) | 모듈·서비스 간 입출력 계약 설계 | API·데이터 전달 형식 합의 | `$aidlc-contract-design` |
| [aidlc-refined-mockups](../.agents/skills/aidlc-refined-mockups/SKILL.md) | 상세 화면·상호작용 구체화 | 화면 구현 전 동작을 확인 | `$aidlc-refined-mockups` |
| [aidlc-delivery-planning](../.agents/skills/aidlc-delivery-planning/SKILL.md) | 구현·전달 순서 계획 | 작업 단위를 실행 순서로 구성 | `$aidlc-delivery-planning` |
| [aidlc-functional-design](../.agents/skills/aidlc-functional-design/SKILL.md) | 기능 내부 동작과 흐름 설계 | 코드 작성 전 상세 로직 정리 | `$aidlc-functional-design` |
| [aidlc-nfr-requirements](../.agents/skills/aidlc-nfr-requirements/SKILL.md) | 성능·보안·신뢰성 요구 정의 | 기능 외 품질 목표를 수치화 | `$aidlc-nfr-requirements` |
| [aidlc-nfr-design](../.agents/skills/aidlc-nfr-design/SKILL.md) | 품질 목표를 충족할 구조 설계 | 재시도·인증·확장 등 설계 | `$aidlc-nfr-design` |
| [aidlc-infrastructure-design](../.agents/skills/aidlc-infrastructure-design/SKILL.md) | 실행 환경과 인프라 구조 설계 | AWS 자원·네트워크 구성 계획 | `$aidlc-infrastructure-design` |
| [aidlc-code-generation](../.agents/skills/aidlc-code-generation/SKILL.md) | 구현 계획과 실제 코드 작성 | 준비된 설계를 구현 | `$aidlc-code-generation` |
| [aidlc-build-and-test](../.agents/skills/aidlc-build-and-test/SKILL.md) | 빌드·테스트 및 결과 확인 | 구현이 요구사항을 충족하는지 검증 | `$aidlc-build-and-test` |
| [aidlc-ci-pipeline](../.agents/skills/aidlc-ci-pipeline/SKILL.md) | 자동 빌드·테스트 경로 구성 | 변경마다 검증을 자동화 | `$aidlc-ci-pipeline` |
| [aidlc-deployment-pipeline](../.agents/skills/aidlc-deployment-pipeline/SKILL.md) | 배포 흐름과 승인 경로 구성 | 배포 절차 자동화 준비 | `$aidlc-deployment-pipeline` |
| [aidlc-environment-provisioning](../.agents/skills/aidlc-environment-provisioning/SKILL.md) | 필요한 실행 환경 준비 | 승인한 인프라를 실제 준비 | `$aidlc-environment-provisioning` |
| [aidlc-deployment-execution](../.agents/skills/aidlc-deployment-execution/SKILL.md) | 릴리스·배포 수행 | 준비된 결과를 대상 환경에 반영 | `$aidlc-deployment-execution` |
| [aidlc-observability-setup](../.agents/skills/aidlc-observability-setup/SKILL.md) | 로그·메트릭·알림 구성 | 실패와 상태를 관찰할 수 있게 함 | `$aidlc-observability-setup` |
| [aidlc-performance-validation](../.agents/skills/aidlc-performance-validation/SKILL.md) | 성능 목표 충족 여부 검증 | 처리 시간·부하 목표 확인 | `$aidlc-performance-validation` |
| [aidlc-incident-response](../.agents/skills/aidlc-incident-response/SKILL.md) | 장애 조사와 복구 대응 | 실행 중인 시스템의 장애 대응 | `$aidlc-incident-response` |
| [aidlc-feedback-optimization](../.agents/skills/aidlc-feedback-optimization/SKILL.md) | 운영 결과 기반 개선 | 사용·운영 데이터를 보고 개선 | `$aidlc-feedback-optimization` |

### 5.3 지식·기록·인계: 4개

| Skill | 한국어 설명 | 사용 시점 | 호출 예시 |
|---|---|---|---|
| [aidlc-knowledge](../.agents/skills/aidlc-knowledge/SKILL.md) | 참고 문서 등록·조회·동기화 | 보유 자료를 AI-DLC 근거로 활용 | `$aidlc-knowledge list` |
| [aidlc-session-cost](../.agents/skills/aidlc-session-cost/SKILL.md) | 시간·단계·검사·학습 기록 집계 표시 | 작업에 사용한 흐름과 규모 확인 | `$aidlc-session-cost` |
| [aidlc-replay](../.agents/skills/aidlc-replay/SKILL.md) | 진행 이력과 결정 이유를 대화에 표시 | 이전 작업을 복습하거나 인계 전 확인 | `$aidlc-replay` |
| [aidlc-outcomes-pack](../.agents/skills/aidlc-outcomes-pack/SKILL.md) | 완료 작업의 인계 문서 생성 | 작업 완료 후 결과·운영법 정리 | `$aidlc-outcomes-pack` |

`session-cost`는 실제 요금 청구액이나 토큰 비용 추정기가 아니다. 런타임 요약에 존재하는 통계만 표시한다. `replay`는 파일을 만들지 않는다. `outcomes-pack`은 Workflow 상태를 바꾸지 않지만, 완료된 Workflow를 대상으로 루트에 `OUTCOMES.md`를 쓴다. Skill의 `read-only` 분류만 보고 파일도 전혀 쓰지 않는다고 해석하지 않는다.

`knowledge`는 Workflow 단계를 넘기거나 승인하지 않지만 문서 카탈로그와 감사 기록을 변경한다. 예시:

```text
$aidlc-knowledge onboard aidlc/spaces/default/knowledge/documents/data-policy.md
$aidlc-knowledge list
$aidlc-knowledge show <id>
$aidlc-knowledge sync
```

파일 경로는 실제 존재하는 자료를 사용한다. 원본 자료는 `knowledge/documents/`, 도구가 관리하는 카탈로그는 `knowledge/documentkb/`에 둔다. 카탈로그를 손으로 편집하지 않는다. `index.json`만 잃은 경우 `sync`로 재구성할 수 있지만, `documentkb/` 전체를 지우면 기존 문서 식별자까지 잃는다. 문서 내용은 참고 데이터이며, 그 안의 지시문이 프로젝트 규칙을 대체하지 않는다.

## 6. Workflow 선택 기준

이 설치에는 Scope 정의가 11개 있다. 전용 Skill 바로가기가 없는 Scope도 `$aidlc --scope <name>`으로 선택할 수 있다. `compose`는 고정 Scope 이름이 아니라 맞춤 계획을 제안받는 기능이다.

| Scope | 적합한 작업 | 설치된 기본 특성 | 예시 |
|---|---|---|---|
| `classic` | 일반적인 기존 방식의 개발 | 기본 Scope, Inception·Construction 중심 | `$aidlc --scope classic 수집 기능 개발` |
| `mvp` | 핵심 동작을 먼저 검증 | 기획 일부와 Operation 생략 | `$aidlc --scope mvp 수집·검증 MVP` |
| `feature` | 새 기능 전체 개발 | 기획부터 운영까지, Standard 깊이 | `$aidlc --scope feature 알림 기능 추가` |
| `bugfix` | 특정 오류 수정 | 기존 코드 전제, Minimal 깊이 | `$aidlc --scope bugfix 중복 저장 오류 수정` |
| `security-patch` | 보안 문제 보완 | 기존 코드 전제, Minimal 깊이 | `$aidlc --scope security-patch 확인된 취약점 수정` |
| `express` | 명확하고 작은 변경 | 설계·리뷰를 줄이고 일부 검사·승인 생략 | `$aidlc --scope express 출력 파일명 변경` |
| `poc` | 가능성 확인용 실험 | Minimal 깊이, 제품 완성보다 실험 | `$aidlc --scope poc 데이터 API 수집 가능성 검증` |
| `refactor` | 외부 동작을 유지하는 내부 정리 | 기존 코드 전제, Minimal 깊이 | `$aidlc --scope refactor 검증 로직 중복 제거` |
| `infra` | 인프라·배포 구성 | 인프라와 운영 측면 중심 | `$aidlc --scope infra 승인된 AWS 실행 환경 구성` |
| `enterprise` | 엄격한 검토·추적이 필요한 작업 | 전체 단계, Comprehensive 깊이, strict 기본값 | `$aidlc --scope enterprise 감사 추적이 필요한 기능 개발` |
| `workshop` | 학습·진행자가 있는 실습 | 필수 체크포인트, Standard 깊이 | `$aidlc --scope workshop 데이터 수집 실습` |

Scope 이름은 작업 내용으로 고른다. 짧은 명령을 쓰고 싶다는 이유만으로 `express`를 선택하지 않는다. 특히 실제 외부 게시나 클라우드 변경이 포함되면 그 범위를 요청에 명시한다.

승인·검사의 주요 기본값은 다음과 같다. 기존 작업의 개별 설정이나 다른 규칙으로 달라질 수 있으므로 실제 값은 `$aidlc --status`에서 확인한다.

| Scope 묶음 | Guard Policy | Sensors | 요약 확인 | 코드 계획 승인 |
|---|---|---|---|---|
| `enterprise` | `strict` | on | on | on |
| `express` | `off` | off | off | off |
| `poc` | `off` | on | on | off |
| `bugfix`, `classic` | `off` | on | off | on |
| `mvp`, `feature`, `infra`, `refactor`, `security-patch`, `workshop` | `off` | on | on | on |

`Guard Policy: off`는 모든 사용자 질문과 승인이 사라진다는 뜻이 아니다. 계획 승인, 요약 확인, 사람의 응답 확인 등은 별도 설정과 절차를 갖는다. 반대로 “AI-DLC는 언제나 모든 단계에서 같은 승인을 받는다”라고 생각해도 안 된다. Construction에서는 승인된 진행 방식에 따라 자동 진행과 체크포인트 검토가 달라진다.

맞춤 계획 예시:

```text
$aidlc compose "MarketFlow는 학습용 프로젝트다. 일일 데이터 수집·검증·로컬 요약 생성에 집중하고 UI와 실제 배포는 제외하는 계획을 제안해 줘."
```

제안된 실행·생략 단계와 이유를 읽고 승인·편집·거절을 선택한다. 승인되면 해당 작업 흐름으로 이어질 수 있으므로 단순한 읽기 전용 보고서 기능으로 보지 않는다.

## 7. Hook과 승인

### 7.1 Hook은 언제 실행되는가

실제 등록은 [`.codex/hooks.json`](../.codex/hooks.json)에 있다. 다음은 등록된 동작의 요약이다. 등록이 존재한다는 사실과 현재 CLI에서 모든 동작이 정상 실행됐다는 사실은 다르다. 설치 진단은 `$aidlc --doctor`로 확인한다.

| 이벤트 | 등록된 주요 처리 | 사용자가 이해할 점 |
|---|---|---|
| `SessionStart` | `session-start` | 이전 작업과 세션 맥락 준비 |
| `UserPromptSubmit` | `record-human-turn` | 사용자의 응답을 기록하는 처리 |
| `PreToolUse` | 세션 연결, 단계 규칙 전달, 상태 전이·리뷰 범위·문서 동결·계획 승인 검사 | 명령이나 파일 작업 전에 조건 확인 |
| `PostToolUse` | 응답 기록, 감사·센서 처리, 진행 상태 동기화, 단계 그래프 갱신 | 파일·도구 작업 뒤 기록과 검증 |
| `PreCompact` | `validate-state` | 대화 맥락 압축 전 상태 검증 |
| `SubagentStop` | `log-subagent` | 하위 Agent 작업 결과 기록 |
| `Stop` | `continue-workflow` | 완료·대기·후속 진행 조건 확인 |

Hook은 사람이 매번 실행하는 Skill이 아니다. 문제가 생겨도 Hook을 직접 실행해 승인을 흉내 내거나 상태 파일을 수정하는 방식으로 해결하지 않는다.

### 7.2 세 가지 확인을 구분하기

| 확인 종류 | 확인하는 것 | 대응 |
|---|---|---|
| AI-DLC 질문·요약 확인 | 목표·요구사항·가정이 맞는가 | 사실과 제약을 답함 |
| AI-DLC 단계·계획 승인 | 문서·계획·결과를 받아들일 수 있는가 | 해당 산출물과 범위를 검토 |
| Codex 명령 실행 권한 | 파일·네트워크·샌드박스 밖 작업을 허용하는가 | 실제 명령과 경로·대상을 검토 |

요구사항 문서에 동의하는 것과 AWS 자원을 실제 생성하는 것은 다른 행동이다. 권한 요청이 나오면 어떤 명령이 어떤 대상에 영향을 주는지 확인한다. 한 번의 명령 허용과 같은 접두사의 향후 실행 허용도 구분한다.

승인 후 계획·검토 대상 파일을 편집하면 설정에 따라 재승인을 요청하거나 변경 사실을 기록하고 계속할 수 있다. `strict`, `relaxed`, `off` 및 개별 검사 설정에 영향을 받는다. 막혔다는 이유만으로 검사를 끄기보다, 표시된 원인과 수정 대상을 먼저 확인한다. 이 문서는 해당 설정을 변경하지 않는다.

## 8. 질문과 승인 요청에 답하기

### 요구사항 질문

“데이터 수집 주기는 무엇인가요?”라는 질문에는 “적당히”보다 다음처럼 답한다.

```text
미국 시장 거래일 기준 하루 한 번이면 된다.
실시간 수집은 이번 범위에서 제외한다.
API 제공자와 정확한 실행 시각은 아직 정하지 않았으니 선택지를 설명해 줘.
```

모르는 항목은 모른다고 답해도 된다. “추천 근거와 대안을 설명해 줘”라고 요청한다. 제시된 선택지가 맞지 않으면 직접 조건을 설명한다.

### 요약 확인

```text
대체로 맞지만 자동 게시는 이번 범위에서 제외해 줘.
수정한 요약을 다시 보여 줘.
```

요약에 동의했다고 아직 생성되지 않은 코드까지 검증된 것은 아니다.

### 코드 계획 승인

이 설치의 Plan Approval에는 원문을 유지하는 선택지가 있다.

| 표시 | 뜻 | 선택 전 확인 |
|---|---|---|
| `Approve Plan` | 계획 승인 | 수정 파일·동작·테스트·제외 범위가 맞는가 |
| `Request Changes` | 계획 수정 요청 | 무엇을 바꿀지 구체적으로 적었는가 |
| `I'll edit the files` | 내가 파일을 편집 | 대상 파일을 확인하고 편집 완료 후 알릴 준비가 됐는가 |

수정 요청 예시:

```text
Request Changes. 실제 API 호출은 테스트에서 제외하고 고정 fixture를 사용해 줘.
중복 실행 시 같은 날짜 데이터가 중복 저장되지 않는 테스트도 계획에 넣어 줘.
```

직접 편집을 선택했다면 편집 후 “계획 파일 편집을 마쳤으니 변경 내용을 검토하고 승인 절차를 이어가 줘”라고 알린다. 직접 편집 선택이 자동 승인은 아니다.

일반 단계 승인에서도 질문이 남아 있으면 먼저 묻는다. “승인하지만…”이라는 말은 승인과 추가 지시로 해석될 수 있다. 승인 전 수정이 필요하다면 “아직 승인하지 않겠다. 다음을 수정해 줘”라고 분명히 적는다.

### 한국어 사용 범위

이 프로젝트의 설명·질문·보고와 새 문서의 자연어 내용은 한국어가 기본이다. 다음은 영어 등 원래 표기를 유지할 수 있다.

- Skill·Agent 이름, 파일 경로, 명령어, 코드 식별자.
- `Approve Plan`, `[Answer]:`, `READY`, `NOT-READY` 같은 고정 표기.
- `## Sources`, `## Assumptions & Open Questions` 등 도구가 정확히 해석하는 제목.
- YAML 키·열거 값, 상태 필드, 원문 인용과 명령 출력.

영어 표기가 남아 있다고 전체 문서가 영어로 작성되어야 하는 것은 아니다. 주변 설명은 한국어로 읽을 수 있어야 한다.

## 9. 중단하고 다음 세션에서 이어가기

### 9.1 중단 요청

Codex 대화창:

```text
오늘은 여기서 중단하자. 현재 상태를 정리하고 다음 단계로 진행하지 마.
다음 세션에서 확인할 문서와 미해결 항목을 알려 줘.
```

승인하고 멈출 경우에는 구분해서 요청한다.

```text
이 단계 결과는 승인한다. 다음 단계는 시작하지 말고 여기서 중단해 줘.
```

AI-DLC의 정상 중단 처리는 가능한 단계 경계에서 진행을 보존한다. 작업 도중 프로세스를 강제 종료하는 것은 같은 의미가 아니다. 이미 실행 중인 외부 명령이 있으면 종료 여부와 결과를 먼저 확인한다. 대화 종료만으로 AWS 자원이나 외부 예약 작업이 정리되지는 않는다.

### 9.2 같은 기기의 새 세션

프로젝트 폴더에서 Codex를 다시 열고 입력한다.

```text
$aidlc --status
```

선택된 작업이 맞으면 다음 중 하나로 이어간다.

```text
$aidlc
```

```text
$aidlc --resume
```

설치된 복구 절차는 이전 기록을 읽어 마지막 미완료 단계에서 이어간다. 과거 대화 전체를 그대로 복원하는 것은 아니다. 중요한 결정은 문서·질문 답변·감사 기록에 남아 있어야 한다.

### 9.3 다른 기기 또는 작업 선택이 사라진 경우

`active-space`와 `active-intent`는 Git에서 제외되므로 다른 기기에서 자동 선택되지 않을 수 있다.

```text
$aidlc space default
$aidlc intent
$aidlc intent <목록에 표시된 실제 이름>
$aidlc --status
$aidlc
```

목록이 여러 개면 이어갈 목표와 기록 이름을 확인한다. 새 작업으로 잘못 초기화하지 않는다. 수정이 필요한 경우 “요구사항 분석 단계로 돌아가서 데이터 범위를 다시 검토하자”처럼 요청할 수 있다. 과거 단계를 다시 열면 후속 산출물의 유효성도 확인해야 한다.

## 10. Windows와 MacBook에서 Git으로 공유하기

### 10.1 공유되는 것과 기기에 남는 것

Git은 커밋해서 원격 저장소에 푸시한 파일을 공유한다. Codex 대화 이력·로그인·설치된 실행 파일·환경변수까지 자동 공유하지 않는다.

| 공유 대상 | 처리 |
|---|---|
| 애플리케이션 코드·테스트·문서 | 필요한 변경을 검토해 커밋 |
| `AGENTS.md`, 공유 `aidlc.settings.json`, 설치된 `.agents/`, `.codex/`의 프로젝트 파일 | 비밀정보·절대 경로 포함 여부 확인 후 버전 관리 |
| `aidlc/spaces/.../memory/`, Intent 상태·산출물·`audit/`, 지식 자료 | 이 프로젝트의 AGENTS.md에 따라 버전 관리 |
| `active-space`, `active-intent`, `.aidlc-clone-id`, `.aidlc-sessions/`, `.aidlc-engine/`, `runtime-graph.json` | `.gitignore` 기준으로 로컬 유지 |
| `aidlc.settings.local.json`, 사용자 `~/.codex/config.toml`, 로그인 정보 | 기기별 유지 |
| `.env`, 비밀키, `data/raw/`, `data/processed/`, `logs/` | 현재 `.gitignore` 기준으로 제외 |
| `.env.example` | 실제 비밀 값 없이 사용 항목만 공유 |

중요한 차이: 설치된 `aidlc-outcomes-pack`의 인계 템플릿에는 일부 감사·상태·질문 기록을 보관하거나 폐기하라는 일반 안내가 들어 있다. **MarketFlow의 `AGENTS.md`는 `aidlc/` 기록을 버전 관리하도록 명시한다.** 이 프로젝트에서는 인계 문서를 만들었다는 이유로 원래 기록을 삭제하거나 Git에서 제외하지 않는다. 서로 다른 안내를 받으면 프로젝트 정책과 차이를 확인해 달라고 요청한다.

### 10.2 Windows에서 작업을 보내기

아래는 사용자가 실행할 예시이며, 이 가이드 작성 과정에서는 실행하지 않았다. 원격 저장소가 이미 연결되어 있다는 전제다.

```powershell
git status
git branch --show-current
git remote -v
git diff
git add docs/AI-DLC-사용-가이드.md
git diff --cached
git commit -m "docs: add Korean AI-DLC guide"
git push
```

이 예시는 가이드만 커밋한다. 실제 개발 작업에서는 공유해야 할 코드와 해당 Intent 기록도 경로를 지정해 추가한다. `git diff`는 새 미추적 파일의 내용까지 보여주지 않으므로 새 파일은 직접 열거나 추가 후 `git diff --cached`로 확인한다. 처음 푸시하는 브랜치라면 실제 브랜치 이름을 확인한 뒤 `git push -u origin <branch>`를 사용한다.

설치 폴더가 `?? .agents/`처럼 보이면 아직 추적되지 않는 상태다. 다른 기기에 필요하다면 내용을 검토해 커밋해야 한다. `.gitignore`에 적었다는 사실만으로 이미 추적 중인 비밀 파일이 제거되는 것은 아니다.

### 10.3 MacBook에서 처음 받기

Git·Codex CLI·AI-DLC 런타임과 인증은 MacBook에서도 준비되어 있어야 한다. Windows 실행 파일과 사용자 홈 설정을 그대로 복사해서 사용하는 방식으로 가정하지 않는다. 설치 방법은 보유한 AI-DLC 배포본 안내와 Codex 공식 안내에 맞춘다. 이 저장소만 보고 확인되지 않은 AI-DLC 설치 명령을 만들어 실행하지 않는다.

```bash
mkdir -p ~/Projects
cd ~/Projects
git clone <repository-url> marketflow
cd marketflow
git status
codex --version
codex
```

`<repository-url>`은 실제 원격 URL로 바꾼다. 저장소 접근 인증, Codex 로그인, 필요한 API 키와 AWS 인증은 기기에서 별도로 준비한다. Windows의 `C:\Users\...` 경로를 MacBook 명령에 복사하지 않는다.

Codex 대화창에서 진단하고 기존 작업을 선택한다.

```text
$aidlc --doctor
$aidlc space default
$aidlc intent
$aidlc intent <실제 작업 이름>
$aidlc --status
```

### 10.4 두 기기를 번갈아 쓰는 습관

1. 이전 기기에서 정상 중단하고, 필요한 변경과 작업 기록을 커밋·푸시한다.
2. 다음 기기에서는 실행 중인 작업을 멈춘 상태로 작업 트리를 확인한다.
3. 변경이 없는 상태에서 같은 작업 브랜치로 이동하고 `git pull --ff-only`로 받는다.
4. Codex를 열어 Intent와 상태를 확인한 뒤 진행한다.
5. 끝나면 다시 커밋·푸시한다.

```bash
git status
git switch <branch>
git pull --ff-only
```

로컬 수정이 있으면 먼저 그 변경을 어떻게 보존할지 결정한다. `--ff-only`가 실패하면 양쪽 이력이 갈라졌을 수 있으므로 내용을 확인해 병합·리베이스 방법을 결정한다. 무조건 강제 푸시하거나 상태 기록을 덮어쓰지 않는다.

같은 Intent를 두 기기에서 동시에 수정하면 코드뿐 아니라 상태·문서에도 충돌이 날 수 있다. 초보자는 한 기기에서 멈추고 넘긴 뒤 다른 기기에서 이어가는 방식이 이해하기 쉽다. 감사 기록은 클론별 shard로 나뉘므로 임의로 하나로 합치지 않는다.

### 10.5 한글·줄바꿈·파일명

문서는 UTF-8로 저장한다. Windows PowerShell에서 한글이 깨지면 `Get-Content -Encoding UTF8`로 확인한다. 이 문서처럼 파일명에 한글이 있으면 터미널 자동완성과 따옴표를 활용할 수 있다.

대소문자만 다른 파일명은 두 운영체제에서 다르게 취급될 수 있으므로 만들지 않는다. 줄바꿈 차이로 문서 전체가 변경된 것처럼 보이면 실제 내용 변경과 구분해 검토한다. 이 가이드를 읽는 것만으로 `core.autocrlf` 같은 Git 설정을 바꿀 필요는 없다.

## 11. MarketFlow 개발 예시

현재 [README](../README.md)는 MarketFlow를 “미국 금융시장 데이터 수집·검증·분석 및 시장 요약 자동 게시 프로젝트”로 설명하고, 개발 상태는 “요구사항 정의 예정”이라고 적고 있다. 아래는 그 목표를 구체화하는 **제안 예시**다. 데이터 제공자, 언어, 저장소, 배포 방식이 이미 결정됐다고 가정하지 않는다.

### 11.1 처음 요구사항 정리부터 시작

```text
$aidlc MarketFlow의 요구사항을 정리하고 첫 MVP를 개발하고 싶다.
목표는 미국 금융시장 데이터 수집·검증·분석과 시장 요약 자동 게시다.
첫 MVP는 거래일별 데이터를 수집하고 검증 결과와 요약 파일을 로컬에 남기는 범위로 제안한다.
데이터 제공자, 수집 항목, 시간대, 저장 방식은 아직 결정하지 않았다.
먼저 선택지와 제약을 질문하고 범위를 확정해 줘.
실제 외부 게시와 AWS 배포는 이번에 하지 않는다.
```

다음 항목에 답할 준비를 하면 좋다.

| 질문 주제 | 답변에 포함할 내용 |
|---|---|
| 수집 대상 | 지수·종목·금리 등 필요한 데이터, 과거 데이터 필요 여부 |
| 제공자 제약 | 사용 가능한 계정·호출 한도·데이터 이용 조건 |
| 시간 기준 | 거래일과 실행 시간, 미국 시장 시간과 한국 날짜 처리 |
| 검증 | 결측·중복·이상값·스키마 변경 시 처리 |
| 재실행 | 같은 날짜를 다시 수집했을 때 기대 동작 |
| 요약 | 포함할 항목, 출력 형식, 근거 데이터 |
| 실패 | 재시도 범위, 실패 기록, 나중에 복구하는 방법 |
| 완료 기준 | 어떤 입력에서 어떤 결과가 나오면 완료인가 |

### 11.2 첫 MVP 구현 요청

요구사항을 검토한 뒤 요청한다.

```text
$aidlc-mvp 승인한 요구사항에 맞춰 첫 MVP를 진행해 줘.
한 거래일 데이터를 입력받아 수집·검증·요약 파일 생성까지 연결하자.
외부 서비스 의존 테스트는 fixture로 분리하는 계획을 검토해 줘.
완료 기준에는 중복 실행과 일부 데이터 누락 처리를 포함해 줘.
각 결정과 결과는 한국어로 설명해 줘.
```

이미 동일한 Intent가 진행 중이라면 새로 시작하는 대신 `$aidlc`로 이어가며 같은 조건을 설명한다. 첫 통합 동작을 보여주는 작은 결과와 검증 명령을 확인하고, 나머지 단위의 진행 방식을 결정한다.

테스트 예시는 아직 언어와 도구가 정해지지 않았으므로 `pytest`나 `npm test`를 실제 프로젝트 명령으로 단정하지 않는다. 생성된 계획과 프로젝트 설정에서 확정한 명령을 사용한다.

### 11.3 기능 추가

기존 MVP가 완성됐다는 전제:

```text
$aidlc-feature 수집 실패 시 운영자가 원인을 확인할 수 있는 알림 기능을 추가하고 싶다.
먼저 알림 대상, 채널, 재시도와 중복 알림 기준을 질문해 줘.
기존 수집·검증 동작을 유지하는 회귀 검증을 포함해 줘.
```

### 11.4 오류 수정

기존 코드에 실제 문제가 있다는 전제:

```text
$aidlc-bugfix 동일 거래일의 수집 작업을 다시 실행하면 데이터가 중복 저장된다.
재현 조건과 현재 저장 구조를 먼저 확인하고, 중복 저장을 막는 수정과 회귀 테스트를 진행해 줘.
```

관찰한 오류와 로그를 전달한다. 로그의 토큰·비밀번호·계정 비밀정보는 제거한다. 원인을 추측해서 확정하기보다 관찰된 동작과 기대 동작을 구분한다.

### 11.5 AWS 운영으로 확장

로컬 MVP와 비용·운영 요구를 검토한 뒤 별도 작업으로 진행할 수 있다.

```text
$aidlc compose "MarketFlow의 로컬 수집·검증 작업을 AWS에서 예약 실행하는 계획을 제안해 줘.
실행 빈도, 저장량, 비용 한도와 실패 대응을 먼저 확인하자.
현재 요청에서는 실제 자원을 생성하거나 배포하지 말고 구성안을 검토하자."
```

AI-DLC를 쓴다고 AWS 계정이나 배포 자원이 자동으로 필요한 것은 아니다. 실제 배포가 범위에 들어갈 때 계정·리전·권한·비용·배포 대상을 구체화한다.

## 12. 문제 해결과 근거 자료

### 12.1 자주 막히는 상황

| 상황 | 확인·대응 |
|---|---|
| Skill이 너무 많음 | `$aidlc`로 시작하고 필요할 때 5장의 목록을 찾음 |
| `$aidlc` 입력 후 셸 오류 | Codex 대화창에 입력했는지 확인 |
| `/skills`에서 보이지 않음 | 올바른 프로젝트 폴더와 `.agents/skills/<name>/SKILL.md` 존재 확인, 새 세션에서 다시 확인 |
| `aidlc` 실행 파일을 찾을 수 없음 | 기기별 런타임 설치·PATH 확인; Skill 폴더만으로 실행 파일이 설치되지는 않음 |
| 기존 작업이 보이지 않음 | Git에서 기록을 받았는지, 공간과 Intent를 선택했는지 확인 |
| 계속 질문함 | 미정 요구사항·가정·승인 대기인지 확인하고 구체적으로 답함 |
| 승인 후 다시 승인 요청 | 계획이나 검토 대상 변경 여부와 현재 Guard Policy 확인 |
| 단계 단독 실행 후 주 작업이 그대로임 | `--single`의 정상 동작; 주 Workflow는 `$aidlc`로 이어감 |
| 파일 쓰기·명령 실행 권한 요청 | 요구사항 승인과 구분해서 명령·대상 경로 확인 |
| 진단이 실패함 | 오류 원문과 `$aidlc --doctor` 결과를 근거로 복구 요청 |
| “이 단계가 완료됐다”는데 테스트를 못 찾음 | 실행한 명령·결과·산출물 경로를 요청 |
| 영어 고정 표기가 남음 | 식별자·필수 표기인지 확인하고 설명만 한국어로 요청 |

### 12.2 짧은 요청 모음

모두 Codex 대화창 입력이다.

```text
$aidlc --help
$aidlc --doctor
$aidlc --status
$aidlc intent
$aidlc
$aidlc --resume
$aidlc-replay
$aidlc-session-cost
```

읽기와 설명만 원할 때:

```text
현재 상태와 다음에 승인할 내용을 설명해 줘. 아직 파일 수정이나 다음 단계 실행은 하지 마.
```

현재 결과를 검토하고 싶을 때:

```text
이번 단계의 산출물 경로, 핵심 결정, 미해결 항목과 검증 결과를 보여 줘.
```

### 12.3 검증 범위와 근거

이 문서는 Skill의 메타데이터·실제 엔진 호출, onboarding, Scope 정의, Hook 등록과 복구 절차를 정적으로 대조했다. 개발·배포 예시는 실행하지 않았다. Hook의 실제 발동, 기기별 설치 상태, 외부 API 인증과 비용은 여기서 검증하지 않았다.

| 자료 | 확인한 내용 |
|---|---|
| [AGENTS.md](../AGENTS.md) | 한국어 규칙, 기록 경로, 재개와 Git 정책 |
| [.codex/onboarding.md](../.codex/onboarding.md) | Codex 호출 방식·버전 전제·설정 구조 |
| [aidlc.settings.json](../aidlc.settings.json) | 공유 모델 프리셋 |
| [.codex/config.toml](../.codex/config.toml) | 프로젝트 규칙 경로·질문 도구·샌드박스 설정 |
| [.agents/skills/aidlc/SKILL.md](../.agents/skills/aidlc/SKILL.md) | 기본 명령·승인·Intent 전환·재개 |
| [.agents/skills/](../.agents/skills/) | 5장의 42개 Skill 실체와 호출 절차 |
| [.codex/scopes/](../.codex/scopes/) | 11개 Scope의 목적과 기본값 |
| [.codex/hooks.json](../.codex/hooks.json) | 이벤트별 Hook 등록 |
| [.codex/aidlc-common/stages/](../.codex/aidlc-common/stages/) | 동봉된 단계별 공식 지침 |
| [복구 프로토콜](../.codex/aidlc-common/protocols/stage-protocol-recovery.md) | 기록에서 중단 작업을 이어가는 절차 |
| [조직 기본 규칙](../aidlc/spaces/default/memory/org.md) | 대화 언어·고정 표기·기본 개발 원칙 |
| [.gitignore](../.gitignore) | 클론별 커서·런타임·비밀정보·로컬 데이터 제외 |
| [README.md](../README.md) | MarketFlow 목표와 개발 상태 |

기존 공식 파일이 참조하는 `docs/guide/`, `docs/reference/`, `docs/harness-engineering/`는 확인 시점에 저장소에 없었다. 따라서 그 문서들을 읽었다고 가정하지 않았다. 일부 동봉 자료에는 과거 `aidlc-docs/` 경로 등이 남아 있으므로 현재 프로젝트 경로는 `AGENTS.md`와 onboarding을 기준으로 읽는다.

Codex 자체 사용법은 [Codex CLI](https://learn.chatgpt.com/docs/codex/cli), [명령어 참고](https://learn.chatgpt.com/docs/developer-commands?surface=cli), [Skill 안내](https://learn.chatgpt.com/docs/build-skills)를 함께 확인할 수 있다. AI-DLC의 Scope와 절차는 이 프로젝트에 설치된 파일을 우선 대조한다. 업데이트 후에는 `$aidlc --help`와 설치 파일을 다시 확인한다.
