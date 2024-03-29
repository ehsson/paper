# Masked Autoencoder

## Introduction
### 딥러닝은 계속해서 용량과 크기가 증가하며 성능이 향상이 되었지만 이제는 너무 크기가 커졌고 overfitting이 많이 일어나고 비용이 많이 드는 label data가 많이 필요하다. 이런 상황에서 NLP분야에서는 GPT의 auto regressing과 BERT의 masked autoencoding이 주목을 받게 되었다. 이러한 아이디어를 비전분야에도 적용을 하려했지만 NLP에서의 Attention과 비전의 Convolution은 모델의 구조가 근본적으로 달랐고, Text가 이미지보다 정보의 밀도가 더 높고, autoencoder가 NLP와 비전에서 수행되는 역할이 달라 비전분야에서는 NLP의 진전을 따라가지 못하였다.
### 하지만 최근에는 ViT의 등장으로 이전의 Convolution 방법과는 상황이 많이 달라졌고, 이를 기반으로 비전분야에도 적용되는 Masked Autoencoder(MAE)를 해당 논문에서는 제안을 하였다. MAE는 입력 이미지에서의 임의의 patch를 masking하고 pixel 수준에서 누락된 patch들을 reconstruction하게 됨. 그리고 비대칭 encoder-decoder 구조를 사용하고 가벼운 decoder를 사용하여 누락된 patch를 재구성하게 되어 pre-training을 더 빠르게 하고, 75%의 masking을 적용하여 더 적은 메모리를 사용하여 MAE를 큰 규모의 모델에 쉽게 확장을 가능하게 하였다.

## Method
### MAE는 autoencoder와 마찬가지로 encoder가 이미지를 latent representation으로 만들고 decoder가 이를 이용하여 이미지를 재구성하였다. 그리고 비대칭 encoder-decoder 구조를 사용, 더 가벼운 decoder를 사용을 하였다.
### Masking: ViT와 같이 patch로 나누고 몇몇 patch들을 sampling하여 sampling 되지 않은 patch들을 masking하는데, 여기서 random sampling을 사용함. random sampling은 중복성을 제거하여 쉽게 문제를 해결할 수 없게 하고 bias를 방지해준다.
### MAE Encoder: 위에서 masking 되지 않은 patch(전체의 25%)들만을 encoder에 넣어 latent representation을 얻음
### MAE Decoder: Decoder의 입력은 encoder에서 얻은 latent representation patch들과 positional embedding을 추가한 masking된 patch들이다. MAE Decoder는 pre-training에만 사용되어 이미지 reconstruction을 하고, 독립적으로 유연하게 사용할 수 있다. 그리고 decoder는 작은 decoder를 사용하여 학습 시간이 크게 줄어듬.
### Reconstruction Traget: decoder의 출력 결과는 각 pixel 값의 vector이고, decoder의 마지막 layer는 입력 이미지에 맞게 reshape됨. 그리고 pixel 수준에서 reconstruction된 pixel들(masking되었던 pixel)에 대해서만 L2 loss를 구하게 된다.