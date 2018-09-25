---
layout: post
title: ubuntu 18.04에 gym_torcs(자율주행 강화학습 환경) 설치하고 테스트해보기
published: True
---

나는 어제 밤에 메일로 다음의 흥미로운 블로그 글 링크를 받았다. [[Using Keras and Deep Deterministic Policy Gradient to play TORCS](https://yanpanlau.github.io/2016/10/11/Torcs-Keras.html)] 설명이 친절하다. 이론을 이해하기 전에 일단 실행해보고 싶다는 생각이 들어서 우분투를 새로 깔았다. 컴퓨터는 동아리 컴퓨터~~헤븐컴~~을 사용했다.



# 설치 환경

* x86-64 Intel Core i7
* ubuntu 18.04

```sh
$ sudo apt install git
$ sudo apt install gcc
$ sudo apt install g++
$ sudo apt install python2.7
$ sudo apt install python-pip
```

~~있는대로 다 설치했다~~



# gym_torcs 설치

gym_torcs는 python gym을 이용해서 만든 자율주행 강화학습 환경이다.

[gym_torcs repo](https://github.com/ugo-nama-kun/gym_torcs) 에 있는 [vtorcs-RL-color/README.md](https://github.com/ugo-nama-kun/gym_torcs/tree/master/vtorcs-RL-color#linux-installation-from-source) 를 참고해서 아래와 같이 설치할 수 있었다.

```sh
$ git clone https://github.com/ugo-nama-kun/gym_torcs.git
$ cd gym_torcs/vtorcs-RL-color
$ sudo apt-get install libglib2.0-dev  libgl1-mesa-dev libglu1-mesa-dev  freeglut3-dev  libplib-dev  libopenal-dev libalut-dev libxi-dev libxmu-dev libxrender-dev  libxrandr-dev libpng-dev
$ ./configure
```

위 스크립트는 링크의 README와 조금 다르다. 설치할 때 해당 링크에 써 있는 대로 `libpng12-dev`라고 작성하면 설치가 안 되는 문제가 있었다.

```sh
'libpng12-dev' 패키지는 설치할 수 있는 후보가 없습니다
```

위와 같은 에러 메세지였다. 검색을 통해 `libpng-dev` 로 설치해주면 된다는 것을 알았고, 위의 스크립트는 그것을 반영해 수정한 것이다.

그 후 다음의 명령어를 입력해 빌드를 시도해본다.

```sh
$ sudo make
```

그랬더니 다음과 같은 에러가 나타났다.

```sh
geometry.cpp:373:8: error: 'isnan' was not declared in this scope
```

이 문제는 `isnan`이 std 안에 정의되어있기 때문에 나타나는 에러이다. `geometry.cpp`를 디렉토리 안에서 검색해서 찾아 해당 줄을 수정해주면 된다. 검색해보면 경로는 `gym_torcs/vtorcs-RL-color/src/drivers/olethros/geometry.cpp`로 나온다. 편집기에서 isnan을 검색해 다음과 같은 줄을 찾는다.

```cpp
if (isnan(r)) {
```

이 부분을 다음과 같이 수정해주고 저장한다.

```cpp
if (std::isnan(r)) {
```

이렇게 한 뒤 `$ sudo make`를 입력했더니 에러 없이 진행되었다. 이어서

```sh
$ sudo make install
$ sudo make datainstall
```

이것들을 마저 입력해 설치를 끝낸다.

```sh
$ torcs
```

오류가 없었다면 터미널에 위와 같이 입력했을 때 게임 창이 뜨는 것을 볼 수 있다.

---

### 직접 게임하기

위처럼 설치한 뒤 뜨는 게임 창에서는 내가 직접 게임을 해볼 수 있는 것은 아니다. 이 환경은 강화학습을 위해 항상 통신을 기다리도록 되어 있는 것 같다. 만일 직접 torcs 게임을 해 보고 싶다면 강화학습 환경이 아니라 해당 게임을 설치하면 된다.

```sh
$ sudo apt install torcs
```

위 명령어는 인간 플레이어가 게임할 수 있는 torcs를 설치해준다. 설치 뒤 실행은 똑같이 `$ torcs`로 한다.

강화학습 환경과 사람용 게임을 동시에 설치하면 아마 충돌이 나겠지만 나는 그렇게 해 보지는 않았다...



# DDPG-Keras-Torcs 로 학습시켜보기

앞서도 말했지만 나는 어떤 분이 이미 만들어 둔 프로젝트를 받아서 사용해 보기 위해 위와 같은 환경을 구축했다. [사용해보려는 코드](https://github.com/yanpanlau/DDPG-Keras-Torcs)는 자율주행 강화학습 네트웍을 Deep deterministic policy gradient를 이용한 방법으로 keras로 구현한 코드이다.

[아까 그 블로그 글](https://yanpanlau.github.io/2016/10/11/Torcs-Keras.html) 쓰신 분이 그 글에 친절하게 사용법을 적어 주셨다. `home` 위치에서 다음 스크립트를 실행하면 된다.

```sh
$ git clone https://github.com/yanpanlau/DDPG-Keras-Torcs.git
$ cd DDPG-Keras-Torcs
$ cp *.* ~/gym_torcs
$ cd ~/gym_torcs
$ python ddpg.py 
```

### 모듈 설치

역시 바로 실행은 안 된다. 에러 내용을 통해 무엇이 없는지 보고 다음과 같이 설치해주었다.

```sh
$ pip install gym
$ pip install keras==1.1.0
$ pip install tensorflow==0.12rc0
```

그리고 실행...해 봤는데도 안 되어서 다음을 설치했다. `autostart.sh` 에서 사용하는 `xte` 명령어가 없어서 실행되지 않았던 것이었다.

```sh
$ sudo apt install xautomation
```

그리고 뉴럴 네트웍을 학습시키기 위해 글에 써 있는 대로 `ddpg.py`를 편집기로 열어 `train_indicator=1`로 수정했다. 이제 `$ python ddpg.py`를 입력하면 뭔가 된다.



# 결과

![결과]({{ "/assets/2018-09-09/image.png" | absolute_url }})

학습 코드를 실행시켰는데 게임 창에서 자동차가 보이지 않는 문제가 있었다. ~~해결할 힘이 없다...~~ 주행 중 `F1`을 누르면 키 힌트를 볼 수 있고, `F2` 키가 드라이버 시점 카메라이다. 드라이버 시점으로 보면 자동차가 얼마나 이상하게 가고 있는지 잘 볼 수 있다. ~~그래도 나보다는 운전 잘 하는 것 같다~~ 에피소드는 기본 코드에는 2000번 학습하도록 되어 있다. 오래 걸린다.



# 후기

사실 생각보다 삽질을 조금 많이 했다. 중간에 gpu 써보겠다고 ~~(괜히 헤븐컴 쓰는 게 아니었다)~~ CUDA 설치 시도했던 게 제일 시간을 많이 잡아먹었다. 서로 다른 두 방법으로 했는데도 계속 의존성 문제가 생겨서 우분투를 두 번 새로 깔았다. 결국 CUDA는 설치 못 하고 그냥 cpu로 돌려보았다. 이 정도 삽질 했으니 이제 그냥 머신러닝이랑 강화학습 이론 공부를 해야겠다.

