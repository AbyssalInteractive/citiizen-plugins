# citiizen-plugins

Central registry of **Abyssal-certified plugins** for the Citiizen game server.

The Citiizen launcher fetches [`manifest.json`](manifest.json) from this repo's
`main` branch to populate the plugin-picker in the server-creation wizard and
the **Add plugin** modal on the server-edit page.

> Want to see the bigger picture? The plugin host that loads these DLLs lives in
> [`AbyssalInteractive/Citiizen.PluginHost`](https://github.com/AbyssalInteractive/Citiizen.PluginHost).
> The contract assembly plugins reference is `Citiizen.Api`.

## Layout

```
citiizen-plugins/
├── manifest.json          ← the registry consumed by the launcher
├── manifest.schema.json   ← JSON Schema for validation
├── LICENSE
└── README.md
```

Plugin **source code** does NOT live here. Each plugin has its own repository
(e.g. [`AbyssalInteractive/citiizen-plugin-sql-adminer`](https://github.com/AbyssalInteractive/citiizen-plugin-sql-adminer)),
publishes versioned GitHub Releases, and is referenced from `manifest.json` by URL.

## How the launcher consumes the manifest

At runtime, [`PluginManager`](https://github.com/AbyssalInteractive/Citiizen.Backend) inside the Citiizen client fetches:

```
https://raw.githubusercontent.com/AbyssalInteractive/citiizen-plugins/main/manifest.json
```

and caches the result for 5 minutes. Each entry's `downloadUrl` is fetched, the
SHA-256 is verified against the manifest's `sha256` field, and the zip is
extracted into `Servers/<server-name>/Plugins/<plugin-id>/` (subfolder, recursive
scan picks up multi-DLL plugins). A failed hash check **aborts the install**.

## Plugin entry schema

See [`manifest.schema.json`](manifest.schema.json) for the authoritative
definition. Quick reference:

| Field               | Required | Description                                                                            |
|---------------------|----------|----------------------------------------------------------------------------------------|
| `id`                | yes      | Unique reverse-DNS-ish identifier (e.g. `abyssal.sql-adminer`).                       |
| `name`              | yes      | Display name.                                                                          |
| `description`       | no       | One-paragraph description shown in the wizard.                                         |
| `author`            | no       | Author / org name.                                                                     |
| `certified`         | no       | `true` if maintained by Abyssal — pre-checked in the wizard.                          |
| `version`           | yes      | SemVer (e.g. `1.0.0`).                                                                 |
| `minBackendVersion` | no       | Minimum `Citiizen.Backend` version required.                                           |
| `downloadUrl`       | yes      | HTTPS URL to a `.zip` containing the plugin DLLs at the archive root.                  |
| `sha256`            | yes      | Lowercase hex SHA-256 of the zip — verified before extraction.                         |
| `homepage`          | no       | GitHub repo URL of the plugin source.                                                  |
| `iconUrl`           | no       | Square icon URL (PNG/SVG).                                                              |
| `commands`          | no       | Chat commands the plugin registers (pre-install collision detection in the launcher). |

## Adding / updating a plugin

1. Create / cut a release on the plugin's own repo (tag `v*.*.*`). The CI
   workflow uploads `<plugin>-<version>.zip` to the Release and prints its
   SHA-256 in the release notes.
2. Open a PR against this repo updating the corresponding entry in
   `manifest.json` with the new `version`, `downloadUrl`, and `sha256`.
3. Bump `registry.updatedAt`.
4. Merge to `main` — the launcher picks the new version up automatically within
   5 minutes (its in-memory cache TTL).

## Current plugins

- [`abyssal.sql-adminer`](https://github.com/AbyssalInteractive/citiizen-plugin-sql-adminer) — loopback-only PostgreSQL inspector.

## License

MIT — see [LICENSE](LICENSE).
