# LTANEBULAX2026

**Train Condition Monitoring · Nebula X Hackathon 2026 · Team rookies**

Turning train sensor recordings into maintenance findings, inspection priorities
and downloadable predictions—all in one dashboard.

**[Hackathon prototype](https://nebulax-workspace-1029817906638.asia-southeast1.run.app/)** ·
**[Model details](docs/rail-model-release.md)** ·
**[Deployment details](integrated_app/CLOUD_DEPLOYMENT.md)**

The prototype was deployed during the hackathon using a temporary Google Cloud
project. Its continued availability depends on that project remaining active.

## About the hackathon

Nebula X challenged teams to turn real-world problem statements into working
prototypes. Our team, **rookies**, tackled **Problem Statement 3: Train Condition
Monitoring**: using onboard sensor data to identify faults and estimate the
condition of train subsystems.

The challenge was more than training a single classifier. Each subsystem had
different input signals, output formats and evaluation metrics. Teams needed to
develop suitable analysis methods, build an application that could process new
recordings, and submit predictions for unlabelled test inputs. The organisers
retained the reference answers for scoring.

## What we built

We brought all four subsystems into a shared maintenance workspace:

| Component | What the system does | Main input |
|---|---|---|
| Rail corrugation | Classifies a recording as Normal, Side I or Side II corrugation | Vibration and shock signals |
| Door operation | Finds opening/closing movements and identifies abnormal resistance | Motor current and door position |
| Air conditioning | Ranks train cars for inspection for a possible refrigerant leak | Temperature and operating-mode readings |
| Structural health | Estimates cumulative fatigue damage | Dynamic stress measurements |

Users can upload a recording, review the predicted result alongside measured
evidence, record an inspection note, and download the required prediction CSV.
The aim is to help technicians decide what needs attention—not replace their
inspection or judgement.

## Engineering highlights

- **One interface, four approaches.** Classification, movement segmentation,
  inspection ranking and regression share the same upload-to-review workflow.
- **A shared-side rail detector.** One Random Forest learns from both rail sides,
  then checks each side using a common fault threshold. The selected submission
  achieved approximately **0.83 macro F1**, as reported by the team from the
  competition scorer. This is not an accuracy percentage.
- **Reproducible exports.** All 86 official input files were rerun through the
  deployed service; all four output CSVs matched the selected submission exactly.
- **Cloud integration.** The public dashboard runs on Cloud Run while raw files
  and stored analysis results remain in private Cloud Storage and Firestore.

This is a team project. The repository combines the four component pipelines
and their integration into the shared application.

## Technology stack

| Layer | Technologies |
|---|---|
| Interface | HTML, CSS, JavaScript |
| Application service | Python, Flask, Gunicorn |
| Data and modelling | pandas, NumPy, SciPy, scikit-learn, rainflow |
| Deployment | Docker, Google Cloud Run |
| Storage | Google Cloud Storage, Firestore |

## Explore the prototype

1. Open **Review findings** and select a component. The page initially shows
   the complete prepared batch: 68 rail recordings, 38 detected door movements,
   one ACV workbook and 16 structural-health recordings.
2. Choose **New analysis** and upload recordings. Rail, Door and SHM use CSV;
   ACV uses XLSX. Door takes one continuous recording.
3. Click **Output predicted result**. Review the prediction, measured evidence
   and suggested technician check. New upload predictions open immediately.
   Reload the page to return to the complete official test results.
4. Choose **Download predictions → Download CSV**. The export matches the
   displayed results; a one-file upload does not export the full test set.

No login is required. Files are limited to 28 MiB each. Uploads use the saved
pipelines without retraining. Results are private to the current browser and
accessible for seven days; review notes are temporary.

## Approaches and development evidence

| Component | Selected approach | Development metric |
|---|---|---|
| Rail | Shared-side Random Forest, 20 fold-selected features, threshold 0.40 | Training macro F1 0.827808 (three split seeds); 0.818237 on two additional splits |
| Door | Gap segmentation, seven current/direction features, small Random Forest | IoU-weighted F1 1.0000 in reused chronological and robustness checks |
| ACV | Rank cars by mean positive temperature gap during eligible cooling | Linear rank-decay 0.9792 over six development cases |
| SHM | Rainflow cycle features and Ridge correction of a damage proxy | Teammate-reported grouped MAPE 2.020%; derived score 0.9798 |

These are **not held-out test scores**. Development data informed model choices.
Door's perfect development result is not a promise of perfect predictions. SHM's
reported validation has not been independently rerun during integration.
Only the organiser holds the official test answers. The rounded competition
result mentioned above is separate from these development metrics.

## Architecture

HTML/CSS/JavaScript dashboard → Flask service → saved subsystem pipeline →
on-screen evidence and competition CSV. Google Cloud Run hosts the container;
private Cloud Storage holds the 86 official test inputs; private Firestore stores
browser-scoped results. Storage, database access and credentials are not public.
The AI improvement tab describes a planned human-reviewed capability;
Gemini/Vertex AI are not running the predictions.

## Run the shared app

The submission app bundle includes the trusted trained models. The selected Rail
bundle is tracked at `rail_corrugation/artifacts/rail_model.joblib`; see the
[rail model release](docs/rail-model-release.md) for validation and reproduction.
Before building from a Git clone, supply the ignored Door bundle at
`door/artifacts/door_selected.joblib`. SHM's bundle is tracked.
Never load an untrusted uploaded pickle/joblib model.

From the repository root, with Docker installed:

```sh
docker build -t nebulax-workspace .
docker run --rm -p 8080:8080 nebulax-workspace
```

Open `http://localhost:8080`. Local results use memory unless cloud variables are
configured. See [deployment notes](integrated_app/CLOUD_DEPLOYMENT.md) for runtime
limits and permissions. No raw dataset is included in the submission bundle;
use recordings that follow the organiser's input schemas. Raw datasets and the
Door model bundle are not included in a fresh clone, so the repository is not a
one-command demo without those prerequisites.

## Repository and checks

`rail_corrugation/`, `door/`, `acv/`, `shm/`: component pipelines.
`integrated_app/`: shared UI/service. `tools/`: submission checks.

Install `integrated_app/requirements-cloud.txt` in an isolated environment, then:

```sh
python -m unittest discover -s integrated_app -p 'test_*server.py'
node integrated_app/test-result-sources.mjs
```

`tools/build_submission.py` runs the official cloud test catalogue through the
app's inference endpoint and validates all four submission CSVs.

## Limitations

This is a hackathon decision-support prototype, not a certified diagnostic system
or remaining-useful-life forecaster. Technicians must confirm findings. Results
on the supplied data do not establish performance on other trains or operating
conditions. The AI improvement tab describes a future human-reviewed workflow;
the deployed models do not automatically learn from uploaded recordings.

For anyone redeploying the project: keep credentials and raw data private, and
monitor cloud usage. Instance limits are not a hard spending cap.

## References

- [Official PS3 specifications](https://github.com/aochinwen/NebulaX-Hackathon-ProblemStatement/blob/main/PS3/01_Problem_Statement_3_Specifications.md)
- [Subsystem info kits](https://github.com/aochinwen/NebulaX-Hackathon-ProblemStatement/tree/main/PS3/03_References)
- [Required example schemas](https://github.com/aochinwen/NebulaX-Hackathon-ProblemStatement/tree/main/PS3/04_Example_Submission)

Raw datasets, credentials and environments must not be committed.
The hackathon video was handled separately and is not included in this repository.
