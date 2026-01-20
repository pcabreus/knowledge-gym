---
id: terraform
title: "Terraform (Detailed)"
type: tool
status: learning
importance: 85
difficulty: medium
tags: [iac, automation, terraform, infrastructure, operations]
canonical: true
aliases: [tf, hashicorp-terraform]
created_at: 2026-01-20
updated_at: 2026-01-20
---

# Terraform (Detailed)

## TL;DR (BLUF)
- Terraform is a declarative Infrastructure as Code (IaC) tool for managing cloud resources across multiple providers (AWS, GCP, Azure).
- Use for reproducible infrastructure, version-controlled configs, and multi-cloud deployments.
- Key trade-off: powerful abstraction and multi-cloud support vs. state management complexity.

## Definition
**What it is:** An open-source IaC tool by HashiCorp that uses declarative configuration (HCL—HashiCorp Configuration Language) to provision and manage infrastructure across cloud providers.  
**Key terms:** HCL (HashiCorp Configuration Language), provider, resource, module, state file, plan/apply/destroy, remote state, workspaces, drift detection.

## Why it matters
- **Multi-cloud:** Single tool for AWS, GCP, Azure, Kubernetes, etc. (1,000+ providers).
- **Declarative:** Define desired state; Terraform figures out how to achieve it.
- **Reproducible:** Same config produces identical infrastructure (no manual clicks).
- **Version-controlled:** Track infrastructure changes in Git (code review, rollback).
- **Collaboration:** Remote state enables team collaboration (locking, shared state).
- **Interview relevance:** Standard IaC tool in cloud infrastructure discussions.

## Scope & Non-goals
**In scope:**
- Terraform core concepts (providers, resources, modules, state).
- Workflow (init → plan → apply → destroy).
- When to use Terraform vs. alternatives.

**Out of scope / NOT solved by this:**
- Provider-specific details (AWS specifics belong in AWS docs).
- Advanced features (custom providers, Sentinel policies).
- Terraform Cloud/Enterprise features.

## Mental model / Intuition
Think of Terraform as a **blueprint for your infrastructure**:
- You write a blueprint (HCL config) describing desired state: "I want 3 EC2 instances, 1 RDS database."
- Terraform reads the blueprint and compares it to current state (state file).
- It calculates a plan: "Create 2 EC2 instances (1 already exists), create RDS."
- You approve, and Terraform executes the plan (API calls to cloud provider).
- State file is updated to match reality.

```hcl
# Blueprint (HCL)
resource "aws_instance" "web" {
  ami           = "ami-12345"
  instance_type = "t2.micro"
  count         = 3  # Desired state: 3 instances
}
```

## Decision rules (When to use / When not to use)
### Use it when
- Managing cloud infrastructure (AWS, GCP, Azure).
- Need multi-cloud support (avoid vendor lock-in).
- Want version-controlled infrastructure (Git workflow).
- Collaborating with team (remote state, locking).
- Need reproducible environments (dev/staging/prod identical).

### Avoid it when
- Provider-native tool is simpler (e.g., AWS CDK for AWS-only, simpler abstractions).
- Configuration management needed (use Ansible; Terraform is for provisioning, not config).
- Immutable infrastructure with container orchestration (Kubernetes YAML may be simpler).
- Small, static infrastructure (manual management acceptable).

## How I would use it (practical)
- **Context:** Provisioning AWS infrastructure for a web app (EC2, RDS, ALB).
- **Steps:**
  1. **Write Terraform config:**
     ```hcl
     # main.tf
     terraform {
       required_providers {
         aws = {
           source  = "hashicorp/aws"
           version = "~> 5.0"
         }
       }
     }

     provider "aws" {
       region = "us-east-1"
     }

     resource "aws_instance" "web" {
       ami           = "ami-12345"
       instance_type = "t2.micro"
       tags = {
         Name = "WebServer"
       }
     }

     resource "aws_db_instance" "db" {
       allocated_storage = 20
       engine            = "postgres"
       instance_class    = "db.t3.micro"
       username          = "admin"
       password          = var.db_password  # From variables
     }
     ```
  2. **Initialize Terraform:**
     ```bash
     terraform init  # Downloads AWS provider plugin
     ```
  3. **Plan changes:**
     ```bash
     terraform plan  # Shows what will be created
     # Output:
     # + aws_instance.web
     # + aws_db_instance.db
     ```
  4. **Apply changes:**
     ```bash
     terraform apply  # Creates resources
     # Prompts for confirmation; type "yes"
     ```
  5. **State file created:**
     - `terraform.tfstate` tracks created resources.
     - Use remote state (S3 + DynamoDB) for teams.
  6. **Make changes:**
     - Edit `main.tf` (e.g., change instance type to `t2.small`).
     - Run `terraform plan` → shows diff.
     - Run `terraform apply` → updates instance.
  7. **Destroy when done:**
     ```bash
     terraform destroy  # Deletes all resources
     ```
- **What success looks like:** Reproducible infrastructure; changes tracked in Git; team uses shared state.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:**
  - Multi-cloud (AWS, GCP, Azure, Kubernetes, etc.).
  - Declarative (simple syntax; focus on desired state).
  - Large ecosystem (modules, providers, community).
  - State management (knows current vs. desired state).
  - Version-controlled (Git workflow for infrastructure).
- **Cons / Risks:**
  - State file complexity (must manage, secure, share state).
  - No built-in secrets management (passwords in plaintext; use Vault).
  - Drift detection (manual; must run plan to detect changes).
  - Locking required for teams (prevent concurrent applies).
  - Refactoring is hard (renaming resources requires state manipulation).

### Alternatives
- **[AWS CloudFormation](aws-cloudformation.md):** AWS-native; deeper AWS integration; free (but AWS-only).
- **Pulumi:** Uses real programming languages (Python, Go); more expressive (but less mature).
- **AWS CDK:** AWS-native; uses TypeScript/Python; generates CloudFormation (but AWS-only).
- **Ansible:** Configuration management; can provision infrastructure (but less declarative).

### How to choose
- **Multi-cloud:** Terraform.
- **AWS-only, deep integration:** CloudFormation or CDK.
- **Programming language preference:** Pulumi or CDK.
- **Configuration management:** Ansible.

## Failure modes & Pitfalls
- **State file loss:** State file deleted; Terraform doesn't know what it created (orphaned resources).
- **State file corruption:** Concurrent applies without locking; state file conflicts.
- **Secrets in plaintext:** Passwords in `.tf` files or state file; committed to Git (security risk).
- **Drift (manual changes):** Someone edits resource in AWS console; Terraform doesn't know; next apply reverts change.
- **Dependency issues:** Resource A depends on B, but not declared; Terraform creates in wrong order (fails).
- **No rollback:** Terraform has no native rollback; must revert Git commit and re-apply.

## Observability (How to detect issues)
- **Metrics:**
  - Terraform run success rate (% of applies that succeed).
  - Drift detection frequency (how often resources deviate from state).
  - Apply duration (slow applies indicate complex dependencies).
- **Logs:** Terraform logs (DEBUG mode for troubleshooting).
- **Alerts:** State file lock held for >30 minutes (stuck apply), drift detected, apply failed.
- **Drift detection:** Run `terraform plan` regularly (CI job); alert if changes detected (indicates manual edits).

## Implementation notes
- **Checklist**
  - [ ] Use remote state (S3 + DynamoDB for locking).
  - [ ] Never commit state file to Git (add `*.tfstate` to `.gitignore`).
  - [ ] Use variables for secrets (pass via env vars or Vault).
  - [ ] Organize code (modules for reusable components).
  - [ ] Use workspaces for environments (dev/staging/prod).
  - [ ] Enable state locking (DynamoDB table for concurrent protection).
  - [ ] Run `terraform plan` before every apply (review changes).

- **Security / Compliance notes**
  - Store state file securely (S3 with encryption, restricted access).
  - Never commit secrets to Git (use environment variables or Vault).
  - Use least-privilege IAM roles (Terraform needs only necessary permissions).
  - Audit Terraform applies (track who applied what, when).

- **Performance notes**
  - Parallelize resource creation (Terraform does this automatically; use `-parallelism` to limit).
  - Use targeted applies for large infrastructures (`terraform apply -target=resource`).

- **Operational notes**
  - Document state file location (team must know where state is stored).
  - Schedule drift detection (daily CI job runs `terraform plan`).
  - Plan for state migration (if changing backend or splitting monolith).

## Mini example (Terraform workflow)
```hcl
# main.tf
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"  # For locking
    encrypt        = true
  }

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}

variable "aws_region" {
  default = "us-east-1"
}

variable "instance_count" {
  default = 2
}

resource "aws_instance" "app" {
  count         = var.instance_count
  ami           = "ami-12345"
  instance_type = "t2.micro"

  tags = {
    Name = "AppServer-${count.index}"
  }
}

output "instance_ids" {
  value = aws_instance.app[*].id
}

# Workflow:
# 1. terraform init          # Download provider, setup backend
# 2. terraform plan          # Preview changes
# 3. terraform apply         # Create resources
# 4. terraform output        # View outputs
# 5. terraform destroy       # Clean up
```

**Example output:**
```bash
$ terraform plan
Terraform will perform the following actions:

  # aws_instance.app[0] will be created
  + resource "aws_instance" "app" {
      + ami           = "ami-12345"
      + instance_type = "t2.micro"
      + tags          = { "Name" = "AppServer-0" }
    }

  # aws_instance.app[1] will be created
  + resource "aws_instance" "app" {
      + ami           = "ami-12345"
      + instance_type = "t2.micro"
      + tags          = { "Name" = "AppServer-1" }
    }

Plan: 2 to add, 0 to change, 0 to destroy.

$ terraform apply
# ... creates resources ...

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.

Outputs:
instance_ids = ["i-abc123", "i-def456"]
```

## Common anti-patterns
- **Anti-pattern:** Committing state file to Git.
  - **Why it's bad:** State contains sensitive data (resource IDs, IPs); merge conflicts break state.
  - **Better approach:** Use remote state (S3 backend); add `*.tfstate` to `.gitignore`.

- **Anti-pattern:** No state locking (multiple applies at once).
  - **Why it's bad:** Concurrent applies corrupt state file; resources created twice.
  - **Better approach:** Enable locking (DynamoDB table for S3 backend).

- **Anti-pattern:** Hardcoding secrets in `.tf` files.
  - **Why it's bad:** Secrets committed to Git; state file contains plaintext passwords.
  - **Better approach:** Use variables with env vars (`TF_VAR_db_password`) or Vault integration.

- **Anti-pattern:** Monolithic config (all resources in one file).
  - **Why it's bad:** Hard to read; slow applies; team conflicts on same file.
  - **Better approach:** Use modules (separate VPC, compute, database into modules).

- **Anti-pattern:** Not running `terraform plan` before apply.
  - **Why it's bad:** Unexpected changes (destroy resources accidentally).
  - **Better approach:** Always plan first; review changes; then apply.

## Interview readiness
### "Explain it like I'm teaching"
Terraform is an Infrastructure as Code tool that lets you define cloud resources in a configuration file using HCL (HashiCorp Configuration Language). You write code like "I want 3 EC2 instances and 1 RDS database," and Terraform makes API calls to AWS to create them. The workflow is: (1) write config, (2) `terraform init` to download provider plugins, (3) `terraform plan` to preview changes, (4) `terraform apply` to create resources. Terraform tracks what it created in a "state file," so it knows current vs. desired state. If you later change the config (e.g., increase instances to 5), Terraform calculates the diff and updates only what changed. The main benefits are multi-cloud support (works with AWS, GCP, Azure), version control (track infrastructure changes in Git), and reproducibility (same config = identical infrastructure). The downsides are state file management (must secure and share state) and no native secrets handling.

### Trap questions (with answers)
1) **Q:** What is the Terraform state file, and why is it important?
   - **A:** The state file (`terraform.tfstate`) is a JSON file that maps Terraform config to real-world resources. It stores resource IDs, attributes, and dependencies. It's critical because Terraform uses it to know what currently exists, so it can calculate diffs (what to create/update/delete). Without state, Terraform can't manage infrastructure. Best practice: use remote state (S3) and enable locking (DynamoDB) to prevent conflicts.

2) **Q:** How does Terraform handle dependencies between resources?
   - **A:** Terraform builds a dependency graph. You can declare explicit dependencies (`depends_on`), but Terraform also infers implicit dependencies (e.g., security group ID referenced in instance config). It creates resources in correct order (security group first, then instance). If creation fails, Terraform stops and leaves partial state (you can retry or destroy).

3) **Q:** What happens if someone manually changes a resource in the AWS console?
   - **A:** Terraform doesn't know about the change (state file is stale). This is called "drift." Next time you run `terraform plan`, Terraform detects the difference (actual state vs. config) and proposes reverting the manual change. To detect drift regularly, run `terraform plan` in CI (e.g., nightly job) and alert if changes detected.

4) **Q:** How do you manage secrets in Terraform?
   - **A:** Don't hardcode secrets in `.tf` files (they'd be in Git and state file). Instead: (1) use variables and pass secrets via environment variables (`TF_VAR_db_password=secret terraform apply`), (2) integrate with HashiCorp Vault or AWS Secrets Manager, (3) use sensitive=true for variables (hides output), (4) encrypt state file (S3 backend with encryption).

5) **Q:** What's the difference between Terraform and CloudFormation?
   - **A:** 
     - **Terraform:** Multi-cloud (AWS, GCP, Azure); HCL syntax; open-source; requires state management.
     - **CloudFormation:** AWS-only; JSON/YAML; free; AWS-managed state (no state file to manage).
     - Choose Terraform for multi-cloud or vendor neutrality; choose CloudFormation for AWS-only with deeper AWS integration.

### Quick self-check (5 items)
- [ ] I can explain the Terraform workflow (init → plan → apply → destroy).
- [ ] I can describe the role of the state file.
- [ ] I can articulate when to use Terraform vs. CloudFormation.
- [ ] I can explain how to manage secrets securely in Terraform.
- [ ] I can identify drift and how to detect it.

## Links (NO duplication)
### Prerequisites
- [Infrastructure as Code basics](infrastructure-as-code.md)

### Related topics
- [AWS CloudFormation](aws-cloudformation.md)
- [CI/CD basics](../quality/ci-cd-basics.md)

### Compare with
- [AWS CloudFormation](aws-cloudformation.md) — CloudFormation is AWS-only; Terraform is multi-cloud.

## Notes / Inbox
- Add examples from real projects: Terraform modules for Heimdall infrastructure.
- Consider adding section on Terraform workspaces (dev/staging/prod).
- Link to Terraform modules best practices when created.
