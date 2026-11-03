# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

ePTM v2 is a Java agent-based model of juvenile salmon migrating through the Sacramento-San Joaquin Delta. It extends DWR's DSM2 Particle Tracking Model (PTM) with fish behavior: swimming, junction routing, and predation mortality. Hydrodynamics are not simulated here; they are read from a DSM2 hydro output (HDF5 "tide file"), and simulated fish do not interact, so runs parallelize trivially.

Calibration, data sources, and usage are covered by `README.md`, the [Wiki](https://github.com/cvclcm/ePTM_v2/wiki), and `documentation/RunningEPTMv2.docx` (the full setup tutorial).

## Platform constraints

- **Windows only, 32-bit Java.** `Globals.initialize()` calls `System.loadLibrary("PTM")`, and much of `PTMFixedData`, `PTMHydroInput`, and `Globals` is `native`. That library is not in this repo; it comes from a DSM2 install, which must be on `Path` (`%DSM2_HOME%\bin`). The model cannot run on Linux/WSL.
- **Java version follows DSM2 version:** 32-bit JDK 1.8 for DSM2 8.1.3 and earlier, 1.6 for DSM2 8.2. The committed `eptm.jar` targets DSM2 8.1.2/8.0.6 with JDK 1.8.
- Paths in run configs should use forward slashes and contain no spaces.

## Build and run

There is no build tool, test suite, linter, or CI. The official build is Eclipse: make a Java project rooted at `ptm_SWFSC/`, add all of `ptm_SWFSC/lib/*.jar`, set the main class to `DWR.DMS.PTM.MainPTM`, and export a Runnable JAR with packaged libraries. Editing source has no effect on `eptm.jar` until it is re-exported.

Compile-only check (needs a JDK; sources contain Cp1252 characters):

```
javac -encoding Cp1252 -cp "ptm_SWFSC/lib/*" -d <scratch_dir> ptm_SWFSC/DWR/DMS/PTM/*.java
```

Run (see `scripts/runEPTMBasic.py`). Arguments are positional; if the fourth is present, all thirteen are required:

```
java -Xss5M -Xms512M -Xmx1024M -jar eptm.jar <ePTM_config.inp> <behavior.h5> <output.h5> <configDir> <configDir> ChansOrientsBends812.csv Junctions812.csv latVel.csv latEps.csv latdEp.csv latd2Ep.csv VerEps.csv VerdEp.csv
```

Eclipse runs add `-Xverify:none` and use the study's `Inputs` folder as the working directory.

Scripts in `scripts/` are templates: replace their `PATH_TO_...` and other ALL_CAPS placeholders in a copy before running.

## Run pipeline

1. **DSM2 config** (`configFiles/DSM2ConfigTemplate.inp`): `ENVVAR` definitions, including `HYDROTIDEFILE`.
2. **`scripts/ePTMCreateInputs.R`** generates two files:
   - The behavior HDF5, holding `channelPars` (from `configFiles/ePTMv2CalibValidParams812.csv`), checkpoints, and mode switches.
   - The ePTM `.inp` config (same layout as `configFiles/ePTMConfigTemplate.inp`), whose `CONFIGURATION` block points at the DSM2 config. It adds one `PARTICLE_INSERTION` row per release and runs from 7 days before the first release to 10 days after the last.
3. **`eptm.jar`** runs the model and writes the output HDF5.
4. **`animateEPTM.jar`** (optional, via `scripts/runAnimEPTM.py`, source in `animateEPTM/plotPTM/`) converts the animation binary to HDF5. The binary is only written if the `.inp` `IO_FILE` block has a `ptm anim out 15min <file>.bin` line.

Output HDF5 layout (written from `BehavedParticle`): `inserted/`, `died/`, and `<checkpoint>/` groups per particle; `insertionArray`, `deathArray`, and `firstArrivalArrays/<checkpoint>` with `"NA"` for missing values; `realizedSurvProb/`.

## Code architecture

All model code is in the single package `DWR.DMS.PTM` under `ptm_SWFSC/`. It splits into inherited DSM2-PTM infrastructure (`*FixedData`, `*FixedInput`, `PTMEnv`, flux and I/O classes, much of it unused) and the ePTM additions where behavior work happens:

- **`MainPTM`**: parses arguments, wires up inputs, and runs the time loop: fetch hydro, update `SmartChannel`s on hydro steps, compute day/night, call `updatePosition` on each particle. Hardcodes `Particle.mixCase = 4` and `behavior = true`.
- **`Particle`**: passive random-walk physics in x/y/z, plus mixing cases, in-channel velocity, and node crossing. Its movement methods are mostly `final`; behavior plugs in through the overridable hooks.
- **`BehavedParticle extends Particle`**: all fish behavior. Overrides the deterministic x-velocity methods (swimming/holding), `makeNodeDecision` (junction routing), `checkHealth` (mortality from mean-free-path theory using `lambda` and `omega`), and `makeReservoirDecision`. Also writes all HDF5 output.
- **`SmartChannel extends Channel`**: per-channel state for behavior, such as flow direction filtering and bends. `PTMFixedInput` instantiates these for every channel.
- **`RiverBendsInput` / `JunctionsInput`** (with `LatData`, `VertData`, `ChannelOrientBendData`): load the `configFiles/` CSVs into static fields.

Behavior is selected by values in the behavior HDF5, not by code or the `.inp`:

- `swimCode`: `11` is the current model; `0` never moves; `1` is passive drift. Other codes are retired and throw.
- `nodeDecision`: `4` is the current routing model; `0` is outflow-based. These are the only two; they live in `BehavedParticle`, although the R script's comment points to `Particle.java`.
- `immortal`: `1` disables death, and survival accumulates in `realizedSurvProb`.
- `testOutType`: controls extra debug output.
- `filterK` and `tideCountThr` are legacy but must still be present because they are read unconditionally.

## Gotchas

- **Static initialization order matters.** `BehavedParticle`'s static initializer opens the behavior and output HDF5 files using `MainPTM`'s static filenames, so those must be set before the class is first touched. Global static state also means one run per JVM.
- **Internal vs. external DSM2 numbers.** Users specify release nodes and checkpoints as *external* node numbers. The R script converts checkpoints to *internal* numbers using `nodesIntExt_v8_1_2.csv`; `getEnvIndex()` and the calibration parameters (`chan_int_num`) use internal numbers. Release nodes stay external in `PARTICLE_INSERTION`.
- **Grid-specific constants are hardcoded.** `BehavedParticle.checkCheckpoints` hardcodes internal IDs for Chipps Island, the exit node, and CVP, and the reservoir name `clifton_court` for SWP. A new DSM2 grid requires changing these as well as the `*812.csv` and `*_v8_1_2.csv` files.
- **`channelPars` column order is positional.** The CSV columns must match the index mapping in `BehavedParticle`'s static initializer, and every channel must have a row.
- **Units are feet and seconds** (`lambda_ft`, `meanMigrationRate_ftsec`, and so on).
- **Known script defects:**
  - `scripts/ePTMBatch.py` launches `ePTMValidation.py`, which does not exist.
  - `scripts/ePTMRun.py` uses `setupPath` but defines only `calibPath`, so it raises `NameError`. Use `runEPTMBasic.py` as the reference.
  - `ePTMCreateInputs.R` reads `nodesIntExt_v8_1_2` without its `.csv` extension.
  - `ePTMCreateInputs.R` parses release times with `mdy_hm`, although the tutorial says `dd/mm/yyyy hh:mm`.

## Repository conventions

- The tutorial asks users to keep `main` only as a mirror of upstream and do local work on a branch, without merging that branch back into `main`. Contributions go through a fork and pull request.
- There is no `.gitignore`, and the repo is large (about 550 MB of videos, documents, and jars). Stage files explicitly, and keep Eclipse metadata, `.class` files, and model outputs out of commits.
