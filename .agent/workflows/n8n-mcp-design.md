---
description: How to design n8n workflows using MCP tools
---

# n8n Workflow Design with MCP

이 워크플로우는 **n8n 자동화 설계 작업**을 수행할 때 항상 따르는 절차입니다.
사용자가 n8n 관련 기획이나 설계를 요청하면 이 워크플로우를 실행하십시오.

## 1. Node Discovery (노드 탐색)
가장 먼저 원하는 기능에 맞는 정확한 노드 타입(Node Type)을 찾아야 합니다.
- **Tool**: `mcp_n8n_search_nodes`
- **Action**: 핵심 키워드(예: "google drive", "slack", "schedule")로 검색하여 `n8n-nodes-base.xxx` 형태의 정확한 ID를 확인합니다.

## 2. Property Verification (속성 검증)
노드의 필수 파라미터와 옵션 이름을 확인합니다. **절대 추측하지 마십시오.**
- **Tool**: `mcp_n8n_get_node`
- **Action**: 
    1. 검색된 `nodeType`을 넣고 호출합니다.
    2. `mode='docs'` 또는 기본 모드로 상세 정보를 봅니다.
    3. `requiredProperties`와 `commonProperties`의 **Key Name**을 메모합니다. (예: `fileId`인지 `id`인지)

## 3. Specification Writing (명세서 작성)
`01_Workflows/02_Node_Spec.md`의 양식에 맞춰 문서를 작성합니다.
- **Rule**: Step 2에서 확인한 파라미터 이름 그대로 작성합니다.
- **Example**:
    ```markdown
    - **Node Type**: `n8n-nodes-base.googleDrive`
    - **Parameters**:
        - `operation`: `download`
        - `fileId`: `{{ $json.file_id }}` (검증됨)
    ```

## 4. Validation (검증)
마지막으로, 작성된 명세서가 실제 n8n 버전과 일치하는지 재확인합니다.
- **Tool**: `mcp_n8n_validate_node` (Optional)
- **Check**: 필수 값이 빠지지 않았는지 확인합니다.
