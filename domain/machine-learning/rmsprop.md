#machine-learning

한글: RMSProp (알엠에스프롭, Root Mean Square Propagation)

## 1. 의미 및 등장 배경
- **한 줄 정의**: 각 가중치(파라미터)마다 기울기(Gradient)의 크기에 맞춰 **학습률(Learning Rate)을 개별적으로 적응(조절)**시키는 경사 하강법 기반 옵티마이저(Optimizer).
- **등장 배경 (AdaGrad의 조기 중단 한계 극복)**:
  - **AdaGrad**는 과거의 모든 기울기 제곱을 단순 합산($G_t = G_{t-1} + g_t^2$)하므로, 학습이 진행될수록 분모가 계속 커져 **학습률이 0에 수렴해 조기에 학습이 멈추는 현상(Learning Rate Vanishing)**이 발생했다.
  - **RMSProp**은 이를 해결하기 위해 **지수 이동 평균(EMA, Exponential Moving Average)**을 도입하여 먼 과거의 기울기는 잊고 **최근 기울기의 크기 위주로 학습률을 보정**하도록 설계되었다.

---

## 2. 수식 및 동작 원리

### (1) 기울기 제곱의 지수 이동 평균 (EMA) 계산
$$
v_t = \gamma v_{t-1} + (1 - \gamma) g_t^2
$$
- $v_t$: 시점 $t$에서의 기울기 제곱의 지수 이동 평균 (과거와 현재의 가중치 조절)
- $\gamma$ (감쇠율, Decay Rate): 과거 정보 반영 비율 (보통 `0.9` 또는 `0.99` 사용)
- $g_t$: 현재 시점의 손실 함수 기울기 ($\nabla_\theta L(\theta_t)$)
- $g_t^2$: 요소별(Element-wise) 제곱값

### (2) 가중치(파라미터) 업데이트
$$
\theta_{t+1} = \theta_t - \frac{\eta}{\sqrt{v_t + \epsilon}} \odot g_t
$$
- $\theta$: 모델의 가중치(Parameter)
- $\eta$ (Learning Rate): 기본 학습률 (일반적으로 `0.001` 권장)
- $\epsilon$ (Epsilon): 분모가 0이 되는 것을 방지하는 아주 작은 상수 ($10^{-8}$)
- $\sqrt{v_t + \epsilon}$: 기울기 제곱 평균의 제곱근 (**RMS**, Root Mean Square)
- $\odot$: 요소별 곱셈 (Hadamard product)

### (3) 직관적 메커니즘 해석
- **기울기가 가파르고 진동이 큰 축**: $g_t^2$가 커지면서 $v_t$가 커짐 $\rightarrow$ 분모($\sqrt{v_t}$)가 커져 **실제 이동 폭(유효 학습률)이 축소**되어 튀지 않고 안정화됨.
- **기울기가 완만하고 평탄한 축**: $g_t^2$가 작아 $v_t$도 작아짐 $\rightarrow$ 분모($\sqrt{v_t}$)가 작아져 **실제 이동 폭이 확대**되어 빠르게 최적점을 향해 진행함.

---

## 3. 실전 사용 예시

### PyTorch 코드 예시
```python
import torch
import torch.nn as nn
import torch.optim as optim

model = nn.Linear(10, 1)
criterion = nn.MSELoss()

# RMSProp 옵티마이저 정의 (기본 alpha/gamma=0.99, lr=0.01)
optimizer = optim.RMSprop(model.parameters(), lr=0.001, alpha=0.9, eps=1e-8)

# 학습 루프
for epoch in range(100):
    optimizer.zero_grad()
    inputs = torch.randn(32, 10)
    targets = torch.randn(32, 1)
    
    outputs = model(inputs)
    loss = criterion(outputs, targets)
    loss.backward()
    
    optimizer.step()
```

### 주로 사용되는 환경
- **순환 신경망(RNN, LSTM)**: 시계열이나 자연어 처리처럼 시점마다 기울기 변화가 큰 모델 학습
- **강화학습(Reinforcement Learning, DQN 등)**: 손실 함수의 형태가 실시간으로 변하는 비정상(Non-Stationary) 환경

---

## 4. 장단점 및 알고리즘 비교

### 장점
1. **학습 조기 중단 방지**: AdaGrad와 달리 최근 기울기만 반영하므로 끝까지 꾸준히 학습 가능.
2. **협곡/안장점(Saddle Point) 탈출**: 축마다 다른 유효 학습률을 가져 급격한 진동을 억제하고 완만한 방향으로 빠르게 이동.

### 한계점 및 주의사항
- **하이퍼파라미터 민감도**: 감쇠율($\gamma$)과 학습률($\eta$) 설정에 영향을 받음.
- **관성(Momentum) 부재**: 속도 방향성을 직접 누적하는 모멘텀 개념이 없어, 지역 최적해(Local Minima) 탈출 시 Adam보다 다소 느릴 수 있음.

### 옵티마이저 비교 요약
- **SGD**: 고정된 학습률, 진동이 심하고 수렴이 느림.
- **AdaGrad**: 모든 과거 기울기 누적 $\rightarrow$ 학습률이 0으로 급감하는 한계.
- **RMSProp**: 최근 기울기만 지수 이동 평균으로 누적 $\rightarrow$ 학습률 급감 해결.
- **Adam**: **RMSProp**(학습률 적응) + **Momentum**(방향/관성 가속)의 결합형.
