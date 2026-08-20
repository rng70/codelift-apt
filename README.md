# codelift APT repository

Signed Debian/Ubuntu packages for [**codelift**](https://github.com/rng70/codelift) —
a tool that downloads a package from its registry, inspects its source or metadata, and
prints every fully qualified function, method and class name it contains, across npm,
PyPI, NuGet, Maven, Cargo, Go and RubyGems.

This repository holds **only the published packages and their indices**. It is generated
and pushed by `scripts/release.sh` in the upstream repository; nothing here is edited by
hand.

## Install

```bash
curl -fsSL https://rng70.github.io/codelift-apt/install.sh | sudo sh
sudo apt install codelift
```

The script adds the signing key to `/etc/apt/keyrings/codelift.gpg`, writes
`/etc/apt/sources.list.d/codelift.list`, and runs `apt update`. If you would rather do
it by hand:

```bash
sudo install -d -m 0755 /etc/apt/keyrings
curl -fsSL https://rng70.github.io/codelift-apt/codelift.asc \
  | sudo gpg --dearmor -o /etc/apt/keyrings/codelift.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/codelift.gpg]" \
     "https://rng70.github.io/codelift-apt stable main" \
  | sudo tee /etc/apt/sources.list.d/codelift.list
sudo apt update
```

## Packages

| Package | Contents | Dependencies |
|---------|----------|--------------|
| `codelift` | metapackage — installs the CLI build | — |
| `codelift-cli` | fully static build, GUI compiled out | none at all |
| `codelift-gui` | GUI build, plus a desktop entry and icon | X11/Wayland/OpenGL runtime libraries |

`codelift-cli` and `codelift-gui` both provide `/usr/bin/codelift` and both declare
`Provides: codelift-bin`, so they are **mutually exclusive** — installing one replaces
the other:

```bash
sudo apt install codelift          # → codelift-cli, no X11 dependencies at all
sudo apt install codelift-gui      # → replaces codelift-cli, adds the desktop app
```

`apt install codelift` is equally satisfied by an existing `codelift-gui` install, so it
will not swap the GUI build out from under you.

Pick `codelift-cli` for servers, containers and CI: it is statically linked and pulls in
nothing. Pick `codelift-gui` for a workstation — it needs the X11/Wayland and OpenGL
client libraries, which apt installs for you.

## Supported releases

| Architecture | Debian | Ubuntu |
|--------------|--------|--------|
| `amd64`, `arm64`, `armhf` | 8 (jessie) and newer | 14.04 and newer |

One set of packages serves every release: the GUI build has its glibc floor pinned to
2.17 with [zig](https://ziglang.org/) as the linker, and the CLI build is fully static,
so there is nothing to rebuild per distribution. The repository publishes a single
`stable` suite with one `main` component.

## Verifying the signature

The `Release` file is signed both inline (`InRelease`) and detached (`Release.gpg`) with
this key:

```
pub   rsa4096 2026-08-20 [SC]
      6821 D7E0 05D1 7578 DADD  21E9 DC5B FD15 6020 B191
uid   codelift APT repository signing key <tanin@openrefactory.com>
```

To check it before trusting anything:

```bash
curl -fsSL https://rng70.github.io/codelift-apt/codelift.asc | gpg --show-keys --fingerprint
```

This key signs the repository indices only. It is not a code-signing key and says nothing
about the provenance of the binaries beyond "this repository published them".

## Uninstalling

```bash
sudo apt purge codelift codelift-cli codelift-gui
sudo rm /etc/apt/sources.list.d/codelift.list /etc/apt/keyrings/codelift.gpg
sudo apt update
```

## Layout

```
dists/stable/Release            signed index (InRelease + Release.gpg alongside)
dists/stable/main/binary-amd64/ Packages, Packages.gz, Packages.xz
dists/stable/main/binary-arm64/
dists/stable/main/binary-armhf/
pool/main/c/codelift/           the .deb files themselves
codelift.asc                    the repository signing key, ASCII-armored
install.sh                      the bootstrap script served to end users
```

The repository is cumulative: indices are regenerated over whatever the pool already
contains, so previously published versions stay installable.

## Other formats

`.rpm` packages and an `AppImage` of the GUI are attached to each
[GitHub release](https://github.com/rng70/codelift/releases), along with plain `.tar.gz`
binaries for Linux, macOS and Windows.

## Source and licence

codelift is licensed **GPL-3.0-or-later**; the same licence covers the packaging files in
this repository, and its full text is in [LICENSE](LICENSE). The corresponding source for
the binaries published here lives in the upstream repository above. If you have received a
binary from this repository and cannot access that repository, you are entitled to the
corresponding source under section 6 of the GPL — request it from
<tanin@openrefactory.com>.
