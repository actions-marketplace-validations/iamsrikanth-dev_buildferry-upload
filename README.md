# BuildFerry Upload

Send every iOS and Android build from GitHub Actions to your testers. [BuildFerry](https://buildferry.app) registers new iPhones with Apple and re-signs the .ipa for them, and serves your .apk on the same link, so testers install from one link and new builds reach it without anyone doing anything.

## Usage

1. Create a token at [buildferry.app/ci](https://buildferry.app/ci).
2. Add it as a repository secret named `BUILDFERRY_TOKEN` (Settings → Secrets and variables → Actions).
3. Add this step after the one that exports your Ad Hoc or Development `.ipa`:

```yaml
- name: Send to testers
  id: buildferry
  uses: iamsrikanth-dev/buildferry-upload@v1
  with:
    token: ${{ secrets.BUILDFERRY_TOKEN }}

- run: echo "Install from ${{ steps.buildferry.outputs.share-url }}"
```

With no `ipa:` input, the newest `.ipa` in the workspace is uploaded.

### Android

Pass the path of a signed `.apk`. It joins the app whose iOS bundle ID matches its package name, so iPhones and Android phones use the same link:

```yaml
- uses: iamsrikanth-dev/buildferry-upload@v1
  with:
    token: ${{ secrets.BUILDFERRY_TOKEN }}
    ipa: app/build/outputs/apk/release/app-release.apk
```

Android App Bundles (`.aab`) can't be installed directly, so build an `.apk` (`./gradlew assembleRelease`).

## Inputs

| Input | Default | |
| --- | --- | --- |
| `token` | (required) | BuildFerry API token |
| `ipa` | newest `.ipa` | Path to the build: an `.ipa`, or an Android `.apk` |
| `wait` | `true` | Wait for BuildFerry to check the build, so the step fails when it can't be shared |
| `timeout` | `600` | Seconds to wait for the check |
| `notes` | | What's new / what to test, shown to testers on the install page |
| `notes-file` | | Path to a file with the notes, such as a changelog |
| `url` | `https://buildferry.app` | BuildFerry address |

## Outputs

`status` (`ok`, `duplicate`, `uploaded`, `incompatible` or `failed`), `share-url`, `app-url`, `version`, `build` and `testers`. The step also writes a job summary with the share link.

## Without the Action

Any CI with Node 18 or newer can run the same thing:

```bash
BUILDFERRY_TOKEN=... npx --yes buildferry@1 upload path/to/App.ipa
```

MIT licensed.
