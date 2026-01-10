# ResNet: Residual Learning과 Identity 오해 정리

> Type: Concept

## 상황/배경(Context)
슬라이드에서 “깊은 네트워크에 identity layer를 추가하면 성능이 유지될 텐데, Conv를 추가하면 오히려 성능이 떨어진다”는 예시를 보고 혼란이 생겼다. 특히 “ResNet은 identity를 넣고 싶어서 나온 건가?” 그리고 “$F(x)$를 학습하는 게 실제로 유의미한가?”를 한 번에 정리하고 싶었다.

---

## 정의(Definition)
- plain(일반) deep CNN은 어떤 함수 $H(x)$를 직접 학습하려고 한다.
- ResNet은 출력을 $H(x)=x+F(x)$로 두고, 블록이 학습할 대상을 **residual mapping** $F(x)=H(x)-x$로 바꾼다.
- 여기서 **identity/skip connection**은 $x$를 “그대로 전달하는 경로”이고, 학습되는 파라미터가 없다(연결 방식).

## 핵심 아이디어(Key Ideas)
- ResNet의 출발점은 “identity를 많이 넣자”가 아니라, **깊어질수록 훈련 오차조차 증가하는 degradation problem(최적화 실패)** 를 해결하자는 문제의식이다.
- identity는 “목표”가 아니라 **baseline(안전장치)** 이다.
  - 깊은 모델이 최소한 얕은 모델만큼은 해야 한다는 가정에서, “필요 없으면 아무것도 안 해도 되는 경로”를 구조적으로 보장한다.
- 실제 성능 향상은 identity 때문이 아니라, identity 위에서 **유의미한 $F(x)$가 누적되기 때문**이다.
  - 필요 없으면 $F(x)=0$에 가까워지면 되고,
  - 필요하면 $F(x)\neq 0$로 “조금 더 좋은 방향”을 제안한다.
- “깊어지면 왜 좋아지나?”를 ResNet 관점으로 말하면:
  - “한 번에 큰 변환”을 강요하는 대신,
  - “많은 작은 보정($F(x)$)을 누적”하는 형태로 학습 단위를 바꿔서 깊이가 성능으로 연결된다.

## 예시(Examples)
슬라이드의 Case를 이렇게 해석하면 깔끔하다.

- **Case 1**: 기본 모델이 정확도 75%를 냄.
- **Case 2(ideal)**: 뒤에 “진짜 identity layer”를 붙이면 출력이 안 바뀌므로 정확도는 유지(75%).
- **Case 3(현실)**: Conv/activation을 추가하면 처음부터 identity가 아니고, 학습이 “그냥 identity로 두면 된다”는 쉬운 선택을 못 해서 최적화가 깨질 수 있음 → 성능 하락(예: 65%).

ResNet 블록은 Case 3의 질문을 이렇게 바꾼다.
- “이 Conv들이 $H(x)$ 전체를 만들어라”가 아니라,
- “이미 있는 $x$는 보장할 테니, 필요한 변화 $F(x)$만 제안해라”

## 주의할 점(Caveats)
- “Conv로 identity를 표현할 수 있다”와 “학습이 그 identity를 쉽게 찾는다”는 다르다.
  - 표현 가능성(expression)만으로는 최적화(optimize) 문제가 해결되지 않는다.
- identity/skip은 보통 **파라미터 없는 연결**이라서, “학습되는 identity layer”라기보단 **connection pattern**으로 보는 게 정확하다.
- 성능이 무조건 깊이에 비례하진 않는다(데이터/정규화/학습률 등 조건이 맞아야 함). 다만 ResNet은 “깊어지면 최소한 망하지 않게” 만드는 쪽에 강한 구조다.

## 한 줄 요약(Summary)
ResNet은 identity를 목표로 만든 게 아니라, identity 경로를 **안전장치로 깔아** $F(x)$(잔차)만 안정적으로 학습하게 만들어 “깊이가 실제 성능으로 이어지게” 만든 구조다.

---

## 용어(Glossary)
- **degradation problem**: 모델이 깊어질수록 일반화가 아니라 훈련 자체가 어려워져 training error가 증가하는 현상(최적화 실패).
- **skip connection / shortcut**: 입력 $x$를 블록 출력에 더하기 위해 우회로로 전달하는 연결.
- **residual mapping**: $F(x)=H(x)-x$ 형태로 “얼마나 바꿀지”만 학습하는 관점.
