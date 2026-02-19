# kunobi-release

Public release metadata for [Kunobi](https://kunobi.ninja). This repo enables version discovery for package managers (mise/aqua) while keeping the source repository private.

## Install Kunobi

### Using mise

```bash
mise use kunobi
```

### Using aqua

```bash
aqua g -i kunobi-ninja/kunobi-release
```

### Manual download

Download the release assets JSON and verify checksums manually:

```bash
# Download the release assets JSON and its checksum
VERSION="0.1.0"
curl -fSL "https://r2.kunobi.ninja/v${VERSION}/Kunobi_${VERSION}_release_assets.json" -o release_assets.json
curl -fSL "https://r2.kunobi.ninja/v${VERSION}/Kunobi_${VERSION}_release_assets.json.sha256" -o release_assets.json.sha256

# Verify the JSON integrity
sha256sum -c release_assets.json.sha256

# List available platforms
jq -r '.platforms | keys[]' release_assets.json

# Download a specific artifact (e.g. darwin-aarch64)
URL=$(jq -r '.platforms["darwin-aarch64"][0].url' release_assets.json)
curl -fSL "$URL" -o kunobi.app.tar.gz

# Verify artifact checksum
EXPECTED=$(jq -r '.platforms["darwin-aarch64"][0].sha256' release_assets.json)
echo "$EXPECTED  kunobi.app.tar.gz" | sha256sum -c -
```

## How it works

1. **Upstream trigger** - When a new Kunobi version is published, a `repository_dispatch` event triggers the `update-release` workflow.
2. **Validation** - The release assets JSON is validated (structure, SHA256 checksum, R2 URL accessibility).
3. **Pull request** - A PR is created automatically for review.
4. **Release** - On merge to `main`, a GitHub release is created with the versioned files.

## Repository structure

The repo maintains two files that are overwritten on each update:

- `release_assets.json` - Current release metadata listing all platform artifacts with SHA256 checksums
- `release_assets.json.sha256` - SHA256 checksum of the JSON file itself

## Release structure

Each GitHub release (tagged `v{version}`) contains versioned copies of the assets files:

- `Kunobi_{version}_release_assets.json` - Platform artifacts and checksums for the release
- `Kunobi_{version}_release_assets.json.sha256` - SHA256 checksum of the JSON file

The JSON file lists all downloadable artifacts grouped by platform, with their SHA256 checksums and R2 CDN URLs:

```json
{
  "version": "0.1.0",
  "channel": "stable",
  "pub_date": "2025-01-01T00:00:00Z",
  "platforms": {
    "darwin-aarch64": [
      { "sha256": "abc123...", "url": "https://r2.kunobi.ninja/v0.1.0/Kunobi_0.1.0_darwin-aarch64.app.tar.gz" }
    ],
    "darwin-x86_64": [
      { "sha256": "def456...", "url": "https://r2.kunobi.ninja/v0.1.0/Kunobi_0.1.0_darwin-x86_64.app.tar.gz" }
    ]
  }
}
```

## Workflows

| Workflow | Trigger | Purpose |
|----------|---------|---------|
| `update-release` | `repository_dispatch` / manual | Downloads assets from R2, creates validation PR |
| `validate-release` | PR / called by update | Validates JSON, checksum, and URL accessibility |
| `create-release` | Push to `main` | Creates tagged GitHub release with versioned files |
