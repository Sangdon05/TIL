#math #machine-learning

한글: 미분 (Differentiation / Derivative)

## 1. 의미 및 개념 (Meaning & Background)
- **한 줄 정의**: 어떤 함수에서 입력값($x$)이 **아주 미세하게 변화할 때, 출력값($y$)이 얼마나 민감하게 변하는지(순간 변화율)**를 구하는 수학적 연산.
- **기하학적 의미**: 함수 그래프 위의 특정 점에서의 **접선의 기울기(Tangent Slope)**.
  - 기울기가 **양수(+)**: $x$가 커질 때 $y$도 증가함 (오르막길)
  - 기울기가 **음수(-)**: $x$가 커질 때 $y$는 감소함 (내리막길)
  - 기울기가 **0**: 평탄한 지점 (극댓값, 극솟값 또는 안장점)
- **머신러닝/딥러닝에서의 핵심 역할**:
  - 모델 학습의 궁극적인 목표는 손실 함수(Loss Function, 오차 $J$)를 최소화하는 것이다.
  - "가중치($w$)를 어느 방향으로, 얼마나 수정해야 오차가 줄어들까?"라는 질문의 나침반 역할을 미분(기울기, Gradient)이 수행한다.

---

## 2. 기본 수식 및 동작 원리 (Mechanism & Breakdown)

### (1) 미분의 수학적 정의 (도함수 극한 공식)
$$
f'(x) = \frac{df}{dx} = \lim_{\Delta x \to 0} \frac{f(x + \Delta x) - f(x)}{\Delta x}
$$
- $\Delta x$ (델타 x): $x$의 미세한 변화량. 이 변화량을 0에 무한히 가깝게 보냄으로써 '평균 변화율'을 '순간 변화율'로 전환한다.
- $\frac{df}{dx}$ (라이프니츠 표기법): $x$의 미소 변화량($dx$)에 대한 $f$의 미소 변화량($df$)의 비율.

### (2) 자주 쓰이는 핵심 미분 공식
1. **거듭제곱 공식 (Power Rule)**:
   $$ \frac{d}{dx}(x^n) = n x^{n-1} $$
   - 예: $\frac{d}{dx}(x^2) = 2x$, $\frac{d}{dx}(x^3) = 3x^2$
2. **상수 미분**:
   $$ \frac{d}{dx}(c) = 0 $$
   - 상수는 변하지 않으므로 변화율이 0이다.
3. **지수/로그 함수 미분**:
   $$ \frac{d}{dx}(e^x) = e^x, \quad \frac{d}{dx}(\ln x) = \frac{1}{x} $$
   - 머신러닝의 활성화 함수(Sigmoid, Softmax)나 교차 엔트로피(Cross-Entropy) 손실 계산의 기본이 된다.
4. **선형성 (합/상수배)**:
   $$ \frac{d}{dx}(a f(x) + b g(x)) = a f'(x) + b g'(x) $$

---

## 3. 머신러닝/딥러닝 필수 확장 개념

### (1) 편미분 (Partial Derivative, $\partial$)
- **개념**: 다변수 함수에서 **관심 있는 단 하나의 변수만 변화시키고, 나머지 변수는 모두 고정된 상수(숫자)로 취급**하여 미분하는 기법.
- **기호**: 라운드 디($\partial$) 사용.
- **수식 예시**: $f(w, b) = w^2 + 3wb + b^2$ 일 때
  - $w$에 대한 편미분: $\frac{\partial f}{\partial w} = 2w + 3b$ ($b$는 상수 취급)
  - $b$에 대한 편미분: $\frac{\partial f}{\partial b} = 3w + 2b$ ($w$는 상수 취급)
- **ML 적용**: 신경망의 수천만 개 가중치($w_1, w_2, \dots$)와 편향($b$) 각각에 대해 개별적인 오차 기여도를 계산할 때 사용.

### (2) 그래디언트 (기울기 벡터, $\nabla$)
- **개념**: 모든 변수에 대한 편미분 결과를 하나의 벡터로 묶은 것.
$$
\nabla f(w, b) = \begin{bmatrix} \frac{\partial f}{\partial w} \\ \frac{\partial f}{\partial b} \end{bmatrix}
$$
- **의미**: 함수가 **가장 가파르게 증가하는 방향과 크기**를 나타낸다. 따라서 오차를 줄이려면 그래디언트의 **반대 방향($-\nabla f$)**으로 이동해야 한다.

### (3) 연쇄 법칙 (Chain Rule)
- **개념**: 여러 함수가 중첩된 합성 함수(Composite Function)를 미분할 때, 바깥쪽부터 안쪽으로 각 단계의 미분을 차례대로 곱하는 규칙.
- **수식**: $y = f(u)$이고 $u = g(x)$일 때,
  $$ \frac{dy}{dx} = \frac{dy}{du} \cdot \frac{du}{dx} $$
- **ML 적용**: 딥러닝 **역전파(Backpropagation)**의 핵심 원리. 출력층의 오차를 입력층 방향으로 거슬러 올라가며 각 층의 가중치 미분값을 효율적으로 계산한다.

---

## 4. 실전 활용 예시 및 코드 (Use Cases & Examples)

### (1) 경사 하강법(Gradient Descent) 가중치 업데이트 원리
손실 함수 $J(w)$의 최솟값을 찾기 위해 기울기의 반대 방향으로 가중치를 갱신한다:
$$
w_{new} = w_{old} - \alpha \frac{\partial J}{\partial w}
$$
- $\alpha$: 학습률 (Learning Rate)
- **해석**:
  - 기울기($\frac{\partial J}{\partial w}$)가 **양수(+)** $\rightarrow$ $w$가 커지면 오차도 커짐 $\rightarrow$ $w$를 **감소(-)**시켜야 함.
  - 기울기($\frac{\partial J}{\partial w}$)가 **음수(-)** $\rightarrow$ $w$가 커지면 오차가 작아짐 $\rightarrow$ $w$를 **증가(+)**시켜야 함.

### (2) PyTorch 자동 미분(Autograd) 코드 예제
```python
import torch

# 1. 미분을 추적할 텐서 선언 (requires_grad=True)
w = torch.tensor(2.0, requires_grad=True)
b = torch.tensor(1.0, requires_grad=True)

# 2. 순전파(Forward Pass): y = 3w^2 + 2b
y = 3 * (w ** 2) + 2 * b

# 3. 역전파(Backward Pass): dy/dw, dy/db 자동 계산
y.backward()

# 4. 미분값 확인
# dy/dw = 6w = 6 * 2.0 = 12.0
# dy/db = 2.0
print(f"dy/dw: {w.grad.item()}")  # 출력: 12.0
print(f"dy/db: {b.grad.item()}")  # 출력: 2.0
```

---

## 5. 핵심 비교 및 요약 (Summary & Comparison)

| 구분 | 수학적 정의 | 머신러닝/딥러닝에서의 실전 역할 |
| :--- | :--- | :--- |
| **미분 (Derivative)** | 한 변수 함수의 순간 변화율 ($\frac{df}{dx}$) | 파라미터 1개의 변화가 전체 함수에 미치는 영향 |
| **편미분 (Partial)** | 다변수 함수에서 특정 변수 1개만의 변화율 ($\frac{\partial f}{\partial w}$) | 수많은 가중치 중 특정 가중치 1개의 오차 기여도 |
| **그래디언트 ($\nabla$)** | 모든 편미분 값들을 모아둔 벡터 | 오차가 가장 가파르게 증가하는 방향 (학습은 반대 방향으로 진행) |
| **연쇄 법칙 (Chain Rule)** | 합성 함수의 미분 곱 ($\frac{dy}{dx} = \frac{dy}{du} \cdot \frac{du}{dx}$) | 다층 신경망 전체의 가중치를 역방향으로 갱신하는 역전파 알고리즘 |
