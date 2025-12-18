# Sigmoid에서 Vanishing Gradient가 생기는 이유와 해결

> Type: Concept

## 상황/배경(Context)
역전파를 공부하다가 “Sigmoid는 왜 vanishing gradient 문제가 생기지?”가 궁금해졌다. 특히 `σ(x)`는 0~1 범위인데, 왜 gradient가 사라지는지 헷갈렸다.

---

## 정의(Definition)
**Vanishing gradient**는 신경망이 깊어질수록 역전파 과정에서 gradient가 점점 작아져, 앞쪽 layer의 파라미터가 거의 업데이트되지 않는 현상이다.

## 핵심 아이디어(Key Ideas)
- 역전파는 연쇄법칙(chain rule) 때문에 여러 layer의 미분값을 **계속 곱**한다.
- 곱해지는 값이 1보다 작으면(특히 0에 가까우면) 깊어질수록 gradient가 빠르게 0으로 수렴할 수 있다.
- Sigmoid는 도함수가 최대 0.25이고, 포화(saturation) 구간에서는 거의 0이라 이 문제가 더 심해진다.

## 역사/배경(Timeline/Why)
- 초기 신경망에서는 `sigmoid/tanh`가 널리 쓰였고, 깊게 쌓을수록 학습이 잘 안 되는 문제가 반복적으로 관찰됐다.
- 이후 `ReLU` 계열 활성함수, 초기화(Glorot/He), 정규화(BatchNorm/LayerNorm), 잔차 연결(ResNet) 같은 기법이 등장하며 깊은 네트워크 학습이 실용화됐다.

## 기호 정리(Symbols)
- `σ(x)`: sigmoid, $ \sigma(x) = \frac{1}{1+e^{-x}} $
- `σ'(x)`: sigmoid의 도함수
- `J`: loss(손실 함수)
- `∂J/∂...`: 해당 변수에 대한 gradient

## 원리/수식(Principle/Math)
### 1) Sigmoid의 출력 범위 vs 도함수 범위
Sigmoid 자체의 출력은 $(0,1)$ 범위다. 하지만 vanishing과 직접 관련 있는 건 `σ(x)`가 아니라 `σ'(x)`(미분값)이다.

Sigmoid의 도함수는:

$$
\sigma'(x) = \sigma(x)\bigl(1-\sigma(x)\bigr)
$$

여기서 $p=\sigma(x)$라고 두면 $p \in (0,1)$이고, $p(1-p)$의 최대는 $p=0.5$에서:

$$
\max \sigma'(x) = 0.5 \cdot (1-0.5) = 0.25
$$

즉, sigmoid의 도함수는 항상 $0 < \sigma'(x) \le 0.25$다.

### 2) 왜 깊어질수록 더 심해지나(곱셈)
역전파는 연쇄법칙으로 여러 layer의 미분을 곱한다. 단순화하면 아래처럼 된다.

$$
\frac{\partial J}{\partial x}
=
\frac{\partial J}{\partial a^{(L)}} \cdot \prod_{l=1}^{L} \frac{\partial a^{(l)}}{\partial z^{(l)}}
$$

여기서 활성함수가 sigmoid면 $\frac{\partial a^{(l)}}{\partial z^{(l)}} = \sigma'(z^{(l)})$이고, 각 항이 최대 0.25라서 깊어질수록 곱이 빠르게 작아질 수 있다.

또한 `z`가 크거나 작으면 $\sigma(z)$가 0 또는 1 근처로 포화되어 $\sigma'(z)\approx 0$이 되므로, gradient가 더 쉽게 사라진다.

## 예시(Examples)
### “0~1이면 문제 없지 않나?”가 헷갈린 이유
“0~1”은 `σ(x)`의 범위이고, 역전파에서 곱해지는 것은 `σ'(x)`다.

예를 들어 어떤 구간에서 매 layer마다 `σ'(z) ≈ 0.1` 정도만 되어도:

- 10층: $0.1^{10} = 10^{-10}$
- 50층: $0.1^{50}$ (사실상 0)

이런 식으로 앞단 gradient는 거의 0이 되어 업데이트가 멈춘다.

## 주의할 점(Caveats)
- vanishing gradient는 활성함수만의 문제가 아니라, 가중치 스케일/초기화/정규화/네트워크 구조가 함께 작동한 결과다.
- Adam 같은 optimizer는 도움은 되지만, “곱이 0으로 수렴하는 구조”를 근본적으로 없애는 건 활성함수/초기화/구조(잔차) 쪽이다.

## 해결 방법(Solutions)
- 활성함수: `ReLU`/`Leaky ReLU`/`GELU` 등(포화 구간 완화, gradient 흐름 개선)
- 초기화: Glorot/Xavier(tanh 계열), He/Kaiming(ReLU 계열)로 분산 유지
- 정규화: BatchNorm/LayerNorm으로 입력 분포 안정화(포화 감소)
- 구조: 잔차 연결(ResNet)로 gradient “우회로” 제공
- 시퀀스 모델: LSTM/GRU처럼 게이트로 긴 의존성에서 gradient 보존

## 한 줄 요약(Summary)
Sigmoid는 도함수가 최대 0.25이고 포화 구간에서는 거의 0이라, 깊은 네트워크에서 연쇄법칙의 곱셈으로 gradient가 빠르게 작아져 vanishing gradient가 발생한다. 이를 완화하려면 `ReLU` 계열, 적절한 초기화/정규화, 잔차 연결 같은 구조적 기법을 함께 사용한다.

## 용어(Glossary)
- **vanishing gradient**: 깊어질수록 gradient가 작아져 앞단 학습이 멈추는 현상
- **saturation**: 활성함수가 평평해져(기울기≈0) gradient가 잘 안 흐르는 구간
- **residual connection**: `y = x + F(x)`처럼 shortcut을 두어 gradient 흐름을 개선하는 구조
