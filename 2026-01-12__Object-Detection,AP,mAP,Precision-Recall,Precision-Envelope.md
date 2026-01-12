# Object Detection: AP/mAP, Precision–Recall, Precision Envelope

## 0) 먼저 뽑은 TIL 후보 목록(=문서 설계도)

- Object Detection에서 `mAP`가 의미하는 것: `Precision/Recall` + `IoU` + `confidence ranking` 요약
- `AP`(Average Precision)란 무엇인가: 한 클래스의 `Precision–Recall` 점들로 성능을 정의하는 방법
- `mAP`는 왜 “평균”인가: `AP1, AP2, ...`가 클래스별 점수인 이유
- `Precision–Recall` 그래프 읽는 법: `threshold`를 내릴수록 `Recall`은 오르고 `Precision`은 왜 흔들리는가
- PR은 점인데 왜 선(커브)로 그리나: 시각화 관습 vs 실제 계산(이산 점 기반)
- `Precision Envelope` 만들기: “`Recall >= R`에서의 최대 `Precision`”을 오른쪽부터 누적하는 절차
- `AP = ∫ Precision(Recall) d(Recall)` 표기의 의미: 적분(개념) vs `Σ`(구현) 연결
- `AP`가 높고 낮을 때 담긴 정보: `FP`가 언제 섞이는지(오탐 유입 속도)로 보는 `confidence` 정렬 품질
- `AP/mAP`의 한계(왜곡): 운영 `threshold`, `IoU` 이진화, envelope 과대평가, 클래스 동등 평균
- 왜 `confidence`로 “순서(ranking)”를 매겨야 하나: 실무 파이프라인에서 상위 박스만 쓰는 이유
- (확인 질문) `Recall`이 높은 구간에서 `Precision`이 얼마나 빨리 무너지는지로 `AP`를 봐도 되나?

이제 아래 섹션은 **위 목록을 순서 그대로** 따라간다.

---

## 1) Object Detection에서 mAP가 의미하는 것

Object Detection은 “맞췄냐/틀렸냐”만으로 끝나지 않는다. 예측 1개가 보통 아래를 같이 가진다.

- class: 무엇인가?
- box: 어디인가? (localization)
- confidence: 내가 얼마나 확신하는가?

그리고 평가에서 흔히 같이 들어오는 것:

- `IoU`: “box가 GT를 충분히 덮었는가?”
- `TP/FP/FN`: “맞춘 검출 / 틀린 검출 / 놓친 정답”

`mAP`는 한 문장으로 이렇게 볼 수 있다.

> “(클래스별로) confidence를 내릴수록 더 많이 찾게 되는데, 그 과정에서 오탐(FP)이 얼마나 늦게 섞이냐(=Precision이 얼마나 늦게 무너지냐)를, IoU 기준까지 포함해서 평균 낸 점수”

즉 “정확도(accuracy) 같은 단일 비율”과 결이 다르다.  
여기에는 `ranking(순서)`가 핵심으로 들어간다.

---

## 2) AP(Average Precision)란 무엇인가 (한 클래스 버전)

`AP`는 **한 클래스에 대해서만** 본다. (예: golfball만)

절차는 직관적으로 이런 흐름이다.

1) “이 클래스다”라고 예측한 박스들을 전부 모은다.
2) confidence 높은 순으로 정렬한다.
3) 위에서부터 하나씩 포함시키며 `TP/FP`를 누적한다.
4) 매 단계마다 `Precision/Recall`을 계산해서 PR 점들을 만든다.
5) 그 PR 관계를 “한 숫자”로 요약한다 = `AP`

`AP`의 본질적 질문:

> “진짜(TP)를 가짜(FP)보다 위에(높은 confidence에) 잘 올려놨나?”

---

## 3) mAP는 왜 ‘평균’인가 (AP1, AP2, …)

클래스가 N개면, 클래스마다 `AP`를 따로 계산한다.

- `AP1`: class 1에 대한 `AP`
- `AP2`: class 2에 대한 `AP`
- ...
- `APN`: class N에 대한 `AP`

그리고 단순 평균:

`mAP = (AP1 + AP2 + ... + APN) / N`

이 방식의 장점:

- “전체적으로 좋아 보이는데 특정 클래스만 망한 모델”을 한 숫자 뒤에 숨기지 못하게 한다.

이 방식의 단점(나중 섹션에서 다시):

- 클래스 중요도가 다르면(치명 vs 무시 가능) 똑같이 `1/N`로 취급해버린다.

---

## 4) Precision–Recall 그래프 읽는 법 (threshold를 움직이면 왜 점들이 생기나)

정의(축 관점):

- `Recall` (x축): “정답을 얼마나 많이 찾았나”
- `Precision` (y축): “찾았다고 말한 것 중 얼마나 진짜였나”

`threshold`를 높게 잡으면:

- 말수를 줄인다(상위 confidence만 남김)
- `Precision`은 올라가기 쉽고, `Recall`은 떨어지기 쉽다

`threshold`를 낮추면:

- 말수를 늘린다(낮은 confidence까지 포함)
- `Recall`은 올라가지만, `FP`가 섞이면 `Precision`이 떨어진다

그래서 `threshold`를 조금씩 내리면, 매 단계마다 `(Recall, Precision)` 한 점이 생긴다.

---

## 5) PR은 점인데 왜 선(커브)로 그리나

중요한 사실:

- PR은 “연속함수 곡선”이 아니라, 실제로는 “이산 점들의 집합”이다.

그런데 선을 그어버리면:

- 사람 눈에는 추세가 잘 보이고,
- “면적(=AP)”이라는 개념이 직관적으로 전달된다.

그래서 선은 대체로 “설명용 시각화 관습”이다.  
`AP`를 계산한다고 해서 점 사이를 진짜로 직선 적분하는 것은 아니다(구현은 보통 다르게 간다).

---

## 6) Precision Envelope(외피선) 만들기

문제:

- 실제 PR 점들은 들쭉날쭉한다(`Recall`이 늘어도 `Precision`이 다시 오를 수 있음).

해결 아이디어(정의):

> “Recall이 R 이상이 되게 만들 수 있다면, 그때 달성 가능한 최대 Precision을 그 R의 값으로 삼자”

표기(말로):

- `Envelope(R) = max Precision among points with Recall >= R`

실제로 선을 그리는 순서(핵심):

- 오른쪽(큰 `Recall`)부터 왼쪽으로 오면서,
- “지금까지 본 최대 `Precision`”을 유지한다(최댓값 누적).

즉, 오른쪽을 봐야 정의가 성립하니까 오른쪽부터 시작한다.

---

## 7) AP=∫ Precision(Recall) d(Recall) 표기는 왜 나오나 (개념 vs 구현)

헷갈림 포인트:

- `Precision(Recall)` 같은 표기는 “Recall을 넣으면 Precision이 나오는 연속 함수”처럼 보인다.
- 하지만 실제로 우리에게 있는 건 `(Rk, Pk)` 점들뿐이다.

왜 적분(∫)을 쓰나:

- “PR 아래 면적”이라는 개념을 표현하는 표기 관습(정의문에 가까움)

실제로는 합(`Σ`)로 계산한다는 메시지를 주려면 이렇게 생각하면 된다.

- 적분은 “무한히 잘게 쪼갠 합”
- 우리는 “유한한 점들”만 있으니 “합”으로 한다

그리고 여기서 중요한 ‘한 틱’은 이것이다.

- `Precision * Recall`  ❌
- `Precision * (Recall의 증가량)`  ✅

즉, “현재까지 누적 Recall”이 아니라 “이번 단계에서 늘어난 Recall”이 가로폭이다.

---

## 8) AP가 높고 낮을 때 담긴 정보 (오탐 유입 속도)

`AP`가 높다:

- `threshold`를 낮춰서 `Recall`을 올려도, `Precision`이 한동안 잘 버틴다
- 즉 `FP`가 “늦게” 섞인다
- (`ranking` 관점) 진짜(`TP`)가 높은 confidence에 몰려 있고, 가짜(`FP`)는 낮은 confidence로 밀려 있다

`AP`가 낮다:

- `threshold`를 조금만 낮춰도 `FP`가 빨리 섞여 `Precision`이 급락한다
- (`ranking` 관점) `FP`가 높은 confidence 상단에 끼어 있거나, `TP`가 낮은 confidence 쪽으로 밀려 있다

실무적 읽기(거칠게):

> “낮은 confidence 영역까지 확장했을 때도 얼마나 깨끗하냐”

---

## 9) AP/mAP의 한계(오류/왜곡)

`AP`는 좋은 지표지만, “그 숫자 하나로 운영 품질을 직접 대체하면” 위험해질 수 있다.

대표적인 왜곡 포인트들:

- 운영 `threshold` 문제: `AP`는 threshold를 스윕해서 전체를 요약한다. 하지만 실제 제품은 특정 threshold 하나(또는 몇 개)에서 동작한다.
- `IoU` 이진화: `IoU=0.49` vs `0.50`처럼 근소한 차이가 `TP/FP`를 갈라 숫자가 요동칠 수 있다(특히 작은 객체).
- Envelope 과대평가 가능성: “한 번이라도 좋았던” precision을 유지하는 방식이라 불안정한 모델을 좋게 보이게 만들 수 있다.
- `mAP`의 클래스 동등 가중: 클래스 중요도가 다르면 `mAP`만으로 의사결정하면 손해를 볼 수 있다.

---

## 10) 왜 confidence로 “순서(ranking)”를 매겨야 하나

현실 시스템은 무한히 많은 박스를 다 쓰지 못한다.

- 화면에는 상위 N개만 그린다
- tracker/후처리 파이프라인도 상위 박스부터 쓴다
- 사람에게도 상위 결과부터 보여준다

그래서 “무엇을 먼저 믿을지”가 필요하고, 그게 `confidence ranking`이다.

`AP`는 사실상 이 질문을 수치화한다.

> “진짜를 먼저 말하고, 가짜는 나중에 말하나?”

---

## 11) (확인 질문) Recall이 높은 구간에서 Precision이 얼마나 빨리 무너지는지로 AP를 봐도 되나?

요지가 맞다.

- `threshold`를 낮춰 `Recall`을 올릴 때
- `FP`가 얼마나 빨리 들어오느냐(=`Precision`이 얼마나 빨리 무너지느냐)

이 “무너짐의 속도”가 `AP`를 크게 좌우한다.

다만 엄밀하게는:

- `AP`는 “PR 점들”을 envelope로 단조화한 뒤
- 그 면적(가로폭=`Recall` 증가량으로 가중)을 누적한 값

이므로, “특정 고-Recall 구간만”이 아니라 “0~1 전체에서의 누적”이다.  
그래도 직관으로는 “끝까지 내렸을 때 얼마나 깨끗했나”라는 해석이 가장 실용적이다.

---

## 마지막으로, 기억할 한 문장
`AP`는 `threshold`를 낮춰 `Recall`을 올려갈 때 `FP`가 얼마나 늦게 섞이는지(= `Precision`이 얼마나 늦게 무너지는지)를 요약한 점수다.
