# GitHub Actions

This repository is a hands-on collection of small GitHub Actions workflow examples. It demonstrates how workflows respond to repository events, run jobs on GitHub-hosted runners, and execute shell commands or actions. The examples are for learning, not a production CI/CD pipeline.

## GitHub Actions building blocks

- **Workflow:** An automated process defined in a YAML file under `.github/workflows/`. The `on` key lists the event or events that start it; `jobs` defines the work.
- **Job:** A group of steps that runs on a runner specified by `runs-on`. Jobs run in parallel by default. Use `needs` to make one job wait for another.
- **Step:** An individual task within a job. A step can run a shell command with `run` or invoke an action with `uses`. Steps in a job run in order.
- **Action:** A reusable unit of work invoked by a workflow step. This repository uses `actions/checkout@v4` and `actions/upload-artifact@v4`. Reusable workflows are called as jobs.
- **Runner:** The machine that executes a job. The examples use GitHub-hosted `ubuntu-slim` and `ubuntu-latest` runners.

## Example: workflow, job, steps, and action

This is the complete [`action-checkout.yml`](.github/workflows/action-checkout.yml) example:

```yaml
name: Github actions workflow
on: workflow_dispatch
jobs:
  action-demo:
    runs-on: ubuntu-slim
    steps:
      - name: Checkout the code
        uses: actions/checkout@v4
      - name: Print index.html file content
        run: cat index.html
```

- The **workflow** is this YAML file. Its `on: workflow_dispatch` trigger lets you start it manually from the Actions tab.
- The **job** is `action-demo`; `runs-on` selects the `ubuntu-slim` runner for its work.
- The **steps** run in order: first check out the repository, then print `index.html`.
- The first step invokes the **action** `actions/checkout@v4` through `uses`. The next step uses `run` to execute a shell command; it is a command, not an action.

For another job example, [`sequential-jobs.yml`](.github/workflows/sequential-jobs.yml) defines three jobs and uses `needs` to make `job-2` wait for `job-1` and `job-3` wait for `job-2`. By contrast, the jobs in [`multiple-jobs-workflow.yml`](.github/workflows/multiple-jobs-workflow.yml) have no dependencies and can run in parallel.

## Workflow-by-workflow guide

Each YAML file below demonstrates a particular feature. The snippets show the relevant syntax; the complete workflows are linked by filename.

### [`action-checkout.yml`](.github/workflows/action-checkout.yml)

This manually triggered workflow uses `actions/checkout@v4` to make repository files available on the runner, then uses a shell step to print `index.html`. The complete workflow is shown in the example above. The key step syntax is:

```yaml
- uses: actions/checkout@v4
- run: cat index.html
```

`uses` invokes an action; `run` executes a shell command. These are two different kinds of steps.

### [`calling-reusable-workflow.yml`](.github/workflows/calling-reusable-workflow.yml)

This workflow runs on `push` and calls another workflow as a job. `jobs.<job_id>.uses` points to the local reusable workflow, and `with` passes its input:

```yaml
on: push
jobs:
  call-reusable-workflow:
    uses: ./.github/workflows/reusable-workflow.yml
    with:
      fname: "Venkat"
```

This is a reusable-workflow call, not an action step. The called workflow must declare `on: workflow_call`, as shown in [`reusable-workflow.yml`](.github/workflows/reusable-workflow.yml).

### [`env-demo.yml`](.github/workflows/env-demo.yml)

The `env` key sets environment variables at workflow, job, or step scope. More specific scopes override broader ones: the step prints `prod`, overriding the job's `test` and workflow's `dev`.

```yaml
env:
  APP_ENV: "dev"
jobs:
  env-demo:
    env:
      APP_ENV: "test"
    steps:
      - env:
          APP_ENV: "prod"
        run: echo "The env variable is $APP_ENV"
```

### [`hello-world.yml`](.github/workflows/hello-world.yml)

This workflow uses `workflow_dispatch` for a manual run and a multiline `run` block for shell commands. It reads a repository configuration variable from the `vars` context:

```yaml
on: workflow_dispatch
# In a step's run block:
run: |
  echo "I am ${{ vars.MY_NAME }}"
```

Set the repository variable `MY_NAME` for the echo to include a value. This workflow does not use a secret.

### [`if-demo-steps.yml`](.github/workflows/if-demo-steps.yml)

The `push.branches` filter starts this workflow only for pushes to `main` or `develop`. A step-level `if` expression then runs the first step only on `main`:

```yaml
on:
  push:
    branches: [main, develop]
jobs:
  if-demo:
    steps:
      - if: github.ref == 'refs/heads/main'
        run: echo "This step runs only on the main branch"
      - run: echo "This step has no branch condition"
```

GitHub Actions uses `if` expressions to conditionally run a job or step; there is no general `else` clause here. The second step is not an `else`: it has no branch condition and runs under the normal step status rules (for example, it is skipped if an earlier step fails or the job is cancelled).

### [`inputs-demo.yml`](.github/workflows/inputs-demo.yml)

`workflow_dispatch.inputs` defines values requested when a user manually starts the workflow. Here, `env` is required, defaults to `dev`, and is restricted to three choices; the step reads it through the `inputs` context.

```yaml
on:
  workflow_dispatch:
    inputs:
      env:
        required: true
        default: "dev"
        type: choice
        options: [dev, test, prod]
# In a step:
run: echo "The input variable is ${{ inputs.env }}"
```

### [`multiple-jobs-workflow.yml`](.github/workflows/multiple-jobs-workflow.yml)

This manually triggered workflow defines three independent jobs. Since none declares `needs`, GitHub Actions can run them in parallel. Each job chooses its runner with `runs-on`; this file uses both `ubuntu-slim` and `ubuntu-latest`.

```yaml
jobs:
  job-1:
    runs-on: ubuntu-slim
  job-2:
    runs-on: ubuntu-latest
```

Each job also contains its own steps, which execute in order within that job.

### [`output-demo-sharing-jobs.yml`](.github/workflows/output-demo-sharing-jobs.yml)

This `push` workflow passes a value from one job to another. A step writes `version` to `$GITHUB_OUTPUT`; the `build` job exposes that step output under `outputs`; `deploy` waits for `build` with `needs` and reads the job output.

```yaml
jobs:
  build:
    outputs:
      version: ${{ steps.step1.outputs.version }}
    steps:
      - id: step1
        run: echo "version=0.0.1" >> "$GITHUB_OUTPUT"
  deploy:
    needs: build
    steps:
      - run: echo "${{ needs.build.outputs.version }}"
```

### [`output-demo-steps-sharing.yml`](.github/workflows/output-demo-steps-sharing.yml)

This `push` workflow shares an output between two steps in the same job. Give the producing step an `id`, write a `name=value` line to `$GITHUB_OUTPUT`, and access it through `steps.<id>.outputs.<name>` in a later step.

```yaml
steps:
  - id: step1
    run: echo "version=0.0.1" >> "$GITHUB_OUTPUT"
  - run: echo "The version is ${{ steps.step1.outputs.version }}"
```

### [`pull-request-workflow.yml`](.github/workflows/pull-request-workflow.yml)

`on: pull_request` starts this workflow for pull request events. It has one job on `ubuntu-slim` with a shell step.

```yaml
on: pull_request
jobs:
  pull-request-workflow:
    runs-on: ubuntu-slim
```

### [`push-workflow.yml`](.github/workflows/push-workflow.yml)

`on: push` starts the workflow when a push event occurs. Its job runs a multiline shell script, written using YAML's `|` block syntax.

```yaml
name: Push Workflow
on: push
jobs:
  push-workflow:
    runs-on: ubuntu-slim
    steps:
      - name: Push Workflow Step
        run: |
          echo "Welcome to my first Push Workflow GitHub action workflow"
          echo "This is line two"
```

### [`reusable-workflow.yml`](.github/workflows/reusable-workflow.yml)

To make a workflow callable by another workflow, declare `on: workflow_call`. Inputs are declared under `workflow_call.inputs` with a type; this example requires a string named `fname`. Within the called workflow, read it using `${{ inputs.fname }}`.

```yaml
on:
  workflow_call:
    inputs:
      fname:
        required: true
        type: string
jobs:
  greetings:
    runs-on: ubuntu-slim
    steps:
      - run: echo "Welcome ${{ inputs.fname }}"
```

The caller supplies the input with `with`, as shown in [`calling-reusable-workflow.yml`](.github/workflows/calling-reusable-workflow.yml).

### [`sequential-jobs.yml`](.github/workflows/sequential-jobs.yml)

This manually triggered workflow uses `needs` to impose a job order. `job-2` waits for `job-1`, and `job-3` waits for `job-2`, forming a chain rather than running all jobs in parallel.

```yaml
jobs:
  job-1:
    runs-on: ubuntu-slim
  job-2:
    needs: job-1
    runs-on: ubuntu-latest
  job-3:
    needs: job-2
    runs-on: ubuntu-slim
```

### [`upload-artifact.yml`](.github/workflows/upload-artifact.yml)

This manually triggered workflow invokes `actions/upload-artifact@v4`. The `with` block configures the artifact name, file path, and retention in days:

```yaml
- uses: actions/upload-artifact@v4
  with:
    name: my-first-artifact
    path: artifact.txt
    retention-days: 10
```

See the artifact path note below: the workflow's configured path does not currently match the sample file location.

## Running the examples

1. Open the repository on GitHub and select **Actions**.
2. Select a workflow. For workflows triggered by `workflow_dispatch`, choose **Run workflow** and provide any requested inputs.
3. The `push` and `pull_request` examples run when their corresponding repository events occur.

Some workflows only print messages. The checkout example reads a repository file; it must check out the repository before accessing files. The reusable workflow is intended to be called by another workflow rather than run directly.

## Artifact example note

[`upload-artifact.yml`](.github/workflows/upload-artifact.yml) refers to `artifact.txt` at the workspace root but does not check out the repository first. The sample file is currently at `.github/workflows/artifact.txt`, so the configured upload path does not match the repository layout. The workflow demonstrates the upload-artifact configuration, but the file must be checked out and the path aligned for an upload to succeed.

## Repository layout

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
    artifact.txt
index.html
README.md
```

No secrets are currently referenced by these workflows.
