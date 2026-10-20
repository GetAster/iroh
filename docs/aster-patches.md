# Aster patch ledger: iroh

This repository is Aster's fork of [n0-computer/iroh](https://github.com/n0-computer/iroh). Aster carries the patches listed below on top of upstream releases. Upstream maintainers are welcome to incorporate any of them, in whole or in part, at their discretion.

This repository also holds the canonical fork manifest, `docs/aster-fork-manifest.toml`, for the whole Aster iroh stack (noq, iroh, iroh-blobs, iroh-docs, iroh-gossip, iroh-util, plus the Salvo record). See "Porting to a new upstream release".

## Branch and tag model

- `main` is the front door: upstream's README with an Aster preamble, plus this ledger. Nothing is built from `main`.
- `upgrade/<version>` is the patched integration line for one upstream release wave: the upstream release plus the patches in this ledger. Consumers pin it by commit.
- Topic branches carry one change each and are named for what the change is, for example `feat/read-into` or `security/plumtree-bounds`. Each is based on the upstream release tag and is cherry-picked with `git cherry-pick -x` onto the upgrade branch, so every upgrade-branch commit names its source.
- Release tags mark the commits that are released and pinned. Published tags are never moved or rewritten. Historical tags in this repository use several schemes (listed below); new tags use `aster-<repo>-v<version>`, where `<version>` is the published version of the repository's main package (for example `aster-salvo-v1.0.4`, `aster-fory-v1.7.6-aster.1`).
- `main` is refreshed by merging upstream `main` into it. It is not force-pushed.

Tag schemes used in this repository:

- `aster-iroh-v<version>`, for example `aster-iroh-v1.0.0-rc.0`, `aster-iroh-v1.0.1`, `aster-iroh-v1.3.2`.
- `aster-iroh-v<version>-p<n>`, a later commit at the same crate version: docs-only retags (`aster-iroh-v1.0.0-rc.1-p1`, `-rc.1-p2`, `aster-iroh-v1.0.0-p1`) or pin-only changes (`aster-iroh-v1.0.1-p1`, `aster-iroh-v1.3.0-p1`).

## Current release

| Field | Value |
|---|---|
| Upstream base | `v1.3.0` (`0072d7d84b`) |
| Integration branch | `upgrade/iroh-v1.3.0` |
| Tag | `aster-iroh-v1.3.2` |
| Pinned commit | `a4a9a76b01bcfbad573dfc3186b014efa1c2532d` |
| Crate version | 1.3.2 |
| Crates pinned from this fork | `iroh`, `iroh-base`, `iroh-relay` |
| Crates published to the Aster registry | `iroh`, `iroh-base`, `iroh-dns`, `iroh-relay` |
| Source repository | `https://forge.emrul.dev/Aster/iroh.git` |

This fork depends on the Aster noq fork at 1.3.2 (`e75f3a5550feae0f40df0ffde8bf438f213a5156`, tag `aster-noq-v1.3.2`) through row 3.

Registry bridges (unmodified crates.io source republished to the Aster registry with only dependency metadata rewritten, so an alternate-registry build never pulls a second copy of iroh):

| Bridge | Source | Commit | Version | Rewrites |
|---|---|---|---|---|
| `iroh-mdns-address-lookup` | `https://github.com/n0-computer/iroh-address-lookups` | `87fdd7ea1a902845695f8f685c1905f096b3d02d` | 0.4.0 | `iroh`, `iroh-base` |
| `iroh-tickets` | `https://github.com/n0-computer/iroh-tickets` | `4899684e1209dea29ed28aa2314450927557b037` | 1.0.0 | `iroh-base` |

Transitive crates.io constraints forced by upstream's dependency updates (not forks): `iroh-tickets = "=1.0.0"`; `iroh-mdns-address-lookup = "0.4"` (0.4.0 tracks iroh ^1.0.0; 0.3 pinned rc.1); `ed25519-dalek = "3.0.0"` (iroh 1.3.0 requires `>=3.0.0-rc.0,<4.0.0`; the lock resolves 3.0.0); `netwatch` and `portmapper` 0.19.3 (net-tools `0030e57fc70981895bb86212ff0f059cc41c8a4e`). `n0-dns-resolver` 0.1.0 replaces hickory as a transitive dependency of `iroh-dns`; its normal dependencies include no noq or iroh crate (only a dev-dependency on `iroh-base`), so it needs no registry bridge.

Bootstrap pin block for source-tree builds (consumers install a versioned Aster SDK release instead and must not copy this):

```toml
[patch.crates-io]
iroh       = { git = "https://forge.emrul.dev/Aster/iroh.git", rev = "a4a9a76b01bcfbad573dfc3186b014efa1c2532d" }
iroh-base  = { git = "https://forge.emrul.dev/Aster/iroh.git", rev = "a4a9a76b01bcfbad573dfc3186b014efa1c2532d" }
iroh-relay = { git = "https://forge.emrul.dev/Aster/iroh.git", rev = "a4a9a76b01bcfbad573dfc3186b014efa1c2532d" }
```

The workspace version spec is `iroh = "=1.3.2"`; the exact-pin version must track the fork's crate version.

### `upgrade/iroh-v1.3.0` stack

| Commit | Subject | Source |
|---:|---|---|
| `b5afe1aa30` | feat(iroh): re-export noq::poll_driver for FFI consumers | `-x` of `4abc891ef4` (row 4) |
| `5e92f80704` | chore: add docs/aster/ to gitignore for internal review docs | `-x` of `84f72a7835` |
| `52e3ef017c` | fix(socket): throttle transport recv-error logging to prevent log flood | `-x` of `4114c449c5` (row 5, conflict, see History) |
| `6db27d8eeb` | chore: run cargo fmt after iroh 1.0.1 patch port | `-x` of `32921714bb` |
| `0a70368262` | build: resolve noq from the Aster 1.3.0 fork | new (row 3); replaces the 1.0.1-wave `a9c547549e`/`ae7e3ae989`/`c29bcd6ce2`/`5228d64d91`/`b15ae80713` patch, pin, lock and URL commits |
| `1974e95825` | build: resolve noq from the Aster 1.3.1 fork | new (row 3) |
| `fc873f0b45` | fix(socket): reach a remote that restarted elsewhere while an old connection is open | `-x` of `26a476cee1` (row 30; stand-in for upstream #4600) |
| `32d0a63fff` | chore: release 1.3.1 with the restarted-remote handshake fix | new |
| `34136dc353` | build: resolve noq from the Aster 1.3.2 fork | `-x` of `a9088cf6f8` (row 3) |
| `ff98745bb6` | fix(socket): derive the stateless reset key from the endpoint's secret key | `-x` of `a55b87cc72` (row 32) |
| `a4a9a76b01` | chore: release 1.3.2 with the deterministic stateless reset key | new |

The ledger and manifest docs commits sit on `upgrade/iroh-v1.3.0` after the tags, so the tags stay code-only. The ledger and `docs/aster-fork-manifest.toml` are not on the upstream base; they were carried over from `upgrade/iroh-v1.0.1` (tip `4b7e53fb8b`, which keeps the full 1.0.1-wave docs history), with the manifest restarted from the Aster repository's more current `iroh-fork-manifest.toml`.

## Patches

Row numbers are stable patch ids shared across Aster's iroh-family forks (iroh, iroh-blobs, iroh-docs, iroh-gossip, iroh-util, noq), so a commit message that says "row 30" resolves here.

| Id | Topic branch | What | Why / what in Aster depends on it | Commits (topic / upgrade) | Status | Conflict or porting notes |
|---|---|---|---|---|---|---|
| 3 | none recorded (original commit `7793d702a5`) | `[patch.crates-io]` entries in the root `Cargo.toml` for `noq`, `noq-udp` and `noq-proto`, pointing at the Aster noq fork (originally `https://github.com/aster-rpc/noq`, now `https://forge.emrul.dev/Aster/noq.git`) at an explicit rev; since 1.3.2 the manifests also require noq/noq-proto/noq-udp 1.3.2. | iroh must build against the noq fork that contains `read_into` and `poll_driver` (noq fork rows 1 and 2) and, since 1.3.1/1.3.2, the `read_to_end` and stateless-reset fixes. | `7793d702a5` (original) / `0a70368262` (noq 1.3.0, Forgejo rev `67c13644…`), `1974e95825` (noq 1.3.1), `34136dc353` (noq 1.3.2, `-x` of `a9088cf6f8`) on `upgrade/iroh-v1.3.0` | carried | Low conflict risk. Keep until upstream noq contains equivalent APIs. Pin an explicit fork rev or tag, never a floating branch. On the 1.3.0 port upstream had appended `[workspace.metadata.cargo-semver-checks.lints]` where the old patch block lived; the new `[patch.crates-io]` follows it, both kept. Regenerate `Cargo.lock` with `cargo update -p noq -p noq-proto -p noq-udp` and check all three resolve from `git+https://forge.emrul.dev/Aster/noq.git?rev=…`. |
| 4 | none recorded (original commit `a6c4e1a2fc`) | Re-exports `noq::poll_driver` as `iroh::poll_driver`. File: `iroh/src/lib.rs`. | Lets Aster FFI consumers reach the poll driver through the iroh dependency instead of importing noq directly. | `a6c4e1a2fc` (original) / `b5afe1aa30` on `upgrade/iroh-v1.3.0` (`-x` of `4abc891ef4`) | carried | Low conflict risk, usually one export line near `pub use endpoint::{Endpoint, RelayMode};`. Requires the noq `poll_driver` patch. |
| 5 | `fix/windows-busy-loop` (based on `v1.0.0`) | `Transports::poll_recv` throttles the per-transport recv-error log: the first error, then 1 in 256 with a `count=` field, reset on any successful recv, via `should_log_recv_error` + `recv_error_log_tests`. File: `iroh/src/socket/transports.rs`. | Defence in depth against recv-error log floods. It was written for the Windows `WSAECONNRESET` busy-loop (root-caused and fixed upstream in noq 1.0.1) and keeps any future recv-error storm from flooding logs. | `022676dabb` / `f0d154aca9` (1.0.0 line), `4114c449c5` (`upgrade/iroh-v1.0.1`), `52e3ef017c` (`upgrade/iroh-v1.3.0`) | carried (partly absorbed upstream) | Low conflict risk. Check each upstream release for native throttling. iroh 1.0.3 (`9f951b216b`, "reduce warn logs") lowered the log from `warn!` to `debug!` but added no throttle, so a sticky error still logs on every poll at debug level. Since 1.3.0 the throttle logs at `debug!` with upstream's `transport = %$transport` field. The 1.3.0 port conflict and its resolution are in History. Test: `cargo test -p iroh --lib recv_error_log_tests`. |
| 30 | `fix/new-connection-selected-path` (based on `1974e95825`, not on the upstream tag) | While any connection to a remote stayed open, `RemoteStateActor` sent a new connection's handshake datagrams only to the path selected for the older connection (`handle_msg_send_datagram`), skipped address lookup while a path was selected (`trigger_address_lookup`), and cleared `selected_path` only when the last connection closed (`handle_connection_close`). A remote killed without CONNECTION_CLOSE that came back with the same key on another relay or port stayed unreachable, even with the new address in the `EndpointAddr`. Now handshake datagrams go to the selected path first and then to every other known path; `ResolveRemote` starts lookup unless one is running; and `select_path` (now also run on every connection close) drops a selected path that no connection still has open. File: `iroh/src/socket/remote_map/remote_state.rs`. | Aster restart/reconnect (Aster repository, `docs/_internal/investigations/2026-10-15-restart-reconnect.md`, root cause 1): a peer restarted on a new port or relay is dialable at once instead of after 30–95 s or never. Loopback probe: new-port Docs recovery 0.4–0.8 s (was never, in 2 of 3 runs). | `26a476cee1` / `fc873f0b45`; released as 1.3.1 at `32d0a63fff` | carried | **Stand-in for upstream n0-computer/iroh#4600 (related #4050, #4531, PR #4435). Drop when a released iroh containing the upstream fix for #4600 is pinned; re-run the restart probe to confirm.** Cost: every `connect()` to an already-connected remote now starts an address lookup (pkarr/DNS) unless one is running. Low conflict risk outside `remote_state.rs`. flub's happy-eyeballs refinement (selected-path head start, cancel lookup on `AddConnection`) was deliberately left out. Tests: three loopback repros in `remote_state::tests` (new relay, new port, new port via lookup) fail before and pass after; ablations show each of the three changes is needed. |
| 32 | `fix/deterministic-reset-key` (based on `32d0a63fff`, not on the upstream tag) | The stateless reset key (`socket.rs`, was `Blake3HmacKey::new(rand)`) and noq's `HashedConnectionIdGenerator` key were random per process, so a restarted endpoint dropped its predecessor's CIDs as invalid and could not sign a token the peer accepts. `noq_endpoint_config` now derives both from the secret key with BLAKE3 `derive_key` (contexts "iroh stateless reset key v1" and "iroh connection id key v1"); neither is the signing key. Trade-off, documented in code: whoever holds the secret key can reset the endpoint's connections, but could already impersonate it. Also re-pins noq to 1.3.2 (row 3) and requires noq/noq-proto/noq-udp 1.3.2. Files: `Cargo.toml`, `Cargo.lock`, `iroh/Cargo.toml`, `iroh/src/socket.rs`, `iroh/src/tls/misc.rs`. | Root cause 2 of the same investigation: a peer restarted on the same port or relay now resets the old connections on their next packet instead of after 30 s. Loopback probe, same port: blob recovery 31.6 s → 1.6 s (median); gossip after a 15 s outage 32.1 s → 16.1 s (≈1 s after restart). A restart on a new port is not helped (nothing answers on the old port); root cause 3 (blob-pool eviction, iroh-util row 33 and iroh-blobs row 34) covers that case. Needs noq fork row 31. | `a9088cf6f8` + `a55b87cc72` / `34136dc353` + `ff98745bb6`; released as 1.3.2 at `a4a9a76b01` | carried | **Aster's own: no upstream equivalent. Keep unless upstream adopts a stable reset key.** Low conflict risk (endpoint config construction in `Socket::spawn`). Tests: `socket::stateless_reset_tests`: a key-derivation unit test plus same-port, same-relay and relay-then-direct kill-without-close restarts; the old connection closes with `ConnectionError::Reset` about 1 ms after its next packet (2 s bound). With a random reset key or random CID key all three time out. On noq 1.3.1 relay-then-direct fails (about 4.8 s, until a packet on the relay path). iroh lib 137 (1 ignored) single-threaded, integration 1, clippy (workspace all-targets, and `-p iroh --all-features --tests`) and fmt clean. |

Bookkeeping and documentation commits (not needed at runtime):

| Commits | Files | Purpose | Status |
|---|---|---|---|
| `a4efd34a28` (original) / `5e92f80704` on `upgrade/iroh-v1.3.0` (`-x` of `84f72a7835`) | `.gitignore` | Ignores `docs/aster/` internal review docs. | carried. Optional; keep while internal audit/roadmap docs stay local. |
| `32921714bb` / `6db27d8eeb` | various | `cargo fmt` after the patch port. | carried |
| `77a68a5477` | `README.md` | Aster fork overview and contribution table. | Not on `upgrade/iroh-v1.3.0`. Replaced by the `main` README preamble under the current model; if reused, refresh contribution rows rather than preserving stale "Pending" statuses. |

### Upstreamed or dropped

No iroh patch has been upstreamed or dropped. Related upstream changes:

- iroh 1.0.3 (`9f951b216b`) partly overlaps row 5 (log level only, no throttle); row 5 is still carried.
- The 1.0.1-wave noq pin commits `a9c547549e`, `ae7e3ae989`, `c29bcd6ce2`, `5228d64d91` and `b15ae80713` were replaced by `0a70368262` in the 1.3.0 port (row 3 re-expressed, not dropped).

### Branches not yet triaged

These topic branches exist in this repository (April 2026, from an earlier security review) but are not on the current upgrade branch and have no row above. Whether each is still needed, was fixed upstream, or should be dropped has not been decided yet.

| Branch |
|---|
| `fix/0rtt-expect` |
| `fix/accept-rate-limit` |
| `fix/integer-arithmetic` |
| `fix/max-incoming` |
| `fix/no-resume-unwind` |
| `fix/relay-addr-trust-doc` |

## Porting to a new upstream release

Order across the iroh family:

1. Port the noq fork first (its patches feed row 3 and row 4 here).
2. Port the iroh patches that depend on the noq fork (this repository).
3. Port iroh-blobs, iroh-docs, iroh-gossip and iroh-util, including their dependency patch blocks. Those blocks name iroh by revision, so re-pin in dependency order noq → iroh → iroh-blobs → iroh-docs / iroh-gossip.
4. Reapply optional docs/`.gitignore` patches where useful.
5. Update the root `[patch.crates-io]` revs in the Aster repository to the new fork commits.
6. Regenerate the fork manifest and this ledger's Current release section (below).

Before touching branches:

1. Read the upstream release notes and changelog for the target tag.
2. Inspect the range from the previous base, e.g. `git log --oneline v1.0.0-rc.0..v1.0.0-rc.1` and `git diff --stat` over the same range.
3. For each patch decide: keep, drop because upstream absorbed it, or rewrite because the APIs it touches changed. In particular check upstream #4600 (row 30), any stable reset key (row 32) and native recv-error throttling (row 5).

Port:

```bash
git fetch --all --tags --prune
git push origin <new-upstream-tag>      # mirror the upstream tag to the forge
git switch -c upgrade/iroh-<new-version> <new-upstream-tag>
git cherry-pick -x b5afe1aa30 5e92f80704 52e3ef017c 6db27d8eeb
# row 3: re-point [patch.crates-io] at the new noq fork commit, then
cargo update -p noq -p noq-proto -p noq-udp
git cherry-pick -x fc873f0b45            # row 30, unless upstream fixed #4600
git cherry-pick -x ff98745bb6            # row 32
git tag -a aster-iroh-v<published version> -m "Aster iroh v<published version>"
git push origin upgrade/iroh-<new-version> aster-iroh-v<published version>
```

Prefer `cherry-pick -x`. If a patch no longer applies, treat the "What" and "Conflict or porting notes" columns as the specification. Do not rewrite a published `upgrade/*` branch or tag unless you are deliberately replacing a failed upgrade attempt. After the 1.3.0 port the downstream lockfiles did not inherit upstream's rustls fix automatically (see History); check `cargo update -p rustls` in the Aster repository after each port.

Validate in this repository:

```bash
cargo fmt --all --check
cargo clippy --workspace --all-targets -- -D warnings
cargo clippy -p iroh --all-features --tests -- -D warnings
cargo test -p iroh --all-features --lib -- --test-threads=1
cargo test -p iroh --lib recv_error_log_tests
cargo test -p iroh --test integration
```

Use `--test-threads=1`: in the rc.1 port the default threaded `cargo test -p iroh --all-features` hit a SIGSEGV in the lib test binary while the named tests passed individually and the full run passed single-threaded; treat it as a threaded/local-interface upstream test issue before relying on the default command in CI. Patchbay tests are Linux-only.

Whenever iroh moves, re-pin the iroh-docs fork's own `[patch.crates-io]` to the same revisions and run its full suite (`cargo test --all-features`, 118 tests including `tests/sync.rs`): iroh-docs exercises the whole stack together. Gotcha when re-pinning a satellite: `cargo update -p iroh` alone can keep crates.io iroh (it kept 1.0.2) and report the patches as unused; `cargo update -p iroh --precise 1.3.2`, or temporarily requiring `iroh = "1.3.0"` and running `cargo update -p <crate>`, moves the transitive dependencies (restoring the original spec keeps the lock).

### Updating the fork manifest after each cycle

`docs/aster-fork-manifest.toml` is the machine-readable record of what to pin for the whole stack (tags, commits, upstream bases, bootstrap Cargo snippets). It is canonical in this repository; a synced copy lives in the Aster repository as `iroh-fork-manifest.toml`. Refresh both every cycle, after pins are flipped and validated, so nobody pins a stale rev. After step 5 above:

1. Edit `docs/aster-fork-manifest.toml`: bump `wave`, `released`, every `[forks.*]` block (`upstream_base`, `tag`, `commit`, `branch`, `crate_version`), the `[transitive]` pins, and the `[snippets]` `version_specs` / `patch_crates_io` (and `salvo_patch_crates_io`) blocks. Each `commit` must equal the rev the Aster repository pins (pin == tag == commit).
2. Update this ledger's Current release section and the other forks' ledgers.
3. Check it parses: `python3 -c "import tomllib,sys; tomllib.load(open('docs/aster-fork-manifest.toml','rb'))"` (Python ≥ 3.11; otherwise `uv run python -c ...`).
4. Commit on the `upgrade/<iroh-version>` branch as a docs-only commit after the `aster-*` tag, so the tag stays code-only, and push.
5. Copy the file to the Aster repository as `iroh-fork-manifest.toml` and commit it there. Consistency check, from a checkout of this repository on the upgrade branch: `diff <(git -C <aster-repository> show HEAD:iroh-fork-manifest.toml) docs/aster-fork-manifest.toml && echo "manifests in sync"`.

Aster validation after updating pins, from the Aster repository root:

```bash
cargo fmt --check
cargo clippy --manifest-path bindings/python/rust/Cargo.toml -- -D warnings
cargo test -p aster_transport_core
cargo test -p aster --all-features
uv run maturin develop -m bindings/python/rust/Cargo.toml
uv run pytest tests/python/ -v --timeout=30
```

Local checkouts: downstream workspaces that use `[patch.crates-io]` path overrides into a local checkout (for example portal-sync) build whatever is checked out, not the pinned rev. After publishing, switch the local checkout to the released tag, or to the upgrade branch if you need the ledger (the docs commit is past the tag); check `git status --short --branch` and `git tag --points-at HEAD`; and run at least one path-override consumer check (for example `cargo check --locked -p portal-cas` in portal-sync).

Before the current branch model, fork `main` mirrored upstream `main` and was reset to it on each port (`git reset --hard upstream/main`, `git push origin main --force-with-lease`, the only place a forced push was allowed). Under the current model `main` is the front door and is not reset.

## History

### 1.3.0 wave (2026-10-03 onward), current

- Upstream released iroh `v1.3.0`, which bumps to noq 1.3.0, net-tools 0.19.3, a stable `ed25519-dalek` 3.0.0 within `>=3.0.0-rc.0,<4.0.0`, and replaces hickory with `n0-dns-resolver` 0.1. The fork was re-ported onto `v1.3.0` (`0072d7d84b`) as `upgrade/iroh-v1.3.0`, validated, tagged `aster-iroh-v1.3.0` and pushed to Forgejo. Fork `main` was fast-forwarded to upstream `main` (`b4c8fc68ef`, no force) and the upstream `v1.3.0` tag was mirrored to Forgejo.
- **Row 5 conflict** in `iroh/src/socket/transports.rs`: upstream renamed the macro argument to `$transport` (`debug!(transport = %$transport, …)`). The throttle was re-expressed at upstream's `debug!` level with that field; upstream's new `#[cfg(test)] mod tests` was kept beside `recv_error_log_tests`; and upstream's `custom_only_transports` test helper (a literal `Transports {..}`) gained `recv_error_count: 0` in the same commit. The commit message records the resolution and the full `-x` provenance chain (`022676dabb` → `4114c449c5`).
- **Cargo.toml:** upstream appended `[workspace.metadata.cargo-semver-checks.lints]` where the old patch block lived; both kept. `Cargo.lock` regenerated with `cargo update -p noq -p noq-proto -p noq-udp`, resolving all three from `git+https://forge.emrul.dev/Aster/noq.git?rev=67c1364…`.
- **Upstream security fix now included:** `2b9f441801` (GHSA merge, first in iroh 1.1.0): the relay path drops oversized datagram segments and keeps processing batches after an empty one. It arrived with the rebase; nothing to carry.
- **RUSTSEC-2026-0285 (rustls):** upstream `78f66a22cb` bumped rustls in iroh's `Cargo.lock` (now 0.23.45; 1.0.1 locked 0.23.41). The manifest requirement stays `rustls = "0.23.33"`, so downstream lockfiles do not inherit the fix: the Aster repository (and any other root) must `cargo update -p rustls` to a patched release.
- `n0-dns-resolver` 0.1.0 is new and needs no registry bridge; `netwatch`/`portmapper` moved to 0.19.3 (net-tools `0030e57fc70981895bb86212ff0f059cc41c8a4e`), and the netwatch bridge was republished at 0.19.3 because it depends on `noq-udp`.
- Validation (macOS, rust 1.96; patchbay tests not run): `cargo fmt --all --check`, `cargo clippy --workspace --all-targets -- -D warnings`, `cargo clippy -p iroh --all-features --tests -- -D warnings` clean; `cargo test -p iroh --all-features --lib -- --test-threads=1` 130 passed / 1 ignored; `recv_error_log_tests` 1 passed; `--test integration` 1 passed.
- **Same day: `aster-iroh-v1.3.0-p1`** (`1974e958`) only re-pins noq to 1.3.1 (the noq `read_to_end` fix for upstream #794), so iroh stays 1.3.0; its `noq = "1.3.0"` requirement admits 1.3.1. Validation: clippy and lib tests 124 single-threaded. The satellites' own patch blocks moved to iroh `1974e958`.
- **1.3.1** (`32d0a63fff`, tag `aster-iroh-v1.3.1`): row 30.
- **1.3.2** (`a4a9a76b01`, tag `aster-iroh-v1.3.2`): row 32 plus the noq 1.3.2 re-pin. iroh-docs 0.101.5-p2 (2026-10-16), iroh-gossip 0.101.1-p1 and iroh-util 0.6.3-p1 (2026-10-18) re-pinned their own patch blocks to iroh `a4a9a76b`.

### 1.0.1 wave (2026-07-02)

| Upstream base | Tag | Pinned commit | Branch |
|---|---|---|---|
| `v1.0.1` | `aster-iroh-v1.0.1-p1` | `b15ae8071384f2643b46600ccfebbc2f4d36b6e9` | `upgrade/iroh-v1.0.1` |

- Upstream `v1.0.1` bumped to noq 1.0.1 and net-tools 0.19.1. noq 1.0.1 absorbed the Windows UDP root-cause fix; the recv-error log throttle (row 5) was not absorbed and stayed.
- Carried: the noq fork `[patch.crates-io]` entries (pinned to noq `c84091f4c`), the `poll_driver` re-export, the gitignore, the recv-error throttle (cherry-picked clean from `fix/windows-busy-loop` `022676dabb`), and lockfile and fmt fixups.
- On 2026-08-13 the iroh-family forks' own `[patch.crates-io]` blocks, which had named the GitHub mirror at the 1.0.0-wave iroh (`3c329f0c`) and noq (`ddb8829d`), were swept to Forgejo at this wave's revisions (`-p1` tags), in dependency order noq → iroh → iroh-blobs → iroh-docs / iroh-gossip. Aster's product graph was never affected (its root patch overrides them), but each fork's own build and tests had compiled a stack Aster does not ship. No `src` change, no crate version change, nothing republished.
- The Aster repository bumped `iroh = "=1.0.1"` alongside the rev pins; `cargo update` pulled transitive `netwatch`/`portmapper` 0.19.1 and `netdev` 0.45.
- Validation: fmt, clippy (`-D warnings`, `--all-features --tests`), iroh 142 tests single-threaded; downstream `aster_transport_core` (242) and `aster --all-features` (94), PyO3 clippy clean, `maturin develop` and 1117 Python tests.
- The branch tip `4b7e53fb8b` holds the full 1.0.1-wave docs history. Local checkout at the end of the cycle: `upgrade/iroh-v1.0.1` (the ledger commit is one past the tag).

### Windows UDP busy-loop fix (2026-07-01)

- The Aster/iroh endpoint busy-looped on Windows (~120% CPU plus a log flood of `recv error … os error 10054`) when a peer or candidate became unreachable mid-session. The root cause was in noq (fixed there, later absorbed upstream in noq 1.0.1). This repository carried the defence-in-depth log throttle (row 5) on `fix/windows-busy-loop`, based on the upstream tag `v1.0.0`: `022676dabb`, cherry-picked `-x` onto the live upgrade branch as `f0d154aca9`. A root-cause write-up and deterministic repro were kept by Aster outside the fork.
- Validation on macOS: `cargo test -p iroh --lib recv_error_log_tests`, clippy and fmt clean.

### 1.0 wave (2026-06-21)

| Upstream base | Tag | Pinned commit | Branch |
|---|---|---|---|
| `v1.0.0` | `aster-iroh-v1.0.0` | `3c329f0c350b98a772bccdf697bfcc519fcdf25a` | `upgrade/iroh-v1.0.0` |

- Coordinated upstream 1.0 wave. The patch stack was carried forward unchanged from rc.1 and cherry-picked clean. Forced transitive bumps from upstream's `feat!: update to 1.0 dependencies`: `ed25519-dalek` `=3.0.0-pre.7` → `=3.0.0-rc.0`, `iroh-mdns-address-lookup` 0.3 → 0.4. No Aster core API changes were needed (already on rc.1).
- Every tag pointed at exactly the commit Aster pinned (no docs-only `-pN` retags this cycle); the ledger commit sat on the branch after the tag, so the local checkout had to stay on `upgrade/iroh-v1.0.0` to see the ledger.
- Validation: per-fork fmt/clippy/tests, then `aster_transport_core` (229) and `aster` (68) Rust tests and `maturin develop` + 1117 Python tests.

### rc.1 port (2026-06-07)

- Before the port Aster pinned `77a68a5477` (tag `aster-iroh-v1.0.0-rc.0`), upstream base `v1.0.0-rc.0` (`96b4b0e80d`), branch `upgrade/iroh-v1.0.0-rc.0`; stack range `v1.0.0-rc.0..aster-iroh-v1.0.0-rc.0`. Target `v1.0.0-rc.1` (`ee8b6a3d93`).
- Ported and published: fork `main` mirrored upstream `main` (`ee8b6a3d93`); branch `upgrade/iroh-v1.0.0-rc.1` and tags `aster-iroh-v1.0.0-rc.1` and `aster-iroh-v1.0.0-rc.1-p1` pushed to `aster-rpc/iroh`, based on `v1.0.0-rc.1`. The `-p1` tag was docs-only relative to the code tag and was the preferred local checkout for reading the ledger.

| Commit | Purpose |
|---:|---|
| `00e016455d` | Cherry-pick of `7793d702a5`: Aster noq `[patch.crates-io]` entries. |
| `e1be58459e` | Cherry-pick of `a6c4e1a2fc`: re-export `noq::poll_driver` as `iroh::poll_driver`. |
| `5aa2363baf` | Pin the noq patches to rev `e968659c47057b18210a4a03dc2366c65fe8f654` instead of `branch = "main"`, because fork `main` now mirrored upstream. |
| `8a751a69f7` | `cargo fmt` after the rc1 port. |
| `06bd1f5a90` | `Cargo.lock` resolves `noq`, `noq-proto` and `noq-udp` from the Aster rc1 noq fork rev. |
| `62a6fe93de` | Cherry-pick of `a4efd34a28`: ignore local `docs/aster/` notes. |
| `ae6c474ee5` | Recorded the satellite rc1 ports and the iroh-docs content-ready patch in the ledger. |

- Validation: `cargo fmt --check`; `cargo test -p iroh --all-features -- --test-threads=1` (the threaded run hit the SIGSEGV noted under Porting).
- The rc.1 command sequence was `git cherry-pick -x 7793d702a5 a6c4e1a2fc` (optionally `a4efd34a28 77a68a5477`) onto `upgrade/iroh-v1.0.0-rc.1`, then tag `aster-iroh-v1.0.0-rc.1`. The Aster root patch block then pointed `iroh`, `iroh-base` and `iroh-relay` at `https://github.com/aster-rpc/iroh`; it now points at the Forgejo repository.
- Local checkout at the end of the cycle: `aster-iroh-v1.0.0-rc.1-p1`.
