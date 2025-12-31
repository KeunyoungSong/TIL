# Webhook: Confluence → Make 주체 정리

> Type: Concept

## 상황/배경(Context)
Confluence 이벤트를 Make 자동화로 연결하려고 할 때 “웹훅을 누가 거는지/누가 쏘는지”가 헷갈리기 쉽다.  
웹훅을 “약속(등록)”과 “실행(전송)”으로 분리해서 정리한다.

---

## 정의(Definition)
**webhook**은 특정 이벤트가 발생했을 때, 미리 등록된 URL(웹 엔드포인트)로 HTTP 요청(보통 `POST`)을 보내는 **이벤트 기반 콜백** 방식이다.

## 핵심 아이디어(Key Ideas)
- webhook은 `hook(이벤트에 걸어두는 트리거)` + `web(HTTP로 호출)`의 합성어다.
- “내가 웹훅을 걸었다”는 말은 보통 **등록/설정했다**는 뜻이고, 실제 요청을 **전송하는 주체는 이벤트를 가진 서비스**다.
- Confluence → Make 조합에서 webhook은 “Confluence 이벤트가 발생하면 Make의 webhook URL로 payload를 보내준다”로 요약된다.

## 역사/배경(Timeline/Why)
- polling(주기적으로 API 조회)은 지연/낭비가 생기기 쉬워서, 이벤트가 날 때만 보내는 push 방식(webhook)이 널리 쓰였다.
- 이후 자동화 서비스(Zapier, Make 등)가 “URL 하나로 이벤트를 받는” 패턴을 표준 워크플로우로 만들면서 더 보편화됐다.

## 도식(Diagram)
```
너(설정/운영) ──(웹훅 등록)──> Confluence

Confluence (producer / sender)
  └─ 이벤트 발생(page created/updated 등)
      └─ HTTP POST + payload ──> Make webhook URL (consumer / receiver)
                                   └─ 시나리오 실행
```

## 원리/수식(Principle/Math)
핵심은 “이벤트 → HTTP 요청”의 매핑이다.
- 이벤트: `page updated`
- 대상: `https://hook.make.com/...`
- 전송: `POST` + JSON(payload) + 헤더(인증/검증 정보)

## 예시(Examples)
### Confluence webhook을 Make에 “걸어뒀다”의 의미
- **네가 한 일(운영 주체)**: Confluence 설정에서 webhook을 만들고, 대상 URL을 Make에서 제공한 webhook URL로 지정했다.
- **Confluence가 하는 일(발신자)**: 이벤트가 실제로 발생할 때마다 Make URL로 HTTP 요청을 전송한다.
- **Make가 하는 일(수신자)**: 그 요청을 받아 시나리오를 시작한다.

## 주의할 점(Caveats)
- **인증/검증**: 누구나 URL로 요청을 보낼 수 있으니, 토큰/서명/헤더 등을 이용해 “진짜 Confluence에서 온 요청인지”를 확인해야 한다.
- **재시도/중복**: 네트워크 오류로 같은 이벤트가 여러 번 전송될 수 있으니, 가능한 한 멱등(idempotent)하게 처리한다.
- **지연/순서**: 이벤트 순서가 항상 보장되진 않는다(특히 재시도 상황).
- **payload 계약**: “어떤 이벤트에 어떤 필드가 오는지”는 Confluence 쪽 문서/설정에 의존한다.

## 한 줄 요약(Summary)
Confluence → Make에서 “웹훅을 걸었다”는 **Confluence에 Make URL을 등록**했다는 뜻이고, 이벤트가 나면 **Confluence가 요청을 보내고 Make가 받는다**.

## 참고(Links)
- https://en.wikipedia.org/wiki/Webhook
