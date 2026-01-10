# ResNet Shortcut 합치는 연산과 차원 정합 조건

> Type: Concept

## 상황/배경(Context)
ResNet에서 main path 출력과 shortcut(지름길) 출력을 “어떻게 합치는지”, 그리고 왜 자꾸 `1×1 Conv`(projection shortcut)가 등장하는지 헷갈렸다. 결론은 합치는 연산 자체는 단순하지만, **shape 조건이 엄격**하다.

---

## 정의(Definition)
ResNet 블록의 출력은 보통 아래처럼 정의된다.

$$
y = F(x) + x
$$

여기서 `+`는 `concatenate`가 아니라 **element-wise addition(요소별 덧셈)** 이다.

## 핵심 아이디어(Key Ideas)
- element-wise addition은 “같은 인덱스끼리 더하는 연산”이라 두 텐서의 shape이 완전히 같아야 한다.
  - `[N, C, H, W] + [N, C, H, W]`
- shape이 다르면 “어떤 채널/어떤 위치끼리 더할지”가 정의되지 않는다.
- ResNet은 그래서 shortcut 경로를 두 가지로 쓴다:
  - **Identity shortcut**: 아무 연산 없이 `x`를 그대로 더함
  - **Projection shortcut**: `1×1 Conv`(필요 시 stride 포함)로 `x`의 차원을 맞춘 뒤 더함

## 예시(Examples)
### 1) 합치는 연산 자체(요소별 덧셈)
개념적으로는 아래와 같다.

`y[n, c, h, w] = F(x)[n, c, h, w] + shortcut(x)[n, c, h, w]`

### 2) Identity shortcut이 가능한 경우
- `stride = 1`
- `C_in = C_out`
- `H, W`가 유지되는 블록

이때 shortcut은 그냥 `x`이고, `y = F(x) + x`가 바로 성립한다.

### 3) Projection shortcut이 필요한 경우
다음 중 하나라도 깨지면 identity로는 더할 수 없다.
- 채널 수가 바뀜(예: `64 → 256`, bottleneck에서 흔함)
- spatial size가 바뀜(예: downsampling으로 `H, W`가 줄어듦, 보통 `stride=2`)

이때는 shortcut에 `1×1 Conv`를 둬서:

$$
y = F(x) + W_s x
$$

처럼 **차원 정합(dim matching)** 을 만든 뒤 더한다.

## 주의할 점(Caveats)
- shortcut의 `1×1 Conv`는 “파라미터 절감”이 목적이 아니라, 주로 **덧셈을 가능하게 하는 shape 맞추기(projection)** 목적이다.
- 프레임워크 수준에서 broadcasting으로 강제로 더할 수는 있어도, ResNet의 residual 해석(“같은 표현 공간에서의 보정”)을 깨기 쉬워 보통 의도적으로 쓰지 않는다.

## 한 줄 요약(Summary)
ResNet에서 shortcut과 main path는 **element-wise add**로 합치며, 이를 위해 `[N, C, H, W]`가 동일해야 하고, 다를 경우 shortcut을 `1×1 Conv`(projection)로 맞춘 뒤 더한다.
