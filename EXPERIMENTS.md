# Experiments

Only actual model calls are listed below. Each row names the run file that records it, or says that no run file exists. Mock-provider checks are documented separately in README.md.

## Model call log

| Date       | Prompt version | Provider / model                 | Ticket / run                                                                    | Observed output                                                                                                    | Problem noticed                                                                                                                 | Change made                                                       |
| ---------- | -------------- | -------------------------------- | ------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------- |
| 2026-10-24 | triage-v2      | Google / gemini-3.5-flash-lite   | TCK-10087 (single check; no run file)                                           | Valid billing / low; summary concerned past invoices                                                               | No injected instruction followed in this sample                                                                                 | Kept prompt; schema checked                                       |
| 2026-10-24 | triage-v2      | Google / gemini-3.5-flash-lite   | 19f9373c-caa3-4a60-af38-20038dceafee (5 labelled)                               | 5/5 successful; category 80%, priority 100%                                                                        | TCK-10358 (expected account) classified technical_bug                                                                           | Recorded disagreement; no prompt change                           |
| 2026-10-25 | triage-v2      | Google / gemini-3.1-flash-lite   | db02720d-f91f-4acc-85f6-eaa0494df66c (5 labelled)                               | 4/5 successful; category 60%, priority 60%                                                                         | TCK-10036 timed out (504 after about 60 s); TCK-10359 (expected account) classified technical_bug; TCK-10313 (high) predicted medium | Kept explicit failure outcome; total cost unavailable             |
| 2026-10-24 | triage-v2      | Google / gemini-3.5-flash-lite   | 47ef0cc7-581f-4c32-879a-7a61d67919b1 (1 unlabelled ticket, 500 set)             | 1/1 successful                                                                                                     | No labels available                                                                                                             | Accuracy shown as N/A                                             |
| 2026-10-24 | triage-v2      | Google / gemini-3.5-flash-lite   | Browser outage example (no run file)                                            | technical_bug / high; input 260, output 80, total 340                                                              | None observed                                                                                                                   | Confirmed UI token display                                        |
| 2026-10-24 | clef-v1        | Cloudflare / @cf/cloudflare/clef-flash | 2853ecc4-ada9-4943-8683-a0ffd8968d38 (5 labelled)                       | 5/5 successful; category 80%, priority 100%; summary and suggested action not returned                              | TCK-10357 (expected account) classified technical_bug                                                                           | None; recorded for comparison                                     |
| 2026-10-24 | clef-v1        | Cloudflare / @cf/cloudflare/clef-flash | 873bf026-abb8-407a-a269-c21794a8d6e6 (same 5 tickets, repeat run)        | Same predictions as 2853ecc4; mean latency 1,007 ms instead of 524 ms                                             | Same as above                                                                                                                   | Counted once in the results below                                 |
| 2026-10-24 | clef-v1        | Cloudflare / @cf/cloudflare/clef-flash | ee61c595-8a72-4351-aa1e-2d7a15ff6d92 (100 labelled, interrupted)        | 98 of 100 processed, all 200; category 91%, priority 66%                                                           | TCK-10162 and TCK-10256 have no row; the file does not record why the run stopped                                              | Not yet investigated                                              |
| 2026-10-24 | clef-v1        | Cloudflare / @cf/cloudflare/clef-flash | 758c7342-5b1b-42aa-97b4-b90159cb2df0 (100 labelled, complete)           | 100 of 100 successful; category 91%, priority 67%                                                                  | 9 category and 33 priority mismatches                                                                                           | Recorded as the full 100-ticket result                            |

## Results

### Gemini 5-ticket runs

The two Gemini runs did not use the same five tickets. The 3.5 sample was TCK-10322, TCK-10358, TCK-10036, TCK-10287 and TCK-10313. The 3.1 sample was TCK-10323, TCK-10359, TCK-10036, TCK-10287 and TCK-10313. Only three tickets are shared, so the two runs are not a controlled comparison.

| Run      | Model                  | Successful | Category accuracy | Priority accuracy | Mean latency                                           | Tokens                                         | Estimated cost                |
| -------- | ---------------------- | ---------- | ----------------- | ----------------- | ------------------------------------------------------ | ---------------------------------------------- | ----------------------------- |
| 19f9373c | gemini-3.5-flash-lite  | 5/5        | 80%               | 100%              | 7.35 s                                                 | 1,827 (input 1,431, output 396)                | 0 (configured free tier)      |
| db02720d | gemini-3.1-flash-lite  | 4/5        | 60%               | 60%               | 15.12 s including the timeout; 3.78 s for the 4 successes | Unavailable at run level; the 4 successful rows sum to 1,462 | Unavailable (one failed call) |

Accuracy uses all processed labelled tickets as the denominator, so the timed-out ticket counts as a mismatch in both category and priority.

### Cloudflare clef-flash runs

The clef-flash runs return category and priority probabilities. Their `summary` and `suggested_action` fields are null. Cost is estimated from the configured rate of $0.038 per million input tokens and $0 for output. Free allocation is not deducted.

| Run      | Tickets                                    | Successful | Category accuracy | Priority accuracy | Mean latency | Input tokens | Estimated cost |
| -------- | ------------------------------------------ | ---------- | ----------------- | ----------------- | ------------ | ------------ | -------------- |
| 2853ecc4 | 5 labelled                                 | 5/5        | 80%               | 100%              | 524 ms       | 2,074        | $0.0000788     |
| 873bf026 | Same 5 tickets as 2853ecc4 (repeat)       | 5/5        | 80%               | 100%              | 1,007 ms     | 2,074        | $0.0000788     |
| 758c7342 | 100 labelled, complete                     | 100/100    | 91%               | 67%               | 510 ms       | 48,454       | $0.00184       |
| ee61c595 | 100 requested, 98 processed (interrupted)  | 98/98      | 91%               | 66%               | 453 ms       | 47,586       | $0.00181       |

The two 5-ticket runs used the same tickets and returned identical predictions, so they count as one result. The 100-ticket complete run (758c7342) is the full-sample result. The interrupted run (ee61c595) covers 98 of the same kind of tickets and is reported separately.

## Comparability

- The three 5-ticket samples (Gemini 3.5, Gemini 3.1 and clef-flash) are different sets of tickets. Their accuracy figures should not be compared directly.
- Earlier versions of this file described the two Gemini runs as using an identical sample. That was incorrect.
- The clef-flash output has no summary or suggested action, so it cannot be judged on those fields.
- Five tickets are insufficient to establish general accuracy. The labelled subset is a development set, and these results do not substitute for the instructor's hold-out evaluation.

## Next step

To compare models fairly, run each model on the same sample using the "Reuse this exact sample" option in /smoke, with the same dataset and count. This has not been done yet.