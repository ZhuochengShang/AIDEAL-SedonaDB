# SedonaDB — AIDEAL five-condition preparation

**Setup only: five baseline source branches; treatments and evaluation pending. No model is selected; this new setup made zero model calls.**
The date in the study/ref names is the setup start date, 2026-09-21; preparation continued after midnight.

[Shared AIDEAL study guide](https://github.com/ZhuochengShang/AIDEAL/blob/setup/2026-09-21-library-five-conditions/studies/sedonadb/2026-09-21-five-condition-setup/README.md) · [Source/setup identities](study.json) · [Shared workflow](https://github.com/ZhuochengShang/AIDEAL/blob/setup/2026-09-21-library-five-conditions/docs/SUMMARY.md)

The source repository and shared AIDEAL serve different purposes: these branches hold library source; shared AIDEAL holds prompts, controller/harness code, settings conventions and future reports. The companion repository requires private access.
Publication policy keeps GitHub Actions disabled while preparation is pending; upstream CI has not validated this setup.

## Source identity and attribution

Upstream: [https://github.com/apache/sedona-db.git](https://github.com/apache/sedona-db.git); selected commit [59ec34c79594222891b1dbaf106bfa9de14a5b99](https://github.com/apache/sedona-db/commit/59ec34c79594222891b1dbaf106bfa9de14a5b99).
Preparation snapshot: [255a2cdc9528f3908ae097f2b498c27d48d4b7d7](https://github.com/ZhuochengShang/AIDEAL-SedonaDB/commit/255a2cdc9528f3908ae097f2b498c27d48d4b7d7); tree `1df3708ef1d66d3bcbeabbc7bf292a4740290cfd` is exactly the upstream tree. This new commit does not ship upstream ancestry.

Original documentation selected for future attachment: [README.md](https://github.com/ZhuochengShang/AIDEAL-SedonaDB/blob/255a2cdc9528f3908ae097f2b498c27d48d4b7d7/README.md), [examples/sedonadb-rust/README.md](https://github.com/ZhuochengShang/AIDEAL-SedonaDB/blob/255a2cdc9528f3908ae097f2b498c27d48d4b7d7/examples/sedonadb-rust/README.md). Source-branch root READMEs remain upstream bytes. This navigation README on `main` is not a documentation treatment. A future installed generated README is selected from `.aideal/treatments/README.md` in the relevant condition.

Upstream license/notice files are copied byte-for-byte: [LICENSE](LICENSE), [NOTICE](NOTICE). They retain their original attribution and terms; this setup does not relicense upstream source.

## Five conditions

| Condition | Intended difference from original | Actual state |
| --- | --- | --- |
| [Original](https://github.com/ZhuochengShang/AIDEAL-SedonaDB/tree/preparation/2026-09-21-five-conditions/original) | Original documentation and APIs | Baseline; not evaluated |
| [README only](https://github.com/ZhuochengShang/AIDEAL-SedonaDB/tree/preparation/2026-09-21-five-conditions/readme-only) | Select a reviewed generated README | Same baseline; treatment pending; not evaluated |
| [Alias only](https://github.com/ZhuochengShang/AIDEAL-SedonaDB/tree/preparation/2026-09-21-five-conditions/alias-only) | Add tested delegating aliases and their interface | Same baseline; treatment pending; not evaluated |
| [Error hints only](https://github.com/ZhuochengShang/AIDEAL-SedonaDB/tree/preparation/2026-09-21-five-conditions/error-hints-only) | Deliver matched development-error hints on repair | Same baseline; treatment pending; not evaluated |
| [Combined](https://github.com/ZhuochengShang/AIDEAL-SedonaDB/tree/preparation/2026-09-21-five-conditions/combined) | Combine the same README, aliases, and error hints | Same baseline; treatment pending; not evaluated |

All rows have the same commit/tree today. No historical generated README, aliases, hints or evaluation artifacts have been installed here. Branch names express the study plan, not measured differences.

## Get the source

For one clean source checkout (private repository access required):

```sh
git clone -b preparation/2026-09-21-five-conditions/original https://github.com/ZhuochengShang/AIDEAL-SedonaDB.git AIDEAL-SedonaDB
```

Or clone the navigation branch once and create five **distinct named branches/worktrees**:

```sh
git clone https://github.com/ZhuochengShang/AIDEAL-SedonaDB.git AIDEAL-SedonaDB
cd AIDEAL-SedonaDB
git worktree add -b review/original ../sedonadb-original origin/preparation/2026-09-21-five-conditions/original
git worktree add -b review/readme-only ../sedonadb-readme-only origin/preparation/2026-09-21-five-conditions/readme-only
git worktree add -b review/alias-only ../sedonadb-alias-only origin/preparation/2026-09-21-five-conditions/alias-only
git worktree add -b review/error-hints-only ../sedonadb-error-hints-only origin/preparation/2026-09-21-five-conditions/error-hints-only
git worktree add -b review/combined ../sedonadb-combined origin/preparation/2026-09-21-five-conditions/combined
```

All five currently resolve to `255a2cdc9528f3908ae097f2b498c27d48d4b7d7`. Sharing a commit is intentional; branch names are different.
`main` contains this navigation guide. Use a preparation branch/worktree for the actual source, not `main`.

## Submodule prerequisites

The source snapshot preserves these Git pins; their repositories were **not initialized or built** during setup.

| Path | Pinned commit | Upstream |
| --- | --- | --- |
| `c/sedona-s2geography/s2geography` | `3ab3dd9ab95dd473647f7ed9fcaccbb784d58dc1` | [https://github.com/paleolimbot/s2geography.git](https://github.com/paleolimbot/s2geography.git) |
| `c/sedona-s2geography/s2geometry` | `a37aba69f14af676e9605cd9515c8aca6cef8342` | [https://github.com/google/s2geometry.git](https://github.com/google/s2geometry.git) |
| `submodules/geoarrow-data` | `e8eaa04487d982398f07713cde527c5c5edeee1e` | [https://github.com/geoarrow/geoarrow-data.git](https://github.com/geoarrow/geoarrow-data.git) |
| `submodules/sedona-testing` | `4a05ef1c3c9dbeaab8568211112b32a2a50fcda4` | [https://github.com/apache/sedona-testing.git](https://github.com/apache/sedona-testing.git) |

All URLs are absolute HTTPS URLs. No relative-origin override is needed. If required for your chosen build, initialize inside a source worktree:

```sh
git submodule update --init --recursive
```

This optional command downloads external source/test data. It was not run for preparation and is not a validated complete build recipe.

## Before evaluation

Rust crate and native-wrapper Rust source scope; other language bindings and submodule internals excluded from initial discovery. Historical 320/322 SQL examples executed without an independent semantic oracle; not a five-condition LLM result. Four submodules uninitialized; no Rust build validation in this setup.

Use the [shared attachment instructions](https://github.com/ZhuochengShang/AIDEAL/blob/setup/2026-09-21-library-five-conditions/studies/sedonadb/2026-09-21-five-condition-setup/README.md#prepare-your-own-local-study). Complete and validate the library runtime, independent task bank and controls, treatment artifacts, per-condition builds, model/prompt/budget settings and a new local freeze. Documentation and draft configuration alone are not a runnable evaluator. No setup test or example execution is a measured semantic score.
