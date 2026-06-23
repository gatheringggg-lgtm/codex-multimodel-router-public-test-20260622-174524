# Install protocol reference

This reference defines the interruption-safe customer install flow for the Codex multi-model router skill.

## Principle

The Codex skill only guides. The package scripts perform changes and persist enough state to recover if the current Codex conversation is interrupted.

## Phase table

| Step | Customer script | Durable output | Safe to proceed when |
| --- | --- | --- | --- |
| Detect | `01-detect` | none | detected mode is official-login or supported custom provider. |
| Dry run | `02-dry-run` | `outputs/dry-run.json`, `outputs/candidate.config.toml` | candidate config matches expected mode and has no `[model_providers.openai]`. |
| Configure | `00-configure` | `config/router.env` | customer saved Base URL and API Key fields locally; secrets were not pasted into chat. |
| Doctor | `00-doctor` | none | router contract and env placeholders are understandable; no secrets printed. |
| Emit | `03-emit-apply-rollback` | `outputs/apply.*`, `outputs/rollback.*`, `ROLLBACK-FIRST.*`, Desktop `一键回滚`, `outputs/install-state.json` | state phase is `ready_for_final_apply`. |
| Apply | `04-apply-reviewed` | modified Codex config plus state phase `applied_restart_required` | customer accepted interruption risk and can restart Codex. |
| Startup | `05-install-startup` | user-level LaunchAgent or scheduled task | router status is healthy after restart. |
| Smoke | manual UI test | customer confirmation | history sidebar, picker, GPT route, and third-party tool tasks pass. |
| Rollback | Desktop `一键回滚`, `ROLLBACK-FIRST.*`, or `99-rollback-all` | restored config; state phase `rolled_back` | Codex returns to known-good behavior after full quit/reopen. |

## Final apply wording

Before `04-apply-reviewed`, the skill must tell the customer:

```text
Final apply may interrupt this Codex conversation. Keep the terminal open. I have checked the rollback safety files and the Desktop entry named 一键回滚. If Codex looks abnormal, double-click 一键回滚 on the Desktop, then fully quit/reopen Codex Desktop and contact the administrator/support person who provided this package. Validate only after restart.
```

## Failure defaults

- Missing history sidebar: use Desktop `一键回滚` first; do not touch session/history storage.
- Missing picker models: rollback or inspect candidate config; do not redefine `[model_providers.openai]`.
- Raw tool-call text: stop testing and rollback; do not ask the model to continue.
- Missing key configuration: do not say only "edit router.env". Prefer `00-configure`, which opens the local Chinese configuration page for Base URL and API Key entry. If the browser page cannot open, then use the manual fallback: show the exact local file path, show a Chinese placeholder-only template, explain that Base URL fields are normally auto-filled from current Codex config in CC Switch / Packy mode, and tell the customer to save locally and reply only `已填好`. Do not paste keys into chat.
