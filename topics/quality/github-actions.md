---
title: GitHub Actions
---

# GitHub Actions

## Definition
GitHub Actions is a CI/CD platform that allows you to automate, build, test, and deploy code directly from your GitHub repository using workflows defined as YAML files.

## Why it matters
It enables automation of software delivery processes, improves code quality, and supports rapid iteration.

## When to use / when not to use
Use for automating builds, tests, deployments, and other workflows in projects hosted on GitHub. Not ideal for very large, complex pipelines that require advanced orchestration.

## Trade-offs
- + Easy integration with GitHub
- + Large ecosystem of actions
- - Can become complex for large workflows
- - Secrets management requires care

## Prerequisites
- [CI/CD basics](../quality/ci-cd-basics.md)

## Related topics
- [Bazel Monorepo](bazel-monorepo.md)
- [Docker](../operations/docker.md)

## Trap questions
1. What is a GitHub Action?
   - A reusable automation step in a workflow.
2. What is a common security risk with GitHub Actions?
   - Exposing secrets in logs or to untrusted code.
3. When should you avoid using GitHub Actions?
   - For workflows requiring advanced orchestration beyond what GitHub provides.
