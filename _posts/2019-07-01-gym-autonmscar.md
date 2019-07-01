---
layout: post
title: "gym으로 env 만들기 연습"
published: true
---

OpenAI gym 을 이용해 RL agent 를 위한 학습 환경을 만드는 연습을 했다. 이 글에서는 연습을 하면서 무슨 일이 있었는지를 간략히 정리한다. 2019년 6월 27일부터 7월 1일까지 5일간 있었던 일이다.

* Project Repository: [gym-autonmscar](https://github.com/Jueun-Park/gym-autonmscar)

## 재료

### OpenAI gym

[OpenAI gym](https://gym.openai.com/) 은 RL agent 를 위해 이미 만들어진 여러 강화학습 환경을 제공한다. 또한 `gym.Env` 를 상속받아서 나만의 강화학습 환경을 만들 수도 있다.

### Stable baselines

[Stable Baselines](https://github.com/hill-a/stable-baselines) 에서는 미리 구현된 최신의 RL agent 들을 제공한다. OpenAI [Baselines](https://github.com/openai/baselines) 대신에 `stable-baselines`를 쓰기로 한 이유는 이 프로젝트는 documentation 이 있다고 리드미에서 자랑하고 있었기 때문이었다. 아무 것도 모르는 사람은 예제 코드가 많이 필요했다.

### Original game code

원본 게임 코드는 [Autonomous-Car-Simulator](https://github.com/x2ever/Autonomous-Car-Simulator) 이다. 작년 11월에 [자율차 프로젝트](https://jueun-park.github.io/2018-11-25/thinkingo-system-architecture) 같이 했던 [x2ever](https://github.com/x2ever) 님의 코드이다. 이 분이 ThinkinGo 자율주행 해커톤 하려고 만들어 둔 자율차 시뮬레이션 코드를 가져와서 강화학습 용으로 바꿔보기로 했다.

#### 게임 선택 이유

1. 원래 코드의 `Brain.py`가 agent의 역할을 하는 구조를 가지고 있었다. 그래서 아무것도 모르는 5일 전의 나는 그 부분을 바꾸면 쉽게 되는 줄 알았다.
2. ~~왜인지~~ LiDAR 데이터가 익숙했다.

## 일의 순서

### 1. 포크 해 와서 디렉토리 구조 바꾸기

[한 블로그 글](https://medium.com/@apoddar573/making-your-own-custom-environment-in-gym-c3b65ff8cdaa)을 참고하여 다음과 같이 디렉토리 구조를 바꾸었다.

```
gym-autonmscar/
├ README.md
├ setup.py
└ gym_autonmscar/
  ├ __init__.py
  └ envs/
    ├ __init__.py
    ├ autonmscar_env.py
    └ autonomous_car_simulator/
      └ [포크해 온 게임 코드들]
```

### 2. `virtualenv` 로 작업 환경 구성하기

모듈들의 버전이 꼬이는 것을 방지하기 위해서 가상 환경을 사용하는 것이 정신 건강에 이롭다. `pip install virtualenv` 를 사용했다.

### 3. 기존 게임 코드 읽기

대충만 읽어보고 포크해 온 게임 코드를 내가 필요한 부분 위주로 다시 자세하게 읽어봤다. 자세하게 읽기 위해 pygame에 대해서도 알아보았다. 키 이벤트를 이벤트 큐를 통해서 받아온다는 것을 알게 되었다.

### 4. `AutonomousCarEnv` 구현

그리고 강화학습 환경을 구현했다. 처음엔 `AutonomousCarEnv` 가 `Brain` class 를 상속받게 해서 구현할 수도 있겠다고 생각했었는데, 그보다는 그냥 Env를 Brain과 비슷한 구조로 만들기만 해도 괜찮겠다는 판단을 내렸다. 본래 게임에 있던 `main.py` 코드를 참고해서 `AutonomousCarEnv` 의 `__init__()` 을 구현했다. 이어서 `step()`, `reset()`, `render()` 메서드를 구현했다. 이 때는 비슷하게 `pygame` 을 이용해 만든 게임을 `gym.Env` 로 만든 프로젝트인 [`gym-worm`](https://github.com/kwk2696/gym-worm) 을 주로 참고했다.

### 5. 테스트와 디버깅

#### 테스트

`stable-baselines` 에 구현되어 있는 DQN 을 이용해 모델을 학습기키고 그 모델을 불러와 컴퓨터가 게임을 하는 걸 사람이 구경할 수 있는 테스트 코드를 짰다. 프로젝트 레포의 `test` 디렉토리에 저장되어 있다.

#### 있었던 문제

차가 벽에 박아서 터지고 새 에피소드가 시작되어야 할 때 자동차 위치가 초기화가 안 되는 문제가 있었다.

#### 디버깅

디버거를 돌려서 어디서 초기화가 안 되는 것인지 확인했다. 자동차 객체가 얕은 복사라서 초기화가 안 되었다. 환경을 `reset()` 할 때 자동차 객체를 늘 새로 만들어주는 것으로 ~~코드 구조는 좀 이상하지만~~ 해결했다.

#### Reward 정하기

학습 테스트를 돌려 가면서 agent 에게 줄 reward 를 *적당히* 정했다. `tensorboard` 를 이용해서 agent 가 timestep 에 따라 reward 를 받는 그래프도 확인해 보았다.

### 6. 리드미 작성

처음으로 프로젝트 리드미를 영어로 써 봤다.

## 결론과 소감

어려웠다. 학습이 잘 되도록 agent 의 파라미터를 바꿔 가며 여러 번 실험해보는 건 이 환경에서는 안 해 보았다. (아마 나중에 해 보게 될 것이다...)

직접 만들어 보니 강화학습 환경의 구조를 조금 알 수 있었다.
