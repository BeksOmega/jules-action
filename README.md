# Jules Invoke GitHub Action

[![Built with Jules](https://img.shields.io/badge/Built%20with-Jules-715cd7?link=https://jules.google)](https://jules.google)

This GitHub Action allows you to invoke [Jules](https://jules.google), an AI-powered coding agent, to perform tasks on your codebase.

## Prerequisites

Before using this action, you must have:

1.  **Authenticated with GitHub at [jules.google.com](https://jules.google.com).**
2.  **A Jules API Key:** Generate an API key from your Jules account settings. For more information, see the [Jules API documentation](https://developers.google.com/jules/api).
3.  **Stored the API key as a secret:** You will need to add this as a secret in your GitHub repository. For more information, see the [GitHub documentation on using secrets in GitHub Actions](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions).

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
    steps:
      - name: Invoke Jules
        uses: BeksOmega/jules-action@v1
        with:
          prompt: ${{ github.event.inputs.prompt }}
          jules_api_key: ${{ secrets.JULES_API_KEY }}
```

## Inputs

| Input                 | Description                                                                                             | Default   |
| --------------------- | ------------------------------------------------------------------------------------------------------- | --------- |
| `prompt`              | **Required.** The prompt to pass to Jules.                                                              | `''`      |
| `jules_api_key`       | **Required.** Your Jules API key. Store this as a secret in your repository settings.                     | `''`      |
| `github_token`        | The GitHub token to use for authentication. Required for private repositories.                          | `${{ github.token }}` |
| `include_last_commit` | Whether to pass the content of the last commit to Jules.                                                | `false`   |
| `include_commit_log`  | Whether to pass the commit history to Jules.                                                            | `false`   |
| `starting_branch`     | The branch for Jules to start from.                                                                     | `'main'`  |

## Attribution

If you find this action useful, spread the word by adding the "Built with Jules" shield to your `README.md`:

```markdown
[![Built with Jules](https://img.shields.io/badge/Built%20with-Jules-715cd7?link=https://jules.google)](https://jules.google)
```

## Learn More

For more information on Jules and how to get the most out of it, please visit the [Jules documentation](https://jules.google/docs).
