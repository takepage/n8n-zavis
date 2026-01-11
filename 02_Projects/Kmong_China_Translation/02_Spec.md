---
description: 2단계 - n8n 노드 상세 설계 워크플로우
---

# Phase 2. Technical Specification (상세 설계)

## 1. Key Nodes Specification

### [Node] On form submission (Trigger)
- **UI**: File Upload Field (`file`)
- **Mode**: Response to Webhook

### [Node] Google Drive (Upload Original)
- **Action**: 원본 저장
- **Output**: File ID needed for sharing

### [Node] Google Drive (Share Link)
- **Action**: `Anyone with link`, `Reader`
- **Purpose**: AI 서비스(Gemini, Kie.ai)가 이미지 URL에 접근할 수 있게 함.

### [Node] Google Gemini Chat
- **Action**: 이미지 분석 및 번역
- **Model**: `gemini-2.0-flash` or `gemini-1.5-pro`
- **Prompt**:
    > "Extract Chinese text, translate to Korean. Describe background and design elements to preserve layout. Output JSON."

### [Node] Code (Sanitize Prompt)
- **Logic**: JSON 파싱 오류 방지. 쌍따옴표(`"`)와 줄바꿈(`\n`) 제거 등 정규표현식 처리 필수.
- **Why**: Kie.ai API 500 에러의 주범.

### [Node] HTTP Request (Create Task - Kie.ai)
- **URL**: `https://api.kie.ai/api/v1/jobs/createTask`
- **Model**: `nano-banana-pro`
- **Body**:
    ```json
    {
      "input": {
        "prompt": "$json.prompt",
        "image_input": ["$json.image_url"]
      }
    }
    ```
    *주의*: `image_input`은 반드시 **배열(`[]`)** 형태여야 함.

### [Node] Wait & Switch & HTTP Request (Polling)
- **Logic**: 10초 대기 -> 상태 확인 -> 완료 시 다운로드 / 대기 시 Loop.

## 2. Credentials
- **Google Drive OAuth2 API**: 파일 업로드/공유용.
- **Google Gemini API Key**: 텍스트 분석/번역용.
- **Kie.ai API Key (Header Auth)**: `Authorization: Bearer <Key>` 이미지 생성용.

## 3. Data Flow
`Form` -> `Drive(Save)` -> `Gemini(Analyze)` -> `Kie.ai(Generate)` -> `Drive(Save Result)` -> `Response`
