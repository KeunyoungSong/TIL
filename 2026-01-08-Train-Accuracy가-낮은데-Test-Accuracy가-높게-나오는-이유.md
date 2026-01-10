# Train Accuracy가 낮은데 Test Accuracy가 높게 나오는 이유

> Type: Troubleshooting

## 상황/배경(Context)
fine-tuning 학습 로그에서 train accuracy가 0.63~0.68 수준인데, validation/test accuracy가 0.82~0.83처럼 훨씬 높게 나오는 경우가 있다. “과적합”의 반대처럼 보여서 당황하지만, 원인은 보통 **측정 조건의 차이**거나(정상), **데이터/라벨 파이프라인 문제**(점검 필요)다.

---

## 문제(Problem)
- train acc가 낮은데 val/test acc가 훨씬 높게 나온다.

## 원인(Cause)
가능한 원인(정상 원인 → 위험 원인 순):

1) **Train에만 강한 augmentation이 적용됨**
- train metric은 “증강된(난이도 높은) 입력”에 대해 계산되고,
- val/test는 증강 없이 “깨끗한 분포”로 평가되면,
- train acc < val/test acc가 자연스럽게 나온다.

2) **Training 모드 vs Inference 모드 차이(Dropout/BatchNorm)**
- `fit()` 중 metric은 보통 training 모드에서 누적되고,
- `evaluate()`는 inference 모드에서 계산된다.
- Dropout은 학습 중 활성/평가 시 비활성이라, eval 쪽이 더 잘 나올 수 있다.
- BatchNorm도 학습 중 batch 통계 vs 평가 시 moving average를 쓰면서 평가가 더 안정적으로 높게 나올 수 있다.

3) **train acc는 epoch “누적 평균”이라 낮게 보일 수 있음**
- epoch 초반 batch에서 낮게 나오면, 후반에 좋아져도 평균이 낮게 남을 수 있다.
- 반면 val/test는 한 번에 전체 세트로 깔끔하게 측정되는 경우가 많다.

4) **(중요) 데이터 누수/분할 문제**
- train/val/test에 중복 이미지가 섞이면 test가 비정상적으로 높아질 수 있다.
- 같은 개체/연속샷/크롭 변형이 split을 가로지르면 “사실상 유사 이미지” 누수도 생긴다.

5) **(숨은 함정) one-hot 컬럼 순서 불일치**
- split별로 `pd.get_dummies()`를 따로 돌리면, 컬럼 순서/존재가 달라질 수 있다.
- 이 경우 보통 성능이 무너지는 편이지만, 데이터 구성에 따라 “우연히” 높게 보일 가능성도 있어 점검 가치가 있다.

## 해결(Solution)
가장 빠르게 원인을 확정하는 체크 순서.

1) **Train을 “증강 없이” 다시 평가해 비교**
```python
train_ds_noaug = Breed_Dataset(
    train_path, train_label,
    augmentor=None, shuffle=False,
    pre_func=eff_preprocess_input,
)
model.evaluate(train_ds_noaug)
```
- 여기서 train_noaug acc가 val/test와 비슷해지면, 원인은 거의 “train만 augmentation/학습 모드” 쪽이다.

2) **Dropout/BatchNorm 영향 확인**
- 모델에 Dropout/BN이 있는지 확인하고,
- training/eval 모드 차이로 metric이 벌어질 수 있음을 전제로 해석한다.

3) **라벨 매핑(특히 one-hot) 컬럼 순서 강제 고정**
```python
# train에서 사용한 class_columns를 기준으로 test도 reindex
test_onehot = (
    pd.get_dummies(test_df["label"])
    .reindex(columns=class_columns, fill_value=0)
    .values
)
```

4) **split 누수/중복 점검**
- 파일명/해시/원본 ID 기준으로 train/val/test 중복 여부 확인
- 같은 개체/연속 프레임이 분할을 가로지르지 않는지 확인

## 결과/현재 상태(Result/Status)
이 현상은 “train만 증강” 또는 “train/eval 모드 차이”면 정상 패턴일 수 있다.  
하지만 데이터 누수/라벨 매핑 불일치 가능성도 있으므로, 위 체크 순서로 빠르게 확정하는 게 안전하다.

## 배운 점(Takeaways)
- train acc와 test acc는 “같은 조건에서 측정된 숫자”가 아닐 수 있다(augmentation, training/eval mode).
- 파라미터/모델 문제가 아니라 **입력 파이프라인/라벨 파이프라인**이 지표를 왜곡시키는 경우가 많다.
- 의심되면 “train을 augmentation 없이 evaluate”가 가장 빠른 판별법이다.
