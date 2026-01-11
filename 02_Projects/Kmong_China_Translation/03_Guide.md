---
description: 3단계 - 사용자 가이드 및 유지보수
---

# Phase 3. User Guide (사용자 가이드)

## 1. 배포 전 필수 준비 (Prerequisites)
- **Google Drive**: '원본 폴더'와 '결과 폴더' 2개를 만들고 ID를 복사해두세요.
- **API Keys**: Gemini와 Kie.ai 키가 있어야 하며, 특히 Kie.ai는 크레딧이 충전되어 있어야 합니다.

## 2. 배포 체크리스트 (Changes Required)
워크플로우를 Import한 후 다음 노드를 **반드시 수정**하세요:
1.  **Node 'Upload Original'**: 준비한 원본 폴더 ID 입력.
2.  **Node 'Save Result'**: 준비한 결과 폴더 ID 입력.
3.  **Credential 연결**: 3가지 Credential (Drive, Gemini, Kie.ai) 연결 확인.

## 3. 트러블슈팅 (Troubleshooting)
### ⚠️ Kie.ai 500 에러
- 대부분 **JSON 문법 오류**입니다. Prompt에 쌍따옴표가 들어가거나 `image_input`이 배열이 아닐 때 발생합니다. `Sanitize` 코드 노드를 확인하세요.

### ⚠️ 이미지가 까만색으로 나와요
- 구글 드라이브 **권한 문제**입니다. `Share Link` 노드가 제대로 작동했는지, 혹은 기업 보안 정책(외부 공유 금지) 때문인지 확인하세요. (시크릿 창에서 이미지 링크 접속 테스트)

### ⚠️ 번역 퀄리티가 낮아요
- Gemini 프롬프트를 수정해야 합니다. "자연스러운 한국어 쇼핑몰 말투로 번역해" 등의 지침을 추가하세요.
