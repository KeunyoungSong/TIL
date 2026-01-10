# ResNet Bottleneck: `1×1 Conv`로 채널 축소해 파라미터 줄이기

> Type: Concept

## 상황/배경(Context)
ResNet-50 이상에서 자주 보이는 bottleneck 블록(`1×1 → 3×3 → 1×1`)을 보면서, 왜 `1×1 Conv`가 “파라미터를 줄인다”는 말이 나오는지(그리고 shortcut에 `1×1`을 쓰는 경우는 왜 다른지) 한 번에 고정하고 싶었다.

---

## 정의(Definition)
Conv 레이어의 파라미터 수는 (bias 제외 시) 아래로 결정된다.

$$
\#params = k_h \times k_w \times C_{in} \times C_{out}
$$

ResNet bottleneck의 핵심은 **연산/파라미터가 비싼 `3×3 Conv`의 입력/출력 채널을 `1×1 Conv`로 먼저 줄였다가 다시 복원**하는 것이다.

## 핵심 아이디어(Key Ideas)
- `1×1 Conv`가 “공간 이웃”을 보지는 않지만, **채널 간 선형 결합(channel mixing)** 을 한다.
- `3×3 Conv`는 **spatial pattern**을 담당하고, `1×1 Conv`는 **채널 차원 관리(compression/expansion)** 를 담당한다.
- bottleneck에서 앞 `1×1`의 목적은 **compression(채널 축소)**, 뒤 `1×1`의 목적은 **expansion(채널 복원 + shortcut과 더하기 정합)** 이다.
- shortcut에 `1×1`이 쓰이는 경우도 있는데, 이건 보통 **파라미터 절감이 아니라 dim matching(projection shortcut)** 목적이다(채널 수/stride가 달라질 때).

## 예시(Examples)
### 1) 채널 축소가 왜 파라미터 감소로 이어지나
입력/출력 채널이 모두 256인 `3×3 Conv` 하나를 그냥 쓰면:

$$
3 \times 3 \times 256 \times 256 = 589{,}824
$$

같은 “256 채널로 돌아오는” 블록을 bottleneck으로 만들고, 중간 채널을 64로 줄이면:
- `1×1`: `256 → 64`
- `3×3`: `64 → 64`
- `1×1`: `64 → 256`

파라미터는 각각:

$$
1 \times 1 \times 256 \times 64 = 16{,}384
$$

$$
3 \times 3 \times 64 \times 64 = 36{,}864
$$

$$
1 \times 1 \times 64 \times 256 = 16{,}384
$$

총합:

$$
16{,}384 + 36{,}864 + 16{,}384 = 69{,}632
$$

즉 `589,824 → 69,632`로 약 8.5배 감소한다(단순히 “줄인다”가 아니라 **비싼 `3×3`의 채널 폭을 줄인 효과**).

### 2) ResNet bottleneck 블록을 텍스트로 그리면
```
input (C)
  ├─ 1×1 Conv: C → C/4   (compression)
  ├─ 3×3 Conv: C/4 → C/4 (spatial)
  └─ 1×1 Conv: C/4 → C   (expansion)
+ shortcut (identity or projection)
```

### 3) “`1×1`은 공간 정보를 못 본다”의 정확한 의미
- 맞는 말: `1×1`은 이웃 픽셀을 보지 않는다.
- 하지만 중요한 기능이 남는다: 같은 위치 `(i, j)`에서 **채널들을 섞어 새로운 채널 표현을 만든다**.
  - 그래서 `3×3`이 뽑아야 할 “입력 채널 폭” 자체를 `1×1`로 재설계할 수 있다.

## 주의할 점(Caveats)
- shortcut에 `1×1 Conv`가 들어가는 경우(Projection shortcut):
  - 보통 `stride`로 해상도를 줄이거나, 채널 수를 바꿔서 **덧셈이 가능하도록 차원을 맞추기 위해** 쓴다.
  - 이때의 목적은 “compression”이 아니라 **dim matching**이다.

## 한 줄 요약(Summary)
ResNet bottleneck에서 `1×1 Conv`는 `3×3 Conv`의 채널 폭을 앞뒤에서 줄였다가 늘려서 파라미터/연산량을 크게 낮추는 장치이고, shortcut의 `1×1`은 주로 차원 정합을 위한 projection이다.

---

## 용어(Glossary)
- **bottleneck**: 중간 채널을 줄여 비용을 낮추는 구조.
- **projection shortcut**: shortcut 경로에 `1×1 Conv`(보통 stride 포함)를 두어 차원을 맞추는 방식.
