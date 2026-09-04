# METIS 기반 AI 배양조건 최적화 — 진행 기록 20260904 (README)

수도권 도로변·공원 토양 유래 분리주 *Priestia megaterium*의 배지 조성 최적화를
[METIS](https://github.com/amirpandi/METIS)(Pandi et al., *Nat. Commun.*, 2022) 능동학습 프레임워크로 진행한 과정 전체 기록.

---

## 1. 대상 균주 및 배경

- **대상 균주**: *Priestia megaterium* — 도로변(HIY2, strain DSM 32)과 공원(JYR2, strain IS4_1) 양쪽 토양에서 독립적으로 분리됨
- **목적**: 4개 배지 성분(Peptone, Meat extract, Yeast extract, Glucose)의 최적 조합을 최소한의 실험으로 탐색
- **도구**: METIS Optimization Notebook (Google Colab) + `utils.py`

---

## 2. Results_1.csv — 2라운드 원본 데이터 구성 과정

### 2-1. 실험 설계
팀원 3명(인영, 유라, 정관)이 각자 4개씩, 총 **12개 배지 조합**을 배양하여 OD600을 측정함.

- 인영(조건 1~4): 전 성분 동일 비율로 25/50/75/100% 스크리닝
- 유라(조건 5~8): 기준 50/50/50/50에서 한 성분씩 100%로 올리는 OVAT(One-Variable-At-a-Time) 방식
- 정관(조건 9~12): 50~75% 구간의 부분 조합

> 이 12개 조합 자체는 METIS가 무작위 생성한 것이 아니라 **팀원이 수동으로 설계한 초기 스크리닝**임. 이 데이터를 모델 학습에 사용하는 단계부터가 실질적인 "머신러닝" 단계.

### 2-2. OD 측정 및 yield 값 결정
각 조합을 1차, 2차 두 번 측정함. 두 값 중 **2차 값만 최종 yield로 채택** (1차보다 재현성이 더 좋다고 판단).

### 2-3. 실제 농도값 매핑
팀 엑셀(`METIS_배양조건_결과.xlsx`)에 정리된 "조건 1~12(비율 %)"는 팀 자체 기록용 라벨이며, 실제 모델 입력에는 **엑셀의 "Concentration" 표에 있는 실제 농도값**(g/L 단위, Peptone/Meat/Yeast/Glucose)을 사용함.

### 2-4. 최종 Results_1.csv

| # | Peptone | Meat | Yeast | Glucose | yield |
|---|---|---|---|---|---|
| 1 | 20 | 12.8 | 0.48 | 20 | 0.712 |
| 2 | 5 | 8.96 | 0.08 | 30 | 0.879 |
| 3 | 10 | 15.2 | 3.68 | 10 | 0.801 |
| 4 | 5 | 4.0 | 3.44 | 1 | 0.812 |
| 5 | 5 | 8.16 | 3.36 | 30 | 0.796 |
| 6 | 1 | 13.76 | 7.68 | 1 | 0.602 |
| 7 | 5 | 15.68 | 8.0 | 30 | 0.571 |
| 8 | 10 | 8.96 | 6.48 | 30 | 0.726 |
| 9 | 10 | 1.12 | 4.48 | 20 | 0.703 |
| 10 | 10 | 11.52 | 6.24 | 1 | 0.799 |
| 11 | 5 | 0.0 | 7.6 | 30 | 0.758 |
| 12 | 5 | 15.2 | 0.0 | 20 | 0.761 |

<details>
<summary>CSV 원본 (클릭해서 펼치기)</summary>

```csv
Peptone,Meat,Yeast,Glucose,yield
20,12.8,0.48,20,0.712
5,8.96,0.08,30,0.879
10,15.2,3.68,10,0.801
5,4.0,3.44,1,0.812
5,8.16,3.36,30,0.796
1,13.76,7.68,1,0.602
5,15.68,8.0,30,0.571
10,8.96,6.48,30,0.726
10,1.12,4.48,20,0.703
10,11.52,6.24,1,0.799
5,0.0,7.6,30,0.758
5,15.2,0.0,20,0.761
```

</details>

### 2-5. 겪었던 문제와 해결

| 문제 | 원인 | 해결 |
|---|---|---|
| `aggregated_data_m`이 빈 표로 출력됨 | `day_finder('Results')`가 `Results_1.csv`를 못 찾아 `day=0`이 되고, `range(start_day, day+1)`이 빈 범위가 됨 (업로드 전에 셀을 실행했거나 파일명이 다름) | 파일 업로드 후 `day_finder` 셀부터 재실행 |
| 컬럼명이 `Peptone`이 아니라 인식됨 (KeyError 위험) | 엑셀 → CSV 저장 시 파일 맨 앞에 **UTF-8 BOM**이 붙어 첫 컬럼명이 `﻿Peptone`으로 저장됨 | `pd.read_csv('Results_1.csv', encoding='utf-8-sig')`로 BOM 자동 제거 |

---

## 3. 사전 설정값 (User Inputs)

```python
m = 4  # 다음 라운드 추천 조합 개수
minimum_drop_size_nanoliter = 1000
final_reaction_volume_nanoliter = 1000000
maximum_volume_of_model_output = 750000
fixed_parts = {'Fixed': 0.25}

concentrations_limits = {
    'Peptone': {'Conc_Values': [1.0, 5.0, 10.0, 20.0], 'Conc_Stock': 200.0},
    'Meat':    {'Conc_Min': 0.0, 'Conc_Max': 16.0, 'Conc_Stock': 160.0},
    'Yeast':   {'Conc_Min': 0.0, 'Conc_Max': 8.0,  'Conc_Stock': 80.0},
    'Glucose': {'Conc_Values': [1.0, 10.0, 20.0, 30.0, 400.0], 'Conc_Stock': 200.0}
}
```
> ⚠️ Glucose의 `Conc_Stock`이 강의자료 표 기준(400.0)과 다르게 200.0으로 설정되어 있음. 실제 스탁 제조 기록과 대조해서 확인 필요 (모델 학습 자체에는 영향 없음, 부피 계산에만 영향).

---

## 4. 분석 파이프라인

### 4-1. 모델 학습
- 기반 모델: **XGBRegressor** (XGBoost)
- 하이퍼파라미터 탐색: **RandomizedSearchCV** (200회 샘플링, 4-fold CV, `neg_mean_absolute_error` 기준)
- 상위 20개 하이퍼파라미터 조합으로 **20개 독립 모델 앙상블** 구성 → 이후 모든 결과는 이 20개 모델의 평균

### 4-2. Feature Importance
20개 모델의 `.feature_importances_`를 평균·표준편차로 집계.

| 성분 | 중요도 (평균 ± SD) |
|---|---|
| Yeast | 0.45 ± 0.08 |
| Meat | 0.30 ± 0.10 |
| Peptone | 0.17 ± 0.09 |
| Glucose | 0.08 ± 0.02 |

### 4-3. Day 2 추천 조합 (`bayesian_optimization`, active learning)
exploitation(예측 수율) + exploration(불확실성) 균형을 고려한 다음 라운드 추천 4개.

| Peptone | Meat | Yeast | Glucose |
|---|---|---|---|
| 5.0 | 11.68 | 3.84 | 1.0 |
| 5.0 | 11.52 | 3.84 | 10.0 |
| 5.0 | 12.00 | 4.32 | 10.0 |
| 5.0 | 11.84 | 4.08 | 10.0 |

> 실제 2라운드 습식 실험은 진행하지 않기로 결정 (시간 제약).

### 4-4. 예측 최적 조성 (전체 탐색공간 5,000개 무작위 샘플 중 exploitation-only 최댓값)

| Peptone | Meat | Yeast | Glucose | predicted_yield |
|---|---|---|---|---|
| 5.0 | 10.24 | 0.16 | 10.0 | 0.856 |
| 5.0 | 9.92 | 0.40 | 10.0 | 0.856 |
| 5.0 | 10.88 | 0.40 | 10.0 | 0.856 |

> 1라운드 실측 최고값(yield=0.879, Yeast=0.08)이 모델 예측 최적값(0.856)보다 높음 — 학습 데이터가 12개로 적어 발생하는 한계로 해석.

### 4-5. K Most Informative Combinations — ⚠️ 참고용
"12개 중 몇 개만으로도 나머지를 잘 예측할 수 있는가"를 테스트. k=8로 시도했으나 **테스트셋이 4개뿐이라 결과가 불안정**(성능 분포 0~1, 평균 0.364). 데이터가 더 쌓이기 전까지는 메인 결과로 사용하지 않기로 함.

### 4-6. NonLinear Interactions — ✅ 채택
성분 쌍별로 상호작용항(곱)을 추가했을 때 예측 성능(Pearson r)이 얼마나 개선되는지 측정.

| 상호작용 | r 개선폭 |
|---|---|
| Peptone × Glucose | +0.128 |
| Peptone × Yeast | +0.123 |
| Meat × Glucose | +0.022 |
| Meat × Yeast | +0.006 |
| Yeast × Glucose | +0.001 |
| Peptone × Meat | 0.000 |

> Peptone은 단독 중요도(17%, 4개 중 3위)는 낮지만, Glucose·Yeast와의 상호작용은 가장 강함 — 단일 변수 중요도만으로는 놓칠 수 있는 결과.

---

## 5. 이번 라운드에서 스킵한 노트북 섹션과 이유

| 섹션 | 스킵 이유 |
|---|---|
| Find Feature Importances (공식 섹션) | 이미 한 것과 방법론 중복, 여러 라운드 있을 때(라운드별 추이 비교)만 의미 있음 |
| Visualising Concentrations From Day_1 to Now | 라운드 1개뿐이라 "추이"를 보여줄 데이터 없음 |
| BoxPlot / Each Metabolite 시각화 | Feature Importance·Top5 결과로 이미 커버됨 |
| Table2Speech / ECHO Liquid Handler 변환 | 액체분주 로봇 전용 포맷, 수작업 파이펫팅이라 불필요 |

---

## 6. 산출 파일 목록

- `Results_1.csv` — 1라운드 실측 데이터 (Peptone/Meat/Yeast/Glucose/yield)
- `feature_importance.png` — Feature Importance 막대그래프
- `Day_2/Concentrations_2.csv`, `Day_2/Volumes_2.csv` — 2라운드 추천 농도·부피 (미실행)
- `Interactions.png`, `Interactions.csv` — 성분 간 상호작용 히트맵

## 7. 논문 반영 상태

- Materials and methods → Data analysis 섹션: 작성 완료
- Results 섹션: Feature Importance, 예측 최적조합 Top5 작성 완료 (Figure/Table 삽입 자리만 남음)
- Interaction 결과: 아직 Results 문단 미작성

## 8. 남은 TODO

- [ ] Interaction 결과를 Results 문단에 추가
- [ ] Feature Importance 그래프(`feature_importance.png`), Interaction 히트맵(`Interactions.png`) 실제 이미지 논문에 삽입
- [ ] Glucose `Conc_Stock` 값(200 vs 400) 실제 스탁 기록과 대조 확인
- [ ] (선택) 2라운드 습식 실험 진행 여부 최종 확정
