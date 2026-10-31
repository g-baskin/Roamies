# AGENTS.md

Roamies: a Tauri 2 desktop app (Rust shell in `showcase/tauri/src-tauri`), a Node sidecar that runs the
agent (`showcase/tauri/sidecar`), shared browser JS in `showcase/roamies`, macOS Swift helpers built by
`src-tauri/build.rs`, and the Windows voice helper (`src-tauri/voice-win`, Rust + Parakeet) built by
`build.sh` for Windows targets. It compiles elsewhere too: `--transcribe <wav>` and `--test-audio <wav>`
test speech without a Windows PC (see the top of its `main.rs`).

## Commands

```sh
cd showcase/tauri/sidecar && npm ci     # install sidecar deps
npm test                                # sidecar tests (reply stream, storage, pointing, safety, voice model download)
npm run build                           # bundle the sidecar into dist/
cd .. && ./build.sh dev                 # web files + sidecar + debug `cargo build`
./build.sh                              # release bundle for this OS (signs on macOS if a Developer ID is in the keychain)
```

There is no linter configured yet.

Releases: push a `v*` tag (or run `.github/workflows/release.yml` by hand). It builds a signed, notarized
macOS DMG (Apple silicon and Intel) and an unsigned Windows installer into a draft GitHub release.

## Running the app locally (macOS)

- Don't test permissions with `target/debug/roamies` run from an agent's shell: it's unsigned and macOS
  credits Microphone, Screen Recording, Accessibility and Files to the parent app (e.g. GG Coder), so
  switches in System Settings never stick.
- Instead: `TAURI_BUNDLES=app ./build.sh` (about 20 min; needs `cargo install tauri-cli --version "^2" --locked`),
  then `ditto src-tauri/target/release/bundle/macos/Roamies.app /Applications/Roamies.app` and
  `open /Applications/Roamies.app`.
- The build ends with "no private key" (`TAURI_SIGNING_PRIVATE_KEY`) when signing the updater archive; that's
  expected locally and `Roamies.app` is still built.
- Without a Developer ID certificate the app is ad-hoc signed, so every rebuild resets its permissions.
- Settings live in `~/.gg/sidekick.json`; once `onboarded` is true, rerun setup from the menu bar's "Setup Guide…".

## Rules

- Every tool that touches files, commands, URLs or other apps goes through `showcase/tauri/sidecar/safety.mjs`
  (tested in `test/safety.test.mjs`); the macOS helper also refuses to type or click in the apps in `HANDS_OFF`.
  Add a test case with every rule change.
- In-app updates read `latest.json` from the newest published release, signed with the key in the
  `TAURI_SIGNING_PRIVATE_KEY` secret. Keep `plugins.updater.pubkey` in `tauri.conf.json` matching it.

- CI lives in `.github/workflows/ci.yml` and must stay green.
- Never commit with `--no-verify`.
- Never commit experiments (prototypes, test pages, design tools). They stay local and are listed in `.git/info/exclude`; check `git status` before every commit.
- Never commit secrets (Apple ID passwords, certificates, `~/.gg/auth.json`); signing credentials go in GitHub Actions secrets.
