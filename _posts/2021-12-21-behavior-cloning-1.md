---
layout: post
title: "Imitation 라이브러리의 Behavior Cloning 써보기"
published: true
---

강화학습의 behavior cloning (BC) 을 해봐야 할 필요가 생겨서 파이썬 `stable baselines3` (SB3) 기반의 imitation learning 라이브러리 `imitation` 을 써 보았다.
그런데 `imitation` 의 데이터셋과 내가 수집해 둔 데이터셋의 인터페이스가 달라서 맞춰주는 일을 해야 했다.
그래서 그 과정을 짧은 글로 담아두고자 한다.

## 환경

실험 환경은 드론 자율주행 태스크를 학습시키기 위해 Microsoft AirSim 을 이용해 구현해둔 AirSim Drone 환경을 사용하고 있다 (구현은 동료분께서).
Observation space shape (141,) action space shape (3,) 으로 이루어져 있고, 매 스텝 reward 로는 이전 스텝에서 목적지까지의 거리에서 이번 스텝에서 목적지까지의 거리를 뺀 값을 받는다.
목적지에 도달하면 그 다음 목적지가 다시 생성되며, 장애물에 부딪히거나 정해진 범위를 벗어나면 에피소드가 종료된다.

## 데이터셋

사실 내가 가지고 있었던 데이터셋은 두 가지 종류였다.
한 가지는 `SB3` 의 `SAC` (soft actor-critic) 알고리즘으로 수집한 `ReplayBuffer` 객체였고, 다른 하나는 이 리플레이 버퍼 객체로부터 변환한 `d3rlpy` 라이브러리의 `MDPDataset` 객체였다.
`d3rlpy` 라이브러리는 오프라인 강화학습을 위한 알고리즘을 모아둔 라이브러리이다.
나는 직접 수집한 데이터를 이용할 때 `d3rlpy` 의 `CQL` (conservative Q-learning) 알고리즘으로 학습이 잘 되지 않는 문제를 겪고 있었으므로, 동료 분의 조언에 따라 우선 **데이터가 잘 모이고 잘 변환되었는지**를 확인하기 위해서 BC 를 해보기로 했다.

## `SB3 ReplayBuffer` to `d3rlpy MDPDataset`

`d3rlpy` 에서는 `SB3` 의 버퍼로부터 `MDPDataset` 을 생성하는 함수를 이미 제공하고 있다.
위치는 [`d3rlpy/d3rlpy/wrappers/sb3.py`](https://github.com/takuseno/d3rlpy/blob/66764c69897a049b2907a024638cae987cae27fe/d3rlpy/wrappers/sb3.py#L63) 이다.
나는 이를 이용해 피클링 된 (`.pkl`) 리플레이버퍼 객체로부터 쉽게 `MDPDataset` 객체를 얻고 dump 하여 `.h5` 파일로 저장할 수 있었다.

## `d3rlpy MDPDataset` to `imitation Trajectories`

`imitation` 라이브러리의 quick start example 에서 BC 학습에 필요한 [데이터셋의 형태를 확인](https://github.com/HumanCompatibleAI/imitation/blob/dacb2425e9d19d57e318578a414b50cb98ead647/examples/quickstart.py#L17)했다.

> `# This is a list of imitation.data.types.Trajectory, where`
>
> `# every instance contains observations and actions for a single expert`
>
> `# demonstration.`

따라서 나는 다음처럼 스크립트를 작성했다.

```python
mdp_dataset = MDPDataset.load("mdp_dataset_expert.h5")
mdp_dataset.build_episodes()

trajectories = []
for epi in mdp_dataset.episodes:
    trajectory = Trajectory(epi.observations, epi.actions[:-1], None)
    trajectories.append(trajectory)

with open("trajectories_expert.pkl", "wb") as f:
    pickle.dump(trajectories, f)
```

위 스크립트를 이용하니 데이터 변환은 잘 이루어졌다.
`for` 문 안에서 `Trajectory` 를 만들 때 `epi.actions[:-1]` 처럼 action 의 맨 마지막을 뺀 이유는 다음과 같은 에러를 만났기 때문이다.

```bash
ValueError: expected one more observations than actions: 235 != 235 + 1
```

데이터셋에 next observation 이 항상 필요하기 때문에 생기도록 한 에러인 듯 하다.

## 학습

`imitation` 의 [quick start example](https://github.com/HumanCompatibleAI/imitation/blob/master/examples/quickstart.py) 을 보고 학습 코드를 작성했다.
여기서 라이브러리가 업데이트가 됐는지 logger 가 안 들어가는 이슈가 있었는데 귀찮은 나는(...) 그냥 로거를 빼 버리고 학습시켰다. 그래서 평가 스크립트를 따로 짰다.

## 평가

저장한 폴리시를 불러와서 평가하는 스크립트를 다음처럼 짰다.

```python
n_eval = 10
env = gym.make("AirSimDrone-v0")
policy = torch.load("AirSimNH_medium.pt")
total_reward = 0
for epi in range(n_eval):
    obs = env.reset()
    done = False
    while not done:
        action, _ = policy.predict(obs, deterministic=True)
        obs, reward, done, info = env.step(action)
        total_reward += reward
print(total_reward / n_eval)
```

액션을 뽑을 때 조금 고생했는데, 디버깅 끝에 `imitation` 의 `policy` 객체가 `SB3` 의 것을 상속받아서 쓰기 때문에 `action, _ = policy.predict(obs, deterministic=True)` 와 같이 `SB3` 와 동일한 형태로 사용해야 한다는 것을 알았다.

## 결과

`SAC` 알고리즘으로 중간까지 학습시킨 폴리시에서 수집한 버퍼인 `medium` 데이터셋으로 우선 학습시켜서 평가해 보았다.
`CQL` 을 사용했을 때 가만히 있는 폴리시로 수렴했던 것에 비해 BC 를 이용하니 일단 드론이 움직이기는 시작했다.
BC 를 할 때는 잘못된 사례를 가르치지 않도록 [주의해야 한다는 글](https://jsideas.net/BC/)을 읽고 지금은 성능이 잘 나오는 `expert` 폴리시로 수집한 데이터셋으로 학습을 진행하고 있다.

지금 학습시키고 있는 BC 가 끝나서 성능을 보면 앞으로의 연구 방향이 다시 결정될 것이다.
문제를 해결하고 있는 과정이니까 긍정적인 신호이다.

## 마치며

이렇게 두 시간 정도만에 글을 다 쓴 것 같다.
학습 돌려 두고 빈 시간동안 갑자기 삘받아서 작성해 봤다...
오랜만에 블로그 업데이트 하려니까 기분이 좋다. 끝.
