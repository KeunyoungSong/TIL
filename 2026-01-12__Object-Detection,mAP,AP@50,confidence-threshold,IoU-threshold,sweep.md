# Object Detection: mAP 해석, sweep, AP@50, confidence threshold vs IoU threshold

## 0) 먼저 뽑은 TIL 후보 목록(=문서 설계도)

- `mAP`가 높다는 건 “confidence를 낮춰도 누락(FN)이 적다”로 해석해도 되나?
- `sweep`은 무엇이고, `mAP`에서 정확히 무엇을 sweep하나?
- `AP@50`(= `AP50`) 같은 지표는 무엇이고, `mAP`와 무엇이 다르나?
- `confidence threshold` 얘기하다가 왜 `IoU threshold`가 같이 나오나?
- `IoU threshold=0.50` 같은 설정이 `Precision/Recall`에 실제로 영향을 주나?
- 실무에서 “누락이 적은가”를 보려면 무엇을 봐야 하나?
- 요약: `mAP`가 높다는 말이 내포하는 의미는 무엇인가?

이제 아래 섹션은 **위 목록을 순서 그대로** 따라간다.

---

## 1) `mAP`가 높다는 건 “confidence를 낮춰도 누락(FN)이 적다”로 해석해도 되나?

그렇게 해석하면 안 된다.

`mAP`는 “어떤 특정 `confidence threshold`에서의 `FN`이 적다”를 직접 보장하지 않는다.  
`mAP`는 `confidence threshold`를 바꿔가며 만들어지는 `Precision–Recall`의 전체 성질(면적)을 요약한 값이다.

정리하면:

- `FN`(누락) 자체는 `Recall = TP/(TP+FN)`로 나타나고, 이건 “운영에서 어떤 threshold를 쓰느냐” 문제다.
- `mAP`는 “threshold를 훑었을 때 PR이 전반적으로 얼마나 위에 있나”를 보지, “threshold=0.3에서 FN이 얼마나 되나”를 말해주지 않는다.

---

## 2) `sweep`은 무엇이고, `mAP`에서 정확히 무엇을 sweep하나?

이 맥락에서 `sweep`은 어떤 파라미터를 **전 범위로 훑어가며** 성능을 계산하는 절차다.

`mAP`에서 sweep하는 것은 `confidence threshold`다.

- `confidence threshold`를 높게 두면: 보수적으로 출력 → `Precision`이 올라가기 쉽고 `Recall`이 떨어지기 쉽다.
- `confidence threshold`를 낮추면: 더 많이 출력 → `Recall`은 올라가지만 `FP`가 섞이면 `Precision`이 내려간다.

이렇게 threshold를 훑으면서 PR 점들을 만들고, 그 전체를 면적으로 요약한 게 `AP`다.

---

## 3) `AP@50`(= `AP50`) 같은 지표는 무엇이고, `mAP`와 무엇이 다르나?

`AP@50`은 **`IoU threshold = 0.50`**로 TP/FP를 판정한 뒤, 그 상태에서 `confidence threshold`를 sweep해서 PR 면적(`AP`)을 계산한 값이다.

`mAP`(COCO 스타일)은 보통:

- `IoU = 0.50, 0.55, ..., 0.95`에서의 `AP`를 각각 구하고
- 그걸 평균낸 값

즉:

- `AP@50`: “정답 판정(`IoU`)을 느슨하게 두었을 때의 탐지 성능(랭킹 포함)”
- `mAP`: “느슨한 판정부터 매우 엄격한 판정까지 전부 평균낸 종합 성능”

---

## 4) `confidence threshold` 얘기하다가 왜 `IoU threshold`가 같이 나오나?

객체탐지 평가는 threshold가 사실상 2개다.

- `confidence threshold`: “이 박스를 출력에 포함할까?”
- `IoU threshold`: “이 박스를 TP로 인정할까?”

`AP/mAP`를 계산할 때는 보통:

1) `IoU threshold`를 고정해서 TP/FP 판정 규칙을 정하고
2) 그 상태에서 `confidence threshold`를 sweep하며 PR을 만든다

그래서 `confidence` 얘기를 하다 보면, TP/FP 기준을 정하는 `IoU`가 같이 등장할 수밖에 없다.

---

## 5) `IoU threshold=0.50` 같은 설정이 `Precision/Recall`에 실제로 영향을 주나?

직접 영향을 준다. `Precision`과 `Recall` 둘 다 바뀐다.

- `IoU threshold`를 올리면(0.50 → 0.75):
  - “살짝 어긋난 박스”가 TP에서 탈락한다(= FP가 되거나, 매칭 실패로 FN이 늘 수 있음)
  - 그래서 `Precision`은 내려가기 쉽고 `Recall`도 내려가기 쉽다
- `IoU threshold`를 내리면 반대다

즉 `IoU threshold`는 “정밀도 요구 수준(엄격도 레버)”이고, PR 곡선 자체를 바꾸는 기준이다.

---

## 6) 실무에서 “누락이 적은가”를 보려면 무엇을 봐야 하나?

“운영에서 threshold를 어느 정도로 둘 때 누락이 적냐”가 궁금하면 `mAP`가 아니라 아래를 봐야 한다.

- `Recall @ confidence=0.3`
- `Recall @ confidence=0.5`
- `Recall @ confidence=0.7`

즉, 고정된 `confidence threshold`에서의 `Recall`(또는 `FN`)을 별도로 확인해야 한다.

---

## 7) 요약: `mAP`가 높다는 말이 내포하는 의미는 무엇인가?

`mAP`가 높다는 말은 대충 “좋다”가 아니라, 더 구체적으로는 이런 의미를 내포한다.

- `confidence threshold`를 sweep했을 때 PR 곡선이 전반적으로 위에 있다(= 같은 `Recall`을 얻을 때 `FP`가 덜 들어오고, 같은 `Precision`을 얻을 때 덜 놓친다).
- `IoU threshold`를 엄격하게 해도 성능이 유지된다(= 박스가 더 타이트하고 localization이 좋다).
- `confidence ranking` 품질이 좋다(= 진짜가 위, 가짜가 아래로 정렬된다).

반대로, `mAP`만으로는 “운영 threshold에서 누락이 적다”를 확정할 수 없다.

---

## 마지막으로, 기억할 한 문장
`mAP`는 “confidence를 낮춰도 누락이 적다”가 아니라, `confidence threshold`를 sweep했을 때(`IoU` 기준 포함) `Precision–Recall` trade-off를 얼마나 안정적으로 잘 유지하는지(=랭킹+localization 품질)를 요약한 값이다.

