---
description: 1단계 - n8n 시나리오 기획 워크플로우
---

# Phase 1. Scenario Design (시나리오 기획)

## 1. Goal (목표)
- **Instagram Reels 소싱 및 분석 자동화**
- 매일 아침 구글 시트에 있는 '참고 계정'을 크롤링하여 고효율(조회수 대비 공유 수 높은) 릴스를 선별.
- 선별된 릴스를 AI가 분석하여 '도파민 자극' 기획안을 작성하고 Notion에 적재.
- 최종적으로 카카오톡 알림을 통해 마케터가 확인하도록 함.

## 2. Input (입력)
- **Source**: Google Sheets (참고 계정 URL 리스트)
- **Trigger**: Schedule (매일 오전 9시)
- **Criteria**:
    - 조회수 대비 공유 수 n% 이상
    - 좋아요 + 댓글 / 조회수 = 3% 이상 (고객 요건)

## 3. Process (처리 과정)
1.  **Sourcing**: Apify를 사용하여 릴스 데이터 크롤링 (닉네임, 좋아요, 댓글, 내용, URL, 대본 등).
2.  **Filtering**: 설정된 효율 조건(Engagement Rate)에 미달하는 콘텐츠 필터링.
3.  **AI Analysis**:
    - **Gemini**: 영상/대본 분석, 핵심 요소 추출.
    - **Claude/Gemini**: 마케팅 소구점 분석, 편집 가이드, 나레이션 대본 작성.
4.  **Drafting**: Notion 데이터베이스에 페이지 생성.
    - **Property**: 제목, 썸네일 문구, 상태(작성중), 해시태그.
    - **Body**: 나레이션 대본, 편집 가이드, AI 분석 요약.

## 4. Output (출력)
- **Notion Page**: 분석 및 기획안이 담긴 노션 페이지.
- **Notification**: 카카오톡(또는 Slack/Telegram)으로 작업 완료 알림 및 노션 링크 전송.
