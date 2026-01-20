---
id: infrastructure-as-code
title: "Infrastructure as Code (IaC)"
type: concept
status: learning
importance: 80
difficulty: medium
tags: [devops, automation, infrastructure, terraform, cloudformation]
canonical: true
aliases: [iac, infrastructure-automation]
created_at: 2026-01-20
updated_at: 2026-01-20
---

# Infrastructure as Code (IaC)

## TL;DR (BLUF)
- Infrastructure as Code manages and provisions infrastructure through machine-readable configuration files instead of manual processes.
- Use declarative configs (Terraform, CloudFormation) to define desired state; tools ensure infrastructure matches.
- Key trade-off: upfront learning and setup cost vs. repeatability and reliability.

## Definition
**What it is:** The practice of managing and provisioning infrastructure (servers, networks, databases) using code and automation rather than manual configuration.  
**Key terms:** declarative (define what you want), imperative (define how to achieve it), desired state, idempotent (safe to run multiple times), drift detection, plan/apply workflow, state management.

## Why it matters
- **Repeatability:** Infrastructure is defined in code; can recreate environments identically.
- **Version control:** Infrastructure changes tracked in Git (audit trail, rollback capability).
- **Reduces errors:** Automation eliminates manual mistakes (typos, forgotten steps).
- **Faster provisioning:** Spin up environments in minutes vs. hours/days.
- **Interview relevance:** IaC is fundamental for modern DevOps and cloud infrastructure discussions.

## Scope & Non-goals
**In scope:**
- IaC concepts (declarative vs. imperative, desired state).
- Common tools (Terraform, CloudFormation, Ansible—conceptually).
- Best practices (state management, modularity, testing).

**Out of scope / NOT solved by this:**
- Specific tool tutorials (Terraform syntax, CloudFormation examples—separate topics).
- Cloud platform specifics (AWS vs. GCP vs. Azure—separate topics).
- Configuration management (Chef, Puppet—related but different focus).

## Mental model / Intuition
Think of IaC as **blueprints for a house**:
- **Manual process:** Verbally instruct builders ("build a wall here, add a window there")—error-prone, not repeatable.
- **IaC:** Detailed blueprints (architectural drawings)—builders can recreate the house identically anywhere.
- **Declarative IaC:** You specify what the house should look like; builder figures out how to build it.
- **Imperative IaC:** You specify step-by-step instructions (first foundation, then walls, then roof).

## Decision rules (When to use / When not to use)
### Use it when
- Managing cloud infrastructure (AWS, GCP, Azure).
- Need to recreate environments (dev, staging, prod).
- Working in teams (IaC enables collaboration and review).
- Infrastructure changes frequently (IaC makes changes predictable and safe).

### Avoid it when
- One-time, throwaway infrastructure (manual setup may be faster).
- Extremely simple setups (single VM with no complexity).
- Team lacks DevOps skills (steep learning curve without support).

## How I would use it (practical)
- **Context:** Provisioning AWS infrastructure for a Go API service (VPC, RDS, ECS, Load Balancer).
- **Steps (Terraform example):**
  1. **Define infrastructure in code:**
     ```hcl
     resource "aws_vpc" "main" {
       cidr_block = "10.0.0.0/16"
     }
     
     resource "aws_db_instance" "postgres" {
       engine         = "postgres"
       instance_class = "db.t3.micro"
       allocated_storage = 20
     }
     
     resource "aws_ecs_cluster" "api" {
       name = "api-cluster"
     }
     ```
  2. **Plan changes:**
     - Run `terraform plan` (shows what will be created/modified/destroyed).
     - Review changes (catch mistakes before applying).
  3. **Apply changes:**
     - Run `terraform apply` (creates infrastructure to match code).
     - Terraform tracks state (knows what exists, what needs to change).
  4. **Version control:**
     - Commit infrastructure code to Git.
     - Changes reviewed via PR (same as application code).
  5. **Recreate environments:**
     - Use same code to create dev, staging, prod (parameterize differences).
- **What success looks like:** Spin up full environment in 10 minutes; zero manual steps; infrastructure changes tracked in Git.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:**
  - Repeatable (recreate environments identically).
  - Auditable (Git history shows who changed what, when).
  - Reduces errors (automation vs. manual).
  - Faster (provision in minutes vs. hours).
- **Cons / Risks:**
  - Learning curve (Terraform, CloudFormation syntax).
  - State management complexity (state files must be stored and locked).
  - Initial setup cost (writing IaC takes time upfront).
  - Drift risk (manual changes outside IaC break desired state).

### Alternatives
- **Manual provisioning:** Click-ops (AWS Console, GCP UI)—fast initially but error-prone and not repeatable.
- **Scripts (Bash, Python):** Imperative automation—flexible but harder to maintain than declarative IaC.
- **Configuration management (Ansible, Chef):** Focus on software config, not infrastructure provisioning.

### How to choose
- **Cloud infrastructure:** Use IaC (Terraform for multi-cloud, CloudFormation for AWS-only).
- **Simple, one-time setup:** Manual provisioning may be faster.
- **Software config (install packages, config files):** Configuration management (Ansible).

## Failure modes & Pitfalls
- **Manual drift:** Engineers make manual changes (click-ops); IaC state no longer matches reality.
- **State file loss:** Terraform state file deleted/corrupted; loses track of what exists.
- **Concurrent modifications:** Two engineers run `terraform apply` simultaneously; state conflicts.
- **No modularity:** Monolithic IaC files (1000+ lines); hard to maintain and reuse.
- **No testing:** Apply changes to production without testing in dev/staging; breaks production.

## Observability (How to detect issues)
- **Metrics:**
  - Drift detection (% of infrastructure out of sync with IaC).
  - Apply duration (how long infrastructure changes take).
  - Apply success rate (% of applies that succeed without errors).
- **Logs:** Track `terraform plan` and `terraform apply` outputs (audit trail).
- **Traces:** N/A for IaC itself.
- **Alerts:** Drift detected, apply failures, state file locking errors.

## Implementation notes
- **Checklist**
  - [ ] Choose IaC tool (Terraform, CloudFormation, Pulumi).
  - [ ] Store state remotely (S3 + DynamoDB for Terraform; prevents loss and enables locking).
  - [ ] Use modules (reusable components; DRY).
  - [ ] Version control infrastructure code (Git).
  - [ ] Require PR reviews for infrastructure changes.
  - [ ] Run `plan` before `apply` (validate changes).
  - [ ] Test in dev/staging before production.
  - [ ] Detect and remediate drift (run drift detection regularly).

- **Security / Compliance notes**
  - Store secrets in secret managers (AWS Secrets Manager, HashiCorp Vault), not in IaC code.
  - Least privilege for IaC execution (limit permissions to what's needed).
  - Audit infrastructure changes (Git history, CloudTrail).

- **Performance notes**
  - Parallelize resource creation where possible (Terraform does this automatically).
  - Use smaller, focused modules (faster to plan and apply).

- **Operational notes**
  - Runbook for state file recovery (backup state files regularly).
  - Document module usage (README in each module).

## Mini example (Terraform)
```hcl
# main.tf - AWS VPC and RDS
terraform {
  required_version = ">= 1.0"
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"  # Prevents concurrent applies
  }
}

provider "aws" {
  region = "us-east-1"
}

resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  
  tags = {
    Name = "main-vpc"
  }
}

resource "aws_subnet" "public" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.1.0/24"
  availability_zone = "us-east-1a"
  
  tags = {
    Name = "public-subnet"
  }
}

resource "aws_db_instance" "postgres" {
  identifier        = "myapp-db"
  engine            = "postgres"
  engine_version    = "15.3"
  instance_class    = "db.t3.micro"
  allocated_storage = 20
  db_name           = "myapp"
  username          = "admin"
  password          = var.db_password  # From secret manager or tfvars
  skip_final_snapshot = true
  
  tags = {
    Name = "myapp-database"
  }
}

# Workflow:
# 1. terraform init    # Initialize, download providers
# 2. terraform plan    # Preview changes
# 3. terraform apply   # Create/update infrastructure
```

## Common anti-patterns
- **Anti-pattern:** Hardcoding secrets in IaC code.
  - **Why it's bad:** Secrets exposed in Git history, version control.
  - **Better approach:** Use secret managers (AWS Secrets Manager, Vault); reference in IaC.

- **Anti-pattern:** No remote state storage.
  - **Why it's bad:** State file stored locally; lost if computer dies; can't collaborate.
  - **Better approach:** Store state remotely (S3, Terraform Cloud) with locking (DynamoDB).

- **Anti-pattern:** Making manual changes outside IaC.
  - **Why it's bad:** Creates drift; IaC no longer reflects reality; next apply may break things.
  - **Better approach:** Always change infrastructure via IaC; import manual resources into state.

- **Anti-pattern:** Monolithic IaC files (1000+ lines).
  - **Why it's bad:** Hard to maintain, reuse, and understand.
  - **Better approach:** Use modules (VPC module, RDS module, etc.); compose infrastructure from reusable components.

## Interview readiness
### "Explain it like I'm teaching"
Infrastructure as Code treats infrastructure the same way we treat application code. Instead of manually clicking through the AWS console to create servers and databases, you write configuration files that declare what you want. Tools like Terraform read these files and create the infrastructure automatically. The benefits are repeatability (recreate environments identically), version control (track changes in Git), and reduced errors (automation beats manual). You run `plan` to preview changes, then `apply` to make them. State files track what exists, so IaC knows what to create, update, or destroy. The main challenge is learning the tool and managing state, but the payoff is reliable, fast infrastructure provisioning.

### Trap questions (with answers)
1) **Q:** What's the difference between declarative and imperative IaC?
   - **A:** 
     - **Declarative (Terraform, CloudFormation):** You specify *what* you want (e.g., "I want a VPC with this CIDR"). Tool figures out *how* to achieve it.
     - **Imperative (scripts, Ansible):** You specify *how* to achieve it (e.g., "Create VPC, then create subnet, then create route table").
     - Declarative is easier to reason about (desired state); imperative gives more control.

2) **Q:** How do you handle secrets in IaC?
   - **A:** Never hardcode secrets in IaC code. Options:
     - Store in secret manager (AWS Secrets Manager, Vault); reference in IaC.
     - Use environment variables or encrypted tfvars files (not committed to Git).
     - For sensitive data (passwords), generate dynamically or rotate regularly.

3) **Q:** What is Terraform state and why does it matter?
   - **A:** Terraform state is a file that tracks what infrastructure exists and how it maps to your code. It's critical because Terraform compares state to code to determine what needs to change. If state is lost, Terraform doesn't know what exists and may try to recreate everything. Store state remotely (S3) with locking (DynamoDB) to prevent loss and concurrent modification.

4) **Q:** How do you handle infrastructure drift?
   - **A:** Drift occurs when infrastructure is manually changed outside IaC. Detect drift by running `terraform plan` (shows differences between code and reality). Remediate by either:
     - Updating IaC to match reality (if manual change is desired).
     - Running `terraform apply` to revert to IaC-defined state (if manual change is unwanted).
     - Use drift detection tools (Terraform Cloud, AWS Config) for continuous monitoring.

5) **Q:** Can you use IaC for multi-cloud?
   - **A:** Yes. Terraform supports multi-cloud (AWS, GCP, Azure, etc.) via providers. Define resources for each cloud in same codebase. CloudFormation is AWS-only. Pulumi also supports multi-cloud and allows using programming languages (TypeScript, Python) instead of DSLs.

### Quick self-check (5 items)
- [ ] I can explain what IaC is and why it's valuable (repeatability, version control, automation).
- [ ] I can describe the plan/apply workflow (preview changes, then execute).
- [ ] I can explain the difference between declarative and imperative IaC.
- [ ] I can describe how to handle secrets and state management in IaC.
- [ ] I can name at least 3 IaC tools and their trade-offs (Terraform, CloudFormation, Ansible).

## Links (NO duplication)
### Prerequisites
- [CI/CD basics](../quality/ci-cd-basics.md)
- [Operational planning](operational-planning.md)

### Related topics
- [Blue/green deployments](blue-green-deployments.md)
- [Canary deployments](canary-deployments.md)
- [Terraform](terraform.md)
- [AWS CloudFormation](aws-cloudformation.md)

### Compare with
- Configuration management (Ansible, Chef) — IaC provisions infrastructure; config management configures software.

## Notes / Inbox
- Add examples from real projects: Terraforming Heimdall infrastructure, Opportunity Actions AWS setup.
- Consider adding section on IaC testing (Terratest, Kitchen-Terraform).
- Link to specific tools (Terraform, CloudFormation, Pulumi) when detailed topics are created.
