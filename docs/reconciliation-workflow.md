## Reconciliation workflow

The reconciliation job periodically compares **on-chain** observed balances/supply on Stellar with the **reported/source-chain** balance used by the bridge verifier (currently Ethereum reserves for bridged assets).

### What it does

- Runs on a schedule (hourly by default) for each configured asset (`USDC`, `EURC`)
- Uses a Redis lock to avoid overlapping runs per asset
- Persists every run (including mismatches and failures) to PostgreSQL/Timescale (`reconciliation_runs`)
- Emits structured logs for mismatch detection and job timing

### Where it runs

- Worker: `backend/src/workers/reconciliation.job.ts`
- Scheduler/registration: `backend/src/workers/index.ts`

### Data stored

Each run records:

- `asset_code`
- `status`: `success`, `mismatch`, or `failed`
- `stellar_supply` and `reported_supply`
- `mismatch_percentage`
- timestamps + error message (if any)

### API

- `GET /api/v1/reconciliation/runs?assetCode=USDC&limit=50`
- `GET /api/v1/reconciliation/latest/:assetCode`

