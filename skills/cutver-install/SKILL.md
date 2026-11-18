---
name: cutver-install
description: "Trigger: install cutver, setup cutver cli, brew install cutver, scoop install cutver, download cutver. Detect platform environment and install Cutver CLI via native package managers or verified binaries."
license: Apache-2.0
metadata:
  author: cutver
  version: "2.0"
---

## Activation Contract
Activate when the user asks how to install Cutver, set up the Cutver CLI, download precompiled binaries, or run Cutver installation commands.

## Hard Rules
- Detect operating system and architecture before prescribing an installation method.
- Prioritize native package managers: Homebrew (macOS/Linux), Scoop (Windows), Cargo (Rust toolchain).
- Verify precompiled release assets before running them: `SHA256SUMS` lists every archive and `SHA256SUMS.bundle` is its Cosign signature, both produced by the release workflow and published with the release.
- Confirm successful installation with `cutver --version`.

## Decision Gates
1. **Operating System & Tooling Detection**:
   - macOS / Linux with Homebrew: use Homebrew tap.
   - Windows with Scoop: use Scoop bucket.
   - Rust / Cargo environment: use `cargo install cutver`.
   - Standalone / Container / CI: download precompiled GitHub release binary or use container image.
2. **Build Variant**:
   - Lean `cutver` (~1.25 MB): the core engine, for minimal CI runners.
   - `cutver-plugins` (~5 MB): adds the WebAssembly plugin runtime (`cutver plugin add/remove/list`). In a lean build, running `cutver plugin <subcommand>` prints "this build of cutver was compiled without the 'plugins' feature, so 'cutver plugin <op>' is unavailable" plus a "Fix: install or build a plugin-enabled cutver" hint, and exits `1`; `cutver plugin --help` still prints help and exits `0`. No flag enables the runtime.
3. **Binary Verification**:
   - Check each downloaded archive against `SHA256SUMS`, then verify `SHA256SUMS` itself with Cosign against `SHA256SUMS.bundle` and the release's signing identity.

## Execution Steps
1. Detect host environment (`uname -s`, `$env:OS`, or tooling presence).
2. Execute appropriate installation command:
   - **Homebrew** (macOS / Linux), pick one:
     ```bash
     brew install Row0902/tap/cutver
     brew install Row0902/tap/cutver-plugins
     ```
   - **Scoop** (Windows), pick one:
     ```powershell
     scoop bucket add row https://github.com/Row0902/scoop-bucket
     scoop install cutver
     scoop install cutver-plugins
     ```
   - **Cargo** (Rust), pick one:
     ```bash
     cargo install cutver --locked
     cargo install cutver --locked --features plugins
     ```
   - **GitHub Precompiled Binary** (Linux/macOS tarball):
     Download `cutver-<version>-<platform>-<arch>.tar.gz` — or the `cutver-plugins-<version>-…` archive for the plugin runtime — from `https://github.com/cutver/cutver/releases`, check it against `SHA256SUMS`, and extract the binary to PATH.
3. Validate installation:
   ```bash
   cutver --version
   ```

## Output Contract
- Emit executed installation command and package manager output.
- Print confirmed version string (`cutver x.y.z`).

## References
- Homebrew Tap: `Row0902/tap/cutver`
- Cargo Crate: `https://crates.io/crates/cutver`
- GitHub Releases: `https://github.com/cutver/cutver/releases`
