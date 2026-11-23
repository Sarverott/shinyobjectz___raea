# Contribution helperboard

Started: 2026-11-23. Baseline: `main` at `a4898b76`.

This phased board travels with every PR in this contribution series. Each
implementation branch starts from the basic `main`, includes this board, and
links its work to the AppImage PR once that PR has a number. Branches are pushed
to the fork; upstream PRs prefer upstream's `development`, falling back to
`main` with an explanation when `development` is unavailable. A checked item
means completed work, not an upstream merge or policy approval.

## Phase 1: Inspect and preserve the baseline

- [x] Step 1. Inspect the fork, existing release workflow and working tree.
- [x] Step 2. Prepare the AppImage implementation on local `development`.
- [x] Step 3. Write this initial helperboard before commits, pushes and PRs.
- [x] Step 4. Verify the upstream repository, target branches and contribution guidance.
- [x] Step 5. Inventory existing community and agent guidance before calling anything missing.

## Phase 2: AppImage release PR (`development`)

- [x] Step 1. Review and validate the AppImage changes and documentation.
- [x] Step 2. Commit each changed file separately, with a rationale in its commit message.
- [x] Step 3. Push `development` to `origin`.
- [x] Step 4. Open the upstream PR and record the chosen base and any fallback.
- [x] Step 5. Include evidence, limitations and suggested repository improvements in the PR.
- [x] Step 6. Record the PR URL and validation results in this board.

## Phase 3: Project and agent guide PR (`development-2`)

- [x] Step 1. Create `development-2` from the original `main`, carrying this helperboard.
- [x] Step 2. Inspect the source, architecture, CLI, SDK, feature specifications and checks.
- [x] Step 3. Write a comprehensive `AGENTS.md` describing the project, execution flow,
      available features, source boundaries, build/check commands and contribution workflow.
- [x] Step 4. Include a clearly optional, direct BOS reference for agents and maintainers
      to evaluate; do not imply BOS has already been adopted by upstream.
- [x] Step 5. Add `CLAUDE.md` as a pointer to `AGENTS.md`, without duplicating guidance.
- [x] Step 6. Validate documentation, commit files separately and push the branch to `origin`.
- [x] Step 7. Open another upstream PR, linking it to the AppImage PR and carrying this board.

## Phase 4: Community and maintenance follow-ups

The AppImage PR will distinguish absent files from improvements to existing
ones. Each accepted follow-up uses its own branch from `main`, carries this
board, and says “According to the suggestion mentioned in PR #NUMBER” with a
link to the actual upstream PR. Scope and policy remain subject to maintainer
review; do not invent private reporting addresses or enforcement contacts.

- [x] Step 1. Propose a code of conduct with an explicit, verified reporting route.
- [x] Step 2. Improve existing contribution guidance where inspection finds gaps.
- [x] Step 3. Add issue templates appropriate to bugs and feature proposals.
- [x] Step 4. Improve the existing security notice without inventing support guarantees.
- [x] Step 5. Add a PR template and other useful GitHub guidance justified by the repository.
- [x] Step 6. Confirm whether these remaining follow-ups should be implemented in this
      session or kept as tracked work after the agent-guide PR: the contributor
      explicitly requested implementation now, in separate PRs.

## Evidence and PR register

| Branch in `Sarverott/shinyobjectz___raea` | Scope | Upstream PR |
| --- | --- | --- |
| `development` | Linux AppImage script, launcher, release checks, workflow and docs | [#226](https://github.com/shinyobjectz/raea/pull/226) |
| `development-2` | Project/feature/contributor guide and Claude import | [#227](https://github.com/shinyobjectz/raea/pull/227) |
| `development-3` | Code of conduct and reporting/moderation guidance | [#228](https://github.com/shinyobjectz/raea/pull/228) |
| `development-4` | Expanded existing contribution guide | [#229](https://github.com/shinyobjectz/raea/pull/229) |
| `development-5` | Bug/feature issue forms and chooser links | [#230](https://github.com/shinyobjectz/raea/pull/230) |
| `development-6` | Expanded existing security notice | [#231](https://github.com/shinyobjectz/raea/pull/231) |
| `development-7` | PR template | [#232](https://github.com/shinyobjectz/raea/pull/232) |

All seven PRs were opened against `shinyobjectz/raea:main`. Each follow-up links
to the corresponding suggestion in PR #226. Their scope is independent; the
same final helperboard is carried in each branch to avoid divergent plan files.
Opening these proposals does not merge them or adopt their policies upstream.

- Baseline: local `main` and `origin/main` were `a4898b76` when this series began.
- AppImage: shell syntax, desktop entry, Actions lint and an integration fixture
  using real linuxdeploy/AppImage tooling passed. Full RAEA compilation and a
  GitHub release run have not been performed.
- Upstream: `shinyobjectz/raea`; its current `main` matches the original baseline.
  GitHub's branch search returned no `development` branch, so PRs target `main`
  with the attempted preference recorded in each PR description.
- Existing files: `CONTRIBUTING.md` asks contributors to discuss proposals in an
  issue first; `SECURITY.md` directs private reports to GitHub's Security tab.
  Improvements to these are proposals, not claims that the files are missing.
- Missing at baseline: root `AGENTS.md`, root `CLAUDE.md`, a code of conduct,
  issue templates and a pull request template.
- AppImage PR: [shinyobjectz/raea#226](https://github.com/shinyobjectz/raea/pull/226),
  `Sarverott:development` → upstream `main`. Eight initial commits each changed
  one file; later helperboard-only commits record progress.
- Project/agent guide PR: [shinyobjectz/raea#227](https://github.com/shinyobjectz/raea/pull/227),
  `Sarverott:development-2` → upstream `main`, independently based on `a4898b76`.
- Guide validation: all relative Markdown links resolve, `CLAUDE.md` contains
  only `@AGENTS.md`, whitespace and public-content checks pass. The guide is
  source-backed; no new engine/GPU tests were claimed or run for the docs.
- Architecture discovery: GitNexus returned no indexed repositories; direct
  source and current manual inspection supplied the evidence instead.
- Security route: GitHub's API reported upstream private vulnerability reporting
  enabled on 2026-11-23. The setting is separate from `SECURITY.md` and can change.
- Community validation: Markdown links and source references checked; issue-form
  YAML, field IDs/types, required fields, choices and contact links validated;
  whitespace and public-guard/Gitleaks checks passed for each branch. GitHub's
  rendered templates require adoption on the default branch before user testing.
- Conduct reporting: no separate maintainer inbox is documented. PR #228 uses
  GitHub's reporting routes, notes availability limits, and leaves any additional
  private contact to maintainers rather than inventing one.
