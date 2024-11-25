# Direct Voxel Grid Optimization (DVGO)

## Abstract

> First, we introduce the **post-activation** interpolation on voxel density, which is capable of producing **sharp surfaces** in lower grid resolution.

적은 수의 voxel을 사용하더라도 (=lower grid resolution) post-activation을 적용하여 3D 이미지 표면을 더 선명하게 보이게 함

> Second, direct voxel density optimization is prone to **suboptimal geometry solutions**, so we robustify the optimization process by imposing several **prior**.

- direct

  - 추가 단계나 조정 없이 복셀 데이터에서 바로 최적화 프로세스가 수행됨
  - ex) 부품이 제대로 정렬되었는지 확인하지 않고 나사만 돌려 자동차 엔진을 고치려는 것
  - 이 방법은 좋은 결과를 얻기 위해 필요한 모든 세부 사항을 고려하지 않을 수 있기 때문에 문제가 발생할 수 있음

- suboptimal geometry solutions

  - 최적화 과정에서 잘못된 또는 비효율적인 장면 기하학적 구조가 생성되는 경우
  - **Cloudy geometry**: 장면의 실제 표면 대신, 공간 전체에 고밀도 "구름" 구조가 생성됨.
  - **Near-camera bias**: 카메라에 가까운 영역에 불필요한 밀도가 집중되어, 잘못된 장면 구조가 생성됨.
  - **Overfitting to single views**: 일부 시점에서만 맞는 구조가 생성되고, 다른 시점에서는 부정확한 결과를 초래.

- optimal geometry
  - 여러 시점(view)에서 관측된 이미지 데이터를 잘 설명하는 밀도 분포를 나타냄

## 1. Introduction

## 2. Related Wrok

## 3. Preliminaries

## 4. Post-activated density voxel grid

### Post-activation

삼선형 보간(interp) → 소프트플러스(softplus) → 알파 함수(alpha)

- 보간 후 비선형 변환을 마지막에 수행하므로, 결정 경계가 더 날카롭게 형성될 수 있음
- 이는 세부 표면을 적은 grid cell로 표현하는 데 유리함

### SoftPlus 활성화 함수

- 비선형 함수
- 입력값 x에 따라 부드럽게 증가하며 항상 양수를 반환
- Shifted Softplus(softplus(𝑥+𝑏))를 활용하면 초기 밀도 값을 원하는 수준으로 조정할 수 있어, 초기 학습 단계에서의 안정성을 더욱 강화
  $$
  softplus(x)=log(1+exp(x))
  $$

**사용이유**

- 밀도는 빛이 흡수되거나 산란되는 정도를 나타내므로, 물리적으로 음수 값이 될 수 없음
- 샘플링 과정에서 얻은 raw 밀도 값이 음수가 될 가능성을 방지하기 위해서 사용
- 비선형성 추가: 학습 과정에서 밀도의 표현력을 향상시키고, 모델의 물리적 타당성과 학습 안정성을 동시에 보장하기 위함
  - 이는 특히 Volume Rendering 과정에서 밀도 값이 중요한 역할을 하기 때문에 필요

> **raw 밀도 값**
>
> - voxel grid에 저장된 학습 가능 파라미터
> - 학습 중 손실 함수에 의해 업데이트됨
> - 물리적인 밀도 값을 직접적으로 표현하는 것이 아니라, 활성화 함수(예: Softplus)를 통해 물리적 의미를 가지는 밀도 값으로 변환

> **raw 밀도 값이 음수가 될 가능성**
>
> - 보간의 특성: raw 밀도 값은 트릴리니어 보간(trilinear interpolation)을 통해 8개의 인접 voxel 값을 가중치로 선형 결합하므로, 만약 인접 voxel 중 일부 값이 음수라면, 보간 결과 역시 음수 값이 될 가능성이 높음
> - 손실 함수의 최적화 과정에서 음수로 조정될 수 있음
>   - 학습 중 사용되는 **raw 값**이 반드시 물리적 의미를 반영하지 않기 때문

## 5. Fast and direct voxel grid optimization

### 5.1. Coarse geometry searching

### Prior 1: low-density initialization

### Prior 2: view-count-based learning rate

- 실제 장면을 캡처할 때 일부 voxel은 너무 적은 training view에서만 보일 수 있음
- 이를 위해, 𝑉(density)(𝑐)의 각 격자점(grid point)에 대해 서로 다른 학습률(learning rates)을 설정
- n_j/n_max = 해당 점이 보이는 학습 시점의 개수 / 모든 격자점 중 최대 시점 수
