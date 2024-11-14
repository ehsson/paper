###Introduction

###Method
##Overview
#KD-INR은 두 단계로 이루어져 있음. 각 시간 단계에서 INR을 통해 압축이 되는데, 데이터의 압축 표현을 학습하며 매개변수를 줄이기 위해
bottleneck layer를 활용, 그리고 더 중요한 부분의 샘플링을 많이하기 위해 관심 특징 보존 기반 샘플링을 사용함. 다음으로 각 시간 단계에서
최적화된 모델을 수집하여, 이들 모델로부터 knowledge distillation을 하여 하나의 모델에 집계하여 시간 단계 간의 일관성을 갖게 함
##Spatial compression
#encoder-decoder 구조를 사용하였고, residual block을 구성하기 위해서 bottleneck layer를 도입함. residual layer는 은닉층에 많은 수의 뉴런을
사용해야 하여 큰 모델 크기를 초래하고, 여러 residual block을 사용할 경우 최적화가 어려워저 local minimum에 빠질 수 있음
#따라서 bottleneck layer를 사용하고, 입력 차원을 줄이는 layer, 줄어든 차원에서 연산을 수행하는 layer, 입력 차원을 원래 크기로 복원하는 
layer 3개의 layer로 구성됨. 이는 매개변수 수를 줄이고 차원을 감소시켜 네트워크가 중요한 특징을 포착하게 함
#encoder는 처음엔 m개의 뉴런, 다음은 2m, 다음은 4m개의 뉴런을 사용하고 각 linear layer 다음에는 sin activation이 사용됨, 그리고 마지막에
bottleneck layer를 두어 차원을 유지함. decoder에서 4m의 입력차원을 출력값 1로 감소시킴
##Entropy-based Adaptive Sampling
#전체 볼륨을 블록 단위로 쪼개어 샘플링하고 학습하게 되는데, 무작위로 블록을 샘플링하게 되면 중요한 부분에 집중할 수 없음
#따라서 entropy 기반 샘플링을 도입.
#전체 볼륨을 서브 볼륨(블록)으로 나누고, 각 블록을 bin으로 이산화하여 이산 확률로 변환. 
#그래서 (해당 bin에 속한 포인트 수)/(블록 내 전체 포인트 수)로 확률을 계산하고, 이를 통해 entropy를 계산. entropy는 각 블록의 정보의
풍부함을 나타냄. 높은 엔트로피는 높은 수준의 무질서와 예측 불가능한 패턴을 의미. 
#모든 블록의 entropy를 계산한 후, (각 블록의 entropy)/(전체 블록의 entropy 합) + ϵ을 계산하여 각 블록의 샘플링 확률을 구함
##Model Aggregation
#위의 블록 단위 INR을 통해 각 시간 단계마다 학습된 모델을 통해 knowlege distillation을 하여 하나의 모델에 집계하여 모델 저장 비용을 줄임
#교사 모델들의 특징과 예측을 모두 맞추는 distillation loss를 사용할 수도 있지만, 교사-학생 모데의 구조 차이로 인해 학습 붕괴로 이어질 수 
있음. 따라서 교사 모델의 예측과 비슷하도록 만드는 것을 선택, 이로인해 뉴런 수나 layer 수 등 구조가 서로 달라도 가능하여 더 많은 유연성