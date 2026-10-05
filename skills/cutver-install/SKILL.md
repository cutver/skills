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
- Always verify precompiled binary releases with Cosign signatures and checksums.
- Confirm successful installation with `cutver --version`.

## Decision Gates
1. **Operating System & Tooling Detection**:
   - macOS / Linux with Homebrew: use Homebrew tap.
   - Windows with Scoop: use Scoop bucket.
   - Rust / Cargo environment: use `cargo install cutver`.
   - Standalone / Container / CI: download precompiled GitHub release binary or use container image.
2. **Binary Verification**:
   - Verify GitHub release asset signatures using `cosign verify-blob` against Cutver release public key.

## Execution Steps
1. Detect host environment (`uname -s`, `$env:OS`, or tooling presence).
2. Execute appropriate installation command:
   - **Homebrew** (macOS / Linux):
     ```bash
     brew install Row0902/tap/cutver
     ```
   - **Scoop** (Windows):
     ```powershell
     scoop bucket add row https://github.com/Row0902/scoop-bucket.git
     scoop install cutver
     ```
   - **Cargo** (Rust):
     ```bash
     cargo install cutver --locked
     ```
   - **GitHub Precompiled Binary** (Linux/macOS tarball):
     Download matching architecture from `https://github.com/cutver/cutver/releases/latest`, extract binary to PATH, and verify signature.
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
