Series: 1차원 구조체

DataFrame: 2차원 표 형태의 구조체

데이터 탐색 속성(shape, dtypes)과 진단(info, describe)
- `shape()`: DataFrame의 크기
- `info()`: DataFrame의 개요를 간단학 표시한다.
	-  누락치 확인 (Non-Null Count)
    - `dtype`: 통계가 가능한 자료형인지 확인 (잘못된 데이터 여부 확인 -> 치환 -> 자료형 변환)
- `describe()`: 숫자로된 열에 대한 기초 통계량을 표시한다.
	- 평균, 중앙값, 최대, 최소값을 통해서 이상치(극단치)를 확인

인덱스 제어
- set_index: 컬럼을 인덱스로 설정한다.
- reset_index: 설정된 인덱스를 초기화 한다.

브로드캐스팅 연산
- 기본 연산자(+, 등 산술 연산자)
    - DataFrame, Series 연산 시 열 이름 기준으로 브로드캐스팅 연산 수행
- 산술 메서드(`.sub()`, `.div()` 등)
    - axis=0(행 인덱스 기준) 또는 axis=1(열 기준)을 명시하여 정밀하게 브로드캐스팅한다.

 loc와 iloc를 활용한 행/열 선택
- `loc[행_라벨, 열_라벨]`: 명칭을 기준으로 선택
- `iloc[행_위치, 열_위치]`: 0부터 시작하는 정수 오프셋 위치로 선택

불리언 인덱싱과 `query()` 조건 검색
- 조건식: `&`, `|`
- query(): 문자열 조건식(`and`, `or`) 및 `@`표시로 외부 변수 참조 활용

```python
import pandas as pd

df_sample = pd.DataFrame({
    '나이': [25, 30, 25, 40],
    '점수': [80, 95, 70, 85],
    '등급': ['B', 'A', 'C', 'B'],
}, index=['user1', 'user2', 'user3', 'user4'])

target_score = 80

filtered_bool = df_sample[(df_sample['나이'] >= 30) & (df_sample['점수'] >= target_score)]
print(filtered_bool)

filtered_query = df_sample.query('나이 >= 30 and 점수 >= @target_score')
print(filtered_query)
```

```
나이  점수 등급
user2  30  95  A
user4  40  85  B
       나이  점수 등급
user2  30  95  A
user4  40  85  B
```

결측치(Missing Data) 7대 처리 기법
- 실무에서 발생하는 다양한 결측 데이터(`NaN`, `<NA>`, `None`, `이상 기호`)를 유형별 최적의 방법으로 정제