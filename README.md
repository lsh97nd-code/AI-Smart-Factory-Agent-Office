# AI Smart Factory Agent Office

Paderborn University Bearing DataCenter의 저장된 Bearing 측정 Data를 ML로 분석해
Bearing의 정상·이상 상태를 판별합니다. ML이 Bearing에 이상이 있다고 판정하면, RAG가
관련 기술문서를 검색합니다. AI Agent가 근거가 충분한지 확인하고, 이상 판별 시 원인을
진단한 뒤 추가 점검이 필요한 항목에 대한 점검 계획을 수립합니다. 작업자에게 조치 방법을 제안하고 최종
보고서를 생성합니다.
중요한 결정은 작업자가 하도록 지원하는 근거 기반 AI Agent MVP(최소 기능 제품)입니다.

현재는 Paderborn University Bearing DataCenter에서 저장된 Bearing의 측정 결과를
불러오고, 선택한 측정 Data의 저장 신호를 해당 측정의 시간축에 따라 순차적으로
재생(Replay)합니다. 측정 Data에는 진동, 전류, 회전속도, 토크(Torque), 힘, 온도
관련 메타데이터가 포함되며, 현재 ML 입력은 진동 1개 채널과 전류 2개 채널에서
추출한 36개 특징값입니다. 현재는 실제 공장 Sensor에서 진동·전류·회전속도·토크(Torque)
등의 상태 Data를 실시간으로 수집·수신하지 않습니다.

### Service 시연(Live Demo)

[https://factory.aipath.kr](https://factory.aipath.kr)

> 심사 및 검증을 위한 외부 접속 주소입니다.

## 빠른 탐색

- [AI Native Final Project 제출 산출물](#ai-native-final-project-제출-산출물)
- [문제 정의](#문제-정의)
- [해결 방식](#해결-방식)
- [대표 입력 Data와 검증 경로](#대표-입력-data와-검증-경로)
- [Data·ML·RAG](#데이터)
- [작업자 판단(HITL)과 안전](#작업자-판단hitl과-안전)
- [시스템 구조 (Architecture)](#시스템-구조-architecture)
- [테스트 결과](#테스트-결과)
- [현재 개발 상태](#현재-개발-상태)
- [KAMP 확장 가능성 검증](#kamp-확장-가능성-검증-개념-검증poc)
- [팀 역할](#팀-역할-요약)
- [출처 및 사용 조건](#출처-및-사용-조건)

## AI Native Final Project 제출 산출물

| 번호 | 제출 산출물 | 내용 | 링크 |
|---|---|---|---|
| ① | 프로젝트 기획서 | 문제 정의, 목표, AI 활용 방식 및 추진 계획 | 별도 제출 파일 |
| ② | 기능 요구 명세서 | 기능·비기능 요구사항, 시스템 구조 (Architecture), AI 활용 명세 및 RTM | 별도 제출 파일 |
| ③ | 배포 URL | 실제 서비스 접속 및 기능 시연 | [Service 시연(Live Demo)](https://factory.aipath.kr) |
| ④ | GitHub 저장소 URL | 소스코드, 구현 구조 및 개발 결과 | [LSH97ND GitHub 저장소](https://github.com/lsh97nd-code/AI-Smart-Factory-Agent-Office) |
| ⑤ | 사용자 feedback 기록 | Google Forms 설문 분석 및 feedback 기반 UI 개선 결과 | 별도 제출 파일 |
| ⑥ | 서면 심사용 발표자료 | 프로젝트 주요 성과와 AI 기술 활용 및 사용자 검증 결과 | 별도 제출 파일 |
| ⑦ | 시연영상 | 주요 기능과 AI 작업 흐름 시연(3분 이내) | 별도 제출 파일 |

### 평가자 안내

- **서비스 접속:** [Service 시연(Live Demo)](https://factory.aipath.kr)
- **소스코드:** [LSH97ND GitHub 저장소](https://github.com/lsh97nd-code/AI-Smart-Factory-Agent-Office)
- **시스템 요약:** 선택한 저장 측정 Data의 신호를 시간축에 따라 재생하고, 진동 1개 채널과 전류 2개 채널에서 추출한 36개 특징값으로 정상·이상을 판별합니다. 이상으로 판정되면 RAG가 기술문서를 검색하고, AI Agent가 근거가 충분한지 확인하고 이상 판별 시 원인을 진단한 뒤 추가 점검이 필요한 항목에 대해 점검 계획을 세우고, 작업자에게 조치 방법을 제안한 후 최종 보고서를 생성합니다.
- **사용자 검증:** 외부에 Service를 제공하고 사용자로부터 feedback을 받았으며, feedback을 반영해 Service를 개선했습니다.
- **안전 범위:** 실제 공장 Sensor Data 실시간 수집·수신, PLC를 통한 설비 직접 제어, 설비 자동 정지, 자동 정비 실행은 포함하지 않습니다.

### AI Native Final Project(전문가 평가 요청 대상 과제물)의 평가자료 및 참고자료

- **발표 Script (Markdown, 참고자료):** [`docs/presentation_script.md`](docs/presentation_script.md)
- **시연 흐름 (개발·재현 참고자료):** [`docs/DEMO_SCRIPT.md`](docs/DEMO_SCRIPT.md)

위 자료는 AI Native Final Project(전문가 평가 요청 대상 과제물)의 평가자료와 참고자료입니다.

## 문제 정의

기존에는 제조설비에 이상이 발견되면 담당자가 측정 결과를 확인하고,
기술문서를 찾아 원인을 파악한 뒤 점검 계획을 세우고 정비 여부를 판단해야
합니다. 이 과정이 여러 작업자에게 나뉘어 전달되거나 담당자의 숙련도가
다르면 판단이 달라질 수 있고, 사람이 직접 자료를 찾고 기록해야 해 시간이
걸립니다. 또한 기록이 충분하지 않으면 나중에 어떤 근거로 어떤 판단과 조치를
했는지 추적하기 어렵습니다.

## 해결 방식

AI Smart Factory Agent Office는 Paderborn University Bearing DataCenter에서
저장된 Bearing 측정 Data를 불러오고, 선택한 측정 Data의 신호를 시간축에 따라
재생하면서 진동·전류 Data를 분석해 다음 진단 절차를 하나의 진단 흐름으로 연결합니다.
현재 ML은 진동 1개 채널과 전류 2개 채널에서 추출한 36개 특징값을 이용합니다.

- 선택한 측정 Data의 저장 신호를 해당 측정의 시간축에 따라 재생(Replay)합니다.
- 진동 1개 채널과 전류 2개 채널에서 신호의 크기와 변화 특성을 요약한 총 36개의 특징값을 추출합니다.
- 여러 의사결정나무의 결과를 종합하는 Random Forest로 Bearing의 정상·이상 상태를 판별합니다.
- 정상으로 판정되면 RAG와 후속 진단을 생략하고 정상 보고서를 생성합니다.
- 이상으로 판정되면 RAG(관련 기술문서 검색)가 상태·진동 이상·점검·정비 관련 문서에서 현재 상황과 관련된 내용을 찾아 진단 근거로 제시합니다.
- AI Agent가 근거가 충분한지 확인하고, 이상 판별 시 원인을 진단한 뒤 추가 점검이 필요한 항목에 대한 점검 계획을 수립하고 작업자에게 조치 방법을 제안해 판단을 지원합니다.
- 중요한 결정이 필요하면 작업자에게 판단을 요청하고 Workflow를 대기 상태로 전환합니다. 작업자가 승인·수정 요청·거절 중 하나를 선택하면 AI Agent가 그 결과를 반영해 처음부터 다시 시작하지 않고 저장된 Checkpoint부터 이후 절차를 이어서 실행합니다.
- 최종 보고서와 AI Agent 실행 이력은 실행 기록 저장소에, Checkpoint는 중단 시점의 Workflow 상태 저장소에, 설비 이력은 별도 설비 이력 저장소(SQLite)에 분리해 저장합니다.

## 전체 진단 흐름

```text
저장된 측정 Data 재생
→ 특징값 추출
→ Random Forest로 Bearing 상태 판별
→ 정상: RAG 생략 → 정상 보고서 생성
→ 이상: 관련 기술문서 검색 → 진단 근거 제시
→ AI Agent가 근거가 충분한지 확인
→ 이상 판별 시 원인 진단
→ AI Agent가 추가 점검이 필요한 항목에 대해 점검 계획 수립
→ AI Agent가 작업자에게 조치 방법 제안
→ 필요 시 작업자 판단(HITL): Workflow 대기 → 승인·수정 요청·거절 → Checkpoint 이후부터 재개
→ 최종 보고서 생성
```

위 흐름은 사용자 관점의 설명입니다. 실제 Graph에서는 RAG가 검색한 기술문서 근거를 바탕으로 AI Agent가 1차 이상 원인 진단 결과를 구성한 뒤, AI Agent가 근거가 충분한지 확인합니다. 이후 추가 점검이 필요한 항목에 대한 점검 계획을 수립하고 작업자에게 조치 방법을 제안해 작업자의 판단을 돕습니다.

## 대표 입력 Data와 검증 경로

| 시나리오 | 실제 입력 | 무엇을 검증하는가 | 결과 |
|---|---|---|---|
| A 정상 | Paderborn 저장 측정 Data `K001` | 정상 판정 후 불필요한 이상 진단을 생략하는 경로 | ML 정상 → RAG·후속 진단 생략 → 정상 보고서 |
| B 이상 | Paderborn 저장 측정 Data `KA01` | 이상 판정 후 기술문서 근거를 활용하는 전체 진단 경로 | RAG 검색 → AI Agent가 근거가 충분한지 확인 → 이상 판별 시 원인 진단 → AI Agent의 추가 점검이 필요한 항목에 대한 점검 계획 → 작업자에게 조치 방법 제안 → 최종 보고서 생성 |
| C 작업자 판단 | Paderborn 저장 측정 Data `KA01` + 작업자 판단이 필요하도록 설정한 통제된 시연 조건 | AI Agent가 실제로 멈추고 작업자 판단을 기다린 뒤 같은 실행 건을 이어가는지 | Workflow 일시 중단 → 근거·조치 방법 제안 전달 → 승인·수정 요청·거절 → 선택 반영 → Checkpoint부터 재개 → 완료 |

발표 대표 시나리오는 A 정상·B 이상·C 작업자 판단 3개이며, 내부 통합 검증에는
Retry(재시도)를 포함한 4개 시나리오가 있습니다.

※ C 시나리오의 통제 조건은 ML을 통한 Bearing의 정상·이상 판정이나 기술문서 내용을
임의로 바꾸는 것이 아니라, 작업자 판단(HITL) 경로를 확실히 검증하기 위한 Workflow
조건입니다.

### 핵심 용어

- **ML(Machine Learning):** 진동 1개 채널과 전류 2개 채널에서 신호의 크기와 변화 특성을 요약한 36개 특징값을 추출해 Bearing의 정상·이상 상태를 판별합니다.
- **Random Forest:** 여러 의사결정나무의 판단을 종합해 상태를 분류하는 ML 모델입니다.
- **RAG(검색 증강 생성):** 미리 준비한 기술문서에서 현재 상황과 관련 있는 내용을 찾아 AI Agent가 진단할 때 근거로 사용하는 방식입니다.
- **근거 자료:** 문서명·발행기관·페이지(Page)·구역(Section)·문서 조각(Chunk)·검색 점수 등 확인할 수 있는 기술문서 근거입니다. 작업자는 해당 근거가 AI Agent가 수행한 Bearing 이상 원인 진단과 AI Agent가 제안한 추가 점검 중 어느 과정에서 사용되었는지도 확인할 수 있습니다.
- **작업자 판단(HITL: Human-in-the-Loop):** 중요한 결정을 AI가 임의로 진행하지 않고 Workflow를 잠시 멈춘 뒤 작업자의 판단을 기다리는 단계입니다. 작업자는 승인·수정 요청·거절 중 하나를 선택할 수 있고, 결과가 반영되면 Checkpoint부터 작업이 재개됩니다.
- **Checkpoint:** AI Agent가 어느 단계에서 멈췄는지와 그때의 Workflow 상태를 저장해, 작업자 판단 후 같은 실행 건을 그 지점부터 이어갈 수 있게 하는 저장 지점입니다.
- **AgentEvent:** AI Agent가 현재 어떤 단계를 시작·완료·대기·오류 처리했는지 기록하는 단계별 실행 이벤트입니다.
- **SSE(Server-Sent Events):** Backend가 AgentEvent를 브라우저 화면으로 실시간 전달해 현재 Workflow 진행 상태를 보여주는 방식입니다. 실제 Sensor Data를 실시간 수집하는 기능과는 다릅니다.
- **E2E(End-to-End):** 저장 측정 Data 입력부터 ML이 Bearing의 정상·이상 상태를 판별하고, RAG가 Bearing 관련 기술문서를 검색하며, AI Agent가 근거가 충분한지 확인하고 이상 판별 시 원인을 진단한 뒤 추가 점검이 필요한 항목에 대해 점검 계획을 수립하고, 작업자 판단과 최종 보고서 생성까지 전체 Workflow를 한 번에 검증하는 것입니다.

## 작업자 판단(HITL)과 안전

본 시스템은 설비를 직접 제어하거나 작업자의 정비를 대신하지 않습니다.
AI Agent는 ML을 통한 Bearing 상태 판별 결과와 기술문서 근거를 바탕으로 근거가 충분한지 확인하고,
이상 판별 시 원인을 진단하며, 추가 점검이 필요한 항목에 대해 점검 계획과 조치 방법을 제안합니다. 중요한 결정은 작업자가 하도록 지원합니다.

- PLC를 통한 설비 직접 제어 없음
- 설비 자동 정지 없음
- 자동 정비 실행 없음
- 중요한 결정이 필요하면 작업자에게 판단을 요청하고 Workflow를 대기 상태로 전환
- 작업자는 승인·수정 요청·거절을 선택
- 최종 정비 판단은 작업자가 수행

## 팀 역할 요약

| 팀원 | 담당 영역 | 관련 소스 |
|---|---|---|
| 이승훈 | 프로젝트 총괄·시스템 통합 | `backend/app/core/` |
| 남병광 | Paderborn 데이터·ML·Backend API·배포 | `backend/app/data/`, `backend/app/ml/`, `backend/app/api/v1/` |
| 김보미 | 기술문서·RAG | `backend/app/rag/` |
| 이혜경 | LangGraph·작업자 확인·설비 이력 | `backend/app/agent/`, `backend/app/memory/` |
| 김시율 | Agent Office UI·사용자 경험 | `frontend/src/` |

## 팀 협업 및 개발 기록

협업 자료는 팀의 논의·검토·개선 과정을 보여주는 보조 증빙이며, AI Native Final
Project 제출 산출물과 구분됩니다. 협업 자료의 실제 포함 여부와 위치는 현재 저장소의
문서 구조를 기준으로 확인합니다.

## 기준 Data 및 기술문서

현재 MVP는 Paderborn University Bearing DataCenter의 저장 Data를 기준으로 사용합니다. RAG용 지식 자료는
Bearing·진동 진단·손상·점검·정비 관련 기술자료로 구성했습니다. Data와 지식 자료 묶음은
Adapter/Manifest 구조로 분리해 향후 교체·확장할 수 있도록 설계했습니다.

## 데이터

- 출처: [Paderborn University Bearing DataCenter](https://mb.uni-paderborn.de/en/kat/research/bearing-datacenter)
- 대표 subset: `K001`, `KA01`, `KI01`
- 측정 건수: 240건
- Dataset이 제공하는 정답(Ground Truth): `healthy` 80건, `damaged` 160건
- 저장 정보: 진동, 전류, 회전속도, 토크(Torque), 힘, 온도 관련 메타데이터
- 현재 ML 입력: `vibration_1`, `phase_current_1`, `phase_current_2`에서 추출한 36개 특징값
- 화면 재생: 선택한 측정 Data의 저장된 신호를 공통 시간축에 맞춰 순차적으로 재생하고, 해당 측정의 운전 조건 정보를 함께 표시
- 시험 범위: 대표 3개 Bearing ID(`K001`, `KA01`, `KI01`)를 사용한 내부 시험 결과이며, 새로운 Bearing이나 실제 현장에서도 같은 성능을 낸다는 의미는 아닙니다.

## ML

| 항목 | 현재 값 |
|---|---|
| Model ID | `bearing_rf_binary_v1` |
| 알고리즘 | `RandomForestClassifier` v1.0 |
| 입력 | 3개 신호 채널에서 추출한 36개 특징값 |
| 분석 구간 | 1초, 중첩 비율 0% |
| 분할 방식 | 측정 Data ID 기준 그룹 분할: 같은 측정에서 만든 분석 구간이 학습과 Test에 동시에 들어가지 않도록 분리 |
| 학습 / 테스트 | 180 / 60 측정 건, 718 / 239 분석 구간 |

Random Forest는 여러 의사결정나무의 결과를 종합해 Bearing의 정상·이상 상태를
판별합니다. 대표 Bearing 3종 내부 시험에서 정확도와 F1은 1.0이었습니다. 다만 같은
Bearing ID가 학습과 Test 양쪽에 포함되어 있으므로, 학습에 사용하지 않은 새로운
Bearing이나 실제 현장에서도 같은 성능이 나온다는 의미는 아닙니다. UI confidence도
실제 고장 확률로 보정된 값이 아닙니다.

## RAG 및 근거 자료

| 항목 | 현재 값 |
|---|---|
| 지식 자료 묶음 | `bearing_v1` v1.0 |
| 문서 수 | 15 |
| 문서 조각(Chunk) / 벡터 수 | 183 / 183 |
| 벡터 데이터베이스 | Chroma, collection `bearing_v1` |
| 벡터 변환기 | `sklearn-hashing-vectorizer-v1` (local HashingVectorizer) |
| 벡터 크기 / 유사도 계산 기준 | 384차원 / cosine(벡터 방향의 유사도 비교) |

RAG는 15개 문서를 183개 작은 조각으로 나누고, 각 조각의 단어 특징을 컴퓨터가
비교할 수 있도록 384개의 숫자로 바꿔 Chroma에 저장합니다. 현재 상황과 관련 있는
자료를 검색해 진단 근거를 제공하는 방식입니다.
검색 결과는 문서명, 발행기관, 페이지, 섹션, 문서 조각(Chunk), 검색 점수와 함께
`EvidenceObject`로 변환됩니다. 작업자는 해당 근거가 AI Agent가 수행한 Bearing
이상 원인 진단과 AI Agent가 제안한 추가 점검 중 어느 과정에서 사용되었는지도 확인할 수 있습니다.
검색 점수는 현재 상황과 기술문서 내용의 관련성을 비교하는 상대 순위 신호이며,
실제 고장 원인이 맞을 확률이나 진단 신뢰도를 뜻하지 않습니다.

- `Hit@5 = 0.913`: 필요한 근거가 상위 5개 검색 결과 안에 포함된 비율 91.3%
- `MRR = 0.862`: 필요한 근거가 검색 결과 앞쪽에 나타나는 정도이며, 1에 가까울수록 좋음

## 추적성

Run/AgentEvent(실행 기록), 작업자 요청·응답, Evidence(근거와 출처),
Checkpoint(중단 상태), 설비 이력, 최종 보고서를 목적별로 분리 저장합니다. 선택형 이미지
기능을 사용할 경우에만 이미지 관찰 메타데이터도 측정 Data, ML 결과, RAG 근거와 별도의
출처로 기록합니다. 진단 이력은
진단 흐름을 다시 실행하지 않고 저장된 상태·근거·최종 보고서를 다시 확인할 수
있으며, SSE는 실제 Sensor 신호를 실시간 수집·전송하는 기능이 아니라 Backend의
AgentEvent를 화면으로 전달하는 방식입니다.

## 개발·재현 참고자료

아래 문서는 개발·재현을 위한 참고자료이며, AI Native Final Project(전문가 평가 요청
대상 과제물) 목록과 구분되는 내부 작업 문서입니다.

<details>
<summary>개발·재현 참고자료 보기</summary>

- `docs/01_PROJECT_OVERVIEW.md` ~ `docs/10_CODEX_MASTER_INSTRUCTION.md`: 무엇을 왜 만드는지 정의하는 설계 문서
- `codex/STEP_01_PROJECT_SCAFFOLD.md` ~ `codex/STEP_15_EVALUATION_DEPLOY.md`: Codex가 순차 실행할 구현·검증 지시서
- `codex/STEP_02A_DATASET_RAG_DOWNLOAD.md`: 데이터셋과 RAG 원천자료 확보를 위한 별도 준비 STEP
- `docs/NON_TECHNICAL_TEAM_RUNBOOK.md`: 비전공자용 실행·테스트·디버깅·업무분장 가이드
- `docs/RAG_MVP_ASSIGNMENTS.md`: ChatGPT/Claude 채팅창에서 진행하는 RAG 구축 과제지
- `docs/RAG_MVP_PROMPTS.md`: 과제별 ChatGPT/Claude 프롬프트 모음
- `docs/RAG_MINI_PROJECT_WORKSHOP.md`: 문서를 업로드해 원리를 체험하는 ChatGPT/Claude 개념 실습자료

### Codex 시작 방법

처음에는 코드를 작성하지 않는다. Codex에게 다음 순서로 문서를 읽게 한다.

1. `README.md`
2. `docs/01_PROJECT_OVERVIEW.md` ~ `docs/10_CODEX_MASTER_INSTRUCTION.md`
3. `codex/` 전체 STEP 목록

첫 요청에서는 프로젝트 구조와 문서 간 충돌 여부를 검토하고 실행 계획만 보고하게 한다. 사용자의 승인을 받은 뒤 `STEP_01_PROJECT_SCAFFOLD.md`부터 한 STEP씩 `READ → PLAN → IMPLEMENT → TEST → REPORT` 순서로 실행한다.

### 실행 순서

```text
STEP_01
→ STEP_02A
→ STEP_02
→ STEP_03
→ STEP_04
→ STEP_05
→ STEP_06
→ STEP_07
→ STEP_08
→ STEP_09
→ STEP_10
→ STEP_11
→ STEP_12
→ STEP_13
→ STEP_14
→ STEP_15
```

실제 실행 순서는 `STEP_01 → STEP_02A → STEP_02 → ... → STEP_15`로 고정한다.

### 중요한 구현 원칙

- UI 애니메이션이 Agent 진행 상태를 만들지 않는다. 실제 `LangGraph → AgentEvent → SSE → UI` 상태가 화면을 움직인다.
- ML 결과, RAG 기술 근거, 설비 이력, 작업자 관찰, 이미지 관찰을 서로 다른 출처로 유지한다.
- Ground Truth와 평가용 정보가 Runtime Agent Context에 유입되지 않게 한다.
- 기술문서에 없는 근거·페이지·수치·고장 원인을 생성하지 않는다.
- Agent는 의사결정 지원 시스템이며 PLC 직접 제어 또는 무승인 고위험 조치를 수행하지 않는다.
- Paderborn과 별도 점검 이미지를 동일 실험체의 동기화 데이터라고 주장하지 않는다.
- Dataset 및 RAG 자료의 라이선스와 재배포 조건은 최종 배포 전에 다시 확인한다.

</details>

## 완료 기준

최종 MVP는 저장된 Paderborn 측정 Data 입력으로 분석을 시작하고, 이상 시 기술
근거를 검색·검증하며, 필요하면 추가 검색과 작업자 확인을 수행하고, 추가 점검 항목에
대한 점검 계획·조치
방법 제안·최종 보고서까지 생성해야 합니다. 과거 설비 이력은 다음 진단에 활용되며 실제 백엔드
진단 흐름 상태가 Agent Office에 실시간 표시되어야 한다.

## 시스템 구조 (Architecture)

```text
Next.js Frontend
    ↓ REST / SSE
FastAPI Backend
    ├── 데이터 / ML 서비스
    ├── RAG 서비스
    ├── LangGraph agent
    ├── 설비 이력 서비스
    ├── 선택형 점검 이미지 / 이미지 관찰 서비스
    └── Checkpoint / Run / Memory persistence
```

현재 프로젝트에는 실행 가능한 프런트엔드/백엔드 애플리케이션,
Paderborn 데이터 변환기와 ML 분석 API, `bearing_v1` 지식 색인,
목적별 근거 자료 검색, 체크포인트 기반 LangGraph 작업 흐름,
점검 계획·조치 방법 제안, 작업자 판단(HITL), 지속 가능한 AgentEvent/SSE API,
Agent Office UI, 설비별 장기 이력, 선택형 관찰 전용 다중 모달 점검 기능이 포함됩니다.

```mermaid
flowchart LR
    S[저장된 측정 Data] --> M[Random Forest 분류]
    M --> G[LangGraph 작업 흐름]
    G <--> R[기술 자료 RAG]
    G <--> H[설비 이력]
    G <--> V[선택형 이미지 관찰]
    G --> W[작업자 판단(HITL)] --> G
    G --> P[점검 계획 / 조치 방법 제안 / 최종 보고서 생성]
    P --> E[AgentEvent / SSE]
    E --> U[Agent Office]
```

각 정보의 기준이 되는 저장·전달 객체는 측정 Data 변환기,
`AnalysisResult`, `EvidenceObject`, LangGraph `AgentState`, 설비 이력,
`VisualObservation`, 저장된 실행 기록/이벤트로 구분합니다.

## 저장소 구조

```text
backend/     FastAPI 애플리케이션 및 테스트
frontend/    Next.js TypeScript 애플리케이션
data/        데이터 위치; 원본 Paderborn 데이터는 저장소에 포함하지 않음
knowledge/   지식 자료 및 목록 위치
models/      모델 관련 기본 경로; 현재 활성 모델 산출물은 `artifacts/ml/`에 저장
artifacts/   검증·평가 결과 및 활성 ML 모델 산출물
docs/        설계 명세
codex/       순차 실행 지시서
```

## 사전 요구사항

- Python 3.11 or newer
- Node.js 20 or newer
- npm 10 or newer

## 환경 변수

로컬 설정을 위해 `.env.example`을 `.env`로 복사한 뒤 환경 변수를 설정합니다.
주요 환경 변수는 다음과 같습니다.

<details>
<summary>전체 환경 변수 보기</summary>

- `API_V1_PREFIX`
- `CORS_ORIGINS`
- `PADERBORN_DATA_ROOT`
- `KNOWLEDGE_BASE_ROOT`
- `MODEL_ROOT`
- `DATABASE_URL`
- `VECTOR_DB_PATH`
- `NEXT_PUBLIC_API_BASE_URL`
- `ACTIVE_ML_MODEL_ID`, `KNOWLEDGE_PACK_ID`, `WORKFLOW_VERSION`
- `AGENT_RUN_DB_PATH`, `AGENT_CHECKPOINT_PATH`, `EQUIPMENT_MEMORY_DB_PATH`
- `VISION_ENABLED`, `VISION_MODEL`, `MAX_IMAGE_SIZE_MB`
- `STARTUP_VALIDATION_STRICT`

</details>

STEP 01에서는 외부 API, 데이터베이스, 데이터셋, 모델 또는 벡터 저장소 연결이 필요하지 않습니다.

## Backend 설치 및 실행

```bash
cd backend
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
uvicorn app.main:app --reload
```

Windows PowerShell에서는 다음과 같이 실행합니다.

```powershell
cd backend
py -3.11 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
python -m uvicorn app.main:app --reload
```

Health endpoint: `GET http://localhost:8000/api/v1/health`

Readiness endpoint: `GET http://localhost:8000/api/v1/ready`. Dataset, 활성 ML
산출물, Knowledge Pack Manifest, Chroma 벡터 데이터베이스, Agent/Checkpoint/Memory/
Vision SQLite 저장소를 확인합니다. 통제된 배포 환경에서 필수 의존성을 사용할 수 없을
때 시작을 실패시키려면 `STARTUP_VALIDATION_STRICT=true`를 설정합니다.

Runtime `READY`는 Backend 실행에 필요한 데이터셋·모델·지식 자료·벡터 저장소·
Agent/Checkpoint/Memory/Vision 저장소를 사용할 수 있는 상태를 뜻합니다. 최종 테스트와
전체 Workflow 검증 결과는 별도로 확인합니다.

**Backend 최종 테스트:** 162건 통과 — 최신 종합 검증 결과

<details>
<summary>별도 Windows 회귀검증 기록 보기</summary>

Windows 회귀 테스트: 143/143 통과

</details>

따라서 평가 시 최신 Backend 테스트 수치는 162건이며, Windows 143/143은 이를
대체하거나 합산하는 수치가 아닙니다. Knowledge Pack 검증은 15개 문서와
183개 조각을 대상으로 중복 안전 재수집까지 확인한 별도 결과입니다. 외부
Service 시연, 사용자 feedback 기록, 최종 제출 산출물 검증도 각각 별도 항목입니다.

OpenAPI UI: `http://localhost:8000/docs`

### ML 분석 API

STEP 05는 색인된 측정 ID를 기준으로 실제 모델 추론을 제공합니다.

```http
POST /api/v1/analysis
Content-Type: application/json

{
  "measurement_id": "paderborn:K001:N09_M07_F10:01",
  "model_id": "bearing_rf_binary_v1"
}
```

`model_id`는 선택 사항이며, 입력하지 않으면 `ACTIVE_ML_MODEL_ID`를 사용합니다.
응답은 공통 `AnalysisResult` 계약을 따르며 모델·특징값 버전, 추론 시각, 제한된
신호 특징값, 처리 시간 정보를 포함합니다. Confidence는 보정되지 않은 모델 출력이며,
설비 고장 확률을 그대로 의미하지 않습니다.

공개 오류 코드와 응답 형식은 [`docs/07_API_DB_SPEC.md`](docs/07_API_DB_SPEC.md)와
OpenAPI UI에서 확인할 수 있습니다. 내부 경로와 stack trace는 클라이언트에
반환하지 않습니다.

프로젝트 루트에서 실제 Dataset/Model 점검을 실행합니다.

```bash
backend/.venv/bin/python scripts/smoke_test_analysis_api.py
```

### RAG 지식 자료 수집

STEP 06은 승인된 `bearing_v1` 기술문서를 HashingVectorizer로 숫자 벡터로 바꿔
로컬 Chroma에 저장합니다. 외부 Embedding API나 별도 Embedding Model은 사용하지
않습니다.

```bash
backend/.venv/bin/python scripts/ingest_knowledge_pack.py --pack bearing_v1 --dry-run
backend/.venv/bin/python scripts/ingest_knowledge_pack.py --pack bearing_v1
```

수집 결과는 `artifacts/rag/bearing_v1`에 기록되고, 로컬 컬렉션은
`artifacts/vector_db`에 저장됩니다.

### RAG 근거 자료 검색

STEP 07은 개발·평가용 API endpoint를 제공합니다.

```http
POST /api/v1/knowledge/search
Content-Type: application/json

{
  "query": "bearing vibration defect frequency",
  "purpose": "DIAGNOSTIC_EVIDENCE",
  "top_k": 5,
  "filters": {}
}
```

검색 목적은 `DATASET_EVIDENCE`, `DIAGNOSTIC_EVIDENCE`, `INSPECTION_ACTION`으로
구분합니다. 검색 결과는 인용 가능한 `EvidenceObject`이며, 검색 점수는 순위 신호일
뿐 근거가 맞을 확률을 뜻하지 않습니다. Bearing별 정답 자료는 기본 `DIAGNOSTIC`
모드에서 제외하고 명시적인 `EVALUATION` 모드에서만 사용합니다. 세부 필터와 응답
계약은 [`docs/07_API_DB_SPEC.md`](docs/07_API_DB_SPEC.md)와 OpenAPI에서 확인합니다.

프로젝트 루트에서 검색 평가를 실행합니다.

```bash
backend/.venv/bin/python scripts/evaluate_retrieval.py
```

### LangGraph 핵심 작업 흐름

STEP 08은 기존 ML 분석과 RAG 검색을 상태를 저장하는 LangGraph로 연결합니다.
정상 측정은 RAG를 생략하고, 이상 측정은 목적별 질의를 만들어 근거를 검색한 뒤
기술문서 근거를 바탕으로 구조화된 진단 결과를 만들고, 그 판단을 뒷받침하는
근거가 충분한지 확인합니다. SQLite Checkpoint는
`run_id`를 LangGraph thread ID로 사용합니다.

프로젝트 루트에서 실제 정상·이상 검증 시나리오를 실행합니다.

```bash
backend/.venv/bin/python scripts/run_langgraph_core_validation.py
```

STEP 09는 근거가 연결된 점검 계획·조치 방법 제안·작업자 판단 대기(승인·수정 요청·거절)와
구조화된 최종 보고서까지 Workflow를 확장합니다. 설비 제어 작업은 실행하지 않습니다.

작업자 입력은 다음 API를 통해 같은 Checkpoint 실행을 재개합니다.

```http
POST /api/v1/agent/runs/{run_id}/human-input
Content-Type: application/json

{
  "request_id": "request_...",
  "response": {"decision": "APPROVED"},
  "comment": "Reviewed"
}
```

작업자 선택지는 승인·수정 요청·거절입니다.

실제 STEP 09 검증 시나리오는 다음 명령으로 실행합니다.

```bash
backend/.venv/bin/python scripts/run_actions_hitl_validation.py
```

STEP 10은 비동기 Agent Run API로 Checkpoint 기반 LangGraph Workflow를 제공합니다.

```text
POST /api/v1/agent/runs
GET  /api/v1/agent/runs/{run_id}
GET  /api/v1/agent/runs/{run_id}/events
GET  /api/v1/agent/runs/{run_id}/events/stream?after_sequence=0
POST /api/v1/agent/runs/{run_id}/human-input
GET  /api/v1/agent/runs/{run_id}/report
GET  /api/v1/agent/runs/{run_id}/evidence
```

실행 기록과 이벤트 메타데이터는 LangGraph Checkpoint와 별도로 저장됩니다.
SSE(Server-Sent Events)는 실제 Sensor 신호가 아니라 Backend의 AgentEvent를 브라우저로
실시간 전달하는 방식입니다.
SSE 이벤트 재연결은 `after_sequence` query parameter만 사용합니다.

<details>
<summary>운영 배포 참고사항</summary>

heartbeat comments는 Workflow event로 저장하지 않습니다. reverse proxy 뒤에 배포할
때는 event 실시간 전송 경로의 response buffering을 끄고 `SSE_HEARTBEAT_SECONDS`보다
긴 keep-alive 및 idle timeout 값을 설정합니다.

</details>

실제 정상·이상·HITL API/SSE 검증 자료(event 전송 확인 자료)를 생성하려면 다음을 실행합니다.

```bash
cd backend
.venv/bin/python -m tools.run_step10_e2e
```

Backend 계약용 이벤트 viewer는 `artifacts/api/agent_sse/event_viewer.html`입니다.
설정된 CORS origin(기본값 localhost 3000)에서 제공하며, UI가 임의의 진행 상태를
만들지 않고 Backend의 실제 AgentEvent만 표시합니다.

STEP 11은 `/`에서 AI Native Final Project MVP Agent Office를 제공합니다. 실제 실행
상태와 SSE 이벤트에서 Agent 영역, LangGraph node 상태, 선택 경로, timeline, 근거,
진단, 점검·조치 정보, HITL 창, 최종 보고서를 표시합니다. Agent Office UI는 한글
중심으로 표시하되 추적에 필요한 기술 식별자는 영어 원문을 유지합니다. 측정 Data
미리보기 주소는 다음과 같습니다.

```http
GET /api/v1/measurements/{measurement_id}/preview?max_points=160
```

STEP 11 브라우저 검증 보고서와 화면 캡처는 `artifacts/ui/agent_office/`에 있습니다.

### 설비 장기 이력

STEP 12는 기술문서 RAG 벡터 컬렉션과 분리된 SQLite에 설비 이력을 간결하고
추적 가능하게 저장합니다. Workflow는 현재 측정 메타데이터를 읽은 뒤 제한된 설비
이력을 불러오며, 설비 이력이 현재 신호나 ML 특징값을 바꾸지는 않습니다.

현재 Paderborn MVP의 대표 subset은 공통 시험 장비 `equipment_id`를 사용합니다.
실제 운영에 적용할 때는 장비·자산·부품/Bearing 식별자를 분리한 뒤 설비별 이력으로
취급해야 합니다.

```http
GET  /api/v1/equipment/{equipment_id}/history
GET  /api/v1/equipment/{equipment_id}/memory
POST /api/v1/equipment/{equipment_id}/maintenance
```

정비 API는 명시적으로 제출된 기록만 저장하며, Agent의 제안만으로 정비 기록을
자동 생성하지 않습니다.

MVP 설비 이력의 저장 범위:

- 저장: 측정 참조, ML 요약, 진단 결과, 점검·조치 요약, 작업자 관찰, 명시적으로 입력된 정비 기록, 완료된 실행 결과
- 저장하지 않음: 중복 원본 신호, 전체 대화, 기술문서 조각, 불필요한 개인정보
- 용도와 한계: 추세·과거 실행·감사·결과 연결에 사용하며, 자동 재학습에는 사용하지 않음
- 삭제·중복 방지: `equipment_id`별로 분리하고, `memory_version`과 출처 조합으로 중복을 방지함

프로젝트 루트에서 실제 2회 실행 검증을 수행합니다.

```bash
backend/.venv/bin/python scripts/run_memory_validation.py
```

JSON 및 보고서 결과는 `artifacts/memory/`에 저장됩니다.

### 선택형 이미지 관찰

STEP 13은 Sensor ML 입력을 바꾸지 않고 작업자가 선택적으로 외부 점검 이미지를
추가할 수 있게 합니다. 이미지 파일은 제한된 로컬 저장소에, 메타데이터와 구조화된
관찰 결과는 별도 SQLite에 저장합니다. JPEG, PNG, WebP 파일은 형식·크기·해상도와
실제 이미지 해석 가능 여부를 확인합니다.

이미지 관찰은 선택 기능이며 이미지의 해상도·밝기·대비·사용 가능 여부만 관찰합니다.
Bearing 내부 고장을 판별하는 Vision Model이 아니며 설비 안전을 인증하지 않습니다.

```http
POST /api/v1/agent/inspection-images
POST /api/v1/agent/runs/{run_id}/inspection-images
GET  /api/v1/agent/runs/{run_id}/inspection-images
GET  /api/v1/agent/inspection-images/{image_id}
GET  /api/v1/agent/inspection-images/{image_id}/observations
```

기본 로컬 provider는 관찰 전용 이미지 품질 Adapter입니다. provider 오류가 발생하면
사용 불가 결과를 기록하고 저장된 측정 Data 분석 + RAG Workflow를 계속 진행합니다. 이미지 관찰,
기술문서 근거, 설비 이력, 작업자 관찰, 현재 Sensor 분석은 서로 다른 Context로
구분합니다.

Agent Office는 시작 전 미리보기·삭제, HITL 이미지 업로드, 분석 상태, 구조화된 관찰
결과, 한계, 진단 활용 근거를 제공합니다. 얼굴·이름표·문서 또는 불필요한 개인정보를
업로드하지 않도록 작업자에게 안내합니다.

프로젝트 루트에서 실제 Data 검증을 수행합니다.

```bash
backend/.venv/bin/python scripts/run_multimodal_validation.py
```

포함된 이미지는 프로젝트에서 생성했으며 `simulated/demo`로 명시했습니다. Paderborn
시험 Bearing과 동기화된 이미지가 아니며, 결과는 `artifacts/multimodal/`에 저장됩니다.

## MVP-RC1 데모 및 통합

고정된 interface 계약은 `config/interface_freeze_mvp_rc1.json`에, 재현 가능한 네
가지 시나리오는 `config/demo_scenarios.json`에 등록되어 있습니다. Retry(재시도)와
HITL(작업자 판단) 시나리오는 실제 Paderborn 저장 Data·활성 ML Model·실제
`bearing_v1` RAG Index를 그대로 사용합니다. 다만 Retry 또는 작업자 판단 경로를
확실히 검증하기 위해 '추가 근거가 필요함' 또는 '작업자 판단이 필요함' 같은
Workflow 경로 결정 조건만 의도적으로 발생시키는 통제된 시연 조건을 사용합니다.

※ 통제 조건은 ML을 통한 Bearing의 정상·이상 판정이나 기술문서 내용을 임의로 바꾸는
것이 아니라, Retry(재시도)·HITL(작업자 판단) 경로의 동작을 재현하기 위한 Workflow
조건입니다.

준비된 환경을 확인하고 전체 통합 검증을 실행합니다.

```bash
backend/.venv/bin/python scripts/verify_setup.py
backend/.venv/bin/python scripts/run_e2e_integration.py
```

출력 결과는 `artifacts/e2e/`에 저장됩니다. 3분 시연 흐름은
`docs/DEMO_SCRIPT.md`에 정리되어 있습니다.

모든 최종 보고서는 애플리케이션, 모델, 지식 자료 묶음 목록 및 작업 흐름 버전을
추적합니다. 실행 이벤트는 `run_id`를 유지하며, 측정 Data·분석 결과·질의·근거 자료·
출처 인용 정보는 각각의 상관관계 ID를 유지합니다.

## STEP 15 평가 및 릴리스

STEP 14 이후 재현 가능한 로컬 평가 결과를 종합합니다.

```bash
backend/.venv/bin/python scripts/run_final_evaluation.py
```

이 명령은 ML, RAG, Agent, E2E, 릴리스 점검, 배포 Smoke Test 결과를
`artifacts/final/`에 기록합니다. 사용자 검증 절차와 실제 결과는
`artifacts/user_test/`에서 확인할 수 있으며, 실제 참여자와 관찰 결과만
기록합니다. 현재 외부 Service 시연과 사용자 feedback 기록은 최종 완료 상태로
반영되어 있습니다. 이 검증 항목은 AI Native Final Project 제출 산출물의
신뢰성을 확인하기 위한 내부 품질 기준입니다.

로컬 smoke-test URL은 `http://127.0.0.1:8000`이며, 외부 접속은 상단의 Service
시연(Live Demo) 주소를 사용합니다. 릴리스 결정과 3분 시연 순서는
`artifacts/final/FINAL_EVALUATION_REPORT.md` 및
`artifacts/final/FINAL_DEMO_GUIDE.md`에서 확인합니다.

## KAMP 확장 가능성 검증 (개념 검증(PoC))

Paderborn 베어링 진단 흐름과 별도로, KAMP 정밀가공 품질보증 AI 데이터셋을
사용해 다른 제조 도메인으로 데이터 변환기와 지식 자료/RAG를 확장할 수 있는지
검증했습니다. 이는 KAMP CNC 설비용 새 ML 모델을 학습한 결과가 아니라, 데이터 입력과
기술문서 검색 흐름의 확장 가능성을 확인한 별도 개념 검증입니다.

- 데이터: 1,085개 관측값 / 43개 열
- Dataset이 제공한 정답(Ground Truth): PASS 986 / FAIL 99
- CNC 변환기: 1,085 / 1,085개 관측값 변환
- 변환기 + RAG 테스트: 29 / 29 통과
- 지식 자료 묶음: `kamp_cnc_v1` / 49개 문서 조각(Chunk)
- 기존 Paderborn 핵심 Workflow와 운영 DB/Chroma를 변경하지 않고, 별도 CNC Adapter와 Knowledge Pack으로 확장 검증
- CNC 전용 ML 성능이나 실제 현장 일반화 성능은 주장하지 않음

## 데이터 및 지식 자료 준비

- Data 다운로드·Checksum: `data/paderborn/metadata/`
- 출처 Manifest(JSON, 정식 기준 파일): `knowledge/bearing_v1/manifests/source_manifest.json`
- Knowledge 수집: `python -m tools.ingest_knowledge`
- 활성 Model metadata: `artifacts/ml/baseline_v1/metadata.json`

사람이 읽기 위한 출처 표는 `knowledge/bearing_v1/manifests/SOURCE_MANIFEST.md`에서
확인할 수 있으며, 실제 기준은 `source_manifest.json`입니다.

원본 Data와 내려받은 외부 문서는 라이선스 조건에 따라 로컬에서만 사용합니다.
공개 배포 전에는 Manifest에 기록된 라이선스와 재배포 가능 여부를 확인합니다.
Paderborn 항목은 CC BY-NC 4.0 또는 출처별 웹사이트 조건을 따르며, 제3자 기술문서는
각 발행처의 조건을 따릅니다.

## 출처 및 사용 조건

- Paderborn Bearing DataCenter Dataset: [공식 Bearing DataCenter](https://mb.uni-paderborn.de/en/kat/research/bearing-datacenter)
- Paderborn Dataset 라이선스: [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)
- Paderborn 원본 RAR/MAT: 저장소에 포함하지 않으며 공식 배포처에서 로컬로 내려받습니다.
- SKF 기술문서: 파일별 공식 URL과 조건은 `knowledge/bearing_v1/manifests/SOURCE_MANIFEST.md`에 기록합니다.
- KAMP CNC Dataset 및 Guidebook: 원본 CSV/PDF와 원문 파생 Chunk/Index는 저장소에 포함하지 않습니다.
- Dataset, 기술문서, 오픈소스 라이브러리의 파일별 출처와 사용 범위는 `knowledge/bearing_v1/manifests/SOURCE_MANIFEST.md` 및 이 절에 정리합니다.

## Frontend 설치 및 실행

```bash
cd frontend
cp .env.local.example .env.local
npm ci
npm run dev
```

`frontend/package-lock.json`에 기록된 의존성 기준으로 설치하려면 `npm ci`를
사용합니다. Windows PowerShell에서도 동일하게 실행할 수 있습니다.

브라우저에서 `http://localhost:3000`을 엽니다. Backend가 실행 중이고 필요한 서비스
의존성이 준비되었으면 화면에서 Backend의 상태가 정상임을 확인할 수 있습니다.

## 테스트 결과

Backend:

```bash
cd backend
source .venv/bin/activate
pytest
```

Windows PowerShell:

```powershell
cd backend
.\.venv\Scripts\Activate.ps1
python -m pytest
```

Frontend type/build validation:

```bash
cd frontend
npm run lint
npm test
npm run build
```

Frontend 테스트와 TypeScript 검사, Production Build는 통과했습니다.

2026-10-16 검증 요약:

| 검증 항목 | 결과 |
|---|---|
| Backend 최종 테스트 | 162건 통과 — 최신 종합 검증 결과 |
| Windows 회귀 테스트 | 143/143 통과 — Backend 162건과 별도 결과 |
| Frontend 테스트 | 통과 |
| TypeScript 검사 | 통과 |
| Next.js production build | 통과 |
| RAG 검색 성능 | Hit@5 = 0.913 (필요한 근거가 상위 5개 결과에 포함된 비율 91.3%) / MRR = 0.862 (필요한 근거가 앞쪽에 나타나는 정도) |

## 현재 개발 상태

완료 기능·검증 결과·현재 한계는 다음과 같습니다.

- 완료 기능: 저장 Data 분석, 정상·이상 분기, RAG 근거 검색, Agent 진단, HITL Checkpoint 재개, 설비 이력, Agent Office UI
- 검증 결과: Backend 162건, Frontend/TypeScript/Build 통과, RAG Hit@5 = 0.913 및 MRR = 0.862
- 현재 한계: 실제 공장 Sensor 미연결, 새로운 Bearing 일반화 추가 검증 필요, KAMP는 Data 입력·문서 검색 확장 검증

<details>
<summary>STEP별 개발 상태 보기</summary>

- STEP 01: implemented
- STEP 02A: implemented with a validated representative dataset subset; full 32-archive acquisition remains an explicit resumable action
- STEP 02: implemented and validated against 240 local Paderborn 측정 Data
- STEP 03: implemented with reproducible profiling artifacts, leakage-safe split guidance, and a fixed STEP 04 input specification
- STEP 04: implemented with a reproducible Random Forest baseline, 측정 단위 분할, model artifact, evaluation, and `AnalysisResult` inference service
- STEP 05: implemented with the real `POST /api/v1/analysis` contract, active-model registry, process cache, public error schema, and Dataset/Model E2E tests
- STEP 06: implemented with manifest validation, PDF/HTML parsing, section-aware chunking, traceable metadata, local HashingVectorizer 기반 벡터화, persistent Chroma indexing, and duplicate-safe re-ingestion
- STEP 07: implemented with purpose-specific queries, metadata-filtered vector retrieval, citation-ready evidence, diagnostic/evaluation isolation, ML-to-RAG integration, and a 24-case evaluation baseline
- STEP 08: implemented with an actual LangGraph, deterministic normal/abnormal routing, structured evidence-grounded diagnosis, four evidence-status routes, bounded retrieval retries, ordered AgentEvents, and SQLite checkpoints
- STEP 09: implemented with evidence-linked inspection plans, deterministic action policy, approval/additional-information HITL interrupts, same-run checkpoint resume, structured reports, and citation/audit trails
- STEP 10: implemented with asynchronous run creation, durable run/event state, ordered SSE, sequence-based reconnect, same-run HITL resume, report/evidence APIs, and real Normal/Abnormal/HITL API verification
- STEP 11: implemented with a backend-event-driven command center, five-zone Agent Office, live LangGraph/timeline, evidence-linked detail views, real HITL resume, refresh/reconnect recovery, bounded waveform previews, and stored Normal/Abnormal/HITL artifacts; browser UI click E2E와 최종 시연 자료를 반영했습니다.
- STEP 12: implemented with equipment-scoped SQLite memory, completed-run summarization, human/maintenance provenance, deterministic context selection, LangGraph history loading, deduplication/isolation, History/Trend UI, and real two-run Paderborn validation
- STEP 13: implemented with secure optional image input, structured observation-only Vision output, LangGraph/HITL integration, visual grounding, failure fallback, bilingual Agent Office UI, and real Paderborn + simulated-image workflow validation
- STEP 14: implemented as MVP-RC1 with interface/config freeze, readiness validation, fixed demo scenarios, real-data Normal/Abnormal integration, controlled Retry/HITL, memory/multimodal/SSE persistence validation, audits, performance baseline, and reproducible demo artifacts
- STEP 15: evaluation artifacts, final demo guide, 사용자 검증 자료, local deployment smoke record, release-gate report, 최종 발표자료와 3분 이내 시연영상 구현; 외부 Service 시연과 사용자 feedback 기록까지 최종 반영했습니다.

</details>

최종 검증 결과와 Runtime `READY`의 의미 차이는
[`artifacts/final/FINAL_EVALUATION_REPORT.md`](artifacts/final/FINAL_EVALUATION_REPORT.md)에서
확인할 수 있습니다.
