---
description: 2단계 - n8n 노드 상세 설계 워크플로우
---

# Phase 2. Technical Specification (상세 설계)

## 1. Workflow Structure
안정성을 위해 4개의 독립 워크플로우로 분할 구현을 권장합니다.

1.  **Sourcing WF**: 크롤링 & 1차 필터링
2.  **Analysis WF**: AI 분석
3.  **Feedback WF**: 노션 상태 변경 감지 (Optional)
4.  **Notification WF**: 알림 발송

## 2. Key Nodes Specification

### [Node] Schedule Trigger
- **Action**: 매일 오전 9시 실행
- **Parameters**: `Cron Expression` or `Fixed Time`

### [Node] Google Sheets (Get Accounts)
- **Action**: 참고 계정 리스트 로드
- **Operation**: `Read` (Rows)
- **File**: User defined Spreadsheet ID

### [Node] Apify (Instagram Scraper)
- **Action**: 릴스 데이터 추출
- **Actor**: `instagram-reel-scraper` (or similar)
- **Input**: Instagram Profile URLs
- **Options**: `Limit` (최신 n개)

### [Node] Code (Filter Logic)
- **Action**: 효율 계산 및 필터링
- **JavaScript Logic**:
    ```javascript
    const engagement = (likes + comments) / views;
    return engagement >= 0.03 ? item : [];
    ```

### [Node] Google Gemini / Anthropic Claude
- **Action**: 콘텐츠 분석 및 기획안 생성
- **Prompt**: "이 릴스의 소구점을 분석하고, 이를 벤치마킹한 새로운 기획안을 작성해줘."
- **Output**: JSON Structure (제목, 대본, 해시태그 등)

### [Node] Notion (Create Page)
- **Action**: 기획안 적재
- **Resource**: `Database Page`
- **Operation**: `Create`
- **Properties**:
    - `Title`: AI Generated Title
    - `Status`: "New"
    - `URL`: Source URL

### [Node] KakaoTalk (HTTP Request)
- **Action**: 알림 발송
- **Note**: 토큰 관리가 복잡하므로 Slack/Telegram 대체 권장. 카카오톡 필수 시 `Refresh Token` 로직 포함된 Custom Auth 워크플로우 필요.

## 3. Credentials
- **Google Sheets OAuth2**
- **Apify API Key**
- **Google Gemini API Key**
- **Anthropic Console API Key**
- **Notion Integration Token**
- **Kakao Developers REST API Key** (If used)
