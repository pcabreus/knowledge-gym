---
id: aws-cloudformation
title: "AWS CloudFormation (Detailed)"
type: tool
status: learning
importance: 80
difficulty: medium
tags: [iac, aws, cloudformation, infrastructure, operations]
canonical: true
aliases: [cfn, cloudformation]
created_at: 2026-01-20
updated_at: 2026-01-20
---

# AWS CloudFormation (Detailed)

## TL;DR (BLUF)
- CloudFormation is AWS's native Infrastructure as Code (IaC) service for provisioning and managing AWS resources using declarative templates (JSON/YAML).
- Use for AWS-only infrastructure, deep AWS integration, and managed state.
- Key trade-off: AWS-specific convenience and free managed state vs. vendor lock-in.

## Definition
**What it is:** AWS's native IaC service that uses declarative templates (JSON or YAML) to provision and manage AWS resources as a single unit called a "stack."  
**Key terms:** stack, template, resource, parameter, output, drift detection, change set, nested stacks, StackSets (multi-account/region), rollback, AWS-managed state.

## Why it matters
- **AWS-native:** Deep integration with AWS services (native support for all AWS resources).
- **Managed state:** AWS manages state for you (no state file to secure or share).
- **Free:** No additional cost (pay only for AWS resources created).
- **Declarative:** Define desired state; CloudFormation handles provisioning.
- **Rollback:** Automatic rollback on failure (stack creation fails → deletes all resources).
- **Interview relevance:** Standard IaC for AWS-centric architectures.

## Scope & Non-goals
**In scope:**
- CloudFormation core concepts (stacks, templates, resources, parameters).
- When to use CloudFormation vs. alternatives.
- Drift detection and change sets.

**Out of scope / NOT solved by this:**
- Multi-cloud (CloudFormation is AWS-only; use Terraform for multi-cloud).
- Advanced features (custom resources, macros, AWS CDK).
- Specific resource syntax (refer to AWS docs for each resource type).

## Mental model / Intuition
Think of CloudFormation as a **construction blueprint for AWS**:
- You write a blueprint (YAML template) describing desired infrastructure: "I want 1 VPC, 2 subnets, 1 EC2 instance, 1 RDS database."
- CloudFormation reads the blueprint and creates a "stack" (group of resources).
- It provisions resources in the correct order (VPC first, then subnets, then instances).
- If anything fails, CloudFormation automatically deletes everything (rollback).
- AWS tracks the stack state internally (you don't manage state files).

```yaml
# Blueprint (YAML template)
Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-12345
      InstanceType: t2.micro
```

## Decision rules (When to use / When not to use)
### Use it when
- AWS-only infrastructure (no multi-cloud needs).
- Want managed state (no S3 buckets or locking to configure).
- Need deep AWS integration (native support for all AWS services).
- Free IaC tool is important (CloudFormation is free).
- Want automatic rollback (stack creation fails → all resources deleted).

### Avoid it when
- Multi-cloud or vendor neutrality needed (use Terraform).
- Prefer programming languages over YAML (use AWS CDK or Pulumi).
- Need advanced abstractions (CDK provides higher-level constructs).
- Template size limits are restrictive (51,200 bytes for YAML; use nested stacks or Terraform).

## How I would use it (practical)
- **Context:** Provisioning AWS infrastructure for a web app (VPC, EC2, RDS).
- **Steps:**
  1. **Write CloudFormation template:**
     ```yaml
     # app-stack.yaml
     AWSTemplateFormatVersion: '2010-09-09'
     Description: Web app infrastructure

     Parameters:
       InstanceType:
         Type: String
         Default: t2.micro
         AllowedValues:
           - t2.micro
           - t2.small

     Resources:
       WebServer:
         Type: AWS::EC2::Instance
         Properties:
           ImageId: ami-12345
           InstanceType: !Ref InstanceType
           Tags:
             - Key: Name
               Value: WebServer

       Database:
         Type: AWS::RDS::DBInstance
         Properties:
           DBInstanceClass: db.t3.micro
           Engine: postgres
           AllocatedStorage: 20
           MasterUsername: admin
           MasterUserPassword: !Ref DBPassword

     Outputs:
       InstanceId:
         Value: !Ref WebServer
         Description: EC2 instance ID
     ```
  2. **Create stack:**
     ```bash
     aws cloudformation create-stack \
       --stack-name my-app-stack \
       --template-body file://app-stack.yaml \
       --parameters ParameterKey=DBPassword,ParameterValue=secret123
     ```
  3. **Monitor stack creation:**
     ```bash
     aws cloudformation describe-stacks --stack-name my-app-stack
     # Status: CREATE_IN_PROGRESS → CREATE_COMPLETE
     ```
  4. **Update stack:**
     - Edit template (e.g., change instance type to `t2.small`).
     - Create change set (preview changes):
       ```bash
       aws cloudformation create-change-set \
         --stack-name my-app-stack \
         --change-set-name update-instance-type \
         --template-body file://app-stack.yaml
       ```
     - Review change set, then execute:
       ```bash
       aws cloudformation execute-change-set \
         --change-set-name update-instance-type \
         --stack-name my-app-stack
       ```
  5. **Delete stack:**
     ```bash
     aws cloudformation delete-stack --stack-name my-app-stack
     # Deletes all resources (EC2, RDS, etc.)
     ```
- **What success looks like:** Reproducible infrastructure; automatic rollback on failure; no state files to manage.

## Trade-offs & Alternatives
### Trade-offs
- **Pros:**
  - AWS-native (supports all AWS services immediately).
  - Managed state (no S3 buckets, no locking configuration).
  - Free (no additional cost beyond AWS resources).
  - Automatic rollback (stack creation fails → all resources deleted).
  - Drift detection (detect manual changes via AWS console).
  - Change sets (preview changes before apply).
- **Cons / Risks:**
  - AWS-only (vendor lock-in; can't manage GCP or Azure).
  - Verbose YAML/JSON (less expressive than Terraform HCL or CDK).
  - Template size limits (51,200 bytes; large infra requires nested stacks).
  - No modules (reusability harder than Terraform modules).
  - Slower updates (AWS adds new services to CloudFormation with delay).

### Alternatives
- **[Terraform](terraform.md):** Multi-cloud; HCL syntax; requires state management (but more flexible).
- **AWS CDK:** Uses programming languages (TypeScript, Python); generates CloudFormation (but adds abstraction layer).
- **Pulumi:** Multi-cloud; uses programming languages; more expressive (but requires state management).
- **Ansible:** Configuration management; can provision infrastructure (but less declarative).

### How to choose
- **AWS-only, free, managed state:** CloudFormation.
- **Multi-cloud or vendor neutrality:** Terraform.
- **Programming language preference (AWS-only):** AWS CDK.
- **Programming language preference (multi-cloud):** Pulumi.

## Failure modes & Pitfalls
- **Rollback on failure:** Stack creation fails (e.g., invalid AMI); CloudFormation deletes all resources (can lose progress).
- **Circular dependencies:** Resource A depends on B, B depends on A; stack creation fails.
- **Manual changes (drift):** Someone edits resource in console; CloudFormation doesn't know; next update may revert change.
- **DeletionPolicy not set:** Delete stack; RDS database deleted (data loss).
- **No parameter validation:** Pass invalid parameter (e.g., wrong instance type); stack creation fails.
- **Update requires replacement:** Change instance type; CloudFormation deletes old instance, creates new one (downtime).

## Observability (How to detect issues)
- **Metrics:**
  - Stack creation success rate (% of stacks that complete successfully).
  - Stack update duration (slow updates indicate complex dependencies).
  - Drift detection frequency (how often resources deviate from template).
- **Logs:** CloudFormation events (resource creation, update, deletion events).
- **Alerts:** Stack creation failed, drift detected, stack update stuck (ROLLBACK_IN_PROGRESS).
- **Drift detection:** Run `detect-drift` regularly (CI job); alert if drift found (manual changes).

## Implementation notes
- **Checklist**
  - [ ] Use parameters for environment-specific values (instance type, DB password).
  - [ ] Set DeletionPolicy for critical resources (RDS, S3; prevent accidental deletion).
  - [ ] Use change sets for updates (preview changes before executing).
  - [ ] Enable termination protection for prod stacks (prevent accidental deletion).
  - [ ] Tag all resources (cost allocation, environment tracking).
  - [ ] Use outputs for inter-stack references (export VPC ID, import in another stack).
  - [ ] Schedule drift detection (daily CI job; alert on drift).

- **Security / Compliance notes**
  - Use IAM roles for CloudFormation (least-privilege; only necessary permissions).
  - Never hardcode secrets in templates (use Secrets Manager or Parameter Store).
  - Enable stack policy (prevent accidental updates/deletions of critical resources).
  - Audit stack changes (CloudTrail logs all CloudFormation API calls).

- **Performance notes**
  - Use nested stacks for large infrastructures (avoid template size limits).
  - Parallelize resource creation (CloudFormation does this automatically).

- **Operational notes**
  - Document stack dependencies (VPC stack before app stack).
  - Use StackSets for multi-account/multi-region deployments.
  - Test templates in dev environment before prod.

## Mini example (CloudFormation template)
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Simple web app infrastructure

Parameters:
  Environment:
    Type: String
    Default: dev
    AllowedValues:
      - dev
      - staging
      - prod

  InstanceType:
    Type: String
    Default: t2.micro
    Description: EC2 instance type

Resources:
  # VPC
  AppVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      Tags:
        - Key: Name
          Value: !Sub '${Environment}-vpc'

  # EC2 Instance
  WebServer:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-12345
      InstanceType: !Ref InstanceType
      SubnetId: !Ref AppSubnet
      Tags:
        - Key: Name
          Value: !Sub '${Environment}-web-server'

  # Subnet (depends on VPC)
  AppSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref AppVPC
      CidrBlock: 10.0.1.0/24
      Tags:
        - Key: Name
          Value: !Sub '${Environment}-subnet'

  # RDS (with DeletionPolicy to prevent accidental deletion)
  Database:
    Type: AWS::RDS::DBInstance
    DeletionPolicy: Snapshot  # Creates snapshot before deletion
    Properties:
      DBInstanceClass: db.t3.micro
      Engine: postgres
      AllocatedStorage: 20
      MasterUsername: admin
      MasterUserPassword: '{{resolve:secretsmanager:MyDBPassword}}'
      Tags:
        - Key: Name
          Value: !Sub '${Environment}-database'

Outputs:
  VPCId:
    Value: !Ref AppVPC
    Description: VPC ID
    Export:
      Name: !Sub '${Environment}-vpc-id'  # Can import in other stacks

  InstanceId:
    Value: !Ref WebServer
    Description: EC2 instance ID

# Create stack:
# aws cloudformation create-stack \
#   --stack-name dev-app-stack \
#   --template-body file://template.yaml \
#   --parameters ParameterKey=Environment,ParameterValue=dev

# Update stack (with change set):
# aws cloudformation create-change-set \
#   --stack-name dev-app-stack \
#   --change-set-name update-1 \
#   --template-body file://template.yaml
# aws cloudformation execute-change-set \
#   --change-set-name update-1 \
#   --stack-name dev-app-stack

# Delete stack:
# aws cloudformation delete-stack --stack-name dev-app-stack
```

## Common anti-patterns
- **Anti-pattern:** Hardcoding secrets in template.
  - **Why it's bad:** Secrets visible in CloudFormation console and API calls; security risk.
  - **Better approach:** Use Secrets Manager or Parameter Store; reference via `{{resolve:secretsmanager:...}}`.

- **Anti-pattern:** No DeletionPolicy for critical resources.
  - **Why it's bad:** Delete stack → RDS database deleted → data loss.
  - **Better approach:** Set `DeletionPolicy: Snapshot` for RDS, `DeletionPolicy: Retain` for S3.

- **Anti-pattern:** No change sets for updates.
  - **Why it's bad:** Unexpected changes (instance replaced instead of updated).
  - **Better approach:** Always create change set; review; then execute.

- **Anti-pattern:** Monolithic template (all resources in one template).
  - **Why it's bad:** Hard to maintain; template size limits; team conflicts.
  - **Better approach:** Use nested stacks or separate stacks per layer (VPC stack, app stack, database stack).

- **Anti-pattern:** No termination protection for prod stacks.
  - **Why it's bad:** Accidental `delete-stack` deletes entire production infrastructure.
  - **Better approach:** Enable termination protection; require manual disable before deletion.

## Interview readiness
### "Explain it like I'm teaching"
CloudFormation is AWS's Infrastructure as Code service. You write a YAML or JSON template describing AWS resources (EC2, RDS, VPC, etc.), and CloudFormation provisions them as a "stack." The workflow is: (1) write template, (2) create stack (CloudFormation provisions resources in correct order), (3) update stack (CloudFormation calculates diff and updates only what changed), (4) delete stack (CloudFormation deletes all resources). The key benefit is that AWS manages state for you—no state files to secure or share, unlike Terraform. CloudFormation also has automatic rollback: if stack creation fails, it deletes all resources automatically. The main downside is vendor lock-in (AWS-only; can't manage GCP or Azure) and verbose YAML syntax (less expressive than Terraform HCL or AWS CDK).

### Trap questions (with answers)
1) **Q:** How does CloudFormation manage state?
   - **A:** CloudFormation manages state internally in AWS. You don't manage state files (unlike Terraform). AWS tracks the stack's resources, their IDs, and dependencies. You can view stack state via AWS console or `describe-stacks` API. This simplifies collaboration (no S3 buckets or locking) but locks you into AWS.

2) **Q:** What is a change set, and why is it important?
   - **A:** A change set is a preview of what CloudFormation will change when you update a stack. It shows which resources will be created, updated, or deleted. It's critical because some updates replace resources (e.g., changing instance type deletes old instance, creates new one—downtime). Always review change sets before executing to avoid surprises.

3) **Q:** What happens if a resource creation fails during stack creation?
   - **A:** CloudFormation automatically rolls back: deletes all resources created so far and sets stack status to `ROLLBACK_COMPLETE`. This prevents partial stacks (half-created infrastructure). You can disable rollback (`--disable-rollback`) for debugging, but this leaves orphaned resources. Better: fix the issue and retry.

4) **Q:** How do you prevent accidental deletion of critical resources?
   - **A:** Use two mechanisms: (1) Set `DeletionPolicy: Retain` or `DeletionPolicy: Snapshot` on critical resources (RDS, S3); when stack is deleted, resource is retained or snapshotted. (2) Enable termination protection on the stack; must manually disable before deletion.

5) **Q:** What's the difference between CloudFormation and Terraform?
   - **A:** 
     - **CloudFormation:** AWS-only; JSON/YAML; AWS-managed state; free; automatic rollback.
     - **Terraform:** Multi-cloud; HCL; self-managed state (S3 + DynamoDB); open-source; no automatic rollback.
     - Choose CloudFormation for AWS-only with managed state; choose Terraform for multi-cloud or vendor neutrality.

### Quick self-check (5 items)
- [ ] I can explain how CloudFormation manages state (AWS-managed, no state files).
- [ ] I can describe the role of change sets (preview updates before executing).
- [ ] I can articulate when to use CloudFormation vs. Terraform.
- [ ] I can explain DeletionPolicy and termination protection.
- [ ] I can identify failure modes (rollback on error, drift from manual changes).

## Links (NO duplication)
### Prerequisites
- [Infrastructure as Code basics](infrastructure-as-code.md)

### Related topics
- [Terraform](terraform.md)
- [CI/CD basics](../quality/ci-cd-basics.md)

### Compare with
- [Terraform](terraform.md) — Terraform is multi-cloud; CloudFormation is AWS-only.

## Notes / Inbox
- Add examples from real projects: CloudFormation templates for Heimdall AWS infrastructure.
- Consider adding section on nested stacks and StackSets.
- Link to AWS CDK when created (higher-level abstraction over CloudFormation).
