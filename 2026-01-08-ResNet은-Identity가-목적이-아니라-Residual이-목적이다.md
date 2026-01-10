# ResNet은 Identity가 목적이 아니라 Residual($F(x)$)이 목적이다

> Type: Concept

## 상황/배경(Context)
ResNet을 “identity layer를 많이 쌓아서 잘 되는 모델”로 오해하기 쉬웠다. 하지만 논문/아이디어의 중심은 identity 자체가 아니라, 깊은 네트워크에서 **$F(x)$를 안정적으로 학습**시켜 성능을 올리기 위한 문제 재정의였다.

---

## 정의(Definition)
ResNet 블록은 목표 함수를 직접 $H(x)$로 학습시키는 대신,

$$
H(x) = x + F(x)
$$

로 두고, 실제 학습 대상은

$$
F(x) = H(x) - x
$$

로 바꾼다. 여기서 identity/skip connection은 $x$를 그대로 전달하는 **참조점(reference) + 안전장치** 역할이다.

## 핵심 아이디어(Key Ideas)
- 출발점은 overfitting이 아니라 **optimization failure**였다: 깊게 만들었더니 training error조차 증가(degradation).
- identity는 “성능을 올리기 위한 목표”가 아니라, **깊어져도 최소한 망하지 않게 하는 baseline**이다.
  - “필요 없으면 $F(x)=0$이면 된다”가 되면서, 블록이 불필요하면 사실상 아무것도 하지 않아도 된다.
- 성능 향상은 identity 때문이 아니라, identity 위에서 **유의미한 $F(x)$가 누적**되기 때문이다.
  - 각 블록이 “완성된 표현을 새로 만들기”보다 “이미 괜찮은 $x$를 조금 더 낫게 보정”하는 형태로 역할이 바뀐다.

## 예시(Examples)
슬라이드/직관을 이렇게 번역하면 이해가 쉬웠다.
- plain CNN: “이 레이어가 $H(x)$ 전체를 만들어라”
- ResNet: “$x$는 보장할 테니, 필요한 변화만 $F(x)$로 제안해라”

그래서 “identity만 있으면 성능이 좋아진다”가 아니라:
- identity만 있으면: 성능 변화 없음(그냥 통과)
- $F(x)$가 의미 있게 학습되면: feature 정제, decision boundary 개선 → 일반화 성능 향상

## 주의할 점(Caveats)
- Conv가 identity를 “표현”할 수 있다는 것과, 학습이 그 identity를 “쉽게 찾는다”는 것은 다르다.
- ResNet에서도 깊이를 늘리면 항상 좋아진다는 보장은 없다(데이터/정규화/학습 설정 영향). 다만 “깊어질수록 훈련이 깨지는 문제”를 구조적으로 완화한다.

## 한 줄 요약(Summary)
ResNet은 identity를 만들기 위한 기법이 아니라, identity를 **참조점으로 깔아** residual $F(x)$를 더 쉽게/안정적으로 학습하게 만들어 깊이가 성능으로 이어지게 한 구조다.

---

## 참고(Links)
- https://arxiv.org/abs/1512.03385
