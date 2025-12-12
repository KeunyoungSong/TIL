# scikit-learn 설치 에러와 load_boston 제거

## 배운 것
Python 3.11 환경에서 `scikit-learn==1.1.1` 설치가 실패한 이유와, 최신 `scikit-learn`에서 `load_boston`이 제거된 배경 및 대체 방법을 정리했다.

## 왜 중요한가
- Kaggle 같은 최신 Python 런타임에서는 구버전 패키지가 잘 안 깔릴 수 있다.
- 라이브러리 버전 업으로 API가 제거/변경되는 경우가 있으니, 에러 메시지를 읽고 대체 경로를 찾는 습관이 필요하다.

## 어떻게 쓰는가
### 1) `scikit-learn==1.1.1` 설치 실패 원인
- Python 3.11에는 `scikit-learn 1.1.1`용 **wheel(.whl)**이 없다.
- 그래서 pip가 `.tar.gz` **sdist(소스 배포)**를 받아 로컬에서 빌드하려고 하는데, Kaggle/로컬 환경에서 빌드 단계가 실패하면 아래처럼 난다.
  - `Preparing metadata (pyproject.toml) did not run successfully`
  - `metadata-generation-failed`

해결:
- Python 3.11을 지원하는 버전으로 올리기(권장):
  ```bash
  pip install -U scikit-learn
  # 예: pip install scikit-learn==1.3.2
  ```
- (정말 1.1.1이 필요하면) Python을 3.10 이하로 낮춰 휠이 있는 조합을 사용해야 한다.  
  Kaggle에서는 런타임 버전 변경이 어려워 비현실적.

### 2) `load_boston` ImportError
`scikit-learn 1.2`부터 `load_boston`이 완전히 제거됐다.
- Boston housing 데이터셋이 인종 관련 편향/윤리적 문제를 포함하고 있어서 유지보수 팀이 사용을 비권장하며 API도 제거.

해결(대체 데이터셋):
- California housing:
  ```python
  from sklearn.datasets import fetch_california_housing
  housing = fetch_california_housing(as_frame=True)
  X = housing.data
  y = housing.target
  ```
- Ames housing(OpenML):
  ```python
  from sklearn.datasets import fetch_openml
  housing = fetch_openml(name="house_prices", as_frame=True)
  X = housing.data
  y = housing.target
  ```

### 3) wheel이 뭐였지?
wheel은 `.whl` 확장자의 **미리 빌드된 패키지**다.
- wheel이 있으면: 다운로드 후 바로 설치(빠름, 에러 적음)
- wheel이 없으면: 소스 빌드가 필요(느림, 컴파일 환경에 따라 실패 가능)

## 예시
Kaggle에서 Python 버전 확인:
```python
import sys
print(sys.version)
```

## 주의할 점
- 패키지 버전을 고정(`==`)할 때는 현재 Python 버전과 호환되는지 먼저 확인.
- 튜토리얼 코드가 오래된 경우, 제거된 API(`load_boston` 등)가 나올 수 있으니 공식 문서/에러 메시지의 대안을 따른다.

## 참고
- scikit-learn `load_boston` 제거 안내(에러 메시지 내 링크)
- Kaggle Python 런타임: Python 3.11 기반
