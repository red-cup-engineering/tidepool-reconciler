# @red-cup-engineering/tidepool-reconciler

<!-- 561-group-standards:badges:start -->
[![Red Cup Engineering](https://img.shields.io/badge/561%20Group-Red%20Cup%20Engineering-c0392b)](https://github.com/red-cup-engineering)
[![License: MIT](https://img.shields.io/badge/license-MIT-c0392b)](LICENSE)
[![Indigenous data: CARE](https://img.shields.io/badge/Indigenous%20data-CARE-4b6f44)](https://github.com/red-cup-engineering/.github/blob/main/GOVERNANCE.md)
[![Tips welcome](https://img.shields.io/badge/tips-welcome-d4a017)](https://github.com/561-group/.github/blob/main/TIPS.md)
<!-- 561-group-standards:badges:end -->

`pool-service`'s successor as the cross-site two-witness reader. It reads every site (EVM `CellLedger`/treasury/desk events under the finality policies of `multi-chain-finality-services-section`; Hive `custom_json` cells via `tidepool-hive`), folds them by knowledge join, reconciles pool-side records against members' declared ledgers into `agreed` / `mismatch` / `postedOnly` / `assertedOnly` / `held`, computes the off-chain H¹ with `signgraph/cellular-sheaf`, projects the public disclosure, and issues attestations. It writes nothing on-chain itself; keepers carry its payloads.

Until TP-ENG-005 moves `pool-service` under `lib/`, this package imports `@red-cup-engineering/pool-service` for `reconcilePoolBooks` and the disclosure schema.
