# GCP Secrets Disabler Action

This GitHub Action disables old versions of secrets in Google Cloud Secret Manager. It's designed to help maintain clean and organized secret management by disabling outdated secret versions while keeping the latest version enabled.

The action is based on the [GCP Secrets Disabler](https://github.com/muehlemann-popp/gcp-secrets-disabler) script, which can also be used standalone. For detailed information about the script's functionality and local usage, please refer to the script repository's README.

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `gcp-credentials` | Path to Google Cloud Service Account JSON key | Yes | - |
| `project-id` | Google Cloud Project ID | Yes | - |
| `gcp-region` | Google Cloud Region for operations | Yes | - |
| `python-version` | Python version to use | No | `3.12` |
| `dry-run` | If true, only show what would be disabled without making changes | Yes | `true` |
| `save-data` | If true, save requests to external resources as pickled binaries | No | `false` |

## Internal Environment Variables

The action sets these environment variables internally based on the inputs:

- `GCP_PROJECT_ID`: Set from the `project-id` input
- `GCP_REGION`: Set from the `gcp-region` input
- `DRY_RUN`: Set from the `dry-run` input
- `SAVE_DATA`: Set from the `save-data` input
- `FILE_MODE`: Always set to "false" in action context (this is a debug flag for local development)
- `GOOGLE_GHA_CREDS_PATH`: Set automatically by google-github-actions/auth

When `save-data` is enabled, the script will save data to the following files:
- `data_secrets.pkl`: Contains retrieved secrets data
- `data_secrets_versions.pkl`: Contains retrieved secret versions data

These files are saved in the `var` directory within the script's working directory.

## Usage Example

```yaml
name: Disable Old Secret Versions
on:
  schedule:
    - cron: '0 0 * * *'  # Run daily at midnight
  workflow_dispatch:      # Allow manual triggering

jobs:
  disable-old-secrets:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Disable old secret versions
        uses: muehlemann-popp/gcp-secrets-disabler-action@v1.0.0
        with:
          gcp-credentials: ${{ secrets.GCP_SA_KEY }}
          project-id: 'your-gcp-project-id'
          gcp-region: 'europe-west6'
          python-version: '3.12'  # Optional, this is the default
          dry-run: true  # Set to false to actually disable old versions
          save-data: false

```

## Security Notes

1. Make sure to store your GCP credentials securely as a GitHub secret.
2. Always test with `dry-run: true` first to verify which secret versions would be affected.
3. The service account used should have the minimum required permissions to manage secrets in Secret Manager.

## Required GCP Permissions

The service account used needs the following IAM roles or equivalent permissions:

- `roles/secretmanager.viewer` - To list secrets and their versions
- `roles/secretmanager.secretVersionManager` - To disable secret versions

## Python Version

The action uses Python 3.12 by default, which matches the minimum version requirement of the underlying script. While you can specify a different Python version using the `python-version` input, it's recommended to stick with 3.12 unless you have specific requirements.

## How It Works

1. Authenticates to Google Cloud using provided credentials
2. Lists all secrets in the specified project
3. For each secret, retrieves all versions
4. Identifies the most recent version of each secret
5. When `dry-run` is false, disables all older enabled versions while keeping the latest version enabled

For more detailed information about the underlying script and its functionality, please refer to the [GCP Secrets Disabler](https://github.com/muehlemann-popp/gcp-secrets-disabler) repository.