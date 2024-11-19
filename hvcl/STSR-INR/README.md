# STSR-INR: Spatiotemporalsuper-resolution for multivariate time-varying volumetric data viaimplicit neural representation
## Introduction

## Method
### Network architecture
#### SIREN and skip connection
CoordNet과 마찬가지로 고주파 신호를 잘 학습할 수 있도록 SIREN 네트워크를 사용, 학습을 안정화하고 더 깊은 layer를 만들 수 있도록
residual block을 사용함, residual block은 2개의 layer를 거친 feature와 1개의 layer를 거친 feature를 sum
#### Variable embedding
각 변수들을 서로 다른 개별 네트워크로 학습하는 것은 비효율적임, 그래서 여러 변수들의 공통 구조적 정보를 활용할 수 있도록 latent vector를
이용하여 하나의 네트워크로 여러 변수들을 학습할 수 있음
각 변수들을 최적화 가능한 latent vector로 임베딩하여 네트워크는 이 latent vector와 INR 매개변수를 공동으로 최적화 함
이 latent vector는 변수의 수가 증가해도 길이가 증가하지 않고, latent vector 간의 보간을 통해 새로운 결과를 유추할 수 있는 가능성을 가짐
#### Modulated structure
네트워크를 학습하기 위해서 2개의 network를 사용하는데, 먼저 좌표를 통해 출력값을 학습하는 synthesis network, 그리고 이 synthesis network에
변수 정보를 임베딩한 modulator network가 synthesis network가 학습하는데 관여를 함.
단순히 입력에 latent vector를 결합하면 feature map의 위상만 변경하지만, 모듈화를 하여 학습하게 되면 feature map의 위상, 주파수, 진폭을
모두 제어할 수 있는 장점이 있다고 설명함
modulator network는 latent vector를 입력으로 받아 Sine layer를 통과하고 synthesis network의 각 블록에 dot product를 통해 활성화를 조절
#### Variation auto-decoder
latent vector가 학습이 완료되면 INR은 훈련된 latent vector를 통해 변수 시퀀스를 재구성함, 이 때 latent vector는 고차원의 다변량 데이터를
차원 축소된 결과로 볼 수 있음.
이러한 변수 임베딩에 강한 정규화를 주는 variational auto-decoder를 활용하는데, 첫째로 latent space가 표준 정규 분포를 따라 더 압축된 형태이고,
둘째로 latent space의 연속적이고 확률적인 특성 덕분에 샘플링된 latent vector를 이용해 더 부드러운 결과를 만들 수 있음
VAE와 비슷하게 최적화 가능한 posterior distribution인 N(μ ,σ^2)를 통해 latent vector를 샘플링함
latent vector의 분포는 직접 최적화할 수 없기 때문에 reparameterization trick을 (Φ=μ+σ⊙ϵ)을 사용하여 최적화. 하지만 σ를 최적화하는 것은
학습의 안정성을 저하하여 μ만 최적화 함, inference 시에는 랜덤 요소를 제거하고 μ만을 사용하여 latent vector를 만듬
이런 latent vector를 샘플링하는 이유는 AD와 같은 이산적인 표현이 아닌 연속적인 표현이 되도록하기 위해 사용
#### Multi-head training
CNN기반 방법들보다 속도가 많이 느리기 때문에 볼륨을 블록 단위로 나눠 학습하는 multi-head 전략을 사용
### Optimization
학습 과정에서 INR 네트워크와 latent vector를 함께 최적화 함(L= LREC + λLKLD)
LKLD는 잠재 공간을 학습하기 위해 KLD loss를 사용, 안정적인 학습을 위해 σ를 최적화하지 않고 𝜇만 업데이트
## Results and discussion
### Dataset and network training
#### Network training
입력 좌표와 목표 값은 모두 [-1,1]로 정규화되고, modulator network와 synthesis network는 모두 5개의 residual block을 가짐
품질과 속도의 균형을 맞추기 위해 8개의 head로 학습
### Limitations
multi-head 학습을 하지만 아직은 CNN 기반 방법들보다 속도가 느림, 그리고 head가 많아질수록 성능이 떨어진다는 단점 존재
데이터셋의 여러 변수들이 유사한 외형?특징?을 공유할 경우 값들 간의 관계를 식별하기 어려움
CoordNet과 마찬가지로 정규화된 데이터로 학습하기 때문에 데이터를 원래 범위로 복원할 수 없음