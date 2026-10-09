# Spicetify CLI reliability patch

This is a fork-ready patch for `spicetify/cli`, focused on making `apply` more predictable around Spotify version changes.

## What it changes

1. **Passes the actual Spotify version into additional-option injection.** The current call site builds `apply.Flag` with `SpicetifyVer` but omits `SpotifyVer`, even though `AdditionalOptions` uses `SpotifyVer` to choose the JavaScript bundle for experimental features. That leaves the selector at `0.0.0` and pushes the modification toward the legacy `vendor~xpui.js` path.
2. **Compares versions numerically and lexicographically.** Independent checks such as `minor >= 2 && patch >= 57` misclassify future versions like `1.3.0`. The new helper compares major, minor, and patch in order and ignores later Spotify build metadata.
3. **Preflights before the destructive copy.** It checks that the extracted `xpui/index.html`, selected helper files, and custom-app injection target exist before `Apply` clears or overwrites the app directory.
4. **Stops reporting a successful apply when required helper writes fail.** Those errors are returned to the caller instead of only being printed while the CLI continues.

## Apply it to a fork

```bash
git clone https://github.com/spicetify/cli.git
cd cli
git checkout -b reliability/apply-preflight
# Put reliability.patch in this directory, then:
git apply --check reliability.patch
git apply reliability.patch
gofmt -w src/utils/version.go src/utils/version_test.go src/apply/apply.go src/cmd/apply.go src/preprocess/preprocess.go
go test ./src/utils ./src/apply ./src/preprocess ./src/cmd
```

Review the diff before building or installing it. If your local upstream has moved since this patch was prepared, `git apply --check` will identify context conflicts; resolve those against your checked-out source instead of forcing the patch.

## Scope and limitations

This is a patch set, **not a published GitHub fork**. It cannot guarantee every Spotify update will remain compatible: Spotify changes bundled JavaScript frequently, and custom themes/extensions can independently break the client. The preflight aims to fail early with a useful error rather than apply known-incompatible custom-app changes and then print success.

The patch was prepared against the public upstream source viewed on 2026-10-09. The container used for this session could not clone GitHub, so the full upstream integration build has not been run here. The standalone version comparator tests are intended to run with the repository's `go test` command above.
