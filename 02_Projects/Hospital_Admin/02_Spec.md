---
description: 2단계 - n8n 노드 상세 설계 워크플로우 (Daily Report)
---

# Phase 2. Technical Specification (상세 설계) - Daily Report

## 1. Workflow Overview
이 문서는 **"일일 원무일보 자동화 (Daily Report)"** 워크플로우를 위한 상세 설계서입니다.
`WF02_Implementation.md`의 로직을 기반으로 하며, MCP로 검증된 파라미터 명칭을 사용합니다.

- **Trigger**: n8n Form (파일 직접 업로드 방식)
- **Logic**: 업로드된 엑셀 파싱 -> 전일 결제 필터링 -> 통계 산출 -> 구글 시트 작성
- **Goal**: 수작업 없는 100% 자동 일보 생성

## 2. Node Specifications (Developer-Ready)

### [Node 1] n8n Form Trigger (Trigger)
- **Node Type**: `n8n-nodes-base.formTrigger` (v2.5)
- **Role**: 사용자로부터 닥터스 엑셀 파일과 기준일자를 입력받음
- **Parameters**:
    - `formTitle`: `원무일보 데이터 업로드`
    - `formDescription`: `닥터스 입원/외래 데이터를 업로드하세요.`
    - `formFields`:
        - `values`:
            - `{ "label": "닥터스 데이터 파일", "name": "file", "type": "file", "required": true }`
            - `{ "label": "기준일자", "name": "targetDate", "type": "date", "required": true }`
            - `{ "label": "파일종류", "name": "fileType", "type": "dropdown", "options": [{"name": "입원", "value": "입원"}, {"name": "외래", "value": "외래"}] }`
    - `responseMode`: `onReceived` (Form Is Submitted)

### [Node 2] Set Variables (Set)
- **Node Type**: `n8n-nodes-base.set`
- **Role**: 전역 변수 설정 (폴더 및 템플릿 ID)
- **Parameters**:
    - `values`:
        - `templateId`: `(Google Sheet Template ID)`
        - `folderId`: `(Target Folder ID)`
        - `targetDate`: `{{ $json.targetDate }}`
        - `fileType`: `{{ $json.fileType }}`

### [Node 3] Check & Upload (Google Drive)
- **Node Type**: `n8n-nodes-base.googleDrive`
- **Role**: 업로드된 파일을 구글 드라이브에 백업
- **Credentials**: `googleDriveOAuth2Api`
- **Parameters**:
    - `resource`: `file`
    - `operation`: `upload`
    - `name`: `닥터스_{{ $('Set Variables').item.json.fileType }}_{{ $('Set Variables').item.json.targetDate }}.xlsx`
    - `parents`: `{{ ['Set Variables'].item.json.folderId }}` (List Type)
    - `binaryPropertyName`: `file`

### [Node 4] Parse Excel (Spreadsheet File)
- **Node Type**: `n8n-nodes-base.spreadsheetFile`
- **Role**: 엑셀 바이너리를 JSON으로 변환
- **Parameters**:
    - `operation`: `fromFile`
    - `binaryPropertyName`: `file`
    - `options`:
        - `readAsString`: `true` (날짜 포맷 이슈 방지)

### [Node 5] Filter Logic (Code)
- **Node Type**: `n8n-nodes-base.code`
- **Role**: 기준일자 및 금액(>0) 필터링
- **Parameters**:
    - `jsCode`:
        ```javascript
        const targetDate = $('Set Variables').item.json.targetDate;
        const fileType = $('Set Variables').item.json.fileType;
        const items = $input.all();
        const dailyPayments = [];
        
        for (const item of items) {
            const row = item.json;
            const date = row['수납일자'];
            // 쉼표 제거 후 숫자 변환
            const amount = parseFloat(String(row['입금총액'] || 0).replace(/,/g, ''));
            
            if (date === targetDate && amount > 0) {
                dailyPayments.push({
                    json: {
                        ...row,
                        amount: amount,
                        occurrenceType: fileType
                    }
                });
            }
        }
        return dailyPayments;
        ```

### [Node 6] Calculate Stats (Code)
- **Node Type**: `n8n-nodes-base.code`
- **Role**: 총 수납액 및 건수 집계
- **Parameters**:
    - `jsCode`:
        ```javascript
        const payments = $input.all().map(i => i.json);
        let totalCollection = 0;
        
        payments.forEach(p => {
            totalCollection += p.amount;
        });
        
        return [{
            json: {
                count: payments.length,
                totalCollection: totalCollection,
                payments: payments
            }
        }];
        ```

### [Node 7] Copy Template (Google Drive)
- **Node Type**: `n8n-nodes-base.googleDrive`
- **Role**: 원무일보 템플릿 복사하여 새 파일 생성
- **Parameters**:
    - `resource`: `file`
    - `operation`: `copy`
    - `fileId`: `{{ $('Set Variables').item.json.templateId }}`
    - `name`: `원무일보_{{ $('Set Variables').item.json.targetDate }}`
    - `parents`: `{{ ['Set Variables'].item.json.folderId }}`

### [Node 8] Write Payments (Google Sheets)
- **Node Type**: `n8n-nodes-base.googleSheets`
- **Role**: 필터링된 결제 내역을 시트에 기입
- **Credentials**: `googleSheetsOAuth2Api`
- **Parameters**:
    - `resource`: `sheet`
    - `operation`: `append`
    - `documentId`: `{{ $('Copy Template').item.json.id }}`
    - `sheetName`: `{{ $('Set Variables').item.json.fileType === '입원' ? '입원수입' : '외래수입' }}`
    - `options`:
        - `valueRenderMode`: `UNFORMATTED_VALUE`

### [Node 9] Update Summary (Google Sheets)
- **Node Type**: `n8n-nodes-base.googleSheets`
- **Role**: 일계표(요약) 시트 업데이트
- **Parameters**:
    - `resource`: `sheet`
    - `operation`: `update`
    - `documentId`: `{{ $('Copy Template').item.json.id }}`
    - `sheetName`: `일계표`
    - `range`: `B3` (예시 위치)
    - `values`:
        - `{{ $('Calculate Stats').item.json.count }}` (건수)
        - `{{ $('Calculate Stats').item.json.totalCollection }}` (총액)

## 3. Data Structure (Input/Output)
- **Input (from Form)**: `{ "file": [Binary], "targetDate": "2025-01-05", "fileType": "입원" }`
- **Output (Google Sheet)**:
    - `원무일보_2025-01-05` (File)
    - Sheets: `입원수입`, `외래수입`, `일계표`

## 4. Verification & Error Handling
- **Verification**:
    - `n8nFormTrigger`가 파일을 제대로 수신하는지 확인 (Binary Data Mode).
    - `spreadsheetFile`이 날짜를 String으로 잘 읽어오는지 확인.
- **Error Handling**:
    - 엑셀 파일 컬럼명(`수납일자`, `입금총액`)이 바뀌면 `Code` 노드에서 에러 발생함. -> **Column Name Validation** 로직 추가 권장.
