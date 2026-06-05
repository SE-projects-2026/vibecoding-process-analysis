# 설계 — FlashAI v2.0

## 구조
단일 HTML 파일 (HTML + CSS + JS 인라인)
외부 의존성: Claude API (api.anthropic.com/v1/messages)

## v1.0 → v2.0 변경
- API 키 입력 화면 추가 (최초 진입 시)
- 옵션 선택 추가 (카드 수, 난이도)
- 토큰 대시보드 추가 (입력/출력/누적/비용)
- 생성 이력 로그 테이블 추가
- 재시도 로직 추가 (최대 2회)

## 프롬프트 설계
"{difficulty} 난이도의 Q&A 플래시카드를 정확히 {numCards}개 만들어줘"
→ v1.0 대비 난이도·개수 명시로 출력 품질 향상
