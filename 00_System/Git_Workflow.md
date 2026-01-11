# 🐙 Git & GitHub Workflow Guide

환경을 이동하며(집 <-> 사무실) 끊김 없이 작업하기 위한 단계별 가이드입니다.

## 1. 현재 PC에서 마무리스마 (Upload)
작업을 끝내고 자리를 비우기 전에 반드시 실행해야 합니다.

1. **저장소 연결 (최초 1회만)**
   ```bash
   # GitHub에서 'New Repository' 생성 후 주소 복사
   git remote add origin https://github.com/사용자ID/REPO이름.git
   ```

2. **작업 내용 올리기**
   ```bash
   # 1. 변경사항 확인
   git status
   
   # 2. 전체 파일 스테이징
   git add .
   
   # 3. 커밋 (무엇을 했는지 기록)
   git commit -m "작업 내용 요약 (예: 병원 로직 설계 완료)"
   
   # 4. 푸시 (GitHub로 전송)
   git push -u origin master
   # (만약 에러나면 git push -u origin main 시도)
   ```

---

## 2. 다음 PC에서 시작하기 (Download)
새로운 환경에 도착해서 작업을 시작할 때 실행합니다.

### A. 처음 세팅하는 PC라면?
1. **필수 설치**: Git, VS Code, n8n 설치.
2. **복제 (Clone)**:
   ```bash
   # 바탕화면 등 원하는 곳에서 터미널 열기
   git clone https://github.com/사용자ID/REPO이름.git n8n-agent
   cd n8n-agent
   ```
3. **작업 시작**:
   - VS Code로 폴더 열기.
   - **Agent 실행**: `.agent/HANDOVER.md`를 읽게 하고 작업 지시.
     > "인수인계 문서 읽고 아까 하던 병원 Spec 작업 계속해줘."

### B. 이미 세팅된 PC라면?
1. **최신 내용 받기 (Pull)**:
   ```bash
   cd n8n-agent
   git pull
   ```
2. **작업 시작**: 위와 동일.

---

## 💡 충돌(Conflict) 방지 팁
- **Rule**: 자리를 옮길 땐 무조건 `Push`, 앉으면 무조건 `Pull`.
- 만약 `git pull` 중 에러가 나면?
  - 당황하지 말고 에이전트에게 "깃 충돌 났어 해결해줘"라고 하세요.
