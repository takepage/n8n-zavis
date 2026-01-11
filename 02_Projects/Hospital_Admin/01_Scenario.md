# 1. 시나리오: 요양병원 원무일보 자동화 (Hospital Admin Daily Report)

## Step 1: 핵심 요약 (Executive Summary)
매일 **전일 결제 내역(입원/외래)**을 닥터스 엑셀 파일에서 추출하여, **과거 수납 이력(6개월 치)**과 대조해 **"발생월(귀속월)"을 판단**하고, 이를 바탕으로 **원무일보(Google Sheet)를 자동 생성** 및 **수납여부 대장에 자동 업데이트**하는 워크플로우입니다.

### 📌 Expected Outcome (목표)
1.  **발생월 자동 추적**: 결제 금액만 보고 "이 돈이 10월분인지, 11월분인지" 자동으로 판별.
2.  **미수금 시각화 (Aging)**: 현재 남은 미수금이 몇 개월 전부터 쌓인 악성 미수금인지(M-3, M-2...) 한눈에 파악.
3.  **수작업 제로**: 엑셀 업로드 한 번으로 일보 작성, 통계, 대장 정리가 끝남.

---

## Step 2: 최적의 시나리오 제안 (Logic Architecture)

### 1. Strict Input/Output Definition
정확한 데이터 필드 정의입니다. 이 규칙을 벗어난 파일은 처리되지 않습니다.

#### **Input Data (Source)**
1.  **닥터스 입원/외래 파일** (Excel)
    -   `수납일자`: `YYYY-MM-DD` 형식 (필수)
    -   `차트번호`: 환자 고유 ID (Key)
    -   `성명`/`수진자명`: 환자 이름
    -   `입금총액`: `1,230,000` (콤마 포함 문자열)
    -   `진료과목`: 내과, 정형외과 등
2.  **과거 수납여부 파일** (Google Sheets, 6개월치)
    -   파일명: `수납여부_YYYY년MM월`
    -   Key Column: `성명` (또는 `차트번호` 권장)
    -   Target Column: `수납여부` (`O`/`X`), `수납액`

#### **Output Data (Target)**
1.  **원무일보** (Google Sheet) - `원무일보_YYYY-MM-DD`
    -   **Sheet 1: 입원수입** (컬럼: 성명, 발생월, 내역, 입원수입, 신용카드, 계좌입금, 현금, 계)
    -   **Sheet 2: 외래수입** (컬럼: 성명, 발생월, 진료일자, 외래수입)
    -   **Sheet 3: 금액별 수납보고** (컬럼: 당월매출, 당월수납, 수납률, 미수금 Aging, 금일수납 Aging)
2.  **수납여부 대장 업데이트** (Google Sheet)
    -   해당 환자의 셀 배경색 `초록색` 변경 및 `O` 표시

### 2. Logic & Algorithm (Process Flow)

#### **Logic A. 발생월 판단 (Matching Logic)**
결제된 금액이 "어느 달 미수금"을 갚은 것인지 역추적합니다.
1.  **Trigger**: 전일(Yesterday) 결제 금액 `P` (예: 120만원) 확인.
2.  **Scan**: 과거 6개월치 `수납여부` 파일 조회.
    -   `M(금월)`: 60만원 미수여부 확인
    -   `M-1(전월)`: 60만원 미수여부 확인
    -   `M-2(전전월)`: ...
3.  **Match**:
    -   `IF (M 미수금 + M-1 미수금 == P)` → 발생월은 "금월, 전월"
    -   `ELSE IF (M-1 미수금 == P)` → 발생월은 "전월"
    -   `ELSE` → "미상(Mismatch)" 플래그 처리 및 근사치 매칭.

#### **Logic B. 미수금 Aging 분석**
현재 병원의 재정 건전성을 판단합니다.
1.  **Collect**: 전체 환자의 미수금 총액 집계.
2.  **Aging Buckets**:
    -   `M`: 이번 달 발생한 미수금
    -   `M-1`: 지난 달부터 밀린 돈
    -   `M-2`: 지지난 달부터 밀린 돈
    -   `M-3+`: 3개월 이상 장기 체납 (악성)
3.  **Report**: "현재 총 미수금 1억 중, 3개월 이상 악성 미수금은 2천만원(20%)입니다." 형태로 보고.

### 3. Workflow Diagram (ASCII)
```
[User] Upload File (Form)
      │
      ▼
[Node] Parse Excel & Filter 'Yesterday'
      │
      ├─(Parallel Scan)─▶ [Read] 수납여부_M
      ├─(Parallel Scan)─▶ [Read] 수납여부_M-1
      │       ...
      ├─(Parallel Scan)─▶ [Read] 수납여부_M-5
      │
      ▼
[Node] Logic: Match Payment to Month (Logic A)
      │
      ▼
[Node] Logic: Calculate Aging Stats (Logic B)
      │
      ▼
[Node] Create Report (Google Sheets)
      │
      └─▶ [Update] Color Marking (수납완료 처리)
```

## Step 3: 사용자 경험 (User Experience)
1.  **Action**: 원무과 직원은 매일 오후 2시, 카카오톡/슬랙으로 온 링크를 열어 엑셀 파일만 업로드합니다.
2.  **Notification**: "작성 완료! (처리건수: 45건, 매칭성공: 44건, 확인필요: 1건)" 메시지를 받습니다.
3.  **Result**: 구글 드라이브에 예쁘게 정리된 `원무일보` 시트를 열어 팀장님께 보고합니다.
