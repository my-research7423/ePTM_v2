# CLAUDE.md

이 파일은 Claude Code(claude.ai/code)가 이 저장소의 코드로 작업할 때 참고할 안내를 제공합니다.

## 프로젝트 개요

ePTM v2는 새크라멘토-샌와킨 델타(Sacramento-San Joaquin Delta)를 통과해 이동하는 어린 연어를 시뮬레이션하는 Java 기반 에이전트 모델입니다. 캘리포니아 수자원부(DWR)의 DSM2 입자 추적 모델(PTM)에 유영, 분기점 경로 선택, 포식에 의한 폐사 같은 어류 행동을 추가했습니다. 수리 현상은 여기서 시뮬레이션하지 않고 DSM2 수리 모델 출력(HDF5 "tide file")에서 읽어옵니다. 시뮬레이션 어류끼리는 상호작용하지 않으므로 실행을 손쉽게 병렬화할 수 있습니다.

보정, 데이터 출처, 사용법은 `README.md`, [Wiki](https://github.com/cvclcm/ePTM_v2/wiki), `documentation/RunningEPTMv2.docx`(전체 설치 튜토리얼)에서 다룹니다.

## 플랫폼 제약

- **Windows와 32비트 Java 전용입니다.** `Globals.initialize()`가 `System.loadLibrary("PTM")`을 호출하며, `PTMFixedData`, `PTMHydroInput`, `Globals`의 많은 메서드가 `native`입니다. 이 라이브러리는 이 저장소에 없고 DSM2 설치본에서 제공되므로, DSM2가 `Path`(`%DSM2_HOME%\bin`)에 등록되어 있어야 합니다. Linux/WSL에서는 모델을 실행할 수 없습니다.
- **Java 버전은 DSM2 버전에 따라 결정됩니다.** DSM2 8.1.3 이하는 32비트 JDK 1.8, DSM2 8.2는 1.6을 사용합니다. 저장소에 커밋된 `eptm.jar`는 JDK 1.8과 DSM2 8.1.2/8.0.6을 대상으로 합니다.
- 실행 설정의 경로는 슬래시(`/`)를 사용하고 공백을 포함하지 않아야 합니다.

## 빌드와 실행

빌드 도구, 테스트, 린터, CI는 없습니다. 공식 빌드 방법은 Eclipse입니다. `ptm_SWFSC/`를 루트로 Java 프로젝트를 만들고, `ptm_SWFSC/lib/*.jar`를 모두 추가한 뒤, 메인 클래스를 `DWR.DMS.PTM.MainPTM`으로 지정하고, 라이브러리를 포함한 Runnable JAR로 내보냅니다. 소스를 수정해도 다시 내보내기 전까지는 `eptm.jar`에 반영되지 않습니다.

컴파일만 확인하는 명령(JDK 필요, 소스에 Cp1252 문자가 포함되어 있음):

```
javac -encoding Cp1252 -cp "ptm_SWFSC/lib/*" -d <scratch_dir> ptm_SWFSC/DWR/DMS/PTM/*.java
```

실행 방법(`scripts/runEPTMBasic.py` 참고). 인수는 위치 기반이며, 네 번째 인수가 있으면 13개 모두 필요합니다.

```
java -Xss5M -Xms512M -Xmx1024M -jar eptm.jar <ePTM_config.inp> <behavior.h5> <output.h5> <configDir> <configDir> ChansOrientsBends812.csv Junctions812.csv latVel.csv latEps.csv latdEp.csv latd2Ep.csv VerEps.csv VerdEp.csv
```

Eclipse에서 실행할 때는 `-Xverify:none`을 추가하고, 스터디의 `Inputs` 폴더를 작업 디렉터리로 사용합니다.

`scripts/`의 스크립트는 템플릿입니다. 실행 전에 복사본에서 `PATH_TO_...` 등 대문자로 된 자리표시자를 바꾸세요.

## 실행 파이프라인

1. **DSM2 설정**(`configFiles/DSM2ConfigTemplate.inp`): `HYDROTIDEFILE`을 포함한 `ENVVAR` 정의입니다.
2. **`scripts/ePTMCreateInputs.R`**가 파일 두 개를 생성합니다.
   - 행동 HDF5: `channelPars`(`configFiles/ePTMv2CalibValidParams812.csv`에서 읽음), 체크포인트, 모드 스위치를 담습니다.
   - ePTM `.inp` 설정(`configFiles/ePTMConfigTemplate.inp`와 같은 구조): `CONFIGURATION` 블록이 DSM2 설정을 가리킵니다. 방류마다 `PARTICLE_INSERTION` 행을 추가하며, 첫 방류 7일 전부터 마지막 방류 10일 후까지 실행합니다.
3. **`eptm.jar`**가 모델을 실행하고 출력 HDF5를 기록합니다.
4. **`animateEPTM.jar`**(선택 사항, `scripts/runAnimEPTM.py`로 실행, 소스는 `animateEPTM/plotPTM/`)가 애니메이션 바이너리를 HDF5로 변환합니다. 바이너리는 `.inp`의 `IO_FILE` 블록에 `ptm anim out 15min <file>.bin` 행이 있을 때만 기록됩니다.

출력 HDF5 구조(`BehavedParticle`에서 기록): 입자별 `inserted/`, `died/`, `<checkpoint>/` 그룹, 값이 없으면 `"NA"`로 채우는 `insertionArray`, `deathArray`, `firstArrivalArrays/<checkpoint>`, 그리고 `realizedSurvProb/`로 구성됩니다.

## 코드 구조

모든 모델 코드는 `ptm_SWFSC/` 아래의 단일 패키지 `DWR.DMS.PTM`에 있습니다. DSM2-PTM에서 물려받은 기반 코드(`*FixedData`, `*FixedInput`, `PTMEnv`, 플럭스 및 입출력 클래스, 상당수는 사용되지 않음)와 행동 관련 작업이 이루어지는 ePTM 추가 코드로 나뉩니다.

- **`MainPTM`**: 인수를 해석하고 입력을 연결한 뒤 시간 루프를 실행합니다. 루프에서는 수리 데이터를 가져오고, 수리 시간 단계마다 `SmartChannel`을 갱신하고, 낮/밤을 계산하고, 각 입자의 `updatePosition`을 호출합니다. `Particle.mixCase = 4`와 `behavior = true`가 하드코딩되어 있습니다.
- **`Particle`**: x/y/z 방향의 수동적 랜덤 워크 물리, 혼합 방식, 채널 내 유속, 노드 통과를 처리합니다. 이동 메서드는 대부분 `final`이며, 행동은 재정의 가능한 훅을 통해 연결됩니다.
- **`BehavedParticle extends Particle`**: 모든 어류 행동을 담당합니다. 결정론적 x 방향 속도 메서드(유영/정지), `makeNodeDecision`(분기점 경로 선택), `checkHealth`(`lambda`와 `omega`를 사용한 평균 자유 경로 이론 기반 폐사), `makeReservoirDecision`을 재정의합니다. 모든 HDF5 출력도 여기서 기록합니다.
- **`SmartChannel extends Channel`**: 유향 필터링, 만곡부 등 행동에 필요한 채널별 상태를 관리합니다. `PTMFixedInput`이 모든 채널을 이 클래스로 생성합니다.
- **`RiverBendsInput` / `JunctionsInput`**(`LatData`, `VertData`, `ChannelOrientBendData` 포함): `configFiles/`의 CSV를 정적 필드로 읽어들입니다.

행동은 코드나 `.inp`가 아니라 행동 HDF5의 값으로 선택합니다.

- `swimCode`: `11`이 현재 모델이고, `0`은 움직이지 않으며, `1`은 수동적 표류입니다. 그 밖의 코드는 폐기되었고 예외를 발생시킵니다.
- `nodeDecision`: `4`가 현재 경로 선택 모델이고, `0`은 유출량 기반입니다. 이 두 가지뿐이며 `BehavedParticle`에 구현되어 있습니다. R 스크립트의 주석은 `Particle.java`를 가리키지만 잘못된 안내입니다.
- `immortal`: `1`이면 폐사가 비활성화되고 생존율은 `realizedSurvProb`에 누적됩니다.
- `testOutType`: 추가 디버그 출력을 제어합니다.
- `filterK`와 `tideCountThr`는 레거시이지만 무조건 읽어들이므로 반드시 있어야 합니다.

## 주의 사항

- **정적 초기화 순서가 중요합니다.** `BehavedParticle`의 정적 초기화 블록은 `MainPTM`의 정적 파일명을 사용해 행동 및 출력 HDF5 파일을 엽니다. 따라서 이 클래스를 처음 사용하기 전에 파일명이 설정되어 있어야 합니다. 전역 정적 상태 때문에 JVM 하나당 한 번만 실행할 수 있습니다.
- **DSM2의 내부 번호와 외부 번호를 구분해야 합니다.** 사용자는 방류 노드와 체크포인트를 *외부* 노드 번호로 지정합니다. R 스크립트가 `nodesIntExt_v8_1_2.csv`로 체크포인트를 *내부* 번호로 변환하며, `getEnvIndex()`와 보정 파라미터(`chan_int_num`)는 내부 번호를 사용합니다. `PARTICLE_INSERTION`의 방류 노드는 외부 번호 그대로입니다.
- **격자별 상수가 하드코딩되어 있습니다.** `BehavedParticle.checkCheckpoints`에는 Chipps Island, 출구 노드, CVP의 내부 ID와 SWP용 저수지 이름 `clifton_court`가 하드코딩되어 있습니다. 새 DSM2 격자를 쓰려면 이 값들과 `*812.csv`, `*_v8_1_2.csv` 파일을 함께 바꿔야 합니다.
- **`channelPars`의 열 순서는 위치 기반입니다.** CSV 열은 `BehavedParticle` 정적 초기화 블록의 인덱스 매핑과 일치해야 하며, 모든 채널에 행이 있어야 합니다.
- **단위는 피트와 초입니다**(`lambda_ft`, `meanMigrationRate_ftsec` 등).
- **알려진 스크립트 결함:**
  - `scripts/ePTMBatch.py`는 존재하지 않는 `ePTMValidation.py`를 실행합니다.
  - `scripts/ePTMRun.py`는 `setupPath`를 사용하지만 `calibPath`만 정의하므로 `NameError`가 발생합니다. `runEPTMBasic.py`를 기준으로 삼으세요.
  - `ePTMCreateInputs.R`은 `nodesIntExt_v8_1_2`를 `.csv` 확장자 없이 읽습니다.
  - `ePTMCreateInputs.R`은 방류 시각을 `mdy_hm`으로 해석하지만, 튜토리얼은 `dd/mm/yyyy hh:mm` 형식을 안내합니다.

## 저장소 규칙

- 튜토리얼은 `main`을 업스트림의 미러로만 유지하고, 로컬 작업은 브랜치에서 하되 그 브랜치를 `main`에 병합하지 말라고 안내합니다. 기여는 포크와 풀 리퀘스트로 진행합니다.
- `.gitignore`가 없고 저장소 크기가 큽니다(동영상, 문서, jar 포함 약 550 MB). 파일을 명시적으로 스테이징하고, Eclipse 메타데이터, `.class` 파일, 모델 출력은 커밋하지 마세요.
