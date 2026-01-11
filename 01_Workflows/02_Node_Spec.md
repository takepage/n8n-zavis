---
description: 2단계 - n8n 노드 상세 설계 워크플로우
---

# Phase 2. Technical Specification (상세 설계)

Phase 1에서 시나리오가 확정되면 실행합니다. **뇌피셜(Hallucination)을 금지**하고 **MCP**를 사용하여 정확한 스펙을 작성합니다.

## Step 1: 노드 선정 및 검색 (Node Discovery)
시나리오에 필요한 노드를 리스트업하고, `mcp_n8n_search_nodes` 툴을 사용해 정확한 노드명(nodeType)을 찾으십시오.
- 예: "Google Sheets" -> `n8n-nodes-base.googleSheets`

## Step 2: 속성 검증 (Property Verification)
주요 노드에 대해 반드시 `mcp_n8n_get_node`를 호출하여 실제 파라미터를 확인해야 합니다.
- **필수 확인 항목**:
    - `nodeType` (정확한 시스템 명칭)
    - `properties` (Parameters, Options의 정확한 Key 이름)
    - `credentials` (필요한 인증 종류)
- 예: "Sheet ID 입력하는 곳 이름이 `fileId`인지 `documentId`인지 확인"

## Step 3: 개발자급 명세서 작성 (Developer-Ready Spec)
사용자가 "복사-붙여넣기"만으로 구현할 수 있을 정도의 **완벽한 명세서**를 작성하십시오.

### 명세서 포맷 (Strict Rule)
```markdown
### [Node ID] Node Name (Node Type)
- **Node Type**: `n8n-nodes-base.example` (MCP Verified)
- **Role**: 노드의 역할 설명
- **Parameters**: (모든 Expression 포함)
    - `resource`: `spreadsheet`
    - `operation`: `append`
    - `fileId`: `{{ $json.file_id }}` (Expression 명시)
    - `range`: `A:F`
    - `options`:
        - `rawOutput`: `true`
        - `valueRenderMode`: `UNFORMATTED_VALUE`
- **Error Settings**:
    - `onError`: `continueRegularOutput`
- **Input Data**: 이 노드로 들어오는 JSON 예시
    ```json
    { "file_id": "1A2b3C..." }
    ```
- **Verification**: 이 노드가 정상 작동하는지 확인할 수 있는 체크리스트
```

## Step 4: 검토 요청
작성된 명세서를 사용자에게 보여주고, 기술적인 부분(API 키 유무 등)에서 무리가 없는지 확인받으십시오.
승인되면 **Phase 3 (User Guide)**로 넘어갑니다.
