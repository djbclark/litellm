---
schema_version: 1
handoff_id: ef5c
parent_handoff_ids: [e310]
lineage: deterministic
chain: [litellm-pr44379-clinepass]
repo: litellm
workspace: clinepass-pr44379
branch: clinepass-upstream
head_sha: 650604088f6dd17deefa1cb3fd159629479f31aa
created_at: 2026-10-08T18:30:00-04:00
writer: claude-code
---
# Handoff — ClinePass PR #44379: lint gates after the upstream merge

## The Goal
Get BerriAI/litellm PR #44379 (ClinePass provider, head `clinepass-upstream` on djbclark/litellm, base `main`) green and mergeable. Parent handoff e310 covered credential isolation; this one covers the lint breakage that appeared when another session merged upstream/main.

## Where We Are
Pushed `650604088f` to `origin/clinepass-upstream`. A fresh full `make lint BASE_REF=upstream/main` exits 0 and the 105 ClinePass tests pass. Hosted `lint` passes on that head (111 checks pass, 3 fail).

The 3 hosted failures:
1. `documentation` and `code-quality` fail only on `CLINEPASS_API_BASE` / `CLINEPASS_API_KEY` missing from docs. They need BerriAI/litellm-docs#2049 (still OPEN) merged.
2. `rust-test` fails on one `assertion left == right` test and nextest cancelled the rest. The PR diff touches no Rust, so this looks like an upstream failure. Cause not diagnosed.

CLA: CLAassistant on #44379 says "All committers have signed the CLA."

Git state: local branch is one commit ahead of the remote, `0c03e1fe37` (the e310 handoff doc). It is deliberately NOT pushed to the PR branch because it would add `docs/handoffs/` to the upstream PR. It is on `origin/handoff-pr44379`. This handoff (ef5c) sits on top of it locally and also goes only to that branch. Untracked `.ignore` and `graft/` belong to another session. Stash `stash@{0}` is inherited. Local branch `backup-pre-rebase-20261003` must stay until #44379 merges.

## What We Tried
1. First fresh lint on the merged head failed the new upstream LIT010/LIT011 gate (18 violations) and five basedpyright per-rule ceilings. The hosted `lint` job had failed the same way at 15:56 on `ba51099d5c`.
2. Bare `dict` annotations in the ClinePass overrides were the bulk of the basedpyright overage (about 50 diagnostics in `transformation.py`). Switching to `dict[str, object]` removed most of them; scoped `pyright: ignore[...]` with reasons covers the three `super()` calls into the bare-dict base class.
3. `# mutable-ok` and `# pyright: ignore` cannot share one comment line, so the `Final[dict[str, object]]` declarations are split from their assignments: the declaration carries `mutable-ok`, the assignment carries the pyright ignore.
4. ruff's formatter fought a long `# rebind-ok` trailing comment in `get_llm_provider_logic.py` (it wrapped the call and moved the comment off the offending line). Fixed by naming the config `cfg` so the line and a short comment fit in 120 columns.
5. A stale full lint was started before the last edits; I killed it and re-ran rather than trusting it.

## Key Decisions
1. Fix the code, not the budgets: AGENTS.md forbids editing the `*-budget.json` files on a PR branch.
2. Keep the handoff doc off the PR branch (rejected: pushing it with the fix, because it pollutes the upstream PR).
3. Keep in-place mutation of `optional_params` in `map_openai_params` (rejected: a functional rewrite), since the base class mutates and returns the caller's dict.
4. Commit message has no Claude attribution, per the repo's AGENTS.md (this overrides the harness default).

## Evidence & Data
1. Local lint log: `/tmp/lint-fresh4.log` ends `OK: every basedpyright rule is within its ceiling (137156 errors total, base d8c0e2c7153d)` and `rc=0`. May be gone; re-run to reproduce.
2. Hosted: `gh pr checks 44379 --repo BerriAI/litellm`: lint pass; code-quality, documentation, rust-test fail.
3. Files in `650604088f`: `litellm/__init__.py`, `litellm/main.py`, `litellm/litellm_core_utils/get_llm_provider_logic.py`, `litellm/llms/clinepass/chat/transformation.py`.
4. Heavy commands must go through `~/ops/site-private/bin/bg`; the machine load was about 223 during this session, so runs queue for a long time.

## Operator Feedback
None this session beyond the follow-up asking to keep `backup-pre-rebase-20261003` and report the CLA status.

## Where We're Going
1. Run `gh pr checks 44379 --repo BerriAI/litellm` and `gh pr view 2049 --repo BerriAI/litellm-docs --json state,mergedAt`. When #2049 has merged, re-run the failed `documentation` and `code-quality` jobs and confirm they pass.
2. Look at the `rust-test` failure: `gh run view --repo BerriAI/litellm --job 113563413257 --log-failed`, strip ANSI codes, find the failing test. If it also fails on upstream `main`, it is not ours; say so on the PR only if the operator wants a comment.
3. Re-sync the PR description with the new commit (AGENTS.md rule). Read `.github/pull_request_template.md` first, no em dashes, no trailing period on paragraphs.
4. After #44379 merges, the local branch `backup-pre-rebase-20261003` may be deleted (not before).
5. Do not push the local `clinepass-upstream` branch as-is. Push handoff docs only with `git push origin <sha>:refs/heads/handoff-pr44379`.

## Quick Start
```bash
cd /Users/djbclark/.cow/pastures/litellm/clinepass-pr44379
git fetch origin && git status -sb
gh pr checks 44379 --repo BerriAI/litellm
gh pr view 2049 --repo BerriAI/litellm-docs --json state,mergedAt
~/ops/site-private/bin/bg rtk proxy make lint BASE_REF=upstream/main > /tmp/lint.log 2>&1; echo rc=$?
```
