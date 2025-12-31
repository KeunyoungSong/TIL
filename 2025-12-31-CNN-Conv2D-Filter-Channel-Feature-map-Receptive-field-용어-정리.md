# CNN / Conv2D / Filter(Kernel) / Channel / Feature map / Receptive field 용어 정리

> Type: Concept

## 상황/배경(Context)
CNN을 처음 배우면 “Conv는 뭐고 필터는 몇 개고 채널은 왜 늘고, receptive field는 또 뭔가” 같은 용어가 한 번에 쏟아져서 헷갈린다.  
지금까지 대화에서 나온 핵심 용어들을 같은 기준(입력/출력 텐서 관점)으로 묶어서 정리한다.

---

## 정의(Definition)
CNN 관련 용어를 “레이어(연산)”, “텐서(형태)”, “학습 루프(훈련 단위)”로 나눠 간단 정의한다.

## 핵심 아이디어(Key Ideas)
- Conv 레이어는 입력을 **필터(가중치)**로 변환해 **feature map(표현)**을 만든다.
- “크기 축소(H, W)”는 Conv의 본질이라기보다 `stride/pooling` 같은 **설계 옵션**이다.
- “깊어질수록 추상화”는 레이어가 추상해진다는 뜻이 아니라, 레이어가 만든 **표현(representation)**이 고수준 특징을 담는 경향을 뜻한다.

## 도식(Diagram)
```
입력 x (H×W×C_in)
  -> Conv2D(filters=C_out, kernel=K×K, stride=s, padding=p)
  -> feature map h (H_out×W_out×C_out)
  -> (반복/다운샘플링)
  -> (Flatten 또는 GAP)
  -> classifier head (Dense/Linear)
  -> class logits/prob
```

## 용어(Glossary)
### CNN (Convolutional Neural Network)
convolution(합성곱) 연산을 핵심 구성요소로 사용하는 신경망.  
실무/강의 문맥에선 종종 “conv 블록으로 이루어진 backbone(특징 추출부)”를 CNN이라고 부르기도 한다.

### Conv / Conv2D
2차원 이미지(또는 feature map)를 대상으로 하는 convolution 레이어.  
입력의 국소 영역을 커널로 스캔하며 선형 변환을 수행하고(그 뒤에 ReLU/BN 같은 비선형이 붙는 경우가 많다), 출력 feature map을 만든다.

### Filter / Kernel (커널, 필터)
Conv에서 사용하는 학습 가능한 가중치 덩어리.
- “커널 크기”는 보통 `3×3`, `1×1`, (초기 stem에서) `7×7`처럼 말한다.
- 한 개의 필터는 “입력 채널 전체”를 보며, 한 개의 출력 채널을 만든다.

### Channel (채널)
텐서의 “깊이” 축. 이미지의 RGB는 보통 `C=3`.
- 입력 채널: `C_in`
- 출력 채널: `C_out`

중요: **필터 개수 = 출력 채널 수(`C_out`)**  
즉 `Conv2D(filters=32)`는 “필터 32개를 학습해서 출력 채널을 32개로 만든다”는 뜻이다.

### Feature / Feature map
Conv 레이어가 만들어낸 출력(중간 표현).
- feature map은 보통 `(H_out, W_out, C_out)` 형태의 텐서다.
- 각 채널은 “특정 패턴(엣지/텍스처/부분 등)에 대한 반응”처럼 해석할 수 있다(항상 깔끔하진 않지만 직관으로 유용).

### Stride (스트라이드)
커널을 몇 칸씩 이동시키는지.
- `stride=1`: 한 칸씩 이동(보통 해상도 유지 가능)
- `stride=2`: 두 칸씩 이동(보통 H, W가 줄어드는 다운샘플링)

### Padding (패딩)
가장자리에 값을 덧대서(보통 0) 출력 크기를 조절하는 방법.
- `padding="same"`: 보통 stride=1일 때 H, W를 유지하도록 패딩
- `padding="valid"`: 패딩 없이 적용(보통 H, W가 줄어듦)

### Receptive field (수용영역)
특정 레이어의 “출력 한 위치(한 뉴런)”가 입력 이미지에서 영향을 받는 영역의 크기/범위.
- `3×3 conv` 한 층만 보면 출력의 한 점은 입력의 대략 `3×3` 영역을 본다.
- 레이어를 여러 개 쌓으면(그리고 stride/pooling이 들어가면) receptive field는 점점 커져서 더 넓은 문맥을 “보게” 된다.

단어 연결: `receptive`는 “받아들이는(receive)”이란 뜻이라, receptive field는 “입력으로부터 영향을 받아들이는 영역(field)”이다.

### Representation (표현), Abstraction (추상화)
- representation: 모델 내부에서 입력을 “유용한 좌표”로 바꾼 중간 표현(feature).
- abstraction: 표현이 픽셀 같은 저수준 신호에서 멀어져, 과제에 유용한 고수준 패턴(부분/객체 등)을 담게 되는 경향.

권장 표현:
- (덜 정확) “레이어가 추상화된다”
- (권장) “레이어가 만든 **표현(feature map/representation)**이 더 추상화되는 경향이 있다”

### Backbone(특징 추출부) / Head(분류기)
분류 CNN을 크게 둘로 나눈 표현.
- backbone: Conv 블록들로 입력에서 feature를 뽑는 부분
- head: 그 feature로 최종 예측을 만드는 부분(예: GAP/Flatten + Dense/Linear)

“잘 분류되느냐”는 head도 중요하지만, 특히 **head가 받는 직전 feature가 클래스 구분에 유리하게 정리되어 있는지**(구분 가능성)가 성능을 크게 좌우한다.

### Flatten (평탄화)
다차원 feature map `(H, W, C)`를 1차원 벡터로 펴서 Dense 레이어에 넣기 위한 변환.
- 요즘은 Flatten 대신 `GlobalAveragePooling(GAP)`을 쓰는 구성도 흔하다(파라미터 감소/일반화 등).

### Epoch / Batch / Step (학습 루프 단위)
훈련을 “몇 번, 어떤 덩어리로” 업데이트하는지에 대한 용어.
- epoch: (보통) 전체 학습 데이터를 1번 소비하는 단위
- batch_size(batch): 한 step에서 처리하는 샘플 수
- step(iteration): 배치 1개 처리 단위(보통 gradient update 1회)

일반적인 mini-batch 학습에서는 **가중치가 매 step(=매 batch)**마다 업데이트된다.

## 한 줄 요약(Summary)
CNN에서 Conv는 필터(커널)로 입력을 변환해 feature map(표현)을 만들고, 필터 개수는 출력 채널을 결정하며, receptive field는 출력이 입력에서 “얼마나 넓게 영향을 받는지”를 뜻한다.

