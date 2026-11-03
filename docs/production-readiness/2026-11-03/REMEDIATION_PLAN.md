# habitat-relay — remediation plan

Planned actions only.

## Phase 0 — baseline and deployment contract
**Entry:** current review accepted as the baseline.  
**Exit:** owner-approved deployment model and verification scope exist.

### HR-T1 — Freeze the product/deployment contract
Linked to: HR-1, HR-2, HR-3.  
Observed paths: `README.md`, `package.json`, `src/server.js`, `public/js/app.js`.  
Prerequisites: owner names intended host; owner identifies operator role; owner confirms whether any public/shared preview is allowed.
1. Record whether Habitat Relay remains local-only, LAN-only, or Internet-hosted, and whether Twitch mode is demo-only, staged live, or production live.
2. Define the trusted reverse-proxy tier, canonical public base URL, and which component may set `Host` and `X-Forwarded-*`.
3. Write an environment/config matrix for `PORT`, `TWITCH_REDIRECT_URI`, `HABITAT_DB_PATH`, `HABITAT_ENCRYPTION_KEY`, and Twitch app credentials, including who owns each secret.
Output: signed-off architecture and environment matrix.  
Verification: cross-check every trust assumption against code.  
DoD: [ ] hosting model fixed [ ] public base URL fixed [ ] proxy ownership fixed [ ] secret owners named.

## Phase 1 — operator authorization and public-surface reduction
**Entry:** Phase 0 complete.  
**Exit:** every operator/control route has an intentional auth model, or the app is explicitly restricted to trusted-local use.

### HR-T2 — Harden operator-only routes
Linked to: HR-1.  
Observed paths: `src/server.js`, `README.md`, `tests/http.test.js`, `tests/e2e/habitat-relay.spec.js`.  
Prerequisites: Phase 0 contract; chosen auth mechanism; approved operator and denied personas.
1. Enumerate all operator-sensitive routes and streams: `/api/operator/overlay-link`, `/api/operator/status`, `/api/events`, command routes, and Twitch connect/disconnect.
2. Choose one control model: authenticated session, local-only binding plus network ACL, or proxy-enforced access control; do not rely on the overlay key alone.
3. Add negative tests for unauthenticated/unauthorized reads and writes, including a public viewer who can load the main app but must not mint a fresh overlay URL or inspect operator event history.
Output: operator access-control design and regression suite.  
Verification: disable the auth gate in a test build and prove the negative tests fail.  
DoD: [ ] sensitive routes inventory complete [ ] explicit auth/containment chosen [ ] unauthorized read blocked [ ] unauthorized write blocked.

### HR-T3 — Separate read-only viewer capabilities from operator controls
Linked to: HR-1 preventive requirement.  
Observed paths: `src/server.js`, `public/js/app.js`, `tests/http.test.js`.  
Prerequisites: T2 route inventory; owner approves whether the overlay remains a capability URL; event-history exposure decision made.
1. Keep overlay capability links session-scoped and read-only, but remove operator-only discovery from the public app shell unless the caller is authorized.
2. Review whether `/api/events` and `/api/operator/status` are needed outside the studio; if not, gate them.
3. Update documentation so public/demo guidance cannot be mistaken for safe remote operator hosting.
Output: reduced public surface and updated docs.  
Verification: request matrix proves overlay consumers can read only what they need.  
DoD: [ ] overlay discovery path constrained [ ] unnecessary operator reads removed or gated [ ] docs match deployed intent.

## Phase 2 — proxy, origin, and OAuth boundary verification
**Entry:** operator model chosen.  
**Exit:** trusted-header behavior is explicit and negative-tested.

### HR-T4 — Make proxy trust explicit and test forged headers
Linked to: HR-2.  
Observed paths: `src/server.js`, `tests/http.test.js`, `README.md`.  
Prerequisites: T1 public URL; selected reverse proxy; staging environment where forged headers can be simulated.
1. Replace implicit header trust with a documented rule: either ignore `X-Forwarded-*` unless the request came from the trusted proxy, or inject a canonical configured origin.
2. Add integration tests for missing `Origin`, foreign `Origin`, forged `Host`, forged `X-Forwarded-Host`, and forged `X-Forwarded-Proto` across command routes, overlay-link generation, and OAuth cookie issuance.
3. Verify redirect URI generation, overlay URL generation, and OAuth callback cookies against the final public origin.
Output: trusted-proxy policy and origin/header regression tests.  
Verification: at least one forged-header test must fail before the fix and pass after the trust policy is enforced.  
DoD: [ ] trusted proxy rule written [ ] forged-header tests exist [ ] staging cookie/URL behavior observed [ ] redirect URI matches owner-approved public URL.

## Phase 3 — data retention, release, rollback, and recovery
**Entry:** network and auth boundaries settled.  
**Exit:** data lifecycle and failure recovery are operational, not just described.

### HR-T5 — Define retention and secret/key lifecycle
Linked to: HR-3.  
Observed paths: `README.md`, `tests/store.test.js`, `tests/service.test.js`.  
Prerequisites: owner privacy decision; selected backup location; named secret custodian.
1. Classify every stored item: active state, event history, accepted voter hashes, delivery IDs, encrypted Twitch tokens, and recap exports.
2. Set retention/deletion rules for each class, including what reset deletes and what persists.
3. Define encryption-key loss/rotation behavior and the expected operator runbook when stored Twitch credentials become undecryptable.
Output: retention and secret-lifecycle policy.  
Verification: check that the policy matches the implementation visible in tests and storage code.  
DoD: [ ] data classes listed [ ] deletion windows approved [ ] key rotation/loss runbook written [ ] recap/export handling defined.

### HR-T6 — Rehearse release, rollback, and recovery
Linked to: HR-3 preventive requirement.  
Observed paths: `package.json`, `README.md`; proposed paths: deployment/runbook docs to be added by owner team.  
Prerequisites: T5 policy; staging deployment; immutable artifact strategy.
1. Define the release artifact, deployed SHA recording, environment promotion path, and smoke-test checklist.
2. Rehearse SQLite backup creation, restore into a clean environment, Twitch disconnect/reconnect handling, and overlay-link revocation.
3. Define rollback triggers, rollback steps, and post-rollback data consistency checks, including what happens to live sessions and stored encrypted tokens.
Output: release/rollback/recovery runbook with observed rehearsal evidence.  
Verification: an independent reviewer watches one restore and one rollback drill.  
DoD: [ ] release artifact identified [ ] restore observed [ ] rollback observed [ ] post-restore consistency checked.

## Phase 4 — meaningful live verification and independent acceptance
**Entry:** prior controls implemented in staging.  
**Exit:** owner has observed live-mode acceptance evidence or keeps the product labeled demo/local-only.

### HR-T7 — Verify live Twitch behavior without overstating coverage
Linked to: open blocker on unverified provider path.  
Observed paths: `README.md`, `tests/twitch.test.js`, `tests/e2e/habitat-relay.spec.js`; proposed path: staged verification notes.
Prerequisites: approved Twitch app; approved broadcaster test account; staging callback URL; written moderation/safety constraints.
1. Run a staged OAuth authorization, callback, scope check, EventSub subscribe, reconnect, revocation, and disconnect flow.
2. Send controlled chat messages and verify one-vote-per-round, pause-on-disconnect, and no raw identities in recap data.
3. Record remaining unverified cases explicitly.
Output: staged live-verification report.  
Verification: observe at least one negative live case, such as revoked scope or reconnect interruption.  
DoD: [ ] OAuth observed [ ] EventSub observed [ ] negative live case observed [ ] remaining unknowns documented.

### HR-T8 — Independent acceptance across five roles
Linked to all findings and preventive requirements.  
Prerequisites: T2-T7 complete; owner nominates independent reviewer(s); staging evidence available.
1. Correctness reviewer validates the command/vote/recap flow and broken-path failures.
2. Automation/testing reviewer validates regression coverage, negative tests, and mocked-versus-live verification boundaries.
3. Security reviewer validates auth, header trust, and secret/retention controls.
4. Operations reviewer validates release, restore, and rollback drills.
5. Accessibility/UI reviewer validates keyboard, screen-reader, OBS readability, and 200% zoom on the deployed surface.
Output: independent acceptance memo with pass/fail by discipline.  
Verification: each reviewer must cite observed evidence; green counts alone are insufficient.  
DoD: [ ] correctness sign-off [ ] automation/testing sign-off [ ] security sign-off [ ] ops sign-off [ ] UI/accessibility sign-off [ ] unresolved risks accepted by owner.

## Task dependency map
| Task | Depends on | Unlocks |
|---|---|---|
| HR-T1 | owner availability | HR-T2, HR-T4, HR-T5 |
| HR-T2 | HR-T1 | HR-T3, HR-T8 |
| HR-T3 | HR-T2 | HR-T8 |
| HR-T4 | HR-T1 | HR-T7, HR-T8 |
| HR-T5 | HR-T1 | HR-T6, HR-T8 |
| HR-T6 | HR-T5 | HR-T8 |
| HR-T7 | HR-T2, HR-T4, owner Twitch approvals | HR-T8 |
| HR-T8 | HR-T2 through HR-T7 | release decision |

## Owner approvals required
- Any move from trusted-local/demo use to shared or public hosting.
- The operator authentication model and identity provider, if any.
- Twitch developer app, broadcaster test account, and redirect URIs.
- Data-retention windows, backup location, and encryption-key custody.
- Whether historical events remain stored after session reset.

## Release, rollback, and recovery policy
- **Release:** promote only an immutable artifact tied to a recorded SHA.
- **Rollback:** revert to the previous verified artifact when auth, header trust, Twitch state, or recap privacy checks fail.
- **Recovery:** restore only from an approved encrypted backup, then verify active state, token decryptability, and overlay-key invalidation.

## Independent acceptance checklist
- [ ] Public/demo labeling still matches the real deployment.
- [ ] Operator-only routes are gated or deployment is provably trusted-local.
- [ ] Forged-header negative tests fail when trust rules are broken.
- [ ] Restore and rollback were observed, not inferred.
- [ ] Live Twitch verification is evidenced, or the app remains explicitly non-live/demo-only.
