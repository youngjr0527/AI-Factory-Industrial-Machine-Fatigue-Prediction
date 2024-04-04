
# AI SPACK 챌린지 - 공기압축기 이상 판단

<img src="https://github.com/youngjr0527/AI-Factory-Industrial-Machine-Fatigue-Prediction/assets/83463280/54281f6b-633b-4167-a5ab-2cf246ec2351" width="500" height="400">

## 리더보드 결과

- Public F1-Score Score: **0.96057**
- Private F1-Score Score: **0.95424**

## 프로젝트 개요

제4회 연구개발특구 AI SPARK 챌린지는 산업기기 피로도를 예측하는 문제이다.
산업용 공기압축기 및 회전기기에서 모터 및 심부 온도, 진동, 노이즈 등은 기기 피로도에 영향을 주는 요소이며, 피로도 증가는 장비가 고장에 이르는 원인이 된다.
피로도 증가 시 데이터 학습을 통해 산업기기 이상 전조증상을 예측하여 기기 고장을 예방하고 그로 인한 사고를 예방하는 모델을 개발하는 것이 목표이다.

## 데이터 구성

<img src="https://github.com/youngjr0527/AI-Factory-Industrial-Machine-Fatigue-Prediction/assets/83463280/68cb1d07-bca3-4e2e-90d4-368577ed8fb3" width="500" height="500">

- air_inflow: 공기 흡입 유량 (㎥/min)
- air_end_temp: 공기 말단 온도 (°C)
- out_pressure: 토출 압력 (Mpa)
- motor_current: 모터 전류 (A)
- motor_rpm: 모터 회전수 (rpm)
- motor_temp: 모터 온도 (°C)  
- motor_vibe: 모터 진동 (mm/s)
- type: 설비 번호
  - 설비 번호 [0, 4, 5, 6, 7]: 30HP(마력)
  - 설비 번호 1: 20HP
  - 설비 번호 2: 10HP
  - 설비 번호 3: 50HP
    
## 데이터 탐색 및 전처리

- 데이터 분포 확인
- Feature 연관성 확인 
- 시계열 데이터 패턴 확인
- 데이터 범위 조절
- Feature 범위 스케일링
- Feature 제거/추가

## 🔧 모델 개발 과정

### 1. Isolation Forest

- Random Forest 알고리즘 기반의 비지도 학습 방식 이상치 탐지
- 데이터 포인트의 평균 경로 길이를 계산하여 이상치 판단 
- 장점:
  - 계산 효율성: 데이터를 무작위로 분할하는 과정 덕분에 대규모 데이터셋에서도 빠르게 작동
  - 파라미터 튜닝이 적게 필요: 최적화된 파라미터 선택에 크게 의존하지 않아 복잡한 튜닝 불필요
- 단점:  
  - 결과 해석의 어려움: 무작위 분할 과정으로 인해 이상치 감지 이유 설명이 어려움
  - 이상치 비율이 높은 경우 정확도 저하: 이상치 비율이 높은 데이터셋에서 성능 저하 가능성

### 2. LSTM-AutoEncoder

- 시계열 데이터 처리에 효과적인 모델
- 고차원 시계열 데이터에서 이상치 탐지 가능  
- 장점:
  - 시계열 데이터의 장기 의존성 문제 해결을 위해 제시된 모델
  - 여러 gate를 활용하여 gradient vanishing 문제에 효과적으로 접근
  - AutoEncoder를 통해 입력 데이터의 노이즈를 제거하면서 차원 축소 및 특징 추출
- 단점:
  - Long-term dependencies 문제 처리에 여전히 한계 존재 
  - Train data는 정상 데이터, test data는 정상+이상 데이터로 구성되어 overfitting 가능성
  - Overfitting 및 pre-trained 모델 생성을 위해 TransAD 또는 Anomaly Transformer 모델 필요
- Public Score: **0.7147570364**

### 3. Transformer-based 모델 조사
<img src="https://github.com/youngjr0527/AI-Factory-Industrial-Machine-Fatigue-Prediction/assets/83463280/aedc823f-9352-46ee-9e79-bc9c245a2ea2" width="500" height="300">

### 4. Anomaly Transformer
- 지역적 시계열 특징과 전반적 시계열 특징을 결합하여 정확한 이상치 탐지 제공
- RNN 기반 방법론의 한계 개선
  
<img src="https://github.com/youngjr0527/AI-Factory-Industrial-Machine-Fatigue-Prediction/assets/83463280/f5569e0d-cd7c-4b94-a2f2-3860c2ae35ad" width="700" height="340">

1. Vanilla Anomaly Transformer 적용
     - Public Score: **0.6092**, Total Score: **0.6038**

2. PSM 데이터셋 Pre-trained 모델 사용  
     - PSM dataset으로 pre-train한 model 사용
     - Public Score: **0.7889**, Total Score: **0.7849**
     - 실험적&이론적 한계로 PSM dataset 활용 불가 판단

<img src="https://github.com/youngjr0527/AI-Factory-Industrial-Machine-Fatigue-Prediction/assets/83463280/7e9b43d5-b7d6-475c-b3f6-525cc4186b5a" width="700" height="340">

3. Feature 추가 및 전체 데이터셋 학습
     - HP(마력) feature를 추가하여 전체 데이터셋 학습  
     - Public Score: **0.9040**, Total Score: **0.9002**

4. 최종 모델
     - HP별로 train/test dataset 분리 및 추론
     - HP별 하이퍼파라미터 튜닝 세트 설정  
       - Anomaly ratio: 시각화를 통한 threshold 변경으로 이상/정상 분류
     - Public Score: **0.9606**, Total Score: **0.9542**



[마력(HP) 20 추론 시각화 예시]

<img src="https://github.com/youngjr0527/AI-Factory-Industrial-Machine-Fatigue-Prediction/assets/83463280/688bce01-1a20-43d2-a90d-a8963c876125" width="400" height="300">

