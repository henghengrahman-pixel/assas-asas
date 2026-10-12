# Audit and actual fixes

1.11.1 adds the user-clarified read-only report-query workflow. See README-SESSION-AGENT-PT.md for live-job validation, connection-time IP checks, limits and simulation labels.

Architecture: built-in Node HTTP server, query-string forms, manual routes/session lookup/CSRF, scrypt passwords, TOTP enrollment, AES-256-GCM agent vault, atomic JSON on volume, public HTTPS feeds, HTML catalog/period parsers, vendor QR code. No SQL database or ORM. All original source files, vendor assets, routes and version notes are retained.

| Reproduction / bug | Root cause and affected file | Actual fix / evidence |
|---|---|---|
| Click live audit submit; browser shows Failed to fetch before a job exists | CSP default-src none without connect-src; server.js | Add connect-src self; real HTTP policy regression test |
| Exports visible before a job exists | Button display CSS overrides hidden attribute; public/style.css | Global [hidden] display:none!important, versioned assets; response/CSS regression assertions |
| Import a valid legacy JSON store containing an unknown field, then save | `readStore` returned a fixed field projection; server.js | Preserve original fields via spread, additive ptReports; HTTP restart and legacy tests |
| Reach 10,001 market records, 5,001 audits or 1,001 backtests | Implicit slicing discarded oldest persisted data; server.js, market-feed.js | Remove automatic retention slicing; UI limits remain display-only |
| Import malformed calendar date | Invalid Date `.toISOString()` throws 500; server.js | Validate finiteness before formatting; actual HTTP request returns 400 |
| Catalog has two IDs with same name or malformed percent encoding | Name-keyed Map collapses IDs; decodeURIComponent throws; agent-readonly.js | Key catalog by site-local ID; handle invalid URI; regression test |
| HTTP 200 page lacks valid period | Empty technical reason; agent-readonly.js | PERIOD_FIELD_NOT_FOUND; never infer success from HTTP status |
| Period exists only on explicitly selected period dropdown | Input-only parser; agent-readonly.js | Parse selected options only in period fields, preserve zeros; regression test |
| Catalog request redirects to login | Some catalog failures swallowed; agent-readonly.js | Expiration/login redirects propagate and authentication fails |
| DNS lookup stalls or large catalog requests repeatedly timeout | DNS not bounded and only per-request deadlines; agent-readonly.js, market-feed.js | Whole agent deadline and bounded DNS/feed reads; stalled-fetch/DNS tests |
| Large HTML body streamed upstream | Buffering entire body before size validation; agent-readonly.js | Stream with hard size ceiling; reject oversized body in test |
| IPv4-mapped private IPv6 resolves | Incomplete agent IP policy; agent-readonly.js | Reuse published-feed reserved-address policy; rejection test |
| Invalid cookie saved, or website names collide | Cookie only validated when probing; feed mapping keyed by duplicate name; server.js | Validate cookie before save; reject duplicate names; enforce URL origin/session scope |
| Enrolled TOTP is readable directly in JSON | Plaintext security.totpSecret; server.js | Non-destructive encryption on next save using ENV key; same OTP continues working; legacy test and actual process restart |
| POST login sent cross-origin | No login origin guard; server.js | Reject cross-site browser metadata or mismatched Origin; legacy test |
| Server exceptions/log caching leak sensitive context | Full Error object and no blanket private-page cache control; server.js | Log exception type only; no-store for authenticated/auth pages and exports |
| More than 12 websites or a filtered analysis total | First 12 columns but ALL counted all records; server.js | Show selected registered websites and calculate historical frequency within that selection |
| Report PT unavailable | No actual structured module; server.js/public | Add pt-reports.js, authenticated import/recap/export endpoints, dedicated menu and file picker |
| Mixed exact periods, duplicate files, fabricated missing zeros | Missing report schema/persistence | Strict source schema, immutable conflicts, exact string period matching, null semantics, deterministic identities and atomic merge |
| Dangerous formula text / pseudo Excel file | Export absent | True OOXML ZIP with typed number/text cells; CSV header formula guarding; independently opened with openpyxl |
| PT module omitted from container | New runtime dependency not copied | Update Dockerfile to include module and lockfile; DATA_DIR default /data; Docker execution unverified |
| Quoted .env value parsed literally | Minimal environment loader; server.js | Strip enclosing quote pair while preserving ENV precedence |

Existing routes inspected: login/logout, QR/setup/activate/cancel 2FA, website create/update/delete, session check/revoke, market sync/import, backtest, all view navigation, health and static assets. New PT endpoints sit behind the same authenticated-stage and CSRF gates. Published-report endpoints do not forward candidate numbers. The separately labeled audit connector forwards a requested number only to the read-only source report form; no endpoint makes settlement/draw writes.

Known limits: no live agent account or Railway access; parser authentication proof remains specific to known source HTML markers; DNS preflight is not connection-pinned; one-replica JSON storage; source report authenticity is the uploader's responsibility; browser/Docker execution unavailable. See TEST-REPORT.md. These are not marked as passed.

1.11.2 UI correction: two forms on /predictions confused direct PT readings with historical frequency analysis. The screen now contains one Session Agent report form. Historical functionality remains at /historical-analysis, using its own script and GET target. Site names are table headers; source periods/status remain separate, and history is collapsed by default. No existing storage fields or report records removed.

1.11.3 fixes stale market-ID use: previously jobs used cached catalogs and skipped newly available markets. Jobs now include selected websites, read fresh catalogs, resolve one normalized name per site, and reject missing/ambiguous names. Tests confirm obsolete IDs are not used for POST requests. Numeric parsing previously rejected integer .00/,00 display formats and explicit SERI zero; added strict support without rounding or turning unknown values into zero. The actual previously reported DENTOTO value format is not yet supplied and is not claimed verified.

1.11.4: reproduced the catalog discovery failure with mock empty root prediction/menu responses. The new connector omitted /admin_angka13.php used by the existing connector and did not attempt parameterized prediction pages. Add the original third catalog source and at most three saved ID hints for GET-only discovery. A fresh parsed name-to-ID mapping and validated target form remain mandatory before report POST. Blank discovery never fabricates a connected state; expired/redirect/401/403 failures halt. Human-readable reason text and collapsible endpoint diagnostics now show discovery evidence. Actual Railway/upstream responses were not accessible.

1.11.5: repeated PT format failures previously continued through all 100 queries with only a generic reason. Structural/value failures now halt the affected website and persist bounded sanitized diagnostics from the recognized total row. Other sites continue and prior valid readings remain. This addresses missing diagnostic evidence and repeated failing requests; it does not establish that the live monetary format has been corrected.

1.11.6 re-audit: four prior-version reproductions confirm mixed thousand separators incorrectly inflated values (agent-pt.js), data-name/data-href accepted as real attributes (agent-pt.js/agent-readonly.js), deleted final cookie reused (agent-pt.js), and unavailable XLSX ALL exported as empty string (pt-reports.js). Fixes verified by new regression tests. Client polling now discards responses for no-longer-selected jobs and aborts stalled fetch after 15 seconds. Client tests execute actual source in a mock DOM; no visual browser claim.

1.11.7: login origin was compared only to backend Host (unless APP_ORIGIN existed), rejecting legitimate public HTTPS login under a differing internal proxy host. Canonical public origin is now resolved from APP_ORIGIN, then RAILWAY_PUBLIC_DOMAIN, then direct connection host/protocol. Reject cross-site metadata, malformed or mismatched full origins and spoofed forwarded-header substitutions. Keep password/TOTP/session/CSRF unchanged. Invalid configuration produces a readable 503 login page. Regression simulates the proxy; actual Railway headers are not available.

1.11.8: the supplied Variables screenshot confirms APP_ORIGIN text matches the public URL, so the earlier suggestion that quote formatting caused the failure was unsupported. Generic rejection messages did not expose the expected/received origin or active application version. Add safe canonical-origin diagnostics and version header/footer, and normalize configured URLs by trimming outer whitespace and using their parsed origin. Retain exact origin matching, cross-site denial and all existing authentication gates. Live failure root cause remains unverified pending runtime diagnostic values.

1.11.9: identified the previously missed Referrer-Policy no-referrer / form-Origin conflict in server.js. Standard HTML POST navigation under no-referrer may carry Origin null and fail the strict validator even with correct APP_ORIGIN. Switch response policy to same-origin; retain strict validation and cross-site denial. Raw live Origin remains unavailable, so consistency with the reported INVALID status is an inference. Malformed and literal null origins now have distinct safe diagnostics.
