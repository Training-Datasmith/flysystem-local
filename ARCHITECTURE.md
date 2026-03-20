# Architecture: flysystem-local

## Purpose

The official Flysystem adapter for local filesystem storage. It wraps PHP's native filesystem functions behind the `FilesystemAdapter` interface, handling path normalisation, visibility (Unix permissions), MIME type detection, and error translation into Flysystem exceptions.

## Directory Structure

```
LocalFilesystemAdapter.php      — Primary adapter: all read/write/delete/list operations on local disk
FallbackMimeTypeDetector.php    — MIME detector that tries extension-based lookup before finfo for speed
LocalFilesystemAdapterTest.php  — Adapter contract tests
```

## Key Design Decisions

- **PathPrefixer** — all paths are prefixed with the configured root; prevents path traversal outside the root.
- **Visibility via Unix permissions** — maps Flysystem `public`/`private` to configurable octal modes for both files and directories (default: `0644`/`0755` public, `0600`/`0700` private).
- **Recursive directory creation** — writes automatically create missing parent directories with the configured directory permissions.
- **MIME detection fallback** — `FallbackMimeTypeDetector` checks the extension first (fast) and falls back to `finfo` only when needed, balancing accuracy and performance.
- **DirectoryIterator-based listing** — uses `RecursiveDirectoryIterator` for recursive listing; `UnixVisibility` extracts permissions from `SplFileInfo::getPerms()`.

## Extension Points

- Inject a custom `MimeTypeDetector` to override MIME resolution logic.
- Inject a custom `PortableVisibilityConverter` to customise file/directory permission mappings.

## Dependency Flow

```
LocalFilesystemAdapter
  ├── PathPrefixer (path normalisation + root enforcement)
  ├── PortableVisibilityConverter (permission ↔ visibility mapping)
  └── MimeTypeDetector (MIME type for metadata)
        └── FallbackMimeTypeDetector
              ├── ExtensionMimeTypeDetector (fast path)
              └── FinfoMimeTypeDetector (fallback)
```
