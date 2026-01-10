# EfficientNet 입력은 정방형이어야 하나? (실무 정리)

> Type: Concept

## 상황/배경(Context)
EfficientNet-B7 같은 모델은 흔히 입력이 `600×600`처럼 “정방형”으로 소개된다. 그래서 “EfficientNet은 입력이 반드시 정사각형이어야 하나?”라는 질문이 생긴다. 결론은 가능/불가능 문제가 아니라, **학습 분포와 전처리 전략 선택** 문제다.

---

## 정의(Definition)
이 글에서의 질문은 두 층위로 나뉜다.
- **기술적으로 가능한가?**: 모델이 non-square 입력을 forward 할 수 있는가
- **성능적으로 합리적인가?**: pretrain/학습 시 가정한 해상도·비율 분포를 얼마나 유지하는가

## 핵심 아이디어(Key Ideas)
- EfficientNet 입력은 **정방형일 “필요는 없다”**.
  - 정방형을 쓰는 관행은 주로 ImageNet 학습에서의 crop, 그리고 `resolution`을 단일 스칼라로 다루는 scaling 정의에서 왔다.
- CNN(backbone)이 Conv/BN/Activation 위주이고 classifier가 global pooling 기반이면, 입력의 `H×W`는 고정일 필요가 없다.
  - EfficientNet은 global average pooling을 쓰는 구조라 non-square도 기술적으로 통과한다.
- 하지만 **입력 비율/해상도를 바꾸면 객체 크기 분포, receptive field 대비 스케일, spatial 통계가 바뀐다**.
  - 따라서 성능은 전처리 선택에 크게 좌우된다.

## 예시(Examples)
### 1) 실무 전처리 전략 3가지
#### 전략 A) Aspect ratio 유지 + padding(권장)
```
1280×720
↓ 비율 유지 resize
600×338
↓ padding
600×600
```
- 장점: 형태 왜곡 최소, “학습 당시 해상도 가정”을 어느 정도 유지
- 단점: padding 영역은 정보가 없음
- 객체탐지에서 가장 흔한 선택(레터박스/letterbox)

#### 전략 B) Non-square 입력 그대로 사용(상황에 따라)
예: `640×384`를 그대로 backbone에 입력
- 장점: padding 낭비가 없음, 영상(16:9) 입력에 자연스러움
- 단점: ImageNet pretrain 분포와 어긋날 수 있어 검증 필요, head의 stride/anchor/feature map 크기와의 정합이 중요

#### 전략 C) 강제 resize(비추천, 특히 detection)
예: `1280×720 → 600×600`로 찌그러뜨리기
- classification에서는 어느 정도 버티는 경우가 있지만,
- detection/segmentation에서는 형태 왜곡이 치명적일 수 있어 보통 비추천

### 2) “원본이 작아도 일단 크게 보간하면 되지 않나?”
- 입력 크기 자체는 맞출 수 있지만(upsampling),
- upsampling은 정보를 새로 만들지 않는다.
  - 객체 스케일이 비정상적으로 커져 학습 가정과 어긋날 수 있다.
- 그래서 단일 resize만 고집하기보다, `scale jitter`/multi-scale training 같은 보완이 함께 쓰이면 더 합리적이다.

## 주의할 점(Caveats)
- B7 같은 큰 모델은 FLOPs/메모리 비용이 매우 커서, 영상 backend에서 latency/throughput 요구가 있으면 부적절할 수 있다.
  - 대신 “성능 상한(upper bound) 확인용” backbone으로는 가치가 있다.
- non-square를 쓸 때는 “백본이 돌아간다”와 “전체 파이프라인이 맞다(특히 detection head)”는 별개의 문제다.

## 한 줄 요약(Summary)
EfficientNet은 입력이 정방형일 필요는 없지만, 학습 시 가정한 해상도·비율 분포를 얼마나 유지하느냐가 성능을 좌우하므로 실무에서는 `aspect ratio 유지 + padding` 또는 `non-square 입력`을 상황에 맞게 선택한다.
