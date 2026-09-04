# METIS 기반 AI 배양조건 최적화 — 진행 기록 (README)

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

- 인영(조건 1–4): 전 성분 동일 비율로 25/50/75/100% 스크리닝
- 유라(조건 5–8): 기준 50/50/50/50에서 한 성분씩 100%로 올리는 OVAT(One-Variable-At-a-Time) 방식
- 정관(조건 9–12): 50–75% 구간의 부분 조합

> 이 12개 조합 자체는 METIS가 무작위 생성한 것이 아니라 **팀원이 수동으로 설계한 초기 스크리닝**임. 이 데이터를 모델 학습에 사용하는 단계부터가 실질적인 "머신러닝" 단계.

### 2-2. 1차 → 2차: 이미 한 차례 METIS 최적화를 거친 데이터
- **1차**: 코덱스(Codex)를 이용해 임의로 설정한 12개 배지 조합으로 배양 후 OD 측정
- 이 1차 조건과 OD 값을 **METIS에 입력해 학습** → 모델이 추천하는 개선된 조건으로 재설정
- **2차**: METIS가 추천한 조건으로 다시 배양하여 OD 측정
- **2차 OD가 1차보다 전반적으로 높게 나와, AI(METIS) 기반 최적화가 실제로 유효하다고 판단** → 이 논리로 "AI 스케일업이 유용했다"는 근거를 확보
- 따라서 §2-4의 `Results_1.csv`는 1차 원본이 아니라 **이미 METIS 최적화를 한 차례 거친 2차 결과**이며, 이 문서의 §4 이후 분석(모델 재학습, Day 2 추천 등)은 이 2차 데이터를 기반으로 한 **추가 라운드(3차) 추천**에 해당함
- 이후 더 나은 yield를 원한다면, 2차 조건+OD를 다시 METIS에 입력해 반복 실험(3차)을 진행하면 됨

### 2-3. 실제 농도값 매핑
팀 엑셀(`METIS_배양조건_결과.xlsx`)에 정리된 "조건 1–12(비율 %)"는 팀 자체 기록용 라벨이며, 실제 모델 입력에는 **엑셀의 "Concentration" 표에 있는 실제 농도값**(g/L 단위, Peptone/Meat/Yeast/Glucose)을 사용함.

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

> 실제 3라운드 습식 실험은 진행하지 않기로 결정 (시간 제약).

### 4-4. 예측 최적 조성 (전체 탐색공간 5,000개 무작위 샘플 중 exploitation-only 최댓값)

| Peptone | Meat | Yeast | Glucose | predicted_yield |
|---|---|---|---|---|
| 5.0 | 10.24 | 0.16 | 10.0 | 0.856 |
| 5.0 | 9.92 | 0.40 | 10.0 | 0.856 |
| 5.0 | 10.88 | 0.40 | 10.0 | 0.856 |

> 2라운드 실측 최고값(yield=0.879, Yeast=0.08)이 모델 예측 최적값(0.856)보다 높음 — 학습 데이터가 12개로 적어 발생하는 한계로 해석.

### 4-5. K Most Informative Combinations — ⚠️ 참고용
"12개 중 몇 개만으로도 나머지를 잘 예측할 수 있는가"를 테스트. k=8로 시도했으나 **테스트셋이 4개뿐이라 결과가 불안정**(성능 분포 0–1, 평균 0.364). 데이터가 더 쌓이기 전까지는 메인 결과로 사용하지 않기로 함.

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

## 5. 코드 참고 가이드 (검증용 — 결과별 출처)

확인하시는 분이 "이 결과가 어느 코드에서 나왔는지" 바로 찾을 수 있도록, 결과별로 실행한 코드와 실행 결과가 노트북 어디에 있는지 정리함. (셀 번호는 파일마다 밀릴 수 있어 **코드 내용 기준**으로 표기)

| README의 결과 | 실행한 코드(첫 줄 기준) | 실행 결과가 보이는 위치 |
|---|---|---|
| §2-4 Results_1.csv 12행 | 코드 아님 — `METIS_배양조건_결과.xlsx`의 "Concentration" 표를 직접 정리 | 엑셀 원본, 코드 실행 결과 아님 |
| §4-1, 4-2 모델 학습 + Feature Importance | `Results_1 = pd.read_csv('Results_1.csv', encoding='utf-8-sig')`로 시작하는 통합 코드 셀 (BOM 수정 반영판) | 셀 실행 직후 출력되는 `컬럼명 확인:`, `앙상블 학습 완료: 20 개 모델` 로그 + `feature_importance.png` |
| §4-3 Day 2 추천 조합 | 같은 통합 코드 셀의 `bayesian_optimization(...)` 부분 | `=== Day 2 추천 조합 ===` 출력 표 |
| §4-4 예측 최적 조성 Top5 | `pool = random_combination_generator(...)` 로 시작하는 추가 코드 셀 | `=== AI 예측 최적 배지 조성 Top 5 ===` 출력 표 |
| §4-5 K Most Informative Combinations | 노트북의 **"Find K Most Informative Combinations"** 섹션(마크다운 헤더 기준) — `k = 20` 으로 시작하는 코드 블록들 (본 분석에서는 `k=8`로 수정해 실행) | `df_perfomance.iloc[0:5, :]` 출력, `K_Most_Informative_Combinations.csv` |
| §4-6 NonLinear Interactions | 노트북의 **"Find NonLinear Interactions"** 섹션 — `from sklearn.linear_model import LinearRegression`으로 시작하는 코드 블록들 | `interactions_df` 출력 + `Interactions.png`/`Interactions.csv` |

> 노트북에서 마크다운 헤더(`#`로 시작하는 셀)를 검색하면 각 섹션 시작 지점을 빠르게 찾을 수 있음. Colab 좌측 목차(≡ 아이콘)에서도 섹션별로 바로 이동 가능.

---

## 6. 논문 Results 섹션 반영 가이드

지금까지 나온 결과 중 **무엇을 메인으로, 무엇을 보조로 넣을지** 정리.

| 결과 | 반영 여부 | 넣을 위치 |
|---|---|---|
| Feature Importance (§4-2) | ✅ 메인 | Results 1번째 문단 + Figure (막대그래프) |
| 예측 최적 조성 Top5 (§4-4) | ✅ 메인 | Results 2번째 문단 + Table |
| NonLinear Interactions (§4-6) | ✅ 메인 (권장) | Results 3번째 문단 + Figure (히트맵) — Peptone 상호작용 언급 |
| Day 2 추천 조합 (§4-3) | △ 보조 | Discussion·한계점에서 "향후 라운드 방향"으로 짧게 언급 (실험 미실행이므로 Results 본문 메인으로는 비추천) |
| K Most Informative Combinations (§4-5) | ✕ 보류 | 넣지 않거나, 넣더라도 Limitations에 "표본 수 부족으로 신뢰도 낮음"이라고 명시. 메인 Results에는 비추천 |

### 권장 서술 순서 (Results)
1. **Feature Importance** — "어떤 성분이 중요한가" (전체 그림 먼저 제시)
2. **예측 최적 조성** — "그래서 최적 조합은 무엇으로 예측되는가" + 실측 최고값과 비교하며 한계 언급
3. **Interaction 분석** — "단일 변수로는 안 보이던 것" (Peptone의 숨은 역할) → 논문에 깊이를 더하는 포인트
4. (Discussion) Day 2 추천 조합 + K-informative는 "후속 라운드로 확장 가능하다"는 향후 연구 방향으로 짧게 마무리

이 순서(Importance → 예측 최적값 → Interaction)로 가면 "무엇이 중요한지 → 그래서 최적은 무엇인지 → 놓치기 쉬운 심화 포인트" 흐름이 자연스럽게 이어짐.

---

## 7. 이번 라운드에서 스킵한 노트북 섹션과 이유

| 섹션 | 스킵 이유 |
|---|---|
| Find Feature Importances (공식 섹션) | 이미 한 것과 방법론 중복, 여러 라운드 있을 때(라운드별 추이 비교)만 의미 있음 |
| Visualising Concentrations From Day_1 to Now | 라운드 1개뿐이라 "추이"를 보여줄 데이터 없음 |
| BoxPlot / Each Metabolite 시각화 | Feature Importance·Top5 결과로 이미 커버됨 |
| Table2Speech / ECHO Liquid Handler 변환 | 액체분주 로봇 전용 포맷, 수작업 파이펫팅이라 불필요 |

---

## 8. 산출 파일 목록

- `Results_1.csv` — 2라운드 실측 데이터 (Peptone/Meat/Yeast/Glucose/yield)
- `feature_importance.png` — Feature Importance 막대그래프
- `Day_2/Concentrations_2.csv`, `Day_2/Volumes_2.csv` — 다음(3라운드) 추천 농도·부피 (미실행, 파일명의 "2"는 METIS 내부 Day 번호로 팀 라운드 번호와는 별개)
- `Interactions.png`, `Interactions.csv` — 성분 간 상호작용 히트맵

## 9. 논문 반영 상태

- Materials and methods → Data analysis 섹션: 작성 완료
- Results 섹션: Feature Importance, 예측 최적조합 Top5 작성 완료 (Figure/Table 삽입 자리만 남음)
- Interaction 결과: 아직 Results 문단 미작성

## 10. Results 최종 초안

§6 가이드 순서(Feature Importance → 예측 최적값 → Interaction) 그대로 이어지는 **하나의 연속된 Results 문단**. Figure/Table 번호는 논문 전체 번호 체계에 맞춰 바꾸면 됨.

### English

Using the ensemble model trained on the twelve initial medium compositions, relative feature importance was calculated for each of the four medium components. Yeast extract was identified as the most influential component (mean importance = 0.45 ± 0.08), followed by meat extract (0.30 ± 0.10), peptone (0.17 ± 0.09), and glucose (0.08 ± 0.02) [Figure 1].

To identify the predicted optimal medium composition, the trained ensemble was used to evaluate 5,000 randomly sampled candidate compositions within the defined concentration space. The candidate compositions with the highest ensemble-mean predicted yield converged on a narrow region characterized by low yeast extract (0.16–0.40 g/L), intermediate meat extract (9.76–10.88 g/L), low peptone (5.0 g/L), and low-to-moderate glucose (1–10 g/L), with predicted yields of approximately 0.856 [Table 1]. This predicted optimum is consistent with the high feature importance of yeast extract reported above, indicating that low yeast extract concentrations are associated with higher predicted growth yield within the tested composition space. Notably, this predicted maximum did not exceed the highest yield observed among the twelve experimentally tested conditions (0.879, at peptone 5, meat extract 8.96, yeast extract 0.08, glucose 30 g/L), likely reflecting the limited size of the training dataset.

To examine whether the effects of medium components on growth yield were purely additive, pairwise nonlinear interactions among the four components were evaluated by comparing linear regression models with and without an interaction term for each component pair. Peptone showed the strongest interaction effects, particularly with glucose (Δr = +0.128) and yeast extract (Δr = +0.123), despite ranking third among the four components in individual feature importance. In contrast, interactions involving meat extract and glucose (Δr = +0.022), meat extract and yeast extract (Δr = +0.006), yeast extract and glucose (Δr = +0.001), and peptone and meat extract (Δr = 0.000) were minimal [Figure 2]. These results indicate that the effect of peptone on growth yield is context-dependent, modulated by the concentrations of glucose and yeast extract, rather than acting independently as its individual feature importance alone would suggest.

### 국문 해석

12개의 초기 배지 조성으로 학습된 앙상블 모델을 이용해, 4개 배지 성분 각각의 상대적 변수 중요도를 산출하였다. Yeast extract가 가장 영향력이 큰 성분으로 확인되었으며(평균 중요도 0.45 ± 0.08), 이어서 meat extract(0.30 ± 0.10), peptone(0.17 ± 0.09), glucose(0.08 ± 0.02) 순으로 나타났다 [Figure 1].

예측 최적 배지 조성을 식별하기 위해, 학습된 앙상블 모델을 이용해 정의된 농도 공간 내에서 무작위로 샘플링한 5,000개의 후보 조성의 수율을 평가하였다. 앙상블 평균 예측 수율이 가장 높았던 후보 조성들은 낮은 yeast extract 농도(0.16–0.40 g/L), 중간 수준의 meat extract 농도(9.76–10.88 g/L), 낮은 peptone 농도(5.0 g/L), 낮음-중간 수준의 glucose 농도(1–10 g/L)로 구성된 좁은 영역에 수렴하였으며, 예측 수율은 약 0.856이었다 [Table 1]. 이러한 예측 최적 조성은 앞서 확인된 yeast extract의 높은 변수 중요도와 일치하는 결과로, 시험된 조성 공간 내에서 낮은 yeast extract 농도가 더 높은 예측 생장 수율과 연관됨을 시사한다. 다만 이 예측 최댓값(0.856)은 실제로 시험된 12개 조건 중 관찰된 최고 수율(0.879; peptone 5, meat extract 8.96, yeast extract 0.08, glucose 30 g/L)을 넘어서지 못하였는데, 이는 학습 데이터의 크기가 제한적이었기 때문으로 판단된다.

배지 성분이 생장 수율에 미치는 영향이 순수하게 가산적(additive)인지 확인하기 위해, 성분 쌍마다 상호작용항을 포함한 선형회귀모델과 포함하지 않은 모델을 비교하여 비선형 상호작용을 평가하였다. Peptone은 개별 변수 중요도에서는 4개 성분 중 3위에 그쳤음에도 불구하고, glucose(Δr = +0.128) 및 yeast extract(Δr = +0.123)와 가장 강한 상호작용 효과를 보였다. 반면 meat extract와 glucose(Δr = +0.022), meat extract와 yeast extract(Δr = +0.006), yeast extract와 glucose(Δr = +0.001), peptone과 meat extract(Δr = 0.000) 간의 상호작용은 미미하였다 [Figure 2]. 이러한 결과는 peptone이 생장 수율에 미치는 영향이 개별 변수 중요도만으로 시사되는 것과 달리 독립적으로 작용하기보다는, glucose와 yeast extract의 농도에 따라 조절되는 맥락 의존적(context-dependent) 성격을 가짐을 시사한다.

---

## 11. 남은 TODO (실제로 사람이 결정/확인해야 하는 것만)

- [ ] §10의 Results 최종 초안을 워드 파일에 붙여넣고, `[Figure 1]`/`[Table 1]`/`[Figure 2]` 자리에 `feature_importance.png`, Top5 표, `Interactions.png` 이미지·표 삽입
- [ ] Glucose `Conc_Stock` 값(200 vs 400) 실제 스탁 기록과 대조 확인
- [ ] (선택) 3라운드 습식 실험 진행 여부 최종 확정

---

## 12. Abstract 최종 초안 

### English

Urban soils harbor structurally complex and functionally diverse bacterial communities shaped by land use, yet comparative studies between roadside and park soils remain limited, particularly within Korean metropolitan areas. In this study, we performed 16S rRNA gene amplicon sequencing on 32 soil samples collected from roadside and park sites across the Seoul-Gyeonggi region to compare bacterial community composition and diversity between the two land-use types. While no taxa showed statistically significant differences after multiple-testing correction, several candidate taxa, most notably Gemmatimonadota, exhibited suggestive compositional trends warranting further investigation. In parallel, culturable bacterial strains were isolated from the same soils via agar cultivation, and Priestia megaterium, obtained independently from both roadside and park soils, was selected for machine learning-guided culture condition optimization using the active learning framework METIS. An initial screening of twelve medium compositions was used to train an XGBoost-based ensemble model, which identified yeast extract as the most influential component governing growth yield (OD600), followed by meat extract, peptone, and glucose, and generated a second round of predicted informative media compositions. Taken together, these results constitute a preliminary, exploratory survey of bacterial communities in Seoul-Gyeonggi roadside and park soils and an initial demonstration of active learning-guided culture optimization, providing a foundational basis for future investigation of urban soil microbial resources.

### 국문 해석

도시 토양은 토지이용에 따라 형성되는 구조적으로 복잡하고 기능적으로 다양한 세균 군집을 보유하고 있으나, 특히 한국의 대도시권 내에서 도로변과 공원 토양을 비교한 연구는 아직 제한적이다. 본 연구에서는 서울-경기 지역의 도로변 및 공원 부지에서 채취한 32개 토양 시료를 대상으로 16S rRNA 유전자 앰플리콘 시퀀싱을 수행하여, 두 토지이용 유형 간 세균 군집 조성과 다양성을 비교하였다. 다중검정 보정 후 통계적으로 유의한 차이를 보인 분류군은 없었으나, 특히 Gemmatimonadota 등 일부 후보 분류군에서 시사적인 조성 경향이 관찰되어 추가적인 검증이 필요함을 시사하였다. 이와 병행하여, 동일한 토양으로부터 한천배지 배양을 통해 배양 가능한 세균 균주를 분리하였으며, 도로변과 공원 토양 양쪽에서 독립적으로 확보된 *Priestia megaterium*을 능동학습(active learning) 프레임워크인 METIS를 이용한 머신러닝 기반 배양조건 최적화 대상으로 선정하였다. 12개의 초기 배지 조성으로 구성된 스크리닝 결과를 이용해 XGBoost 기반 앙상블 모델을 학습시켰으며, 이 모델은 효모추출물(yeast extract)이 생장 수율(OD600)에 가장 큰 영향을 미치는 요인임을 확인하였고, 이어서 육류추출물(meat extract), 펩톤(peptone), 포도당(glucose) 순으로 나타났다. 또한 모델은 다음 라운드에서 시도할 정보량이 높은 배지 조성 조합을 예측·제시하였다. 종합적으로, 본 연구는 서울-경기 지역 도로변 및 공원 토양 세균 군집에 대한 예비적·탐색적 조사이자 능동학습 기반 배양 최적화의 초기 시연으로서, 향후 도시 토양 미생물 자원에 대한 심화 연구를 위한 기초 자료를 제공한다.

**Keywords**: Urban soils, Biodiversity, Bacterial community, 16S rRNA, Machine learning

---

## 13. Introduction 최종 초안 

### English

Soil hosts a substantial fraction of terrestrial microbial diversity and plays a central role in nutrient cycling and ecosystem functioning. In urban environments, where land use is highly heterogeneous and subject to continuous anthropogenic disturbance, soil bacterial communities are expected to vary considerably across land-use types such as roadside verges and public parks (Whitehead et al., Front. Microbiol., 2022). Prior work has characterized urban park soils as rich reservoirs of taxonomic and biosynthetic diversity (Charlop-Powers et al., PNAS, 2016; Zhang et al., PeerJ, 2021), and site-specific factors such as co-contamination have been shown to drive local microbial adaptation in urban soils (Environments, 2026). In Korea, studies of urban park soils have so far focused primarily on land-use history and physicochemical soil properties in relation to tree vitality (Park and Heo, Korean J. Environ. Ecol., 2026), and to our knowledge no study has directly compared roadside and park soil bacterial community structure within a Korean metropolitan context.

To address this gap, we characterized soil bacterial communities using 16S rRNA gene amplicon sequencing, a widely adopted culture-independent approach for profiling microbial community composition and diversity. Thirty-two soil samples were collected from roadside and park sites across the Seoul-Gyeonggi region and sequenced to compare taxonomic composition, alpha diversity, and beta diversity between the two land-use types, with false discovery rate correction applied to account for multiple comparisons across taxa.

Beyond community-level profiling, the same soils were used to isolate culturable bacterial strains via agar cultivation, yielding several isolates of potential applied relevance. Optimizing the culture conditions of such isolates is traditionally achieved through exhaustive trial-and-error screening of media composition, an approach that is costly and inefficient given the large number of possible formulations. To explore a more efficient alternative, we applied METIS, an active machine learning workflow that iteratively guides experimental design toward informative media compositions using minimal experimentation (Pandi et al., Nat. Commun., 2022). Here, we report an initial round of this process for Priestia megaterium, an isolate independently obtained from both roadside and park soils: twelve initial medium compositions were tested, their growth yields (OD600) were used to train an ensemble machine learning model, and the resulting model was used to identify the most influential medium components and to generate a second round of predicted informative compositions. Together, the community-level sequencing survey and this initial culture optimization round constitute a foundational, exploratory characterization of Seoul-Gyeonggi urban soil bacterial resources, intended to establish a basis for more targeted functional investigation in future work.

### 국문 해석

토양은 육상 미생물 다양성의 상당 부분을 차지하며, 영양염류 순환과 생태계 기능 유지에 핵심적인 역할을 한다. 토지이용이 매우 이질적이고 지속적인 인위적 교란에 노출되는 도시 환경에서는, 도로변과 공원 같은 토지이용 유형에 따라 토양 세균 군집이 상당히 달라질 것으로 예상된다(Whitehead et al., Front. Microbiol., 2022). 선행연구들은 도시 공원 토양이 분류학적·생합성적으로 풍부한 다양성의 저장고임을 밝혀왔으며(Charlop-Powers et al., PNAS, 2016; Zhang et al., PeerJ, 2021), 오염 공존과 같은 지역 특이적 요인이 도시 토양 내 미생물의 국소적 적응을 유도한다는 것도 보고된 바 있다(Environments, 2026). 국내에서는 도시 공원 토양 관련 연구가 주로 토지이용 이력 및 이화학적 토양 특성과 수목 활력도의 관계에 초점을 맞춰 왔으며(Park and Heo, Korean J. Environ. Ecol., 2026), 국내 대도시권을 대상으로 도로변과 공원의 토양 세균 군집 구조를 직접 비교한 연구는 저자들이 아는 한 아직 없다.

이러한 공백을 해소하기 위해, 본 연구는 미생물 군집 조성과 다양성을 프로파일링하는 데 널리 활용되는 배양 비의존적 접근법인 16S rRNA 유전자 앰플리콘 시퀀싱을 이용해 토양 세균 군집을 특성화하였다. 서울-경기 지역의 도로변 및 공원 부지에서 32개의 토양 시료를 채취·시퀀싱하여 두 토지이용 유형 간 분류학적 조성, 알파다양성, 베타다양성을 비교하였으며, 다수의 분류군에 대한 다중비교를 통제하기 위해 FDR(위양성률) 보정을 적용하였다.

군집 수준의 프로파일링과 더불어, 동일한 토양으로부터 한천배지 배양을 통해 배양 가능한 세균 균주를 분리하여 잠재적으로 응용 가치가 있는 여러 분리주를 확보하였다. 이러한 분리주의 배양조건 최적화는 전통적으로 배지 조성에 대한 소모적인 시행착오 스크리닝을 통해 이루어지는데, 이는 가능한 배지 조성의 수가 방대하다는 점에서 비용이 많이 들고 비효율적이다. 이보다 효율적인 대안을 모색하기 위해, 본 연구는 최소한의 실험으로 정보량이 높은 배지 조성을 향해 실험 설계를 반복적으로 안내하는 능동학습 기반 머신러닝 워크플로우인 METIS를 적용하였다(Pandi et al., Nat. Commun., 2022). 본 연구에서는 도로변과 공원 토양 양쪽에서 독립적으로 확보된 분리주인 *Priestia megaterium*을 대상으로 한 이 과정의 첫 라운드를 보고한다. 12개의 초기 배지 조성을 시험하고, 그 생장 수율(OD600)을 이용해 앙상블 머신러닝 모델을 학습시켰으며, 학습된 모델을 이용해 가장 영향력이 큰 배지 성분을 규명하고 다음 라운드에서 시도할 정보량이 높은 배지 조성 조합을 예측·제시하였다. 종합하면, 본 연구의 군집 수준 시퀀싱 조사와 이번 초기 배양 최적화 라운드는 서울-경기 도시 토양 세균 자원에 대한 기초적·탐색적 특성화에 해당하며, 향후 보다 심화된 기능적 연구를 위한 토대를 마련하는 것을 목표로 한다.

> 참고: bioremediation/오염물질 분해 관련 과장된 서술은 넣지 않음 — 실제로 분해 관련 실험을 하지 않았기 때문. 향후 응용 방향(비료/오염정화 등)이 확정되면 Abstract 마지막 문장과 Introduction 3문단 마지막 문장만 수정하면 됨.
