# CFPB GitHub Actions

Reusable GitHub actions for use by the CFPB organization.

## docker-build-push

Build, optionally test, and push a Docker image to GHCR.

**PRs from forks build but never get pushed to GHCR.**

### Tagging scheme

| Event            | Tags created                              |
| ---------------- | ----------------------------------------- |
| Push to `main`   | `latest`, `main`, `main-20260228-abc1234` |
| Pull request #42 | `pr-42`, `pr-42-20260228-abc1234`         |
| Git tag `v1.2.3` | `v1.2.3`                                  |
| Other            | `local`                                   |

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

### Inputs

| Input          | Required | Default | Description                                                        |
| -------------- | -------- | ------- | ------------------------------------------------------------------ |
| `image-name`   | Yes      | -       | Name for the image (e.g. "myapp" → ghcr.io/cfpb/myapp)             |
| `token`        | Yes      | -       | GitHub token for registry authentication                           |
| `context`      | No       | `.`     | Docker build context                                               |
| `skip-push`    | No       | `false` | Skip pushing (build only)                                          |
| `test-command` | No       |         | Test command to run against built image. Receives `IMAGE` env var. |

### Outputs

| Output            | Example                                                                       |
| ----------------- | ----------------------------------------------------------------------------- |
| `short-sha`       | `abc1234`                                                                     |
| `registry-image`  | `ghcr.io/cfpb/myapp`                                                          |
| `mutable-version` | `main`, `pr-42`, `v1.2.3`, or `local`                                         |
| `mutable-tag`     | `ghcr.io/cfpb/myapp/main`, ...                                                |
| `sha-version`     | `main-20260228-abc1234`, ...                                                  |
| `sha-tag`         | `ghcr.io/cfpb/myapp/main-20260228-abc1234`, ...                               |
| `tags`            | `ghcr.io/cfpb/myapp/main-20260228-abc1234,...` (empty for unsupported events) |
| `pushed`          | `true` or `false`                                                             |
| `digest`          | `sha256:...` (if pushed)                                                      |
