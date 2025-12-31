# ReduceLROnPlateau: 왜 쓰나, 언제 통하나

> Type: Concept

## 상황/배경(Context)
훈련을 돌리다 보면 `val_loss`나 `val_accuracy` 같은 지표가 한동안 개선되지 않는(plateau) 구간이 생긴다.  
이때 학습률(Learning rate, LR)을 줄이면 좋아진다는 말을 자주 듣지만, “항상 도움이 되나?”는 별개 문제라서 정리한다.

---

## 정의(Definition)
**ReduceLROnPlateau**는 지정한 지표(`monitor`)가 일정 기간(`patience`) 개선되지 않으면 학습률을 `lr ← lr * factor`로 줄이는 **plateau 기반 learning rate scheduler**다.

## 핵심 아이디어(Key Ideas)
- LR을 줄인다고 항상 성능이 좋아지진 않는다. 다만 **최적점 근처에서 overshoot/진동**이 문제일 때는 “마무리 수렴”에 도움이 될 수 있다.
- 고정 스케줄(step/cosine 등) 대신, **지표 변화에 반응하는 적응형 스케줄**이라서 “언제 LR을 내릴지”를 덜 추측해도 된다.
- plateau는 (1) 정말로 더 못 배우는 신호일 수도 있고, (2) LR이 커서 못 내려가는 신호일 수도 있다. ReduceLROnPlateau는 (2)에 특히 유효하다.

## 역사/배경(Timeline/Why)
- polling처럼 “정해진 시점에 LR을 내리는” 방식은 간단하지만, plateau 시점이 문제마다 달라 과하게 빠르거나 늦을 수 있다.
- 그래서 “실제로 지표가 멈췄을 때만 LR을 낮추자”는 형태의 스케줄이 실무에서 널리 쓰였다.
- 프레임워크(Keras/PyTorch)가 이를 지원하는 이유는, 학습 파이프라인에서 **안전한 기본값(sensible default)**로 자주 쓰이기 때문이다.

## 도식(Diagram)
```
epoch:    1 2 3 4 5 6 7 8 9 10 ...
val_loss: ↓ ↓ ↓ → → → → ↓  ↓  ...
                 ^patience 구간(개선 없음)
                 lr := lr * factor
```

## 원리/수식(Principle/Math)
“개선”의 정의는 보통 다음 요소로 정해진다.
- `mode='min'`: 지표가 **감소**해야 개선 (`val_loss`)
- `mode='max'`: 지표가 **증가**해야 개선 (`val_accuracy`)
- `min_delta`: 개선으로 인정할 최소 변화량

알고리즘(요지):
1. 매 epoch마다 `monitor` 값을 관찰한다.
2. 직전 최고(best) 대비 `min_delta`만큼 개선되면 best를 갱신하고 카운터를 0으로 리셋한다.
3. 개선이 없으면 카운터를 +1 한다.
4. 카운터가 `patience`에 도달하면 `lr ← max(lr * factor, min_lr)`로 줄이고, (옵션) `cooldown` 동안은 카운터 증가를 잠시 멈춘다.

## 예시(Examples)
### 언제 도움이 되나
- **LR이 커서** 최적점 근처에서 손실이 들쭉날쭉하고 평균적으로는 안 내려가는 경우(overshoot/진동).
- 큰 LR로 빠르게 내려온 뒤, 더 작은 LR로 **미세 조정(fine-tuning)**이 필요한 경우.

### 언제 “거의” 안 통하나
- **underfitting**(모델 용량/특징/학습 시간이 부족): LR을 낮추면 더 못 배울 수 있다.
- 데이터/라벨/분포 문제, 지나친 regularization(weight decay, dropout 등): 원인이 LR이 아니면 효과가 제한적이다.
- `monitor`가 너무 noisy(작은 validation set 등): “개선 없음”을 오판해 불필요하게 LR을 떨어뜨릴 수 있다.

### 파라미터 감각(실무에서 자주 하는 선택)
- `monitor='val_loss'`, `mode='min'`: 비교적 안정적(연속적인 값이라 변동이 덜함).
- `patience`: 지표 변동폭보다 충분히 길게(너무 짧으면 노이즈에 반응).
- `factor`: `0.1`은 공격적일 수 있어 `0.5` 같은 완만한 감소도 후보.
- `min_lr`: LR이 너무 낮아져 학습이 사실상 멈추는 걸 방지.

## 주의할 점(Caveats)
- **“지표가 안 좋아짐” vs “안 좋아지진 않지만 정체”**: overfitting이면 LR을 낮추는 것보다 early stopping/regularization/데이터 쪽이 우선일 수 있다.
- **다른 scheduler와 충돌**: cosine/one-cycle 같은 스케줄을 이미 쓰고 있으면 plateau 스케줄을 겹치지 않는 게 보통 더 깔끔하다.
- **평가지표 선택**: accuracy는 계단형으로 변할 수 있어 plateau 판정이 예민해질 수 있다. 가능하면 `val_loss`를 monitor로 두는 편이 안전한 경우가 많다.
- **중복 이벤트/재현성**: seed에 따라 plateau 타이밍이 달라질 수 있으니, “한 번 좋아졌다/나빠졌다”로 결론 내리기보다 로그를 같이 본다.

## 한 줄 요약(Summary)
ReduceLROnPlateau는 “지표가 멈췄을 때 LR을 낮춰 마무리 수렴을 돕는” 도구지만, plateau 원인이 LR이 아닐 때는 효과가 없거나 학습만 느려질 수 있다.

## 참고(Links)
- https://keras.io/api/callbacks/reduce_lr_on_plateau/
- https://pytorch.org/docs/stable/generated/torch.optim.lr_scheduler.ReduceLROnPlateau.html

