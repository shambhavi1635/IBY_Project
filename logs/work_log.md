# IBY Project : Work Log

## Day 1 Setup & Dataset A Audit

- Set up GitHub repository, Python environment, dependencies, and project structure.
- Inspected Dataset A and confirmed **63 sessions**.
- Selected the first session for detailed analysis.
- Studied chunk structure, `events.jsonl`, `gt.jsonl`, and event schema.
- Found that sessions can span multiple chunks and raw events need chronological sorting.
- **Decision:** Understand the data first before designing the segmentation approach.

## Day 2 First-Session Analysis

- Analyzed process-level ground truth and reconstructed the process timeline.
- Identified **9 process types** and multiple cases per process.
- Observed process switching, interleaving, and interruptions.
- Found that `process_switched_out` does not necessarily mean process completion.
- Compared `process_started` and `task_started`; they occur within approximately **1–3 ms** in the inspected session.
- Reconstructed **2,348 raw events** from 2 event files.
- Tested raw-event windows around process boundaries; found that timing varies and a fixed window is unreliable.
- **Decision:** Check whether these patterns hold across all 63 sessions before building the segmentation pipeline.


## Day 3 Segmentation Model Development

- Built and evaluated a temporal-gap baseline (V0), followed by supervised boundary detection using Dataset A.
- Compared multiple ML models; Random Forest achieved the best validation F1 of 0.846.
- Evaluated on 10 unseen test sessions, achieving Precision = 0.758, Recall = 0.733, and F1 = 0.745 at 3-second tolerance; generated 303 segments.
- Finalized and saved the Random Forest model, preprocessing pipeline, and metadata; verified successful artifact loading.

## Day 3.5 Segmentation Model Correction & Finalization (V2)

* Re-evaluated the Day 3 CatBoost model; the 0.95 threshold overfit a small validation split (~9 sessions).
* Selected the boundary threshold using **5-fold GroupKFold CV**, grouped by session to prevent leakage.
* Restricted boundary events to context shifts and included `process_resumed` as a valid segment start.
* Set the minimum segment duration to **15s**, based on the measured GT minimum of 16.64s.
* Compared Logistic Regression, Random Forest, XGBoost, and CatBoost; **CatBoost performed best**.
* Retrained on train+validation and saved/reloaded the final model and preprocessing artifacts.
* **Test:** Precision 0.957, Recall 0.685, F1 0.799; Segment IoU≥0.5 F1 = 0.727.
* **Decision:** CatBoost is the final boundary detection model, replacing the earlier Random Forest result.

## Day 4 Segment Labeling (Dataset A) & Dataset B Application

* Created behavioral features for each Dataset A segment and evaluated `extracted_text` coverage.
* Selected the number of KMeans clusters using a **k=5–20 silhouette sweep**; compared against HDBSCAN.
* Validated clustering with process-code purity and ARI only as post-selection sanity checks.
* Kept the **Dataset A labeler separate from Dataset B**, as their processes/apps differ and cluster correspondence is not meaningful.
* Applied the trained **boundary detector + preprocessor from A** to Dataset B's raw events.
* Added fallback handling for sessions with no predicted boundaries: one whole-session segment.
* Rebuilt B's segment features and **refit clustering independently on Dataset B**.
* Performed structural quality checks and saved the final segmentation outputs and reports.
* Reloaded `segments.jsonl` from disk and verified the submission schema.
* **Final design:** Boundary detection transfers **A → B**; segment labeling is **refit independently on B**.



## Day 5  Process Selection
Validated existing outputs and identified a minor 9/98 segment merge-floor issue, logged for the next re-run.
Built and tested an automation suitability framework; ranking remained robust across alternative scoring methods and weightings.
Selected process_cluster_0 as the top candidate: 64 occurrences, 14/15 sessions, 51.5% activity time, low behavioral variance, and limited app usage.
Saved the selection notebook and outputs. Next: identify the actual business process from extracted text/screenshots before automation design.

## Day 5.5 — Automation Justification
Built process_automation_justification.ipynb to provide evidence for why process_cluster_0 is the best automation candidate, independently validating its profile from raw segmentation outputs.
Confirmed strong evidence: 64 occurrences, 14/15 sessions, 4 machines, 51.5% activity time, low behavioral variance (CV 0.020), and 0.929 automation score vs. 0.354 for the runner-up; 100% win rate across 5,000 weightings.
Documented that human decision-making and error rates cannot yet be established from current data. Saved the profile, comparisons, methodology, and evidence plots to outputs/.
Decision: identify the actual business process using sampled screenshots/extracted_text before Step 3. The Day 5 merge-floor issue remains documented.