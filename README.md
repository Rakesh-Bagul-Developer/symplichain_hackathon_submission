# Symplichain Hackathon Submission

**Role:** Software Engineering Intern  
**Submitted by:** Candidate  

## Repository Structure

```
.
├── .github/
│   └── workflows/
│       ├── deploy-staging.yml      # Part 3: Staging CI/CD workflow
│       └── deploy-production.yml   # Part 3: Production CI/CD workflow
└── README.md
```

## Part 3: GitHub Actions Workflows

### deploy-staging.yml
Triggered on push to `staging` branch.

**Jobs:**
1. `test` — Runs Django test suite with real Postgres + Redis service containers
2. `deploy-backend` — SSH into staging EC2, git pull, pip install, migrate, restart Gunicorn + Celery
3. `deploy-frontend` — npm build with `REACT_APP_ENV=staging`, sync to S3, invalidate CloudFront

### deploy-production.yml
Triggered on push to `main` branch.

**Differences from staging:**
- Uses `environment: production` — requires manual approval in GitHub Environments
- Uses `kill -HUP` (SIGHUP) for zero-downtime Gunicorn reload instead of `systemctl restart`
- Uses separate production secrets (S3 bucket, EC2 host, CF distribution)

### Required GitHub Secrets

| Secret | Description |
|--------|-------------|
| `AWS_ACCESS_KEY_ID` | IAM user with EC2, S3, CloudFront, Secrets Manager access |
| `AWS_SECRET_ACCESS_KEY` | Corresponding secret key |
| `STAGING_EC2_HOST` | Staging EC2 public IP or hostname |
| `STAGING_EC2_SSH_KEY` | PEM private key for staging EC2 |
| `STAGING_S3_BUCKET` | Staging S3 bucket name |
| `STAGING_CF_DISTRIBUTION_ID` | Staging CloudFront distribution ID |
| `STAGING_API_URL` | Base URL for staging API (used in frontend build) |
| `PROD_EC2_HOST` | Production EC2 host |
| `PROD_EC2_SSH_KEY` | PEM private key for production EC2 |
| `PROD_S3_BUCKET` | Production S3 bucket name |
| `PROD_CF_DISTRIBUTION_ID` | Production CloudFront distribution ID |
| `PROD_API_URL` | Base URL for production API |

### Assumptions
- Django project lives at `/var/www/symplichain` on the EC2 instance
- A Python virtualenv exists at `/var/www/symplichain/venv`
- Gunicorn PID file is at `/run/gunicorn/gunicorn.pid`
- Systemd units: `gunicorn`, `celery`, `celerybeat`, `nginx`
- The EC2 instance role has `secretsmanager:GetSecretValue` permission
- Frontend React app lives in `./frontend/` subdirectory
- Node 20, Python 3.11

## Full Submission PDF

See `Symplichain_Hackathon_Submission.pdf` for answers to all 4 parts:
- Part 1: Shared Gateway Problem (rate limiting + fairness architecture)
- Part 2: Mobile App Architecture
- Part 3: CI/CD (this repo + Docker/Terraform discussion)
- Part 4: Monday Morning Outage Debugging walkthrough
