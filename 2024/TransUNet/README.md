# TransUNet

## Introduction
#### 기존의 UNet은 convlution을 이용하여 medical image 분야에서 좋은 성능을 보여주었다. 하지만 convolution 기반의 접근방식은 장거리 관계를 모델링 하는데에는 제한이 있음. 반면 NLP분야에서 크게 사용되는 Transformer는 시퀀스 간 예측을 위해 설계되었고 convolution 없이 self-attention 기반으로 모델링 하여 전역 context 특징을 잡고, downstream task에서 좋은 성능을 보여줌.
#### 해당 논문에서는 Transformer를 사용한 image segmentation을 제시하였는데, 우선 그냥 단순히 Transformer를 사용한 것은 좋은 성능이 나오지 못하였다고 한다. 이는 transformer가 입력을 1차원으로 시퀀스로 처리하고 모든 단계에서 전역적으로 context를 처리하기 때문이고 이로써 local한 정보가 부족하다고 설명하였다. 반면 CNN은 세부 공간적 특징을 잘 추출하여 저수준의 단서를 추출한다고 설명했다. 
#### 따라서 해당 논문에서는 local한 정보를 얻을 수 있는 CNN과 전역적인 특징을 얻을 수 있는 Transformer를 결합하고 UNet의 구조에서 영감을 받은 TransUNet을 제시하였다.

## Method
### Transformer as Encoder
#### Image Sequentialization: 먼저 입력 이미지를 patch sequence로 나누어 token화를 한다. 각 patch의 크기는 P X P 크기이다.
#### Patch Embedding: linear projection을 사용해 벡터화된 patch들을 D차원 embedding 공간으로 mapping하고 위치 정보를 위해 positional encoding을 해준다. 그런 다음 patch들은 Multihead Self-Attention(MSA)와 Multi-Layer Perceptron(MLP)로 구성된 Transformer에 입력된다.
### TransUNet
#### segmentation을 위해 transformer에서 encoding된 representation을 최대 해상도로 upsampling을 수행한다, 즉 HW/P^2에서 H/P X W/P로 변환(sequence -> image). 그런 다음 class수에 맞추기 위해 1X1 conv를 사용하고 최종 이미지 크기로 복구하기 위해 bilinear upsampling을 한다. Transformer와 단순 upsampling을 결합하는 것도 좋지만 H/P X W/P가 원본 이미지 크기에 비해 작아 세부정보의 손실이 있을 수 있기 때문에 encoder로 Hybrid CNN-Transfomer와 Cascaded Upsampler를 사용함
#### CNN-Transformer Hybrid as Encoder: 순수 Transfomer를 사용하는 대신 CNN을 먼저 사용하여 특성 맵을 생성하고 추출된 특성 맵을 patch화 시키게 된다. 이러한 이유는 decoding 중에 중간 크기의 고해상도 CNN 특성 맵을 활용할 수 있고, Hybrid CNN-Transformer가 순수 Transformer보다 더 좋은 성능을 내는 것을 확인함
#### Cascaded Upsampler: 여러 upsampling 단계로 구성된 cascaded upsampler를 제안하였고, Hybrid encoder와 함께 Cascaded Upsampler가 U자형 구조를 형성해 skip connection을 통해 다양한 해상도에서의 feature를 활용할 수 있다.