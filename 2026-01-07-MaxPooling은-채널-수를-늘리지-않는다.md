# MaxPooling은 채널 수를 늘리지 않는다

> Type: Concept

## 상황/배경(Context)
CNN을 볼 때 “Pooling을 지나면 채널이 늘어나는 것 같다”는 착각을 자주 한다. shape 관점으로 한 번만 고정해두면 이후 레이어를 읽을 때 덜 헷갈린다.

---

## 정의(Definition)
`MaxPooling2D`/`AveragePooling2D`는 **공간 해상도(H, W)를 줄이고**, 채널 수(C)는 **그대로 유지**하는 연산이다.

## 핵심 아이디어(Key Ideas)
- 채널 수(C)를 바꾸는 건 보통 `Conv2D(filters=...)` 같은 convolution 계열이다.
- pooling은 채널 간 정보를 섞지 않고, **채널별로 독립적으로** max/avg를 취한다.
- 그래서 “Pooling 뒤에 채널이 늘었다”면, 대부분 **Pooling 다음 Conv**를 같이 보고 착각한 것이다.

## 예시(Examples)
shape로 보면 바로 정리된다.

### Conv2D
입력: `(H, W, C)`  
`Conv2D(filters=F)` 출력: `(H', W', F)` (채널이 `F`로 바뀜)

### MaxPooling2D(stride=2)
입력: `(H, W, C)`  
출력: `(H/2, W/2, C)` (채널 유지)

예:
- 입력: `(224, 224, 64)`
- `MaxPooling2D(pool_size=2, strides=2)` 출력: `(112, 112, 64)`

## 주의할 점(Caveats)
- `GlobalAveragePooling2D`/`GlobalMaxPooling2D`는 공간 축을 “완전히 제거”해서 `(H, W, C) → (C)`가 된다.
  - 채널을 늘리진 않지만, shape가 급격히 바뀌어서 착각을 유발할 수 있다.

## 한 줄 요약(Summary)
Pooling은 공간(H, W)을 줄이는 압축이고, 채널(C)을 늘리는 건 convolution의 역할이다.

---

## 도식(Diagram)
```
Conv2D:      (H, W, C) -> (H', W', F)
MaxPooling:  (H, W, C) -> (H/2, W/2, C)
```
