# ePTM_v2
향상된 입자 추적 모델 버전 2.0(Enhanced Particle Tracking Model Version 2.0, ePTM v2)은 강에서 산란하고 바다에서 성숙하는 어류가 바다로 이동하는 과정을 시뮬레이션하는 전산 모델입니다. 일종의 합성 어류 원격 측정 시스템으로, 과학자, 실무자, 자연자원 관리자가 어류 같은 수생 생물이 환경을 어떻게 이용하는지 이해하도록 돕습니다.

![Alt Text](https://github.com/cvclcm/ePTM_v2/blob/main/SupportingMaterial/animOutputCropped_1.gif)

모델의 설치, 사용법, 구조에 관한 정보는 모델 Wiki 페이지에서 확인할 수 있습니다: [링크](https://github.com/cvclcm/ePTM_v2/wiki).

모델에 관한 질문은 *vamsi.sridharan@noaa.gov* 로 문의해 주세요.


# 목차
1. 소개
2. 활용 사례
3. 배경
4. 컴퓨팅 요구 사항
6. 프로젝트 정보

# 소개
이 모델은 구체성의 정도에 따라 여러 방식으로 설명할 수 있습니다.

이 모델은 의사결정 지원 도구입니다. 기후 변화, 해수면 상승, 노후 기반 시설, 토지 이용 및 토지 피복 변화 같은 불확실성 아래에서 다목적 지표수 시스템의 물 공급에 관한 관리 및 정책 결정을 내리는 데 필요한 과학적 통찰을 제공하는 데 현재 쓰이고 있고, 앞으로도 쓰일 수 있습니다.

또한 에이전트 기반 모델의 한 유형으로, 시뮬레이션 어류는 일련의 행동 규칙, 즉 "에이전트"에 따라 주변 수생 환경과 상호작용합니다.

개체 기반 모델이기도 합니다. 넓은 공간 영역에 걸쳐 시뮬레이션 어류 개체 각각의 이동 생활사를 추적합니다. 다만 현재 형태의 모델은 어군 내 상호작용이나 포식자와 피식자 간의 상호작용을 명시적으로 시뮬레이션하지 않습니다.

이 모델은 과정 기반 모델, 즉 기계론적 모델로서 실제 어류 관찰에 근거한 물리적·생물학적 제약 조건을 사용해 시뮬레이션 어류의 움직임을 결정합니다. 구체적으로는 입자 추적 모델 또는 확률적 랜덤 워크 모델로, 지역 환경(여기서는 물의 흐름과 하루 중 시간)의 영향을 받는 시뮬레이션 어류의 자기 상관적 움직임을 시뮬레이션합니다.

마지막으로, 이 모델은 완전히 데이터 기반입니다. 복잡한 환경을 통과하는 동물의 이동은 본질적으로 여러 규모에 걸친 과정이며, 이 모델의 이동 규칙을 정할 때 어류 이동에 관해 활용할 수 있는 최선의 음향 원격 측정 데이터를 사용했습니다.

**새크라멘토강의 수로 분기점에서 음향 태그를 부착한 치누크 연어의 고해상도 궤적과 태그를 부착한 포식자에 대한 노출을 분석해 중간 규모 행동을 연구하고 시뮬레이션 어류의 행동 규칙을 정할 수 있습니다**

![Image](https://github.com/cvclcm/ePTM_v2/blob/main/SupportingMaterial/Tracks2011.jpg) 

**새크라멘토강에서 바다 쪽으로 이동하는 음향 태그 부착 치누크 연어의 이동 통계에서 거시적 패턴이 드러나며, 이 패턴을 사용해 모델의 행동 파라미터를 보정합니다**

![Image](https://github.com/cvclcm/ePTM_v2/blob/main/SupportingMaterial/MigrationRateVsReach.jpg)

이 모델은 연어과 어류의 이동에 관한 과학 문헌의 철저한 검토, 이해관계자 참여, 독자적인 과학 연구를 통해 개발되었습니다.
어류의 중간 규모 이동 행동에 관한 기초적인 발견은 Olivetti et al. (2021)의 최근 논문에 보고했습니다: [링크](https://besjournals.onlinelibrary.wiley.com/doi/full/10.1111/2041-210X.13604?casa_token=R24dUwXx0FcAAAAA%3AUYvokc4iyCMO8VpBF6dAU1C306BvSg4r60KTjsV27wgm0W_EKtuH34H8F6m2rLX9-JD7LwvseX23zHU).
하구를 통과하는 연어과 어류 이동의 거시적 표현은 Sridharan et al. (2019)의 최근 논문에서 개발하여 보고했습니다: [링크](https://agupubs.onlinelibrary.wiley.com/doi/full/10.1029/2019WR025429?casa_token=8r5Cth5KgH4AAAAA%3Ac67189ZeR9wPmDHx6iglofh9-e719x0znYf-Svraj5li_6-EaScwiWG_0wH9kZhUKxO3HX8V31_CEfo).

## ePTM v2가 필요한 이유
이동성 수생 동물이 복잡한 지표수 환경을 어떻게 헤쳐 나가는지 이해하려는 과학자이거나, 새로운 수자원 사업이나 정책이 지역 야생 생물에 어떤 영향을 미칠지 알아야 하는 물 관리자 또는 보전 정책 결정자라면, 상황을 파악하는 데 도움이 되는 민첩하고 신뢰할 수 있는 모델이 필요합니다. 음향 원격 측정 프로그램은 운영 비용이 많이 들고 단편적인 데이터만 제공합니다. 데이터 범위가 제한된 지역에서는 무슨 일이 일어나고 있을까요? 유량 같은 거시적 시스템 변수와 생존율을 연결하는 통계 모델이면 충분하다고 생각할 수도 있습니다. 하지만 기후 변화 등에서 비롯된 불확실성으로 그 통계적 관계가 바뀌면 어떻게 될까요? ePTM 같은 기계론적 모델은 새로운 환경 조건에서 어류에게 어떤 일이 일어날 가능성이 높은지 예측할 수 있습니다. 아래는 모델 검증 결과(보정 시 제외해 둔 데이터와 모델 성능의 비교)입니다.

**새크라멘토-샌와킨 델타의 겨울 회귀군 치누크 연어 치어에 대한 유량-생존율 관계의 표본 외 검증. 낮은 편향(<0.1), 음향 원격 측정 데이터와 모델 결과의 95% 신뢰구간이 상당히 겹치는 점, 모델이 기능적으로 유사한 관계를 재현한 점은 모델의 가치를 보여줍니다**

![Image](https://github.com/cvclcm/ePTM_v2/blob/main/SupportingMaterial/FlowvsSurvival.jpg)

**창조와 낙조 동안 여러 이동 경로로 유입될 확률에 대한 표본 외 검증은 모델이 이동 경로를 잘 예측함을 보여줍니다. 2014년에는 Georgiana Slough 입구에 부유식 어류 유도 구조물이 설치되었으며, 모델에서는 이를 근사적으로만 표현했습니다**

![Image](https://github.com/cvclcm/ePTM_v2/blob/main/SupportingMaterial/RoutingProbabilties.jpg)

# 활용 사례
현재 ePTM v2는 중부 캘리포니아의 새크라멘토-샌와킨 델타를 지나 샌프란시스코만과 태평양으로 이동하는 치누크 연어 치어에 대해 보정되어 있습니다. 강의 흐름과 조석 흐름에 대한 수치 시뮬레이션이 필요하며, 이는 캘리포니아 수자원부(DWR)의 델타 시뮬레이션 모델(DSM2) v.8.1.2가 제공합니다. DSM2는 DWR에서 호스팅합니다: [링크](https://github.com/CADWRDeltaModeling/dsm2).

**새크라멘토강 겨울 회귀군 치누크 연어 스몰트의 이동에 적용한 ePTM v2 모델 영역(새크라멘토강 강조 표시)**

![Image](https://github.com/cvclcm/ePTM_v2/blob/main/SupportingMaterial/DeltaMapwSacRiver.jpg)

캘리포니아 센트럴밸리에서 이 모델은 새크라멘토-샌와킨 델타에서 성장하고 이동하는 멸종 위기종인 새크라멘토강 겨울 회귀군 치누크 연어 치어의 월별 생존율 추정치를 제공하는 데 사용됩니다. 이 추정치는 더 큰 규모의 단계 구조 개체군 동태 모델인 센트럴밸리 생활사 모델(Central Valley Life Cycle Model)에 입력됩니다: [링크](https://oceanview.pfeg.noaa.gov/wrlcm/intro).

현재 이 모델은 다른 수리 엔진과 함께 실행하도록 구성되어 있지 않습니다. 따라서 대상 수계에 ePTM v2와 DSM2를 이용한 모델을 개발하려면, 모델 격자를 만들고, 이 격자로 유량을 예측하는 DSM2 모델을 보정 및 검증한 뒤, ePTM v2에 사용할 수리 출력을 생성해야 합니다.

![Image](https://github.com/cvclcm/ePTM_v2/blob/main/SupportingMaterial/SacRiverWinterRunChinookSmolt_SiskoJakeUSFWS.jpg)

**새크라멘토강 겨울 회귀군 치누크 연어 스몰트(사진 제공: Jake Sisko, 미국 어류·야생동물관리국)**

마찬가지로 현재 모델에는 연어과 치어의 이동을 시뮬레이션하는 데 적합한 행동 규칙이 들어 있으며, Perry (2010; 2012)가 발표한 데이터를 사용해 캘리포니아 새크라멘토강 및 샌와킨강 수계의 치누크 연어에 대해 보정 및 검증되었습니다 [링크](https://github.com/cvclcm/ePTM_v2/tree/main/documentation). 모델 보정 및 검증에 관한 자세한 설명은 여기에서 볼 수 있습니다: [링크](https://github.com/cvclcm/ePTM_v2/blob/main/SupportingMaterial/Sridharan_ePTMv2_CalibrationValidation_Dec1_2021.pptx). 모델 구축, 보정, 검증에 사용한 모든 데이터는 다음 위치에 보관되어 있습니다: [링크](https://github.com/cvclcm/ePTM_v2/tree/main/Data).

소스 코드를 수정해 행동 규칙을 바꾸고, 다른 수계의 다른 어종에 맞게 모델을 다시 보정할 수 있습니다. 방법은 Wiki 페이지를 참고하세요: [링크](https://github.com/cvclcm/ePTM_v2/wiki).

# 배경
ePTM v2는 중립 부력의 수동적 추적자를 시뮬레이션하는 DWR의 입자 추적 모델(PTM)에 행동 클래스를 추가한 것입니다. PTM과 ePTM v2 모두 물의 유속과 수심을 시뮬레이션할 수리 엔진이 필요합니다. 이 역할은 1차원 천수 방정식 솔버인 DSM2 모델이 담당합니다. 모델에 대한 상세한 동료 검토는 Sridharan et al. (2018)의 논문에서 볼 수 있습니다 [링크](https://escholarship.org/uc/item/0vm955tw). ePTM v2는 PTM의 기반 랜덤 워크 알고리즘을 수치적으로 크게 개선한 내용도 포함하며, 대부분은 Sridharan et al. (2017)의 논문에 자세히 설명되어 있습니다 [링크](https://ascelibrary.org/doi/full/10.1061/(ASCE)HY.1943-7900.0001399?casa_token=yMf5O160xyoAAAAA:v891cN9CzxrTTSQBoi3FTBvZsfMKstbIU1Et8QPf5Dh6dHIJsE-wh8eotqCa2S-8X-MV4hgnXA).

ePTM v2는 수생 동물 이동의 세 가지 측면을 시뮬레이션합니다. 유영에 의한 이동, 조석과 물 운영으로 인한 흐름 역전이 있는 상황에서 강의 지류와 수로 분기점 같은 복잡한 지형을 통과하는 경로 선택, 그리고 포식에 의한 폐사입니다. 이 메커니즘의 세부 사항은 여기서 다루기에는 너무 복잡하지만, 모델 구조에 대한 간략한 설명은 다음에서 볼 수 있습니다: [링크](https://oceanview.pfeg.noaa.gov/wrlcm/documents/documentation/ePTM_Study_Plan_FINAL_02_12_2021_v2.pdf). 이 문서에는 모델이 다양한 물 관리 분야에서 어떻게 계속 활용되고 있고 활용될 수 있는지에 관한 정보도 들어 있습니다. 모델의 생물학적 행동 규칙을 설명하는 동영상도 제작했습니다: [링크](https://github.com/cvclcm/ePTM_v2/blob/main/SupportingMaterial/Sridharan_04_09_21_BDSC_ePTMV2Description.mp4).

## 모델 개발 과정
이 과학 모델은 다음 핵심 원칙에 따라 개발되었습니다.
1. 여러 이동 규모에 걸쳐 자기 일관성을 유지하고 현실적인 물리적·생물학적 과정을 포함한다.
2. 수십 년 규모의 개체군 동태 시뮬레이션에 활용할 만큼 민첩하고 확장 가능해야 하며, 최신 수준의 수치적 정교함과 현실성을 유지하면서 단독으로든 더 큰 모델링 생태계의 구성 요소로든 사용할 수 있을 만큼 다재다능해야 한다.
3. 양질의 과학 데이터, 다학제적 개발팀, 캘리포니아 센트럴밸리의 분야별 전문가 및 의사결정자를 포함한 이해관계자의 적극적인 참여를 활용하는 다기관 협력을 통해 포용적이고 투명한 과정으로 만들어져야 한다.

ePTM v1.0은 미국 국립해양어업청, 캘리포니아 어류·야생동물국, 미국 지질조사국의 협력으로 개발되었습니다. 이 모델은 DSM2-PTM 모델을 확장해 다양한 어류 행동을 포함했습니다. 이후 치누크 연어 생활사 모델에 통합하고 이해관계자 참여를 통해 모델 아이디어를 논의하도록 미국 개간국이 아낌없이 지원했고, 연어과 어류의 이동을 보다 근본적으로 이해하기 위해 여러 규모의 데이터 출처를 결합하도록 캘리포니아 어류·야생동물국이 지원했습니다. 이러한 지원으로 흐름 표현을 개선하고 행동 모델을 더 현실적으로 만든 버전 2.0이 개발되었습니다.

모델 개발 과정 전반에 걸쳐 지역, 광역, 연방 차원의 이해관계자들과 소통했습니다. 소통 이력과 모델로 다양한 단계에서 평가 중인 이해관계자 제공 행동 가설은 다음 페이지의 *Workshops* 및 *Stakeholder shared documents* 탭에서 각각 확인할 수 있습니다: [링크](https://oceanview.pfeg.noaa.gov/wrlcm/resources).

# 컴퓨팅 요구 사항
ePTM v2는 Java로 개발되었으며 현재 Windows PC에서만 실행됩니다. 단일 실행 또는 배치 모드로 실행할 수 있습니다. 모델을 실행하려면 다음 하드웨어와 소프트웨어가 필요합니다.

1. 수개월 또는 수년 분량의 시뮬레이션에는 고성능 노트북이나 데스크톱, 수십 년 분량의 시뮬레이션에는 고성능 컴퓨터, 클러스터 또는 클라우드 컴퓨팅
2. 모델을 컴파일하고 실행하기 위한 32비트 Java 런타임 환경 8 [![Image](https://img.shields.io/badge/java-jre%201.8-blue)](https://www.java.com/download/ie_manual.jsp)
3. 모델 소스 코드 작업을 위한 Eclipse 같은 IDE 환경. 아래 설치 안내에서 Eclipse 사용법을 제공합니다: [링크](https://www.eclipse.org/downloads/packages/release/juno/sr1/eclipse-ide-java-ee-developers). **32비트 Java 8을 지원하는 IDE가 필요합니다.** 다음에서 구할 수 있습니다 [![Image](https://img.shields.io/badge/Eclipse-4.16-red)](https://archive.eclipse.org/eclipse/downloads/).
4. 모델의 수리 엔진인 DSM2 [![Image](https://img.shields.io/badge/DSM2-v.8.1.2-orange)](https://water.ca.gov/Library/Modeling-and-Analysis/Bay-Delta-Region-models-and-tools/Delta-Simulation-Model-II) 
5. 수리 모델 결과를 시각화하고 다루기 위한 HDF5 도구 모음: [링크](https://www.hdfgroup.org/solutions/hdf5/)
6. Notepad++ 같은 좋은 텍스트 편집기: [링크](https://notepad-plus-plus.org/downloads/)
7. 모델 실행 및 출력 처리 스크립트를 작성하기 위한 Python 3.x [![Image](https://img.shields.io/badge/python-3.7%2B-green)](https://www.python.org/downloads/) 
8. 모델 결과 분석을 위한 R 4.x 또는 Matlab 2018b 이상 [![Image](https://img.shields.io/badge/R-4.0%2B-yellow)](https://www.r-project.org/)
9. DSM2의 수리 시계열을 시각화하기 위한 HEC-DssVue: [링크](https://www.hec.usace.army.mil/software/hec-dssvue/)
10. DSM2 격자를 시각화하기 위한 Google Earth Pro(캘리포니아 센트럴밸리에서 맞춤형으로 모델을 구성할 때 매우 유용합니다): [링크](https://www.google.com/earth/download/gep/agree.html?hl=en-GB)

# 프로젝트 정보
프로젝트의 최신 소식, 정보, 현황을 여기에 업데이트합니다.

## 소식

![Image](https://img.shields.io/static/v1?label=01/12&message=2022&labelColor=lightgrey&color=brightgreen&style=plastic&logo=githubactions)
ePTM 모델 개발을 일부 지원한 Proposition 1 프로젝트에 관해 캘리포니아 어류·야생동물국에 제출한 최종 보고서를 다음에서 볼 수 있습니다: [링크](https://github.com/cvclcm/ePTM_v2/blob/main/documentation/Raimondi_et_al_2021_CDFW_FinalReport.pdf). 다음과 같이 인용해 주세요.

Raimondi, P., Hein, A., Sridharan, V., Danner, E., and Lindley, S. 2021. A next-generation model of juvenile salmon migration through the Sacramento-San Joaquin Delta. Report submitted by the University of California Santa Cruz to the California Department of Fish and Wildlife Under Proposition 1 Scientific Award No. P1896007. Sacta Cruz, CA. 55p. Available at: https://github.com/cvclcm/ePTM_v2/blob/main/documentation/Raimondi_et_al_2021_CDFW_FinalReport.pdf.  

![Image](https://img.shields.io/static/v1?label=12/01&message=2021&labelColor=lightgrey&color=brightgreen&style=plastic&logo=githubactions)
ePTM v2는 현재 캘리포니아 델타 수로 사업(Delta Conveyance Project)에서 기후 변화와 해수면 상승 조건의 대체 물 관리 운영안을 평가하는 데 사용되고 있습니다.

![Image](https://img.shields.io/static/v1?label=01/06&message=2022&labelColor=lightgrey&color=brightgreen&style=plastic&logo=githubactions)
ePTM v2가 Maven's Notebook의 기사에 소개되었습니다.

https://mavensnotebook.com/2022/01/06/bay-delta-science-conference-applying-the-winter-run-life-cycle-model-to-pressing-hydromanagement-questions-in-the-central-valley/

![Image](https://img.shields.io/static/v1?label=06/28&message=2022&labelColor=lightgrey&color=brightgreen&style=plastic&logo=githubactions)
*scripts* 폴더의 ePTMCreatInputs.R과 Wiki의 사용법에 여러 수정 사항이 있으니 참고해 주세요. 체크포인트를 지정할 때는 DSM2 *외부(External)* 노드 번호를 사용하세요. 이전에는 *내부(Internal)* 노드 번호로 잘못 안내했습니다. 또한 ePTM을 실행하는 기본 스크립트와 Wiki의 설명을 단순화했습니다.

![Image](https://img.shields.io/static/v1?label=06/28&message=2022&labelColor=lightgrey&color=brightgreen&style=plastic&logo=githubactions)
ePTM v2는 캘리포니아 SITES 저수지 사업에서 기후 변화와 해수면 상승 조건의 대체 물 관리 운영안을 평가하는 데 사용될 예정입니다.

![Image](https://img.shields.io/static/v1?label=07/05&message=2022&labelColor=lightgrey&color=brightgreen&style=plastic&logo=githubactions)
2022년 6월 30일 이해관계자 참여 워크숍의 모든 교육 동영상을 documentation/videos 섹션에서 볼 수 있습니다.

![Image](https://img.shields.io/static/v1?label=07/13&message=2022&labelColor=lightgrey&color=brightgreen&style=plastic&logo=githubactions)
이 readme 파일이 수정되었으니 참고해 주세요. 이전에는 ePTM 실행에 DSM2 v8.1.2가 필요하다고 잘못 안내했습니다. 사전 요구 사항만 갖추면 ePTM은 어떤 DSM2 버전으로도 실행할 수 있습니다.

## 향후 개선 사항
모델은 다음과 같은 부분에서 개선될 수 있습니다.

1. 어류는 단순히 흐름을 따라가는 수동적 입자처럼 행동하지 않는다는 점이 알려져 있습니다(Sridharan et al. 2017; Hance et al. 2020; Gross et al. 2021). 따라서 횡방향 및 연직 방향의 랜덤 워크 이동은 시뮬레이션 어류가 난류 와류에 단순히 밀려다니게 하는 대신, 지역 흐름을 조건으로 하는 관측 분포에서 값을 추출하도록 업데이트해야 합니다.
2. 복잡한 지표수 시스템에서 이동이 주로 2차원 및 3차원으로 나타나는 곳과 1차원 하천 방향 이동 시뮬레이션으로 충분한 곳을 연구하면 효율을 더 높일 수 있습니다.
3. 실험실 수조에서 얻은 실험 유동 프로파일 대신, 다양한 흐름 조건에서 모델의 실제 물리적 영역 내에서 측정한 유속을 사용합니다.
4. 64비트 Java를 사용하도록 코드를 업데이트합니다.
5. Linux와 Mac에서도 실행되도록 코드를 업데이트합니다.
6. 기반 수리 엔진과 독립적으로 실행할 수 있도록 코드와 DSM2의 연동 방식을 업데이트합니다.

이러한 개선에 기여하고 싶으신가요? 그렇다면 코드를 본인의 저장소로 포크하고, 안정적인 빌드가 준비되면 풀 리퀘스트를 보내 주세요.

## 핵심 팀
핵심 모델 개발팀은 다음 기관으로 구성됩니다.

캘리포니아 대학교 샌타크루즈 캠퍼스: 해양과학연구소

미국 국립해양대기청 국립해양어업청 남서부 수산과학센터

미국 지질조사국 서부 수산연구센터

QEDA Consulting, LLC.

## 감사의 말
핵심 모델링 팀은 프로젝트에 크게 기여한 캘리포니아 수자원부에 감사드립니다.

프로젝트에 기술 및 프로그램 지원을 제공한 캘리포니아 대학교 샌타크루즈 캠퍼스의 수산 협력 프로그램(Fisheries Cooperative Program)에도 감사드립니다.

모델 구축, 보정, 검증에 사용한 데이터를 제공한 미국 지질조사국, 캘리포니아 어류·야생동물국, 미국 어류·야생동물관리국, 시애틀 워싱턴 대학교에 감사드립니다.

또한 모델 개발 초기에 반영한 지역, 광역, 연방 기관 및 학계와 산업계 이해관계자들의 전문 지식에도 감사드립니다.

## 연구비 지원
ePTM v2 개발의 과학적 기초에 대한 주요 지원은 캘리포니아 어류·야생동물국이 수여한 캘리포니아 Proposition 1 과학 연구비로 제공되었습니다.

의사결정 지원을 위한 캘리포니아 센트럴밸리 모델 통합과 이해관계자 참여에 대한 주요 지원은 캘리포니아 대학교 샌타크루즈 캠퍼스와의 협력 협약을 통해 미국 개간국이 제공했습니다.

**연구비 지원 기관**

![Image](https://github.com/cvclcm/ePTM_v2/blob/main/SupportingMaterial/CDFWLogo.png) ![Image](https://github.com/cvclcm/ePTM_v2/blob/main/SupportingMaterial/BORLogo.png)
