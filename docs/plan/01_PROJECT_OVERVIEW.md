# AI Smart Factory Agent Office

## 01. 프로젝트 개요 (Project Overview)

**문서 버전:** v0.1
**프로젝트 유형:** AI Native Final Project / 제조설비 정상·이상 상태 판별 및 기술문서 근거 기반 진단 AI Agent
**문서 성격:** AI Native Final Project(전문가 평가 요청 대상 과제물)의 평가자료와 참고자료입니다.
**기준 Data:** Paderborn University Bearing DataCenter의 저장된 Bearing 측정 Data
**Service 시연(Live Demo):** https://factory.aipath.kr
**설계 원칙:** 설비마다 다른 원본 Data를 공통 입력 형식으로 변환하는 Input Adapter(설비별 원본 Data를 공통 형식으로 바꾸는 기능)와, 진단에 참고할 기술문서 묶음인 Knowledge Pack(설비 진단에 참고할 문서 모음)을 분리합니다. 불필요한 기능 확장보다 안정적인 End-to-End Workflow(입력부터 최종 보고서까지 이어지는 전체 작업 흐름)를 우선하고, 최소 기능 제품을 안정적으로 완성한 뒤 선택 기능을 단계적으로 고도화합니다.

현재 최소 기능 제품(MVP)은 Paderborn University Bearing DataCenter의 저장된 Bearing 측정 Data를
사용합니다. 실제 공장에서 Sensor로 진동·전류·회전속도·토크(torque) 등의 상태
Data를 실시간으로 수집·수신하는 시험은 하지 않았습니다. AI Agent는 설비에
이상이 있다고 판단되면 점검 계획을 수립해 작업자에게 조치 방법을 제안하고,
중요한 결정은 작업자가 하도록 판단을 지원합니다.

------------------------------------------------------------------------

## 1. 프로젝트 정의

AI Smart Factory Agent Office는 저장된 Bearing 측정 결과값을 ML로 분석해
정상·이상 상태를 판별합니다.

ML이 Bearing에 이상이 있다고 판정하면, RAG가 관련 기술문서를 검색합니다. AI Agent가 근거가 충분한지 확인하고, 이상 판별 시 원인을 진단한 뒤 추가 점검이 필요한 항목에 대한 점검 계획을 수립합니다.
AI Smart Factory Agent Office는 작업자에게 조치 방법을 제안해 판단을 지원하는 제조설비 진단 시스템입니다.

선택한 Bearing 측정 Data에 저장된 신호는 해당 측정 Data의 시간축에 따라
순차적으로 재생(Replay)합니다.

현재 MVP의 상태 판별 Model은 Random Forest이며 DL(Deep Learning) Model은 사용하지
않습니다. 중요한 결정이 필요한 경우 AI Agent는 Workflow를 멈추고 작업자의
승인·수정 요청·거절을 기다립니다.

## 2. 문제 정의

제조설비에 이상이 발생하면 담당자는 측정 Data를 확인하고, 관련 기술문서를
찾아 원인을 파악한 뒤 점검 계획을 세우고 어떤 조치를 할지 판단해야 합니다.
여러 담당자에게 업무가 나뉘면 정보가 제대로 전달되지 않거나 작업자의 숙련도에
따라 판단 기준이 달라질 수 있습니다. 또한 자료 검색과 기록에 시간이 걸리고,
기록이 부족하면 나중에 어떤 근거로 판단했는지 추적하기 어렵습니다.

본 프로젝트는 저장된 Bearing 측정 결과값을 입력받아 해당 측정의 내부 신호를
시간축에 따라 재생하고, ML 모델이 정상·이상 상태를 판별한 뒤, 이상 상태에서
RAG가 기술문서를 검색합니다. AI Agent가 근거가 충분한지 확인하고, 이상 판별 시
원인을 진단한 뒤 추가 점검이 필요한 항목에 대한 점검 계획을 수립하고 작업자에게
조치 방법을 제안해 판단을 지원합니다. 중요한 결정이 필요한 경우 Workflow를 일시 중단하고 작업자의
판단을 기다리며, 승인·수정 요청·거절 결과를 반영합니다.

여기서 E2E(End-to-End)는 **저장된 Bearing 측정 결과값을 입력하는 단계부터 ML이
Bearing의 정상·이상 상태를 판별하고, 이상으로 판정되면 RAG가 관련 기술문서를 검색합니다.
AI Agent가 근거가 충분한지 확인하고, 이상 판별 시 원인을 진단한 뒤 추가 점검이 필요한
항목에 대한 점검 계획을 수립하는 흐름**을 뜻합니다. 작업자 판단이
필요하면 Workflow를 중단해 작업자에게 관련 내용을 알리고, 판단 결과를 반영한 뒤
최종 보고서까지 중간 과정이 끊기지 않고 이어지는 전체 작업 흐름입니다.

본 프로젝트가 답하고자 하는 질문은 다음과 같습니다.

- 어떤 Bearing 측정 Data가 입력되었는가?
- ML 모델은 Bearing의 정상·이상 상태를 어떻게 판별하는가?
- 이상 상태의 원인을 판단할 수 있는 기술적 근거는 무엇인가?
- 어떤 점검 계획과 조치 방법을 제안하는가?
- 작업자가 결과를 승인·수정 요청·거절할 수 있는가?
- 이전 진단 과정과 최종 보고서를 다시 확인할 수 있는가?

## 3. 핵심 목표

1. 선택한 Bearing 측정 결과값을 불러와 내부 저장 신호를 해당 시간축에 따라 재생하고 특징값을 추출한다.
2. 진동 1개 채널과 전류 2개 채널에서 추출한 36개 특징값을 사용해 Random Forest로 정상·이상 상태를 판별한다.
3. Bearing 관련 기술문서를 별도의 진단 근거 자료로 준비하고, LangChain 기반 RAG가 현재 상황과 관련 있는 내용을 검색하도록 구성한다.
4. LangGraph Workflow에서 ML 분석, RAG·DB·작업자 판단 단계를 필요한 순서로 연결한다.
5. 앞 단계의 결과에 따라 정상 경로·이상 경로·추가 검색·작업자 판단 경로를 선택한다.
6. 이상 상태에서 RAG가 기술문서 근거를 검색하고, AI Agent가 근거가 충분한지 확인하고 이상 판별 시 원인을 진단한 뒤 추가 점검이 필요한 항목에 대한 점검 계획과 조치 방법을 제안한다.
7. 중요한 결정이 필요한 경우 Workflow를 멈추고 작업자의 승인·수정 요청·거절 결과를 반영해 처음부터 다시 시작하지 않고 저장된 Checkpoint부터 이어서 실행한다.
8. AgentEvent와 SSE를 통해 현재 처리 단계, 실행 상태, 진단 근거, 작업자 판단 대기·결과 반영 상태, 최종 보고서를 Agent Office 화면에 표시한다.
9. Input Adapter와 Knowledge Pack의 분리 구조를 설계하고, KAMP CNC Data와 관련 기술문서로 입력·검색 계층의 확장 가능성을 검증한다.

## 4. ML 분석용 측정 Data와 RAG 진단 근거용 기술문서

### 4-1. ML이 분석하는 Bearing 측정 Data

ML 분석에는 Paderborn University Bearing DataCenter의 저장된 Bearing 측정
결과값을 사용합니다. 대표 Bearing ID `K001`, `KA01`, `KI01`에서 확보한
240건의 측정 결과값을 사용하며, 저장된 자료에는 진동·전류·회전속도·토크·힘·온도와
관련된 정보가 포함되어 있습니다.

현재 ML이 직접 사용하는 입력은 진동 1개 채널과 전류 2개 채널의 3개 신호입니다.
이 신호에서 총 36개의 특징값을 추출하고, Random Forest가 이를 이용해 Bearing의
정상·이상 상태를 판별합니다.

정상·손상 여부에 대한 판단(Ground Truth)은 Model 학습과 내부 성능 평가에
사용하는 정보이며, Runtime AI Agent가 설비의 고장 원인을 진단할 때 직접 근거로
사용하는 자료는 아닙니다.

Random Forest는 여러 의사결정나무의 판단을 종합해 Bearing의 상태를 분류하는
ML 모델입니다.

### 4-2. RAG가 검색하는 진단 근거용 기술문서

RAG는 Paderborn University의 측정 Data를 분석하는 기능이 아닙니다. RAG는 별도로
준비한 Bearing 상태·진동 진단·손상·점검·정비 관련 기술문서에서 현재 상황과
관련 있는 내용을 찾아 AI Agent의 진단 근거로 제공하는 검색 기능입니다.
LangChain은 이러한 문서 검색 등 AI 기능을 연결하는 데 사용하는 도구입니다.

현재는 기술문서 15개를 183개의 작은 문서 조각(Chunk)으로 나누고,
HashingVectorizer로 문서의 단어 특징을 384개의 숫자로 바꿔 Chroma에 저장한
뒤 관련 내용을 검색합니다.

Input Adapter는 설비마다 다른 원본 Data 형식을 시스템의 공통 입력 형식으로
변환하는 기능입니다. Knowledge Pack은 특정 설비를 진단할 때 참고할 기술문서를
한 묶음으로 관리한 자료입니다. 기준 Data는 이 프로젝트에서 활용한 Paderborn
University Bearing DataCenter의 실험 결과값입니다. ML은 이 자료값을 이용해
Bearing의 정상·이상 상태를 판별하고, RAG는 진단 안내서·정비 참고 자료 등
기술문서에서 진단 근거를 검색합니다. Agent는 ML 분석 결과와 RAG 검색 결과,
설비 이력을 바탕으로 이상 원인을 진단하고 추가 점검 계획을 수립한 뒤 작업자가
조치할 방법을 제안합니다. 더 적합한 기준 Data나 RAG 기술문서를 확보하면 출처와
사용 조건을 확인한 뒤 기존 자료를 해당 자료로 교체합니다. 설비에 맞는 Input
Adapter와 기술문서 묶음을 추가하거나 바꾸면 다른 설비 Data와 문서로 확장할 수
있습니다.

## 5. 전체 서비스 흐름

```text
Paderborn University Bearing DataCenter의 저장된 Bearing 측정 Data 중 원하는 자료 불러오기
→ 측정 내부 신호를 해당 시간축에 따라 재생(Replay)
→ 진동 1채널 + 전류 2채널에서 36개 특징값 추출
→ Random Forest로 Bearing의 정상·이상 판별
├─ 정상 → RAG·후속 이상 진단 생략 → 정상 보고서 생성
└─ 이상 → RAG 기술문서 검색
          → AI Agent가 근거가 충분한지 확인
          → 이상 판별 시 원인 진단
          → AI Agent가 추가 점검 계획 수립 → 조치 방법 제안
          → 중요한 결정 필요?
             ├─ 아니오 → 최종 보고서
             └─ 예 → Workflow 일시 중단
                    → 작업자 승인·수정 요청·거절
                    → 결과 반영
                    → 저장된 Checkpoint부터 작업을 이어서 실행
                    → 최종 보고서 생성

각 단계 → AgentEvent 기록 → SSE로 Agent Office에 진행 상태 전달
```

위 흐름은 사용자 관점의 설명입니다. 실제 Graph에서는 RAG가 검색한 기술문서 근거를 바탕으로 AI Agent가 1차 이상 원인 진단 결과를 구성한 뒤, AI Agent가 근거가 충분한지 확인합니다. 이후 추가 점검이 필요한 항목에 대한 점검 계획을 수립하고 작업자에게 조치 방법을 제안해 작업자의 판단을 돕습니다.

정상으로 판정되면 불필요한 RAG 검색과 후속 이상 진단을 생략합니다. 이상으로
판정되면 기술문서를 검색해 근거를 확보합니다. AI Agent가 근거가 충분한지 확인하고, 이상 판별 시 원인을 진단한 뒤 추가 점검이 필요한 항목에 대한 점검 계획을 수립하고 작업자에게 조치 방법을 제안해 판단을 지원합니다. 중요한 결정이 필요한 경우에만 작업자 판단을
요청합니다.

## 6. Agent Workflow

하나의 LangGraph Workflow 안에서 다음 처리 단계를 연결합니다.

- **Data Load/Replay:** 선택한 Bearing 측정 결과값을 불러오고 그 안의 저장 신호를 해당 시간축에 따라 재생(Replay)
- **Feature Extraction:** 진동 1채널과 전류 2채널에서 총 36개 특징값 추출
- **Detection:** Random Forest가 36개 특징값을 이용해 Bearing의 정상·이상 판별
- **Conditional Routing:** 정상 경로와 이상 경로 분기
- **RAG Search:** 이상으로 판단되면 Bearing 관련 기술문서에서 근거 검색
- **Diagnosis:** 검색한 기술문서 근거를 바탕으로 1차 이상 원인 진단 결과 구성
- **Verification:** AI Agent가 근거가 충분한지 확인
- **Inspection Plan:** AI Agent가 추가 점검이 필요한 항목에 대한 점검 계획 수립
- **Recommended Action:** AI Agent가 작업자에게 조치 방법 제안
- **HITL:** 중요한 결정이 필요한 경우 Workflow 일시 중단 후 작업자의 승인·수정 요청·거절 대기
- **Report:** 최종 보고서 생성
- **Run/AgentEvent:** 실행 이력은 별도 저장

작업이 중단된 상태에서 작업자 판단이 입력되면 처음부터 작업을 다시 시작하지
않고, 저장된 중단 지점(Checkpoint)부터 같은 실행 건의 이후 절차를 이어서
수행합니다.

```text
START
  ↓
Data Load/Replay → Feature Extraction → Detection
  ↓
정상?
  ├─ YES → 정상 보고서 생성 → END
    └─ NO → RAG Search → Diagnosis → Verification
                              ├─ 근거 부족 → 추가 검색 → Verification
                              └─ 근거 확인 → Inspection Plan → Recommended Action
                                                   ↓
                                        작업자 판단(HITL) 필요?
                                                   ├─ YES → Workflow 대기
                                                   │          ↓
                                                   │   승인·수정 요청·거절
                                                   │          ↓
                                                   │   Checkpoint부터 재개
                                                   └─ NO
                                                        ↓
                                                   Report → END
```

## 7. AI Smart Factory Agent Office

최종 화면은 Smart Factory Dashboard, Agent Office, 작업 흐름/시간 순서
(Workflow/Timeline), 근거 자료(Evidence)·진단(Diagnosis)·작업자 판단(HITL)·
최종 보고서(Report) 영역으로 구성합니다. AgentEvent는 AI Agent가 어떤 처리
단계를 시작·완료·대기·오류 처리했는지 기록하는 실행 이벤트입니다.
SSE(Server-Sent Events)는 Backend가 이 AgentEvent를 브라우저 화면으로
실시간 전달하는 방식이며, 실제 Sensor Data를 실시간 수집하는 기능과는
다릅니다.

Agent Office는 여러 AI Agent가 실제 사무실에서 대화한다는 의미가 아닙니다.
AI Agent가 현재 어떤 처리 단계를 진행하고 있는지, Bearing이 이상하다고
판단되었을 때 근거를 찾았는지, 작업자 판단을 기다리는지 등을 역할별 화면
영역으로 보여주는 사용자 화면입니다. Backend의 실제 AgentEvent가 SSE를 통해
전달될 때 화면 상태가 변경됩니다.

화면에는 다음 내용을 표시합니다.

- 설비 및 Bearing 상태
- 정상·이상 판별 결과
- 현재 처리 단계와 실행 상태
- RAG가 검색한 결과 자료와 문서명·페이지(Page)·구역(Section)·문서 조각(Chunk)·검색 점수 등 근거 자료, 그리고 해당 근거가 이상 원인 진단과 추가 점검 제안 중 어느 과정에서 사용되었는지
- 이상 원인 진단, 점검 계획 수립, 조치 방법
- 작업자 승인·수정 요청·거절 화면
- 최종 보고서와 실행 이력

검색 점수는 현재 상황과 기술문서 내용의 관련성을 비교하는 상대적인 점수이며,
실제 고장 원인이 맞을 확률을 의미하지 않습니다.

화면의 상태는 UI 애니메이션이 임의로 만드는 것이 아니라,
`LangGraph → AgentEvent → SSE → UI` 흐름에 따라 변경됩니다. AgentEvent는 각
처리 단계의 시작·완료·대기·오류 상태를 기록하고, SSE(Server-Sent Events)는
이 상태를 Agent Office 화면에 실시간으로 전달합니다.

## 8. 현재 최소 기능 제품(MVP) 구현 범위

- 선택한 Paderborn Bearing 측정 결과값 로딩 및 내부 신호 시간축 Replay
- 진동·전류 Data에서 36개 특징값 추출
- Random Forest 기반 Bearing 정상·이상 판별
- 정상·이상 조건 분기
- 이상 상태에서 15개 기술문서·183개 Chunk·HashingVectorizer(384차원)·Chroma 기반 RAG 검색
- 기술문서 근거 기반 근거가 충분한지 확인 및 이상 판별 시 원인 진단
- 추가 점검 항목과 Bearing 점검 계획 수립 및 조치 방법 제안
- 작업자 승인·수정 요청·거절
- Checkpoint 기반 중단 후 재개
- Run/AgentEvent 실행 이력, 근거 출처, Checkpoint, 설비 이력 저장·재조회 및 최종 보고서 생성
- 선택 기능: 외부 점검 이미지를 업로드해 해상도·밝기·대비 등 사용 가능 여부를 관찰합니다. Bearing 내부 고장을 판별하는 Vision Model은 아닙니다.
- AgentEvent/SSE 기반 Workflow 상태 표시 화면
- Service 시연(Live Demo) 제공 및 외부 배포, 사용자 feedback 반영

설비 이력은 AI의 장기 기억처럼 활용되는 과거 기록입니다. 과거 Bearing 측정
결과값 참조, ML 분석 요약, 진단 결과, 점검·조치 요약, 작업자 관찰, 명시적으로
입력한 정비 기록 등을 저장해 다음 진단에서 이전 결과를 다시 확인할 수 있도록
합니다. 이 이력이 ML을 자동으로 다시 학습시키는 것은 아닙니다.

## 9. 향후 고도화 범위

현재는 핵심 End-to-End Workflow(입력부터 최종 보고서까지 이어지는 전체 작업 흐름)의
안정성을 우선합니다. 최소 기능 제품을 완성한 뒤 실제 Sensor, 다양한 Bearing
설비의 운전 조건, MES/CMMS 연계, Vision 분석, Multi-Agent 등의 기능을 하나씩
적용해 단계적으로 고도화합니다.

- 실제 Sensor 연결 및 실시간 Sensor Data 수집
- 더 다양한 Bearing과 운전 조건에 대한 검증
- 단어가 비슷한 자료와 의미가 비슷한 자료를 함께 찾는 Hybrid 검색으로 개선
- MES/CMMS 등 외부 시스템 연계
- 현재 선택형 이미지 관찰 기능과 구분되는 실제 목적의 Vision 분석은 별도 Data와 성능 검증 후 확장
- 필요 시 Multi-Agent 확장

## 10. 범용화 원칙

현재 MVP는 Paderborn Bearing Data를 기준으로 하지만, Input Adapter·Domain ML
Model·기술문서 묶음을 분리해 다른 제조 Data와 관련 문서로 확장할 수 있게
설계합니다.

```text
Data Source(원본 측정 자료값의 출처)
    ↓
Input Adapter(설비별 원본 Data를 공통 형식으로 변환)
    ↓
Domain ML Model(해당 설비의 상태 판별 Model)
    ↓
표준 분석 결과(AnalysisResult)
    ↓
LangGraph Agent
    ├─ 필요 시 RAG Search ↔ Knowledge Pack(진단 근거용 기술문서)
    ├─ 필요 시 설비 이력 조회
    └─ 중요한 결정 시 작업자 판단(HITL: AI가 잠시 멈추고 사람이 승인·수정 요청·거절을 선택)
    ↓
최종 보고서 생성
```

KAMP CNC Data 1,085건에 별도의 Input Adapter와 CNC 기술문서 검색을 적용했으며,
Data 입력과 기술문서 검색 관련 29/29 시험 결과를 통과했습니다. 이 검증은 CNC
Data 입력과 기술문서 검색 가능성을 확인한 것이며, CNC 설비의 ML 이상 판별
성능까지 검증한 것은 아닙니다.

## 11. 최소 기능 제품(MVP) 성공 기준과 현재 검증 결과

현재 MVP는 다음 Workflow를 기준으로 구현·검증했습니다.

핵심 성공 기준은 Bearing의 정상·이상 상태 판별, 이상으로 판별되면 RAG 자료 검색,
AI Agent가 근거가 충분한지 확인하고 이상 판별 시 원인을 진단하며, 추가 점검 및 작업자에 대한 조치 방법 제안, 작업자
판단(HITL), 최종 보고서 생성까지 하나의 End-to-End(입력부터 최종 보고서까지
이어지는 전체 작업 흐름)로 안정적으로 완료되는 것입니다.

```text
저장된 Bearing 측정 결과값 입력
→ 측정 내부 신호를 해당 시간축에 따라 Replay
→ 36개 특징값 추출
→ Random Forest로 Bearing 정상·이상 판별
→ 정상은 RAG·후속 이상 진단 생략 → 정상 보고서 생성
→ 이상은 RAG 기술문서 검색
→ AI Agent가 근거가 충분한지 확인
→ 이상 판별 시 원인 진단
→ AI Agent가 추가 점검 계획 수립, 조치 방법 제안
→ 중요한 경우 Workflow 중단 및 작업자 판단 요청
→ 작업자 판단 결과 반영 및 중단된 지점부터 재개
→ 최종 보고서 생성 및 실행 이력 저장
→ AgentEvent/SSE로 전체 작업 과정 표시
```

여기서 RAG는 AI Agent가 Bearing 관련 기술문서에서 현재 이상 상태와 관련된
내용을 검색해 진단 근거로 사용하는 기능입니다. 측정 Data를 다시 분석하는
기능이 아닙니다. Run/AgentEvent 실행 이력과 최종 보고서는 각각 별도로 저장해
필요할 때 다시 확인합니다.

검증 결과는 다음과 같다.

- Hit@5 = 0.913: 필요한 근거가 상위 5개 검색 결과 안에 포함된 비율 91.3%
- MRR = 0.862: 필요한 근거가 검색 결과 앞쪽에 나타나는 정도이며, 1에 가까울수록 좋음
- Backend Test 162건 통과
- Windows 회귀 Test 143/143 통과 — Backend 162건과 별도 결과
- Frontend Test, TypeScript 검사, 배포용 Build 통과
- 정상·이상 경로와 HITL(중요한 결정에서 AI가 잠시 멈추고 작업자가 승인·수정 요청·거절을 선택하는 단계) 경로 검증

## 12. 현재 확정 사항

| 항목 | 결정 |
|---|---|
| 프로젝트명 | AI Smart Factory Agent Office |
| 분야 | 제조설비 정상·이상 상태 판별 및 기술문서 근거 기반 진단 AI Agent |
| 기준 Data | Paderborn University Bearing DataCenter의 저장된 Bearing 측정 Data |
| 현재 ML | 진동·전류 기반 36개 특징값을 사용하는 Random Forest |
| 현재 DL | 현재 MVP 구현 범위에 포함하지 않음 |
| RAG | 15개 기술문서·183개 Chunk·HashingVectorizer 384차원·Chroma 기반 기술문서 검색 |
| Agent | LangGraph Workflow 기반 LangChain(검색 기능 연결 도구) RAG·ML·DB·작업자 판단 연결 |
| UI | Agent Office + Workflow/Timeline + 근거·진단·HITL·보고서 화면 |
| 상태 전달 | AgentEvent 기록 및 SSE 기반 화면 전달 |
| 설비 이력 | 과거 Bearing 측정 결과값, ML 분석 요약, Bearing 진단·점검·조치 내용, 작업자 관찰 및 정비 기록 저장·재조회 |
| 우선순위 | 최소 기능 제품의 핵심 End-to-End Workflow를 안정화한 뒤 선택적으로 필요한 기능을 추가해 고도화 |
| 개발 전략 | 현재 MVP 검증 후 실제 Sensor·다양한 설비·외부 시스템으로 확장 |
| 안전 범위 | 설비 직접 제어·자동 정지·자동 정비를 수행하지 않고 작업자의 판단을 지원 |
| 현재 입력 | 저장된 Paderborn Bearing 측정 결과값 Replay; 실제 공장 Sensor 실시간 수집은 미포함 |
| KAMP 확장 검증 | CNC Data 입력·기술문서 검색 29/29 Test; CNC ML 성능은 미검증 |
| 주요 검증 | Backend 최종 Test 162건 통과, Windows 회귀 Test 143/143 통과(Backend 162건과 별도 결과), Frontend·TypeScript·Production Build 통과, Hit@5 0.913, MRR 0.862 |

## 13. 관련 설계 문서 안내

다음 문서는 Input Adapter(설비별 원본 Data를 공통 입력 형식으로 바꾸는 기능)와
Knowledge Pack(설비 진단에 참고할 기술문서 모음)을 이용해 Data와 기술문서를
교체·확장하는 구조를 안내합니다.

관련 설계 문서는 **`02_SYSTEM_ARCHITECTURE.md`**이며, 해당 문서에서는
Frontend, Backend, ML, RAG, LangGraph Agent, DB, API, 배포 구조 및 Data와
기술문서 묶음의 교체 구조를 상세히 설명합니다.
