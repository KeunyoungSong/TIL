# FLOPs와 FLOPS 차이, Conv에서의 계산법

> Type: Concept

## 상황/배경(Context)
모델을 비교할 때 “FLOPs가 얼마냐”를 자주 보는데, FLOPs를 속도처럼 해석하거나 FLOPs/FLOPS를 섞어 쓰는 경우가 많다. Conv에서 FLOPs를 어떻게 세는지도 함께 고정해 둔다.

---

## 정의(Definition)
- **FLOPs**(Floating Point Operations): 한 번의 forward/inference에서 수행되는 **부동소수점 연산의 총 개수**(정적 지표).
- **FLOPS**(Floating Point Operations Per Second): 하드웨어가 초당 수행할 수 있는 **부동소수점 연산 성능**(속도/성능 지표).

## 핵심 아이디어(Key Ideas)
- FLOPs는 “얼마나 빠른가”가 아니라 **“얼마나 많이 계산해야 하는가”**를 뜻한다.
- 파라미터 수는 모델 크기(저장되는 `W`)에 가깝고, FLOPs는 한 번 실행할 때의 **추론 비용**에 더 가깝다.
- FLOPs가 낮아도 실제 latency가 반드시 낮아지는 것은 아니다(메모리/병렬화/커널 최적화 영향).

## 예시(Examples)
### 1) FLOPs vs FLOPS 구분 예
- 모델 A: `4 GFLOPs` (한 번 실행에 4G ops 필요)
- GPU: `20 TFLOPS` (초당 20T ops 처리 가능 “이론치”)

이론상 계산만 보면 빠를 수 있지만, 실제 속도는 메모리/병렬화 효율에 크게 좌우된다.

### 2) Conv FLOPs 기본식(자주 쓰는 근사)
Conv가 FLOPs의 대부분을 차지하는 경우가 많아, 보통 아래 형태로 잡는다.

$$
FLOPs \approx 2 \times H \times W \times C_{out} \times (k_h \times k_w \times C_{in})
$$

- `2`: multiply + add(MAC을 2 ops로 세는 관행)
- `H×W`: 출력 feature map의 공간 크기
- `C_in`, `C_out`: 입력/출력 채널 수
- `k_h×k_w`: 커널 크기

### 3) 예시 계산
입력: `224×224×64`, 커널: `3×3`, 출력 채널: `128`이면:

$$
2 \times 224 \times 224 \times 128 \times (3 \times 3 \times 64)
$$

→ 대략 `7.4 GFLOPs` 수준으로 잡을 수 있다.

### 4) 왜 `1×1 Conv`가 FLOPs를 줄이나(ResNet 맥락)
`1×1 Conv`는 `k_h×k_w = 1`이라:

$$
FLOPs \approx 2 \times H \times W \times C_{out} \times C_{in}
$$

큰 커널(`3×3`) 앞에서 채널을 줄이면, 이후 `3×3`의 `C_in`/`C_out`가 작아져 FLOPs가 크게 감소한다(ResNet bottleneck의 핵심 직관).

## 주의할 점(Caveats)
- FLOPs는 계산량 proxy라서, 실제 속도는 아래 요인에 의해 달라질 수 있다:
  - 메모리 접근/대역폭, 캐시 미스
  - 병렬화 효율(특히 작은 텐서/채널에서)
  - 하드웨어 커널 최적화 여부
- 예: depthwise conv는 FLOPs는 적어도 GPU에서 느리게 나오는 경우가 있다.

## 한 줄 요약(Summary)
FLOPs는 “한 번 실행에 필요한 연산량”, FLOPS는 “초당 처리 성능”이고, Conv FLOPs는 보통 `2×H×W×C_out×(k_h×k_w×C_in)` 근사로 비교한다.

---

## 용어(Glossary)
- **MAC**: Multiply–Accumulate(곱셈 1 + 덧셈 1로 세어 2 FLOPs로 환산하는 경우가 많음).
