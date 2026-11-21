---
name: fork-project-engineering
description: Use when the user explicitly asks to inspect, modify, debug, modernize, build, or maintain a forked software repository, especially when GitHub Actions, CI failures, external dependencies, architecture-specific packaging, or release artifacts are involved. This is an optional engineering reference, not a mandatory workflow.
---

# Fork Project Engineering

## Purpose

Apply proportional engineering practices when working on a fork of an existing software project. The goal is to understand the upstream project, make bounded and reviewable changes, diagnose CI failures from evidence, and verify the actual deliverables.

This Skill is a reference, not a rigid template:
- Do not force a full process onto a trivial change.
- Do not copy another project's build system, directory layout, dependencies, or workflow.
- Do not modify core source, publish Releases, change branches, or perform unrelated cleanup unless authorized.
- Respect explicit user constraints and existing project conventions.
- Distinguish verified facts, hypotheses, and untested assumptions.

## Workflow

### 1. Establish repository context

Identify:
- upstream repository and fork;
- default branch and active development branch;
- current commit and fork/upstream divergence;
- working-tree changes and recent relevant commits;
- language, build system, toolchain, dependencies, CI workflow, packaging, and deliverables.

Do not assume the fork's default branch is its development branch. Do not assume the fork is synchronized with upstream.

### 2. Define scope before editing

Write down:
- required changes;
- permitted supporting changes;
- explicit non-goals / files or behavior that must not change;
- deferred work.

Prefer the smallest change that satisfies the goal. Avoid opportunistic refactors, unrelated dependency upgrades, and removal of existing CI steps without evidence that they are unnecessary.

Keep changes reviewable and reversible. Separate unrelated fixes into distinct commits where practical. Inspect the complete diff and changed-file list before committing.

### 3. Understand the complete build and packaging chain

Trace:
trigger → runner → checkout → toolchain → dependencies → compilation → post-processing → installer/portable packaging → validation → artifact upload.

Determine where each important file is downloaded, extracted, copied, transformed, and ultimately included. A successful download does not prove that a dependency reaches the final package.

### 4. Diagnose GitHub Actions by failure layer

Before rerunning a failed workflow, inspect:
- run ID, event, branch, commit SHA, status/conclusion;
- whether Jobs were created;
- each Job conclusion and step logs;
- annotations/check information;
- artifacts and their metadata.

Use the observed failure layer to choose the next investigation:

| Symptom | Investigate first |
|---|---|
| Run failed with zero Jobs/logs/artifacts | Workflow recognition, YAML structure, event/branch/path filters, job conditions |
| Runner did not start | Runner labels, availability, permissions, environment setup |
| Early setup step failed | checkout, permissions, toolchain, environment variables, network |
| Dependency download failed | pinned URL/version/asset, network, rate limit, checksum |
| Compilation failed | compiler/SDK, flags, source changes, missing headers/libraries |
| Packaging failed | paths, architecture mapping, installer scripts, staging files |
| Artifact upload failed | upload paths/globs, file existence, permissions |
| Artifact exists but app fails | package contents, runtime dependencies, architecture, target environment |

Zero Jobs is a symptom, not a root cause. It can point toward workflow parsing or triggering, but verify both. Do not automatically label it a YAML error.

When Jobs are zero, inspect the entire workflow file, including the end:
1. Confirm path and extension under `.github/workflows/`.
2. Validate YAML indentation, mappings, lists, expressions, and block scalars.
3. Look for duplicated or orphaned fragments, especially at EOF.
4. Check `on` event configuration and branch/path filters against the actual event.
5. Check job-level `if`, matrix expressions, and dependencies.
6. Inspect GitHub check/workflow metadata for the commit.

Use YAML parsers and actionlint where available. Validate complex PowerShell or shell scripts separately when practical. Never claim a lint check passed unless it was actually run.

### 5. Make CI changes conservatively

- Read the entire workflow and understand step dependencies before editing.
- Preserve effective build, test, and validation steps.
- Do not delete steps merely to save minutes or make the run green.
- Do not turn failures into success with `continue-on-error` or swallowed exit codes unless explicitly justified.
- Keep download, checksum, extraction, staging, packaging, and verification failures diagnosable.
- Make one logical change at a time where practical.

Do not rerun without a reason. Before every rerun, state the evidence-based hypothesis, the change made, and the expected observable difference.

### 6. Pin and verify external dependencies

When CI downloads external components, record and pin:
- source and publisher;
- release tag/version or commit;
- exact asset name and URL;
- checksum (prefer SHA-256);
- license/distribution conditions;
- expected files and architecture.

After download:
1. verify file exists and is nonempty;
2. verify checksum;
3. extract into a known directory;
4. validate expected filenames, counts, architecture, and plausible sizes;
5. copy into the actual build staging path;
6. verify inclusion in final packages.

A matching checksum proves consistency with the chosen digest; it does not independently establish publisher trust or license compliance.

Prefer CI-time dependency injection over core-source changes only when the project's actual integration mechanism supports it. Do not assume all features can be integrated without source changes.

### 7. Verify architecture and packaging

For every target architecture, map:
- build target;
- dependency binary;
- runtime loading path;
- portable package path;
- installer destination;
- verification method.

Do not infer architecture from a filename alone. Do not treat file existence in a staging directory as proof of inclusion in the final artifact.

For ZIPs, inspect archive integrity and actual contents. For installers, use an appropriate listing/extraction tool or another reliable content-level verification method. If installer internals were not inspected, explicitly say so; a successful installer build or nonempty EXE is not equivalent to internal-content verification.

### 8. Verify artifacts and runtime behavior

Inspect artifact names, count, expiration, sizes, and downloadability. Download and inspect the actual deliverables where feasible.

Record the relationship among:
- workflow run URL / run ID;
- commit SHA;
- artifact ID/name;
- each deliverable filename, size, and SHA-256.

Keep hashes associated with the exact file they describe. Do not confuse an outer artifact archive hash with hashes of files inside it.

Separate verification levels:
1. workflow recognized and Jobs created;
2. steps succeeded;
3. artifacts generated;
4. archives and package contents verified;
5. installer internals verified;
6. software launched and key functions tested;
7. user acceptance confirmed.

Do not claim a higher level than the evidence supports.

### 9. Document the outcome

For significant failures, record:
- symptom and impact;
- observed evidence;
- hypotheses considered;
- confirmed root cause;
- files and commit changed;
- before/after workflow runs;
- artifact checks;
- verification not performed;
- user acceptance, if provided.

Keep facts, inferences, and unknowns separate.

## Reusable checklist

Use applicable items only:
- [ ] Upstream, fork, branches, divergence, and current state identified.
- [ ] Required changes and explicit non-goals defined.
- [ ] Full build and packaging chain understood.
- [ ] Failed Run/Jobs/steps/logs/artifacts inspected before retry.
- [ ] Zero-Job runs investigated at workflow recognition and trigger layers.
- [ ] Entire workflow checked, including duplicate/orphaned EOF fragments.
- [ ] YAML/action/script static validation performed where available.
- [ ] CI changes are minimal and preserve valid steps.
- [ ] Every rerun has an evidence-based reason.
- [ ] External dependencies are pinned and verified where appropriate.
- [ ] Architecture-specific staging and final package contents checked.
- [ ] Artifact metadata and actual deliverables inspected.
- [ ] Installer internals are verified or explicitly marked unverified.
- [ ] Runtime and user acceptance claims match actual testing.
- [ ] Commit, run, artifact, and verification evidence documented.

## Reference document

The full Chinese guide, detailed troubleshooting decision table, checklist, and MPC-HC case study are maintained in:

`docs/Fork 项目改造与 GitHub Actions 工程实践规范.md`

Read the full document for complex CI failures, external dependency integration, packaging, or when the user asks for a full project workflow. For routine fork tasks, this Skill is a compact operational guide.

## Final decision rule

Understand the repository before changing it. Diagnose the failure layer before editing. Make the smallest authorized change. Verify the final deliverables and runtime behavior, not only the green CI badge. State only what the available evidence proves.
