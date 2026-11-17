# CFPB GitHub Actions

Reusable GitHub actions for use by the CFPB organization.

## docker-build-push

Build, optionally test and/or scan, and push a Docker image to GHCR.

**PRs from forks build but never get pushed to GHCR.**

### Tagging scheme

| Event            | Tags created                                         |
| ---------------- | ---------------------------------------------------- |
| Push to `main`   | `latest`, `main`, `main-20260228-abc1234`, `abc1234` |
| Pull request #42 | `pr-42`, `pr-42-20260228-abc1234`, `abc1234`         |
| Git tag `v1.2.3` | `v1.2.3`, `abc1234`                                  |
| Other            | `local`                                              |

### Usage

#### Basic

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v4
      - uses: cfpb/actions/docker-build-push@main
        with:
          image-name: myapp
          token: ${{ secrets.GITHUB_TOKEN }}
```

#### Build with custom Docker context

```yaml
- uses: cfpb/actions/docker-build-push@main
  with:
    image-name: myapp
    token: ${{ secrets.GITHUB_TOKEN }}
    context: ./path/to/dockerfile/
```

#### Build a specific multi-stage target

```yaml
- uses: cfpb/actions/docker-build-push@main
  with:
    image-name: myapp
    token: ${{ secrets.GITHUB_TOKEN }}
    target: base
```

#### Skip push (build only)

```yaml
- uses: cfpb/actions/docker-build-push@main
  with:
    image-name: myapp
    token: ${{ secrets.GITHUB_TOKEN }}
    skip-push: true
```

#### Run test command before push

Test command receives `IMAGE` as an environment variable.

```yaml
- uses: cfpb/actions/docker-build-push@main
  with:
    image-name: myapp
    token: ${{ secrets.GITHUB_TOKEN }}
    test-command: my-test-command.sh
```

#### Scan image before push

Scans the built image with the [Wiz](https://www.wiz.io/) vulnerability scanner.

- `scan-mode: enforce` (default) fails the job, and skips the push, if the scan fails.
- `scan-mode: report` records the result but always continues.

The scan is skipped if scanner credentials are empty.

```yaml
- uses: cfpb/actions/docker-build-push@main
  with:
    image-name: myapp
    token: ${{ secrets.GITHUB_TOKEN }}
    scan: true
    scan-mode: ${{ github.event_name == 'pull_request' && 'enforce' || 'report' }}
    scanner-client-id: ${{ secrets.SCANNER_CLIENT_ID }}
    scanner-client-secret: ${{ secrets.SCANNER_CLIENT_SECRET }}
```

### Inputs

| Input                   | Required | Default   | Description                                                        |
| ----------------------- | -------- | --------- | ------------------------------------------------------------------ |
| `image-name`            | Yes      | -         | Name for the image (e.g. "myapp" → ghcr.io/cfpb/myapp)             |
| `token`                 | Yes      | -         | GitHub token for registry authentication                           |
| `context`               | No       | `.`       | Docker build context                                               |
| `target`                | No       |           | Multi-stage Dockerfile target to build                             |
| `skip-push`             | No       | `false`   | Skip pushing (build only)                                          |
| `test-command`          | No       |           | Test command to run against built image. Receives `IMAGE` env var. |
| `scan`                  | No       | `false`   | Scan the built image for vulnerabilities before pushing            |
| `scan-mode`             | No       | `enforce` | `enforce` (fail job on scan failure) or `report`                   |
| `scanner-client-id`     | No       |           | Scanner client ID. Scan is skipped if empty.                       |
| `scanner-client-secret` | No       |           | Scanner client secret. Scan is skipped if empty.                   |

### Outputs

| Output            | Example                                                                       |
| ----------------- | ----------------------------------------------------------------------------- |
| `short-sha`       | `abc1234`                                                                     |
| `registry-image`  | `ghcr.io/cfpb/myapp`                                                          |
| `mutable-version` | `main`, `pr-42`, `v1.2.3`, or `local`                                         |
| `mutable-tag`     | `ghcr.io/cfpb/myapp/main`, ...                                                |
| `sha-version`     | `main-20260228-abc1234`, ...                                                  |
| `sha-tag`         | `ghcr.io/cfpb/myapp/main-20260228-abc1234`, ...                               |
| `short-sha-tag`   | `ghcr.io/cfpb/myapp/abc1234`, ...                                             |
| `tags`            | `ghcr.io/cfpb/myapp/main-20260228-abc1234,...` (empty for unsupported events) |
| `pushed`          | `true` or `false`                                                             |
| `digest`          | `sha256:...` (if pushed)                                                      |
| `scan-result`     | `passed`, `failed`, `skipped`, or empty if `scan` is `false`                  |
