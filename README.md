# AIGT Detection: Lightweight Gemma-3-4B Baseline

이 프로젝트는 인공지능이 생성한 한국어 텍스트를 탐지하기 위한 경량화된 베이스라인 모델을 제공합니다. **Gemma-3-4B-it** 모델을 기반으로 하며, **4비트 양자화(4-bit Quantization)**와 **LoRA**를 적용하여 효율적인 학습이 가능하도록 설계되었습니다.

## 1. 핵심 특징

- **모델 경량화**: `google/gemma-3-4b-it` 모델 사용 및 4비트 양자화를 통해 GPU 메모리 점유율을 대폭 낮췄습니다.
- **대조 학습(Contrastive Learning)**: `ScheduledCLTrainer`를 도입하여 학습 진행도에 따라 분류 손실(Cross-Entropy)과 대조 손실(InfoNCE)을 조절하며 텍스트 간의 미세한 차이를 학습합니다.
- **효율적 전처리**: 데이터 크기를 전략적으로 샘플링(1/4)하고, 문단 길이의 35%~95% 퍼센타일을 기준으로 이상치를 필터링하여 데이터 품질을 높였습니다.

## 2. 주요 경로 및 구조

- `./data/original_data/`: 대회 원본 데이터 파일 위치 (`train.csv`, `test.csv`)
- `./train_simple/data/`: 전처리 및 샘플링이 완료된 학습/검증용 데이터 저장 경로
- `./train_simple/gemma_model/`: 학습 완료 후 저장되는 LoRA 어댑터 및 토크나이저 경로

## 3. 실행 방법

### **Step 1: 데이터 전처리**

`data_preprocess_simple.ipynb` 파일을 실행합니다.

1. 원본 텍스트를 문단 단위로 분리합니다.
2. **Stratified 샘플링**과 **언더샘플링**을 통해 클래스 비율을 1:1로 맞춥니다.
3. 학습용(80%) 및 검증용(20%) 데이터를 생성합니다.

### **Step 2: 모델 학습**

`gemma_train.ipynb` 파일을 실행합니다.

1. Hugging Face 로그인을 수행합니다.
2. **BitsAndBytes** 설정으로 모델을 4비트로 로드하고 LoRA 레이어를 추가합니다.
3. `ScheduledCLTrainer`를 통해 1 에폭 동안 학습을 진행하며 최적의 모델을 저장합니다.

### **Step 3: 추론 및 제출**

학습 코드 하단의 추론 섹션을 실행합니다.

1. 테스트 데이터를 전처리된 형식으로 로드합니다.
2. 학습된 모델로 각 문단의 AI 생성 확률을 예측합니다.
3. 결과는 `./train_simple/submission.csv`로 저장됩니다.

## 4. 학습 파라미터 요약

- **Base Model**: `google/gemma-3-4b-it`
- **LoRA Rank**: 32 (Alpha: 16)
- **Learning Rate**: 2e-5
- **Batch Size**: 8 (per device)
- **CL Scheduler**: 초기 30% 구간 이후 대조 학습 활성화 (Max Lambda: 0.05)
