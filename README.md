# AWS Cloud Cost Estimator

A dual-interface cloud cost estimation tool — a polished **CLI** for terminal workflows and a **web UI** for interactive architecture design. Estimates monthly AWS costs across EC2, RDS, S3, Lambda, and CloudFront using the AWS Pricing API.

![Python 3.9+](https://img.shields.io/badge/python-3.9+-blue?style=flat-square)
![No dependencies](https://img.shields.io/badge/dependencies-none-green?style=flat-square)

---

## Features

- **EC2** — all T3, M6i, C6i, R6i instance types; on-demand and 1-year reserved pricing
- **RDS** — T3/M6g/R6g instance classes; single-AZ and Multi-AZ; gp3/io1 storage
- **S3** — all storage classes (Standard → Glacier Deep Archive); request and egress costs
- **Lambda** — request + compute costs with free tier deduction
- **CloudFront** — data transfer + HTTPS request pricing
- **Data Transfer** — tiered egress pricing (first 10TB, next 40TB, next 100TB)
- **Live API** — attempts to connect to the AWS Pricing API at startup; gracefully falls back to cached rates
- **Config files** — JSON architecture configs for repeatable estimates
- **Export** — save estimates to JSON for documentation or CI pipelines

---

## Quick Start

No installation required — uses only Python 3.9+ standard library.

```bash
git clone https://github.com/yourusername/cloud-cost-estimator
cd cloud-cost-estimator

# Interactive mode
python src/estimator.py

# Load from config file
python src/estimator.py --config examples/webapp.json

# Export result to JSON
python src/estimator.py --config examples/webapp.json --export estimate.json

# Skip live API check (faster)
python src/estimator.py --config examples/serverless_pipeline.json --no-live
```

### Web UI

Open `index.html` in any browser. No server required.

---

## CLI Output

```
  ╔════════════════════════════════════════════════════════════════════╗
  ║                      AWS CLOUD COST ESTIMATE                       ║
  ╚════════════════════════════════════════════════════════════════════╝

  Architecture: 3-Tier Web App
  Region:       us-east-1

  SERVICE BREAKDOWN
  ──────────────────────────────────────────────────────────────────────
  RDS             $  110.78  ███████████████████ 48.6%
  EC2             $   75.92  █████████████ 33.3%
  S3              $   20.50  ███ 9.0%
  CloudFront      $   11.70  ██ 5.1%
  Data Transfer   $    9.00  █ 3.9%
  ──────────────────────────────────────────────────────────────────────

  ESTIMATED MONTHLY COST:  $227.90
  ESTIMATED ANNUAL COST:   $2,734.80
```

---

## Architecture Config Format

Define your architecture in JSON and version-control it alongside your infra code:

```json
{
  "name": "My App",
  "region": "us-east-1",
  "ec2": [
    { "instance_type": "t3.medium", "count": 2, "hours_per_month": 730, "pricing_model": "on-demand", "os": "Linux" }
  ],
  "rds": [
    { "instance_class": "db.t3.medium", "engine": "PostgreSQL", "count": 1, "multi_az": true, "storage_gb": 100, "storage_class": "gp3" }
  ],
  "s3": [
    { "storage_gb": 500, "storage_class": "Standard", "get_requests_monthly": 5000000, "put_requests_monthly": 500000, "data_transfer_out_gb": 50 }
  ],
  "lambda": [],
  "cloudfront": [],
  "data_transfer_out_gb": 0
}
```

See the `examples/` directory for complete configs.

---

## AWS Pricing API

The tool attempts to reach the [AWS Pricing API](https://pricing.us-east-1.amazonaws.com) at startup — no authentication or AWS account required. On success it confirms connectivity; on failure (offline, restricted network) it silently falls back to the bundled pricing data (updated April 2025, us-east-1).

---

## Project Structure

```
cloud-cost-estimator/
├── src/
│   └── estimator.py       # CLI tool — zero dependencies
├── examples/
│   ├── webapp.json         # 3-tier web application
│   └── serverless_pipeline.json  # Serverless data pipeline
├── index.html              # Web UI — open in any browser
└── README.md
```

---

## Extending

Adding a new service is straightforward:

1. Add pricing data to `FALLBACK_PRICING` in `estimator.py`
2. Create a `@dataclass` for the component
3. Add a cost method to `PricingEngine`
4. Handle it in `CostCalculator.calculate()`
5. Mirror the logic in `index.html` for the web UI

Planned additions: ECS/EKS node groups, ElastiCache, API Gateway, SQS/SNS.

---

## Disclaimer

Estimates are based on us-east-1 on-demand rates unless specified. Actual AWS costs depend on exact usage patterns, Reserved Instance commitments, Savings Plans, and regional pricing variations. Always verify with the [official AWS Pricing Calculator](https://calculator.aws).

