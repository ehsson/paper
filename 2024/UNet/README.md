# UNet

## Purpose
### AlexNet 등장 이후로 CNN 모델들은 크기가 점점 커짐, 그리고 지금까지의 모델들은 image classification을 위한 것이었음
### 따라서 해당 논문에서는 image segmentation을 연구함
### 기존의 방법에서는 image를 patch단위로 쪼개 픽셀별로 구분하는 방법이 있었지만, 이는 patch별로 연산하기 때문에 느리고, localiztion과 context 사이의 trade-off가 존재
### 따라서 UNet은 대칭적인 구조로 여러 resolution의 feature를 사용하고 이를 통해 local한 feature와 global한 feature를 모두 고려하는 모델을 만들게 됨
### 또한 FCN을 사용하지 않아 patch에서 얻은 정보만으로 classification을 수행하고, 적은 data로 인해 많은 augmentation을 사용함
### 또한 세포 사이를 구분하는 것에 대해 높은 가중치를 두는 loss function을 사용

## Network Architecture
### Downsampling 부분과 Upsampling 부분으로 나뉘는데
### Donwsampling에서는 2번의 3X3 conv, 그리고 ReLU와 2X2의 pooling이 적용되고 채널 수가 downsampling 될수록 2배씩 늘어남
### Upsampling에서는 2X2 up-conv, 그리고 3X3 conv와 ReLU가 오게 되고 upsample 될수록 채널 수가 2배 줄어들게 됨
### 마지막에서는 class 수를 맞추기 위해 1X1 conv가 오게 되고, 중요한 것은 upsample 단계에서 맞은 편의 downsample하는 feature를 concat하여 여러 feature를 사용

## Training
### loss function은 추후 게재
### Data Augmentation은 의료 영상에서는 색깔이 다양하지 않고 비슷한 색이다 보니 data augmenation을 적용해 많은 data를 확보하기 위함
### shift, rotation 외에 random-elastic deformation을 사용, 이 random-elastic deformation은 segmentation을 학습할 때 key concept로 작용한다고 함

