# ReLU 켜짐/꺼짐과 Gradient 업데이트

> Type: Concept

## 상황/배경(Context)
ReLU는 정방향(forward)에서 음수 구간을 0으로 “잘라버린다”는 점이 직관적이지만, 그게 역전파(backward)와 가중치 업데이트에 어떤 영향을 주는지 헷갈렸다. 작은 수치 예시로 “켜짐/꺼짐”이 업데이트 신호를 어떻게 바꾸는지 정리했다.

---

## 정의(Definition)
ReLU는 $ReLU(z)=\max(0,z)$인 활성함수다. 역전파에서는 $dReLU/dz$가 $z>0$에서 1, $z<0$에서 0이므로, 뉴런이 켜진 영역에서는 gradient가 통과하고 꺼진 영역에서는 gradient가 차단된다.

## 핵심 아이디어(Key Ideas)
- forward:
  - $z>0$이면 출력이 $z$ 그대로 통과
  - $z \le 0$이면 출력이 0으로 잘림
- backward:
  - $z>0$이면 $da/dz=1$이라 gradient가 1배로 통과(값을 1로 만드는 게 아니라 “미분 계수가 1”)
  - $z<0$이면 $da/dz=0$이라 gradient가 차단되어 해당 경로의 가중치가 업데이트 신호를 못 받을 수 있음

## 추가 질문(FAQ)
### Q1. forward에서 $z>0$이면 “손실에 기여한 것”으로 확정인가?
아니다. $z>0$은 ReLU가 **경로를 열어둔다(gradient gate open)**는 뜻이다. 실제로 업데이트가 생기려면 위에서 내려오는 upstream gradient가 0이 아니어야 한다.

예를 들어:

$$
\frac{\partial J}{\partial w}
=
\underbrace{\frac{\partial J}{\partial a}}_{\text{upstream}}
\cdot
\underbrace{\frac{da}{dz}}_{\text{ReLU}}
\cdot
\underbrace{\frac{\partial z}{\partial w}}_{=x}
$$

- $z>0$이면 $da/dz=1$이라 gradient가 흐를 “길”은 열린다.
- 그래도 $\partial J/\partial a = 0$이면 $\partial J/\partial w = 0$이라 업데이트는 없다.

### Q2. upstream gradient($\partial J/\partial a$)는 어디서 나오나?
역전파(backward)에서 loss $J$에서 시작한 gradient가 네트워크를 따라 내려오며, 어떤 노드의 출력 $a$에 도달했을 때 그 값이 $\partial J/\partial a$로 표현된다. 즉 “손실 쪽(출력 쪽)에서 내려온 gradient”다.

## 기호 정리(Symbols)
- $z = wx + b$: pre-activation
- $a = ReLU(z)$: activation(출력)
- $J$: loss(손실 함수)
- $\eta$: learning rate

## 원리/수식(Principle/Math)
ReLU의 도함수는:

$$
\frac{da}{dz} =
\begin{cases}
1 & (z>0) \\
0 & (z<0)
\end{cases}
$$

그리고 체인룰로:

$$
\frac{\partial J}{\partial w}
=
\frac{\partial J}{\partial a}
\cdot
\frac{\partial a}{\partial z}
\cdot
\frac{\partial z}{\partial w}
=
\frac{\partial J}{\partial a}
\cdot
\frac{da}{dz}
\cdot
x
$$

여기서 $\frac{da}{dz}$가 0이면(뉴런 꺼짐) $\partial J/\partial w$도 0이 된다.

## 예시(Examples)
뉴런 1개짜리 모델을 두고, 켜짐/꺼짐에 따라 업데이트가 생기는지 확인한다.

- $z = wx + b$
- $a = ReLU(z)$
- 예시 loss: $J = a$ (단순히 $a$가 커질수록 손실이 커진다고 가정)
  - 따라서 $\partial J/\partial a = 1$
- 입력은 $x=1$로 둔다.

### 케이스 1) ReLU가 꺼짐($z<0$)
예: $w=-1$, $b=0$

forward:
- $z=-1\cdot 1 + 0 = -1$
- $a=ReLU(-1)=0$

backward:
- $da/dz=0$ (음수 구간)
- $\partial J/\partial z = (\partial J/\partial a)\cdot(da/dz) = 1\cdot 0 = 0$
- $\partial J/\partial w = (\partial J/\partial z)\cdot x = 0\cdot 1 = 0$
- $\partial J/\partial b = \partial J/\partial z = 0$

결과: 이 샘플 기준으로 $w$, $b$ 업데이트가 없다.

### 케이스 2) ReLU가 켜짐($z>0$)
예: $w=1$, $b=0$

forward:
- $z=1\cdot 1 + 0 = 1$
- $a=ReLU(1)=1$

backward:
- $da/dz=1$ (양수 구간)
- $\partial J/\partial z = 1\cdot 1 = 1$
- $\partial J/\partial w = 1\cdot x = 1$
- $\partial J/\partial b = 1$

GD 업데이트:

$$
w \leftarrow w - \eta \frac{\partial J}{\partial w},\quad
b \leftarrow b - \eta \frac{\partial J}{\partial b}
$$

결과: 이 샘플 기준으로 $w$, $b$가 실제로 움직인다.

## 주의할 점(Caveats)
- 어떤 뉴런이 많은 샘플에서 계속 $z \le 0$이면 gradient를 거의 못 받아 “dead ReLU”처럼 동작할 수 있다.
- 이를 완화하려고 `Leaky ReLU`/`ELU` 같은 변형을 쓰거나, 초기화/학습률/정규화를 조정하기도 한다.

## 한 줄 요약(Summary)
ReLU는 forward에서 음수 구간을 0으로 만들고, backward에서는 그 구간의 미분이 0이라 gradient가 차단되어 해당 경로의 가중치가 업데이트되지 않을 수 있다(반대로 $z>0$이면 gradient가 1배로 통과한다).
