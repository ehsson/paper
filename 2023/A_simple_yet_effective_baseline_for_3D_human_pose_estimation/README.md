# A simple yet effective baseline for 3d human pose estimation

## Introduction
### 3차원 GT 데이터의 부족은 3d human pose estimation에서 어려움을 낳음
### 해당 논문은 2차원 이미지에서 바로 3차원 관절 좌표를 얻는 것이 아닌, 2차원 관절 좌표를 입력받아 3차원 관절 좌표를 prediction

## Network Design
### ReLU layer: 선형 데이터에 비선형성을 주어 더 복잡한 패턴이나 특징을 학습하게 함

### Batch Normalization: batch normalization을 통해 불균형적인 포즈 데이터 scale을 조정하여 학습을 안정화, 가속화

### Residual Connection: 당시 깊은 네트워크에서 학습을 용이하게 해주는 용도, 성능향상, 학습시간 단축

### Dropout: Overfitting을 방지하기 위한 장치, 무작위로 노드를 선택하여 해당 노드를 무시하고 학습을 진행

## Data Preprocessing
### Camera coordinate: 회전이나 평행이동 같은 변형을 가하지 않은 input을 임의의 좌표계에서 단순히 추론하는 것은 비현실적,
### 따라서 카메라 좌표계를 사용하여 3d관절을 추론함으로써 특정한 좌표계에 overfitting되는 것을 방지하고 더 많은 훈련 데이터 확보

### 2d detection: 모델이 2d pose를 기반으로 추론을 하므로 2d detector에 대한 선정, 당시 SOTA이던 Stacked Hourglass를 선정