# CDP Marketplace Source

This folder is the source of truth for Creator Dashboard Pro marketplace content.

Changes under `marketplace/` can be published independently of a desktop app release. The `publish-marketplace.yml` workflow mirrors this folder to the public `dylantheoz-dev/cdp-releases` repository, where installed copies of CDP read the live catalog.

## Current package model

Marketplace widgets are HTML/CSS/JavaScript documents loaded inside a sandboxed iframe by CDP. They do not receive Tauri/native privileges. CDP verifies the published SHA-256 value before rendering the widget.

The catalog is intentionally versioned so a future signed-package format can be added without breaking existing installations.
