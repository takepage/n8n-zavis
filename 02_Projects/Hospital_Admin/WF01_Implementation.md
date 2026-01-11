# WF01 월별 수납여부 - 완전 구현 가이드

이 문서를 보며 n8n에서 직접 노드를 하나씩 추가하고 설정하세요.

> **워크플로우 구조**:
> ```
> [Form] → [Drive Upload] → [Drive Download] → [Extract] → [Code] → [Convert to XLSX] → [Drive Upload]
> ```

---

## 사전 준비
1. **닥터스 데이터 폴더** 생성 (Folder ID 메모)
2. **결과물 저장 폴더** 생성 (Folder ID 메모)
3. Google Drive 자격 증명 설정

---

## 노드 1: n8n Form Trigger

| 설정 항목 | 값 |
|-----------|-----|
| **노드 타입** | n8n Form Trigger (v2.5) |
| **Form Title** | `수납여부 데이터 업로드` |
| **Form Description** | `입원수납 엑셀 파일을 업로드하세요` |

**Form Fields 설정:**

| Field Type | Label | Required |
|------------|-------|----------|
| File | `입원수납 파일` | Yes |

---

## 노드 2: Google Drive (원본 파일 보관)

| 설정 항목 | 값 |
|-----------|-----|
| **노드 타입** | Google Drive |
| **Resource** | File |
| **Operation** | Upload |
| **Input Data Field Name** | `data` |
| **File Name** | `{{ $json['입원수납 파일'].filename }}` |
| **Parents** | `원본_데이터_폴더_ID_입력` |

---

## 노드 3: Google Drive (파일 불러오기)

| 설정 항목 | 값 |
|-----------|-----|
| **노드 타입** | Google Drive |
| **Resource** | File |
| **Operation** | Download |
| **File** | By ID |
| **File ID** | `{{ $json.id }}` |

---

## 노드 4: Extract from File (엑셀 읽기)

| 설정 항목 | 값 |
|-----------|-----|
| **노드 타입** | Extract from File (v1.1) |
| **Operation** | Extract from XLSX |
| **Input Binary Field** | `data` |

## 노드 5: Code (미수금 통합 계산)

| 설정 항목 | 값 |
|-----------|-----|
| **노드 타입** | Code (v2) |
| **Language** | JavaScript |
| **Mode** | Run Once for All Items |

> ⚠️ **참고**: n8n Cloud에서는 ExcelJS 같은 외부 모듈 사용이 제한됩니다.
> Self-hosted 환경에서는 `NODE_FUNCTION_ALLOW_EXTERNAL=exceljs` 환경변수 설정 후 스타일링 가능.

**JavaScript Code:**

```javascript
// 모든 데이터 가져오기
const items = $input.all();

// 금액 파싱 함수
const parseAmount = (v) => {
  if (v === null || v === undefined || v === '') return 0;
  const cleaned = String(v).replace(/,/g, '').trim();
  const num = parseFloat(cleaned);
  return isNaN(num) ? 0 : num;
};

// 차트번호별 데이터 수집
const patientMap = new Map();

items.forEach(item => {
  const row = item.json;
  const chartNo = row['차트번호'];
  const recordType = row['종류'] || '';
  
  if (!chartNo) return;
  
  if (!patientMap.has(chartNo)) {
    patientMap.set(chartNo, {
      chartNo: chartNo,
      name: row['성명'] || '',
      총_미수금: 0,
      총_입금총액: 0
    });
  }
  
  const patient = patientMap.get(chartNo);
  
  if (recordType === '중간수납' || recordType === '퇴원수납') {
    patient.총_미수금 += parseAmount(row['미수금']);
  }
  
  if (recordType === '미수입금') {
    patient.총_입금총액 += parseAmount(row['입금총액']);
  }
});

// 결과 출력
const results = [];
patientMap.forEach(patient => {
  const finalBalance = patient.총_미수금 - patient.총_입금총액;
  if (finalBalance !== 0) {
    results.push({
      json: {
        '차트번호': patient.chartNo,
        '성명': patient.name,
        '미수금': Math.round(finalBalance).toLocaleString('ko-KR') + '원'
      }
    });
  }
});

return results;
```

---

## 노드 6: Convert to File (XLSX 생성)

| 설정 항목 | 값 |
|-----------|-----|
| **노드 타입** | Convert to File (v1.1) |
| **Operation** | Convert to XLSX |
| **Put Output File in Field** | `data` |

---

## 노드 7: Google Drive (결과 파일 업로드)

| 설정 항목 | 값 |
|-----------|-----|
| **노드 타입** | Google Drive |
| **Resource** | File |
| **Operation** | Upload |
| **Input Data Field Name** | `data` |
| **File Name** | `{{ $now.format('MM') }}월_미수금현황.xlsx` |
| **Parents** | `결과물_폴더_ID_입력` |

> **결과**: 매월 실행 시 `12월_미수금현황.xlsx` 같은 파일이 생성됩니다.

---

## 노드 연결 다이어그램

```
[1] Form Trigger
    ↓
[2] Drive Upload (원본 보관)
    ↓
[3] Drive Download
    ↓
[4] Extract from File
    ↓
[5] Code (미수금 계산)
    ↓
[6] Convert to File (XLSX)
    ↓
[7] Drive Upload (결과물)
```

---

## 출력 파일 예시

| 차트번호 | 성명 | 미수금 |
|----------|------|--------|
| 10083 | 강태홍 | 2,500원 |
| 15570 | 고남석 | 1,752,710원 |
| 15183 | 김문영 | -29,690원 |

> **음수 미수금**: 과납된 금액 (환불 또는 다음 청구에 적용)

