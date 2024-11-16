# DreamFusion: TEXT-TO-3D USING 2D DIFFUSION
## Introduction
해당 논문은 3D 데이터 없이 2D diffusion 모델을 통해 3D object를 합성할 수 있는 기술을 소개

diffusion을 다른 modality에 적용하는 것은 성공적이지만 큰 규모의 3D 데이터셋을 필요로 한다. 

2D 데이터셋은 많이 존재하므로 이를 통해서 text-to-3D 생성 모델을 제시, 주로 NeRF를 다룸

## Score Distillation Sampling
diffusion 모델은 보통 pixel 단위에서 이미지를 sampling 하지만 해당 논문은 pixel 단위에는 관심이 없음, 대신 랜덤한 각도에서 좋은 이미지처럼 보이는 3D 생성 모델을 만드는 것에 관심이 있음, 보통 이러한 모델은 DIP(INR의 하위 개념)를 사용함

저자들은 Diffusion Training Loss를 사용해보았으나 좋지 않은 성능을 내어 Diffusion Training Loss에서 UNet Jacobian 항을 제거한 loss를
사용하였음, 이것이 DIP를 최적화하는 데에 더 효율적이라고 설명함 설명하자면 DIP가 만들어낸 렌더링 이미지에 무작위 시간 단계의 노이즈를 더하고, 이와 시간 t, text y를 입력으로 한 diffusion 모델의 예측된 노이즈와 실제 노이즈를 비교하여 DIP를 학습하는 방식, diffusion 모델이 학습 방향을 제시하여 diffusion은 stopgrad

## DreamFusion Algorithm
diffusion 모델로는 64x64를 생성하는 Imagen 모델을 사용, 그리고 NeRF는 mip-NeRF 360을 사용함
### Neural Rendering of a 3D Model
Shading에서는, 기존 NeRF 모델은 카메라의 각도나 광선에 따라 rendering되는 색이 달라지는데, 해당 논문의 rendering은 카메라나 광선에 상관없이 물체 그 자체의 색깔을 rendering함, 각 점에 대한 RGB albedo를 사용

Scene Structure에서는 고정된 구형의 형태에서만 NeRF의 scene을 query하고, ray color를 alpha값을 사용해 background color위에 물체의 밀도에 따라 색을 합성하는 방식을 사용

Geometry regularizers에서는 불필요한 빈 공간을 채우는 것을 방지하기 위해 regularization penalty를 포함해 학습하였음, 이는 textureless shading을 할 때 중요함
### Text-to-3D Synthesis
1. 랜덤하게 카메라와 light를 샘플
2. 64x64 크기의 NeRF의 이미지를 rendering, 조명이 적용된 render, 텍스쳐 없는 render, 음영 없는 albedo render 중 무작위로 rendering
3. SDS loss 계산
4. optimizer를 사용해 NeRF parameter 업데이트