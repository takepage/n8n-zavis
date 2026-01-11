---
description: 1단계 - n8n 시나리오 기획 워크플로우
---

# Phase 1. Scenario Design (시나리오 기획)

고객의 요청을 받으면 **가장 먼저** 이 워크플로우를 따르십시오. 기술적인 노드 설계부터 하지 말고, 전체적인 그림을 그립니다.

## Step 1: 요구사항 파악 (Requirement Analysis)
사용자에게 질문하여 다음 정보를 확보하십시오. (이미 정보가 있다면 생략 가능)
- **목표(Goal)**: 무엇을 자동화하고 싶은가?
- **입력(Input)**: 데이터는 어디서 오는가? (예: 웹폼, 이메일, 웹훅)
- **처리(Process)**: AI 요약, 데이터 포맷 변경, 필터링 등
- **출력(Output)**: 최종 결과물은 어디로 가는가? (예: 슬랙 알림, 노션 저장)

## Step 2: 최적의 시나리오 제안 (Logic Architecture)
단순한 텍스트 나열이 아닌, **실제 구현 가능한 청사진**을 제안하십시오.

### 필수 포함 항목
1.  **Workflow Diagram**: 노드의 연결 흐름을 ASCII Art나 Mermaid로 시각화하십시오.
2.  **Strict Input/Output Definition** (Crucial):
    - **Data Project**: 정확한 파일 포맷, 컬럼명(`수납일자` 등), 데이터 타입.
    - **AI Project**: AI에게 제공될 `Context`(시스템 프롬프트), `User Input` 예시, 기대하는 `Response Fields`.
    - **Expected Outcome**: 이 자동화가 달성해야 하는 정량적/정성적 목표.
3.  **Key Logic & Algorithm**: 
    - **Rule-based**: "A값이 B보다 크면 C로 분기"와 같은 명확한 조건문.
    - **AI-based**: "A글을 분석하여 B, C 요소를 추출하고 D톤으로 작성"과 같은 프롬프트 로직 설계.
    - 데이터 흐름(Loop, Merge)이나 판단 기준을 상세히 기술하십시오.
4.  **Error Handling**: API 실패, AI 환각(Hallucination), 데이터 누락 대응 방안.
4.  **Error Handling**: API 실패, 데이터 누락 등 예외 상황에 대한 처리 방안.
5.  **User Experience**: 최종 사용자가 경험할 인터페이스와 알림 메시지 예시.

## Step 3: 사용자 승인 (Approval)
제안한 시나리오가 마음에 드는지 사용자에게 확인을 받으십시오.
- 사용자가 "좋아", "진행해"라고 하면 **Phase 2 (Node Specification)** 워크플로우로 이동할 준비를 하십시오.
