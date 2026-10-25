

### **New Interaction**
- **Agent Version**: unknown
- **Date**: 08-10-2026 17:26
- **User**: denis.amselem@gmail.com
- **Prompt**: commit and push
- **Claude Model**: unknown
- **Permission Mode**: unknown
- **Changes Made**: Committed and pushed the README sample-data documentation update.
- **Context and Reasons for Changes**: The requested documentation edit was pending in the working tree, and the current branch tracks origin/main.

### **New Interaction**
- **Agent Version**: unknown
- **Date**: 08-10-2026 18:46
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