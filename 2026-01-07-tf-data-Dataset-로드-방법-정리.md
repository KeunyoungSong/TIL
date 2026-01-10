# `tf.data.Dataset` 로드(생성) 방법 정리

> Type: Concept

## 상황/배경(Context)
TensorFlow에서 “데이터셋을 준비한다”는 말이 애매하게 느껴졌다. `tf.data.Dataset`으로 학습을 하려면 결국 “Dataset을 어디서 만들고(소스), 어떻게 변환해서(batch/전처리/최적화) `fit()`에 넣는지”가 핵심이라, 자주 쓰는 생성 루트를 먼저 고정해 둔다.

---

## 정의(Definition)
`tf.data.Dataset`은 “(샘플 또는 배치) 스트림”을 표현하는 입력 파이프라인 API다. 보통 흐름은 아래 2단계다.
- **Source**: Dataset을 만든다(메모리/파일/TFRecord/공개 데이터 등).
- **Transform**: `map`/`shuffle`/`batch`/`prefetch`로 파이프라인을 구성한다.

## 핵심 아이디어(Key Ideas)
- “로드 방법”은 결국 **Dataset을 만드는 Source 선택**이다.
- 성능/확장성 기준으로는 “파이썬 제너레이터”보다 **파일 기반 + `map(..., num_parallel_calls=...)` + `prefetch`**가 유리한 경우가 많다.
- Keras의 여러 로딩 유틸(`image_dataset_from_directory` 등)도 결과는 대부분 `tf.data.Dataset`로 수렴한다.

## 예시(Examples)
자주 쓰는 Source 루트들:

### 1) 메모리(배열/텐서/리스트) → `from_tensor_slices`
```python
ds = tf.data.Dataset.from_tensor_slices((x, y))
ds = ds.shuffle(1000).batch(64).prefetch(tf.data.AUTOTUNE)
```

### 2) 파일 패턴(여러 파일) → `list_files` + `map(read/decode)`
```python
files = tf.data.Dataset.list_files("/data/train/*.jpg", shuffle=True)
ds = files.map(tf.io.read_file, num_parallel_calls=tf.data.AUTOTUNE)
ds = ds.map(decode_and_resize, num_parallel_calls=tf.data.AUTOTUNE)
ds = ds.batch(64).prefetch(tf.data.AUTOTUNE)
```

### 3) 레코드 포맷 → `TFRecordDataset`
```python
raw = tf.data.TFRecordDataset(["train-000.tfrecord", "train-001.tfrecord"])
ds = raw.map(parse_example, num_parallel_calls=tf.data.AUTOTUNE)
ds = ds.batch(256).prefetch(tf.data.AUTOTUNE)
```

### 4) 텍스트/CSV
- 텍스트(라인 단위): `TextLineDataset(...)` + `map(parse_line)`
- CSV(빠른 시작): `tf.data.experimental.make_csv_dataset(...)` (배치를 바로 생성)

### 5) Keras 유틸(디스크 → Dataset)
디렉토리 기반 이미지 분류에서 자주 쓰는 루트:
```python
train_ds = keras.utils.image_dataset_from_directory(
    "/data/train",
    image_size=(224, 224),
    batch_size=64,
    shuffle=True,
)
train_ds = train_ds.prefetch(tf.data.AUTOTUNE)
```

### 6) 공개 데이터 → TFDS(`tfds.load`)
```python
(ds_train, ds_val), info = tfds.load(
    "mnist",
    split=["train", "test"],
    as_supervised=True,
    with_info=True,
)
```

### 7) (주의) 파이썬 제너레이터 → `from_generator`
파이썬 로직이 핵심일 때 마지막 수단으로 쓸 수 있지만, 병목이 되기 쉽다.

## 주의할 점(Caveats)
- `from_tensor_slices`는 편하지만 “메모리에 다 올리는” 전제에 가깝다(대용량이면 파일/TFRecord 루트를 먼저 고려).
- `list_files + map(decode)`는 유연하지만, 라벨 파싱/샘플링 정책이 복잡해지면 코드가 길어질 수 있다.
- “증강(augmentation)을 어디에 두나”도 로딩 설계의 일부다:
  - 모델 안(preprocessing layers) vs `Dataset.map`(tf.image 등) vs 파이썬 로더(Sequence)

## 한 줄 요약(Summary)
`tf.data.Dataset` 로드는 “Dataset을 만드는 Source 선택”이고, 실전에서는 파일/TFRecord/유틸로 Dataset을 만든 뒤 `map → batch → prefetch`로 파이프라인을 고정하는 게 기본이다.

---

## 역사/배경(Timeline/Why)
- TF1 시절엔 Queues/Queue runner가 입력 표준처럼 쓰였고 운영 난도가 높았다.
- 2017년 이후 `tf.data`가 “입력 파이프라인 표준 추상화”로 자리잡으면서, Source/Transform의 일관된 모델이 정착했다.
- TF2에서 `tf.keras`가 중심이 되면서도, 성능/분산과 궁합이 좋아 `tf.data`가 기본 입력 루트로 굳어졌다.

## 참고(Links)
- https://www.tensorflow.org/guide/data
- https://www.tensorflow.org/guide/data_performance
- https://keras.io/api/data_loading/
