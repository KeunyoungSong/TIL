# Transfer Learning과 Fine-tuning 관계

> Type: Concept

## 상황/배경(Context)
`pretrained` 모델을 가져다 쓰는 상황에서 “전이학습”과 “파인튜닝”을 같은 말처럼 쓰다가, 둘의 포함 관계와 실무에서의 선택 기준을 짧게 고정해두고 싶었다.

---

## 정의(Definition)
- **Transfer learning**: 큰 데이터/다른 과제로 미리 학습된 모델의 지식(가중치/표현)을 내 과제에 **재사용**하는 것(상위 개념).
- **Fine-tuning**: transfer learning을 적용하는 방법 중 하나로, 가져온 모델의 **일부/전체 가중치를 내 데이터로 다시 학습(업데이트)** 하는 것(하위 개념).

## 핵심 아이디어(Key Ideas)
- 관계는 `transfer learning ⊃ fine-tuning`이다. (fine-tuning은 transfer learning의 한 방식)
- transfer learning은 “어디서 왔는지(사전학습)”보다 “어떻게 가져다 쓰는지(재사용)”에 초점이 있다.
- 보통 모델을 `feature extractor(backbone) + classifier(head)`로 보고, 어떤 부분을 학습할지로 전략이 갈린다.

## 예시(Examples)
- **Feature extraction**: backbone은 `freeze`하고 head만 학습한다.
  - 데이터가 적거나, 빠르게 baseline을 만들고 싶을 때.
- **Fine-tuning**: head를 붙여 학습한 뒤 backbone의 일부/전체도 추가로 학습한다(보통 더 작은 learning rate).
  - 내 데이터 도메인이 다르거나(예: 의료/야간/열화상), 성능을 더 끌어올리고 싶을 때.

## 주의할 점(Caveats)
- “전이학습 = ImageNet으로 전체 모델을 먼저 학습”은 흔한 사례일 뿐, 전이학습의 정의 조건은 아니다. (`pretrained`가 ImageNet일 수도, 다른 데이터/자기지도학습일 수도 있음)
- 용어 혼용에 주의:
  - head만 학습하는 경우도 넓게는 transfer learning이지만, 문맥상 “fine-tuning”이라고 부르면 오해가 생길 수 있다.
- fine-tuning은 데이터/학습 설정에 따라 **catastrophic forgetting**이나 과적합이 생길 수 있어, 학습 범위(backbone 어디까지)와 learning rate를 보수적으로 잡는 편이 안전하다.

## 한 줄 요약(Summary)
Transfer learning은 “재사용”이라는 큰 틀이고, fine-tuning은 그 안에서 `pretrained` 가중치를 실제로 다시 학습시키는 방법이다.

---

## 용어(Glossary)
- **pretrained**: 큰 데이터/다른 과제로 미리 학습된 상태(가중치 초기값으로 가져옴).
- **backbone**: 특징을 뽑는 공통 본체(`feature extractor`).
- **head**: 과제에 맞춘 마지막 부분(분류/회귀/탐지 등).
