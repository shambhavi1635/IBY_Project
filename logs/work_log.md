# IBY Project — Work Log

## Day 1 — Setup & Dataset A Audit

- Set up GitHub repository, Python environment, dependencies, and project structure.
- Inspected Dataset A and confirmed **63 sessions**.
- Selected the first session for detailed analysis.
- Studied chunk structure, `events.jsonl`, `gt.jsonl`, and event schema.
- Found that sessions can span multiple chunks and raw events need chronological sorting.
- **Decision:** Understand the data first before designing the segmentation approach.

## Day 2 — First-Session Analysis

- Analyzed process-level ground truth and reconstructed the process timeline.
- Identified **9 process types** and multiple cases per process.
- Observed process switching, interleaving, and interruptions.
- Found that `process_switched_out` does not necessarily mean process completion.
- Compared `process_started` and `task_started`; they occur within approximately **1–3 ms** in the inspected session.
- Reconstructed **2,348 raw events** from 2 event files.
- Tested raw-event windows around process boundaries; found that timing varies and a fixed window is unreliable.
- **Decision:** Check whether these patterns hold across all 63 sessions before building the segmentation pipeline.
