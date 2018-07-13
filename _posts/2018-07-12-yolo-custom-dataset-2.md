---
layout: post
title: 커스텀 데이터 셋으로 Yolo 써 보기 2
published: True
---



YOLO v3와 커스텀 데이터셋을 이용해서 기계를 학습시키고 웹캠을 통해 실시간으로 표지판을 인식하는 데모를 해 보았다. Custom dataset을 만드는 과정은 [커스텀 데이터 셋으로 Yolo 써 보기](https://jueun-park.github.io/2018-07-12/yolo-custom-dataset)에 정리해 두었다.



# 학습!

## Dataset

7개 각 클래스당 10장의 사진을 준비해 총 70장의 이미지를 data로 사용했다.

```
tags:
crosswalk
moving_obstacle
narrow_road
parking
static_obstacle
s_curve
u_turn
```



## 그래픽카드와 CUDA

학습용 머신은 우리 동아리 컴퓨터의 GeForce GTX 1070을 사용하려 했으나, `batch=64`일 때 `subdivision=32`로 해야만 out of memory 에러가 안 나서 그렇게 했더니 학습 속도가 너무 느렸다. 그래서 작업실에 있는 시뮬레이터 컴퓨터의 GTX 1080도 함께 사용했다. 결국은 시뮬레이터를 이용한 모델만 학습이 되어서(loss가 수렴) 그것의 결과를 데모에 이용했다.

* GeForce GTX 1080
* CUDA: 9.1
* cuDNN: 7.0



## Configuration

`yolo-roadsign.cfg`

```
batch=64
subdivision=16
width=416
height=416
```

 `yolov3.cfg`를 복사한 뒤 필요한 부분만 확인하고 바꿔서 사용했다.



## 학습 완료?

새벽에 학습을 돌려 두고 자고 와서 봤더니 6500번 이터레이션을 넘어선 뒤에 컴퓨터의 하드 용량 부족으로 학습 프로그램이 저절로 꺼져 있었다. 그래서 loss 그래프를 확인할 수 없었다(...). `backup` 디렉토리에 저장된 `.weights`파일 중 적당히 6000번째에 저장된 파일을 이용해 데모를 해 봤다.



# 데모와 결론

데모도 학습에 이용한 컴퓨터와 같은 컴퓨터에서 실행했다. [데모 영상](https://www.facebook.com/elpion19/posts/1760625427385820?notif_id=1531383678736860&notif_t=feedback_reaction_generic)은 개인 페이스북에 올려 두었다. 실내에서만 테스트해 보았다. 필요하다면 실외에서도 테스트해봐야 할 것 같다.

확실히 실시간으로 검출되는 것이 아주 매력적이었다. 그러나 영상에서도 볼 수 있듯이 일부 정확하지 않은 검출 결과가 있다. 이는 데이터의 개수와 다양성을 늘리고 loss를 확인하며 학습을 적절히 시키면 나아질 것으로 예상한다.

---

# (추가)

사실 이 이틀만에 끝난 작은 프로젝트는 [자율차 대회](https://jueun-park.github.io/2018-07-08/postscript-autonomous-car)가 끝난 뒤, 표지판 인식 프로그램에서 아쉬움이 남아 [대회 때 사용한 방법 말고 다른 방법론](https://jueun-park.github.io/2018-07-11/cnn-models)으로 학습시켜보면 어떨까 하는 생각에 갑자기 해 본 것이다. 아래 내용은 이 데모를 실제로 자율차 프로그램에서 사용할 때 어디를 건드려야 할지 궁금해서 코드를 따라가 본 뒤 기록했던 내용이다. (깃헙 레포 리드미에 작성해 뒀었는데 프라이빗 레포라서 여기에다 복붙해둔다... 지금 보니까 같이 한 팀원이 몇 번째 줄인지에 대한 내용까지 추가해뒀다)

## 검출 데이터는 어디에 있지?

`demo.c`가 실행될 때 라벨이 붙는다. 코드를 따라가보면 `image.c`의 `void draw_detections_cv_v3`에 detection 하는 코드가 있는 것을 볼 수 있다. `image.c`의 547번 줄과 548번 줄 사이에 다음과 같은 코드를 넣으면 결과를 확인할 수 있다.

`printf("Detected sign: %s\n", labelstr);`

여기에서 통신 코드를 짜서 다른 프로그램에서 쓸 수 있지 않을까?