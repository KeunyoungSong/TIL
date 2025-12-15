# Backpropagation(역전파)와 수치미분의 차이

> Type: Concept

## 상황/배경(Context)
선형 모델에서는 `∂Loss/∂W`를 손으로 쉽게 유도해서 GD/SGD로 가중치를 업데이트할 수 있다.
하지만 다층 신경망에서는 출력이 합성함수로 중첩되어 있어 “각 가중치가 Loss에 미치는 영향”을 어떻게 계산해야 하는지 헷갈렸다.

---

## 정의(Definition)
**Backpropagation(역전파)**는 다층 신경망에서 손실 함수 `Loss`의 gradient(예: `∂Loss/∂W`)를 **연쇄법칙(chain rule)**으로 효율적으로 계산하는 알고리즘이다.

## 핵심 아이디어(Key Ideas)
- GD/SGD는 “업데이트 규칙”이고, 역전파는 “그 업데이트에 필요한 gradient를 계산하는 방법”이다.
  - 업데이트: `W := W - η * ∂Loss/∂W`
- 수치미분은 gradient를 “근사”하고, 역전파는 gradient를 “계산”한다.
- 역전파는 `forward`에서 만든 중간값을 재사용해서, 모든 파라미터의 gradient를 `backward` 한 번으로 구한다.

## 역사 타임라인(Timeline)
- 1960년대: 최적제어/미분 계산 맥락에서 reverse-mode 형태의 아이디어가 등장
- 1974년: Werbos가 신경망 가중치 학습(gradient 계산)에 연결
- 1986년: Rumelhart/Hinton/Williams로 다층 신경망 학습 알고리즘으로 대중화
- 2012년 전후: 데이터·GPU·기법(초기화/활성함수/정규화 등)과 결합되며 “deep learning” 성능이 널리 입증

## 원리/수식(Principle/Math)
다층 신경망 학습 루프는 크게 아래 뼈대를 가진다.

1. `forward`: 입력 → 예측(`prediction`)
2. `loss`: `prediction`과 `y` 비교해서 `Loss` 계산(MSE/CE 등)
3. `gradient`: `∂Loss/∂W` 계산
4. `update`: `W := W - η * ∂Loss/∂W`
5. 1–4 반복

여기서 3) `gradient` 단계가 수치미분 vs 역전파로 갈린다.

### 수치미분(Numerical Differentiation)
각 파라미터 `W_i`를 아주 조금 바꿔서 loss 변화량으로 기울기를 근사한다.

**Forward difference**

$$
\frac{\partial Loss}{\partial W_i} \approx \frac{Loss(W_i+\epsilon) - Loss(W_i)}{\epsilon}
$$

**Central difference** (보통 더 정확하지만 2번 계산)

$$
\frac{\partial Loss}{\partial W_i} \approx \frac{Loss(W_i+\epsilon) - Loss(W_i-\epsilon)}{2\epsilon}
$$

특징:
- 파라미터가 `P`개면, `∂Loss/∂W`를 구하기 위해 `forward → loss`를 **대략 O(P)번** 추가로 반복해야 한다.
- `ε` 선택에 따라 수치오차가 커질 수 있다.

### Backpropagation(역전파)
`forward`에서 저장한 중간값을 이용해 `Loss`의 미분을 출력층부터 입력층 방향으로 전파하며 계산한다.

전형적인 형태(개념 스케치):
- forward:
  - $z^l = W^l a^{l-1} + b^l$
  - $a^l = \sigma(z^l)$
- backward:
  - 출력에서 시작한 오차 신호 `δ`를 뒤로 전달하며 `∂Loss/∂W^l`를 계산
  - 예: $\frac{\partial Loss}{\partial W^l} = \delta^l (a^{l-1})^T$

특징:
- 모든 파라미터의 gradient를 **`forward` 1번 + `backward` 1번**으로 계산한다(중복 계산을 줄임).
- 규모가 커져도 학습이 “스케일”되기 때문에, 심층 신경망에서 실용적인 학습 루프를 구성할 수 있다.

## 예시(Examples)
“각 가중치가 Loss에 미치는 영향”을 구하는 방식의 차이를 단계로 비교하면:

- 수치미분: `W_i`를 `+ε/-ε`로 바꿔가며 `Loss`를 여러 번 다시 계산해서 `∂Loss/∂W_i`를 근사
- 역전파: 한 번의 `backward`에서 모든 `∂Loss/∂W`를 계산

## 주의할 점(Caveats)
- 역전파는 GD/SGD의 대체가 아니라, GD/SGD가 필요로 하는 gradient를 구하는 방법이다.
- 역전파가 있다고 해서 “아무 deep net이나” 잘 학습되는 것은 아니다. (예: `vanishing gradient`, 초기화, 활성함수 선택 같은 최적화 이슈는 별개의 문제)

## 한 줄 요약(Summary)
수치미분은 파라미터를 흔들어 gradient를 **근사**하고, Backpropagation은 연쇄법칙으로 gradient를 **효율적으로 계산**해서 GD/SGD 업데이트를 가능하게 만든다.

## 용어(Glossary)
- **gradient**: 파라미터 변화에 따른 `Loss` 변화의 방향/크기(예: `∂Loss/∂W`)
- **chain rule**: 합성함수의 미분 법칙(역전파의 핵심)
- **autodiff**: 프로그램이 미분을 자동으로 계산하는 기법(역전파는 그 핵심 알고리즘/구현 방식 중 하나)
