# Experiments

Only actual model calls are listed below. Each row names the run file that records it, or says that no run file exists. Mock-provider checks are documented separately in README.md.

**⚠ Status note (this update):** the three Gemini run files (`19f9373c-caa3-4a60-af38-20038dceafee`, `47ef0cc7-581f-4c32-879a-7a61d67919b1`, `db02720d-f91f-4acc-85f6-eaa0494df66c`) were located and every figure below for them was re-derived directly from the saved JSON. The four clef-flash run files referenced below (`2853ecc4`, `873bf026`, `758c7342`, `ee61c595`) were **not** found under `TRIAGE_STORAGE_DIR` and are marked ⚠ MOCK / UNVERIFIED — their rows are carried over from prior documentation, not re-confirmed against a run file. The single confirmed clef-flash data point is the injection-test call in `output/decision-verification.json`.

## Model call log

| Date       | Prompt version | Provider / model                 | Ticket / run                                                                    | Observed output                                                                                                    | Problem noticed                                                                                                                 | Change made                                                       |
| ---------- | -------------- | --------------------------------- | --------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------- |
| 2026-10-25 | triage-v2      | Google / gemini-3.5-flash-lite   | Injection probe, `output/live-verification.json` (no run file)                    | billing / low; summary concerned past invoices                                                                     | No injected instruction followed in this sample                                                                                 | Kept prompt; schema checked                                          |
| 2026-10-24 | triage-v2      | Google / gemini-3.5-flash-lite   | 19f9373c-caa3-4a60-af38-20038dceafee (5 labelled)                                 | 5/5 successful; category 80%, priority 100%                                                                        | TCK-10358 (expected account) classified technical_bug                                                                           | Recorded disagreement; no prompt change                              |
| 2026-10-25 | triage-v2      | Google / gemini-3.1-flash-lite   | db02720d-f91f-4acc-85f6-eaa0494df66c (5 labelled)                                 | 4/5 successful; category 60%, priority 60%                                                                         | TCK-10036 timed out (504 after 60.49 s); TCK-10359 (expected account) classified technical_bug; TCK-10313 (expected high) predicted medium | Kept explicit failure outcome; total cost unavailable                |
| 2026-10-24 | triage-v2      | Google / gemini-3.5-flash-lite   | 47ef0cc7-581f-4c32-879a-7a61d67919b1 (1 unlabelled ticket, 500 set: TCK-10015)    | 1/1 successful                                                                                                     | No labels available                                                                                                              | Accuracy shown as N/A                                                 |
| 2026-10-25 | n/a            | Cloudflare / @cf/cloudflare/clef-flash | Injection probe, `output/decision-verification.json` (no run file)            | billing / low; category prob. 0.9461, priority prob. 0.8069; summary and suggested_action null; 401 input tokens, 524 ms, $0.0000152 | None observed; confirms `DecisionResultSchema` shape (null summary/suggested_action, full probability breakdown) | Confirmed decision-endpoint schema against lib/schema.ts              |
| 2026-10-24 | triage-v2      | Cloudflare / @cf/cloudflare/clef-flash | ⚠ MOCK: 2853ecc4-ada9-4943-8683-a0ffd8968d38 (5 labelled)                    | 5/5 successful; category 80%, priority 100%; summary and suggested action not returned                              | TCK-10357 (expected account) reportedly classified technical_bug                                                                 | Run file not located; figures unverified                              |
| 2026-10-24 | triage-v2      | Cloudflare / @cf/cloudflare/clef-flash | ⚠ MOCK: 873bf026-abb8-407a-a269-c21794a8d6e6 (same 5 tickets, repeat run)    | Reportedly same predictions as 2853ecc4; mean latency 1,007 ms instead of 524 ms                                   | Same as above                                                                                                                     | Run file not located; figures unverified                              |
| 2026-10-24 | triage-v2      | Cloudflare / @cf/cloudflare/clef-flash | ⚠ MOCK: ee61c595-8a72-4351-aa1e-2d7a15ff6d92 (100 labelled, interrupted)     | Reportedly 98 of 100 processed, all 200; category 91%, priority 66%                                                | TCK-10162 and TCK-10256 reportedly have no row; stop reason not recorded                                                        | Run file not located; not yet investigated                            |
| 2026-10-24 | triage-v2      | Cloudflare / @cf/cloudflare/clef-flash | ⚠ MOCK: 758c7342-5b1b-42aa-97b4-b90159cb2df0 (100 labelled, complete)        | Reportedly 100 of 100 successful; category 91%, priority 67%                                                       | 9 category and 33 priority mismatches reported                                                                                   | Run file not located; figures unverified                              |

## Results

### Gemini 5-ticket runs (confirmed from run files)

The two Gemini runs did not use the same five tickets. The 3.5 sample (`19f9373c`) was TCK-10322, TCK-10358, TCK-10036, TCK-10287 and TCK-10313. The 3.1 sample (`db02720d`) was TCK-10323, TCK-10359, TCK-10036, TCK-10287 and TCK-10313. Only three tickets are shared (TCK-10036, TCK-10287, TCK-10313), so the two runs are not a controlled comparison.

| Run      | Model                  | Successful | Category accuracy | Priority accuracy | Mean latency                                              | Tokens                                           | Estimated cost                |
| -------- | ----------------------- | ---------- | ------------------- | ------------------- | ------------------------------------------------------------ | --------------------------------------------------- | -------------------------------- |
| 19f9373c | gemini-3.5-flash-lite   | 5/5        | 80%                 | 100%                | 7,348.2 ms                                                   | 1,827 (input 1,431, output 396)                     | 0 (configured free tier)         |
| db02720d | gemini-3.1-flash-lite   | 4/5        | 60%                 | 60%                 | 15,121.4 ms including the timeout; 3.78 s for the 4 successes | Unavailable at run level; the 4 successful rows sum to 1,462 | Unavailable (one failed call)    |

Accuracy uses all processed labelled tickets as the denominator, so the timed-out ticket (TCK-10036, status 504 after 60.49 s) counts as a mismatch in both category and priority in the db02720d run.

Per-ticket detail (from the run files):
- TCK-10322/10323 (billing/low): both models predicted correctly.
- TCK-10358/10359 (account/medium): both models predicted technical_bug — the shared mismatch.
- TCK-10036 (technical_bug/medium, tagged `adversarial|prompt_injection`): 3.5 predicted correctly; 3.1 timed out (504).
- TCK-10287 (technical_bug/medium): both models predicted correctly.
- TCK-10313 (technical_bug/high): 3.5 predicted correctly; 3.1 predicted medium (priority mismatch).

### Cloudflare clef-flash runs ⚠ MOCK / UNVERIFIED

**The run files below were not found in `runs/` under the currently configured `TRIAGE_STORAGE_DIR`.** The table is retained from prior documentation as a placeholder and must be replaced once the real files are located or the runs are redone.

The clef-flash runs return category and priority probabilities. Their `summary` and `suggested_action` fields are null, which is consistent with `DecisionResultSchema` in `lib/schema.ts` (confirmed). Cost is estimated from a configured rate of $0.038 per million input tokens and $0 for output — this rate itself is ⚠ unverified against `lib/decision.ts`/`lib/config.ts`. Free allocation is not deducted.

| Run      | Tickets                                    | Successful | Category accuracy | Priority accuracy | Mean latency | Input tokens | Estimated cost |
| -------- | -------------------------------------------- | ---------- | ------------------- | ------------------- | -------------- | --------------- | ----------------- |
| 2853ecc4 | 5 labelled                                   | 5/5        | 80%                 | 100%                | 524 ms         | 2,074           | $0.0000788          |
| 873bf026 | Same 5 tickets as 2853ecc4 (repeat)          | 5/5        | 80%                 | 100%                | 1,007 ms       | 2,074           | $0.0000788          |
| 758c7342 | 100 labelled, complete                       | 100/100    | 91%                 | 67%                 | 510 ms         | 48,454          | $0.00184             |
| ee61c595 | 100 requested, 98 processed (interrupted)    | 98/98      | 91%                 | 66%                 | 453 ms         | 47,586          | $0.00181             |

The two 5-ticket runs are described as using the same tickets and returning identical predictions, so they would count as one result. The 100-ticket complete run (758c7342) would be the full-sample result; the interrupted run (ee61c595) covers 98 of the same kind of tickets and would be reported separately. **None of this has been re-verified.**

One confirmed clef-flash data point exists: the single injection-test call in `output/decision-verification.json` (2026-10-25, status 200, billing/low, category probability 0.9461, priority probability 0.8069, 401 input tokens, 524 ms, $0.0000152 under the same mock $0.038/million basis). It is a single call, not a batch run, and does not by itself confirm the table above.

## Comparability

- The Gemini 3.5 and 3.1 five-ticket samples are different sets of tickets (three shared, two different). Their accuracy figures should not be compared directly as a controlled test.
- Earlier versions of this file described the two Gemini runs as using an identical sample. That was incorrect; it has been corrected above using the actual `ticket_ids` arrays in the run files.
- The clef-flash figures in this update are unverified placeholders, not confirmed results — see the Status note at the top of this file.
- The clef-flash output has no summary or suggested action by design (`DecisionResultSchema`), so it cannot be judged on those fields.
- Five tickets are insufficient to establish general accuracy. The labelled subset is a development set, and these results do not substitute for the instructor's hold-out evaluation.

## Next step

1. Locate the four clef-flash run files (`2853ecc4`, `873bf026`, `758c7342`, `ee61c595`) under the actual `TRIAGE_STORAGE_DIR`, or re-run `scripts/verify-clef.mjs` to regenerate them, and replace the mock table above with confirmed figures.
2. Confirm the $0.038/$0 clef-flash pricing against `lib/decision.ts` and the full `lib/config.ts` (the pricing read path was not visible in the excerpt reviewed).
3. Run each model on the same sample using the "Reuse this exact sample" option in /smoke, with the same dataset and count, to get a controlled Gemini-vs-clef-flash comparison. This has not been done yet.