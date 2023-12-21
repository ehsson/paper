# HRNet

## Introduction
### 기존의 방법들은 low resolution feature에서 high resolution feature로 복구 하며 예측​을 함
### 이로써 upsampling하는 과정에서 정보 손실이 일어날 수 있음
### HRNet은 각 resolution feature를 병렬로 연결하며 해결하려 함

## Parallel Multi Resolution Convolutions
### HRNet은 각 resolution feature를 병렬로 연결하여 feature를 유지

## Repeated Multi Resolution Fusions
### 병렬로 연결된 feature들을 downsample, upsample하여 각각의 resolution feature들에게 영향을 미침(상호보완 해주는 역할)