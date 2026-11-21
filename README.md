# Movie Analyst - AWS migration (Terraform + Ansible + CloudWatch)

Status: written and syntax-checked (`tofu fmt`, YAML parse, `ansible-playbook --syntax-check` up to the
missing collection), but NOT applied against a real AWS account. Expect to fix small things on first `apply`.

```
terraform/   root module + modules: network, security, iam, instance, alb, rds, monitoring
ansible/     roles: common (CloudWatch agent), nodejs, api, ui, db_init; dynamic inventory per env
app-fixes.patch   fixes for the application code (see "Application fixes")
```

## Deploy

0. **App fixes:** from the `devops-rampup` folder run `patch -p1 < app-fixes.patch`, then push each
   component to its own GitHub repo and set `api_repo_url` / `ui_repo_url` in `ansible/group_vars/all.yml`.
1. **State bucket (manual, per the task tip):** create an S3 bucket with versioning + encryption and put its name in
   `terraform/backend.tf`.
2. **Infra:**
   ```bash
   cd terraform
   cp terraform.tfvars.example terraform.tfvars   # edit your IP and email
   terraform init
   terraform workspace new qa
   terraform apply
   ```
   Confirm the SNS subscription email AWS sends you.
3. **Configure with Ansible, from the bastion:**
   ```bash
   ssh -A ec2-user@<bastion_public_ip>          # -A forwards your key; nothing secret is stored on the bastion
   cloud-init status --wait                     # bastion bootstrap installs Ansible
   # from your laptop:  scp -r ansible ec2-user@<bastion_public_ip>:~
   cd ansible && ansible-playbook -i inventories/qa.aws_ec2.yml site.yml
   ```
4. **Check:** open the `app_url` output; `curl http://<app_url>/movies` should list movies.
5. **Prod:** `terraform workspace new prod && terraform apply`, then run Ansible with `inventories/prod.aws_ec2.yml`.
   Run `terraform destroy` in `qa` when you are not testing (cost).

## Decisions and reasons (for the client write-up)

| Decision | Reason |
|---|---|
| Workspaces `qa` / `prod`, settings in an `environments` map keyed by workspace | One code base, two isolated states; no chance of applying prod values to qa with the wrong `-var-file` |
| S3 backend with native locking (`use_lockfile`), bucket created by hand | Team can work remotely without state corruption; no extra DynamoDB table; bucket must exist before Terraform can use it |
| Private modules (network, security, iam, instance, alb, rds, monitoring) | Reusable across company projects and maintainable by the team; public modules can replace them later |
| NAT **instance** instead of NAT Gateway | Gateway is ~USD 32/month and not free tier; trade-off: single instance, lower throughput, you patch it. Use a gateway if budget allows |
| Two ALBs (public for UI, internal for API) | Task requires backend traffic to go through a load balancer; the UI server calls the API, so an internal ALB sits between them |
| UI in public subnet, API/DB in private subnets, bastion in its own subnet | Matches the reference diagram and the mandatory rules |
| RDS MySQL 8.4, `db.t3.micro`, 20 GB gp2, single-AZ, encrypted, not public | Free-tier sizing; managed DB required by the task; DB reachable only from the API security group |
| Security groups reference other groups, not CIDRs | Least privilege chain Internet -> ALB -> UI -> ALB -> API -> RDS; SSH only via the bastion |
| IMDSv2 only, encrypted gp3 volumes, scoped IAM roles | Baseline hardening (Well-Architected security pillar) at no cost |
| Secrets in SSM Parameter Store (SecureString) | Free, unlike Secrets Manager; Ansible reads them with the bastion's/instances' IAM role. They are also in Terraform state, so protect the bucket |
| Ansible runs from the bastion with the `aws_ec2` dynamic inventory | Task requirement; inventory follows instance tags so no IP lists to maintain |
| App user `movie_app` with only DML on `movie_db` | The app does not need admin rights |
| CloudWatch only: 9 alarms, 1 dashboard, agent memory/disk metrics, 7-day logs, SNS email | In-house monitoring; sized to stay inside CloudWatch free-tier limits |

## Cost and free-tier notes (verify against your own account)

- AWS moved new accounts to a credit-based free plan in 2025; older accounts still have the 12-month free tier. Check yours.
- ALB hours are free only up to 750/month in total; two ALBs x two environments exceed that. Keep QA destroyed when idle.
- Public IPv4 addresses are billed hourly outside the free allowance (UI, NAT, bastion, ALBs).
- `t3.micro` is free-tier eligible only in some accounts/regions; some need `t2.micro`. Change `instance_type` in `environments.tf`.
- Custom metrics: the agent sends mem + disk per app instance (4 per env); the free tier covers 10.

## Application fixes (`app-fixes.patch`)

- `mysql` -> `mysql2`: the old `mysql` driver cannot authenticate against MySQL 8.4 (`caching_sha2_password`).
- `res.staus` -> `res.status` (3 places) and the `/pending` query (missing spaces, wrong column `release`, `movie_db.` prefixes).
- UI no longer crashes when the API is unreachable.
- Note: `schema.png` shows `movies.release`, but `seeds.js` and `/movies` use `release_year`; `schema.sql` follows the code.

## Things I could not verify

- `terraform plan/apply` (no AWS access here). Package names `nodejs20` / `nodejs20-npm` and the `amazon.aws.aws_ssm` lookup
  name depend on Amazon Linux 2023 / collection versions; adjust `nodejs_packages` or use `amazon.aws.ssm_parameter` if they fail.
- Region is `us-east-2` in `backend.tf`, `variables.tf`, `group_vars/all.yml` and the inventories; change all four together.
#
