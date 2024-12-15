# Adaptively Placed Multi-Grid Scene Representation Networks for Large-Scale Data Visualization

## Introduction
기존에 존재하는 SRNs은 네트워크 리소스를 더 중요한 영역에 적응적으로 배치하지 못한다는 단점이 존재. 그래서 최근 방법들은 Octree, KD-tree 등의 방법을 사용하지만 추가 저장 공간이나 계산 비용이 많이 필요함

또한 대규모 데이터를 학습하는 데에 단일 GPU를 사용하여 학습하는 데, 이는 multi GPU 환경을 제대로 활용하지 못 함

따라서 해당 논문에서는 학습 중 오류가 높은 볼륨 영역에 네트워크 매개변수를 집중시키는 adaptive feature grid를 사용, 그리고 대규모 데이터를 빠르게 학습하기 위해 여러 SRNs을 병렬로 학습시키는 도메인 분할 전략을 사용

## Method
해당 연구에서는 변환행렬로 구현되는 다수의 공간 adaptive feature grid를 사용하고, 이후 얕은 MLP를 이용하여 출력 값을 생성
### APMGSRN architecture
#### Adaptively placed multi-grid encoder
##### Transform to local space
feature grid는 (M, C, D, H, W)의 shape으로 M개의 feature를 가진 C개의 채널을 가진 형태로 구현됨

transform matrix는 4x4의 matrix로 구현되고, global 좌표를 local 좌표로 매핑하는 역할
##### Encoding
이후 변환된 local 좌표를 이용하여 해당 격자 내에서 쿼리된 좌표에 대한 trilinear interpolation을 수행, 만약 feature grid 범위 밖이라면 0을 반환

여러 feature grid의 보간된 결과를 concat하여 최종 encoding 결과를 만듬
#### Decoding
decoder는 bias가 없는 2개의 얕은 MLP이고, 각 layer는 64개의 뉴런, 마지막 layer를 제외하고는 ReLU 활성화 함수를 사용

### Learning Feature Grid Transformation Matrices
단순히 reconstruction loss로는 변환행렬을 학습하는 것이 힘듦. 이는 쿼리 포인트가 feature grid 밖에 있다면 반환 값이 0이 되므로 기울기를 제공할 수 없어, grid 외부의 오류가 높은 위치로 grid를 끌어오는 효과를 얻지 못 함

따라서 변환행렬의 학습을 위해 현재의 특징 밀도 ρ와, target 특징 밀도 ρ* 사이의 차이를 측정하는 미분 가능한 방법인 "feature density loss"를 사용, 여기서 feature density란 grid의 특정 공간에 존재하는 평균적인 feature 수를 의미

오류가 상대적으로 높은 부분은 현재 feature density를 증가시키고, 오류가 낮은 부분은 감소시키도록 현재 feature density를 왜곡하는 "target feature density를 도출"하는 방법
#### Feature density
feature density의 식은 ρ(x) = det(Gi0:3,0:3)을 사용함, 이는 global point x에 대해 얼마나 많은 feature grid가 겹치는가를 설명하고, 또한 작은 feature grid는 더 밀집되므로 크기가 작을수록 값이 큰 행렬식을 사용하여 구현

하지만 이럴 경우 어떤 point가 grid 외부에 있다면 반환값이 0이 되어 변환행렬 G를 변경할 기울기가 없는데, 이를 해결하기 위해 flat-top gaussian을 사용하여 모든 부분에서 미분이 가능하도록 함
#### Target density
해당 density의 목표는 오류가 높은 부분에서 ρ(x)를 증가시키고 낮은 곳에서 ρ(x)를 감소시켜주는 역할
target density는 ρ∗(x) = exp(h/(h(x)+ϵ)⋅log(ρscaled(x)+ϵ))를 사용함

여기서 h(x)는 x에서의 모델의 오류값, h는 모델의 평균 오류 값, ρscaled(x)는 현재 feature density ρ(x)를 0~1로 scaling

해당 식의 의미는 오류가 평균 수준일 경우 target feature density는 ρscaled와 동일하게 유지되고, log 스케일은 오류가 높은 부분에서 feature density를 증가시키고 낮은 부분에서 감소시키는 역할을 함
#### Feature density loss function
feature density loss(Ldensity)는 최종적으로 KL-divergence 형태가 되어 target feature density를 따라 각 grid의 위치와 크기를 조정

### APMGSRN training
#### Training routine
Lrec을 사용하여 feature grid와 decoder 파라미터를 업데이트, Ldensity를 사용하여 변환행렬을 업데이트, Ldensity가 변환행렬을 업데이트하는 유일한 loss
##### Delayed start
학습 초기에는 무작위로 초기화되어 있어 어느 부분이 오류가 많고 적은지 알 수 없음, 따라서 변환행렬 학습을 지연시켜 500iteration동안 고정되며 이후부터 변환행렬을 학습
##### Early stopping
변환행렬과 네트워크 매개변수 학습이 동시에 이루어지면 모델이 움직이는 목표를 따라가야한다는 과제를 안게 됨, 따라서 1000iteration 동안 행렬의 이동 평균이 0.01%이상 감소하지 않으면 early stopping, 그리고 총 학습의 80% 이후에도 early stopping 기준을 만족하지 못하면 early stopping

실험한 hyperparameter로 50000 iteration 학습에 최대 4분, 최소 40초 소요
#### Initialization
변환행렬의 초기화는 대각 요소(행렬의 크기)는 정규분포 N(1,0.05)에서 샘플링, 나머지 요소는 N(0,0.05)에서 샘플링하여 거의 전범위를 다루도록 함

그리고 feature grid도 U(−0.0001,0.0001)에서 초기화하여 무작위성을 부여

### Domain Decomposition Scene Representation
병렬 학습 방식을 사용, 볼륨을 블록으로 나누고 각 모델 별로 서로 다른 블록을 학습
#### Data partitioning
##### Ghost cell
인접한 블록 간의 경계 아티팩트를 줄이기 위해 네트워크 간에 겹치는 ghost cell을 사용함, 각 네트워크의 학습 범위가 일정 수의 ghost cell만큼 확장됨을 의미
#### Domain decomposition training
모델을 학습시키기 위해 도메인을 분할하고 사용가능한 GPU에서 모델을 병렬로 학습시킴

사용 가능한 gpu에 할당될 작업 목록을 생성, gpu가 학습을 마치면 gpu는 사용 가능한 gpu목록으로 돌아가 다음 모델이 학습할 수 있을때가지 대기
#### Domain decomposition inference
inference 시에 쿼리 point에 해당하는 모델을 찾기 위해 spatial hashing function을 사용