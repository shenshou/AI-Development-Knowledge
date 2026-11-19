---
name: cross-site-userscript-engineering
description: Use when the user asks to create, modify, debug, review, or test a browser userscript (such as Tampermonkey/Greasemonkey) for a website, especially when the task involves website data, APIs, authentication, pagination, batch requests, rate limits, or real-browser validation. This skill is an optional engineering reference, not a mandatory architecture.
---

# Cross-site Userscript Engineering

## Purpose and scope

Apply this skill when the task is about a website userscript and the guidance is relevant. It distills engineering lessons from two verified case studies (JavDB review batch search and LTX batch checking).

**This skill must improve engineering decisions, not constrain implementation choices.**

- Do not force every userscript to use APIs, queues, a panel UI, persistent storage, multiple modules, or the architecture of either case study.
- For simple DOM enhancements, styling, keyboard shortcuts, form helpers, or other small scripts, use the simplest suitable implementation.
- Treat every website as a new system. Never assume it shares another site's endpoints, authentication, signatures, encryption, data schema, limits, or anti-abuse behavior.
- Use only the sections relevant to the current task. Skip irrelevant checklist items.
- User requirements, observed site behavior, documented site interfaces, security/privacy boundaries, and legal/terms constraints take precedence over case-study patterns.
- Do not add abstractions or complexity merely to comply with this skill.

## Workflow

### 1. Understand the task

Establish the desired behavior, inputs, outputs, supported pages, success criteria, and expected lifecycle. Identify whether this is a new script, an extension of an existing script, or a debugging task.

### 2. Inspect the target site before choosing an approach

Determine whether the data/action is best handled through:
- DOM inspection/manipulation;
- data already present in the page or frontend state;
- a site interface/API;
- a combination of these.

Do not default to API calls. If using an API, investigate method, path, parameters, headers, session requirements, response schema, business errors, pagination, and rate limits from normal site behavior. Do not invent endpoints or infer undocumented semantics from field names alone.

When collecting browser diagnostics, warn that HAR/Network exports may contain credentials and personal data. Ask for sanitized, minimal evidence only. Never ask the user to provide passwords, cookies, or tokens.

### 3. Design proportionally

Keep responsibilities clear when complexity warrants it (for example: input parsing, site adapter, data normalization, task scheduling, presentation). A small script may remain a single file. Split by responsibility and change boundaries, not by line count.

Prefer minimal permissions, least-privilege host matching, and the site's normal browser session behavior. Avoid unnecessary global state, duplicate sources of truth, dependencies, and persistence.

### 4. Treat authentication and protocols as site-specific

Confirm exactly which requests require the current page's session, CSRF data, signatures, or other credentials. Send credentials only to the intended origin and only when required. Do not expose or persist secrets.

If a site uses signing, encryption, compression, or dynamic protocol steps, investigate and validate each stage. Encapsulate site-specific protocol logic in that site's adapter. Never copy a protocol from another site without evidence.

Case-study examples only (not defaults):
- JavDB: same-origin page session for movie search; its Reviews API signature flow; no JavDB Cookie sent to the Reviews API.
- LTX: its observed client protocol used ECDH P-256, HKDF-SHA256, AES-256-GCM, and Gzip processing.

These details are not general requirements and must not be presumed relevant to another site.

### 5. Protect data correctness

- Normalize identifiers deliberately and use exact matching where the task requires identity matching.
- Distinguish confirmed present, confirmed absent, and failed/unknown.
- Never interpret a timeout, failed page, malformed response, or incomplete pagination as “not found.”
- Verify pagination completeness and consistency before making negative claims.
- Separate core results from optional enrichment (such as resource links, images, or comments), so enrichment failure does not invalidate a confirmed core result.
- Treat page and API data as untrusted input.

### 6. Handle batch work only when needed

For multi-request or long-running tasks:
- Start with serial or low-concurrency requests unless site behavior and evidence justify otherwise.
- Choose batch size and interval based on observed constraints; there is no universal default.
- Define task states and pause/resume/stop behavior when those controls are useful.
- Set sensible input limits and progress feedback.
- Define what happens on refresh, tab close, and navigation.
- Prefer in-memory state unless persistence is genuinely needed and its privacy/lifecycle are clear.

Do not use concurrency, proxy rotation, or retries to evade rate limits or access controls.

### 7. Classify errors and retry safely

Differentiate HTTP status, business-level errors, network failures, malformed responses, and protocol/decryption failures.

Typical handling guidance:
- 401 / login required: explain session or permission requirement.
- 403 / access denied: pause or stop affected work; do not loop retries.
- CAPTCHA / Cloudflare / challenge: tell the user normal user interaction is required; do not bypass it.
- 429: respect Retry-After; use a bounded wait and then manual pause if necessary.
- 5xx / transient network failure: only bounded retries with backoff when safe and appropriate.
- Unknown response schema: report incompatibility; do not silently convert it to empty success.

Check idempotency before retrying. Never retry indefinitely.

### 8. Security and privacy

Do not request or log passwords, cookies, or tokens. Avoid sending session credentials to third parties. Minimize permissions, data collection, storage, and telemetry. Sanitize logs and diagnostic material. Do not bypass login, paywalls, CAPTCHA, anti-abuse systems, or access controls.

### 9. Test at the appropriate level

Use proportional testing:
- static checks and syntax/lint;
- unit tests for parsing, normalization, matching, pagination, and error classification;
- mocked responses for normal and failure paths;
- real-browser tests when site session, browser permissions, CORS, dynamic protocols, or live site behavior matter;
- user acceptance after the user installs and runs the script.

Do not describe static or mocked tests as real-site verification. Record version, environment, date, and observed limitations where useful.

### 10. Release and maintain

When appropriate, use a reviewable workflow: implement → inspect → test → real-site verify → document → review/merge → publish/update. Keep version metadata and update URLs accurate. Avoid changing unrelated projects or deploying without authorization.

## Reusable checklist

Use only applicable items:
- [ ] User goal and acceptance criteria are clear.
- [ ] Correct integration method selected (DOM, page state, API, or hybrid).
- [ ] Site-specific interface and session requirements verified, if applicable.
- [ ] Identifier normalization and exact matching validated, if applicable.
- [ ] Pagination completeness validated, if applicable.
- [ ] Unknown/failure is not mislabeled as absence or success.
- [ ] Batch limits, task lifecycle, and pacing are appropriate, if applicable.
- [ ] Errors are classified and retries are bounded.
- [ ] Permissions, secrets, and persistence are minimized.
- [ ] Appropriate automated and real-browser tests completed.
- [ ] Claims about verification accurately reflect what was tested.
- [ ] Versioning, release, and user update instructions are correct.

## Reference document

The full Chinese engineering guide and case studies are maintained in this repository:

`docs/跨网站油猴脚本开发规范.md`

Read the full document when a task needs deeper guidance or when the user asks to apply the complete workflow. For routine tasks, this Skill is sufficient; do not load the entire document unnecessarily.

## Final decision rule

Reuse the investigation method, engineering judgment, and reliability lessons. Do not reuse site-specific implementation without evidence. The purpose is to help choose the right solution for the current website—not to force every website into the same solution.
