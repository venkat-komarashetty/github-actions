# GitHub Actions

This repository is a hands-on collection of GitHub Actions workflow examples for learning and experimenting with CI/CD automation on GitHub.

It includes examples for common GitHub Actions patterns such as:

- basic workflow triggers (`push`, `workflow_dispatch`)
- pull request workflows
- reusable workflows
- job and step outputs
- conditional execution with `if`
- workflow inputs and environment variables
- uploading and using artifacts
- action checkout examples

## Repository structure

```text
.github/
  workflows/
    action-checkout.yml
    calling-reusable-workflow.yml
    env-demo.yml
    hello-world.yml
    if-demo-steps.yml
    inputs-demo.yml
    multiple-jobs-workflow.yml
    output-demo-sharing-jobs.yml
    output-demo-steps-sharing.yml
    pull-request-workflow.yml
    push-workflow.yml
    reusable-workflow.yml
    sequential-jobs.yml
    upload-artifact.yml
index.html
README.md
```

## Key workflow examples

### Hello world
- `hello-world.yml` demonstrates a basic manual workflow triggered with `workflow_dispatch`.

### Push and pull request triggers
- `push-workflow.yml` runs on `push`
- `pull-request-workflow.yml` runs on pull request events

### Reusable workflows
- `reusable-workflow.yml` defines a reusable workflow that accepts an input.
- `calling-reusable-workflow.yml` shows how to call it from another workflow.

### Outputs and dependencies
- `output-demo-steps-sharing.yml` demonstrates output sharing between steps
- `output-demo-sharing-jobs.yml` demonstrates output sharing between jobs
- `multiple-jobs-workflow.yml` and `sequential-jobs.yml` show job ordering and dependencies

### Conditions and inputs
- `if-demo-steps.yml` shows conditional step execution
- `inputs-demo.yml` demonstrates workflow inputs
- `env-demo.yml` demonstrates environment variables

### Artifacts
- `upload-artifact.yml` uploads a file as a build artifact
- `artifact.txt` is used as the uploaded artifact content

## How to use this repo

1. Clone the repo.
2. Open the GitHub repository in the browser.
3. Go to the **Actions** tab.
4. Select a workflow and run it manually or trigger it with a push/PR event.

Example:

```bash
git clone https://github.com/venkat-komarashetty/github-actions.git
cd github-actions
```

## Notes

This repository is designed as a learning reference for GitHub Actions syntax and workflow behavior. It is not a production application and does not require a build or runtime setup.

## License

This project does not currently declare a license.
