## GitHub Repository Extension Build Process

When building extensions from GitHub repositories, the action follows these steps:

1. Clone the repository with the specified branch, tag, or commit hash
2. Attempt to build the extension using one of the following methods (in order):
   - CMake build system (if CMakeLists.txt is found)
   - Custom build script (`build.sh`)
   - Makefile (if Makefile is found and CMakeLists.txt is not)
3. Find and copy the built extension files to the output directory

The action looks for extension files with the following patterns:
- Files with `.duckdb_extension` extension
- Shared libraries (`.so`, `.dll`, `.dylib`)

### Tips for GitHub Extension Repositories

For best results with GitHub repository extensions:
- Ensure your repository has a clear build system (CMake, build.sh, or Makefile)
- Place built extension files in a standard location (like `build/extension/`)
- Make sure dependencies are handled in a build script or via `scripts/install_dependencies.sh`# DuckDB Extension Manager

A GitHub Action for downloading and caching DuckDB extensions from various sources.

## Features

- Downloads DuckDB extensions from multiple sources:
  - Official DuckDB core extensions
  - DuckDB core nightly extensions
  - Custom repositories with specific tags or commits
  - Direct URLs to extension files
- Caches extensions to improve workflow performance
- Supports custom cache paths and keys

## Usage

### Basic Example

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Download DuckDB Extensions
        uses: your-username/duckdb-extension-manager@v1
        with:
          extensions: |
            [
              {"name": "json", "source": "core"},
              {"name": "httpfs", "source": "core", "version": "v0.8.1"},
              {"name": "spatial", "source": "core_nightly"},
              {"name": "custom_ext", "source": "https://github.com/username/custom-duckdb-extension.git", "version": "v1.0.0"}
            ]
```

### Advanced Example

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Download DuckDB Extensions
        id: duckdb-extensions
        uses: your-username/duckdb-extension-manager@v1
        with:
          extensions: |
            [
              {"name": "json", "source": "core"},
              {"name": "httpfs", "source": "core"},
              {"name": "spatial", "source": "core_nightly"}
            ]
          duckdb-version: "0.9.0"
          cache-key: "my-custom-suffix"
          cache-path: "./custom/path/to/extensions"
          
      - name: Use the extensions
        run: |
          echo "Extensions downloaded to: ${{ steps.duckdb-extensions.outputs.extensions-path }}"
          echo "Cache hit: ${{ steps.duckdb-extensions.outputs.cache-hit }}"
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `extensions` | JSON list of extensions to download | Yes | |
| `duckdb-version` | DuckDB version to use | No | `latest` |
| `cache-key` | Custom cache key suffix | No | `` |
| `cache-path` | Path to store extensions | No | `.duckdb/extensions` |

### Extension Format

Each extension in the `extensions` array should have the following format:

```json
{
  "name": "extension_name",
  "source": "core|core_nightly|repository_url|direct_url",
  "version": "tag_or_hash", // Optional, defaults to "latest"
  "url": "https://direct-url-to-extension-file.zip" // Optional, only for direct URL downloads
}
```

The action supports four ways to specify extensions:

1. **Core extensions**: Use `source: "core"` for official DuckDB extensions
2. **Nightly extensions**: Use `source: "core_nightly"` for nightly builds
3. **GitHub repositories**: Use `source: "https://github.com/username/repo"` with `version` to specify tag/branch/commit
4. **Direct URLs**: Either:
   - Set `url: "https://direct-url-to-extension.zip"` (recommended)
   - Or use `source: "direct_url"` and put the URL in the `version` field (legacy support)


## Outputs

| Output | Description |
|--------|-------------|
| `cache-hit` | Boolean indicating if a cache hit occurred |
| `extensions-path` | Path where extensions are stored |

## License

MIT
