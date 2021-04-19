---
layout: post
title: "Paper Reading [Clipper]"
published: false
---

Clipper는 예측 모델 서빙 시스템이다. 단순히 하나의 머신 러닝 모델을 서비스하는 게 아니라, 더 나은 결과를 내놓기 위해 여러 가지 머신 러닝 모델을 동시에 사용한다. 그러면서 동시에 레이턴시 *latency*[^1] 도 줄여 실시간 시스템으로 사용할 수 있게 하고자 한다.

[^1]: 지연 시간

프로그램 구조와 *Model Selection Layer* 위주로 페이퍼를 읽어 보았다. 앙상블 러닝이나 컴퓨터 시스템에 대한 이해가 적어서 처음 보는 용어가 많다. 나중에 다시 찾아볼 때를 대비해 간략히 정리해둔다.

##### 관련 링크

Crankshaw, Daniel, et al. ["Clipper: A Low-Latency Online Prediction Serving System."](https://www.usenix.org/system/files/conference/nsdi17/nsdi17-crankshaw.pdf) *NSDI*. 2017.

[Clipper Repository](https://github.com/ucbrise/clipper), [Presentation Video](https://www.usenix.org/conference/nsdi17/technical-sessions/presentation/crankshaw)

# System Architecture

![architecture]({{"/assets/2019-01-23/1.PNG" | absolute_url}})

* RPC: remote procedure call[^2]

총 두 개의 레이어인 *Model Selection*, *Model Abstraction* 으로 이루어져 있다.

[^2]: 호출하려는 함수나 프로시저가 원격의 다른 장치에 있어도 일반적으로 코딩하듯이 짜면 호출을 위한 통신까지 대신 해 주는 기술이라고 이해했다.

---

## Model Selection Layer

Model Selection 은 더 정확하고 강인 *robust* 한 예측을 하는 데에 그 목적이 있다. 이 레이어가 하는 일은 여러 가지 모델 중 상황에 따라 적절한 모델을 선택하는 것이다.

### 1 Single Model Selection Policy

쿼리를 주고 답을 얻을 모델을 선택하는 문제는 마치 손잡이가 여러 개인 벤딧 문제인 것처럼 생각할 수 있다. 확률적인 보상을 갖는 *k*개의 손잡이(모델)중 하나를 최적의 정책으로 선택하는 것이다. 페이퍼에서는 이 문제를 푸는 방법으로 *Exp3 algorithm*[^3] 을 시도해 봤다고 말하고 있다.

[^3]: 자세히는 모르겠지만... 대충 각 선택지를 선택하는 가중치를 설정하고 loss를 통해 이 가중치를 개선시켜나가는 알고리즘인 것 같다.

### 2 Ensemble Model Selection Policies

여러 개의 모델로부터 예측된 결과를 결합하는 기법인 앙상블 *ensemble* 은 예측의 정확도를 높이는 방법이라고 알려져 있다.

Clipper는 base model의 가중평균을 계산하는 데에는 *linear ensemble methods* 를 사용했다. 앙상블 가중치를 예측(학습)하는 데에는 *Exp4 algorithm* 을 사용했다[^4].

[^4]: ...라고 썼지만 어떤 개념인지 모르겠다.

##### 강인한 예측 *Robust Predictions*

강인한 예측을 위해 신뢰도의 threshold 값을 정해두고 그 값보다 낮은 값으로 예측할 경우 *sensible default decision*을 선택하도록 해 모델의 안정성을 높인다.

##### 낙오자 완화 *Straggler Mitigation*
모델의 개수를 늘려서 앙상블의 크기가 커질수록 낙오자[^5]가 생기고, 이는 평균 레이턴시를 증가시킨다. Clipper는 이 문제에 대해 *best-effort straggler-mitigation* 전략을 사용한다.

[^5]: 여러 모델이 있을 때 연산이 늦게 끝나는 모델을 낙오자 *straggler* 라고 표현한 것인가 하고 생각했다.

### 3 Contextualization

환경의 컨텍스트에 따라 다른 모델을 선택할 수 있다. 예를 들어, 음성 인식을 위해 사투리나 억양을 먼저 알아내고 그에 따라 다른 음성 인식 모델을 사용해서 인식 결과를 내놓을 수 있다.

---

## Model Abstraction Layer

여러 가지 머신 러닝 모델을 동시에 사용하는 Clipper는 서로 다른 모델을 동등한 계층에서 관리할 수 있는 다음과 같은 모듈 구조들을 가지고 있다. 따라서 새로운 머신 러닝 프레임워크를 쉽게 추가할 수 있다고 한다.

### 1 Caching

캐시 구조를 사용한다. 자주 오는 쿼리에 대한 답을 미리 기억해두고 같은 쿼리가 올 때 모델 연산 대신 사용하면 더 빠르게 답을 낼 수 있어 레이턴시를 줄일 수 있다. *LRU[^6] eviction policy* 를 사용한다.

[^6]: least recently used

### 2 Batching

빠른 연산을 위해 배치 *batch* 를 만든다. 배치는 GPU등의 하드웨어 자원을 효율적으로 사용할 수 있게 해 준다.

##### Dynamic Batching

다이나믹 배칭은 배치 크기를 결정하는 문제를 위한 방법이다. AIMD[^8] 방식을 이용해 최적의 배치 사이즈를 동적으로 결정한다.

##### Delayed Batching

기다렸다가 배치를 만드는 방식이다. 쿼리가 집중적으로 한 번씩 소규모로 *bursty* 오는 경우 최대 배치 사이즈보다 작을 수 있다. 이런 경우 어떤 모델은 쿼리가 더 오기를 기다렸다가 모아서 보내는 것이 효율적이라고 한다.

[^8]: additive-increase-multiple-decrease

### 3 Model Containers

Clipper에서 사용되는 각 모델은 *model container* 에 의한 캡슐 구조이다. *Model Container* 는 서로 다른 장비에 머신 러닝 모델들이 있을 때 동일한 인터페이스를 통해 쉽게 사용할 수 있는 환경을 제공한다. *Container-based architecture* 의 오버헤드를 줄이면서 동시에 서로 다른 언어로 작성된 모델도 쉽게 추가할 수 있도록 하는 것이 목표이다.



---

# 느낀 점

모르는 게 너무 많아서 문제다. 페이퍼의 인용을 따라가서 다 읽어야 하는 걸까? 아니면 아직 학부를 다 못 끝내서 읽는 게 버거운 걸까? ~~학부를 끝내도 버거우면 어쩌지?~~

앙상블 러닝에 대해 처음 알게 되었다. 배깅 *bagging*, 부스팅 *boosting* 같은 용어도 나왔는데 전부 처음 들어서 온전히 이해하지 못했다. 결국 전체 구조만 정리하게 되었고, 구현에 관한 내용들은 아직 많이 모른다는 사실을 깨달았다.

---

