# Object Detection Datasets: Pascal VOC / MS COCO / Open Images 어노테이션 구조 정리

## 0) 한 문장 상황/목표
`Pascal VOC`, `MS COCO`, `Google Open Images`의 어노테이션 “파일 단위 구조”와 “클래스 개수(20/80/600)가 의미하는 바”를 헷갈리지 않게 고정한다.

---

## 1) 목록(Outline) = 문서 설계도
- `Pascal VOC`: 왜 “1 image = 1 XML”이라고 말하는가?
- `MS COCO`: “JSON 하나”는 무슨 뜻이고, 무엇이 오해 포인트인가?
- `Open Images`: “CSV format”이라고만 말하면 왜 부족한가?
- “20/80/600 classes”는 정확히 무엇의 개수인가?
- 세 데이터셋을 한 줄로 요약하면?

---

## 2) 재구성(Reframe) + 본문(Write)

## 2.1) `Pascal VOC`: 왜 “1 image = 1 XML”이라고 말하는가?
결론(1–3문장):
`Pascal VOC`는 이미지 파일 하나당, 그 이미지에 대응하는 `XML` 어노테이션 파일 하나가 붙는 구조다. 그래서 “1 image = 1 XML 메타데이터”라고 말한다.

핵심(3–6줄):
- `images/xxx.jpg`에 대해 `annotations/xxx.xml`이 1:1로 대응한다.
- `XML` 안에는 보통 이미지 크기 정보와 객체 리스트가 들어간다.
- 객체마다 `class`와 `bounding box`가 반복 구조로 기록된다.

예시(필요하면):
```
image.jpg
image.xml  (size + objects[*]{class, bbox})
```

주의/오해(있으면):
“한 이미지에 객체가 하나만 있다”는 뜻이 아니다. “메타데이터 파일이 이미지별로 나뉜다”는 뜻이다.

## 2.2) `MS COCO`: “JSON 하나”는 무슨 뜻이고, 무엇이 오해 포인트인가?
결론(1–3문장):
`COCO`는 보통 split(예: `train2017`)마다 `instances_train2017.json` 같은 **단일 JSON 파일 1개**에 “전체 이미지/어노테이션”이 함께 들어간다. 다만 “이미지 전체에 대해 메타데이터가 하나”라고 말하면 내부 구조를 놓쳐 오해가 생긴다.

핵심(3–6줄):
- 파일은 하나지만, JSON 내부는 `images[]`, `annotations[]`, `categories[]`처럼 **구조적으로 분리**되어 있다.
- `annotation`은 `image_id`로 어떤 이미지에 속하는지 연결된다.
- (세그멘테이션을 쓰는 경우) `segmentation`도 같은 JSON 내부에 함께 들어간다.

예시(필요하면):
```
images/
  000000000001.jpg
  000000000002.jpg
annotations/
  instances_train2017.json  (images[], annotations[], categories[])
```

주의/오해(있으면):
“메타데이터가 하나”라는 말은 “파일이 하나”라는 뜻으로만 쓰는 게 안전하다. “이미지별 정보가 뭉개져 있다”는 뜻이 아니다.

## 2.3) `Open Images`: “CSV format”이라고만 말하면 왜 부족한가?
결론(1–3문장):
`Open Images`는 어노테이션이 `CSV` 중심인 건 맞지만, 실제로는 역할별 파일이 여러 개로 나뉘어 “정규화된 테이블”처럼 구성된다(그리고 일부는 `JSON`도 섞인다).

핵심(3–6줄):
- 이미지 목록, bbox 어노테이션, 클래스 설명(이름 매핑), 클래스 계층(트리/그래프)이 파일로 분리된다.
- bbox는 보통 별도 `annotations-bbox.csv` 같은 형태로 제공된다.
- 클래스 이름/ID 매핑도 별도 `class-descriptions.csv`로 제공된다.
- 클래스 계층은 `hierarchy.json` 같은 형태로 제공될 수 있다.

예시(필요하면):
```
images.csv
annotations-bbox.csv
class-descriptions.csv
hierarchy.json
```

주의/오해(있으면):
“CSV 하나에 다 있다”가 아니라 “CSV 여러 개(테이블들)로 쪼개져 있다”가 핵심이다.

## 2.4) “20/80/600 classes”는 정확히 무엇의 개수인가?
결론(1–3문장):
`20 / 80 / 600`은 “한 이미지에 들어 있는 객체 개수”가 아니라, **데이터셋이 정의한 클래스 사전(category set)의 크기**다. 즉 “데이터셋 전체에 걸쳐 등장할 수 있도록 정의된 객체 타입 수”다.

핵심(3–6줄):
- `VOC`: 20 classes
- `COCO`: 80 classes
- `Open Images`: (대표적으로) 600 classes
- 한 이미지에 600개가 다 찍힌다는 뜻이 아니라, 데이터셋 전체 분포로 600종류가 존재한다는 뜻이다.

예시(필요하면):
`Open Images`의 “600”은 `Person`, `Car`, `Apple`, `Traffic light` 같은 타입이 600개 정의돼 있다는 의미다.

주의/오해(있으면):
클래스가 많아질수록 “클래스 계층/동의어/라벨 품질” 같은 문제가 같이 생기기 쉬워서, 구조(`hierarchy.json` 등)가 함께 중요해진다.

## 2.5) 세 데이터셋을 한 줄로 요약하면?
결론(1–3문장):
세 데이터셋의 차이는 “어노테이션을 파일로 쪼개는 단위”가 다르다는 점이다.

핵심(3–6줄):
- `Pascal VOC`: **이미지마다 XML 하나**
- `MS COCO`: **split마다 전체를 담은 JSON 하나**
- `Open Images`: **이미지/박스/클래스/계층을 분리한 CSV(+JSON) 여러 개**

예시(필요하면):
| Dataset | Annotation structure |
| --- | --- |
| Pascal VOC | 1 image : 1 XML |
| MS COCO | 1 split : 1 JSON (structured) |
| Open Images | normalized tables (multiple CSV + JSON) |

주의/오해(있으면):
“format(XML/JSON/CSV)”만 외우면 구조를 놓친다. 중요한 건 “무엇을 어떤 단위로 분리했는지”다.

---

## 3) 마지막 한 문장(재독용)
`VOC`는 이미지별 `XML`, `COCO`는 split별 구조화된 단일 `JSON`, `Open Images`는 역할별 `CSV(+JSON)` 테이블 분리이며, `20/80/600`은 “클래스 사전 크기”다.
