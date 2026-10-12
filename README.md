# Portrait standalone binary candidates

Local preparation only. Proposed repository: `islee23520/portrait-generator-releases`; proposed tag: `portrait-v0.1.0`. These download URLs are prospective and are not live. Source commit: `1bbfbdc8f1518c379307862449027ed87330b174`. No publication has occurred.

Only the ten executables in `assets/` are payload candidates. This guide and SHA256SUMS are public-safe metadata candidates. No shell/PowerShell bootstrap scripts are staged: their source would be an additional disclosure. The published installation guide, if approved, downloads binaries directly and verifies a separately reviewed checksum before execution. No source, JS, source maps, tarballs, private inputs or reference images are staged. Binaries contain executable code and may retain comments or internal source paths; they are not claimed to resist reverse engineering.

## Platform status and prerequisites

- `darwin-arm64`: native Apple Silicon install and help verified; fresh source-bound preflight receipt recorded separately. Requires macOS with the matching CPU architecture.
- `darwin-x64`: candidate only; these exact binaries have not been executed on this target. Requires macOS with the matching CPU architecture.
- `linux-x64`: candidate only; these exact binaries have not been executed on this target. Requires x86_64 Linux and a glibc dynamic loader; not suitable for minimal Alpine.
- `linux-x64-musl`: candidate only; these exact binaries have not been executed on this target. Requires x86_64 musl Linux plus libstdc++ and libgcc; not fully static. These prerequisites are historical observations, not fresh native certification.
- `windows-x64`: candidate only; these exact binaries have not been executed on this target. Requires native 64-bit Windows.

Bun and Node are embedded/not required to run these standalone binaries. POSIX download commands need curl, shasum (macOS) or sha256sum (Linux), chmod and standard shell tools. Windows needs PowerShell with Invoke-WebRequest and Get-FileHash. Minimum OS/glibc versions are not certified by this preparation. Portrait generation additionally needs authorized user-supplied manifests/pins/connection settings and a supported provider; no provider calls were made here. `portrait-install --help` is unsupported by the current installer contract and returns INVALID_ARGUMENTS; use its documented --url, --sha256, --destination arguments.

## Exact executable pins

```text
f2929ed0c860a742b6a714bf043bc9ade7ce7121e95c1273634192a0e5889c31  portrait-generator-darwin-arm64
09cfe348b0d548bc7df2d9cc29d08b62bdb8bbbdae57b0913b03d07d65a4b9a9  portrait-install-darwin-arm64
e0e547d0ed81bc90d7edc6e8593b6db178d3f2a757ac6e85e61e751f66fe4b1a  portrait-generator-darwin-x64
7e549932aae99f101ea46884b62cfe4143c9fea1f0100e75f970d461553a2b97  portrait-install-darwin-x64
29ce9885d7a96499f767a76ddd488a133bde8122983ec6677ee990bce9a5eb9b  portrait-generator-linux-x64
993b42e198322604e881037c6a6dde460b1e65c6b36885e5e651e880545c7b1c  portrait-install-linux-x64
eb91ed26f8204512e5f5a476d91275678c37ae183fe68bc7239776977c07cb6e  portrait-generator-linux-x64-musl
282abfe8d603bc71aae838c7f281cb481f730bce338b8c60132440075f01ede5  portrait-install-linux-x64-musl
cb367825daef9b73bfd9f94a03fbda4b29db7f8e6f91e715cbe06ecc938d11c8  portrait-generator-windows-x64.exe
4e66f538b0019cbc471162a43812f48ab8e19da5e1fcb38e873cea4c7fda816f  portrait-install-windows-x64.exe
```

## POSIX direct download example (macOS arm64)

Do not run until the release is published and independently reviewed. Choose the matching asset and exact pin from the table for another POSIX target. This is direct installation, not the no-redirect bootstrap CLI; GitHub asset downloads require redirects.

```sh
set -eu
asset=portrait-generator-darwin-arm64
expected=f2929ed0c860a742b6a714bf043bc9ade7ce7121e95c1273634192a0e5889c31
url="https://github.com/islee23520/portrait-generator-releases/releases/download/portrait-v0.1.0/$asset"
mkdir -p "$HOME/.local/bin"
destination="$HOME/.local/bin/portrait-generator"
[ ! -e "$destination" ] || { printf "Destination already exists\n" >&2; exit 1; }
work=$(mktemp -d)
trap 'rm -rf "$work"' EXIT
curl --fail --show-error --location --proto "=https" --proto-redir "=https" --output "$work/$asset" "$url"
measured=$(shasum -a 256 "$work/$asset")
[ "${measured%% *}" = "$expected" ] || { printf "CHECKSUM_MISMATCH\n" >&2; exit 1; }
chmod 755 "$work/$asset"
# Stage on destination filesystem and use a hard link to fail if destination appeared.
staged=$(mktemp "$HOME/.local/bin/.portrait-download.XXXXXXXX")
trap 'rm -rf "$work"; rm -f "$staged"' EXIT
cp "$work/$asset" "$staged"
chmod 755 "$staged"
ln "$staged" "$destination"
"$destination" --help
```

For Linux replace `shasum -a 256` with `sha256sum`, select glibc or musl deliberately, and use the exact corresponding pin. Keep candidate status until native review passes.

## Windows direct download example

```powershell
$ErrorActionPreference = "Stop"
$asset = "portrait-generator-windows-x64.exe"
$expected = "cb367825daef9b73bfd9f94a03fbda4b29db7f8e6f91e715cbe06ecc938d11c8"
$url = "https://github.com/islee23520/portrait-generator-releases/releases/download/portrait-v0.1.0/$asset"
$destination = Join-Path $HOME "portrait-generator.exe"
if (Test-Path -LiteralPath $destination) { throw "DESTINATION_EXISTS" }
$staged = Join-Path $HOME (".portrait-download-" + [Guid]::NewGuid().ToString("N") + ".exe")
try {
  Invoke-WebRequest -Uri $url -OutFile $staged
  if ((Get-FileHash -Algorithm SHA256 -LiteralPath $staged).Hash.ToLowerInvariant() -ne $expected) { throw "CHECKSUM_MISMATCH" }
  [IO.File]::Move($staged, $destination)
  & $destination --help
  if ($LASTEXITCODE -ne 0) { throw "HELP_FAILED" }
} finally {
  if (Test-Path -LiteralPath $staged) { Remove-Item -LiteralPath $staged }
}
```

The install CLI binaries remain candidates alongside the generator. GitHub release assets redirect; the existing portrait-install and bootstrap contracts reject redirects. Do not pass GitHub release asset URLs straight to those installers without an independently reviewed download contract change or a direct HTTPS no-redirect endpoint. The direct-download examples above avoid that deployment mismatch.

## Source-change gate

This staging is bound to the pinned commit and package tree identities in the private manifest. If generator, acceptance, dependency or build inputs change during parallel work, do not promote this staging as current: rebuild, repin and re-review. A moved branch alone does not rewrite the identities of these retained candidates.

