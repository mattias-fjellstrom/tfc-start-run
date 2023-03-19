# GitHub Actions for starting a run in a Terraform Cloud workspace

With this action you can start a new run in a Terraform Cloud workspace as part of your GitHub Actions workflows.

- This action should be preceded by the [mattias-fjellstrom/tfc-setup](https://github.com/mattias-fjellstrom/tfc-setup/blob/main/action.yaml) action to configure required environment variables. See the sample below.
- You have to specify `organization` and `workspace` as input to this action, either as `with:` parameters or through environment variables using the [mattias-fjellstrom/tfc-setup](https://github.com/mattias-fjellstrom/tfc-setup/blob/main/action.yaml) action.

## Sample workflow

Below is a full sample workflow that sets up a new workspace in Terraform Cloud when a pull-request is opened and initiates a first run, and deletes the workspace once the pull-request is closed. If the pull request is opened for a branch named `feature-1` the resulting workspace will be named `my-workspace-feature-1` in this sample.

```yaml
name: Terraform Cloud workspaces for pull-requests

on:
  pull_request:
    types:
      - opened
      - closed

env:
  ORGANIZATION: my-terraform-cloud-organization
  PROJECT: my-terraform-cloud-project
  WORKSPACE: my-workspace-${{ github.head_ref }}

jobs:
  create-workspace:
    if: ${{ github.event.action == 'opened' }}
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: mattias-fjellstrom/tfc-setup@v1
        with:
          token: ${{ secrets.TERRAFORM_CLOUD_TOKEN }}
          organization: ${{ env.ORGANIZATION }}
          project: ${{ env.PROJECT }}
          workspace: ${{ env.WORKSPACE }}
      - uses: mattias-fjellstrom/tfc-create-workspace@v1
        with:
          directory: infrastructure
          branch: ${{ github.head_ref }}
      - uses: mattias-fjellstrom/tfc-start-run@v1
  delete-workspace:
    if: ${{ github.event.action == 'closed' }}
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: mattias-fjellstrom/tfc-setup@v1
        with:
          token: ${{ secrets.TERRAFORM_CLOUD_TOKEN }}
          organization: ${{ env.ORGANIZATION }}
          project: ${{ env.PROJECT }}
          workspace: ${{ env.WORKSPACE }}
      - uses: mattias-fjellstrom/tfc-delete-workspace@v1
```