# TensorFlow/Keras 입력 파이프라인이 “자꾸 바뀌는” 것처럼 느껴지는 이유

> Type: Concept

## 상황/배경(Context)
TensorFlow/Keras를 따라가다 보면 데이터 로딩/학습 입력 쪽이 계속 바뀌는 느낌이 든다. 그런데 이 변화는 “함수명 변경” 수준이 아니라 실행 모델, 고수준 API의 중심, 입력 파이프라인 표준이 단계적으로 재정렬된 결과로 보면 더 명확하다.

---

## 정의(Definition)
이 글에서 말하는 “바뀐다”는 느낌은 주로 아래 3축의 중심이 이동한 것을 뜻한다.
- **실행 모델**: TF1의 `Graph/Session` → TF2의 `eager` 기본(+ 필요 시 `tf.function`으로 graph 컴파일)
- **고수준 API 중심**: Keras가 TensorFlow의 기본 고수준 API로 통합(`tf.keras.Model.fit()` 중심)
- **입력 파이프라인 표준**: Queues/제너레이터 중심 → `tf.data.Dataset` 중심

## 핵심 아이디어(Key Ideas)
- TF1에서 “입력 파이프라인”은 운영 난도가 높았다(스레드/종료/예외 처리 등). 그래서 커뮤니티 생산성은 종종 Keras 제너레이터로 기울었다.
- TF2로 오면서 “훈련 루프의 표준”이 `Model.fit()`로 수렴했고, “성능/분산/그래프 실행과 궁합이 좋은 표준 입력”으로 `tf.data`가 밀려났다.
- 오늘의 권장 흐름은 “로딩은 `tf.data.Dataset`으로 만들고, 전처리/증강은 preprocessing layers(또는 `tf.data`의 `map`)로 붙인다”이다.

## 예시(Examples)
오늘 기준(TF 백엔드)으로 입력을 선택할 때의 지도:
- **정석(권장)**: `tf.data.Dataset`
  - 로딩: `keras.utils.image_dataset_from_directory` 등(결과가 `Dataset`)
  - 최적화: `map(num_parallel_calls=...)`, `prefetch`, `cache` 등
  - 증강: preprocessing layers 또는 `Dataset.map`
- **특수/불가피**: `Sequence` / `PyDataset`
  - OpenCV/Albumentations처럼 “파이썬 로직이 핵심”이거나 TF로 옮기기 어려운 샘플링/조합이 필요한 경우
- **레거시/프로토타입**: `ImageDataGenerator`
  - 새 코드에서는 비추천(Deprecated로 분류)

## 주의할 점(Caveats)
- “Keras 제너레이터를 쓰면 안 된다”가 아니라, **표준 파이프라인은 `tf.data`로 정리**되었고 `fit()`이 필요 시 여러 입력 소스를 받아들이는 방향으로 **어댑터가 확장**되었다고 보는 편이 정확하다.
- 동일한 “입력”이라도 목표가 다르면 선택이 갈린다:
  - 분산/성능/재현성 중심이면 `tf.data`가 유리한 경우가 많고
  - 복잡한 파이썬 기반 변환/샘플링이 핵심이면 `Sequence`/`PyDataset`가 현실적일 수 있다.

## 한 줄 요약(Summary)
TensorFlow/Keras가 “자꾸 바뀌는” 느낌은 실행 모델(TF1→TF2), 고수준 API의 중심(Keras 통합), 입력 표준(Queues/제너레이터→`tf.data`)이 단계적으로 재정렬된 결과다.

---

## 역사/배경(Timeline/Why)
데이터 로딩/학습 입력 관점에서의 큰 흐름:
- **2015–2017(TF1)**: `Graph/Session` + Queue runner 계열 입력이 흔함(`tf.train.string_input_producer` 등)
- **2017-11(TF 1.4)**: Dataset API(= `tf.data` 계열) 등장 → Queues를 대체할 “표준 추상화”로 밀기 시작
- **2017–2019(Keras 2.x 확산)**: 커뮤니티 생산성은 `ImageDataGenerator`/`Sequence`로 기울기도 함(입력이 쉬웠음)
- **2019-09(TF 2.0)**: eager 기본 + Keras가 기본 고수준 API → 입력도 점점 `tf.data` 표준 루트로 정리
- **2020s**: `ImageDataGenerator`는 레거시화(Deprecated) + “`Dataset` + preprocessing layers” 권장
- **2023+(Keras 3)**: `fit()` 입력 어댑터가 더 범용화(`tf.data` 외에도 NumPy/Pandas/PyTorch DataLoader 등)

## 참고(Links)
- TensorFlow r1.4 발표(Dataset API): https://developers.googleblog.com/announcing-tensorflow-r14/
- `tf.data` 논문: https://vldb.org/pvldb/vol14/p2945-klimovic.pdf
- TensorFlow 2.0 릴리스: https://blog.tensorflow.org/2019/09/tensorflow-20-is-now-available.html
- `tf.data` 가이드: https://www.tensorflow.org/guide/data
- `ImageDataGenerator` API 문서(Deprecated): https://www.tensorflow.org/api_docs/python/tf/keras/preprocessing/image/ImageDataGenerator
- Keras data loading(유틸 → `tf.data.Dataset`): https://keras.io/api/data_loading/
- Keras 3 소개: https://keras.io/keras_3/
- `PyDataset`: https://www.tensorflow.org/api_docs/python/tf/keras/utils/PyDataset
- `tf.data` 성능 가이드: https://www.tensorflow.org/guide/data_performance
