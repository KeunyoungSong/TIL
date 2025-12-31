# One-hot encoding: 라벨과 입력 인코딩

> Type: Concept

## 상황/배경(Context)
분류 문제를 구현하다 보면 “왜 라벨은 one-hot으로 바꾸는데 이미지(입력)는 안 바꾸지?” 같은 혼란이 자주 생긴다.  
one-hot encoding이 “데이터를 바꾸는 기술”이라기보다 “범주형(category) 정보를 수치로 표현하는 방식”이라는 점을 정리해두려고 한다.

---

## 정의(Definition)
**one-hot encoding**은 $K$개의 범주 중 하나를 가지는 값을 길이 $K$의 벡터로 바꾸되, 해당 범주의 위치만 1이고 나머지는 0인 표현으로 만드는 방법이다.

## 핵심 아이디어(Key Ideas)
- one-hot은 **범주형(category) 정보를 모델이 계산 가능한 숫자 형태로 표현**하는 방식이다.
- “이미지/텍스트/표” 같은 원본 데이터 타입과 무관하게, **그 값이 category인지 여부**가 핵심이다.
- 분류에서 라벨($y$)은 보통 category이므로 one-hot이 자연스럽고, 픽셀 값 같은 입력($x$)은 보통 **연속형/순서형 수치**라 one-hot 대상이 아니다.
- one-hot 자체는 학습을 “더 똑똑하게” 만들기보다, **손실함수/출력과의 형식을 맞추는 역할**을 하는 경우가 많다.

## 역사/배경(Timeline/Why)
- 분류 출력이 $K$개일 때, 모델은 보통 $K$차원 점수(logits)를 내고(`softmax`) 확률 분포로 해석한다.
- 이때 “정답도 $K$차원에서 비교”하면 수식/구현이 단순해져서 one-hot 라벨이 널리 쓰였다.
- 다만 많은 프레임워크는 **메모리 효율** 때문에 라벨을 one-hot으로 만들지 않고도 동등한 손실을 계산할 수 있게(`class index + cross entropy`) API를 제공한다.

## 기호 정리(Symbols)
- $K$: 클래스 개수
- $x$: 입력(feature)
- $y$: 정답 라벨(범주)
- $\hat{p}$: 모델이 예측한 클래스 확률 분포(길이 $K$)
- $z$: logits(softmax 직전 점수, 길이 $K$)

## 도식(Diagram)
```
입력 x  ---->  모델  ---->  logits z (K)  ----softmax---->  p_hat (K)
                           정답 y  ----one-hot---->  y_onehot (K)
```

## 원리/수식(Principle/Math)
라벨이 one-hot일 때, cross-entropy는 다음처럼 쓸 수 있다.

$$
L = -\sum_{k=1}^{K} y_k \log(\hat{p}_k)
$$

정답 클래스가 $c$라면 $y_c=1$이고 나머지는 0이므로,

$$
L = -\log(\hat{p}_c)
$$

즉 “정답 클래스 확률만 꺼내서 로그를 취해 패널티를 준다”로 이해해도 된다.

## 예시(Examples)
### 1) 라벨(y) one-hot: 이미지 분류에서 흔한 형태
클래스가 `cat`, `dog`, `fox` 3개라고 하자.

- 원본 라벨: `["cat", "dog", "cat"]`
- 클래스 인덱스(예: cat=0, dog=1, fox=2): `[0, 1, 0]`
- one-hot:
  - `0`(cat) → `[1,0,0]`
  - `1`(dog) → `[0,1,0]`
  - 결과: `[[1,0,0], [0,1,0], [1,0,0]]`

### 2) 입력(x) one-hot: 입력이 범주형 feature일 때
표 데이터에서 `색상=color`가 `red/blue/green` 중 하나라면 입력 feature 자체가 category이므로 one-hot을 적용할 수 있다.

- 원본 feature: `["red", "blue", "red"]`
- one-hot(순서: blue, green, red 가정):
  - `red` → `[0,0,1]`
  - `blue` → `[1,0,0]`

## 주의할 점(Caveats)
- **라벨은 one-hot “일 수도”** 있다: 많은 프레임워크는 `class index`를 입력으로 받아 내부에서 $-\log(\hat{p}_c)$를 바로 계산한다. (one-hot을 직접 만들 필요가 없는 경우가 많다)
- **multi-class vs multi-label**:
  - multi-class(정답 1개): one-hot(합이 1)
  - multi-label(정답 여러 개 가능): multi-hot(여러 위치가 1), 보통 `sigmoid + binary cross entropy`
- **입력 수치에 one-hot을 억지로 적용하지 않기**: 픽셀(0~255)처럼 연속형/순서형 값을 one-hot으로 바꾸면 차원이 폭증하고 값의 “크기 정보”가 사라진다.
- **차원 폭발**: category 종류($K$)가 크면 one-hot은 메모리/연산이 비싸다. 이때는 `embedding`이나 `target encoding` 같은 대안을 검토한다.

## 한 줄 요약(Summary)
one-hot encoding은 “데이터 타입”이 아니라 **값이 category인지**에 따라 적용하며, 분류에서는 보통 **라벨(y)을 확률 분포 출력과 맞추기 위해** one-hot(또는 동등한 형태)로 다룬다.

## 참고(Links)
- https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.OneHotEncoder.html
- https://pytorch.org/docs/stable/generated/torch.nn.CrossEntropyLoss.html
