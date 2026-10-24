# Contributing to EL-Rouji Documentation

This repository documents the current EL-Rouji app and the proposed future jobs subsystem. The core application code is maintained in [EL-Rouji](https://github.com/looloofy69-sketch/EL-Rouji).

## Documentation rules

1. Separate **implemented**, **tested**, **planned**, and **unknown** behavior.
2. Link factual technical claims to the relevant code path or source document in the core repository.
3. Never convert a roadmap item into a present-tense feature claim.
4. Keep route names, request fields, limits, configuration keys, and build commands aligned with source.
5. Do not include credentials, personal conversation exports, private test files, or production logs.
6. Update the relevant document and changelog when the supported behavior or acceptance criteria change.
7. Record the date and evidence for environment-dependent claims such as hosted status, provider configuration, and dependency-audit results.

## Change procedure

- Make focused changes on a branch.
- Review links and Markdown formatting.
- Check all status statements against source or actual test evidence.
- Explain any assumption directly in the document.
- Open a pull request when collaboration/review is available.

## Review checklist

- [ ] Claims are backed by source or explicitly labeled as proposed.
- [ ] API docs match request validation and response shapes.
- [ ] Security limitations are visible rather than hidden.
- [ ] Future jobs are not described as already implemented.
- [ ] No secrets or sensitive user content were added.
- [ ] Navigation links and the changelog are updated.
