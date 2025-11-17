# Jules Invoke GitHub Action

[![Built with Jules](https://img.shields.io/badge/Built%20with-Jules-715cd7?link=https://jules.google)](https://jules.google)

This GitHub Action allows you to invoke [Jules](https://jules.google), an AI-powered coding agent, to perform tasks on your codebase.

## Prerequisites

Before using this action, you must have:

1.  **Authenticated with GitHub at [jules.google.com](https://jules.google.com).**
2.  **A Jules API Key:** Generate an API key from your Jules account settings. For more information, see the [Jules API documentation](https://developers.google.com/jules/api).
3.  **Stored the API key as a secret:** You will need to add this as a secret in your GitHub repository. For more information, see the [GitHub documentation on using secrets in GitHub Actions](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions).

## Inputs

| Input                 | Description                                                                                             | Default   |
| --------------------- | ------------------------------------------------------------------------------------------------------- | --------- |
| `prompt`              | **Required.** The prompt to pass to Jules.                                                              | `''`      |
| `jules_api_key`       | **Required.** Your Jules API key. Store this as a secret in your repository settings.                     | `''`      |
| `include_last_commit` | Whether to pass the content of the last commit to Jules.                                                | `false`   |
| `include_commit_log`  | Whether to pass the commit history to Jules.                                                            | `false`   |
| `starting_branch`     | The branch for Jules to start from.                                                                     | `'main'`  |

## Usage

Here's an example of how to use the Jules Invoke action in your workflow:

```yaml
name: Invoke Jules

on:
  workflow_dispatch:
    inputs:
      prompt:
        description: 'The prompt for Jules'
        required: true

jobs:
  jules-invoke:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - name: Invoke Jules
        uses: BeksOmega/jules-action@v1
        with:
          prompt: ${{ github.event.inputs.prompt }}
          jules_api_key: ${{ secrets.JULES_API_KEY }}
```

**Note on permissions:** This action requires the `contents: read` permission to be able to check out the repository. This is especially important for private repositories.

### Running on unblocked issues

This triggers a Jules task with the contents of an issue whenever an issue becomes unblocked because another issue is closed. It uses the [find unblocked issues action](https://github.com/marketplace/actions/find-unblocked-issues).

```
name: Execute Unblocked Issues

on:
  issues:
    types: [closed]

jobs:
  detect-unblocked:
    runs-on: ubuntu-latest
    permissions:
      issues: read
    outputs:
      matrix: ${{ steps.finder.outputs.issues }}
      found: ${{ steps.finder.outputs.has_issues }}
    steps:
      - name: Find Unblocked Issues
        id: finder
        uses: BeksOmega/on-unblocked@v1.0.0

  invoke-jules:
    needs: detect-unblocked
    if: needs.detect-unblocked.outputs.found == 'true'
    runs-on: ubuntu-latest
    permissions:
      contents: read
      issues: read
    strategy:
      matrix:
        issue-number: ${{ fromJson(needs.detect-unblocked.outputs.matrix) }}
    steps:
      - name: Get issue details
        id: get_issue
        uses: actions/github-script@v6
        with:
          script: |
            const issue = await github.rest.issues.get({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: ${{ matrix.issue-number }}
            });
            return issue.data;
      - name: Invoke Jules on Unblocked Issue
        uses: BeksOmega/jules-action@v1.0.0
        with:
          prompt: "${{ fromJson(steps.get_issue.outputs.result).title }}. ${{ fromJson(steps.get_issue.outputs.result).body }}"
          jules_api_key: ${{ secrets.JULES_API_KEY }}
```

### Running periodically

You can also run workflows on a cron, such as to do general code clean-up. (Note: This prompt is just an example, it hasn't been thoroughly tested)

```
name: Cleanup

on:
  workflow_dispatch:
  schedule:
    - cron: '0 0 * * *'

jobs:
  cleanup:
    runs-on: ubuntu-latest
    steps:
      - name: Invoke Jules for Cleanup
        uses: BeksOmega/jules-action@v1.0.0
        with:
          prompt: |
            Please analyze the codebase for potential areas of improvement. Specifically, I'd like you to focus on the following code smells:
            1.  **Dead Code:** Identify any unused variables, functions, or classes that can be safely removed.
            2.  **Code Duplication:** Look for blocks of identical or very similar code that could be refactored into a single, reusable function or module.
            3.  **Overly Complex Code:** Pinpoint functions or classes that are excessively long or have a high cyclomatic complexity. Suggest ways to simplify them by breaking them down into smaller, more manageable pieces.
            4.  **Poor Naming:** Find variables, functions, or classes with names that are unclear, misleading, or do not follow a consistent convention. Propose more descriptive and conventional names.
            5.  **Inconsistent Formatting:** Check for areas where the code formatting deviates from the established style guide (if one exists) or from general best practices.
            For each issue you identify, please provide a fix.
          jules_api_key: ${{ secrets.JULES_API_KEY }}
```

## Attribution

If you find this action useful, spread the word by adding the "Built with Jules" shield to your `README.md`:

```markdown
[![Built with Jules](https://img.shields.io/badge/Built%20with-Jules-715cd7?link=https://jules.google)](https://jules.google)
```

## Learn More

For more information on Jules and how to get the most out of it, please visit the [Jules documentation](https://jules.google/docs).
