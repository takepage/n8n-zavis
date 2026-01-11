---
description: 1단계 - n8n 시나리오 기획 워크플로우
---

# Phase 1. Scenario Design (시나리오 기획)

## 1. Goal (목표)
- **중국 상세페이지 이미지 자동 번역 및 레이아웃 유지 재생성**
- 사용자가 중국어 이미지를 업로드하면, AI가 텍스트를 감지해 한국어로 번역하고, 원본 디자인(배경, 상품)을 유지한 채 텍스트만 교체한 이미지를 생성하여 제공.

## 2. Input (입력)
- **Trigger**: n8n Form (이미지 파일 업로드)
- **Source**: 업로드된 이미지 파일 (JPG, PNG)

## 3. Process (처리 과정)
1.  **Upload**: 구글 드라이브에 원본 이미지 저장 및 외부 접근용 링크 생성.
2.  **Analysis (Gemini)**:
    - 이미지 OCR (중국어 텍스트 좌표 추출).
    - 텍스트 번역 (중국어 -> 한국어).
    - 디자인 분석 (폰트 스타일, 배경 요소, 레이아웃 보존 가이드).
3.  **Prompt Engineering**: Kie.ai(Nano Banana Pro) 모델에 맞게 프롬프트 정제 (Clean JSON).
4.  **Generation (Kie.ai)**: 원본 이미지를 Input으로 넣고, 텍스트 부분만 Inpainting/Regeneration 하여 새로운 이미지 생성.
5.  **Loop**: 생성 완료 시까지 상태 대기 (Polling).

## 4. Output (출력)
- **Result**: 최종 번역된 이미지 파일.
- **Delivery**: 웹훅 응답으로 다운로드 링크 제공 및 구글 드라이브 저장.
