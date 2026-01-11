# WF02 원무일보 - 노드별 구현 가이드 (폼 트리거 방식)

이 문서를 보며 n8n에서 직접 노드를 하나씩 추가하고 설정하세요.

> **변경사항**: Schedule Trigger → **Form Trigger**로 변경
> 사용자가 폼에서 파일을 업로드하면 워크플로우가 시작됩니다.

---

## 노드 1: n8n Form Trigger (파일 업로드 폼)

| 설정 항목 | 값 |
|-----------|-----|
| **노드 타입** | n8n Form Trigger |
| **Form Title** | 원무일보 데이터 업로드 |
| **Form Description** | 닥터스 입원/외래 데이터를 업로드하세요 |
| **Respond When** | Form Is Submitted |

**Form Fields:**

| Field Type | Label | Required |
|------------|-------|----------|
| File | 닥터스 데이터 파일 | Yes |
| Date | 기준일자 | Yes |
| Dropdown | 파일종류 (Options: 입원, 외래) | Yes |

> **💡 사용법**: 워크플로우 활성화 후, **Production URL**을 사용자에게 공유하세요.

---

## 노드 2: Set (설정 변수)

| 설정 항목 | 값 |
|-----------|-----|
| **노드 타입** | Set |

**Values to Set:**
| Name | Type | Value |
|------|------|-------|
| templateId | String | `여기에_원무일보_템플릿_ID_입력` |
| folderId | String | `여기에_저장할_폴더_ID_입력` |
| targetDate | String | `{{ $json['기준일자'] }}` |
| fileType | String | `{{ $json['파일종류'] }}` |

---

## 노드 3: Google Drive (업로드된 파일 저장)

| 설정 항목 | 값 |
|-----------|-----|
| **노드 타입** | Google Drive |
| **Resource** | File |
| **Operation** | Upload |
| **File Name** | `닥터스_{{ $('Set').item.json.fileType }}_{{ $('Set').item.json.targetDate }}.xlsx` |
| **Parents** | `{{ $('Set').item.json.folderId }}` |

---

## 노드 4: Spreadsheet File (엑셀 추출)

| 설정 항목 | 값 |
|-----------|-----|
| **노드 타입** | Spreadsheet File |
| **Operation** | Read from File |

---

## 노드 5: Code (전일 결제 필터링)

| 설정 항목 | 값 |
|-----------|-----|
| **노드 타입** | Code |
| **Language** | JavaScript |

**JavaScript Code:**

```javascript
const targetDate = $('Set').item.json.targetDate;
const fileType = $('Set').item.json.fileType;
const items = $input.all();
const dailyPayments = [];

for (const item of items) {
  const row = item.json;
  const date = row['수납일자'];
  const amountStr = String(row['입금총액'] || 0).replace(/,/g, '');
  const amount = parseFloat(amountStr);

  // 기준일자와 일치하고 금액이 있는 경우만 필터링
  if (date === targetDate && amount > 0) {
    dailyPayments.push({ 
      json: {
        ...row, 
        amount: amount,
        occurrenceType: fileType // 폼에서 선택한 파일 종류 사용
      }
    });
  }
}
return dailyPayments;
```

---

## 노드 6: Code (통계 계산)

| 설정 항목 | 값 |
|-----------|-----|
| **노드 타입** | Code |
| **Language** | JavaScript |

**JavaScript Code:**

```javascript
const payments = $input.all().map(i => i.json);
const fileType = $('Set').item.json.fileType;

let collection = 0;
const dataList = [];

payments.forEach(p => {
  collection += p.amount;
  dataList.push({
    성명: p['성명'] || p['수진자명'],
    발생월: '당월',
    금액: p.amount
  });
});

return [{
  json: {
    fileType: fileType,
    stats: {
      collection: collection,
      count: dataList.length
    },
    dataList: dataList
  }
}];
```

---

## 노드 7: Google Drive (템플릿 복사)

| 설정 항목 | 값 |
|-----------|-----|
| **노드 타입** | Google Drive |
| **Resource** | File |
| **Operation** | Copy |
| **File ID** | `{{ $('Set').item.json.templateId }}` |
| **Name** | `원무일보_{{ $('Set').item.json.targetDate }}` |
| **Parents** | `{{ $('Set').item.json.folderId }}` |

---

## 노드 8: Code (데이터 분리)

| 설정 항목 | 값 |
|-----------|-----|
| **노드 타입** | Code |
| **Language** | JavaScript |

**JavaScript Code:**

```javascript
const data = $('Code1').first().json;
const fileId = $input.first().json.id;

return data.dataList.map(p => ({
  json: {
    spreadsheetId: fileId,
    sheetName: data.fileType === '입원' ? '입원수입' : '외래수입',
    성명: p.성명,
    발생월: p.발생월,
    금액: p.금액
  }
}));
```

---

## 노드 9: Google Sheets (데이터 입력)

| 설정 항목 | 값 |
|-----------|-----|
| **노드 타입** | Google Sheets |
| **Resource** | Sheet Within Document |
| **Operation** | Append Row |
| **Document** | By ID: `{{ $json.spreadsheetId }}` |
| **Sheet** | `{{ $json.sheetName }}` |
| **Data Mode** | Map Each Column Manually |

**Column Mapping:**
| Column | Value |
|--------|-------|
| A | `{{ $json.성명 }}` |
| B | `{{ $json.발생월 }}` |
| C | `{{ $json.금액 }}` |

---

## 노드 10: Google Sheets (일계표 업데이트)

| 설정 항목 | 값 |
|-----------|-----|
| **노드 타입** | Google Sheets |
| **Resource** | Sheet Within Document |
| **Operation** | Update Row |
| **Document** | By ID: `{{ $('Google Drive1').first().json.id }}` |
| **Sheet** | 일계표 |
| **Range** | B3 |
| **Data Mode** | Map Each Column Manually |

**Column Mapping:**
| Column | Value |
|--------|-------|
| A | `{{ $('Code1').first().json.stats.count }}` |
| B | `{{ $('Code1').first().json.stats.collection }}` |

---

## 노드 연결 순서

```
[1] n8n Form Trigger (파일 업로드)
    ↓
[2] Set (설정)
    ↓
[3] Google Drive (Upload)
    ↓
[4] Spreadsheet File (추출)
    ↓
[5] Code (필터링)
    ↓
[6] Code (통계)
    ↓
[7] Google Drive (Copy)
    ↓
[8] Code (데이터 분리)
    ↓
[9] Google Sheets (Append)
    ↓
[10] Google Sheets (Update)
```

---

## 사용 방법

1. 워크플로우를 **Active** 상태로 전환
2. Form Trigger 노드에서 **Production URL** 복사
3. 해당 URL을 브라우저에서 열면 업로드 폼 표시
4. **파일 선택** + **기준일자** + **파일종류** 입력 후 제출
5. 자동으로 워크플로우 실행

---

## 사전 준비사항

1. **원무일보 템플릿 시트** 생성 필요
   - 시트 3개: `입원수입`, `외래수입`, `일계표`
   - 각 시트에 헤더 미리 작성

2. **Google 계정 연동**
   - 모든 Google 노드에 동일한 Credential 선택
