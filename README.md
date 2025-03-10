# GitHub Actions Workflow for APIM Extraction and Publishing

This repository uses GitHub Actions to automate API Management (APIM) extraction and publishing workflows, following a structured branching strategy for deployment.

## Branching Strategy

1. **Feature Branch (`feature/branch`)**:
   - Branches are created from the `prod` branch for new features or fixes.
   - Pull Requests (PRs) to `dev` trigger automated code scans.

2. **Development (`dev`)**:
   - Code is merged into `dev` from feature branches after passing automated scans.
   - Deployment to the `dev` environment is automated upon merge.

3. **Quality Assurance (`qa`)**:
   - Code promoted to `qa` from `dev`.
   - Requires a successful deployment in `dev.1`.

4. **User Acceptance Testing (`uat`)**:
   - Code promoted to `uat` from `qa`.
   - Requires a successful deployment in `qa.1`.

5. **Performance Testing (`perf`)**:
   - Code promoted to `perf` from `qa`.
   - Requires a successful deployment in `qa.1`.

6. **Production (`prod`)**:
   - PR to `prod` requires:
     - 2 approvals from Admin/Lead Dev.
     - Successful deployments in both `qa.1` and `perf.1`.

## Workflows Overview

### Extractor Workflow

The **Extractor** workflow extracts APIs and artifacts from APIM environments. 

#### Trigger

- Automatically triggered via `workflow_call` or manually with `workflow_dispatch`.

#### Inputs
- `environment`: Target environment for extraction (default: `dev`).
- `USE_CONFIG_FILE`: Use a configuration file for extraction (default: `false`).
- `API_SPECIFICATION_FORMAT`: API specification format (default: `OpenAPIV3Yaml`).
- `CONFIGURATION_YAML_PATH`: Path to the configuration YAML file.

#### Key Steps
1. **Environment Setup**:
   - Installs PowerShell and Azure modules.
2. **Configuration**:
   - Configures environment variables based on inputs.
3. **API Extraction**:
   - Downloads and executes the APIM extractor.
4. **Artifact Handling**:
   - Clears existing artifacts and uploads new ones.
5. **Pull Request Creation**:
   - Automatically creates a PR with extracted artifacts.

### Publisher Workflow

The **Publisher** workflow publishes APIs and artifacts to APIM environments.

#### Trigger

- Triggered via `workflow_call` or `workflow_dispatch`.

#### Inputs
- `environment`: Target environment for publishing (default: `dev.1`).
- `environmentFrom`: Source environment for deployment.
- `instance`: Instance to publish to.
- `USE_CONFIG_FILE`: Use a configuration file for deployment (default: `false`).
- `DEPLOY_COMMIT_ONLY`: Deploy changes from a specific commit (default: `false`).
- `DEPLOY_FROM_FEATURE`: Deploy from a feature branch (default: `false`).
- `FEATURE_BRANCH_NAME`: Name of the feature branch (required if `DEPLOY_FROM_FEATURE` is true).

#### Key Steps
1. **Environment Setup**:
   - Installs PowerShell and Azure modules.
2. **Configuration**:
   - Sets environment variables and checks branch existence.
3. **Token Replacement**:
   - Replaces environment-specific tokens in configuration files.
4. **API Publishing**:
   - Downloads and executes the APIM publisher with the appropriate arguments.
5. **Tagging**:
   - Tags the branch with `-successful` or `-failed` based on deployment status.

## Usage Instructions

### Extractor Workflow

#### Manually Trigger
1. Navigate to the **Actions** tab.
2. Select the `APIM - Extract APIs and Artifacts` workflow.
3. Click **Run workflow** and provide the required inputs:
   - `environment`
   - `USE_CONFIG_FILE`
   - `API_SPECIFICATION_FORMAT`
   - `CONFIGURATION_YAML_PATH`

#### Automation
Use `workflow_call` in other workflows to trigger the extractor programmatically.

---

### Publisher Workflow

#### Manually Trigger
1. Navigate to the **Actions** tab.
2. Select the `APIM - Publish APIs and Artifacts` workflow.
3. Click **Run workflow** and provide the required inputs:
   - `environment`
   - `environmentFrom`
   - `instance`
   - `USE_CONFIG_FILE`
   - `DEPLOY_FROM_FEATURE`
   - `FEATURE_BRANCH_NAME`

#### Automation
Use `workflow_call` in other workflows to trigger the publisher programmatically.

---

## Deployment Security

1. **Approval for Production**:
   - PR to `prod` requires:
     - Two approvals from Admin/Lead Dev.
     - Successful deployments in `qa.1` and `perf.1`.

2. **Tagging**:
   - Successful deployments are tagged with `<environment>-successful`.
   - Failed deployments are tagged with `<environment>-failed`.

## Repository Requirements

- **Secrets**:
  - `AZURE_CLIENT_ID`, `AZURE_CLIENT_SECRET`, `AZURE_TENANT_ID`, `AZURE_SUBSCRIPTION_ID`.
- **Variables**:
  - `AZURE_RESOURCE_GROUP_NAME`, `API_MANAGEMENT_SERVICE_NAME`.

## Contributing

1. Branch/Fork the repository.
2. Create a branch (`feature/<name>`).
3. Open a PR to `dev.1`.
4. Ensure all checks pass before requesting a review.

For further assistance, refer to the documentation or contact the Admin team.
