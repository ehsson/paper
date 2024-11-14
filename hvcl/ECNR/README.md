# ECNR
## Introduction

## Method
### Unified Space-Time Approach
(x, y, z, t)로 이루어진 데이터를 다루기 위해 각 차원을 반으로 나누고, 시간과 공간 차원을 분리한다. block 크기, scale의 수는 미리 정해지고, 만약 볼륨 데이터 x의 길이를 x라 하고, block의 크기를 xb, scale을 s라 하면, x축에서의 block 개수는 x/(2^(i-1)*xb)가 됨.
각 scale에서는 block의 크기는 모두 같고, scale이 작아질수록 로드되는 block의 수는 많아짐(fine해짐)
### Multiscale Block Partitioning
여러 크기의 block들의 합으로 볼륨 데이터를 표현하는데, 볼륨을 가장 coarse한 scale(s)부터 fine한 scale로 encoding을 함, coarse한 scale에서 encoding한 것을 upsample하고 한 단계 fine해진 scale의 이미지와 비교하여 residual이 많은 부분만 해당 scale에서 encoding, 이를 반복한다.
모든 scale에서 block의 크기는 같고 scale마다 다른 해상도를 다루게 되는 것임. 또한 압축률을 높이기 위해, 각 유효 block에 학습 가능한 latent code를 같이 학습하여 각 블록들과 구분을 쉽게 함
### Block Assignment
각 block들을 개별적인 MLP에 할당을 해야 하는데, 한 MLP가 여러 block을 맡아야 함. 유사성을 가지는 block을 하나의 MLP로 학습해야 함.
1. 먼저 k-means clustering을 하여 먼저 block들을 clustering하고 이를 재 할당을 함
2. 각 block에 대해 각 cluster의 거리를 계산
3. 각 block에 대해 (cluster 거리 max) - (cluster 거리 min)을 구하여 저장
4. 3번의 값들을 내림차순으로 정렬해 우선순위 선정, 각 block을 균등한 cluster 크기를 유지하며 재할당
### Loss Function
각 block에 대해 GT와 MSE를 사용, 각 scale의 MLP들은 병렬로 학습됨
### Deep Compression of MLPs
Block-guided Pruning: 각 scale에서의 MLP에 대해 모든 MLP에서 중요하지 않은 뉴런을 pruning을 하여 압축률을 증가. 성능저하를 줄이기 위해 pruning된 네트워크를 낮은 learning rate로 finetuning을 함. 
또한 반복 pruning을 하여, 한 번에 pruning을 하는 게 아니라 여러 번에 걸쳐 조금씩 pruning을 함
### Boundary Artifact Mitigation
마지막으로 block들을 통합해야 하는데 경계 부분에서 artifact가 나타날 수 있음, 이를 해결하기 위해서 CNN을 사용하여 가장 작은 scale(s)에서 적용하여 artifact를 줄이지만 약간의 encoding 시간 증가와 압축률 감소를 가져옴