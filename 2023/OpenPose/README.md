# OpenPose

## Introduction
### Bottom-Up approach: 먼저 이미지를 입력 받아 사람들의 joint(관절)들을 predict하고, 그 다음 각 instance(사람) 별로 나누는 방법
### Multi Person을 real time으로 inference 가능하게 한 점에서 주목

## Part Confidence Map
### 각 관절의 위치를 detect하는 작업
### 픽셀마다 관절이 있을만한 부분을 heatmap형태로 prediction

## PAFs(Part Affinity Fields)
### 서로 연결되는 두 관절 사이의 관계를 추정
### 두 관절 연결선 상에 있는 픽셀에 vector field를 할당, 이를 통해 관계를 나타냄