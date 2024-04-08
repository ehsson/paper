# TransUNet

## Introduction
#### 기존의 UNet은 convlution을 이용하여 medical image 분야에서 좋은 성능을 보여주었다. 하지만 convolution 기반의 접근방식은 장거리 관계를 모델링 하는데에는 제한이 있음. 반면 NLP분야에서 크게 사용되는 Transformer는 시퀀스 간 예측을 위해 설계되었고 convolution 없이 self-attention 기반으로 모델링 하여 전역 context 특징을 잡고, downstream task에서 좋은 성능을 보여줌.
#### 해당 논문에서는 Transformer를 사용한 image segmentation을 제시하였는데, 우선 그냥 단순히 Transformer를 사용한 것은 좋은 성능이 나오지 못하였다고 한다. 이는 transformer가 입력을 1차원으로 시퀀스로 처리하고 모든 단계에서 전역적으로 context를 처리하기 때문이고 이로써 local한 정보가 부족하다고 설명하였다. 반면 CNN은 세부 공간적 특징을 잘 추출하여 저수준의 단서를 추출한다고 설명했다. 
#### 따라서 해당 논문에서는 local한 정보를 얻을 수 있는 CNN과 전역적인 특징을 얻을 수 있는 Transformer를 결합하고 UNet의 구조에서 영감을 받은 TransUNet을 제시하였다.

## Method
### Transformer as Encoder
#### 