# tidepool-reconciler roadmap (revision 3)

Spec: master specification revision 3, §2.5–2.9 (storage is state; cells are the record; the cycle cell index; per-site meet), §7 (S3\* audit), Appendix C, Appendix D. Conventions: ESM, `node --test` with injected RPC fakes, records as JSON under `data/records/` with schemas, every EVM read finality-qualified by `multi-chain-finality-services-section`'s `assayReceiptFinalityPolicy`, every Hive read `≤ last_irreversible_block_num` from two nodes (`tidepool-hive`). **Refusals are reverts** (no `Refused` event exists): the reconciler reproduces refusals by simulating `draw` with `eth_call`/`eth_estimateGas` for the disclosure's `refusals[]` and decodes the custom error data. **Note rule:** every task's acceptance includes a note at `tidepool-engineering-services-section/content/text/documents/articles/notes/YYYY/MM/DD/YYYY-MM-DD-note-on-<task-id>-<slug>.md`; a task without its note is unfinished.

## Phase 1 (v0.1)

### TP-REC-001 `v0.1` EVM site reader
- Goal: `src/evm-site.mjs`: from a manifest (ABIs and addresses from `tidepool-evm`'s `TP-EVM-014` artifacts and end-to-end fixture; the manifest's `charterHash` is checked against `charter.charterHash()` and a mismatch refuses the site) and a finality policy, read `CellLedger` `CellOpened`/`Told`/`Held`, treasury `YieldReceived`/`CycleClosed`/`CycleRolled`/`DistributablePulled`/`Accrued`/`Claimed`, desk `Drawn`/`SettledLocal`/`MarkedOverdue`/`VoidedUnreleased`/`ForeignReleaseWitnessed`/`ForeignSettleWitnessed`/`FxResidueAccrued`, allocator `Opened`/`Participated`/`Quiesced`/`Executed`, site witness `CellIndexed`/`SectionCommitted`/`HeadTold`/`TotalChallenged`/`MeetSettled`, mutual `Claimed`/`Granted`/`Refused`, clearinghouse `EdgeOpened`/`EdgeAdmitted`/`CycleCleared`, plus the storage views the disclosure needs (`deriveLine`, `barrierAt`, `meetOf`, `claimable`); return `{site, cells, events[], storage, height}` with every event carrying its `chainReceiptKey`.
- Files: `src/evm-site.mjs`, `src/abi/*.json`, `test/evm-site.test.mjs`.
- Acceptance: the TP-EVM-014 end-to-end fixture's logs fold to the pinned census and storage snapshot; unfinalized receipts are listed under `unfinalized`; a manifest with a wrong `charterHash` is refused.
- Depends: TP-ENG-001, TP-EVM-014, multi-chain-finality.

### TP-REC-002 `v0.1` Hive site reader
- Goal: `src/hive-site.mjs` wrapping `tidepool-hive`'s `readHiveSection` with the roster from `buildRoster` (registry `HiveLinked` events + Hive link ops), returning the same shape as TP-REC-001 with `height = LIB`, `ignored[]` listed.
- Acceptance: TP-HIVE-003's fixtures yield the same digest and the same `ignored` set here.
- Depends: TP-HIVE-003, TP-HIVE-004, TP-REC-001.

### TP-REC-003 `v0.1` Cross-site two-witness reconcile
- Goal: `src/reconcile.mjs`: `reconcileSites(sections[])` merges cell values by knowledge join across sites and lists `held` (Both) with the disagreeing witnesses; `reconcileMember` is `pool-service`'s `reconcilePoolBooks` unchanged with pool postings derived from desk events; `defaultsHeld` lists draws whose storage status is `Overdue` (no cell is Both by design) and `provisional` lists foreign-site halves; `meetOf(site, cycle)` reproduces `SiteWitnessV1.meetOf` off-chain; the output record `data/records/reconciliations/<poolId>/<cycle>.json` with buckets `agreed`, `mismatch`, `postedOnly`, `assertedOnly`, `held`, `defaultsHeld`, `provisional`, `unfinalized`, `ignored`.
- Acceptance: a fixture with a challenged total produces one `held` entry with `heldMass`; `defaultsHeld` matches the desk's `Overdue` statuses; the `pool-service` reconcile cases pass unchanged; "unknown never zero" holds (one-sided entries bucketed, never dropped).
- Depends: TP-REC-001, TP-REC-002.

### TP-REC-004 `v0.1` Disclosure and opsroom data
- Goal: `src/disclosure.mjs`: per pool per cycle, JSON conforming to `pool-service`'s `pool-cycle-disclosure.schema.json` extended with `sites[]` (per-site `meet`, `heldMass`, `challengesOpen`), `heldCount`, `unheard`, `defaultsHeld`, `claims` (open, granted, refused), `clearings`, `barrier` (day buckets), `writeoff` and `budget`, `personDrawsPaused`, `refusals[]` (reproduced by simulation, by name, no member detail), `keeperBounties`; aggregates only.
- Acceptance: schema-valid output from the TP-REC-003 record and the TP-REC-001 storage; no member or sleeve address appears in the output; the opsroom page (TP-PUB-002) renders from this file alone.
- Depends: TP-REC-003.

### TP-REC-005 `v0.1` Sorted cell lists and head payloads
- Goal: `src/heads.mjs`: for each site and cycle, the strictly ascending list of cell ids the site witness indexed (`CellIndexed` events; verified against `cycleCells`) for `commitSection`, the expected digest, and the `tellHead` payloads (Head with `blockId`, `total`, `hiveFloatE3`, and the §2.9 envelope) for the keeper relay; for Hive the list and digest come from TP-HIVE-003.
- Acceptance: the expected digest equals the Foundry-pinned `commitSection` result for the end-to-end fixture; a list with a missing cell is detected before submission.
- Depends: TP-REC-001, TP-REC-002.

## Phase 2 (v0.2)

- TP-REC-006 Off-chain H¹ (`signgraph/cellular-sheaf`) and the `tellH1` payload; TP-REC-007 EAS attestations (Base, Arbitrum, OP only); TP-REC-008 `pool-service` capability adapters and the Beancount sidecar (`tp-cycle-transactions-beancount`, `tp-pool-ledger-postings`, `tp-house-ledger-accounts`); TP-REC-009 worklist solver attestation; TP-REC-010 k-of-n relayer coordination (the reconciler as one of n).

## Phase 3 (v0.3)

- TP-REC-011 Decorrelation table.

## Capability-ledger citations

`tp-two-witness-reconcile`, `tp-ledger-witness` (TP-REC-003 imports them unchanged); `tp-reconcile-base-receipts` (TP-REC-001 generalizes; the missing `ethers` dev dependency is added in TP-ENG-005); `tp-cycle-disclosure`, `tp-site-public-projection` (TP-REC-004; the restored cycle-1 disclosure is the first fixture); `tp-cycle-transactions-beancount`, `tp-pool-ledger-postings`, `tp-house-ledger-accounts` (TP-REC-008, TP-OPS-010); `tp-treasury-morpho-moonwell` (TP-REC-001 reads vault positions through `treasury-observation-service`); `tp-receipt-bound-split` (the `yieldEvidence` NI from `YieldReceived`); `tp-store-receipts` (retire-with-reason: the chain is the store).
