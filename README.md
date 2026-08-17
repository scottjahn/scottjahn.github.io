# scottjahn.github.io

The user site for [github.com/scottjahn](https://github.com/scottjahn). It does two jobs:

1. Serves `/.well-known/assetlinks.json` for the Android apps built from the game repos.
2. Hosts a short overview of my projects at [scottjahn.github.io](https://scottjahn.github.io/).

## Digital Asset Links

Android verifies app links **per origin**, not per path. Every project here lives under one
origin — `https://scottjahn.github.io` — so there is exactly one asset links file, at:

```
https://scottjahn.github.io/.well-known/assetlinks.json
```

A file at `https://scottjahn.github.io/power-of-2/.well-known/assetlinks.json` is never
fetched by Android and will not verify anything.

### Combining two apps into one file

The file is a **JSON array of statements**. Merging two per-app files means concatenating
their arrays — one object per app, no other changes:

```json
[
  { "relation": ["..."], "target": { "package_name": "app.one", "...": "..." } },
  { "relation": ["..."], "target": { "package_name": "app.two", "...": "..." } }
]
```

Two rules to keep in mind:

- **Different package names → separate objects.** Each app gets its own statement, even
  though they share a domain.
- **Same package name, multiple certificates → one object, multiple fingerprints.**
  `sha256_cert_fingerprints` is an array, so a debug build and a Play-signed build of the
  same app belong in one statement together, not in two.

Both apps verify against the whole origin, which means each one is technically authorised
to handle any URL on `scottjahn.github.io`. In practice each APK's TWA is scoped to its own
start URL and subdirectory, so they don't collide.

### Which fingerprint to use

If the app is distributed through Google Play with Play App Signing, use the **app signing
certificate** SHA-256 from Play Console → *Release* → *Setup* → *App integrity*, not the
upload certificate. Sideloaded APKs use whatever key actually signed the APK:

```bash
keytool -list -v -keystore my-release.keystore -alias my-alias | grep SHA256
```

### Verifying after deploy

```bash
curl -s https://scottjahn.github.io/.well-known/assetlinks.json | python3 -m json.tool
```

Then run it through Google's checker:

```
https://digitalassetlinks.googleapis.com/v1/statements:list?source.web.site=https://scottjahn.github.io&relation=delegate_permission/common.handle_all_urls
```

On a device, `adb shell pm verify-app-links --re-verify <package>` re-runs verification, and
`adb shell pm get-app-links <package>` shows the result.

## Why `.nojekyll`

GitHub Pages runs Jekyll by default, and Jekyll skips files and directories whose names start
with a dot — which would silently drop `.well-known/` from the published site. The empty
`.nojekyll` file at the repo root turns Jekyll off so the directory is published as-is.
