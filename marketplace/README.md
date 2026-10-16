# CDP Marketplace Source

This directory contains the Marketplace packages published independently of Creator Dashboard Pro desktop releases.

## Compatibility

- `catalog.json` is a **widgets-only** catalog for CDP 0.2.8. **Do not put `kind: "integration"` entries here**: CDP 0.2.8 rejects unknown kinds and could lose Marketplace catalog availability.
- `catalog-v2.json` is the catalog read by CDP 0.2.9 and newer. It includes widgets and externally versioned integration manifests.
- `marketplace-test/catalog.json` on development branches is a separate, opt-in test channel; it does not replace the production catalog.

When publishing a new widget that should work in 0.2.8, add it to both production catalogs with the same immutable widget asset URL and SHA-256. New integrations belong in `catalog-v2.json` only.

## Versioned widget rollback

CDP 0.2.9 uses a dedicated Marketplace signing key, separate from the desktop updater signing key. Production publishing requires `MARKETPLACE_SIGNING_PRIVATE_KEY` and `MARKETPLACE_SIGNING_PRIVATE_KEY_PASSWORD`; production desktop builds embed the matching public key from `MARKETPLACE_SIGNING_PUBLIC_KEY`.

The 0.2.9 Marketplace can expose older immutable widget packages with an optional `previousReleases` array on the current widget manifest:

```json
"previousReleases": [{
  "version": "1.0.0",
  "entryUrl": "https://raw.githubusercontent.com/dylantheoz-dev/cdp-releases/main/marketplace/widgets/cdp.viewer-queue/1.0.0/index.html",
  "integritySha256": "9a79b4ebcf7333994c7be5e724e2dbd2572de67310386c08fecab9e931154df7",
  "minimumCdpVersion": "0.2.8"
}]
```

A viewer who installed an older version and later updated also receives a *local manifest history* for one-click restoration, even if the current catalog does not publish the older version. The Marketplace offers a "Downgrade to" version picker when at least one older release is available.

Before a downgrade or update, the desktop host fetches and SHA-256 verifies the target package and checks CDP compatibility. It changes the installed version only after verification succeeds. Widget instance storage is retained on rollback; changing versions does not guarantee compatibility of settings between releases, so package authors must make migrations backward-compatible where possible. The exact old HTML URL must remain hosted; never overwrite immutable version paths. Downgrades do not automatically disable future update availability.

For local development testing, Viewer Queue v1.0.1 has v1.0.0 listed in `previousReleases`, while the 0.2.8 source catalog remains on v1.0.0. **Source catalog entries do not mean the files are publicly available.** For isolated external tester access, stage both immutable releases in `marketplace-test/` and publish the test channel only after approval; see `docs/0.2.9-marketplace-rollback-test.md`.

## Supported Marketplace integration manifests

CDP 0.2.9 supports three strictly allowlisted **configuration packages** with native hosts still bundled in the desktop app:

- `cdp.discord-dock` → `integrationId: discord`, `host: browser-dock`, exact URL `https://discord.com/app`.
- `cdp.mix-it-up` → `integrationId: mixitup`, `host: mix-it-up`, local Developer API bridge.
- `cdp.stream-deck` → `integrationId: streamdeck`, `host: stream-deck`, existing local plugin bridge.

These manifests do not download arbitrary native or web code into a privileged context. Installing/removing and updating a manifest is independent of CDP's installer, but changes to the native connector implementation or to the external Stream Deck plugin still require a separately tested binary/plugin release. Do not claim native-code updates are delivered through these manifests.

## Signed catalog and sandbox security

CDP 0.2.9 production builds require `catalog-v2.json.sig` next to `catalog-v2.json`. The manual Marketplace publish workflow signs the exact catalog bytes with the existing Tauri/Minisign release signing key. CDP verifies that signature with its embedded public key before parsing or trusting the catalog. A missing, invalid, or tampered signature fails closed. Development/test catalog overrides remain unsigned by design so isolated branches can be tested without publishing production metadata.

Downloaded widget HTML is still treated as untrusted after the catalog is verified. CDP enforces the package SHA-256, caps package size, runs widgets in a script-only sandbox without same-origin access, injects a restrictive CSP before package head content, blocks network/form/frame/plugin access, and gives each widget instance a size-limited storage namespace. Marketplace packages do not receive Tauri/native privileges.

Destructive actions inside sandboxed widgets should use inline/two-step confirmations. Do not add `allow-modals` or `allow-same-origin` to the Marketplace iframe simply to support browser dialogs.

## Security and update boundaries

Widget packages are HTML/CSS/JavaScript hosted in sandboxed iframes. The desktop host verifies SHA-256 of the HTML before running it. Widgets receive only the documented `cdp:init`, `cdp:storage:*`, and approved host messages, never arbitrary Tauri/native privileges.

Integration manifests are **data/configuration**, not downloadable native code. CDP 0.2.9's Browser Dock host is built into the app. Browser-dock packages are strictly allowlisted by integration ID and destination origin to protect persistent sessions. Changes to the native host itself still require a desktop app update, while manifest metadata/versioning can update independently.

Publish workflows must not run on every commit. Do not release CDP desktop builds or publish new public integration manifests without approval.
