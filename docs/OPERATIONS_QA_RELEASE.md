# Operations, QA, and Release Runbook

This runbook separates source-level/CI evidence from checks that must be performed against the actual AppDeploy deployment.

## 1. Environment/source of truth

- **Hosted app:** [EL-Rouji](https://el-rouji-70120b.v2.appdeploy.ai/)
- **Core source:** [looloofy69-sketch/EL-Rouji](https://github.com/looloofy69-sketch/EL-Rouji)
- **Documentation/jobs workspace:** [looloofy69-sketch/EL-Rouji-jobs](https://github.com/looloofy69-sketch/EL-Rouji-jobs)
- **Hosted platform:** AppDeploy
- **Research secret:** `TAVILY_API_KEY` in secure server-side AppDeploy settings
- **Known-good source snapshot recorded by core docs:** `1792755107550`
- **Versioning:** the core docs report no release tag as of this documentation baseline.

Do not treat a Git commit as proof that a corresponding version is running in production. Record the Git commit and AppDeploy version separately.

## 2. Change procedure

1. Create a focused feature/fix branch in the core source repository.
2. Update source, QA cases, and docs together when behavior changes.
3. Commit package manifest and lockfile together when dependencies change.
4. Run the core CI checks: locked install, full audit summary, blocking production audit, typecheck, and CI-only Vite build.
5. Inspect CI results; explicitly note unresolved development/build-tool advisories.
6. Test relevant flows in the real hosted environment, including mobile, English/French, and Arabic RTL where affected.
7. Review changes and resolve required feedback.
8. Deploy using the AppDeploy workflow, never the CI-shim bundle.
9. Record source commit, AppDeploy version, test results, and rollback target.
10. Run a hosted smoke test. If failed, roll back using the platform version workflow and verify the runtime returns to its ready state.

## 3. Minimum QA matrix

Use synthetic data and dedicated test accounts only.

| Area | Required test | Expected evidence |
|---|---|---|
| Core chat | Send user message; render text, Markdown, links, code and table | Reply displays correctly; errors are actionable |
| Modes | Assistant, Research, Writing, Study, Coding, Planning | Correct mode is selected and request succeeds/fails clearly |
| Language | English, French, Arabic; mixed Darija/Latin text | Correct labels and readable RTL/mixed-script layout |
| Mobile | Small viewport, composer, sidebar, scrolling | Controls remain reachable; no critical overlap |
| Built-in tools | Prompts populate without auto-sending; calculator success and invalid input | Correct local result; invalid input does not crash or execute arbitrary code |
| Research | Valid key, missing/invalid key in test settings, source cards | Working synthesis and valid links or clear failure; no fabricated citations |
| Attachments | PNG/JPEG/WebP, TXT/MD/CSV/JSON, unsupported/oversized/malformed inputs | Valid input analyzed; invalid input rejected clearly |
| Local history | Copy/export/delete/clear and reload | UI behavior matches expectations |
| Cloud sync | Sign-in, save/load, sign-out, separate accounts, deletion | User's data is isolated and deletion behavior is as documented |
| API validation | Empty, wrong-type, malformed, extreme-size fields | Bounded request handling and safe errors |
| Operations | Health endpoint, research configuration, rollback | Version/state recorded; failure and rollback reproducible |

The core repo's `tests/tests.json` contains example AppDeploy QA scenarios, not an executable npm test suite.

## 4. Dependency audit interpretation

The core CI includes:
- Full-tree npm audit summary for triage (currently informational).
- Blocking production-only audit: `npm audit --omit=dev --audit-level=low`.
- TypeScript check.
- Vite build with the CI-only AppDeploy client shim.

The documented result in run #67 was zero findings in the production-only audit and three moderate findings in the full development/build tree. Re-run on the exact release commit; advisory metadata changes over time. A clean production-only audit is not a clean full-tree or application security audit.

## 5. Research smoke test

1. Confirm `TAVILY_API_KEY` is stored in secure server-side settings.
2. Check `GET /api/health`; interpret `capabilities.webResearch` only as secret-name presence.
3. Run a non-sensitive, specific research question through the hosted UI.
4. Verify response content, source count, URL reachability, and source relevance.
5. Try an invalid key only in an isolated test setting; ensure the error does not disclose the key.
6. Record date, AppDeploy version, source commit, and pass/fail without including secrets or sensitive query content.

## 6. Rollback

1. Capture the failing version identifier and a minimal error summary.
2. Identify the last-known-good AppDeploy version in platform history.
3. Restore it using the platform's supported version workflow.
4. Wait for the platform to report ready, then run core chat plus the affected workflow.
5. Record failed/restored version IDs, symptoms, test result, and next action.
6. Verify repository state and hosted runtime state separately.

A Git revert does not automatically prove that the hosted deployment changed.

## 7. Release gate

Do not create a release tag until the exact commit has:
- [ ] Passed CI and a fresh dependency audit.
- [ ] Passed hosted smoke tests for the affected and core flows.
- [ ] Passed relevant language/mobile/RTL tests.
- [ ] Passed account isolation, sync and deletion tests where applicable.
- [ ] Passed live research and attachment tests where applicable.
- [ ] Had security/privacy findings reviewed and assigned.
- [ ] Had full-tree audit findings remediated or explicitly risk-accepted.
- [ ] Had a tested rollback target and accurate deployment record.
- [ ] Had docs and changelog updated.

The current core release checklist lists several of these as outstanding. The status must be updated only after the corresponding evidence exists.

## 8. Operational maturity limitations

Do not infer that a formal SLA, on-call rotation, automated backups, uptime monitoring, alerting, or a tested disaster-recovery process exists. Establish and test these capabilities before they are promised.
