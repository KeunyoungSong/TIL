# Object Detection AP/mAP, Precision–Recall, Precision Envelope 정리

> Type: Concept

## 상황/배경(Context)
`Object Detection`에서 `mAP`/`AP`를 “정의/계산/해석”까지 연결해서 이해하려고 정리했다. 특히 `Precision–Recall`이 본질적으로 점들의 집합인데도 논문/문서에서 적분 표기로 쓰는 이유, 그리고 `Precision Envelope`가 왜 등장하는지를 중심으로 고정한다.

---

## 정의(Definition)
`AP`(Average Precision)는 한 클래스에 대해 `confidence threshold`를 바꿔가며 얻는 `Precision–Recall` 관계를 “한 숫자(면적)”로 요약한 지표이고, `mAP`는 여러 클래스의 `AP`를 산술평균한 값이다.

## 핵심 아이디어(Key Ideas)
- 탐지 결과는 “박스 + 클래스 + confidence”로 나오며, 평가에는 `IoU` 기준(맞았는지/틀렸는지)과 `confidence ranking`(무엇을 먼저 믿을지)이 함께 들어간다.
- `threshold`를 높이면 보수적으로 말해서 `Precision`은 올라가고 `Recall`은 내려간다. `threshold`를 낮추면 더 많이 말해서 `Recall`은 올라가고 `Precision`은 내려간다.
- 그래서 한 점이 아니라 “threshold sweep으로 생기는 점들의 집합”으로 성능을 봐야 하고, 그 전체를 한 숫자로 만든 것이 `AP`다.
- `AP`는 “낮은 confidence 영역까지 포함시켜도 FP가 얼마나 늦게 섞이느냐(=Precision이 얼마나 천천히 무너지는지)”를 잘 반영한다.
- `mAP`는 “클래스별 AP를 동등 가중으로 평균” 내기 때문에, 클래스 중요도가 다른 실무에서는 별도 지표(클래스 가중치/운영 threshold 성능 등)와 같이 보는 게 안전하다.

## 용어(Glossary)
- **TP/FP/FN**: True Positive / False Positive / False Negative
- **IoU**: Intersection over Union, 박스 겹침 비율
- **confidence**: 모델이 해당 예측을 얼마나 확신하는지(보통 score)
- **ranking**: confidence로 예측을 정렬하는 것(무엇을 먼저 처리/표시할지)
- **Precision Envelope**: 각 `Recall`에서 “그 `Recall` 이상에서 가능한 최대 `Precision`”만 남긴 외피(계단형)

## 기호 정리(Symbols)
- $N_{GT}$: 해당 클래스의 GT(정답 객체) 개수
- $k$: confidence 순서로 정렬했을 때의 k번째 예측(=threshold sweep의 한 단계)
- $TP_k, FP_k$: k번째까지 누적 TP/FP
- $R_k$: k번째까지의 `Recall`
- $P_k$: k번째까지의 `Precision`
- $P_k^\*$: `Precision Envelope`로 보정된 `Precision`

## 도식(Diagram)
confidence를 내림차순으로 훑어가며(=threshold를 낮춰가며) TP/FP가 어떻게 섞이는지 보는 구조다.

```
predictions sorted by confidence (high -> low)

k:   1    2    3    4    5   ...
     TP   FP   TP   TP   FP  ...

R (x-axis): 0 -> 1 increases only when a new TP appears
P (y-axis): changes whenever TP/FP mix changes
```

## 원리/수식(Principle/Math)
### 1) PR 점(이산 데이터)이 먼저다
한 클래스에 대해 예측을 confidence 내림차순으로 정렬하고, 각 단계 k에서 누적값으로 `Precision/Recall`을 만든다.

$$
P_k=\frac{TP_k}{TP_k+FP_k}
$$

$$
R_k=\frac{TP_k}{N_{GT}}
$$

이때 `Precision–Recall curve`는 연속 곡선이라기보다 $(R_k, P_k)$ 점들의 집합이다.

### 2) 왜 “면적”이 AP인가
`Recall` 축($x$)은 “얼마나 많이 찾았나”, `Precision` 축($y$)은 “내가 찾았다고 한 것 중 얼마나 진짜냐”를 뜻한다.

따라서 “Recall을 0에서 1까지 올리는 과정에서, Precision을 평균적으로 얼마나 높게 유지했는가”를 한 숫자로 요약하면 비교가 쉬워진다. 그 요약값이 면적(=가로 길이 × 세로 높이의 누적)이다.

### 3) 적분 표기(개념) vs 시그마(구현)
문서/논문에서 자주 보는 표기는 개념(면적)을 나타내는 정의에 가깝다.

$$
AP=\int_{0}^{1} P(R)\,dR
$$

하지만 실제 데이터는 점들뿐이라, 구현은 합(시그마)으로 계산한다.

1) 먼저 `envelope`를 만든다.

$$
P_k^\*=\max_{j \ge k} P_j
$$

2) 그 다음 “Recall 증가량”으로 가중된 합(계단형 면적)을 계산한다.

$$
AP=\sum_{k=1}^{n} (R_k - R_{k-1})\cdot P_k^\*
$$

핵심은 `Precision × Recall`이 아니라 `Precision × (Recall의 증가량)`이 한 틱의 면적이라는 점이다.

### 4) Precision Envelope를 그리는 순서(실제 절차)
`P(R)=\max_{R_k \ge R} P_k`의 정의 때문에, 오른쪽(큰 `Recall`)부터 왼쪽으로 오면서 “지금까지 본 최대 Precision”을 유지한다.

- 가장 오른쪽 점에서 시작: 오른쪽에 아무 점도 없으니 자기 자신이 최대값
- 한 칸씩 왼쪽으로 이동: 현재 점의 precision과 “오른쪽에서 유지 중인 최대값” 중 큰 값을 채택
- 이렇게 만든 계단형 선이 `Precision Envelope`이고, 그 아래 면적이 `AP`

## 주의할 점(Caveats)
- `AP`는 threshold 하나의 운영 성능을 직접 주지 않는다. (서비스에서는 “어떤 threshold에서 쓸지”가 별도 문제)
- `IoU` 기준 때문에 localization이 이진화된다(예: 0.49 vs 0.50이 극단적으로 갈림).
- `mAP`는 클래스별 `AP`를 동등 가중 평균하므로, 클래스 중요도가 다른 경우에는 별도 지표를 같이 봐야 한다.

## 한 줄 요약(Summary)
`AP`는 confidence를 낮춰 `Recall`을 올려갈 때 `Precision`이 얼마나 늦게 무너지는지를(=FP 유입 속도) `Precision Envelope`의 면적으로 요약한 지표이고, `mAP`는 이를 클래스 평균낸 값이다.

