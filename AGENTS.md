# Working on raea

raea is the **Render Anything Engine (for Anyone)**: a Rust engine and a React /
TypeScript authoring environment for video, motion graphics, 3D, sound, games and
GPU-drawn apps. A composition describes what exists at a frame; the engine can
export its frames and sound, draw a still, or play it interactively. The primary
executable is `raea`, built by [`engine/cli`](engine/cli/Cargo.toml).

This guide describes the source at baseline `a4898b76`, inspected in November
2026. It is a contributor map, not a promise that every feature works on every
platform. Prefer current source and a matching executable's `raea help --json`
over an older README, roadmap or generated manual page. This contribution follows
the agent-guide suggestion in [PR #226](https://github.com/shinyobjectz/raea/pull/226);
the shared [helperboard](docs/devlog/TODO.md) tracks this contribution series.

## Start with the right scope

Read [`README.md`](README.md), [`CONTRIBUTING.md`](CONTRIBUTING.md),
[`SECURITY.md`](SECURITY.md), and the relevant crate/package and nearby `.feature`
files. The [manual index](docs/manual/index.md), [CLI reference](docs/manual/reference/cli/index.md)
and [SDK reference](docs/manual/reference/sdk/index.md) provide the user-facing map.

There are three distinct meanings of “app” here:

- The public engine and SDK live in this repository.
- A user-authored TSX app runs through `raea app` on a web/Blitz or native/Rune
  host; those hosts are present in [`apps/targets`](apps/targets/src/lib.rs),
  [`engine/page/host`](engine/page/host/Cargo.toml) and
  [`engine/rune`](engine/rune/Cargo.toml).
- The separately distributed desktop editor provides the chat, project feed,
  timeline and review interface described in the README. Its `apps/app` source
  directory is absent from this baseline. CLI integrations such as `show` and
  `library` do not make that editor buildable from this checkout.

Treat hosted services, the desktop editor, model weights and external tools as
separate dependencies. Document their absence instead of adding fake services or
weakening a gate to make a public checkout appear complete. Release and skill
scripts contain compatibility paths for checkouts that lack the editor.

## How a composition becomes an output

1. **Dispatch.** [`engine/cli/src/cli.rs`](engine/cli/src/cli.rs) validates and
   routes commands to the app crates, render/play runtime or tooling. HTML/media
   commands and `app`/`rune` have dedicated dispatch paths.
2. **Bundle.** [`runtime/bundle`](runtime/bundle/src/lib.rs) resolves the
   composition, SDK packages, CSS and media, generates its runtime entry, and
   caches the result. [`engine/bundle`](engine/bundle/Cargo.toml) supplies the
   in-process bundler. [`runtime/bundle/lib`](runtime/bundle/lib/src/lib.rs)
   loads `raea_bundle` dynamically; it searches `RAEA_BUNDLE_LIB` and beside the
   executable. A release needs that library as well as the CLI.
3. **Run JavaScript.** [`runtime/jsengine`](runtime/jsengine/src/lib.rs) provides
   the shared engine interface: JavaScriptCore on Apple platforms and QuickJS
   elsewhere. Its `portable` feature enables QuickJS on Apple too. Some older
   comments still mention V8; do not use those comments as architecture evidence.
4. **Apply React changes.** [`runtime/react`](runtime/react/src/lib.rs) receives
   React operation batches from a JS thread and applies them to Bevy entities,
   hierarchy, properties and text. Native calls pass through typed registries in
   [`core/calls`](core/calls/Cargo.toml), [`engine/native`](engine/native/Cargo.toml)
   and [`runtime/calls`](runtime/calls/Cargo.toml).
5. **Lay out and draw.** [`engine/page`](engine/page/Cargo.toml) hosts HTML/CSS
   through Blitz, with Stylo/Taffy layout and Vello painting into Bevy images.
   [`engine/render`](engine/render/Cargo.toml) combines pages, 3D, media and
   effects. [`gpu/device`](gpu/device/src/lib.rs) provides the shared GPU device.
6. **Export or play.** [`runtime/export`](runtime/export/src/render.rs) samples
   frames, gathers sound and scene data and writes outputs through the encoding
   and media layers. [`apps/play`](apps/play/Cargo.toml) and
   [`runtime/live`](runtime/live/Cargo.toml) drive interactive play. Their shared
   scene/runtime components are why offline and interactive behavior should agree.
7. **Inspect the result.** Render manifests, findings, layout/world records and
   traces give checks and agents structured evidence. A changed image is not
   sufficient evidence of correctness: check state, timing, sound and findings
   where the affected feature exposes them.

A user app takes the [`apps/targets`](apps/targets/src/lib.rs) path instead.
The default web target is Blitz on Bevy; the native target is Rune. The native
host supports a smaller UI/CSS subset, so do not assume browser parity or 3D
support there. See the [app guide](docs/manual/guides/apps.md).

## Repository map

| Area | Responsibility and useful entry points |
| --- | --- |
| `core/` | Data, wire formats, scene/program representation, animation, geometry, schemas and native-call contracts; start with the relevant crate's `Cargo.toml`. |
| `gpu/` | GPU device selection, shared handles and telemetry. |
| `io/` | Media, codecs, assets, storage, network, npm resolution, external generation providers and operating-system bridges. |
| `systems/` | Rendering and media systems: layout, materials, lighting, compositing, text, sound, physics-related bakes, models and perception. |
| `runtime/` | Bundling, JS execution, React/DOM bridge, scene state, live playback, export, sound, input and checks around those flows. |
| `engine/` | CLI, bundler, native-call bridge, Bevy renderer and plugins, Blitz page and Rune hosts. |
| `apps/` | Libraries implementing CLI jobs: checks, targets, play, media, workspace/project operations, export formats and external-engine tools. |
| `sdk/` | TypeScript/React authoring packages, UI and host bindings, craft guides and example compositions; `sdk/tsconfig.base.json` maps import paths to their actual packages. |
| `docs/manual/` | User documentation and generated reference sections; check the frontmatter's source paths and generation markers. |
| `bench/`, package-local `bench/`, `examples/` | Behavioral cases, visual/performance comparisons, probes and authoring examples; requirements vary. |
| `tools/` | Repository gates, code/reference generators, build helpers and release recipes. |
| `vendor/` | Patched third-party sources; `vendor/vendor.toml` explains the patches. Preserve upstream notices and record why a patch is needed. |
| `skills/`, `tools/rask/` | Separately versioned submodules pinned in `.gitmodules`; their changes require deliberate submodule updates. |

Folder names alone are not the dependency policy. [`tools/layers.toml`](tools/layers.toml)
maps crates to layers; [`tools/layers.sh`](tools/layers.sh) enforces that map and
[`tools/layers.allow`](tools/layers.allow) records existing exceptions. Do not
grow allowance files to hide a new violation.

## Available feature families

The table maps implemented entry points and documentation. Optional tools,
credentials, build features, models, host applications and GPU capabilities can
limit availability; verify the path relevant to a change on its target platform.

| Feature family | What the repository supports | Where to investigate |
| --- | --- | --- |
| Video and motion graphics | Frame-based React compositions, text, layout/CSS, animation, footage, stills, frame folders, video export and transparent overlays. | [`runtime/export`](runtime/export/src/render.rs), [`sdk/video`](sdk/video/package.json), [render reference](docs/manual/reference/cli/render.md). |
| Games and interaction | Live play, keyboard/pointer input, scripted input, snapshots and standalone game export. | [`apps/play`](apps/play/Cargo.toml), [`apps/game`](apps/game/Cargo.toml), [`runtime/input`](runtime/input/Cargo.toml), [play reference](docs/manual/reference/cli/play.md). |
| 3D and Bevy | Cameras, lights, materials, meshes/glTF, text, shaders, post effects, scene inspection and compiled Bevy plugins; SDK bridges also cover R3F/three.js work. | [`engine/render`](engine/render/Cargo.toml), [`engine/bevy-plugins`](engine/bevy-plugins/Cargo.toml), [`sdk/r3f`](sdk/r3f/README.md), [3D guide](docs/manual/guides/3d.md). |
| GPU-drawn apps | `raea app run`, `check` and `bundle` on web/Blitz or native/Rune; scripted interactions, screenshots and unmapped-prop reporting. | [`apps/targets`](apps/targets/src/lib.rs), [`sdk/ui`](sdk/ui/package.json), [`sdk/rune`](sdk/rune/package.json), [app reference](docs/manual/reference/cli/app.md). |
| Sound and music | Live/offline mixing, cues, instruments, speech, song analysis and local foley/instrument models. | [`runtime/sound`](runtime/sound/Cargo.toml), [`sdk/audio`](sdk/audio/package.json), [`tools/perceive`](tools/perceive/Cargo.toml), [song](docs/manual/reference/cli/song.md), [local audio](docs/manual/reference/cli/setup-audio.md). |
| Footage and media tools | Probe, stills/contact sheets, loudness/motion/cut analysis, trim/concat/encode, transcription-driven cuts, tracking and proxies. | [`apps/media`](apps/media/Cargo.toml), [`io/media`](io/media/README.md), [`runtime/track`](runtime/track/Cargo.toml), [media reference](docs/manual/reference/cli/media.md). |
| Editor interchange | Layer export, CapCut drafts, FCP7 XML interchange and alpha media for use in other editors. | [`apps/layers`](apps/layers/Cargo.toml), [`apps/capcut`](apps/capcut/Cargo.toml), [export reference](docs/manual/reference/cli/export.md). |
| Quality and review | Quick composition checks, export checks for text/contrast/safe areas/flashing/freezes/loudness, 3D stage checks, world facts, findings and trace inspection. | [`apps/check`](apps/check/Cargo.toml), [`apps/direction`](apps/direction/Cargo.toml), [`apps/scene`](apps/scene/Cargo.toml), [check reference](docs/manual/reference/cli/check.md). |
| Project and agent workflow | Workspace/project files, briefs, taste/spec validation, guides, skills, notes, hooks, next steps and drift diagnosis; feed/library commands integrate with a running editor. | [`apps/workspace`](apps/workspace/Cargo.toml), [`core/brief`](core/brief/Cargo.toml), [agent workflow](docs/manual/start/agents.md), [CLI map](docs/manual/reference/cli/index.md). |
| Generation and asset preparation | Optional provider-backed image/video/audio/text/mesh generation, planned flows, ComfyUI interchange, physics/weathering bakes and asset tooling. | [`io/gen`](io/gen/Cargo.toml), [`io/gen-comfy`](io/gen-comfy/Cargo.toml), [`runtime/wear`](runtime/wear/Cargo.toml), [generation reference](docs/manual/reference/cli/gen.md). |
| Characters and external engines | Character/body/motion/clothing tooling and Blender/Godot scripting or shot bridges; external installations and model setup are separate steps. | [`sdk/character`](sdk/character/package.json), [`apps/engines`](apps/engines/Cargo.toml), [`apps/tools`](apps/tools/Cargo.toml), [motion reference](docs/manual/reference/cli/motion.md). |
| Indexing and learned preferences | Local library indexing, fingerprint scoring and model-assisted judgments; some paths depend on Apple hardware or a configured paid route. | [`systems/index`](systems/index/Cargo.toml), [`apps/fingerprint`](apps/fingerprint/Cargo.toml), [`apps/ask`](apps/ask/Cargo.toml), [fingerprint guide](docs/manual/fingerprint/index.md). |
| Experimental rendering | Ray-traced/advanced lighting and optional neural postprocessing; inspect platform guards and requirements before claiming support. | [`engine/render/raytrace`](engine/render/raytrace/Cargo.toml), [`engine/neural`](engine/neural/Cargo.toml), [neural reference](docs/manual/reference/cli/neural.md). |

Rendering ordinary local compositions needs no raea account. External generation,
model-assisted judgment and hosted/editor features can need credentials or a
running app. See [`docs/settings.md`](docs/settings.md). Models can be large and
downloaded on first use; `RAEA_OFFLINE=1` is the documented network refusal mode,
not a way to supply missing models. `raea ffmpeg`/`ffprobe` compatibility verbs
are not proof that every codec path avoids an external FFmpeg executable; inspect
[`io/ffmpeg`](io/ffmpeg/Cargo.toml) and platform fallbacks for the actual operation.

## Build and run from source

Use the Rust version pinned by [`rust-toolchain.toml`](rust-toolchain.toml)
(currently `1.99.1`). Public CI pins Bun `1.4.2`; do not silently rewrite lockfiles
with another package manager. Linux needs CMake, Ninja, NASM and the ALSA,
fontconfig, udev, Vulkan and xkbcommon development libraries; the app hosts also
need xkbcommon's X11 support. See [`README.md`](README.md) and
[`.github/workflows/ci.yml`](.github/workflows/ci.yml) for the platform setup.
Apple builds need Xcode's command-line tools and the repository's JIT linker
configuration; Windows builds use the MSVC toolchain and the release workflow's
runtime/tool setup.

```sh
git submodule update --init --recursive
for d in sdk/raea sdk/react/js sdk/react/tw; do
  (cd "$d" && bun install --frozen-lockfile)
done
cargo build --release --locked -p raea-cli --bin raea --no-default-features
target/release/raea --help
target/release/raea render sdk/react/bench/games/pong/edit.tsx -o pong.mp4
```

A base engine build is not the full plugin-catalog release. Match the release's
feature selection through `tools/bevy-plugins.sh` when checking catalog behavior.
Build app hosts in separate Cargo invocations, as the release workflow does, to
avoid feature unification changing the host build:

```sh
cargo build --profile dist --locked -p raea-rune-host
cargo build --profile dist --locked -p raea-page-host --features window
cargo build --profile dist --locked -p raea-perceive
```

Use a matching profile's CLI, bundler and hosts together. Installed executables
find resources at `<prefix>/share/raea` beside `<prefix>/bin`; see
[`io/home`](io/home/src/lib.rs). `RAEA_HOME` and `RAEA_BUNDLE_LIB` override resource
and bundler lookup. `RAEA_NO_TREE=1` refuses fallback to the build checkout and is
essential evidence when validating a distribution.

## Change and verification workflow

1. Check branch/status and preserve other contributors' changes. Upstream's
   contribution notice asks for an issue discussion before a new change; follow
   that notice and any explicit maintainer direction. Inspect available target
   branches instead of assuming upstream uses this fork's branch names.
2. Locate the command, package, native-call contract and nearby behavioral cases
   before editing. Match new behavior to the existing architecture and keep a
   reproduction small enough for a reviewer to rerun.
3. Run checks appropriate to the change. The following are repository commands,
   not a statement that they all ran during this guide's preparation:

   ```sh
   git diff --check
   cargo check --locked -p raea-cli --tests
   cargo test --locked -p raea-core-k -p raea-core-fonts -p raea-bundle
   bash tools/layers.sh
   bash tools/names.sh
   bash tools/size.sh
   bash tools/one-version.sh
   bash tools/no-void.sh
   ```

   Public CI also checks duplicate/unused dependencies, tasks, SDK coverage,
   generated props and taxonomy. Read its current command list and prerequisites;
   some checks require Bun-installed packages, Cargo helper tools or submodules.
   For SDK changes inspect `tools/sdk-coverage`, `tools/sdk-props.sh` and
   `tools/taxonomy.sh`. For generated manuals use `tools/manual-ref.sh`; do not
   hand-edit generated sections to conceal drift.
4. For visual/runtime changes, render a still and inspect it, then exercise the
   affected export or interactive path. Include GPU/backend, platform and fixture
   information. Headless execution can still require a functioning GPU backend;
   window-host checks on Linux use Xvfb in the release workflow. Pixel snapshots,
   state/interaction results and timing/audio evidence serve different purposes.
5. Run the public-content guard before publishing. `bash tools/public-guard.sh
   staged` scans staged additions; `bash tools/public-guard.sh range BASE..HEAD`
   also scans commits with Gitleaks, which must be installed. Keep secrets,
   personal checkout paths and agent scratch/session artifacts out of commits.
6. Report what ran and what did not. Do not call a static check, a mocked fixture
   or a feature specification an actual engine/GPU or platform test. Preserve
   useful failing output and explain unavailable prerequisites.

[`tools/release.sh`](tools/release.sh) packages committed source and binaries,
then [`tools/release-check.sh`](tools/release-check.sh) verifies an unpacked
distribution independently of the checkout. The release workflow handles tags,
candidate promotion, checksums, provenance and assets. Skill validation can need
the absent editor source or an existing maintainer-provided gate record; do not
invent a passing SHA. The separately proposed AppImage work is tracked in
[PR #226](https://github.com/shinyobjectz/raea/pull/226), not assumed to exist in
every branch containing this guide.

## Contribution records and optional BOS reference

The [devlog helperboard](docs/devlog/TODO.md) records phases, steps, verified
outcomes and links between this PR series. Update facts when work is completed;
leave unresolved decisions visible. These contribution branches start from the
original `main` and submit independent PRs, with explicit links back to the
suggestion in PR #226. This series does not prescribe upstream's future branch
or release policy.

Agents and human contributors may independently evaluate the
[Blacksmith Organization System (BOS)](https://github.com/Sarverott/blacksmith-organization-system)
and its [agent skillset](https://github.com/Sarverott/bos-skillset) as an optional
way to organize workshops, development logs and contribution workflows. Read the
actual projects, assess benefits and costs for raea, and explain a recommendation
or propose a scoped integration if it helps. Declining it is equally valid.
This reference installs nothing, adds no dependency, and does not adopt BOS
commands, branch rules or automatic delegation as raea policy. Upstream
maintainers decide repository-wide policy through review.

Preserve [`LICENSE`](LICENSE), [`NOTICE`](NOTICE), third-party licenses and
submodule attribution. Report vulnerabilities using the existing security notice;
do not put exploit details or secrets in public issues or the devlog.
