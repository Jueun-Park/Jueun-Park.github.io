---
layout: post
title: "Paper Reading [IDK Cascades]"
publish: false
---

머신 러닝 모델 여러 개를 결합하는 어떤 방식을 소개하는 페이퍼이다. **IDK는 *I don't know* 의 약자**라고 한다.

Wang, Xin, et al. ["IDK Cascades: Fast Deep Learning by Learning not to Overthink."](https://arxiv.org/abs/1706.00885) *arXiv preprint arXiv:1706.00885*(2017).

# 목적

머신 러닝 모델의 정확도를 높이려면 연산 처리량을 늘려야 한다는 트레이드오프가 발생한다. 그러나 대부분의 경우 정확도가 필요한 순간은 그리 많지 않다. 페이퍼에서 제시하는 예시로는 감시카메라가 있다. 감시카메라는 보통 많은 시간 빈 거리를 살펴보기 때문에 어떤 물체가 화면 안에 들어오는 경우가 아니면 큰 정확도를 필요로 하지 않는다.

또한 저자들의 실험 결과, 모델 인풋 데이터 중 꽤 큰 비율[^1]이 *빠른 모델* 과 *느리지만 더 정확한 모델* 모두가 맞는 답을 내놓는 인풋이라고 한다. 이러한 인풋을 태생적으로 쉬운 *inherently easier* 문제라고 묘사하고 있다. 따라서 해당 페이퍼에서는 모델이 정답이라고 확신하는 정도가 높은 결과는  빨리 내어놓고, 잘 모르겠다고 하는 *IDK* 결과는 더 정확한 모델에 넘겨서 결과를 내어놓는 방식을 제안한다. 그리고 이러한 구조를 이용하면 간단하고 빠른 모델을 로컬에서 사용하고 복잡하고 정확도가 높은 모델은 클라우드에 올려서 필요할 때만 사용하는 등의 응용이 가능하다.

[^1]: 페이퍼의 예제에서는 약 48%.

# 구조

IDK prediction cascades는 *IDK classifier* 여러 개로 이루어져 있다. *IDK classifier* 는 이미 존재하는 분류기 모델 *base model* 과 그 모델의 불확실성을 측정하는 증대 분류기 *augmenting classifiers* 가 결합된 구조이다.

다음은 예시를 위해 두 가지 모델 $m^{fast}$ (빠르지만 정확도는 낮음), $m^{acc}$ (느리지만 정확도는 높음)을 결합하는 IDK classifier를 수식으로 표현하는 과정이다.

## 1 기본 정의

데이터셋을 $$\mathcal{D} = \left\{ (x_{i}, y_{i}) \right\} ^{n}_{i = 1}$$ 로 정의한다. 머신 러닝 모델은 다음과 같이 추정량으로 나타낼 수 있다. *(base model)*

$$m^{fast} (x) = \mathbf{\hat{P}}(\mathrm{class}\ \mathrm{label}\ |\ x)$$

이 모델의 불확실성을 0과 1 사이의 값으로 나타내는 *augmenting classifier* 를 정의한다.

$$h_{\alpha}(m^{fast}(x)) \to [0, 1]$$

이 *augmenting classifier* 를 학습시키는 것이 곧 *IDK classifier* 를 학습시키는 것과 같다. $h_{\alpha}(\cdot)$ 을 정의하고 나면 다음과 같이 IDK prediction cascade를 정의할 수 있다.

$$m^{casc}(x) =

\begin{cases}

m^{fast}(x) & \mathrm{if}\ h_{\alpha}(m^{fast}(x)) <= 0.5 \\

m^{acc}(x) & \mathrm{otherwise.}

\end{cases}$$

따라서 *전체 결합 모델의 정확도*를 최대로 하고 *결합 모델 내에서 연산량이 큰 모델을 사용하는 비율*을 최소로 하는 파라미터 $\alpha$의 최적값을 찾는 문제가 된다. 정확도는 $\mathbf{Acc}(m)$ 로 표현한다. 결합 모델 내에서 연산량이 큰 모델을 사용하는 비율은 곧 어떤 IDK classifier에서 불확실성이 높은 것의 비율로 나타낼 수 있으므로, $\mathbf{IDKRate}(h)$ 로 표현한다. 따라서 문제의 목표를 나타내면 다음과 같다.

$$\underset{\alpha}{min}\ \mathbf{IDKRate}(h_{\alpha})$$

$$\mathrm{s.t.:}\quad \mathbf{Acc}(m^{casc}) \ge (1 - \epsilon)\mathbf{Acc}(m^{acc})$$

*정확도가 높은 모델*만 사용하는 경우랑 거의 비슷한 정확도를 가지면서 동시에 *그 모델*을 사용하는 비율을 줄여 전체 연산량을 줄일 수 있는 모델을 만드는 $\alpha$값을 찾는 것이다.

## 2 각 모델이 예측하는 결과의 불확실성 계산하기

이제 $h_{\alpha}$ 를 구체적으로 정의할 차례이다. *confidence scores*를 도입하면, $m^{fast}$ 의 confidence가 충분하지 못할 때 더 정확한 모델이 사용되어야 한다는 것을 직관적으로 생각해볼 수 있다. 다음은 *uncertainty*를 예측하기 위해 entropy를 사용하는 예시이다. 다음과 같이 엔트로피 기반의 IDK classifier를 제안하고 있다.

$$h^{ent}_{\alpha}(m^{fast}(x)) = \mathbb{I}\left[\mathbf{H}\left[m^{fast}(x)\right] > \alpha \right]$$

$\mathbb{I}$ 는 indicator function이다[^5]. $\mathbf{H}$ 는 엔트로피 함수이다[^4].

[^4]: $$\mathbf{H} \left[ m^{fast}(x) \right] = - \sum_{j = 1}^{k} m^{fast}(x)_{j} \cdot \mathrm{log}\ m^{fast}(x)_{j} $$

[^5]: 0 또는 1을 반환한다.

### 실험

본 페이퍼에서는 세 가지 $h_{\alpha}$ (IDK classifier) 를 제안한 뒤 각각에 대해 실험하였다.

- Cascade by Probability
- Cascade by Entropy (Entropy based IDK classifier)
- Neural Network based IDK classifier

## 3 $\alpha$ 찾기

### Cost-aware Objective

ERM[^2]을 이용하기 위해 목표함수를 다음과 같이 정의한다.

$$J(\alpha) = {1 \over n} \sum_{i = 1}^{n} \left[ \mathbf{L}(y_{i}, m^{casc}_{\alpha}(x_{i})) + \lambda \cdot \mathbf{C}(m^{casc}_{\alpha}(x_{i})) \right]$$

$\mathbf{L}(\cdot, \cdot)$ 은 loss의 총합, $\mathbf{C}(\cdot)$ 는 연산량을 나타낸다[^3]. $\lambda$ 는 정확도와 연산량의 트레이드오프를 결정하는 하이퍼 파라미터이다. 따라서 이 *regularized loss* 목표함수가 있으면 stochastic gradient descent based algorithm 을 통해 IDK prediction cascade를 최적화할 수 있다고 한다. 목표로 했던 두 가지 (예측을 잘 하는 모델 만들기, 연산량 줄이기)를 한 번에 달성할 수 있도록 하는 목표함수이다.

[^2]: Empirical Risk Minimization, 통계적 학습법 중 하나라고 한다.
[^3]: 자세한 정의가 필요하게 되면 페이퍼를 다시 보는 걸로 하자. 두 개의 모델을 결합하는 경우와 일반적으로 여러 모델을 결합하는 경우에 대해 각각 설명되어 있다.

### 실험

다음과 같은 방법 중 $h_{\alpha}$ 에 따라 가능한 것을 조합해 여러 가지로 실험해보았다고 한다.

- Grid Search
- Cost-oblivious Cross-entropy (Binary)
- Cost-aware Objective

# 실험 결과

성능을 알아보기 위해 *accuracy*, *IDK rate* 를 측정하고 연산 속도를 측정하기 위해 *average flops* 와 *relative computational cost*를 계산하였다고 한다. 여러 모델의 cascade 형태일 경우 IDK rate 를 각 레벨마다 측정하였다.

#### 이미지 분류

![result1]({{"/assets/2019-01-31/1.PNG" | absolute_url}})

이미지 분류 모델 두 개를 IDK cascade로 각각 방법을 다르게 해 결합하여 성능을 측정한 결과이다. 이 외에도 모델 세 개를 결합한 cascade의 성능을 측정한 결과도 제시하고 있다[^7].

[^7]: 이 또한 페이퍼를 참고하는 걸로 하자.

#### Robustness Analysis

익스트림 케이스에서의 robustness 를 평가하기 위해 실험했다고 한다[^6].

[^6]: 잘 이해하지 못했다.

#### 자율주행 예측

Berkeley DeepDrive Video dataset을 이용해 IDK cascades가 실제로는 어떻게 응용될 수 있는지 보여주기 위한 실험을 진행했다. 비디오 프레임을 받으면 자동차의 다음 모션을 예측한다.

---

