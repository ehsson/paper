# SimCLR

## Introduction
#### supervise 없이 representation을 학습하는 방식은 2가지로 나뉘었는데 먼저 Generative 접근 방식은 pixel-level의 generation은 cost가 크고 representation에 필요하지 않음, 두번째로 discriminative 접근 방식은 heuristic을 사용하여 representation의 일반성을 잃을 수 있음
#### 해당 논문에서는 contrastive learning을 적용한 SimCLR라는 간단한 framework를 제시한다. 해당 논문에서는 다음과 같은 주요 point를 제시하는데, 먼저 많고 강한 data augmentation은 대조적 예측을 하는 작업에서 중요한 역할을 하고, representation과 contrastive loss 사이에 non-linear transformation을 도입하면 학습된 representation의 품질이 향상되고, Contrastive cross entropy loss 사용 시, normalized embedding과 temperature parameter가 중요한 역할을 하고, 마지막으로 constrastive learning은 supervised learning보다 큰 batch size와 긴 training 시간이 필요하고 더 큰 네트워크에서 더 잘 학습한다고 하였다.

## Method
### The Contrastive Learning Framework
#### Data Augmentation module: SimCLR의 framework는 4가지로 구성되는데 먼저 Data Augmentation module은 이미지에 2개의 augmentation을 적용하여 이 둘을 positive pair로 정의하고, 적용되는 augmentation은 크게 3종류인데 random crop, random color distortion, random gaussian blur를 적용함
#### Base encoder: augment된 이미지로부터 representation vector를 얻는 network로 여러 network를 사용할 수 있는데, 해당 논문에서는 ResNet을 사용함
#### Projection Head: representation vector를 contrastive loss가 적용되는 공간으로 mapping하는 역할
#### Contrastive loss: N개의 mini batch에 augmentation을 적용하여 이미지 2N개를 얻고 같은 이미지에서 얻어진 augmented 이미지는 positive pair, 아닌 것은 negative pair로 간주

### Training with Large Batch Size
#### 이전의 다른 연구에서 사용된 memory bank를 사용하지 않고 batch size를 256~8192로 사용했고, 큰 batch size에서 안정적인 학습을 위해 LARS optimizer를 사용함, 또한 Global Batch Normalization을 사용함

### Evaluation Protocol
#### Datasets and Metric: ImageNet과 추가적인 실험으로 CIFAR-10을 사용, Metric으로는 Linear evaluation protocol을 사용함

## Data Augmentation for Contrastive Representation Learning
#### 해당 논문에서는 random crop과 같은 augmentation을 통해 Network architecture 상에서 receptive field를 제한하여 global-to-local view prediction 수행, Context aggregattion network로 image patch에 대한 neighboring view prediction 수행할 수 있다고 함.
### Composition of data augmentation operations is crucial for learning good representations
#### Spatial/geometric transformation으로는 crop, resize, rotation, cutout을 사용하고, Appearance transformation으로는 color distortion, gaussian blur, sobel filtering을 사용하였다.
#### 크기가 다른 이미지로 인해 crop과 resize를 하고 다른 augmentation을 적용했는데 하나의 aug를 적용했을 때의 결과가 좋지 않았고, 해당 논문에서는 crop과 color distortion을 같이 사용했을 때 성능이 가장 좋았다고 설명함

### Contrastive learning needs stronger data augmentation than supervised learning
#### color distortion의 강도를 높일수록 SimCLR로 학습된 model의 성능이 좋아졌지만, supervised 방식에서는 성능이 더 안좋아짐, 또한 autoaugmentation을 적용했을 때도 성능이 저하됨

## Architectures for Encoder and Head
### Unsupervised contrastive learning benefits (more) from bigger models
#### supervised model과 마찬가지로 model의 width와 depth가 클수록 성능이 좋아졌고, supervised에 비해 크기가 커질수록 contrastive learning의 성능향상 폭이 큰 것을 확인

### A nonlinear projection head improves the representation quality of the layer before it
#### projection head로 identity mapping, linear projection, non-linear projection을 비교하였는데 non-linear projection의 성능이 가장 좋았다고 함, 해당 논문은 contrastive loss에 의해 정보를 잃어버릴 수 있기 때문에 representation 이후 비선형 projection을 사용하는 것이 중요하다는 가설을 제시하고 실험함

## Loss Functions and Batch Size
### Normalized cross entropy loss with adjustable temperature works better than alternatives Permalink
#### 추후 작성

### Contrastive learning benefits (more) from larger batch sizes and longer training
#### batch size와 training 시간에 따른 성능 변화도 실험하였는데 batch size가 클수록, training 시간이 길수록 더 좋은 성능을 내는데 이같은 이유는 batch size가 클수록 negative sample의 양이 많아지고, training을 오래할수록 마찬가지로 negative sample이 많아져 더 좋은 성능을 낸다고 함