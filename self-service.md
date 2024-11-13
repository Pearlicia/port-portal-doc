---

### 1. **Create GitHub Secrets**

Set up the necessary GitHub secrets for authenticating the workflow trigger.

#### Required Secrets:

1. **PERSONAL_ACCESS_TOKEN**: Generate a Personal Access Token (PAT) with the following scopes:
   - **repo** (for repository access)
   - **workflow** (to trigger workflows)

2. **PORT_CLIENT_ID** and **PORT_CLIENT_SECRET**: These are specific to Port and are used to authenticate requests from Port to GitHub.

---

### 2. **Define a Port Blueprint**

Define a basic Port blueprint to identify the workflow you wish to trigger. This blueprint is minimal and only includes the workflow name.

#### Example Blueprint Definition:

```json
{
  "identifier": "triggerworkflow",
  "title": "TriggerWorkflow",
  "icon": "Github",
  "schema": {
    "properties": {
      "workflow_name": {
        "icon": "Github",
        "title": "Workflow Name",
        "type": "string",
        "description": "The name of the workflow to trigger"
      }
    },
    "required": ["workflow_name"]
  }
}
```

---

### 3. **Create a Port Action**

Create a Port Action that triggers the specified GitHub workflow file. Replace `<GITHUB_ORG_NAME>` and `<GITHUB_REPO_NAME>` with the organization and repository where the workflow resides.

#### Port Action JSON Definition:

```json
{
  "identifier": "trigger_github_workflow",
  "title": "Trigger GitHub Workflow",
  "icon": "Github",
  "description": "Triggers a GitHub workflow in the repository",
  "trigger": {
    "type": "self-service",
    "operation": "CREATE",
    "userInputs": {
      "properties": {
        "workflow_name": {
          "icon": "DefaultProperty",
          "title": "Workflow Name",
          "type": "string"
        }
      },
      "required": ["workflow_name"],
      "order": ["workflow_name"]
    },
    "blueprintIdentifier": "triggerworkflow"
  },
  "invocationMethod": {
    "type": "GITHUB",
    "org": "<GITHUB_ORG_NAME>",
    "repo": "<GITHUB_REPO_NAME>",
    "workflow": "{{ .inputs.\"workflow_name\" }}",
    "reportWorkflowStatus": true
  },
  "requiredApproval": false
}
```

- **`workflow_name`** is the name of the workflow file you want to trigger (e.g., `run-workflow.yml`).
- **`reportWorkflowStatus`** allows Port to report the workflow status back to the Port dashboard.

---

### 4. **Create the GitHub Workflow File**

Place a GitHub workflow file in `.github/workflows/<workflow-name>.yml` with only the basic structure, as no extra input parameters or steps are needed.

#### Workflow File (`run-workflow.yml`):

```yaml
name: Trigger Workflow

on:
  workflow_dispatch:

jobs:
  auto_run:
    runs-on: ubuntu-latest
    steps:
      - name: Confirm Workflow Run
        run: echo "The workflow has been triggered successfully."
```


---

### 5. **Trigger the Action**

To execute the action:
1. Open the **Self-service** tab in Port.
2. Select **Trigger GitHub Workflow**.
3. Enter the name of the workflow file (e.g., `run-workflow.yml`) as the `workflow_name`.
4. Submit the form, and the GitHub workflow will run automatically.

---

