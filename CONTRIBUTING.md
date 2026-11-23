# Contributing

raea is a canary release and changes frequently. Filing a useful issue is often
the quickest way to help. Pull requests are considered case by case: **open an
issue first**, describe the proposed change and agree on its scope with the
maintainers before spending time on an implementation. A contribution that lands
keeps you as the author of your commits in this repository's history.

## Reports and proposals

For a bug, include:

- `raea --version`, the command you ran, and whether you used a release or a source build.
- OS/version, CPU architecture and, for rendering problems, GPU, driver and
  backend (`WGPU_BACKEND` if you set it).
- Expected and observed behavior, including the frame or timestamp for a visual problem.
- A minimal composition, app or script; include small assets only when you have
  permission to redistribute them, or explain how to reproduce without them.
- Relevant output or traces with credentials, private paths and personal data removed.

A reference frame and the corresponding raea render help with visual reports.
For an idea, describe the task you want to accomplish and why the existing
workflow is insufficient. Use [the security policy](SECURITY.md) for
vulnerabilities; do not include exploit details in public bug reports.

## Find the implementation

Start with [the README](README.md), [the manual](docs/manual/index.md), the relevant
crate/package and its nearby `.feature` files. The Rust command is in
`engine/cli`; TypeScript SDK import paths are mapped by `sdk/tsconfig.base.json`.
The public engine, user-authored TSX app hosts and the separately distributed
desktop editor are different components. The editor's `apps/app` directory is
absent from this baseline, and a CLI integration does not supply its source.

## Reproducible setup

Use Rust from `rust-toolchain.toml` and the Bun version in
[public CI](.github/workflows/ci.yml). Install the platform prerequisites in the
README. Initialize submodules when working with skills or task definitions:

```sh
git submodule update --init --recursive
for d in sdk/raea sdk/react/js sdk/react/tw; do
  (cd "$d" && bun install --frozen-lockfile)
done
cargo build --release --locked -p raea-cli --bin raea --no-default-features
target/release/raea --help
```

Create a topic branch from the agreed upstream base. Check which branches exist;
the fork's `development` names do not establish an upstream branching policy.
Keep the change focused, preserve other contributors' work, and use explanatory
commit messages. Update generated files using their generators rather than
editing generated sections to make them appear current.

## Verification and review

Choose checks for the code you changed; use CI's current commands and prerequisites
as the reference. Examples include:

```sh
git diff --check
cargo check --locked -p raea-cli --tests
cargo test --locked -p raea-core-k -p raea-core-fonts -p raea-bundle
bash tools/layers.sh
bash tools/names.sh
bash tools/size.sh
```

Changes to SDK props, coverage, taxonomy or generated references need their
corresponding `tools/` checks. Do not expand layer/size allowances to hide a new
violation. Visual changes need a small render or app-interaction reproduction and
inspection of the affected result; audio, timing, platform and packaging changes
need evidence appropriate to that behavior. GPU/headless/window checks have
different requirements, and Linux window-host checks may need Xvfb.

Before publishing, run `bash tools/public-guard.sh staged` and
`bash tools/public-guard.sh range BASE..HEAD`; the range check requires Gitleaks.
Do not commit secrets, personal machine paths, private assets or agent session
artifacts. Automated or AI-assisted work is held to the same evidence and review
standards as any other contribution; verify generated claims and code.

In the PR, link the agreed issue or proposal, describe the trigger and resulting
behavior, list the checks that actually ran, and state missing prerequisites or
untested platforms. A mocked fixture or static check is useful evidence but is
not an actual engine/GPU test. Documentation-only changes can be checked for
source accuracy, links and formatting without compiling the whole engine.

The contribution series proposed in [PR #226](https://github.com/shinyobjectz/raea/pull/226)
carries a phased [helperboard](docs/devlog/TODO.md) in each independent PR. Its
branch names and one-file commit convention describe that series rather than
changing upstream's contribution policy for everyone.
