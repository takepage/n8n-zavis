---
description: 3단계 - 사용자 가이드 및 유지보수
---

# Phase 3. User Guide (사용자 가이드)

## 1. 준비 사항 (Prerequisites)
- **Apify 계정**: 유료 플랜 권장 (일일 크롤링 양에 따라 크레딧 소모 큼).
- **Google Sheet**: '참고 계정 리스트'가 작성된 시트가 있어야 합니다. URL 컬럼 확인 필수.
- **Notion**: 기획안이 쌓일 데이터베이스를 만들고, n8n Integration을 연결해주세요.

## 2. 설정 방법 (Configuration)
1.  **Credential 연결**: 제공해드린 Credentials 입력란에 각 API 키를 입력하세요.
2.  **Google Sheet ID 변경**: `Get Accounts` 노드에서 본인의 시트 ID로 교체하세요.
3.  **Notion DB ID 변경**: `Create Page` 노드에서 본인의 데이터베이스 ID를 선택하세요.
4.  **카카오톡 설정**: (사용 시) Redirect URI 설정 및 토큰 발급 절차를 선행해야 합니다.

## 3. 유지보수 및 트러블슈팅
- **Apify 비용**: 크롤링이 멈추면 Apify 크레딧 잔액을 확인하세요.
- **AI 퀄리티**: 기획안이 마음에 들지 않으면 `AI Analysis` 노드의 프롬프트를 수정하세요. (예: "더 자극적으로 써줘")
- **Notion 오류**: 속성(Property) 이름이 바뀌면 에러가 납니다. 노션 DB 컬럼명을 유지해주세요.

## 4. 커스터마이징
- **필터 기준 변경**: `Code` 노드에서 `0.03` (3%) 수치를 조정하여 더 엄격하거나 느슨하게 소싱할 수 있습니다.
