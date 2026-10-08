# semver

Parse, order and match [Semantic Versioning 2.0](https://semver.org) versions,
and version requirements written in Cargo's syntax, for
[Meadow](https://github.com/meadow-lang/meadow).

This package is a port of Rust's [`semver`](https://github.com/dtolnay/semver)
1.0.28 by David Tolnay. It follows the crate's rules exactly, error messages
included.

## Install

```sh
meadow add meadow-lang/Semver
```

## Use

```meadow
use Semver

def main =
  match (parseVersionReq ">=1.2.3, <1.8.0", parseVersion "1.4.0") with
  | (Ok req, Ok v) -> (matches req v, versionReqText req)
  -- (True, ">=1.2.3, <1.8.0")
  | _ -> (False, "")
```

| function | |
|---|---|
| `parseVersion s` | `Ok Version`, or `Err Error` |
| `parseVersionReq s` | `Ok VersionReq`: comma-separated comparators such as `>=1.2`, `~1.4.2`, `^0.3`, `1.*`, or `*` |
| `parseComparator s` | `Ok Comparator`: a single comparator |
| `prerelease s`, `buildMetadata s` | `Ok s` if `s` is a valid pre-release or build identifier |
| `version major minor patch` | a plain `Version` from three numbers |
| `matches req v` | whether `v` satisfies every comparator in `req` |
| `matchesComparator c v` | whether `v` satisfies the one comparator `c` |
| `compareVersions a b` | `Less`, `Equal` or `Greater`, with build metadata as the final tie-breaker |
| `comparePrecedence a b` | the same, ignoring build metadata, as SemVer specifies |
| `comparePrerelease`, `compareBuildMetadata` | order the two identifier kinds on their own |
| `versionText`, `versionReqText`, `comparatorText` | format as text |
| `errorMessage e` | the crate's error message for `e` |
| `star` | the requirement `*` |

`Version` is a record: `major`, `minor` and `patch` are `UInt64`s, and `pre`
and `build` are strings that are empty when the version has none. In the crate
these two are wrapper types; here they are plain strings. Use the `compare…`
functions to order them, because ordinary string comparison is wrong for them
(`alpha.10` must come after `alpha.9`).

Requirements follow Cargo's rules:

- A bare `1.2.3` means `^1.2.3`.
- A pre-release version such as `1.4.0-rc.1` satisfies a requirement only if one
  of its comparators names that same release with a pre-release of its own.
  For example, `>=1.2.3` does not match `1.4.0-rc.1`.

`Op`'s constructors are written `Op.Caret`, `Op.Less` and so on. They are not
exported bare, because `Less` would clash with `Ordering`'s.

## How it's made

`src/` is a hand translation of the crate. **`src/Cases.mw`** is generated test
data drawn from every string literal in the crate's own tests, plus random
strings, versions and requirements:

- 2,502 strings, each parsed as a version, a requirement, a comparator, a
  pre-release and build metadata;
- 3,000 pairs of versions, compared four ways;
- 7,000 requirement–version pairs, matched.

Every expected answer, including each error message, comes from calling the
crate, and `meadow test` checks the port against all of them.

To regenerate, run `scripts/generate.sh`. It needs a Rust toolchain. If the
crate changed, the generator stops so that `src/` can be updated first.

## Licence

Dual-licensed under [Apache-2.0](LICENSE-APACHE) or [MIT](LICENSE-MIT), at your
option, like the crate. See [COPYRIGHT](COPYRIGHT).
