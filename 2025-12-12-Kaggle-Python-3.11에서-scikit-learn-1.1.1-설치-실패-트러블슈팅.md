# Kaggle Python 3.11에서 scikit-learn 1.1.1 설치 실패 트러블슈팅

## 상황(Context)
Kaggle 노트북에서 예전 튜토리얼을 따라가며 `scikit-learn==1.1.1`을 설치하려고 했다. 런타임 Python 버전은 3.11.13이었다.

## 문제(Problem)
1) `pip install scikit-learn==1.1.1` 실행 시 메타데이터 생성 단계에서 실패했다.  
   주요 로그는 아래 형태였다.
   - `Preparing metadata (pyproject.toml) did not run successfully`
   - `metadata-generation-failed`

2) 설치를 해결하려고 최신 버전(`scikit-learn==1.3.2`)으로 올린 뒤 강의 코드를 실행했는데, `load_boston` ImportError가 발생했다.
   ```
   from sklearn.datasets import load_boston
   ImportError: load_boston has been removed from scikit-learn since version 1.2
   ```

## 원인(Cause)
1) **설치 실패 원인**  
Python 3.11 환경에서 `scikit-learn 1.1.1`에 맞는 wheel(.whl)이 제공되지 않는다. 그래서 pip가 소스 배포(sdist, `.tar.gz`)를 내려받아 로컬 빌드를 시도했고, Kaggle 환경에서 빌드 단계가 실패하며 메타데이터 생성 에러가 났다.

2) **`load_boston` 에러 원인**  
`load_boston`은 scikit-learn 1.2부터 제거되었다. Boston housing 데이터셋이 인종 관련 편향/윤리적 문제를 포함하고 있어 유지보수 팀이 사용을 비권장하며 API를 삭제했다.

## 해결(Solution)
1) **Python 3.11에 맞는 scikit-learn 설치(문제 1 해결)**
```bash
pip install -U scikit-learn
# 또는 명시적으로
pip install scikit-learn==1.3.2
```
`-U`는 기존 버전 대신 Python 3.11용 wheel이 있는 버전으로 업그레이드해 소스 빌드를 피하게 해준다.

2) **Boston 데이터 그대로 쓰기: 원본에서 직접 로드(문제 2 대안)**
강의 코드가 기대하는 Boston 데이터와 동일한 값을 원본 사이트에서 읽어온다.
```python
import pandas as pd
import numpy as np

data_url = "http://lib.stat.cmu.edu/datasets/boston"
raw_df = pd.read_csv(data_url, sep=r"\s+", skiprows=22, header=None)

X = np.hstack([raw_df.values[::2, :], raw_df.values[1::2, :2]])
y = raw_df.values[1::2, 2]

feature_names = [
    "CRIM","ZN","INDUS","CHAS","NOX","RM","AGE","DIS",
    "RAD","TAX","PTRATIO","B","LSTAT"
]

bostonDF = pd.DataFrame(X, columns=feature_names)
bostonDF["PRICE"] = y
```

3) **Python 버전 확인(재현/점검용)**
```python
import sys
print(sys.version)
```

## 결과/현재 상태(Result/Status)
- 문제 1) `scikit-learn==1.1.1` 설치 실패 → Python 3.11 지원 버전으로 업그레이드해 해결.
- 문제 2) `load_boston` ImportError → 함수는 제거되어 복구 불가. 대신 원본 Boston 데이터를 직접 로드해 강의 흐름을 유지.

## 배운 점(Takeaways)
- 패키지 버전을 고정(`==`)할 때는 **현재 Python 버전과의 호환성**(특히 wheel 제공 여부)을 먼저 확인해야 한다.
- 최신 런타임(Kaggle, Colab 등)에서는 구버전 패키지가 소스 빌드를 타면서 쉽게 깨질 수 있으니, 가능하면 **호환되는 최신 버전으로 맞추는 게 현실적**이다.
- 튜토리얼이 오래되면 API가 제거/변경되어 있을 수 있다. 이때는 “데이터/목표는 유지하면서 API만 바꾸는” 최소 우회가 가능한지 먼저 확인한다.
- 우회/대안을 채택했다면 왜 그 방법이 필요한지(제약/승인/트레이드오프)를 함께 기록하면 다음에 판단이 빨라진다.

## 참고(Links)
- scikit-learn `load_boston` 제거 안내(에러 메시지 내 링크)
- Kaggle Python 런타임: Python 3.11 기반

## 용어(Glossary)
- **wheel**: `.whl` 확장자의 미리 빌드된 패키지. 설치 시 컴파일 없이 바로 설치된다.
- **sdist**: `.tar.gz` 같은 소스 배포 파일. 설치 시 로컬에서 빌드/컴파일이 필요해 환경에 따라 실패할 수 있다.
