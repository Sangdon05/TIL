#machine-learning

한글: 희소 범주형 교차 엔트로피 (Sparse Categorical Cross-Entropy, SCCE)

## 1. 의미 및 등장 배경
- **한 줄 정의**: 다중 클래스 분류(Multi-Class Classification) 문제에서 정답 레이블을 원-핫 인코딩(One-Hot Encoding) 없이 **정수형 정답 인덱스(Integer Index)** 그대로 사용하여 손실을 계산하는 손실 함수(Loss Function).
- **등장 배경 (기존 CCE의 메모리/연산 비효율 개선)**:
  - **일반 CCE(Categorical Cross-Entropy)**는 정답을 `[0, 0, 1, 0, ...]`과 같은 원-핫 벡터로 표현해야 한다.
  - 클래스 수가 수백~수십만 개(예: 거대 언어 모델의 어휘집, 대규모 상품 분류)에 달할 경우, 원-핫 벡터 변환으로 인해 **막대한 메모리 낭비(Memory Overhead)**와 대부분이 0인 불필요한 행렬 곱셈 연산이 발생한다.
  - **SCCE**는 정답을 단일 정수(예: `2`)로 전달받아 **정답 인덱스의 예측 확률만 즉시 참조(Direct Indexing / Lookup)**함으로써 메모리와 연산 속도를 대폭 최적화한다.

---

## 2. 수식 및 동작 원리

### (1) 일반 CCE vs SCCE 수식 비교

- **일반 CCE (One-Hot Target $y$)**:
  $$
  L_{\text{CCE}} = -\sum_{c=1}^{C} y_c \log(\hat{y}_c)
  $$
  - $C$: 전체 클래스 개수
  - $y_c$: 클래스 $c$의 원-핫 정답값 ($0$ 또는 $1$)
  - $\hat{y}_c$: 모델이 예측한 클래스 $c$의 확률 (Softmax 출력값)

- **SCCE (정수 인덱스 정답 $y_{\text{true}} \in \{0, 1, \dots, C-1\}$)**:
  $$
  L_{\text{SCCE}} = -\log(\hat{y}_{y_{\text{true}}})
  $$
  - $y_{\text{true}}$: 실제 정답 클래스의 인덱스 번호
  - $\hat{y}_{y_{\text{true}}}$: 모델의 출력 확률 벡터 $\hat{\mathbf{y}}$에서 정답 인덱스 위치의 예측 확률

### (2) 직관적 메커니즘 해석
- 원-핫 벡터 $y$에서 실제 정답 클래스($y_{\text{true}}$)를 제외한 나머지 원소는 모두 $0$이다.
- 따라서 $\sum$ 합산 연산에서 $0 \times \log(\hat{y}_c)$는 모두 0으로 소거되고, 오직 정답 인덱스에 해당하는 $-\log(\hat{y}_{y_{\text{true}}})$만 남는다.
- **SCCE는 수학적으로 일반 CCE와 완벽히 동일한 손실 및 기울기(Gradient)**를 도출하지만, 중간 과정에서 원-핫 벡터를 메모리에 생성하지 않고 정답 위치의 값만 바로 추출하여 계산한다.

---

## 3. 실전 사용 예시

### (1) TensorFlow / Keras 예시
```python
import tensorflow as tf
from tensorflow.keras import layers, models

# 모델 구성
model = models.Sequential([
    layers.Dense(64, activation='relu', input_shape=(20,)),
    layers.Dense(10, activation='softmax')  # 10개 클래스 출력
])

# 정수형 레이블을 그대로 사용하므로 sparse_categorical_crossentropy 선택
model.compile(
    optimizer='adam',
    loss=tf.keras.losses.SparseCategoricalCrossentropy(),
    metrics=['accuracy']
)

# 정답 레이블 y: 원-핫이 아닌 정수 배열 [batch_size, 1] 또는 [batch_size]
# 예: y_train = np.array([3, 0, 9, 2, 5, ...])
# model.fit(x_train, y_train, epochs=10)
```

> **Tip (from_logits 옵션)**:
> 모델의 마지막 레이어에 `softmax`를 두지 않고 선형 출력(Logits)을 그대로 낼 경우, 수치적 안정성(Numerical Stability)을 위해 `SparseCategoricalCrossentropy(from_logits=True)`로 설정하는 것이 권장된다.

### (2) PyTorch 예시
PyTorch의 `nn.CrossEntropyLoss`는 기본적으로 **내부에서 LogSoftmax + SCCE(Negative Log Likelihood with integer target)** 형태로 구현되어 있어 타깃으로 정수 인덱스 텐서를 전달한다.

```python
import torch
import torch.nn as nn

# [batch_size=3, num_classes=5] 예측값 (Logits)
logits = torch.randn(3, 5, requires_grad=True)

# 정수형 정답 인덱스 (Shape: [batch_size])
targets = torch.tensor([1, 0, 4]) 

criterion = nn.CrossEntropyLoss()
loss = criterion(logits, targets)

loss.backward()
```

---

## 4. 장단점 및 손실 함수 비교

### 장점
1. **메모리 절약**: 클래스 수($C$)가 클수록 원-핫 인코딩 대비 $\mathcal{O}(B \times C)$의 메모리 할당을 $\mathcal{O}(B)$로 줄여 GPU 메모리 사용량을 크게 절약한다.
2. **연산 속도 향상**: 불필요한 $0$ 곱셈 및 텐서 변환 과정이 생략된다.
3. **코드 간결성**: `to_categorical`이나 `one_hot` 전처리 과정 없이 원본 정수 라벨을 그대로 파이프라인에 사용 가능하다.

### 한계점 및 주의사항
- **다중 라벨 분류(Multi-Label) 불가**: 한 데이터 샘플이 여러 개의 정답 클래스를 가질 수 있는 문제에서는 사용할 수 없으며, 각 클래스별로 `Binary Cross-Entropy(BCE)`를 사용해야 한다.
- **소프트 라벨(Soft Label) 미지원**: Label Smoothing, Mixup, 지식 증류(Knowledge Distillation) 등 정답이 확률 분포 형태($[0.1, 0.8, 0.1]$ 등)인 경우에는 정수 인덱스 방식(SCCE)을 쓸 수 없고 일반 CCE를 사용해야 한다.

### 손실 함수 비교 요약
- **BCE (Binary Cross-Entropy)**: 이진 분류(0 or 1) 또는 다중 라벨(Multi-label) 분류에 사용.
- **CCE (Categorical Cross-Entropy)**: 다중 클래스 단일 정답 분류에 사용 (타깃: **원-핫 벡터** 또는 **확률 분포**).
- **SCCE (Sparse Categorical Cross-Entropy)**: 다중 클래스 단일 정답 분류에 사용 (타깃: **정수 인덱스**).
