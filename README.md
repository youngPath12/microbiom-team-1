# microbiom-team-1
# 수도권 도로변 및 공원 토양의 세균 군집 비교와 환경 모니터링 바이오마커 발굴

## 결론부터 정리하면

이번 프로젝트에는 서로 다른 두 분석이 포함되어 있습니다.

첫 번째는 채취 직후 동결한 토양에서 DNA를 직접 추출해 전체 세균 군집을 분석한 **16S amplicon NGS 분석**입니다. 이 자료는 도로변·도시 생활권 토양과 공원 토양의 세균 군집을 비교하고 환경별 바이오마커 후보를 찾는 데 사용합니다.

두 번째는 교육 중 각자가 선택한 토양을 MRS agar와 Nutrient agar에 배양한 뒤, 2nd streaking으로 분리한 균주의 **16S rRNA 유전자 염기서열 분석**입니다. 이 자료는 각 환경에서 배양 가능한 개별 균주를 확인하는 데 사용합니다.

```text
동결 토양 16S NGS
→ 전체 세균 군집 비교
→ 도로변·도시 생활권 토양과 공원 토양의 차이 확인
→ 환경 모니터링 바이오마커 후보 발굴

MRS/Nutrient agar 배양
→ 배양 가능한 균주 분리
→ 분리주별 16S rRNA 유전자 분석
→ 환경 유래 후보 균주 동정
```

---

# 1. NGS와 agar 배양 분석 비교

| 구분 | 토양 16S NGS | MRS/Nutrient agar 분리주 16S |
|---|---|---|
| 분석 목적 | 토양 전체 세균 군집 비교 | 배양 가능한 개별 균주 동정 |
| 사용 시료 | 채취 직후 동결한 토양 32개 | 각자 선택한 토양 시료 |
| 배양 여부 | 일반적으로 배양하지 않음 | 배지에서 배양함 |
| 분석 대상 | 토양에 포함된 다양한 세균의 DNA | 순수분리한 단일 colony |
| 분석 유전자 | 16S rRNA 유전자의 V4 등 일부 구간 | 거의 full-length 16S rRNA 유전자 |
| 분석 방법 | Illumina NGS | Sanger sequencing으로 추정 |
| 주요 결과 | 분류군별 상대풍부도와 군집 구조 | 가장 유사한 균종·균주명 |
| 핵심 자료 | `taxonomic(1조).xlsx` | MRS/Nutrient 폴더의 `.seq`, `.fas` |
| 연구에서의 역할 | 군집 비교 및 바이오마커 발굴 | 배양 가능한 후보 균주 제시 |

여기서 흔히 말하는 “16S rRNA 분석”은 RNA 자체를 분석한 것이 아니라, 세균 DNA에 존재하는 **16S rRNA 유전자**를 분석한 것입니다.

---

# 2. 토양 16S NGS 분석

## 2.1 NGS 실험 목적과 시료 구성

NGS 분석의 목적은 토양에서 배양되는 균 몇 종을 확인하는 것이 아니라, 배양 여부와 관계없이 토양 전체 세균 군집의 구성을 비교하는 것입니다.

1조의 NGS 분석에는 총 32개 토양 시료가 사용되었습니다.

| NGS 번호 | Code | 토양 샘플링 번호 | NGS 그룹 | 실제 환경 구분 |
|---:|---|---|---|---|
| 1 | Sl-1-1 | 1-1 | Soil | 도로변·도시 생활권 토양 |
| .. | .. | .. | .. | .. |
| 17 | Sl-1-17 | 1-17 | Soil | 도로변 화단 토양 |
| 18 | Sl-2-1 | 2-1 | Park_Soil | 뚝섬한강공원 토양 |
| .. | .. | .. | .. | .. |
| 32 | Sl-2-15 | 2-15 | Park_Soil | 보라매공원 토양 |

코드 연결 방법은 다음과 같습니다.

```text
Sl-1-1  ↔ 토양 샘플링 번호 1-1 ↔ NGS 번호 1
Sl-1-17 ↔ 토양 샘플링 번호 1-17 ↔ NGS 번호 17

Sl-2-1  ↔ 토양 샘플링 번호 2-1 ↔ NGS 번호 18
Sl-2-15 ↔ 토양 샘플링 번호 2-15 ↔ NGS 번호 32
```

따라서 최종 비교 그룹은 다음과 같습니다.

- `Soil`, n=17: 도로변·주차장·공사장·도심 화단 등 도시 생활권 토양
- `Park_Soil`, n=15: 한강공원·어린이대공원·근린공원 등 공원 토양

엄밀하게는 `Soil` 17개 모두가 도로 바로 옆에서 채취된 것은 아닙니다. 따라서 연구에서는 `Soil`을 **도로변 및 도시 생활권·교란 토양**으로 정의하는 것이 정확합니다.

## 2.2 NGS 실험 방법

### ① 토양 채취 및 동결

```text
도로변·도시 생활권 및 공원 토양 채취
→ 채취 직후 동결
→ DNA 추출 전까지 저온 보관
```

동결은 채취 후 일부 세균이 증식하거나 사멸하여 현장의 군집 구성이 달라지는 것을 최소화하기 위한 과정입니다.

### ② 토양 전체 DNA 추출

```text
일정량의 동결 토양 사용
→ bead-beating으로 세균 세포 파쇄
→ 토양 입자 제거
→ 휴믹산·단백질 등 PCR 저해물질 제거
→ 실리카 컬럼에 DNA 결합
→ 세척
→ DNA 용출
```

NGS용 시료는 배지에 배양한 후 분석한 것이 아니라, 동결 토양에서 전체 DNA를 직접 추출했을 가능성이 높습니다.

### ③ DNA 농도와 순도 확인

추출 DNA에 대해 일반적으로 다음 항목을 확인합니다.

- DNA 농도
- A260/A280: 단백질 오염 정도
- A260/A230: 휴믹산·염 등 오염 정도

### ④ 16S rRNA 유전자 PCR

토양 전체 DNA 중 세균의 16S rRNA 유전자 일부 구간을 PCR로 증폭합니다.

강의자료에는 515F와 806R primer를 사용한 V4 구간 분석이 제시되어 있습니다. V4 구간은 약 292 bp입니다.

### ⑤ Index 및 adapter 부착

여러 토양 시료를 한 번에 sequencing하기 위해 각 시료에 서로 다른 index를 부착합니다.

```text
Sl-1-1 → 고유 index
Sl-1-2 → 다른 index
...
Sl-2-15 → 고유 index
```

Sequencing이 끝나면 index를 이용해 read를 다시 시료별로 분리합니다.

### ⑥ Illumina NGS

증폭된 16S DNA를 Illumina 장비로 대량 sequencing합니다. 한 시료에서도 다양한 세균에 해당하는 많은 수의 16S read가 생성됩니다.

### ⑦ 생물정보학 처리

일반적인 처리 과정은 다음과 같습니다.

```text
FASTQ 생성
→ 시료별 read 분리
→ 낮은 품질 read 제거
→ primer·adapter 제거
→ forward/reverse read 병합
→ chimera 제거
→ ASV 또는 OTU 생성
→ reference database와 비교
→ taxonomy 지정
→ 분류군별 상대풍부도 계산
```

현재 `ngs` 폴더에는 최종 taxonomy 결과가 중심이므로, 외주업체가 사용한 프로그램, database, filtering 조건 등은 정확히 확인하기 어렵습니다.

## 2.3 NGS 데이터 설명

### `ngs/Sample Information.xlsx`

NGS 번호와 토양 시료 코드를 연결합니다.

| 열 | 의미 |
|---|---|
| `Sl. No.` | NGS 결과의 시료 번호 1~32 |
| `Code` | `Sl-1-1`, `Sl-2-1` 등의 시료 코드 |
| `Source Information` | Soil-1, Soil-2 등 출처 구분 |
| `Type` | 시료 유형 |
| `Project` | 프로젝트 정보 |

### `토양 샘플링 정보.xlsx`

각 시료의 실제 채취정보를 제공합니다.

- 샘플링 번호
- 채취자
- 채취 위치
- 채취시간
- 특이사항

### `ngs/taxonomic(1조).xlsx`

1조 토양 32개 시료의 분류군별 상대풍부도 자료입니다.

| 시트 | 분류 단계 |
|---|---|
| `phylum` | 문 |
| `Class` | 강 |
| `Order` | 목 |
| `Family` | 과 |

각 열은 NGS 시료 번호 1~32이고, 각 행은 세균 분류군입니다.

예를 들어 `Actinobacteriota = 0.535`라면 해당 시료의 분류된 16S read 중 약 53.5%가 Actinobacteriota에 해당한다는 뜻입니다. 이는 절대 세균 수가 아니라 **상대풍부도**입니다.

---

# 3. Agar 배양 및 분리주 16S 분석

## 3.1 Agar 실험 목적

Agar 실험은 토양 전체 군집을 분석한 것이 아니라, 토양에서 실험실 조건으로 배양 가능한 세균을 분리하고 동정하기 위한 실험입니다.

각자가 선택한 토양을 MRS agar와 Nutrient agar에 배양한 후, 2nd streaking으로 순수분리한 colony의 16S rRNA 유전자 염기서열을 분석했습니다.

## 3.2 Agar 실험 방법

```text
각자의 토양 시료 중 하나 선택
→ 토양 현탁액 준비
→ MRS agar 및 Nutrient agar에 접종
→ colony 배양
→ colony 선택
→ 새로운 배지에 2nd streaking
→ 단일 colony 확보
→ 분리주 DNA 추출
→ 16S rRNA 유전자 PCR
→ Sanger sequencing
→ NCBI BLAST
→ 가장 가까운 균종 확인
```

### MRS agar

유산균 분리에 많이 이용되는 비교적 선택적인 배지입니다. 다만 MRS에서 자랐다고 해서 반드시 유산균인 것은 아닙니다.

### Nutrient agar

다양한 비영양요구성 세균이 자랄 수 있는 일반 배지입니다.

## 3.3 도로변·도시 생활권 토양의 2nd streaking 결과

| Code | 채취 환경 | MRS agar | Nutrient agar |
|---|---|---|---|
| LJM | 도로변 토양 | - | - |
| YSJ | 도로변·주차장 인근 토양 | - | **YSJ**: *Mycoplana dimorpha* strain DSM 7138 |
| HIY | 공사장·주차장 등 도시 생활권 토양 | **HIY2**: *Priestia megaterium* strain DSM 32 | **HIY**: *Bacillus stratosphericus* strain T-7.3 |

- LJM: 이지민의 도로변 토양 시료
- YSJ: 윤수진의 도로변·주차장 인근 토양 시료
- HIY: 황인영의 공사장·주차장 등 도시 생활권 토양 시료

다만 각 사람이 보유한 여러 시료 중 정확히 어느 번호를 agar 배양에 사용했는지는 현재 표에 기록되어 있지 않습니다. 따라서 사람별 환경 범위까지만 연결할 수 있습니다.

## 3.4 공원 토양의 2nd streaking 결과

| Code | 채취 환경 | MRS agar | Nutrient agar |
|---|---|---|---|
| HJG | 충효어린이공원·보라매공원 토양 | **HJG2**: *Bacillus altitudinis* strain B6 | **HJG**: *Bacillus safensis* strain 1P09SA |
| SR | 광명시 현충근린공원 토양 | - | **SR**: *Paenibacillus alvei* isolate Paenibacillus B-LR1 |
| JYR | 뚝섬한강공원·어린이대공원 토양 | **JYR2**: *Priestia megaterium* strain IS4_1 | **JYR**: *Paenibacillus thiaminolyticus* strain PATH554 |

- HJG: 함정관의 공원 토양 시료
- SR: 주세린의 공원 토양 시료
- JYR: 정유라의 공원 토양 시료

마찬가지로 각 사람이 채취한 여러 공원 시료 중 정확히 어느 번호를 배양에 사용했는지는 별도의 배양 기록이 필요합니다.

## 3.5 환경별 분리주 요약

| 환경 구분 | MRS agar | Nutrient agar | 총 분리주 |
|---|---:|---:|---:|
| 도로변·도시 생활권 토양 | 1 | 2 | 3 |
| 공원 토양 | 2 | 3 | 5 |
| 합계 | 3 | 5 | 8 |

### 도로변·도시 생활권 토양 유래

- *Priestia megaterium*
- *Mycoplana dimorpha*
- *Bacillus stratosphericus*

### 공원 토양 유래

- *Bacillus altitudinis*
- *Bacillus safensis*
- *Paenibacillus alvei*
- *Priestia megaterium*
- *Paenibacillus thiaminolyticus*

이 분류는 각 코드에 해당하는 채취자가 채취한 토양의 환경 유형을 기준으로 한 것입니다. 정확한 배양 시료 번호가 없으므로 특정 채취지점까지 단정해서 연결하지 않는 것이 안전합니다.

## 3.6 Sequence 파일의 의미

MRS 및 Nutrient 폴더에 있는 `.seq`와 `.fas` 파일은 분리주의 16S rRNA 유전자 염기서열입니다.

```text
>JYR
GCTCAGGACGAACGCTGGCGGCGTGCCTAATACATGCA...
```

- `>` 다음 문자열: sequence ID
- A, T, G, C 문자열: 실제 16S rRNA 유전자 염기서열
- `_align`: sequencing 결과를 정리하거나 forward/reverse read를 조합한 서열로 추정

대부분 약 1,400~1,500 bp이므로 짧은 NGS read가 아니라 거의 full-length 16S Sanger sequence로 판단됩니다.

## 3.7 BLAST 결과의 의미

분리주의 sequence는 NCBI database의 reference sequence와 비교하여 동정합니다.

주요 지표는 다음과 같습니다.

- Percent identity
- Query coverage
- Alignment length
- E-value
- Bit score

표에 적힌 strain 이름은 분리주와 가장 가까운 reference입니다. 따라서 논문에서는 다음처럼 표현하는 것이 안전합니다.

> JYR2 was most closely related to *Priestia megaterium* strain IS4_1 based on 16S rRNA gene sequence analysis.

즉, JYR2가 IS4_1과 가장 가까웠다는 뜻이지, 분리주가 정확히 기존의 IS4_1 strain과 동일하다는 뜻은 아닙니다.

---

# 4. 환경 모니터링 바이오마커 발굴 방향

## 4.1 NGS 기반 바이오마커 발굴

환경 바이오마커는 32개 토양 NGS 자료를 중심으로 발굴해야 합니다.

```text
Soil 17개
vs.
Park_Soil 15개
```

다음 조건을 만족하는 분류군이 우선적인 후보가 될 수 있습니다.

1. 특정 환경에서 반복적으로 검출됨
2. 반대 그룹보다 상대풍부도가 일관되게 높음
3. 통계적으로 유의한 차이가 있음
4. 한 시료에만 집중되지 않음
5. 서로 다른 채취자와 장소에서도 같은 경향이 나타남

## 4.2 Agar 분리주의 활용

Agar 결과는 NGS 바이오마커 분석을 보완하는 자료로 활용합니다.

예를 들어 NGS 분석에서 공원 토양의 `Bacillales` 또는 `Paenibacillaceae`가 유의하게 증가했다면, 공원 토양에서 분리된 다음 균주들과 연결해 해석할 수 있습니다.

- *Bacillus altitudinis*
- *Bacillus safensis*
- *Paenibacillus alvei*
- *Paenibacillus thiaminolyticus*
- *Priestia megaterium*

다만 배양 분리주는 배지와 배양조건의 영향을 크게 받기 때문에, 한 번 분리됐다는 사실만으로 환경 바이오마커라고 결론 내릴 수는 없습니다.

---

# 5. 최종 연구 구성

## 연구 질문

> 도로변·도시 생활권 토양과 공원 토양의 세균 군집은 서로 다른가?  
> 두 환경을 구분할 수 있는 세균 분류군이 존재하는가?  
> 각 환경에서 배양 가능한 후보 균주를 확보할 수 있는가?

## 분석자료

```text
주 분석
16S amplicon NGS
├─ Soil: 17개
└─ Park_Soil: 15개

보조 분석
2nd streaking 분리주 8개
├─ 도로변·도시 생활권 유래: 3개
└─ 공원 유래: 5개
```

## 최종 해석 방향

> 채취 직후 동결한 32개 토양 시료의 16S amplicon NGS 결과를 이용해 도로변·도시 생활권 토양과 공원 토양의 세균 군집을 비교하고, 환경별로 차등적으로 나타나는 분류군을 환경 모니터링 바이오마커 후보로 탐색한다. 이와 함께 MRS 및 Nutrient agar에서 2nd streaking으로 순수분리한 8개 균주의 16S rRNA 유전자 염기서열을 분석하여, 각 환경에서 배양 가능한 후보 균주와 NGS 바이오마커의 연계 가능성을 검토한다.
