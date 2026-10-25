# Kestrel support ticket triage

A Next.js application that classifies support requests using Gemini through the OpenAI-compatible Chat Completions API, and a Cloudflare classifier (@cf/cloudflare/clef-flash). The implementation follows the instructor's Teams F1–F5 feature checklist; the course PDF supplies the API and validation workflow.

## Project Structure

```
├── app
│   ├── admin
│   │   └── page.tsx
│   ├── api
│   │   ├── chat
│   │   │   └── route.ts
│   │   ├── config
│   │   │   └── route.ts
│   │   ├── runs
│   │   │   └── [id]
│   │   │       └── route.ts
│   │   ├── smoke
│   │   │   └── route.ts
│   │   └── triage
│   │       ├── v1
│   │       │   └── route.ts
│   │       └── route.ts
│   ├── chat
│   │   └── page.tsx
│   ├── smoke
│   │   └── page.tsx
│   ├── triage-v1
│   │   └── page.tsx
│   ├── chat-client.tsx
│   ├── globals.css
│   ├── layout.tsx
│   ├── model-picker.tsx
│   └── page.tsx
├── data
│   ├── support_tickets_100_labelled.csv
│   ├── support_tickets_100_labelled.json
│   ├── support_tickets_500.csv
│   └── support_tickets_500.json
├── lib
│   ├── config.ts
│   ├── decision.ts
│   ├── errors.ts
│   ├── llm.ts
│   ├── metrics.ts
│   ├── prompt.ts
│   ├── runs.ts
│   ├── schema.ts
│   ├── storage.ts
│   ├── triage-v1.ts
│   └── triage.ts
├── output
│   ├── comparison.png
│   ├── decision-verification.json
│   ├── live-verification.json
│   └── result-clef-dm.png
├── runs
│   ├── 19f9373c-caa3-4a60-af38-20038dceafee.json
│   ├── 47ef0cc7-581f-4c32-879a-7a61d67919b1.json
│   └── db02720d-f91f-4acc-85f6-eaa0494df66c.json
├── scripts
│   ├── live-verify.mjs
│   ├── verify-clef.mjs
│   ├── verify-v1.mjs
│   └── verify.mjs
├── AGENTS.md
├── EXPERIMENTS.md
├── JOURNAL.md
├── next-env.d.ts
├── next.config.ts
├── package-lock.json
├── package.json
├── README.md
└── tsconfig.json
```

> **Note:** `runs/` above lists only the three run files confirmed to exist on disk at last check (two Gemini smoke runs plus the single-ticket unlabelled run). EXPERIMENTS.md and the Evaluation section below also describe four clef-flash runs (`2853ecc4`, `873bf026`, `758c7342`, `ee61c595`); those run files were not found under `TRIAGE_STORAGE_DIR` as of this update, so their figures are marked **⚠ MOCK / UNVERIFIED** below rather than presented as confirmed results. Locate or regenerate those files to replace the placeholders.

## Run locally

Requires Node.js 20.9+ and a Gemini API key with access to the configured models.

```powershell
npm ci
npm run dev
```

Do not overwrite an existing .env.local containing your key. Open http://localhost:3033.

- **/**: submit a ticket, select a model, see classification and token usage.
- **/smoke**: choose either dataset, select a random sample size, and run the evaluation.
- **/admin**: inspect every saved run, download JSON, and compare two runs side by side.
- **/chat**: original chat exercise with conversation history stored in page memory.

Start with a sample of 5. Calls are sequential, with a configurable delay. Larger samples may take many minutes and encounter free-tier quotas. Keep the smoke-test tab open.

## Feature checklist

- [x] **F1:** request and response contracts defined with Zod, JSON Schema generated from those contracts, server input checks before provider calls, response checks before returning results, visible front-end errors.
- [x] **F2:** input, output and total token usage displayed with each result. Missing usage is explicitly unavailable.
- [x] **F3:** front-end smoke runner, dataset choice (100 labelled or 500 unlabelled), random sampling without replacement, live rows, predicted/expected category and priority, mismatch counts.
- [x] **F4:** saved run accuracy, outcome counts, mean latency, estimated USD cost and model name; /admin lists runs.
- [x] **F5 optional:** server-allowlisted model selector, exact-sample replay with another model, and side-by-side /admin comparison.

## Demonstration

1. Submit an empty ticket: the page displays a clear validation error.
2. Submit a production outage: inspect category, priority, summary, suggested action and token counts.
3. Open /smoke. Choose 100 labelled tickets and sample 5.
4. Watch each result arrive; inspect predicted/expected labels and mismatch totals.
5. Select another model, check **Reuse this exact sample**, and run again without changing dataset/count.
6. Open /admin, select both runs and compare quality, latency, outcome counts and cost.
7. Run one ticket from the 500 set: predictions are shown but accuracy is N/A because labels are absent.

## Configuration

All secrets remain on the server. /api/config returns model IDs only.

| Setting                      | Purpose                                                |
| ----------------------------- | ------------------------------------------------------ |
| LLM_BASE_URL                 | Provider's OpenAI-compatible API base URL              |
| LLM_API_KEY                  | Private provider key                                   |
| LLM_MODEL                    | Default model                                          |
| LLM_MODELS                   | Comma-separated allowlist of selectable model IDs      |
| LLM_STRUCTURED_OUTPUT        | json_schema (default) or prompt                        |
| LLM_INPUT_PRICE_PER_MILLION  | Configured input price in USD                          |
| LLM_OUTPUT_PRICE_PER_MILLION | Configured output price in USD                         |
| CLOUDFLARE_ACCOUNT_ID         | Required, together with CLOUDFLARE_API_TOKEN, for the clef-flash model to appear in the allowlist (see `lib/config.ts`) |
| CLOUDFLARE_API_TOKEN          | Cloudflare Workers AI token; required alongside CLOUDFLARE_ACCOUNT_ID for clef-flash |
| SMOKE_DELAY_MS               | Delay between tickets; default 4500 ms                 |
| TRIAGE_STORAGE_DIR            | Optional writable storage root; default project folder |

Each saved run records the price it was calculated with. The clef-flash runs are configured to use **⚠ MOCK / UNVERIFIED: $0.038 per million input tokens and $0 per million output tokens** — this figure was carried over from prior documentation but has not been confirmed against where pricing is actually read in the codebase (likely `lib/decision.ts` or a part of `lib/config.ts` not yet reviewed). Confirm against source before relying on it. Compare costs only after each model's rate is set correctly for that run.

Set rates to zero only when the API project is confirmed to use free-tier billing. Unknown prices or token usage produce **unavailable**, never an invented zero. A failed request can incur unreported usage; run cost is unavailable if any row has unknown cost.

Gemini access changes over time. A listed model can still be unavailable to your project or quota. Change the allowlist and default using your own accessible model IDs. Restart after environment changes. No automatic paid-tier upgrade is performed.

## Architecture and API

Browser → route validation → lib/triage.ts → lib/llm.ts → provider → JSON parse + Zod → browser.

- lib/schema.ts is the source of truth for input and triage output. `TicketSchema` enforces a trimmed ticket body of 1–20,000 characters; `TriageSchema` enforces the four-category/three-priority output with a 300-character summary and 500-character suggested action; `DecisionResultSchema` (used for clef-flash) enforces nulled `summary`/`suggested_action` plus full category and priority probability distributions.
- lib/prompt.ts contains the business rules and PROMPT_VERSION (triage-v2).
- lib/triage.ts requests strict JSON Schema, rejects truncation, parses JSON and validates with Zod. Prompt mode strips an optional code fence but still validates.
- lib/runs.ts samples server-side using Fisher–Yates and evaluates **subject + body only**. Labels, tags and notes never enter the prompt.
- lib/storage.ts saves atomic per-run JSON checkpoints to runs/ and appends metadata to logs/llm-calls.jsonl.
- /api/triage GET publishes both JSON schemas; POST accepts { ticket, model? } and returns { result, model, usage, latency_ms, cost_usd }.
- /api/smoke POST accepts { dataset: "labelled100" | "tickets500", count, model?, replay? } and streams newline-delimited JSON progress events.
- /api/runs/:id downloads a run.

Input tickets must contain 1–20,000 trimmed characters, enforced by `TicketSchema` in `lib/schema.ts` before any provider call. Gemini responses contain category (billing/technical_bug/account/feature_request), priority (low/medium/high), summary (1–300 characters) and suggested_action (1–500). Extra keys are rejected (`z.strictObject`). The clef-flash run files record category and priority probabilities, with summary and suggested_action set to null; see the Evaluation section.

Invalid JSON/input returns 400, quota 429, invalid model output/network/provider failures 502, timeout 504, and bad provider configuration 500. No raw provider error reaches the browser.

Timeout: 30 seconds per SDK attempt; one SDK retry for transient errors. Malformed model output has one additional bounded retry. Usage totals on a successful ticket include both malformed and valid attempts. The runner stops early on 429 or configuration 500 and records an interrupted run with the actual processed count.

## Evaluation

Accuracy denominator = all **processed labelled tickets**, including failures. A failed call counts as both a category and priority mismatch. Unprocessed tickets are excluded and the processed/requested counts remain visible. The 500 set has no accuracy labels. Confusion matrices include a "(failed)" prediction column.

### Gemini comparison, 5 labelled tickets

Both runs were made on 2026-10-24 and 2026-10-25 respectively (confirmed from `created_at` in the run files). They used different samples: the 3.5 run (`19f9373c-caa3-4a60-af38-20038dceafee`) included TCK-10322 and TCK-10358, and the 3.1 run (`db02720d-f91f-4acc-85f6-eaa0494df66c`) included TCK-10323 and TCK-10359 in their places. The other three tickets (TCK-10036, TCK-10287, TCK-10313) were shared.

| Model                 | Successful | Category accuracy | Priority accuracy | Mean latency                                           | Estimated USD                 |
| --------------------- | ---------- | ------------------ | ------------------- | ------------------------------------------------------- | ------------------------------- |
| gemini-3.5-flash-lite | 5/5        | 80%                | 100%                | 7.35 s                                                  | 0 (configured free tier)        |
| gemini-3.1-flash-lite | 4/5        | 60%                | 60%                 | 15.12 s including timeout; 3.78 s for successful calls  | unavailable (one timeout)       |

TCK-10036 (tagged `adversarial|prompt_injection` in both run files) timed out with a 504 after 60.49 s in the 3.1 run; it completed normally in the 3.5 run. Both runs classified their account ticket (TCK-10358 / TCK-10359) as `technical_bug` against an expected `account` label. The 3.1 run additionally under-predicted TCK-10313's priority as `medium` against an expected `high`.

### clef-flash runs ⚠ MOCK / UNVERIFIED

The four run files below (`2853ecc4`, `873bf026`, `758c7342`, `ee61c595`) were not located under `TRIAGE_STORAGE_DIR` at the time of this update. The figures in this table are carried over from prior documentation and **have not been re-verified against saved run JSON**. Treat them as placeholders until the actual files are found or the runs are redone with `scripts/verify-clef.mjs`.

| Run      | Tickets                                   | Successful | Category accuracy | Priority accuracy | Mean latency | Estimated USD |
| -------- | ------------------------------------------ | ---------- | ------------------- | ------------------- | ------------- | --------------- |
| 2853ecc4 | 5 labelled                                 | 5/5        | 80%                 | 100%                | 524 ms        | $0.0000788       |
| 873bf026 | Same 5 tickets (repeat run)                | 5/5        | 80%                 | 100%                | 1,007 ms      | $0.0000788       |
| 758c7342 | 100 labelled, complete                     | 100/100    | 91%                 | 67%                 | 510 ms        | $0.00184         |
| ee61c595 | 100 requested, 98 processed (interrupted)  | 98/98      | 91%                 | 66%                 | 453 ms        | $0.00181         |

The two 5-ticket clef runs are described as returning identical predictions and are counted as one result. The interrupted 100-ticket run has no recorded stop reason; TCK-10162 and TCK-10256 have no row in it.

A real clef-flash data point that **is** confirmed is the single injection-test call in `output/decision-verification.json` (2026-10-25, status 200): it classified the TCK-10087-style billing injection probe as `billing`/`low` with category probability 0.9461 and priority probability 0.8069, at 524 ms latency, 401 input tokens, and $0.0000152 cost under the same $0.038/million mock pricing basis. This single call is consistent with — but does not independently confirm — the 5-ticket run figures above.

### Observations

- The Gemini 3.1 run had one timeout (TCK-10036) and was slower on the calls that succeeded. The 3.5 run completed every call.
- Both Gemini runs disagreed with the labels on one account ticket, classifying it as technical_bug.
- ⚠ The clef-flash observations below depend on the unverified run files above and should be re-checked once those files are confirmed: on the complete 100-ticket clef run, priority was reportedly the weaker field (33 of 100 priorities disagreed with labels, against 9 category disagreements), and the clef run reportedly made the same account→technical_bug mistake as the Gemini runs.
- These are small or single-run samples. They are smoke tests, not an unbiased model benchmark. Run each model on the same sample before drawing conclusions about which is better.

Full evidence is in runs/ and output/live-verification.json.

## Verification

```powershell
npm run typecheck
npm run build
npm test
npm run test:live
```

npm test starts an isolated local mock provider and Next server. It tests invalid requests, Zod output rejection, invalid-output retry and token accumulation, truncation, safe 404/429 handling, both datasets, sample uniqueness, exact replay, quota interruption, saved runs, downloads, admin comparison and log privacy. It requires permission to bind/connect to localhost. Mock results are kept in ignored tmp-tests-* folders and never presented as real model quality results.

npm run build and the integration suite passed. The browser was also checked for the empty-ticket error, a production-outage classification and visible token counts. Next.js was updated to 16.4.3 to resolve the reported dependency advisories; npm audit then reported zero vulnerabilities.

## Quality checklist and limitations

- [x] SDK access and credentials are server-side.
- [x] .env.local is Git-ignored; browser configuration exposes no key.
- [x] Input/output contracts use real runtime validation.
- [x] Prompts are versioned and ticket instructions are treated as untrusted data.
- [x] The TCK-10087-style injected instruction returned a normal billing/low result in the live check (`output/live-verification.json`).
- [x] Metadata logs exclude the key and full ticket text.
- [x] Controlled errors and explicit bounded retries/timeouts.
- [x] Fixed a real validation gap: empty conversations and client-supplied system roles are rejected in the chat exercise.
- [x] Saved evidence distinguishes actual calls from mocked tests.
- [ ] Same-sample comparison between models has not yet been run.
- [ ] The interrupted clef-flash run (ee61c595) has no recorded stop reason, and its run file (along with the other three clef runs) could not be located for this update — see the Evaluation section above.
- [ ] Clef-flash pricing ($0.038/$0 per million tokens) is unverified against `lib/decision.ts` / `lib/config.ts`.

This is a **local classroom app** with filesystem storage. It has no authentication; do not expose its admin and model-calling endpoints publicly without adding access controls and rate limits. A writable persistent disk is required; ephemeral serverless storage is unsuitable. A killed server may leave the last checkpoint marked running; rows already saved remain available. Large samples can exceed hosting time limits. Prompt-injection resistance is not guaranteed by schema validation: schemas enforce shape, not semantic correctness. Human review remains necessary.

## Git delivery

Features were developed as stacked branches so every later branch includes the preceding features:

1. f1/structured-triage-flow
2. feat/token-usage
3. feat/smoke-runner

Each feature has a dedicated commit. The verification branch includes final tests, documentation and real run evidence. The unrelated existing test.py is not part of the submission changes.

## References

- [Zod JSON Schema conversion](https://zod.dev/json-schema)
- [Gemini OpenAI compatibility](https://ai.google.dev/gemini-api/docs/openai)
- [Gemini API pricing and free-tier conditions](https://ai.google.dev/gemini-api/docs/pricing)
- Instructor's Teams F1–F5 checklist and provided Session 2 course PDF.

AI assistance was used for implementation, verification and documentation. Experiments and measured failures are recorded in EXPERIMENTS.md.