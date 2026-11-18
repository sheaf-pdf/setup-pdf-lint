# Setup pdf-lint

Installs [pdf-lint](https://github.com/sheaf-pdf/pdf-lint), which checks
PDF files against the Arlington PDF Model, on a GitHub Actions runner. It
takes the release archive for the runner's platform, checks it against the
release's SHA-256, and puts `pdf-lint` on the `PATH`: there's nothing
else to install. From 0.2.0, pdf-lint decodes and decrypts by itself;
0.1.x's archive has the qpdf it decodes with beside it.

```yaml
steps:
  - uses: sheaf-pdf/setup-pdf-lint@v1
    with:
      version: 0.2.0
  - run: pdf-lint --last-update out.pdf
```

## Inputs

| Input | |
|---|---|
| `version` | The pdf-lint version to install, such as `0.2.0` (`v0.2.0` and `pdf-lint-v0.2.0` work too), or `latest`, the default. Pin one, so that a new release doesn't change what a workflow finds. A pre-release, such as `0.2.0-rc.1`, is installed only when it's named: `latest` is the newest release, never a pre-release. |

## Outputs

| Output | |
|---|---|
| `version` | The version installed. |
| `path` | The folder it was unpacked into, which is put on the `PATH`. 0.1.x's has the qpdf it decodes with in its `qpdf/`. |

## Runners

Each platform pdf-lint has a release archive for: Linux on x86_64 and
aarch64, macOS on Apple silicon and Intel, and Windows on x64. Another
fails the step, saying so. On Apple silicon it takes the arm64 archive,
as the runner's Node is arm64; the Intel one is for Intel Macs.

From 0.2.0 the Linux binary is static and needs nothing of the system, so
it runs in any container, Alpine's too. 0.1.x's bundled qpdf needs glibc
2.34 or later, which every GitHub-hosted Ubuntu runner has; in a container
without it, 0.1.x still runs and checks files in plain view, but leaves
unread those with anything compressed or encrypted, and the step warns of
it (naming another qpdf, 11.9 or later, in `PDF_LINT_QPDF` mends that).

It installs into the runner's tool cache, so a self-hosted runner reuses
an install from one job to the next. Behind a proxy, Node's `fetch` uses
`HTTPS_PROXY` only when `NODE_USE_ENV_PROXY=1` is set too.

## How it works

A JavaScript action with no dependencies: Node's `fetch` and `crypto`, and
the system's `tar`, which on Windows reads zip archives too. The newest
version is read from where the release page's `latest` link leads, not
from GitHub's API, so it needs no token and meets no rate limit.

`node --test` runs the unit tests.

## License

Apache-2.0, as pdf-lint is.
