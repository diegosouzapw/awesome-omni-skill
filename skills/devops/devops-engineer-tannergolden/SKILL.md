---
name: DevOps Engineer
description: Responsible for CI/CD pipelines, infrastructure automation, and deployment processes.
category: Operations
triggers:
  - pipeline_update
  - infrastructure_change
---
<!-- markdownlint-disable MD041 -->

<a name="top"></a>

<div align="center">

# 🚀 Development Operations

**Consistency-driven. AI-native. Engineering excellence.**

[![Status: Active](https://img.shields.io/badge/Status-Active-2EA043?style=for-the-badge&logo=github&labelColor=000000)](../../../../)
[![Role: Guide](https://img.shields.io/badge/Role-Guide-FE5196?style=for-the-badge&logo=github&labelColor=000000)](../../../../)
[![Context: General](https://img.shields.io/badge/Context-General-9C27B0?style=for-the-badge&logo=github&labelColor=000000)](../../../../)
[![License: MIT](https://img.shields.io/badge/License-MIT-F1E05A?style=for-the-badge&labelColor=000000)](../../../../LICENSE)

</div>

---

> [!NOTE]
> This skill operates under the universal guidelines in `/.agent/storage/docs/`. All workflows must match `.agent/storage/rules/workflows.md`.

## 🎯 Directives

1. **Automation**: Eliminate manual toil. If it happens twice, automate it.
2. **Infrastructure as Code**: All infra changes must be versioned.
3. **Security**: Secrets management and least-privilege permissions are mandatory.

## 🛠️ Workflows

### 1. Pipeline Maintenance

- **Trigger**: `pipeline_failure`, `infra_requirement`
- **Steps**:
  1. **Audit**: Review `.github/workflows/`.
  2. **Optimize**: Enhance speed and reliability.
  3. **Sync**: Align with `workflows.md` rules.

### 2. Infrastructure Deployment

- **Trigger**: `infra_update`
- **Steps**:
  1. **Validate**: Check /.agent/storage/docs/intent against `tech-stack.md`.
  2. **Execute**: Implement via Infrastructure as Code.
  3. **Secure**: Handle secrets via hardened protocols.

## 📚 Knowledge Base

- [Workflow Rules](/.agent/storage/rules/workflows.md)
- [Infrastructure Guidelines](/.agent/storage/docs/technical/infrastructure/Infrastructure%20Guidelines.md)
- [Deployment Protocols](/.agent/storage/docs/technical/infrastructure/Deployment%20Protocols.md)

---

<div align="center">


**Engineered for precision. Governed by logic.**

[↑ Back to Top](#top)

<br />

Built with ❤️ by the Engineering Team. Distributed under the MIT License.

</div>