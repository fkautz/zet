# Nix build notes

## Build a package in nixpkgs

```sh
nix-build -E 'with import <nixpkgs> {}; callPackage ./default.nix {}'
```

## Generating sha256 hash for fetchFromGitHub, or what to do when the hash doesn't match.

Make sure to use the `--unpack` when you fetch from github.

```sh
nix-prefetch-url --unpack https://github.com/spiffe/spire/archive/refs/tags/v1.1.3.tar.gz
```

## Generating the vendorSha256

### Go with modules
It's easiest to pull them from the failed build. The hash includes the unpacked vendor directory.
