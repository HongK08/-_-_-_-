# 창원시 지하수 수질 분석 및 AI 예측 모델링

<p align="center">
  <img src="https://img.shields.io/badge/Python-3.9%2B-blue?style=for-the-badge&logo=python" alt="Python Version">
  <img src="https://img.shields.io/badge/Status-Completed-blue?style=for-the-badge" alt="Status">
  <img src="https://img.shields.io/badge/License-Private-lightgrey?style=for-the-badge" alt="License: Private">
</p>

> **창원시 충적 대수층**의 시공간적 지하수 수질 분석 및 AI 기반 예측. 본 프로젝트는 다양한 딥러닝 아키텍처와 특징 세트를 체계적으로 평가하여 지하수 동특성을 예측하는 최적의 모델을 구축합니다.

이 리포지토리는 창원 지역의 지하수 동특성을 분석하고 예측하는 완전한 파이프라인을 포함합니다. 원시(Raw) 시계열 데이터를 처리하고, 센서 오류 보정 및 데이터 공백 보간을 수행하며, 다양한 특징 공학 기법을 적용하여 일련의 통제된 실험을 통해 최적의 모델과 특징 조합을 식별합니다.

---

## 목차 (Table of Contents)
- [데이터셋 (Dataset)](#데이터셋-dataset)
- [방법론 (Methodology)](#방법론-methodology)
- [실험 결과 (Experimental Results)](#실험-결과-experimental-results)
- [핵심 발견 및 결론 (Key Findings & Conclusion)](#핵심-발견-및-결론-key-findings--conclusion)
- [최종 모델 추천 (Final Model Recommendation)](#최종-모델-추천-final-model-recommendation)
- [파일 구조 (File Organization)](#파일-구조-file-organization)

---

## 데이터셋 (Dataset)

데이터는 **국가지하수정보센터(GIMS)**로부터 수집하였으며, 창원시에 위치한 3개 충적층 관측정의 시간당 시계열 데이터로 구성됩니다.

- **주요 변수**: 수위(m), 수온(°C), 전기전도도(EC, µS/cm)
- **데이터 기간**: **2021년 – 2025년** (연속 관측)
- **최종 데이터셋**: 전처리 및 보간 완료 후 **32,673건**의 연속적인 시간당 관측 데이터

---

## 방법론 (Methodology)

본 분석은 시계열 모델링에 적합한 **연속적이고 신뢰도 높은 데이터셋**을 구축하기 위해 엄격하고 다단계의 실험 설계를 따릅니다.

1.  **전처리 (Preprocessing)**: 3개 관측정의 원시 데이터를 `outer join`을 통해 통합하여 완전한 시간 인덱스를 생성했습니다. 물리적으로 불가능한 센서 값(`EC=0`)과 데이터 공백은 **선형 보간(Linear Interpolation)**을 통해 채워 넣어 데이터의 연속성을 확보했습니다.

2.  **특징 공학 (Feature Engineering)**: 모델에 포괄적인 정보를 제공하기 위해 다음과 같은 특징들을 설계했습니다.
    - **핵심 특징**: 수위, 수온
    - **타겟 관련 특징**: 원시 `EC` 및 온도 보정 `EC25`
    - **정적 특징**: DEM으로 추출한 관측소별 `고도(Elevation)`
    - **변동성 특징**: `변화율(EC_RoC)` 및 `6시간 이동 표준편차(EC_Std6H)`
    - **LAG 특징**: 24시간, 168시간(주별), 720시간(월별) 전의 과거 `EC25` 값

3.  **모델 아키텍처 (Model Architectures)**: 두 가지 주요 아키텍처를 평가했습니다.
    - **기본 LSTM (Vanilla LSTM)**: 강력한 베이스라인 역할을 하는 표준 LSTM 네트워크.
    - **어텐션 LSTM (LSTM with Attention)**: 장기 예측을 위해 가장 관련 있는 과거 시점에 '집중'하도록 설계된 고도화 모델.
    - 두 모델 모두 과적합 방지를 위해 규제(Regularization) 기법과 조기 종료(Early Stopping)를 적용했습니다.

4.  **통제된 실험 (Controlled Experiments)**: 최적의 조합을 찾기 위해, 다음과 같은 3가지 특징 세트에 대해 모델을 학습시키는 통제된 실험을 수행했습니다.
    - **`all_features`**: 모든 공학적 특징을 사용.
    - **`ec25_only`**: 핵심 특징 + `EC25`와 그 LAG 특징만 사용.
    - **`ec_only`**: 핵심 특징 + 원시 `EC`와 그 LAG 특징만 사용.

---

## 실험 결과 (Experimental Results)

다음 표는 90일 예측 구간(t+90)에 대해, 가장 우수한 아키텍처인 **어텐션 + LAG 모델**의 R² 점수를 3가지 특징 세트 실험별로 비교한 결과입니다.

| 관측소 (Well) | `all_features` (R²) | **`ec25_only` (R²) 🏆** | `ec_only` (R²) |
|:---:|:---:|:---:|:---:|
| **천선 (Cheonseon)** | 0.941 | **0.942** | 0.941 |
| **신촌 (Sinchon)** | 0.670 | **0.705** | 0.664 |
| **성산 (Seongsan)** | -0.409 | -0.836 | **-0.381** |

---

## 핵심 발견 및 결론 (Key Findings & Conclusion)

체계적인 실험을 통해 다음과 같은 몇 가지 중요한 통찰을 얻었습니다.

1.  **"단순함의 미학" (Less is More 원칙)**: **`ec25_only`** 특징 세트가 `all_features` 세트보다 일관되게 더 나은 성능을 보였습니다. 이는 수위, 변동성 등의 추가 특징들이 오히려 모델에게 '잡음(Noise)'으로 작용했음을 시사합니다. 온도 보정된 `EC25`와 과거(`LAG`) 값이라는 핵심 정보만 간결하게 제공했을 때 모델이 가장 효과적으로 학습했습니다.

2.  **'어텐션 + LAG' 조합의 승리**: **어텐션 LSTM** 모델은 **LAG 특징**과 결합했을 때 가장 강력한 아키텍처임이 증명되었습니다. 기본 어텐션 모델은 과적합 문제를 겪었지만, 명시적인 LAG 특징을 제공하자 예측 가능한 지역(신촌, 천선)에서 최고의 정확도를 달성하며 그 잠재력을 발휘했습니다.

3.  **성산 지역의 미스터리 - 데이터의 한계**: 모든 실험에 걸쳐, **어떤 모델도 성산 지역의 장기 수질을 성공적으로 예측하지 못했습니다.** 모델의 복잡도나 특징 세트와 무관한 일관된 실패는 이것이 모델링의 실패가 아닌 **데이터의 한계 문제**임을 강력하게 시사합니다. 성산의 동특성은 현재 데이터셋에 없는 복잡한 외부 요인(불규칙한 강우, 미기록된 양수, 토지 이용 변화 등)에 의해 좌우될 가능성이 높습니다.

---

## 최종 모델 추천 (Final Model Recommendation)

-   **신촌 및 천선 지역**: **`ec25_only` 특징 세트와 LAG 특징으로 학습된 `LSTMAttentionModel`**을 최종 모델로 채택합니다. 이 조합은 R² 0.70 이상의 매우 정확하고 신뢰도 높은 예측을 제공합니다.

-   **성산 지역**: 다른 접근법이 필요합니다. 향후 연구는 **외부 데이터(특히 강우량 데이터)를 통합**하거나, 예측 문제에서 **이상 탐지(Anomaly Detection)** 문제로 전환하여 외부 이벤트가 시스템에 영향을 미치는 시점을 식별하는 데 초점을 맞춰야 합니다.

---

## 파일 구조 (File Organization)

```bash
.
├── Model_Res
│   └── LSTM
│       ├── all_features
│       │   ├── lstm_lag/
│       │   └── attention_lag/
│       ├── ec25_only
│       │   ├── lstm_lag/
│       │   └── attention_lag/
│       └── ec_only
│           ├── lstm_lag/
│           └── attention_lag/
│
├── UQML_Changwon
│   ├── CODE
│   │   ├── Get_Data
│   │   │   └── FIN_DATA_IMP.py           (최종 전처리 스크립트)
│   │   └── Models
│   │       └── run_all_models_final.py   (최종 학습/실험 스크립트)
│   │
│   └── DATA_SET
│       ├── FINAL_DATA
│       │   └── Changwon_Preprocessed_with_Features.csv (모델링용 최종 데이터)
│       │
│       ├── RAW_DATA
│       │   ├── Changwon_Cheonseon_Alluvial.csv
│       │   ├── Changwon_Seongsan_Alluvial.csv
│       │   └── Changwon_Sinchon_Alluvial.csv
│       │
│       └── SITE_ELEVATIONS
│           └── Changwon_site_elevations.csv
│
└── ... (기타 프로젝트 폴더)
