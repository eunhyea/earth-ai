# R_VGAE

## 모델 개요
이 모델은 **Heterogeneous Graph**에서 노드 간의 **잠재적인 링크**를 예측하고, 예측된 링크가 **어떤 타입**인지까지 함께 예측하는 **관계 예측(Relation Prediction) 모델**입니다.  
특히 이 모델은 **엣지의 존재 여부**뿐 아니라, **관계 유형**까지 함께 학습하는 **반지도 학습(Semi-supervised)** 방식을 기반으로 작동합니다.

즉, 단순한 구조 복원을 넘어, 각 연결에 내재된 의미 있는 관계 유형(예: 인과, 상하, 관계있음 등)을 예측하는 것을 목표로 합니다.

또한 이 모델은 **Transductive 학습 방식**을 따릅니다.  
학습 시점에 주어진 **하나의 고정된 그래프 전체를 기반**으로 모델이 학습되며,  
**그래프에 포함되지 않은 새로운 노드에 대해서는 일반화가 어렵습니다.**  
따라서 새로운 노드를 예측에 포함하려면 전체 그래프를 다시 구성하고 모델을 재학습해야 합니다.

---

## 모델 구조

- **인코더**: R-GCN을 사용하여 노드 간 관계 정보를 반영한 **잠재 벡터 z**를 생성합니다.  
- **디코더**: 생성된 z를 바탕으로 MLP를 통해 노드 쌍의 **링크 존재 여부(score)** 및 **링크 관계 유형(type)**을 예측합니다.

### 비지도(Unsupervised) vs 반지도(Semi-supervised) 학습 비교

| 항목 | 비지도 학습 (Unsupervised) | 반지도 학습 (Semi-supervised) |
|------|-----------------------------|-------------------------------|
| 노드 타입 라벨 | ❌ 없음 | ✅ 일부 노드에 있음 (선택적) |
| 엣지 타입 라벨 | ❌ 없음 | ✅ 있음 (예: 인과, 상하 등) |
| 학습 목적 | 연결 구조만으로 z 임베딩 학습 및 숨은 링크 예측 | 구조 + 관계 유형 학습 (링크 존재 + 타입 예측) |
| 손실 함수 구성 | 재구성 손실 (link reconstruction loss) | 재구성 손실 + 관계 분류 손실 (cross-entropy) |
| 라벨 필요성 | 라벨 없이 구조만으로 학습 가능 | 일부 라벨을 학습에 활용 |

![alt text](images/flow.png)


## 모델 성능
<p align="center">
  <img src="images/loss_plot.png" width="300"/>
  <img src="images/link_performance.png" width="300"/>
  <img src="images/type_performance.png" width="300"/>
</p>

## 실행 방법
#### 1. 환경 설정
```
- Python 3.10.16  
- CUDA 12.1  
- torch 2.6.0  
- torch-geometric 2.6.1  
- numpy 1.24.4  
- pandas 2.2.3
```
#### 2. 의존성 설치
```
pip install -r requirements.txt
```

#### 3. 실행
```
python predict.py
```
## 데이터 구조

모델은 **하나의 고정된 그래프**를 입력으로 사용합니다. 다음과 같은 `.npy` 배열로 구성됩니다:

```
data/
├── x.npy # 각 노드의 임베딩 특성 (shape: [num_nodes, feat_dim])
├── edge_index.npy # 노드 간 연결 정보 (shape: [2, num_edges])
└── edge_type.npy # 각 연결의 관계 유형 정보 (shape: [num_edges])
```

## 프로젝트 구조
```
RVGAE/
├── images/                # Input 데이터
├── model.py               # R-VGAE 모델 정의
├── predict.py             # 링크 및 타입 예측 실행
└── requirements.txt       # 환경 설정
```
