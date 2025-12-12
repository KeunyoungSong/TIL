# Kaggle Python 3.11에서 scikit-learn 1.1.1 설치 실패 트러블슈팅: wheel 부재와 해결

## 상황(Context)
Kaggle 노트북에서 예전 튜토리얼을 따라가며 `scikit-learn==1.1.1`을 설치하려고 했다. 런타임 Python 버전은 3.11.13이었다.

## 문제(Problem)
`pip install scikit-learn==1.1.1` 실행 시 메타데이터 생성 단계에서 실패했다.
주요 로그는 아래 형태였다.
- `Preparing metadata (pyproject.toml) did not run successfully`
- `metadata-generation-failed`

그래서 최신 버전(`scikit-learn==1.3.2`)으로 올린 뒤 코드를 실행했는데, 이번에는 다음 ImportError가 발생했다.
```
from sklearn.datasets import load_boston
ImportError: load_boston has been removed from scikit-learn since version 1.2
```

## 원인(Cause)
1) **설치 실패 원인**  
Python 3.11 환경에서 `scikit-learn 1.1.1`에 맞는 wheel(.whl)이 제공되지 않아, pip가 소스 배포(sdist, `.tar.gz`)를 내려받아 로컬 빌드를 시도했다. Kaggle 환경에서 이 빌드 단계가 실패하며 메타데이터 생성 에러가 났다.

2) **`load_boston` 에러 원인**  
`load_boston`은 scikit-learn 1.2부터 제거되었다. Boston housing 데이터셋이 인종 관련 편향/윤리적 문제를 포함하고 있어 유지보수 팀이 사용을 비권장하며 API를 삭제했다.

## 해결(Solution)
1) **Python 3.11에 맞는 scikit-learn 설치**
```bash
pip install -U scikit-learn
# 또는 명시적으로
pip install scikit-learn==1.3.2
```

2) **대체 데이터셋으로 코드 변경**
- California housing 사용:
```python
from sklearn.datasets import fetch_california_housing

housing = fetch_california_housing(as_frame=True)
X = housing.data
y = housing.target
```

- Ames housing(OpenML) 사용:
```python
from sklearn.datasets import fetch_openml

housing = fetch_openml(name="house_prices", as_frame=True)
X = housing.data
y = housing.target
```

3) **Python 버전 확인(재현/점검용)**
```python
import sys
print(sys.version)
```

## 배운 점(Takeaways)
- 패키지 버전을 고정(`==`)할 때는 **현재 Python 버전과의 호환성**(특히 wheel 제공 여부)을 먼저 확인해야 한다.
- 최신 런타임(Kaggle, Colab 등)에서는 구버전 패키지가 소스 빌드를 타면서 쉽게 깨질 수 있으니, 가능하면 **호환되는 최신 버전으로 맞추는 게 현실적**이다.
- 튜토리얼이 오래되면 API가 제거/변경되어 있을 수 있다. 에러 메시지와 공식 문서를 보고 **대체 API/데이터셋으로 전환**하는 습관이 필요하다.
- “왜 안 되는지(원인)”를 먼저 적고 “어떻게 고쳤는지(해결)”를 기록하면, 나중에 비슷한 이슈가 왔을 때 바로 재사용 가능하다.

## 참고(Links)
- scikit-learn `load_boston` 제거 안내(에러 메시지 내 링크)
- Kaggle Python 런타임: Python 3.11 기반

## 용어(Glossary)
- **wheel**: `.whl` 확장자의 미리 빌드된 패키지. 설치 시 컴파일 없이 바로 설치된다.
- **sdist**: `.tar.gz` 같은 소스 배포 파일. 설치 시 로컬에서 빌드/컴파일이 필요해 환경에 따라 실패할 수 있다.
