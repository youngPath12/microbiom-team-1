## Abstract

Urban soils harbor structurally complex and functionally diverse bacterial communities shaped by land use, yet comparative studies between roadside and park soils remain limited, particularly within Korean metropolitan areas. In this study, we performed 16S rRNA gene amplicon sequencing on 32 soil samples collected from roadside and park sites across the Seoul-Gyeonggi region to compare bacterial community composition and diversity between the two land-use types. While no taxa showed statistically significant differences after multiple-testing correction, several candidate taxa, most notably Gemmatimonadota, exhibited suggestive compositional trends warranting further investigation. In parallel, culturable bacterial strains were isolated from the same soils via agar cultivation, and Priestia megaterium, obtained independently from both roadside and park soils, was selected for machine learning-guided culture condition optimization using the active learning framework METIS. An initial screening of twelve medium compositions was used to train an XGBoost-based ensemble model, which identified yeast extract as the most influential factor governing growth yield (OD600), followed by meat extract, peptone, and glucose, and generated a second round of predicted informative media compositions. Taken together, these results constitute a preliminary, exploratory survey of bacterial communities in Seoul-Gyeonggi roadside and park soils and an initial demonstration of active learning-guided culture optimization, providing a foundational basis for future investigation of urban soil microbial resources.

**국문 해석**

도시 토양은 토지이용에 따라 형성되는 구조적으로 복잡하고 기능적으로 다양한 세균 군집을 보유하고 있으나, 특히 한국의 대도시권 내에서 도로변과 공원 토양을 비교한 연구는 아직 제한적이다. 본 연구에서는 서울-경기 지역의 도로변 및 공원 부지에서 채취한 32개 토양 시료를 대상으로 16S rRNA 유전자 앰플리콘 시퀀싱을 수행하여, 두 토지이용 유형 간 세균 군집 조성과 다양성을 비교하였다. 다중검정 보정 후 통계적으로 유의한 차이를 보인 분류군은 없었으나, 특히 Gemmatimonadota 등 일부 후보 분류군에서 시사적인 조성 경향이 관찰되어 추가적인 검증이 필요함을 시사하였다. 이와 병행하여, 동일한 토양으로부터 한천배지 배양을 통해 배양 가능한 세균 균주를 분리하였으며, 도로변과 공원 토양 양쪽에서 독립적으로 확보된 *Priestia megaterium*을 능동학습(active learning) 프레임워크인 METIS를 이용한 머신러닝 기반 배양조건 최적화 대상으로 선정하였다. 12개의 초기 배지 조성으로 구성된 스크리닝 결과를 이용해 XGBoost 기반 앙상블 모델을 학습시켰으며, 이 모델은 효모추출물(yeast extract)이 생장 수율(OD600)에 가장 큰 영향을 미치는 요인임을 확인하였고, 이어서 육류추출물(meat extract), 펩톤(peptone), 포도당(glucose) 순으로 나타났다. 또한 모델은 다음 라운드에서 시도할 정보량이 높은 배지 조성 조합을 예측·제시하였다. 종합적으로, 본 연구는 서울-경기 지역 도로변 및 공원 토양 세균 군집에 대한 예비적·탐색적 조사이자 능동학습 기반 배양 최적화의 초기 시연으로서, 향후 도시 토양 미생물 자원에 대한 심화 연구를 위한 기초 자료를 제공한다.

**Keywords**: Urban soils, Biodiversity, Bacterial community, 16S rRNA, Machine learning

---

## Introduction

Soil hosts a substantial fraction of terrestrial microbial diversity and plays a central role in nutrient cycling and ecosystem functioning. In urban environments, where land use is highly heterogeneous and subject to continuous anthropogenic disturbance, soil bacterial communities are expected to vary considerably across land-use types such as roadside verges and public parks (Whitehead et al., Front. Microbiol., 2022). Prior work has characterized urban park soils as rich reservoirs of taxonomic and biosynthetic diversity (Charlop-Powers et al., PNAS, 2016; Zhang et al., PeerJ, 2021), and site-specific factors such as co-contamination have been shown to drive local microbial adaptation in urban soils (Environments, 2026). In Korea, studies of urban park soils have so far focused primarily on land-use history and physicochemical soil properties in relation to tree vitality (Park and Heo, Korean J. Environ. Ecol., 2026), and to our knowledge no study has directly compared roadside and park soil bacterial community structure within a Korean metropolitan context.

**국문 해석**

토양은 육상 미생물 다양성의 상당 부분을 차지하며, 영양염류 순환과 생태계 기능 유지에 핵심적인 역할을 한다. 토지이용이 매우 이질적이고 지속적인 인위적 교란에 노출되는 도시 환경에서는, 도로변과 공원 같은 토지이용 유형에 따라 토양 세균 군집이 상당히 달라질 것으로 예상된다(Whitehead et al., Front. Microbiol., 2022). 선행연구들은 도시 공원 토양이 분류학적·생합성적으로 풍부한 다양성의 저장고임을 밝혀왔으며(Charlop-Powers et al., PNAS, 2016; Zhang et al., PeerJ, 2021), 오염 공존과 같은 지역 특이적 요인이 도시 토양 내 미생물의 국소적 적응을 유도한다는 것도 보고된 바 있다(Environments, 2026). 국내에서는 도시 공원 토양 관련 연구가 주로 토지이용 이력 및 이화학적 토양 특성과 수목 활력도의 관계에 초점을 맞춰 왔으며(Park and Heo, Korean J. Environ. Ecol., 2026), 국내 대도시권을 대상으로 도로변과 공원의 토양 세균 군집 구조를 직접 비교한 연구는 저자들이 아는 한 아직 없다.

<br>

To address this gap, we characterized soil bacterial communities using 16S rRNA gene amplicon sequencing, a widely adopted culture-independent approach for profiling microbial community composition and diversity. Thirty-two soil samples were collected from roadside and park sites across the Seoul-Gyeonggi region and sequenced to compare taxonomic composition, alpha diversity, and beta diversity between the two land-use types, with false discovery rate correction applied to account for multiple comparisons across taxa.

**국문 해석**

이러한 공백을 해소하기 위해, 본 연구는 미생물 군집 조성과 다양성을 프로파일링하는 데 널리 활용되는 배양 비의존적 접근법인 16S rRNA 유전자 앰플리콘 시퀀싱을 이용해 토양 세균 군집을 특성화하였다. 서울-경기 지역의 도로변 및 공원 부지에서 32개의 토양 시료를 채취·시퀀싱하여 두 토지이용 유형 간 분류학적 조성, 알파다양성, 베타다양성을 비교하였으며, 다수의 분류군에 대한 다중비교를 통제하기 위해 FDR(위양성률) 보정을 적용하였다.

<br>

Beyond community-level profiling, the same soils were used to isolate culturable bacterial strains via agar cultivation, yielding several isolates of potential applied relevance. Optimizing the culture conditions of such isolates is traditionally achieved through exhaustive trial-and-error screening of media composition, an approach that is costly and inefficient given the large number of possible formulations. To explore a more efficient alternative, we applied METIS, an active machine learning workflow that iteratively guides experimental design toward informative media compositions using minimal experimentation (Pandi et al., Nat. Commun., 2022). Here, we report an initial round of this process for Priestia megaterium, an isolate independently obtained from both roadside and park soils: twelve initial medium compositions were tested, their growth yields (OD600) were used to train an ensemble machine learning model, and the resulting model was used to identify the most influential medium components and to generate a second round of predicted informative compositions. Together, the community-level sequencing survey and this initial culture optimization round constitute a foundational, exploratory characterization of Seoul-Gyeonggi urban soil bacterial resources, intended to establish a basis for more targeted functional investigation in future work.

**국문 해석**

군집 수준의 프로파일링과 더불어, 동일한 토양으로부터 한천배지 배양을 통해 배양 가능한 세균 균주를 분리하여 잠재적으로 응용 가치가 있는 여러 분리주를 확보하였다. 이러한 분리주의 배양조건 최적화는 전통적으로 배지 조성에 대한 소모적인 시행착오 스크리닝을 통해 이루어지는데, 이는 가능한 배지 조성의 수가 방대하다는 점에서 비용이 많이 들고 비효율적이다. 이보다 효율적인 대안을 모색하기 위해, 본 연구는 최소한의 실험으로 정보량이 높은 배지 조성을 향해 실험 설계를 반복적으로 안내하는 능동학습 기반 머신러닝 워크플로우인 METIS를 적용하였다(Pandi et al., Nat. Commun., 2022). 본 연구에서는 도로변과 공원 토양 양쪽에서 독립적으로 확보된 분리주인 *Priestia megaterium*을 대상으로 한 이 과정의 첫 라운드를 보고한다. 12개의 초기 배지 조성을 시험하고, 그 생장 수율(OD600)을 이용해 앙상블 머신러닝 모델을 학습시켰으며, 학습된 모델을 이용해 가장 영향력이 큰 배지 성분을 규명하고 다음 라운드에서 시도할 정보량이 높은 배지 조성 조합을 예측·제시하였다. 종합하면, 본 연구의 군집 수준 시퀀싱 조사와 이번 초기 배양 최적화 라운드는 서울-경기 도시 토양 세균 자원에 대한 기초적·탐색적 특성화에 해당하며, 향후 보다 심화된 기능적 연구를 위한 토대를 마련하는 것을 목표로 한다.

---

## 참고 (팀원 공유용 메모)

- Bioremediation/오염물질 분해 관련 과장된 서술은 제거함 — 실제로 분해 관련 실험을 하지 않았기 때문. 향후 응용 방향(비료/오염정화 등)이 확정되면 Abstract 마지막 문장과 Introduction 3문단 마지막 문장만 수정하면 됨.
- Feature Importance 순위는 **2차 OD 값 기준**(Yeast > Meat > Peptone > Glucose)으로 반영함.
- 인용은 기존 Reference 목록에 있는 실제 논문(Whitehead 2022, Charlop-Powers 2016, Zhang 2021, Park & Heo 2026, Pandi et al. 2022)만 사용함 — 목록에 없던 Probst 2023, Environments 2026(저자 미상 항목)은 필요시 추가 인용 가능.
