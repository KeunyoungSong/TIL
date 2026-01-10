# EfficientNet `Compound Scaling` 직관과 사용법

> Type: Concept

## 상황/배경(Context)
EfficientNet의 `Compound Scaling` 슬라이드를 처음 보면 수식이 많아 “이게 학습 중에 모델 안에서 동작하는 뭔가인가?”처럼 느껴진다. 그런데 이건 모델 내부 메커니즘이 아니라, **모델을 만들기 전에** 적용하는 “스케일링 설계 규칙”이다.

---

## 정의(Definition)
모델 성능을 키우는 축은 보통 3가지다.
- **depth**: 레이어 수(깊이)
- **width**: 채널 수(너비)
- **resolution**: 입력 해상도

EfficientNet의 `Compound Scaling`은 이 3가지를 **한 번에, 일정 비율로 같이 키우는 규칙**이다.

## 핵심 아이디어(Key Ideas)
- 한 가지만 키우면 비효율적일 수 있다. (depth만, width만, resolution만)
- FLOPs 관점에서 대략적인 증가율은 이렇게 잡을 수 있다.
  - depth를 `d`배 → FLOPs `× d`
  - width를 `w`배 → FLOPs `× w^2` (대략 `C_in×C_out`가 같이 커짐)
  - resolution을 `r`배 → FLOPs `× r^2` (대략 `H×W`가 커짐)
- 그래서 EfficientNet은 “스케일 단계”를 올릴 때 FLOPs 예산이 일정 규칙으로 늘도록(예: 단계당 약 2배) 비율을 맞춘다.

## 예시(Examples)
### 1) 왜 `width`, `resolution`이 2배면 FLOPs가 4배인가
- 해상도 2배: `H×W`가 `2×2=4`배 → 연산량도 대략 4배
- 채널 2배: `C_in×C_out`이 `2×2=4`배 → 연산량도 대략 4배
- 깊이 2배: 블록 반복이 2배 → 연산량도 대략 2배

### 2) 슬라이드의 수식이 의미하는 것(직관)
EfficientNet은 스케일 레벨을 `φ(phi)`로 두고, 다음처럼 모델 크기를 정한다.

$$
depth = \alpha^\phi,\quad width = \beta^\phi,\quad resolution = \gamma^\phi
$$

그리고 “스케일 레벨을 1 올릴 때 FLOPs가 약 2배”가 되도록 비율을 고른다.

$$
\alpha \cdot \beta^2 \cdot \gamma^2 \approx 2
$$

여기서:
- `α`: depth 증가 비율
- `β`: width 증가 비율
- `γ`: resolution 증가 비율
- `φ`: 모델 크기 등급(예: B0, B1, …)

### 3) 제일 중요한 포인트: 이건 “런타임 로직”이 아니다
`Compound Scaling`은 다음 중 어느 것도 아니다.
- forward/backward 때 동적으로 켜지는 로직
- gradient로 학습되는 파라미터
- 학습 중 자동으로 depth/width/resolution이 바뀌는 메커니즘

정확히는:
- (연구자가) **베이스 모델(B0)** 을 만든 뒤,
- (연구자가) `α, β, γ`를 한 번 찾아 고정해두고,
- (사용자가) `φ`(=B0/B1/B2…)를 선택해 **서로 다른 크기의 “별도 모델”** 을 생성해서 학습한다.

즉 “B3를 쓴다”는 말은, 학습 중에 B0가 커지는 게 아니라 **B3 구조로 만들어진 모델을 처음부터 학습**한다는 뜻이다.

## 주의할 점(Caveats)
- FLOPs는 속도 자체가 아니라 “계산량 지표”다. FLOPs가 줄어도 실제 latency는 메모리/병렬화/커널 최적화에 따라 달라질 수 있다.
- `α, β, γ` 값은 “모든 모델에 보편적으로 최선”이라기보다, 특정 설계(베이스 네트워크/블록)에 대해 효율적이었던 경험적 결과다.

## 한 줄 요약(Summary)
EfficientNet의 `Compound Scaling`은 학습 중에 작동하는 알고리즘이 아니라, depth/width/resolution을 FLOPs 관점에서 균형 있게 키우도록 **모델을 설계할 때 미리 적용하는 스케일링 규칙**이다.

---

## 역사/배경(Timeline/Why)
- 모델을 “크게 만들기”는 오래된 성능 향상 전략이지만, depth/width/resolution 중 하나만 키우면 효율이 무너질 수 있다.
- EfficientNet은 “같은 계산 예산에서 더 잘 커지는 방법”을 목표로, 세 축을 함께 스케일링하는 규칙을 제안했다.
- 이후 많은 아키텍처에서 “스케일링 레시피”를 명시적으로 다루는 흐름이 강해졌다.

## 참고(Links)
- https://arxiv.org/abs/1905.11946
