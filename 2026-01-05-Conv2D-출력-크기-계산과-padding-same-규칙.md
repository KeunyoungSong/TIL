# Conv2D 출력 크기 계산과 padding="same" 규칙

> Type: Concept

## 상황/배경(Context)
Keras/TensorFlow에서 `Conv2D` 설정(`kernel_size`, `strides`, `padding`)을 바꿀 때 feature map 크기와 padding이 “어디에, 몇 칸” 들어가는지 정리했다.

---

## 정의(Definition)
`Conv2D`의 출력 공간 크기(H/W)는 “필터가 입력 위에 놓일 수 있는 위치 개수”로 결정되며, `padding="same"`은 특히 TensorFlow 규칙(출력 크기 우선)에 따라 필요한 padding을 계산·분배한다.

## 핵심 아이디어(Key Ideas)
- `valid`: 패딩 없음 → 커널이 가장자리 밖으로 나갈 수 없어서 공간 크기가 줄어든다.
- `same`: 출력 크기를 먼저 정한 뒤(특히 `stride > 1`에서) 그 출력을 만들기 위한 “최소 padding”을 역산한다.
- padding이 홀수로 필요하면 TensorFlow/Keras는 **오른쪽/아래쪽에 1칸 더** 준다(비대칭 padding).

## 기호 정리(Symbols)
- `N`: 입력 한 축의 크기(가로 또는 세로)
- `K`: 커널 크기(해당 축)
- `S`: stride(해당 축)
- `P`: 한쪽 기준 padding 크기(`valid`면 0)
- `O`: 출력 한 축의 크기
- `P_total`: 한 축에 필요한 전체 padding(좌+우 또는 상+하의 합)

## 원리/수식(Principle/Math)
### 1) `valid`(일반적인 공식)
한 축 기준:

$$
O=\left\lfloor \frac{N+2P-K}{S} \right\rfloor + 1
$$

`valid`는 `P=0`.

### 2) TensorFlow/Keras의 `same`(특히 `stride > 1`)
출력 크기를 먼저 정한다:

$$
O=\left\lceil \frac{N}{S} \right\rceil
$$

이 출력을 만들기 위한 전체 padding을 역산한다:

$$
P_{total}=(O-1)\cdot S + K - N
$$

분배 규칙(한 축):
- left/top: $\left\lfloor \frac{P_{total}}{2} \right\rfloor$
- right/bottom: $\left\lceil \frac{P_{total}}{2} \right\rceil$

## 예시(Examples)
### 예시 1) `kernel=3`, `stride=1`, `padding="valid"`
- `N=32`, `K=3`, `S=1`, `P=0`
- $O=\lfloor (32-3)/1 \rfloor + 1 = 30$
- 출력: `30×30` (채널은 `filters`로 결정)

### 예시 2) `kernel=3`, `stride=1`, `padding="same"`
- `N=32`, `S=1`이면 $O=\lceil 32/1 \rceil = 32$
- 이때는 보통 좌/우, 상/하가 `1+1`로 대칭이 된다(`K=3`일 때).

### 예시 3) `kernel=3`, `stride=2`, `padding="same"` (비대칭이 생기는 대표 케이스)
- `N=32`, `K=3`, `S=2`
- $O=\lceil 32/2 \rceil = 16$
- $P_{total}=(16-1)\cdot 2 + 3 - 32 = 1$
- 분배:
  - 좌/위: $\lfloor 1/2 \rfloor = 0$
  - 우/아래: $\lceil 1/2 \rceil = 1$
- 결론: **오른쪽 1, 아래 1만 padding**이 들어갈 수 있다(좌/위는 0).

## 주의할 점(Caveats)
- `padding="same"`은 “항상 입력과 출력이 같은 크기”가 아니라, **`stride=1`일 때만** 같은 크기가 된다.
- `same`은 “대칭 padding”이 아니라, **출력 크기 우선 규칙**이다(나머지가 생기면 우/아래로 치우칠 수 있음).
- H/W는 위 규칙으로 계산하고, 채널 수는 `filters`로 결정된다.

## 한 줄 요약(Summary)
`same`은 “크기 유지”가 아니라 “출력 크기(`ceil(N/S)`)를 먼저 고정하고 필요한 padding을 최소로 넣는 규칙”이며, 홀수 padding이면 우/아래로 1칸 더 간다.

