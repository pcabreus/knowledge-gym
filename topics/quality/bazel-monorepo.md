---
title: Bazel Monorepo
---

# Bazel Monorepo

## Definition
A Bazel monorepo is a single repository that uses Bazel as its build system to manage, build, and test multiple projects or services together.

## Why it matters
It enables reproducible builds, strong dependency boundaries, and scalable CI/CD for large codebases.

## When to use / when not to use
Use for large organizations with many interdependent projects. Not ideal for small teams or simple projects.

## Trade-offs
- + Fast, incremental builds
- + Consistent developer experience
- - Steep learning curve
- - Requires careful rule maintenance

## Prerequisites
- [CI/CD basics](../quality/ci-cd-basics.md)

## Related topics
- [GitHub Actions](github-actions.md)
- [Docker](../operations/docker.md)

## Trap questions
1. What is a key benefit of using Bazel in a monorepo?
   - Fast, reproducible builds across projects.
2. What is a common challenge with Bazel?
   - Complex rule definitions and maintenance.
3. When should you avoid a Bazel monorepo?
   - For small, simple projects where the overhead is not justified.
