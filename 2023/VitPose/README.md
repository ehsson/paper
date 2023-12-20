# VitPose

## Purpose
### 기존의 transformer 기반의 pose estimation은 CNN을 backbone으로 사용하여 너무 복잡한 구조, 안에서 무슨 일이 일어나는지 알기 힘듬
### 해당 논문에서는 단순한 plain vision transformer가 pose estimation에서 얼마나 잘 수행 되는지 확인

## Simplicity
### 네트워크 구조가 간단함

## Scalability
### 모델 구조의 큰 변경 없이 레이어 수, 채널 수 등의 변경으로 다양한 크기의 모델 구현 가능

## Flexibility
### 해상도의 유연성, Attention 유연성, Fine tuning 유연성, Task에 대한 유연성

## Transferability
### 큰 모델을 작은 모델에 모방학습이 가능