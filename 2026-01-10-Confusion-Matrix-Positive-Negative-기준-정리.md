# Confusion Matrix에서 Positive/Negative 기준 정리

> Type: Concept

## 상황/배경(Context)
`threshold`(임계값/기준값)와 Confusion Matrix를 보다가, `Positive/Negative(P/N)`가 “실제 상태”인지 “모델 예측”인지 문맥에 따라 섞여서 헷갈렸다. 특히 의료(암 진단) vs Object Detection에서 설명 방식이 달라서, 같은 단어를 다른 축에 붙이고 있다는 걸 뒤늦게 깨달았다. 나중에 다시 봐도 같은 흐름으로 이해에 도달할 수 있게 “내 질문의 이동 경로”를 남긴다.

---

## 내가 먼저 고정한 한 문장(결론)
Confusion Matrix는 항상 `Actual` vs `Predicted` 두 축이고, `P/N`은 그 축의 라벨일 뿐이라서 **먼저 “Positive가 어떤 축의 무엇인지”를 선언**해야 한다. (Object Detection은 보통 `Prediction event`와 `Evaluation`을 2단계로 분리한다.)

## 내가 헷갈렸던 이유
- `P/N`을 한 글자로 말하면, 그게 `Actual`인지 `Predicted`인지 생략되기 쉽다.
- 의료(분류)는 `Actual P/N`이 명확해서 “P/N=실제 상태” 관례가 강한데, Object Detection은 “탐지했다/안 했다” 같은 **행동(event)** 중심으로 설명이 시작된다.
- `threshold`가 하나가 아니라 단계별로 다르다(`decision threshold`, `confidence threshold`, `IoU threshold`). 이걸 한 덩어리로 말하면 정의가 뒤틀린다.

## 내가 던진 질문(흐름)
- “`threshold`는 임계점/임계값이 맞나, 문맥에 따라 뭐라고 부르는 게 좋은가?”
- “Object Detection에서 Positive/Negative는 ‘실제 상태’야, ‘모델이 탐지했는지’야?”
- “의료(암 진단)에서는 P/N을 환자(Actual)로 봐야 돼, 예측(Predicted)으로 봐야 돼?”
- “그럼 내가 느낀 ‘기준이 바뀌는 느낌’은 기준이 바뀐 게 아니라, 축을 바꿔 말한 거였나?”
- “P/N 표가 있고 Pred 표가 따로 있는 건가, 아니면 표는 하나인데 내가 축을 놓친 건가?”

## 내가 정한 기준(의미 고정)
1) **표는 하나**: Confusion Matrix는 항상 `Actual`(세로) vs `Predicted`(가로)다.
2) **P/N은 라벨**: `Actual P/N`과 `Predicted P/N`을 섞지 않는다. (말로만 “P/N”이라고 하지 말고 축을 같이 말한다.)
3) **Object Detection은 2단계**로 분리한다.
   - `Prediction event`: 박스 예측 + `confidence score ≥ confidence threshold` (예측을 “낼지 말지”)
   - `Evaluation`: GT와 매칭해서 `class 일치` + `IoU ≥ IoU threshold`면 정답 인정 (예측이 “맞았는지”)
4) `threshold`를 말할 때는 “어느 단계의 threshold인지”를 같이 붙인다.

## 예시(Examples)
아래에 **같은 형식의 오차행렬(Confusion Matrix)**을 ① 암 진단, ② 스팸메일 분류로 **나란히** 정리한다. 표 구조는 동일하지만, “비용이 큰 오류(리스크)”가 어디에 있느냐에 따라 해석 포인트와 지표 우선순위, `threshold` 전략이 달라진다.

### 1) 암 진단 (의료 진단)
#### 기준
- **Actual (실제 상태)**: P = 실제 암 환자, N = 실제 비암 환자
- **Predicted (예측)**: P = 암이라고 예측, N = 암이 아니라고 예측

#### 오차행렬
| 실제 상태 \ 예측 | Pred N (암 아님) | Pred P (암) |
| --- | --- | --- |
| **Actual N (비암)** | **TN** 정상 판정 | **FP** 오진(과잉 진단) |
| **Actual P (암)** | **FN** ⚠️ 미진단 | **TP** 정확 진단 |

#### 의료적 해석 포인트
- **FN (False Negative)**: 실제 암인데 놓침 → 치료 기회 상실로 비용이 매우 큼
- **FP (False Positive)**: 불필요한 검사·불안 유발
- `Recall(Sensitivity)`를 우선시하는 문맥이 많음: $\frac{TP}{TP+FN}$

### 2) 스팸메일 분류
#### 기준
- **Actual (실제 상태)**: P = 실제 spam email, N = 실제 정상 email
- **Predicted (예측)**: P = spam으로 분류, N = 정상으로 분류

#### 오차행렬
| 실제 상태 \ 예측 | Pred N (정상) | Pred P (spam) |
| --- | --- | --- |
| **Actual N (정상)** | **TN** 정상 수신 | **FP** ⚠️ 정상 email 차단 |
| **Actual P (spam)** | **FN** spam 통과 | **TP** spam 차단 |

#### 서비스적 해석 포인트
- **FP (False Positive)**: 중요한 email이 spam으로 가서 UX에 치명적
- **FN (False Negative)**: spam이 하나 더 옴 → 상대적으로 허용 가능한 경우가 많음
- `Precision`을 우선시하는 문맥이 많음: $\frac{TP}{TP+FP}$

### 3) 두 문제의 구조는 동일, “리스크 위치”만 다름
| 항목 | 암 진단 | 스팸메일 |
| --- | --- | --- |
| 표 구조 | 동일 | 동일 |
| Actual P | 암 환자 | spam email |
| Pred P | 암으로 진단 | spam 분류 |
| 가장 위험한 오차 | **FN** | **FP** |
| 우선 최적화 지표 | `Recall/Sensitivity` | `Precision` |

> 암 진단과 스팸메일은 오차행렬 구조는 완전히 동일하지만, 어떤 칸이 “비용이 큰 오류”인지가 다르기 때문에 모델 설계와 `threshold` 전략이 달라진다.

### 4) Object Detection: “예측 이벤트”와 “평가”를 2단계로 분리
- `Prediction event(모델 행동)`: 박스 예측 + `confidence score ≥ confidence threshold`
- `Evaluation(사후 판정)`: GT와 매칭해서 `class 일치` + `IoU ≥ IoU threshold`면 정답 인정

이 구조에서는 보통 아래만 명시적으로 쓴다.
- `TP`: 탐지했고, 매칭도 성공(class + IoU 기준 통과)
- `FP`: 탐지했지만, 매칭 실패(오탐/위치 틀림/클래스 틀림 등)
- `FN`: 탐지하지 못했는데, 실제로는 객체가 존재
- `TN`: 배경을 “얼마나”로 셀지 정의가 어려워서 일반 평가(Precision/Recall/mAP)에서는 거의 계산하지 않음

## 주의할 점(Caveats)
- “P = 조건을 통과했다” 같은 문장은 `Prediction(행동)`과 `Evaluation(판정)`을 섞기 쉬우니 피한다.
- `threshold`는 단계별로 의미가 다르다.
  - 분류: decision threshold(예: $p \ge 0.5$면 positive)
  - 탐지: `confidence threshold`(예측을 낼지 말지) vs `IoU threshold`(맞았다고 인정할지)
- `Precision`/`Recall`은 각각 100%여도 “좋은 모델”을 뜻하지 않을 수 있다.
  - **Precision 100%의 맹점**: `FN`은 분모에 안 들어간다. 기준을 엄격하게 잡아 “확실한 1개만” `Pred P`로 내고 그 1개를 맞추면 $\frac{TP}{TP+FP}=1$이 된다(하지만 많은 `Actual P`를 놓쳐 `Recall`은 낮을 수 있음).
  - **Recall 100%의 맹점**: `FP`는 분모에 안 들어간다. 전부 `Pred P`로 예측하면 $\frac{TP}{TP+FN}=1$이 될 수 있다(하지만 `Actual N`까지 양성으로 만들어 `Precision`은 크게 떨어질 수 있음).

## 복구용 체크 질문
- “여기서 Positive는 **Actual(실제 상태)**인가, **Predicted(모델 판단/행동)**인가?”
- “내가 말하는 `threshold`는 decision/confidence/IoU 중 **어느 단계**인가?”
- “지금 문제는 ‘모든 샘플이 P/N 중 하나’(분류)인가, ‘예측 이벤트를 평가’(Detection)인가?”

## 다음에 더 빨리 도달하기 위한 질문
- “Confusion Matrix를 그리기 전에, `Actual P/N`과 `Predicted P/N` 정의를 한 줄씩 써줄래?”
- “이 문제에서 `TN`은 정의/카운팅이 가능한가? 안 되면 왜 안 되는가?”
- “내가 최적화해야 하는 건 `FP` 비용이 큰 문제인가, `FN` 비용이 큰 문제인가? 그럼 어떤 지표/threshold를 먼저 보나?”

## 한 줄 요약(Summary)
Confusion Matrix는 `Actual vs Predicted` 한 표로 고정하고, `P/N`은 어느 축의 라벨인지 먼저 선언하면(그리고 Detection은 event/evaluation을 분리하면) 기준이 흔들리지 않는다.

## 다음으로 이어질 수 있는 주제(Next)
- 왜 암 진단은 `Recall` 우선, 스팸은 `Precision` 우선인가(비용/리스크 관점)
- 같은 모델에서 `threshold`를 어떻게 다르게 잡는가(결정 경계 이동)
- ROC Curve / PR Curve가 두 문제에서 다르게 해석되는 이유

## 역사/배경(Timeline/Why)  (Concept 권장)
- 의료/분류는 “샘플 단위로 Actual P/N이 명확”해서 `TN`까지 포함한 Confusion Matrix가 자연스럽게 정의된다.
- 의료는 `FN`(놓친 암 환자)의 비용이 크기 때문에 `Sensitivity/Specificity`처럼 Actual 축 기준 지표가 강하게 쓰인다.
- Object Detection은 “배경(negative)의 단위/개수”가 명확하지 않아 `TN`을 명시적으로 세기 어렵고, 실무 평가는 `TP/FP/FN` 기반의 Precision/Recall(및 mAP)로 굳어졌다.
- 그래서 탐지 평가는 보통 `confidence threshold`로 “예측을 낼지”를 제어하고, `IoU threshold`로 “맞았다고 인정할지”를 제어하는 2단계 구조가 표준처럼 자리 잡았다.

## 용어(Glossary)
- **threshold**: 상태/판정이 바뀌는 경계값(단계별 의미를 분리해서 써야 함)
- **Confusion Matrix**: Actual vs Predicted 조합으로 TP/FP/FN/TN을 정리한 표
- **IoU (Intersection over Union)**: 예측 box와 GT box의 겹침 정도를 나타내는 값
