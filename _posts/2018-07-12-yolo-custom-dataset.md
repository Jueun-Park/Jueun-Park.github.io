---
layout: post
title: 커스텀 데이터 셋으로 Yolo 써 보기 1
published: true
---



YOLO를 위한 데이터셋을 만들고 학습시키기까지 시도해 보았다. ~~방학이라 할 일이 없어서~~ 7개 종류의 표지판을 실시간으로 태깅하는 모델을 만들고 테스트 해 보고 싶었다.



# Dataset 만들기

## 동영상 준비

Team. HEVEN_A (자율차 2018팀)이 2018년 5월 연습 주행 때 웹캠으로 이미 녹화해 뒀던 로그 동영상을 이용했다.

## 동영상 캡처하기

5월에 서로 다른 두 날에 찍은 10개의 로그 동영상을 하나씩 읽어 shape detection 알고리즘(자율차 팀원이 개발해둔 것)을 거친 뒤 표지판이 있을 만한 프레임을 골라서 이미지로 저장하게 했다. python 스크립트를 작성해서 사용했다.

## 이미지 선별하기

shape detection의 정확도가 그리 높지 않아 스크립트를 거친 후에도 약 15,000장의 이미지가 나왔다. 그래서 수작업으로 먼저 표지판이 없거나 있더라도 너무 작은 이미지를 지웠다. 그래도 약 6,800개의 이미지가 남아서 그냥 한 class당 10개의 이미지를 임의로 선별하기로 했다. 총 70개 이미지의 데이터셋을 만들었다.

## Yolo_mark를 이용해 dataset 만들기

[how-to-train-to-detect-your-custom-objects](https://github.com/AlexeyAB/darknet#how-to-train-to-detect-your-custom-objects)를 통해 방법을 자세히 알 수 있었다. 그리고 [Yolo_mark](https://github.com/AlexeyAB/Yolo_mark)를 이용하면 쉽게 Yolo 학습용 데이터 셋을 만들 수 있다. 다음과 같이 생겼다.

![making-dataset]({{ "/assets/2018-07-11/2018-07-11-making-dataset.PNG" | absolute_url }})



[`Yolo_mark/x64/Release/data`](https://github.com/AlexeyAB/Yolo_mark/tree/master/x64/Release/data) 안에 예시 파일이 있어서 그것을 보고 필요한 파일들을 준비했다. 필요한 학습 데이터는 다음과 같다. img1은 임의의 이미지 데이터 이름이다.

- `img1.jpg` : 웹캠으로 녹화한 동영상의 전체 화면을 캡처한 이미지들이다.
- `img1.txt` : `<object-class>, <x>, <y>, <width>, <height>`의 내용이 한 줄씩 작성되어 있다. 각 줄은 해당 이미지에 존재하는 오브젝트의 종류, 그 오브젝트가 존재하는 x좌표, y좌표, 너비와 높이를 의미한다. 한 이미지 내에 여러 오브젝트가 있을 경우 여러 줄이다. Yolo_mark를 이용해 생성한다.

모든 이미지들에 대해 위의 세트를 모아서 한 디렉토리에 넣어 둔다. (예시 파일에서는 `./img`) 그리고 예시 파일을 보고 아래의 파일도 준비한다.

- `obj.data` : 커스텀 데이터에 대한 몇 가지 설정을 작성한다.
- `obj.names` : class 이름을 한 줄에 하나씩 적어 준다. 
- `train.txt` : `data/img/img1.jpg` 와 같이 학습시킬 이미지의 경로와 이름을 한 줄에 하나씩 적어 준다. (Yolo_mark를 이용하면 자동으로 생성된다)

# 학습 준비

## CUDA 및 cuDNN 설치

머신은 우리 동아리 컴퓨터의 GeForce GTX 1070을 사용했다. CUDA는 9.1, cuDNN은 7.0을 사용했다.

## darknet 다운로드하고 빌드하기

darknet은 Yolo 개발자가 제공하는 프레임워크로, 이미 구현된 Yolo 소스코드를 함께 제공한다. Windows 환경에서 돌려 보기 위해, github에 있는`AlexeyAB/`[`darknet`](https://github.com/AlexeyAB/darknet)`/build/darknet`을 사용했다. 로컬에 clone 한 뒤 README를 잘 읽으며 따라했더니 빌드할 수 있었다. (Linux 버전은 Yolo 개발자의 [원본 저장소](https://github.com/pjreddie/darknet)에 있다.) 

## configuration

`.cfg` 파일을 작성하여 몇 가지 설정을 해 준다. 아까의 그 [How to train](https://github.com/AlexeyAB/darknet#how-to-train-to-detect-your-custom-objects)을 보고 위 저장소의 [`yolov3.cfg`](https://github.com/AlexeyAB/darknet/blob/master/build/darknet/x64/cfg/yolov3.cfg)를 복사하여 수정했다. batch나 subdivisions 등을 설정할 수 있다. 설정에 따라 학습 결과가 달라질 것을 예상할 수 있다.



# 학습. . .

너무 오래 걸린다.

* 뭔가 설정을 잘못 했거나
* 데이터 셋의 질이 좋지 않거나
* 원래 오래 걸리거나...

일단은 기다려 보고 있다. 시간이 되는 한에서 여러 설정값으로 학습시켜 볼 생각이다.

추가: [2편](https://jueun-park.github.io/2018-07-12/yolo-custom-dataset-2)을 작성했다.

# 참고해볼 한국어 자료

[[YOLOv2]](https://kimbom.co.kr/yolov2/)

[YOLOv2 와 YOLO9000](https://m.blog.naver.com/PostView.nhn?blogId=sogangori&logNo=221011203855&proxyReferer=https%3A%2F%2Fwww.google.co.kr%2F)