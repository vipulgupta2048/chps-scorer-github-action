# CHPs Scorer GitHub Action

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-CHPs%20Scorer-blue.svg?colorA=24292e&colorB=0366d6&style=flat&longCache=true&logo=data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAA4AAAAOCAYAAAAfSC3RAAAABHNCSVQICAgIfAhkiAAAAAlwSFlzAAAM6wAADOsB5dZE0gAAABl0RVh0U29mdHdhcmUAd3d3Lmlua3NjYXBlLm9yZ5vuPBoAAAERSURBVCiRhZG/SsMxFEZPfsVJ61jbxaF0cRQRcRJ9hlYn30IHN/+9iquDCOIsblIrOjqKgy5aKoJQj4O3EEtbPwhJbr6Te28CmdSKeqzeqr0YbfVIrTBKakvtOl5dtTkK+v4HfA9PEyBFCY9AGVgCBLaBp1jPAyfAJ/AAdIEG0dNAiyP7+K1qIfMdonZic6+WJoBJvQlvuwDqcXadUuqPA1NKAlexbRTAIMvMOCjTbMwl1LtI/6KWJ5Q6rT6Ht1MA58AX8Apcqqt5r2qhrgAXQC3CZ6i1+KMd9TRu3MvA3aH/fFPnBodb6oe6HM8+lYHrGdRXW8M9bMZtPXUji69lmf5Cmamq7quNLFZXD9Rq7v0Bpc1o/tp0fisAAAAASUVORK5CYII=)](https://github.com/marketplace/actions/chps-scorer)

GitHub Action to run automated security checks on container images according to [CHPs specification](https://github.com/chps-dev/chps) (Container Hardening Points).

This action assesses container images against multiple security vectors and gives them a grade (A+ to E) based on their:
- Minimalism (image size, layer count, etc.)
- Provenance (build sources, signatures, etc.)
- Configuration (user, permissions, etc.)
- CVE vulnerabilities

When used in conjunction with [Create Issue From File](https://github.com/peter-evans/create-issue-from-file), issues will be opened when the action finds security problems (make sure to specify the `issues: write` permission in the [workflow](https://docs.github.com/en/actions/writing-workflows/workflow-syntax-for-github-actions#permissions) or the [job](https://docs.github.com/en/actions/writing-workflows/workflow-syntax-for-github-actions#jobsjob_idpermissions)).

## Usage

Here is a full example of a GitHub workflow file:

This workflow will scan your container images once every day and create an issue if security issues are found. Save this under `.github/workflows/chps-scorer.yml`:

```yaml
name: "CHPs Container Security Check"

on:
  repository_dispatch:
  workflow_dispatch:
  schedule:
    - cron: "00 18 * * *"

jobs:
  chps-scorer:
    runs-on: ubuntu-latest
    permissions:
      issues: write # required for peter-evans/create-issue-from-file
    steps:
      - uses: actions/checkout@v4

      - name: CHPs Security Check
        id: chps-scorer
        uses: vipulgupta2048/chps-scorer-github-action@v1
        with:
          image: REPLACE_WITH:YOUR_IMAGE
          dockerfile: ./Dockerfile

      - name: Create Issue If Problems Found
        if: steps.chps-scorer.outputs.create_issue == 'true'
        uses: peter-evans/create-issue-from-file@v5
        with:
          title: Container Security Issues Detected
          content-filepath: ./chps-report.md
          labels: security, container, automated-issue
```

## Inputs

| Input          | Required | Default | Description                                               |
|----------------|----------|---------|-----------------------------------------------------------|
| image          | Yes      | -       | Container image to scan (e.g., nginx:latest)              |
| output-format  | No       | json    | Output format (options: json)                             |
| skip-cves      | No       | false   | Skip CVE scanning                                         |
| dockerfile     | No       | -       | Path to Dockerfile for additional checks                  |

## Outputs

| Output         | Description                                              |
|----------------|----------------------------------------------------------|
| output   | Whether findings should trigger an issue (true/false)    |

## Examples

### Basic scan of a public image

```yaml
- name: Scan nginx image
  uses: vipulgupta2048/chps-scorer-github-action@v1
  with:
    image: nginx:latest
```

### Scan with Dockerfile context

```yaml
- name: Scan custom image with Dockerfile
  uses: vipulgupta2048/chps-scorer-github-action@v1
  with:
    image: my-custom-image:latest
    dockerfile: ./path/to/Dockerfile
```

### Skip CVE scanning for faster results

```yaml
- name: Quick scan without CVEs
  uses: vipulgupta2048/chps-scorer-github-action@v1
  with:
    image: my-image:latest
    skip-cves: true
```


## Security and Updates

It is recommended to pin the CHPs Scorer action to a fixed version for security reasons. You can use Dependabot to automatically keep your GitHub actions up-to-date. This is a great way to pin the action while still receiving updates.

Create a file named `.github/dependabot.yml` with the following contents:

```yml
version: 2
updates:
  - package-ecosystem: "github-actions"
    directory: ".github/workflows"
    schedule:
      interval: "weekly"
```

When you add or update the `dependabot.yml` file, this triggers an immediate check for version updates.
See [the documentation](https://docs.github.com/en/code-security/dependabot/dependabot-version-updates/configuration-options-for-the-dependabot.yml-file) for all configuration options.

### Security tip

For additional security when relying on automation to update actions, you can pin the action to a SHA-256 instead of the semver version to avoid tag spoofing. Dependabot will still be able to automatically update this.

For example:

```yml
- name: CHPs Security Check
  uses: vipulgupta2048/chps-scorer-github-action@abcdef123456789abcdef123456789abcdef1234 # v1.0.0
```

## License

This action is licensed under the Apache License, Version 2.0.