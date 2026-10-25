### **New Interaction**
- **Agent Version**: unknown
- **Date**: 2026-10-08 17:26
- **User**: denis.amselem@gmail.com
- **Prompt**: commit and push
- **Claude Model**: unknown
- **Permission Mode**: unknown
- **Changes Made**: Committed and pushed the README sample-data documentation update.
- **Context and Reasons for Changes**: The requested documentation edit was pending in the working tree, and the current branch tracks origin/main.

### **New Interaction**
- **Agent Version**: unknown
- **Date**: 2026-10-08 18:46
- **User**: denis.amselem@gmail.com
- **Prompt**: update the readme to mention the support_tickets_100_labelled.json/csv files then add them to the repo and commit and push
- **Claude Model**: unknown
- **Permission Mode**: unknown
- **Changes Made**: Updated README.md to document the labelled 100-ticket CSV/JSON dataset and committed and pushed both files with the documentation.
- **Context and Reasons for Changes**: The labelled datasets provide category, priority, and human-review annotations for students to test and evaluate triage approaches.

### Project completion — 2026-10-24
- Implemented the instructor's Teams F1–F5 checklist in separate feature branches and commits.
- Added strict request/output validation, usage display, random smoke sampling, durable run history, model selection and exact-sample comparison.
- Verified with isolated integration tests, production build, browser checks and real Gemini calls. Saved real runs and documented the timeout and label disagreements in EXPERIMENTS.md.
- Updated vulnerable Next.js dependencies; documented local hosting and cost-estimation limits.
- API keys remain in ignored .env.local. The unrelated pre-existing test.py was left untouched.

### Experiment records reconciled with run files — 2026-10-25
- Rewrote EXPERIMENTS.md from the saved run files in runs/, adding the Cloudflare clef-flash runs (2853ecc4, 873bf026, 758c7342, ee61c595) and correcting the Gemini run dates. The 3.1 run (db02720d) is dated 2026-10-25.
- Recorded that the two Gemini 5-ticket runs did not use the same sample: three tickets were shared, and the earlier "identical sample" wording was wrong.
- Treated the repeated clef 5-ticket run (873bf026) as the same result as 2853ecc4, since both returned identical predictions.
- Updated the README evaluation and configuration sections to cover the clef-flash runs and their pricing. Left the same-sample model comparison and the README's schema claims unverified.
- Open items: the same-sample comparison between models has not been run; the interrupted run ee61c595 has no recorded stop reason; the clef-flash output schema and pricing setup need checking against lib/schema.ts and lib/config.ts.

### Documentation reconciliation pass — 2026-10-25
- Re-verified the Gemini run figures in EXPERIMENTS.md and README.md directly against the saved JSON for `19f9373c-caa3-4a60-af38-20038dceafee`, `47ef0cc7-581f-4c32-879a-7a61d67919b1` and `db02720d-f91f-4acc-85f6-eaa0494df66c`: all accuracy, latency, token, and per-ticket mismatch figures matched, including TCK-10036's 504 timeout at 60.49 s and TCK-10313's priority miss in the 3.1 run.
- Found that the four clef-flash run files cited in EXPERIMENTS.md and README.md (`2853ecc4`, `873bf026`, `758c7342`, `ee61c595`) do not exist in the current `runs/` listing. Their figures were kept in both files but explicitly marked **⚠ MOCK / UNVERIFIED** rather than silently carried forward as confirmed results, per the project's own stated policy that mock results are never presented as real model quality data.
- Confirmed via `lib/schema.ts` that the clef-flash (`DecisionResultSchema`) output shape — nulled `summary`/`suggested_action`, full category and priority probability breakdown — matches what both docs describe, and that `TicketSchema` enforces a single 1–20,000 character bound (not the previously garbled "larger bound than 5,000" wording in README, which was rewritten).
- Confirmed via `lib/config.ts` that `CLOUDFLARE_ACCOUNT_ID` and `CLOUDFLARE_API_TOKEN` must both be set for the clef-flash model to appear in `availableModels()`; added both to the README Configuration table, where they had been missing.
- Could not confirm the clef-flash $0.038/$0 per-million-token pricing — the price-lookup logic was not visible in the `lib/config.ts` excerpt reviewed (likely in `lib/decision.ts` or elsewhere in `lib/config.ts`) — so this remains flagged unverified in both README.md and EXPERIMENTS.md.
- Normalized all JOURNAL.md dates to YYYY-MM-DD (the two earliest entries were previously DD-MM-YYYY).
- Open items: locate or regenerate the four clef-flash run files; confirm clef-flash pricing against source; run the same-sample Gemini-vs-clef-flash comparison via /smoke's "Reuse this exact sample" option.