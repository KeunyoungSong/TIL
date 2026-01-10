# `tf.data.Dataset`과 Keras `Sequence` 차이(코드 관점) 정리

> Type: Concept

## 상황/배경(Context)
강의에서 “Keras `Sequence`로 dataset 직접 구현”을 보는데, `tf.data.Dataset`으로 만드는 것과 무엇이 본질적으로 다른지 코드 레벨에서 헷갈렸다. 둘 다 `model.fit()`에 배치를 공급한다는 공통점은 있지만, “배치를 어디서/누가 만드는지”가 다르다.

---

## 정의(Definition)
- `tf.data.Dataset`: TF 런타임이 최적화할 수 있는 **선언형 파이프라인**(`shuffle → map → batch → prefetch`)으로 배치를 만든다.
- `keras.utils.Sequence`: 파이썬 클래스의 `__getitem__`이 **배치 하나를 직접 구성**해서 `fit()`에 넘긴다.

## 핵심 아이디어(Key Ideas)
- 결정적 차이는 “배치 생성 위치”다.
  - `tf.data`: 파이프라인 연산 체인이 배치를 만든다.
  - `Sequence`: 파이썬 코드가 배치를 만든다.
- `tf.data`는 `num_parallel_calls`, `prefetch` 등으로 파이프라인 최적화가 자연스럽고 분산과 궁합이 좋다.
- `Sequence`는 샘플 조립/라벨 구조가 복잡하거나(탐지/세그), 외부 파이썬 증강(Albumentations/OpenCV)이 핵심일 때 현실적이다.

## 예시(Examples)
같은 입력(이미지 경로 + 정수 라벨) 기준 최소 골격 비교.

### A) `tf.data.Dataset`
```python
AUTOTUNE = tf.data.AUTOTUNE

def load_decode_resize(path, label):
    img_bytes = tf.io.read_file(path)
    img = tf.image.decode_jpeg(img_bytes, channels=3)
    img = tf.image.resize(img, (224, 224))
    img = tf.cast(img, tf.float32) / 255.0
    return img, label

ds = tf.data.Dataset.from_tensor_slices((paths, labels))
ds = ds.shuffle(len(paths))
ds = ds.map(load_decode_resize, num_parallel_calls=AUTOTUNE)
ds = ds.batch(64).prefetch(AUTOTUNE)
```

### B) Keras `Sequence`
```python
class MySeq(keras.utils.Sequence):
    def __init__(self, paths, labels, batch_size=64, shuffle=True):
        self.paths = np.array(paths)
        self.labels = np.array(labels, dtype=np.int32)
        self.batch_size = batch_size
        self.shuffle = shuffle
        self.indices = np.arange(len(self.paths))
        self.on_epoch_end()

    def __len__(self):
        return math.ceil(len(self.paths) / self.batch_size)

    def on_epoch_end(self):
        if self.shuffle:
            np.random.shuffle(self.indices)

    def __getitem__(self, idx):
        batch_ids = self.indices[idx*self.batch_size:(idx+1)*self.batch_size]
        batch_paths = self.paths[batch_ids]
        batch_labels = self.labels[batch_ids]

        imgs = []
        for p in batch_paths:
            img_bytes = tf.io.read_file(p)
            img = tf.image.decode_jpeg(img_bytes, channels=3)
            img = tf.image.resize(img, (224, 224))
            img = tf.cast(img, tf.float32) / 255.0
            imgs.append(img)

        x = tf.stack(imgs, axis=0)
        y = tf.convert_to_tensor(batch_labels)
        return x, y
```

## 주의할 점(Caveats)
- `Sequence`는 `workers`/`use_multiprocessing` 튜닝에 따라 성능/안정성이 달라질 수 있다(환경 의존).
- `tf.data`에서 “가변 길이 라벨”(예: bbox 개수)이면 `padded_batch`/`RaggedTensor`로 처리해야 해서 코드가 길어질 수 있다.
- 용어 혼동: 강의에서 “dataset 구현”은 보통 **개념적 dataset(데이터 공급 모듈)**을 뜻하고, `tf.data.Dataset` 클래스를 구현한다는 뜻은 아닐 수 있다.

## 한 줄 요약(Summary)
`tf.data`는 “파이프라인이 배치를 만든다”, `Sequence`는 “파이썬 클래스가 배치를 만든다” — 배치 생성 위치가 코드 구조를 갈라놓는다.

---

## 역사/배경(Timeline/Why)
- TF1의 입력(Queues)이 복잡하던 시절, 커뮤니티는 생산성이 좋은 Keras 제너레이터/`Sequence`를 많이 썼다.
- `tf.data`가 표준 입력 추상화로 자리잡고 분산/성능과 결합되면서, “기본값은 `tf.data`”로 수렴했다.
- 그럼에도 “파이썬 로직이 핵심”인 데이터 조립 문제는 여전히 `Sequence`가 실용적이다.

## 참고(Links)
- https://www.tensorflow.org/guide/data
- https://keras.io/api/data_loading/
