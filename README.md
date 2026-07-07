# Delivery layout

```
infra2/               ← your root project (this is what you `terraform apply`)
github-modules/        ← 5 folders, one per module — push each as its OWN GitHub repo
```

## One-time setup

1. For each folder under `github-modules/`, create a GitHub repo with the
   **same name** and push its contents to the `main` branch:

   ```bash
   cd github-modules/terraform-aws-vpc
   git init
   git add .
   git commit -m "initial commit"
   git remote add origin https://github.com/<YOUR_GITHUB_ORG>/terraform-aws-vpc.git
   git push -u origin main
   ```

   Repeat for `terraform-aws-sg-module`, `terraform-aws-alb-module`,
   `terraform-aws-ecs-module`, `terraform-aws-rds-module`.

2. In `infra2/main.tf`, replace every `<YOUR_GITHUB_ORG>` placeholder (5
   occurrences, one per module block) with your actual GitHub org/user.

3. Update `infra2/envs/dev/backend.tf` and `infra2/envs/prod/backend.tf`
   with your real S3 state bucket name if `yourcompany-terraform-state1`
   isn't yours.

4. Deploy:

   ```bash
   cd infra2
   terraform init  -backend-config=envs/dev/backend.tf
   terraform apply -var-file=envs/dev/dev.tfvars
   ```

See `infra2/README.md` for the full architecture, SSM parameter paths, and
verification commands, and each module's own `README.md` for its
inputs/outputs.

## What's new vs. the previous version

- **`main.tf`** — all 5 `module` blocks now source from `git::https://...`
  instead of `./modules/...`. The old local path is kept as a commented-out
  line for quick local testing.
- **`data.tf`** (new) — `aws_caller_identity`, `aws_region`, `aws_partition`
  data sources, used for SSM parameter naming and available for any future
  tagging/ARN needs.
- **`parameter.tf`** (new) — publishes every important output (VPC/subnet
  IDs, security group IDs, ALB DNS/ARN, target group ARN, ECS
  cluster/service names, RDS address/port/instance id/secret ARN, account
  ID, region) to SSM Parameter Store under `/<project_name>/<environment>/<name>`,
  so other stacks can look values up without touching this stack's remote
  state.
- **`README.md`** — added at the root and inside every module folder.
