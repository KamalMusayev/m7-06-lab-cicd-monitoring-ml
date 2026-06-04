# Rollback Runbook — Vision Moderation

## When to roll back

Rollback immediately if any of these happen after deploy or during canary:

- 5xx error rate is above **1% for 10 minutes**
- p95 latency is above **500 ms for 10 minutes**
- `AvailabilityBurnFast` or `ModelVersionMismatch` alert is firing
- Canary verification fails
- Model quality proxy drops more than **5%** from the previous stable version
- More than one production replica is unhealthy or restarting repeatedly

## How to roll back

Stop rollout if canary is still running:

```bash
./scripts/rollback.sh production
```

If rollback is triggered from GitHub Actions, use:

```text
GitHub Actions → Deploy Model → Run workflow → environment=production → rollback=true
```

Confirm the previous stable image/model version is active in production, then run:

```bash
./scripts/smoke.sh production
```

## What to verify

Before closing the incident, confirm:

- `AvailabilityBurnFast` is green
- `LatencyP99High` is green
- `ModelVersionMismatch` is green
- 5xx error rate is below **0.5%**
- p95 latency is back below **250 ms**
- p99 latency is back below **500 ms**
- All replicas serve the same expected `model_version`
- Production smoke test passes

## Who to notify

Notify:

- PagerDuty incident owner
- `#ml-platform` channel
- `#sre-oncall` channel
- Partner support lead if customer traffic was affected
- Create or update the Linear incident ticket

## What not to do

- Do not roll forward before the root cause is understood
- Do not continue full rollout if canary verification failed
- Do not silence SLO alerts without incident lead approval
- Do not manually patch only one replica
- Do not delete logs, metrics, or failed deployment artifacts

## When to roll forward

Roll forward only when:

- Root cause is identified
- Fix is merged and reviewed
- Unit, contract, and smoke tests pass
- Staging deploy is healthy
- Canary stays healthy for **30 minutes**
- Error rate, p95/p99 latency, and model version alerts remain green