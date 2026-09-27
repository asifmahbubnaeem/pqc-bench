# PQC-Bench

Reproducible post-quantum TLS performance benchmarks on AWS.

**What it measures:** the real-world cost of enabling `X25519MLKEM768` (NIST FIPS 203 hybrid PQ key exchange) vs classical `X25519` in TLS 1.3, under realistic load.

**Design goals:**
- **Reproducible.** Every published number ships with the Terraform, the Gatling simulation, and the raw JSON results.
- **Cheap.** A full benchmark run costs under $1 on AWS spot instances. Sub-$5/month if you re-run monthly.
- **Independent.** Not a vendor benchmark, not a library microbenchmark.

## Quick start

Prerequisites: `terraform >= 1.6`, `aws-cli >= 2.15`, `make`, an AWS account with an IAM user configured as profile `pqc-bench`.

```bash
# One-time setup
make init

# Spin up target + load generator (same-AZ, both c7g.large spot)
make bench-up

# Run scenario 01 (fresh-handshake latency, classical vs PQ)
make scenario-01

# Tear everything down (do this every time!)
make bench-down
```

Estimated cost per full scenario run: **~$0.30**. Estimated cost per `bench-up` for smoke testing: **~$0.05** (spin up, verify, tear down within 15 minutes).

## Project structure

```
pqc-bench/
├── terraform/
│   ├── target/          # nginx-pq target module
│   ├── loadgen/         # Gatling + JMeter loadgen module
│   └── bench/           # Root module composing both
├── scenarios/
│   └── 01-fresh-handshake-latency/
│       ├── gatling/     # Gatling simulation (Scala)
│       └── run.sh       # Driver: apply configs, run, collect
├── analysis/            # Jupyter notebooks for per-scenario analysis
├── results/             # Raw Gatling JSON + notebook outputs (committed!)
├── docs/                # Blog post drafts, methodology, published writeups
├── scripts/             # Helper scripts (SSH wrappers, log fetchers)
└── Makefile             # All commands live here
```

## Cost discipline

The number-one killer of this project's budget is forgotten instances. Rules:

1. **Always** `make bench-down` after any run. Set a daily calendar reminder for the first 30 days.
2. Set an AWS Budget alert at $10/month in the AWS console.
3. Never keep an Elastic IP allocated (this project doesn't need one).
4. Spot instances only — the Terraform enforces this.

## Status

- [x] Phase 0: Foundation (Terraform, user-data, Makefile)
- [ ] Phase 1: First scenario measured, first blog post
- [ ] Phase 2: Scenario matrix (resumption, payload, concurrency, CPU)
- [ ] Phase 3: Cross-architecture (c7g vs c7i vs c6a)
- [ ] Phase 4: Nightly regression detection
- [ ] Phase 5: Path B algorithms via oqsprovider

## License

Apache 2.0. See `LICENSE`.
