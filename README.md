# .github

Shared defaults for this organisation.

`default.json` is the Renovate preset every repository here extends with
`{"extends": ["github>go-richdoc/.github"]}`.

`.github/workflows/vulns.yml` is the organisation's govulncheck lane, defined
once. A module calls it with two lines:

```yaml
  vulns:
    uses: go-richdoc/.github/.github/workflows/vulns.yml@main
```

It runs `govulncheck` and hands the JSON stream to `vulngate` rather than
believing the exit status, which answers a different question — non-zero over
findings it has itself decided the module does not *call*, zero over some it
does.

It scans under the caller's own `go.mod`, because the standard library's
findings are decided by one version and the version a module declares is the
one its consumers may build it with: identical source reports eight
standard-library advisories under `go1.27.1` and none under `go1.27.2`. Pass
`with: {go-version: "1.27.x"}` only to ask a different question on purpose.

It exists because on 2026-11-08 none of this organisation's ten Go modules ran
`govulncheck` in CI, while three advisories had already been fixed in one of
them by hand — the last of which, GO-2026-6888, was *reachable*
(`pdf.WriteTo → engine.CompileToPDF → brotli.Reader.Read`, unbounded allocation
on a crafted stream). The fix had been an explicit `brotli v1.2.7` requirement,
and `go mod tidy` correctly removed it the morning a newer engine made it
redundant. A pin that `tidy` may retire is not a control.

Minor and patch Go dependency updates merge themselves once the tests pass.
**The Go toolchain does not.** `go1.27.0` miscompiles on loong64
([golang/go#81000](https://github.com/golang/go/issues/81000), bisected on real
hardware to `3fdac6780b`), and a repository without a loong64 lane sees nothing
wrong with the bump. A library that breaks is one library; a toolchain that
breaks is every architecture at once.
