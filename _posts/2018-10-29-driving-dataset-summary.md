---
layout: post
title: "Driving Dataset Summary"
published: True
---

이 글은 자율주행 기술 관련 오픈소스 데이터셋의 목적과 구조를 정리한 글입니다. 단순히 검색을 통해 알 수 있는 정보들을 제가 나중에 한 곳에서 보기 쉽게 정리한 것입니다. 대표적으로 Berkeley DeepDrive 와 Baidu Apollo 두 군데에서 제공하는 데이터셋들 중 제가 사용해 볼 가능성이 있는 것들을 위주로 정리했습니다.



# 용어

## 어노테이션 데이터

일반적으로 classification을 위한 supervised 딥러닝 모델을 학습시킬 때 사용하는 데이터셋은 모델이 구분해야 할 input 데이터와 그에 맞는 label 데이터가 한 쌍입니다. 이를 annotation data라고 부르고 있습니다. 이 데이터는 인풋 데이터가 시간 순서에 따라 연속적일 수도 있고, 아닐 수도 있습니다. 연속적인 경우에는 모델의 성능을 높이기 위해 인위적으로 불연속적이고 순서가 무관한 데이터로 가공해서 사용할 수도 있습니다.



## 시간 연속 데이터

센서의 데이터를 시간 순서대로 연속적으로 입력해서 저장한 데이터를 시간 연속 데이터라고 칭하겠습니다. 이런 데이터는 자율주행 인지 시스템의 테스트 환경을 구성할 때 더미 센서 데이터로 활용할 수 있습니다. 예를 들어, 주행 중 연속적으로 녹화한 전방 카메라 영상 데이터는 그 영상이 마치 현재 카메라 센서로 들어오는 데이터인 것처럼 테스트하고자 하는 인지 시스템에 인풋으로 줄 수 있습니다.



---



# 오픈 소스 데이터셋

## [Berkeley DeepDrive](http://bdd-data.berkeley.edu/index.html)

Berkeley DeepDrive는 BDD100K 라는 이름으로 오픈 소스 데이터셋을 제공하고 있습니다. 메일 인증을 통해 가입하고 프로필 페이지에 들어가면 다운로드할 수 있습니다. 데이터셋에 대한 상세한 설명은 다음 레포지토리 링크에도 서술되어 있습니다. [`ucbdrive/bdd-data`](https://github.com/ucbdrive/bdd-data). 이 레포지토리에는 라벨 데이터를 파싱하는 스크립트도 공개되어 있습니다.

다음은 다운로드 페이지에서 접근 가능한 각 파일들에 대한 설명들입니다.

### Videos and Info

주행 영상 데이터와 그와 함께 로깅한 GPS/IMU 데이터입니다. (주행 데이터이므로 시간 연속 데이터인 것 같습니다.)

> 100K video clips
>
> The GPS/IMU information recorded along with the videos

### Labels

각 비디오와 대응되는 라벨 데이터의 정보는 JSON 포맷으로 제공됩니다.

>  Annotations of road objects, lanes, and drivable areas in JSON format.

포멧의 상세 설명은 다음 링크에 있습니다. [`bdd-data/doc/format.md`](https://github.com/ucbdrive/bdd-data/blob/master/doc/format.md). 다양한 종류의 어노테이션을 제공합니다.

### Segmentation

> Full-frame semantic segmentation maps. The corresponding images are in the same folder.

프레임의 전체 영역을 segmentation annotation 한 라벨 데이터입니다.

### Drivable Maps

> Segmentation maps of Drivable areas.

위와 달리 전체 영역을 segmentation 하지 않고, 주행 가능 영역에 대해서만 segmentation annotation을 진행한 라벨 데이터입니다. `.png` 파일이며, per-pixel annotation 되어있습니다.

### Images

이미지 데이터셋은 두 개의 서브폴더로 나누어져 있습니다. 위의 비디오 데이터에서 추출한 이미지 데이터입니다.

> 1) 100K labeled key frame images extracted from the videos at 10th second
>
> 2) 10K key frames for full-frame semantic segmentation.



---



## [Baidu Apollo](http://data.apollo.auto/?locale=en-us&lang=en)

바이두 아폴로 오픈 플랫폼은 다음과 같은 어노테이션 데이터셋을 제공합니다. Simulation Scenarios Data 와 Demonstration Data 또한 제공하나 다음 내용에는 Annotation Data에 대한 설명만을 정리했습니다. Apollo 데이터는 홈페이지에서 핸드폰 번호 인증을 한 뒤 다운로드 받을 수 있습니다.

### Lidar Point Cloud Obstacle Detection & Classification

3차원 LiDAR 데이터와 그에 직육면체 라벨을 단 데이터셋입니다. 라벨 종류는 pedestrians, vehicles, non-motor vehicles (cyclists) and others (dontCares) 로 이루어져 있습니다. 20,000개 프레임을 샘플로 제공합니다.

* LiDAR: `.bin`
* Label: `.txt`

### Traffic Lights Detection

신호등 이미지와 그 라벨 데이터셋입니다. 20,000개 프레임을 샘플로 제공합니다.

* Image: `.jpg`
* Label: `.txt`

### Road Hackers

End-to-end 학습을 위한 데이터셋입니다. 센서의 raw 데이터와 차량의 motion 데이터가 쌍을 이루고 있습니다.

* Image: `.h5`, Key-Value (Key:UTC time, Value:320\*320\*3 pixel matrix)
* Label: `.h5`, 2D array

### Image-Based Obstacle Detection And Classification

비전 센서를 통해 장애물을 인식하기 위한 데이터셋입니다. 라벨은 다음과 같습니다: vehicles, pedestrians, cyclists, tricycle and other unmovable obstacles on the road (ex. traffic cone). 20,000개 프레임을 샘플로 제공합니다.

* Image: `.jpg`
* Label: `.txt`

### Obstacle Trajectory Prediction

장애물의 움직임과 장애물의 종류를 라벨링 한 데이터셋입니다. 다른 차량의 움직임을 보고 그 차량의 미래 행동을 예측하는 모델을 학습시키기 위해 사용할 수 있습니다. 다른 차량이 현재 차선을 따르는 중인지의 여부, 앞으로 1초 안에 차선을 따르고 있을지의 여부 두 가지가 각각 T/F로 구분되어 총 네 가지의 라벨이 붙습니다.

* Features of obstacles (obstacle movement data) and lables: `.h5`

### Scene Parsing Dataset

카메라로 인식한 이미지와 LiDAR로 인식한 depth 이미지를 인풋으로 넣으면 per-pixel classification 하는 모델을 학습시키는 데에 사용할 수 있는 데이터셋입니다. 26개 종류의 라벨로 어노테이션 되어있습니다.

* Image: `.jpg`
* Depth: `.png`
* Label: `.png`



---



# 추가 정보

## Domain Adaptation

BDD100K 데이터셋은 추가로 Apollo 데이터셋과의 adaptation을 가능하게 하는 자료들을 제공하고 있습니다. [(링크)](https://github.com/ucbdrive/bdd-data/tree/master/doc/apollo) 어떻게 쓰는지는 잘 모르겠지만, 아마 segmentation domain의 정보를 상호간에 변환할 수 있게 하는 것 같습니다.

## [Scalabel](https://www.scalabel.ai/)

Scalabel 은 데이터 라벨링을 도와주는 오픈 소스 툴입니다. Berkeley DeepDrive에서 제공합니다.



# 생각

데이터셋 용량이 너무 큽니다. Apollo는 샘플로 2만 프레임씩밖에 안 주는데, BDD100K는 영상 데이터라 그런지 무려 1.8TB라는 어마어마한 사이즈를 자랑합니다. (그래서 토렌트도 제공하고 있습니다.) 실제로 받아서 사용하려면 작정하고 머신을 잘 세팅한 다음에 시작하거나, 일부만 받아서 사용할 수 있는지를 알아봐야 할 것 같습니다.