# AIFactory-Industrial-Machine-Fatigue-Prediction
제 4회 2023 연구개발특구 AI SPACK 챌린지 - 공기압축기 이상 판단

이 프로젝트는 제4회 2023 연구개발특구 AI SPARK 챌린지 - 공기압축기 이상 판단 대회에 참가하여 개발한 산업기기 이상 전조증상 예측 모델입니다.

![AIfactory공모전제목사진](https://github.com/youngjr0527/AI-Factory-Industrial-Machine-Fatigue-Prediction/assets/83463280/54281f6b-633b-4167-a5ab-2cf246ec2351)


## 🏆 리더보드 결과

- Public Score: 0.9605793854
- Private Score: 0.9542428615

## 📝 프로젝트 개요

이 프로젝트는 제4회 2023 연구개발특구 AI SPARK 챌린지 - 공기압축기 이상 판단 대회에 참가하여 개발한 산업기기 이상 전조증상 예측 모델입니다. 대회의 목표는 피로도 증가 시 데이터 학습을 통해 산업기기 이상 전조증상을 예측하여 기기 고장과 사고를 예방하는 모델을 개발하는 것입니다.

## 📊 데이터 구성
![데이터구성](https://github.com/youngjr0527/AI-Factory-Industrial-Machine-Fatigue-Prediction/assets/83463280/68cb1d07-bca3-4e2e-90d4-368577ed8fb3)

- air_inflow: 공기 흡입 유량 (㎥/min)
- air_end_temp: 공기 말단 온도 (°C)
- out_pressure: 토출 압력 (Mpa)
- motor_current: 모터 전류 (A)
- motor_rpm: 모터 회전수 (rpm)
- motor_temp: 모터 온도 (°C)  
- motor_vibe: 모터 진동 (mm/s)
- type: 설비 번호

## 🔍 데이터 탐색 및 전처리

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
- 한계:
  - Long-term dependencies 문제 처리에 여전히 한계 존재 
  - Train data는 정상 데이터, test data는 정상+이상 데이터로 구성되어 overfitting 가능성
  - Overfitting 및 pre-trained 모델 생성을 위해 TransAD 또는 Anomaly Transformer 모델 필요
- Public Score: 0.7147570364



![TranAD vs AT](https://github.com/youngjr0527/AI-Factory-Industrial-Machine-Fatigue-Prediction/assets/83463280/aedc823f-9352-46ee-9e79-bc9c245a2ea2)


### 3. Anomaly Transformer
![AT12](https://github.com/youngjr0527/AI-Factory-Industrial-Machine-Fatigue-Prediction/assets/83463280/f5569e0d-cd7c-4b94-a2f2-3860c2ae35ad)
![AT34](https://github.com/youngjr0527/AI-Factory-Industrial-Machine-Fatigue-Prediction/assets/83463280/7e9b43d5-b7d6-475c-b3f6-525cc4186b5a)

- 지역적 시계열 특징과 전반적 시계열 특징을 결합하여 정확한 이상치 탐지 제공
- RNN 기반 방법론의 한계 개선
- 단계별 모델 개선 과정:
  1. Vanilla Anomaly Transformer 적용
     - Public Score: 0.6092, Total Score: 0.6038

  2. PSM 데이터셋 Pre-trained 모델 사용  
     - PSM dataset으로 pre-train한 model 사용
     - Public Score: 0.7889, Total Score: 0.7849
     - 실험적&이론적 한계로 PSM dataset 활용 불가 판단

  3. HP(마력) feature 추가 및 전체 데이터셋 학습
     - HP(마력) feature를 추가하여 전체 데이터셋 학습  
     - Public Score: 0.9040, Total Score: 0.9002

  4. 최종 모델
     - HP별로 train/test dataset 분리 및 추론
     - HP별 하이퍼파라미터 튜닝 세트 설정  
       - Anomaly ratio: 시각화를 통한 threshold 변경으로 이상/정상 분류
       - Window size: Domain 특성을 고려하여 train/test 시 window size를 1로 설정  
       - Shuffle: train 시 다양한 상황 학습을 위해 True, test 시 분류 결과 순서 유지를 위해 False
     - Public Score: 0.9606, Total Score: 0.9542



