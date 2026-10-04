---
schema_version: 1
handoff_id: e310
parent_handoff_ids: []
lineage: none
chain: [litellm-pr44379-clinepass]
repo: litellm
workspace: clinepass-pr44379
branch: clinepass-upstream
head_sha: 0bc31c076f2faed3e8de64543b53f1393e2e649e
created_at: 2026-10-04T07:01:50-04:00
writer: copilot-cli
---

# Handoff - ClinePass credential isolation

## The Goal

Finish the user's work on BerriAI/litellm#44379, head branch
`djbclark/litellm:clinepass-upstream`: prevent unrelated provider credentials
from reaching ClinePass, address every Greptile/Veria/code-scanning finding,
prove isolation with tests, run local lint, and report CI and CLA accurately.
Companion documentation is BerriAI/litellm-docs#2049.

## Where We Are

All implementation changes are pushed through
`0bc31c076f2faed3e8de64543b53f1393e2e649e`. CLAassistant comment 5970608754
explicitly says all committers signed. Latest hosted lint and CodeQL pass;
Veria comment 5970687919 says no open security concerns remain.

The user invoked `/loose`, chose to fix the newly discovered moderation and
managed-WebSocket leaks, then chose `/handoff` at the closing prompt.
Implementation is complete; final full local lint verification is not.

Workspace: `/Users/djbclark/.cow/pastures/litellm/clinepass-pr44379`.
Branch: `clinepass-upstream`. Before this handoff, tracked files were clean,
with inherited untracked `.ignore` and `graft/` only. Remote head was verified
equal to the implementation head above. This handoff receives a local-only
documentation commit; do not accidentally push it with a subsequent code fix.
The primary LiteLLM and docs checkouts have changes not made by this session;
leave them alone. Copied primary changes are preserved in a pasture stash.

Implementation commits:

- `ace7be673df91323206dd70d3f9224527ec6ce60`: reject realtime fallback,
  move credential policy out of the shared chat dispatcher, fix duplicate
  unnecessary-lambda code-scanning findings, add credential regressions.
- `497dc8a7aa584a3864325f423583f9b123db3414`: reject sync/async moderation
  and isolate managed Responses WebSocket credentials.
- `0bc31c076f2faed3e8de64543b53f1393e2e649e`: pin authenticated WebSocket
  models and fix the moderation routing argument type.

Changed source/test surfaces: `litellm/llms/clinepass/chat/transformation.py`,
`litellm/main.py`, `litellm/utils.py`, `litellm/responses/main.py`,
`litellm/responses/streaming_iterator.py`,
`tests/unit/llms/clinepass/chat/test_clinepass_chat_transformation.py`, and
`tests/unit/llms/clinepass/test_clinepass_endpoint_guard.py`.

## What We Tried

- Ordinary `make lint` selected the fork's stale staging base and reported
  unrelated deltas. The real PR base is upstream `main`; use
  `make lint BASE_REF=upstream/main`.
- Initial pytest lacked boto3; frozen proxy/e2e development dependencies were
  restored with `uv sync --inexact --frozen --group proxy-dev --group e2e-dev`.
  Do not reinstall without a missing-dependency failure.
- `pytest --disable-socket` conflicts with the repository's allowed-host
  setup. Ordinary unit pytest already restricts networking through conftest.
- The full follow-up lint run started before the final typing correction.
  It finished with one added `reportUnknownArgumentType` and exit 2.
  That same error was fixed in the final commit; latest hosted lint passes.
  Do not describe this older local failure as a successful latest local run.
- Code-scanning alert metadata API returned 403. All public alert comments
  were read and addressed; latest CodeQL passes.

## Key Decisions

- Keep credential and unsupported-endpoint policy in the provider config.
  ClinePass chat needs a custom transformation; registering it as a generic
  OpenAI-compatible provider would also enable unsupported endpoint routes.
- Reject unsupported realtime and moderation calls before key selection or
  dispatch rather than trying to adapt them to OpenAI endpoints.
- Managed chat-bridge sockets may use explicit/provider-resolved keys, not
  global/OpenAI defaults. Frames changing provider cannot inherit the
  connection's key or base URL.
- Authenticated managed sockets cannot change models beyond their authorized
  connection model/alias; changing models requires a new authorized connection.
  Unauthenticated internal/SDK switching remains supported with key isolation.
- Retain `https://docs.litellm.ai/docs/providers/clinepass`: it matches the
  companion page. Do not merge docs or alter upstream to make CI green.
- Do not change type/test budgets or suppress errors to pass gates.

## Evidence & Data

Logs live outside Git under
`/Users/djbclark/.copilot/session-state/eae7d2c0-39a7-4e38-9a22-e54f508b45e2/files/`.

- `followup-auth-tests.log`: 281 passed, 8 existing warnings; 105 ClinePass
  cases, including actual in-memory managed WebSocket bridge/header checks.
- `followup-auth-gates.log`: latest type-discipline and test-quality gates
  pass. Both local validation processes have finished; none remain running.
- `followup-lint.log`: formatting, Ruff, import safety, e2e typing, strict
  budgets and discipline/test budgets pass; older full basedpyright result
  fails on the subsequently fixed argument-type error.
- `followup-targeted-types.json`: targeted diagnostics after the type fix.
- Mutation logs prove restored leaks break tests: chat fallback 3 failures,
  realtime rejection removal 12, moderation rejection removal 8, borrowed
  WebSocket credentials 4, authorization pin removal 12.
- Initial full local lint passed on `ace7be673d`.
- Latest hosted snapshot: 100 successes, 1 skipped, 2 failures, 3 in progress
  (`schema-migration`, `rust-test`, `rust-wheel`). Hosted lint passes.
- Latest failed code-quality run 37194420117 and documentation run 37194420067
  both name only undocumented `CLINEPASS_API_BASE` and `CLINEPASS_API_KEY`.
  All docs validators previously passed with the companion page present.
  Docs PR remains open at `b353ac7367aec1239448547e45dd485491ab3491`.

Downloaded documentation validation artifacts were removed. No source probes
or temporary mutation edits remain. PR description was updated, but no
comments, reviews, review requests or thread resolutions were posted.

## Operator Feedback

- Original request requires local lint/provider tests, normal fork-branch
  pushes only, and a report of each finding, CLA, commits and CI.
- User signed the CLA, chose "fix" for the loose-audit leaks, then "handoff".
- Do not post PR comments, request reviews, resolve threads, force-push, write
  upstream branches, or merge the companion docs PR. The user handles those.
- Prefix shell commands with `rtk`; use `rtk proxy` for unsupported commands.

## Where We're Going

1. **Run full local lint on implementation head `0bc31c076f` with
   `BASE_REF=upstream/main` and collect its final result.** The old run is
   complete, so there is no active checker to duplicate.
2. Refresh hosted checks and bot comments, especially the three pending checks.
   Reconfirm any failed docs checks are still only the companion dependency.
3. If a new, tightly coupled problem appears, investigate and fix without
   budget changes; ensure the local-only handoff does not enter the code push.
4. Report final local/hosted outcomes. Docs merging and PR communication remain
   the user's responsibility. Follow `/loose` with a closing decision if needed.

## Quick Start

```bash
cd /Users/djbclark/.cow/pastures/litellm/clinepass-pr44379
rtk git status -sb
rtk git log -4 --oneline
rtk proxy make lint BASE_REF=upstream/main
rtk proxy gh pr view 44379 --repo BerriAI/litellm --json headRefOid,statusCheckRollup
rtk proxy gh pr view 2049 --repo BerriAI/litellm-docs --json state,headRefOid
```

Tier 1 pointer:
`~/.local/state/handoffs/litellm/clinepass-pr44379/SESSION_LOG.md`.
Canonical chain log:
`~/.local/state/handoffs/chains/litellm-pr44379-clinepass/SESSION_LOG.md`.
Lineage is explicitly none: this session did not start from a parent handoff.
