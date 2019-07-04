---
layout: post
title: "직접 만든 gym-env 에서 강화학습 돌려보기"
published: true
---

[지난 글](https://jueun-park.github.io/2019-07-01/gym-autonmscar)에서 기록했던 대로 직접 만든 강화학습 환경이 잘 만들어졌는지 확인하기 위해서 이미 만들어진 RL 에이전트를 그 환경 위에서 학습시켜 보았다. 이 글은 그 과정과 결과를 간략히 설명하고 있다. DQN, DDPG로 각각 학습시킨 모델의 행동 모습을 gif 애니메이션으로 저장했다. 또한, discrete action space 로 구현한 env 를 어떻게 continuous action space 를 가지는 환경으로 추가로 구현할 수 있었는지를 기록해 두었다.

* [Project repository](https://github.com/Jueun-Park/gym-autonmscar)

## Training code 작성

Callback 함수를 구현해서 최근의 평균 리워드를 매번 구해서 제일 좋은 것으로 갱신될 때마다만 모델이 저장되도록 하였다. `stable-baselines`의 [예제 코드](https://stable-baselines.readthedocs.io/en/master/guide/examples.html#using-callback-monitoring-training)를 참고하였다.

## DQN

### 조금 더 간단한 맵으로

일단 DQN 에이전트를 이용해서 학습을 돌려 보았다. 원래 게임의 1레벨 맵이 너무 복잡하고 오히려 3레벨 맵이 더 쉽길래 그것으로 교체했다.

### DQN이 게임 하는 모습

학습시키는 중간에 궁금해서 지금까지 저장된 모델이 플레이하는 모습을 확인했더니 다음과 같았다.

![493000]({{ "/assets/2019-07-04/atnms-dqn_493000.gif" | absolute_url }})

이건 좀 웃겼다. 약 49만 타임스텝 쯤에 저장된 모델이다.

그리고 가장 높은 리워드를 받은 모델은 다음과 같이 플레이했다. 54만 타임스텝쯤 지난 후의 모델이다.

![544000]({{ "/assets/2019-07-04/atnms-dqn_544000.gif" | absolute_url }})

벽에 부딪히지 않으려고 가만히 멈춰 버리는 모습이 참 안타깝다.

다음 날 아침에 보니 reward 그래프가 음수 영역에서 계속 아래로 내려가고만 있었다. 더 이상 올라가지 않을 것 같아서 학습을 중지시켰다.

## DDPG

### Continuous environment implementation

아무 생각 없이 DQN을 DDPG 모델로 바꾸기만 해서 실행시켰더니 다음과 같은 메세지를 만났다.

```console
AssertionError: Error: DDPG cannot output a Discrete(4) action space, only spaces.Box is supported.
```

DDPG를 사용하려면 `spaces.Box` 를 이용해서 continuous action space를 만들어야 한다는 것을 이 때 처음으로 알았다. 그래서 그렇게 구현했다.

#### 이미 구현된 discrete action space 환경에 continuous action space 환경 추가 구현하기

우선 나는 어떻게 discrete action space 를 가진 env에 동시에 continuous action space 를 가지는 것을 추가하여 작성할 수 있는지를 몰랐다. `stable-baselines` 의 예제 코드에서 DDPG 를 쓰는 코드가 환경을 무엇을 사용하고 있는지 보았더니, `LunarLanderContinuous-v2` 를 예시로 보여주고 있었다. 그래서 [그 코드](https://github.com/openai/gym/blob/master/gym/envs/box2d/lunar_lander.py)를 읽어보았다. 참고하여 다음과 같이 상속을 해 줘서 쉽게 구현할 수 있었다.

##### [`gym_autonmscar/envs/autonmscar_env.py`](https://github.com/Jueun-Park/gym-autonmscar/blob/master/gym_autonmscar/envs/autonmscar_env.py)

```python
class AutonomousCarEnv(gym.Env):
    continuous = False

    (...)
    [원래 구현된 env]

class AutonomousCarEnvContinuous(AutonomousCarEnv):
    continuous = True

```

이렇게 하면 `[원래 구현된 env]` 부분을 `self.continuous == True` 일 때 실행되어야 하는 부분을 그렇지 않은 부분과 나누어 작성하여 continuous action space 를 가지는 환경을 구현할 수 있다.

그리고 `gym_autonmscar/__init__.py`에 다음과 같은 줄을 추가해서 작성했다. `gym.envs` 의 레지스터에 `id`를 등록하는 코드이다.

##### [`gym_autonmscar/__init__.py`](https://github.com/Jueun-Park/gym-autonmscar/blob/master/gym_autonmscar/__init__.py)

```python
from gym.envs.registration import register
(...)
register(
    id='autonmscarContinuous-v0',
    entry_point='gym_autonmscar.envs:AutonomousCarEnvContinuous',
)
```

이렇게 하고 나니 다음과 같이 환경을 만들 때 `id`만 바꿔서 적어주면 continuous action space 를 갖는 환경을 생성할 수 있었다.

```python
import gym
import gym_autonmscar
env = gym.make('autonmscarContinuous-v0')
```

#### Continuous action spaces

이제 `spaces.Box` 를 이용해서 continuous action space를 만들면 바로 테스트해볼 수 있게 되었다. Continuous 한 값을 action 으로 맵핑하는 방법 중에 생각해낼 수 있었던 가장 간단한 방법은 에이전트더러 네 개의 action 중에 각각을 선택할 확률을 내뿜으라는 것이었다. [0, 1] 범위에서 선택하라는 식으로 하면 되나 하면서 썼더니

```console
AssertionError: Error: the action space low and high must be symmetric
```

이렇다고 했다. 생각해 보니 그렇게 하면 다 더해서 1이 되지도 않을 것 같다. 그래서 다음과 같이 [-1, 1] 중에 고를 수 있도록 action space 를 짠 다음에

```python
from gym import spaces
(...)
    self.action_space = spaces.Box(low=-1, high=1, shape=(4, ))
```

아래와 같이 `step()` 메서드에서 softmax 함수를 이용해서 확률로 바꾼 뒤 그 확률에 따라 액션을 선택하도록 작성했다.

```python
if self.continuous:
    action = softmax(action)
    action = int(np.random.choice(4, 1, p=action))
```

사실 지금 생각해보면 이 액션 모델은 별로 현실적이지도 않고 좋은 디자인이 아닌 것 같다. 예를 들어 `LunarLanderContinuous-v2` 는 로켓 엔진의 출력 정도를 연속된 실수 값으로 선택할 수 있게 하지, 고정된 출력 값을 선택하게 하지는 않는다.

##### [`lunar_lander.py`](https://github.com/openai/gym/blob/master/gym/envs/box2d/lunar_lander.py) 의 action space

```python
if self.continuous:
    # Action is two floats [main engine, left-right engines].
    # Main engine: -1..0 off, 0..+1 throttle from 50% to 100% power. Engine can't work with less than 50% power.
    # Left-right:  -1.0..-0.5 fire left engine, +0.5..+1.0 fire right engine, -0.5..0.5 off
    self.action_space = spaces.Box(-1, +1, (2,), dtype=np.float32)
else:
    # Nop, fire left engine, main engine, right engine
    self.action_space = spaces.Discrete(4)
```

### DDPG가 게임 하는 모습

다음은 25만 타임스텝 정도 학습시킨 후의 DDPG 모델이 게임하는 모습이다.

![252000]({{ "/assets/2019-07-04/atnms-ddpg_252000.gif" | absolute_url}})

DQN과 다르게 확률에 따라 액션을 선택하고 있기 때문에 매 에피소드마다 다른 행동 양상을 보여준다.

## gif 로 저장하기

저장된 모델의 실행 결과를 gif로 저장하기 위해서 `stable-baselines`의 [예제 코드](https://stable-baselines.readthedocs.io/en/master/guide/examples.html#bonus-make-a-gif-of-a-trained-agent)를 참고했다. `import imageio`후 `mimsave()`(i.e. `mimwrite()`) 메서드를 이용하면 `ndarray` 데이터 타입의 이미지를 gif 파일로 저장할 수 있다.

이를 위해서 `'rgb_array'` mode로 `render()`메서드를 실행시켜서 `ndarray` 형태의 이미지를 반환할 수 있도록 구현했다. 왜인지 직접 형태를 바꿔서 반환하면 안 되길래 디스크에 썼다가 다시 불러오는 방식으로 코드를 작성했다. 나중에 바꾸고 싶다고 생각은 했지만 일단 돌아가는 코드니까 아마 안 바꾸겠지.

```python
img = env.render(mode='rgb_array')
```

대충 이런 식으로 사용하면 된다.

## README 수정하기

구현한 내용이 바뀌어서 그 설명을 리드미에 추가했다.

## 생각

구현을 시작하기 전에 강화학습 공부를 너무 제대로 안 한 것 같다. 그래도 직접 해 보면서 더 공부하면 나중에 더 잘 기억나겠지 하는 생각으로 살아보자.

이 프로젝트를 하다 보니 모방은 창조의 어머니라는 말이 떠올랐다. 아직 *창조*를 한 것 같지는 않지만. *모방*은 많이 했다.
