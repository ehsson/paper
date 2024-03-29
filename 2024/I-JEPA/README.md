# I-JEPA

## Introduction
### self-supervised learning에는 2가지 접근방식이 있는데 
### 먼저 invariance-based 방식은 한 이미지의 2가지 이상의 view에 대해 positive한 embedding을 만들게 하도록 encoder를 최적화 함, 여기서 이미지는 rotation, color jitter, crop 등 여러 augmentation이 들어간다. 이러한 방법은 높은 sementic 수준의 representation을 학습하지만 특정 donwstream task에는 bias로 인하여 안좋은 영향을 끼칠 수 있다. 
### 다음으로 mask 제거 접근 방식은 픽셀이나 토큰 level에서 랜덤하게 masking 된 패치를 reconstruction 하여 representation을 학습한다. mask 제거 방식은 invariance 방식과 달리 사전 지식이 덜 필요하고 쉽게 일반화 될 수 있다.하지만 sementic 수준의 representation이 좋지 않으며 invariance 방식보다 성능이 좋지 않음.
### 따라서 해당 논문에서는 사전지식을 사용하지 않고 sementic 수준을 향상시키는 방법을 제시, 그리고 더 빠른 학습이 가능하다. abstract한 representation 공간에서 masking 된 부분을 예측

## Method
### 이미지에서 N개의 target patch로 target block을 만들고 이를 target encoder에 넣어 target representation을 얻음, 여기서 N은 보통 4를 사용.
### 또한 이미지에서 Context를 만드는데 이는 context block을 하나 만들어 block 부분을 제외하고 masking을 해준다. 그리고 위에서 얻은 target block 부분들도 masking을 해준다. 이렇게 얻은 context를 context encoder에 넣어 context representation을 얻게 됨.
### 마지막으로 이렇게 얻은 context에 target block 하나에 해당하는 mask token을 더해 predictor에 넣어 해당하는 block의 representation을 예측하고 이를 target에 해당하는 이미지의 block 부분의 representation(GT?)과 L2 loss를 구함