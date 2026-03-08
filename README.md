# aiku-26-W-AlphaDent
> 📢 **2026년 겨울학기 AIKU 활동으로 진행한 프로젝트입니다**

# AlphaDent - 치아 병변 Instance Segmentation

## 소개

치과 진료 과정에서 촬영되는 구강 내 사진에는 **치아 마모, 충전물, 크라운, 다양한 단계의 충치** 등 여러 병변이 동시에 존재합니다.  
이러한 병변을 정확히 구분하고 표시하는 작업은 현재 대부분 **숙련된 치과 의사의 수작업 판독**에 의존하고 있습니다.

하지만 수작업 기반 판독은 다음과 같은 한계를 가지고 있습니다.

- 판독에 **시간과 비용이 많이 소요됨**
- **판독자에 따라 결과가 달라질 수 있음**

최근 컴퓨터 비전 분야에서는 객체의 위치뿐 아니라 **정확한 형태까지 예측하는 Instance Segmentation 기술**이 발전하면서 의료 영상에서도 자동화 연구가 활발히 진행되고 있습니다.

본 프로젝트에서는 **Kaggle AlphaDent**에서 제공하는 데이터를 사용하여  
구강 내 사진에서 **9개의 치과 병변 클래스를 인스턴스 단위로 분할**하는 문제를 해결하고자 합니다.

---

# 목표

구강 사진으로부터 **9개의 클래스에 대한 Instance Segmentation**을 수행하는 것이 목표입니다.

1. **YOLO 기반 모델을 사용하여 세그멘테이션 수행**
2. **파인튜닝 및 추가 기법을 통해 성능 개선**
3. **Kaggle 리더보드 Top 3 진입**

---

# 데이터셋

| 데이터셋 | 설명 |
|---|---|
| Kaggle AlphaDent Dataset | 총 **295명의 환자**로부터 수집된 **1,320장의 구강 사진 데이터** |

총 **9개의 클래스**

- Abrasion (마모)
- Filling (충전물)
- Crown (크라운)
- Caries Class 1~6 (충치 단계)

## 데이터 예시

<img width="800" height="1013" alt="image" src="https://github.com/user-attachments/assets/1b5a98ed-36d7-4149-81a6-cc709d824fa2" />

<img width="1200" height="800" alt="image" src="https://github.com/user-attachments/assets/2c5793d9-35f0-4c58-83d9-9ebb4ea1b96f" />

## 클래스 분포

데이터 분포를 확인한 결과 다음과 같은 특징을 확인할 수 있었습니다.

- **Abrasion 클래스의 데이터가 가장 많음**
- **Caries Class 4, 6 데이터가 매우 적음**

<img width="388" height="407" alt="image" src="https://github.com/user-attachments/assets/e13443eb-4b38-43fe-a059-2326153792de" />


---

## 데이터셋 링크

https://www.kaggle.com/competitions/alpha-dent/overview

---

# 데이터 전처리

## Jitter

색상과 밝기 변화를 적용하여 데이터 증강을 수행했습니다.

목적

- 환자마다 치아 색상과 병변 색상이 다름
- 촬영 환경에 따라 사진 밝기가 다름

→ 이러한 차이에 모델이 강건하게 대응하도록 하기 위함

---

## Translate / Scale / Flip

사람마다 치아의 **위치와 크기가 다르기 때문에**  
다양한 공간적 변형을 적용하여 모델의 일반화를 향상시켰습니다.

---

## Mosaic

YOLO에서 사용되는 대표적인 데이터 증강 기법입니다.

효과

- 하나의 이미지에 여러 객체를 동시에 학습 가능
- 작은 객체를 더 잘 탐지하도록 도움
- 모델의 **robustness 증가**

---

# 모델링

기본 모델

- **YOLO11m-seg**

Instance segmentation을 수행하기 위해 **YOLO segmentation 모델을 사용했습니다.**

---

# 훈련 및 평가

## Training 설정

- Base Model : YOLOv8
- Epoch : 50
- Batch Size : 16
- Optimizer : SGD

---

# 평가 지표

## mAP50

평가 과정

1. 예측 결과와 GT 사이의 **IoU 계산**
2. IoU threshold **0.5 이상이면 True Positive**
3. Confidence threshold를 변화시키며 **Precision-Recall curve 생성**
4. PR curve의 **AUC가 AP**
5. **9개 클래스의 AP 평균 → mAP50**

---

# 결과
<img width="1738" height="262" alt="image" src="https://github.com/user-attachments/assets/fb482092-cd55-4ae1-b822-202ffb7c6836" />

## Validation Dataset 결과

| Precision | Recall | mAP50 | mAP50-95 |
|---|---|---|---|
| 0.526 | 0.402 | 0.402 | 0.203 |

Public Test Dataset

- **mAP50 = 0.270**

---

# 문제 분석
<img width="962" height="800" alt="image" src="https://github.com/user-attachments/assets/c3201a4b-6741-4cf0-9454-9f8c60789961" />

클래스별 Recall을 확인한 결과 다음과 같은 문제를 발견했습니다.

| Class | Recall |
|---|---|
| Abrasion | 0.86 |
| Filling | 0.68 |
| Crown | 0.84 |
| Caries Class 1 | 0.11 |
| Caries Class 2 | 0.08 |
| Caries Class 3 | 0.15 |
| Caries Class 4 | 0 |
| Caries Class 5 | 0.42 |
| Caries Class 6 | 0 |

문제 원인

1. **데이터 불균형**

- Caries Class 4, 6 데이터가 매우 적음

2. **작은 객체 문제**

- 작은 병변이 background와 잘 구분되지 않음

---

# 해결 방법

### 클래스별 Confidence Threshold 조정

훈련 데이터와 검증 데이터 분포는 유사하지만  
**테스트 데이터 분포가 다르다는 점**을 확인했습니다.

따라서,

- 클래스별 **confidence threshold를 다르게 설정**

효과

- 작은 객체에서 발생하는 **False Positive 감소**

---

# 최종 결과

🎉 **Kaggle Leaderboard 3위 달성**

<img width="2170" height="506" alt="image" src="https://github.com/user-attachments/assets/4264054b-ab8d-4284-98e6-82893636672e" />


---

# 한계 분석

1. **소량 데이터 문제**

- Caries Class 4, 6 데이터가 약 **30개 수준**
- Transformer 기반 모델 파인튜닝이 어려움

2. **이미지 색상 분포 차이**

- 촬영 환경에 따른 색상 차이가 큼
- **Image color normalization 필요**

---

# 후속 연구 방향

1. **2단계 모델 구조**

- 작은 CNN 모델 → 병변 존재 여부 1차 분류
- YOLO → 상세 segmentation

2. **클래스별 최적 confidence threshold 탐색**

- 각 클래스별 detection threshold 최적화
