# @red-cup-engineering/tidepool-reconciler

`pool-service`'s successor as the cross-site two-witness reader. It reads every site (EVM `CellLedger`/treasury/desk events under the finality policies of `multi-chain-finality-services-section`; Hive `custom_json` cells via `tidepool-hive`), folds them by knowledge join, reconciles pool-side records against members' declared ledgers into `agreed` / `mismatch` / `postedOnly` / `assertedOnly` / `held`, computes the off-chain H¹ with `signgraph/cellular-sheaf`, projects the public disclosure, and issues attestations. It writes nothing on-chain itself; keepers carry its payloads.

Until TP-ENG-005 moves `pool-service` under `lib/`, this package imports `@red-cup-engineering/pool-service` for `reconcilePoolBooks` and the disclosure schema.
