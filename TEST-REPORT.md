# Test report — local evidence, 11 October 2026

Current input: the user-attached ATLAS-CONTROL-SUITE-V1.11.2-SINGLE-SESSION-AGENT-PANEL (1).zip. Earlier audits used the attached 1.9.7 (3) archive.

Baseline: 26 original tests passed before changes; the preceding 1.10.0 revision had 42 passing tests. This revision adds read-only PT audit after the user clarified that source queries are for reporting, not outcome selection. Current run: **71 tests passed, 0 failed, 0 skipped**, with complete JavaScript syntax checks. Execution runtime: Node v24.19.0 / npm 11.9.0. Node 22 Docker runtime was not executed here. Actual command and output: `npm run check`, TEST-OUTPUT.txt. Some old tests assert source code structure only; they are retained and not presented as functional proof. New functional tests exercise the modules and local HTTP.

| Check | Final outcome / evidence |
|---|---|
| Browser request policy / hidden controls | PASS: actual local HTTP headers permit same-origin fetch only; versioned CSS/JS and hidden export markup checked. Browser rendering not executed |
| Direct PT report audit | PASS: form fields from provided source, strict PT label/amount/echo parsing, 100 numbers per website, source-specific IDs and exact periods; mock upstream, no live account |
| Direct PT HTTP API | PASS: auth/CSRF, background job, status, 200 POST report queries across two mock websites, one invalid-session website handled independently, exports, reload, interrupted jobs |
| Cookie/token changes and cancellation | PASS: source cookie rotation, form token refresh, expired-session halt, cancellation retains previous readings |
| JavaScript syntax | PASS: all shipped .js recursively |
| ZIP extraction and dependency workflow | PASS: ZIP CRC/integrity, complete extraction, offline npm ci and all JavaScript syntax checks from extracted source |
| Local startup and health | PASS: child `node server.js` process and HTTP /healthz |
| Admin password, first enrollment, QR, TOTP login | PASS: existing enrollment test plus local HTTP |
| Existing TOTP migration | PASS: same authenticator accepted, no reenrollment, no plaintext secret persisted |
| CSRF / authentication / cross-origin login | PASS: HTTP mutation denial, report export protection, origin rejection |
| Session encryption/decryption/tamper / host change | PASS: AES-GCM tests, round trip, missing/wrong key, forbidden cookie reuse |
| Catalog / period parsing / leading zeros | PASS: parser regressions and fake upstream functional responses |
| Session expired, redirect, 403, bad content type, body too big | PASS: simulated upstream outcomes, not live agent |
| Whole-agent stalled fetch/DNS deadline | PASS: injected stalled dependencies rejected |
| Valid JSON and CSV reports | PASS: schema tests, actual JSON HTTP import |
| Invalid imports / conflict / ambiguous market IDs | PASS: strict validation, immutable/atomic merge tests and HTTP denial |
| Positive/negative values / partial missing / real zero / ALL | PASS: source-equivalent synthetic fixture arithmetic |
| Different website market IDs and exact periods | PASS: p1 vs p99; 00123 stays separate from 123 |
| Duplicate import | PASS: same structured report reimport does not grow storage |
| Pagination/search data | PASS: actual HTTP pagination; recap search function; browser interaction not run |
| CSV/XLSX export | PASS: authenticated HTTP downloads and typed OOXML; XLSX separately opened with Python openpyxl, values ('0001', 5000, -29000, -24000) and source sheet verified |
| Data after actual process restart | PASS: report recap, existing data, unknown legacy metadata and authenticated cookie survive |
| Original features | PASS: original 26 tests remain, three enrollment tests now obtain secret from the authorized enrollment HTML rather than plaintext disk |
| Responsive visual rendering | **BLOCKED / NOT VERIFIED**: Playwright available, Chromium executable missing; installation failed because downloaded browser archive was invalid/empty. Optional scripts/test-ui.js supplied, not counted as passed |
| Docker build/runtime | NOT RUN: Docker CLI unavailable. Configuration/source copies inspected only |
| Live Railway / agent/source website | NOT RUN: no authorized credentials/account/session/deployment access supplied |
| Real production PT values | NOT VERIFIED: no authorized PT report file supplied; no synthetic data seeded into normal application storage |

Earlier in the 1.10.0 revision, 3 new mock-upstream tests initially failed because native constructed Response objects have an empty URL. The reader now checks the known request target as fallback, and the full suite was rerun successfully. Browser launch and download failures remain disclosed above and were not relabeled as success.

Direct PT audit transport now checks IP policy at actual connection-time DNS resolution. The prior preflight-only limitation still applies to older catalog/feed paths; it is not claimed universally fixed.

The delivered build is locally verified for the Node suite, not claimed universally production-ready. Preserve ENV and /data during deployment, then perform live smoke testing with your authorized account and actual exported reports.

1.11.1 corrects a confirmed defect visible in the supplied screenshot: CSP default-src none blocked same-origin fetch because connect-src was absent. Earlier non-browser tests missed this. Added explicit connect-src self, regression assertions on real HTTP responses, a global hidden CSS rule, and versioned assets to avoid stale browser cache. This validates configuration and markup, not live browser/upstream behavior.

1.11.2: 53 tests passed, 0 failed, 0 skipped after separating the main Session Agent screen from historical analysis. The authenticated HTTP regression checks exactly one live report form, no historical form/results on /predictions, and a working historical form at /historical-analysis. The existing two-source job test verifies 100 rows per website and exact signed values. Browser visual rendering and live upstream/Railway remain unverified.

1.11.3: final local run **58 passed, 0 failed, 0 skipped**. New tests cover grouped currency, integer decimal suffixes, entities/minus, explicit SERI zero, fractional/ambiguous value rejection, the provided DENTOTO unsubmitted HTML, fresh normalized market-name discovery, missing/ambiguous names and expired catalog. The authenticated HTTP test now starts with stale IDs p777/p888 and confirms fresh p1/p99 are discovered before 200 source POST reads. One invalid-session site remains independent. An initial new expiration test failed because the injected mock transport did not implement the native transport's expiration check; explicit catalog expiration validation was added and the complete suite passed on rerun.

No submitted-number DENTOTO HTML has been provided. Its exact PT_VALUE_INVALID trigger remains unverified; format support tests are not represented as proof of fixing that live response. Browser/Docker/Railway and live accounts remain untested as documented above.

1.11.4: final local suite **62 passed, 0 failed, 0 skipped**. Four new regressions exercise empty prediction/menu roots with angka13 fallback, cached-ID GET discovery yielding a changed live ID, complete discovery failure with zero POSTs, and HTTP 404 fallback/session-expiry halt. The full authenticated two-site HTTP test now exercises these paths: website A has empty root pages and discovers p1 through a GET hint p777; website B discovers p99 via admin_angka13. Each reads 100 signed reports and exports/persists them. Diagnostic endpoints contain no session credentials. The screenshot's actual root HTML is unavailable; live resolution remains unverified.

1.11.5: **64 passed, 0 failed, 0 skipped**. New functional regressions verify safe numeric diagnostics and masking of arbitrary/script/credential-like text, stop on first amount-format failure with earlier readings retained, and continued processing of an independent website. The live format error remains unresolved pending a submitted-number response or new diagnostics. No expanded monetary syntax is claimed from the screenshot alone.

1.11.6: final suite **70 passed, 0 failed, 0 skipped**. Four regressions were separately run against the prior 1.11.5 code and produced four expected failures (REPRODUCED-BUGS.txt); after fixes they pass. They cover mixed amount grouping, misleading data-* attribute matches, deleted/expired cookies and unavailable XLSX totals. Two tests execute shipped client JavaScript in a minimal DOM harness and verify stale-response rejection and 15-second abort/failure handling; these are not visual browser tests. Live PT_VALUE_INVALID remains unresolved pending actual source value diagnostics. Browser visual rendering, Docker execution, live source accounts and Railway remain unverified.

1.11.7 — input adasas-asa-main.zip: **71 tests passed, 0 failed, 0 skipped**. Added production-mode HTTP login-origin regression covering internal Host plus public Railway origin, exact APP_ORIGIN override, scheme mismatch, malformed/null origin, foreign origin, forged X-Forwarded-* headers, cross-site metadata, bad credentials, actual TOTP activation/login and production cookie flags. A first test run found malformed Origin was incorrectly classified as invalid configuration (503); origin parsing was separated and the full suite passed after correction. Live Railway/browser cookie behavior remains unverified; no user credentials or data were used in tests.

1.11.8 — user-attached 1.11.7 ZIP: **71 tests passed, 0 failed, 0 skipped**. Extended the production proxy-origin test with safe runtime version/expected/received/source diagnostics, configured URL whitespace/path canonicalization and masking of credential-bearing incoming origins. Tests continue verifying cross-site rejection, untrusted forwarded headers, bad credentials, TOTP and secure cookie flags. Actual incoming headers for the user's live failure are unavailable; this is an origin-normalization/diagnostic update, not proof that the live rejection is resolved.

1.11.9: **71 passed, 0 failed, 0 skipped**. Extended production HTTP regression checks the real login document emits Referrer-Policy same-origin, confirms null-origin login remains 403 with NULL_ORIGIN diagnosis, and preserves proxy-origin login/TOTP/security checks. Browser navigation and live Railway untested. Found a concrete browser-policy conflict: no-referrer can make form POST Origin null while the login validator rejects it. Changed policy rather than accepting null or relaxing origin validation. Primary reference is linked in README.
