# 🔍 FlashAI v2.0 코드 설명

`index.html` 한 파일에 HTML(구조) + CSS(디자인) + JavaScript(기능)가 모두 들어있습니다.
아래는 핵심 코드를 부분별로 설명한 문서입니다.

---

## 1. API 키 입력 & 시작

```javascript
function startApp() {
  const key = document.getElementById('apiKeyInput').value.trim();
  if (!key.startsWith('sk-ant-')) {
    alert('올바른 Anthropic API 키를 입력해주세요 (sk-ant-... 형식)');
    return;
  }
  apiKey = key;
  // 입력 화면 숨기고 메인 앱 표시
}
```

앱을 열면 API 키 입력 화면이 먼저 뜹니다. `sk-ant-` 형식을 확인한 뒤
메인 화면으로 전환합니다. 키는 변수에만 저장되어 외부로 나가지 않습니다.

---

## 2. 입력 검증 (프로세스 적용 포인트)

```javascript
if (!text) { alert('텍스트를 입력해주세요'); return; }
if (text.length < 20) { alert('최소 20자 이상 입력해주세요'); return; }
if (text.length > 5000) { alert('5000자를 초과했습니다. 줄여주세요'); return; }
```

빈 입력, 너무 짧은 입력, 너무 긴 입력을 미리 걸러냅니다.
요구사항 분석 단계에서 도출한 엣지케이스 대응입니다.

---

## 3. AI 호출 + 자동 재시도 (품질 관리)

```javascript
for (let retry = 0; retry < 2 && !success; retry++) {
  const res = await fetch('https://api.anthropic.com/v1/messages', {
    method: 'POST',
    headers: {
      'x-api-key': apiKey,
      'anthropic-version': '2023-06-01',
      'anthropic-dangerous-direct-browser-access': 'true'
    },
    body: JSON.stringify({
      model: 'claude-sonnet-4-20250514',
      max_tokens: 1500,
      messages: [{ role: 'user', content: prompt }]
    })
  });
  // 응답 파싱 → 실패하면 한 번 더 시도
}
```

Claude API에 카드 생성을 요청합니다. JSON 파싱에 실패하면
최대 2회까지 재시도하여 안정성을 높였습니다.

---

## 4. 토큰 추적 & 비용 계산

```javascript
const PRICE_IN = 3 / 1000000;   // 입력 100만 토큰당 $3
const PRICE_OUT = 15 / 1000000; // 출력 100만 토큰당 $15

cumInput += inT;
cumOutput += outT;
const cost = cumInput * PRICE_IN + cumOutput * PRICE_OUT;
```

API 응답의 `usage` 값에서 입력/출력 토큰을 받아 누적하고,
Claude Sonnet 4 단가를 곱해 예상 비용을 실시간 계산합니다.

---

## 5. 생성 이력 로그 (프로세스 증빙)

```javascript
logs.push({
  no: attemptNo,
  time: startTime.toLocaleTimeString('ko-KR'),
  chars: text.length,
  cards: success ? cards.length : 0,
  inT, outT,
  status: success ? 'OK' : 'FAIL'
});
```

매 생성 시도마다 시각, 글자 수, 카드 수, 토큰, 성공 여부를
테이블에 기록합니다. 이 로그가 프로세스 적용의 증빙 자료가 됩니다.

---

## 6. 카드 뒤집기 (3D 애니메이션)

```css
.card-inner { transform-style: preserve-3d; transition: transform 0.45s; }
.card-inner.flipped { transform: rotateY(180deg); }
.card-face.back { transform: rotateY(180deg); }
```

CSS의 `rotateY`로 카드가 Y축을 기준으로 회전하며 뒤집힙니다.
클릭 시 `flipped` 클래스가 토글됩니다.

---

## 전체 흐름 요약

```
API 키 입력 → 텍스트 입력 → 입력 검증
   → Claude API 호출 (실패 시 재시도)
   → JSON 파싱 → 카드 렌더링
   → 토큰/비용 갱신 → 이력 로그 기록
```
