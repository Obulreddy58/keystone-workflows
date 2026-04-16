# Keystone Workflows

Reusable GitHub Actions workflows for Terraform/Terragrunt CI/CD pipelines. Used across all Keystone infrastructure repos.

## Workflows

| Workflow | Trigger | Description |
|----------|---------|-------------|
| `terragrunt-plan.yaml` | PR opened/updated | Runs `terragrunt plan` on changed modules, posts plan output to PR |
| `terragrunt-apply.yaml` | PR merged to main | Runs `terragrunt apply` on changed modules |
| `terragrunt-destroy.yaml` | Manual dispatch | Destroys infrastructure for a specific module path |
| `pull-request.yaml` | PR events | Validates Terraform format, runs tfsec/checkov security scans |
| `deploy.yaml` | Push to main | Builds and deploys the Keystone platform (Docker image) |
| `destroy.yaml` | Manual dispatch | Tears down platform infrastructure |

## How Teams Use These

Teams reference these workflows in their infra repos:

```yaml
# .github/workflows/terraform.yaml in any team repo
name: Terraform
on:
  pull_request:
    paths: ['**/*.hcl']

jobs:
  plan:
    uses: Obulreddy58/keystone-workflows/.github/workflows/terragrunt-plan.yaml@main
    with:
      working-directory: ${{ github.event.pull_request.head.ref }}
    secrets:
      AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
```

## Authentication

All workflows use **OIDC keyless authentication** — no static AWS credentials:

```yaml
permissions:
  id-token: write   # Required for OIDC
  contents: read

steps:
  - uses: aws-actions/configure-aws-credentials@v4
    with:
      role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
      aws-region: us-east-1
```

## Related Repos

| Repo | Description |
|------|-------------|
| [keystone-modules](https://github.com/Obulreddy58/keystone-modules) | Terraform modules referenced by Terragrunt |
| [keystone-infra-live](https://github.com/Obulreddy58/keystone-infra-live) | Live Terragrunt configurations that trigger these workflows |
