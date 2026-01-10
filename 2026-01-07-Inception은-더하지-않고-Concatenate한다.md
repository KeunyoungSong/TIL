# Inception은 더하지 않고 `concatenate`한다

> Type: Concept

## 상황/배경(Context)
GoogleLeNet(Inception)을 볼 때 “여러 conv 결과를 합친다”는 말을 듣고 element-wise sum(더하기)로 오해하기 쉽다. Inception의 핵심은 “더하기”가 아니라 “쌓기(concatenate)”다.

---

## 정의(Definition)
Inception 모듈은 여러 branch(예: `1×1`, `3×3`, `5×5`, pooling)를 **병렬로 적용**한 뒤, 결과를 **채널 축으로 `concatenate`** 해서 하나의 feature tensor로 만든다.

## 핵심 아이디어(Key Ideas)
- `concatenate`의 조건:
  - **공간 크기(H, W)가 같아야** 한다.
  - 채널 수(C)는 달라도 된다(오히려 다르게 설계한다).
- 그래서 Inception에서는 branch마다 **stride/padding을 “같은 H×W가 나오도록” 설계**한다.
  - 전형적으로 `stride=1` + `padding="same"` 조합을 쓴다.
- 결과는 “하나의 feature map(단일 채널)”이 아니라 “채널이 늘어난 feature tensor”다.

## 예시(Examples)
예를 들어 출력 공간 크기를 28×28로 맞춘 뒤 채널 축으로 쌓는다.
- branch1: `(28, 28, 32)`
- branch2: `(28, 28, 64)`
- branch3: `(28, 28, 16)`

`concatenate(axis=channel)` 결과:
- `(28, 28, 112)`

## 주의할 점(Caveats)
- Inception의 “합치기”는 `add`가 아니라 `concatenate`다.
  - `add`(element-wise sum)는 ResNet의 skip connection과 더 가깝다.
- 커널 크기가 다르면 기본적으로 출력 H×W가 달라지기 때문에, **padding/stride 설계가 없으면 concatenate 자체가 불가능**하다.

## 한 줄 요약(Summary)
Inception은 다양한 receptive field의 출력을 “더해 섞는” 게 아니라, 같은 H×W로 맞춘 다음 “채널 방향으로 쌓아” 풍부하게 만든다.

---

## 역사/배경(Timeline/Why)
- “어떤 커널 크기가 최선인지”는 사전에 고르기 어렵다 → 여러 스케일을 병렬로 두고 학습으로 선택하게 한다.
- 다만 큰 커널(예: `5×5`)은 연산량이 커서, `1×1` 같은 병목(bottleneck)으로 채널을 줄여 비용을 낮추는 설계가 같이 등장했다.
