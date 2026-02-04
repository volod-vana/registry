# Connectors Registry

This repository serves as the registry for DataBridge connectors. It enables over-the-air (OTA) updates for connectors without requiring users to update the entire application.

## How It Works

### Registry Structure

```
registry/
├── registry.json           # Manifest listing all available connectors
├── openai/
│   ├── chatgpt-playwright.js      # Connector script
│   └── chatgpt-playwright.json    # Connector metadata
├── meta/
│   ├── instagram-playwright.js
│   └── instagram-playwright.json
└── linkedin/
    ├── linkedin-playwright.js
    └── linkedin-playwright.json
```

### registry.json

The manifest file contains:

```json
{
  "version": "1.0.0",
  "lastUpdated": "2026-02-03T00:00:00Z",
  "baseUrl": "https://raw.githubusercontent.com/.../main",
  "connectors": [
    {
      "id": "chatgpt-playwright",
      "company": "openai",
      "version": "1.0.0",
      "name": "ChatGPT",
      "description": "...",
      "files": {
        "script": "openai/chatgpt-playwright.js",
        "metadata": "openai/chatgpt-playwright.json"
      },
      "checksums": {
        "script": "sha256:...",
        "metadata": "sha256:..."
      }
    }
  ]
}
```

### Update Flow

1. **App checks for updates** - Fetches `registry.json` and compares versions with locally installed connectors
2. **Download if newer** - Downloads script and metadata files from `baseUrl + files.script/metadata`
3. **Verify integrity** - Validates SHA256 checksums before installing
4. **Install locally** - Saves to user's `~/.databridge/connectors/` directory

### Connector Files

Each connector consists of two files:

- **`{id}.js`** - The connector script executed by the Playwright runner
- **`{id}.json`** - Metadata including display name, description, and available scopes

## Publishing Updates

### 1. Update Connector Files

Place updated `.js` and `.json` files in the appropriate company directory.

### 2. Generate Checksums

```bash
shasum -a 256 openai/chatgpt-playwright.js
shasum -a 256 openai/chatgpt-playwright.json
```

### 3. Update registry.json

- Increment the connector's `version`
- Update the `checksums` with new SHA256 hashes
- Update `lastUpdated` timestamp
- Optionally increment the registry `version`

### 4. Commit and Push

```bash
git add .
git commit -m "chore: update chatgpt connector to v1.1.0"
git push
```

Users will receive the update on their next app launch or manual update check.

## Connector Development

Connectors are JavaScript files that run in the Playwright runner environment. They have access to:

- `page` - Playwright Page object for browser automation
- `sendUpdate(data)` - Send progress updates to the app
- `complete(data)` - Signal completion with final data

See the existing connectors for implementation examples.

## Security

- All connector files are verified via SHA256 checksums before execution
- Connectors run in a sandboxed Playwright browser context
- User credentials are never stored by connectors
