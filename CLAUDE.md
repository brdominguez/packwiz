# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Packwiz is a Go CLI tool for creating and managing Minecraft modpacks. It uses TOML metadata files instead of JAR files, enabling git-friendly version control. Supports CurseForge and Modrinth for mod sourcing, and can export to both platforms' pack formats.

## Build Commands

```bash
# Build the project
go build

# Install globally
go install

# Run directly
go run .

# Tidy dependencies
go mod tidy
```

No test suite exists in this codebase.

## Architecture

### Plugin Registration Pattern

Modules register themselves via `init()` functions using blank imports in `main.go`:

```go
// main.go
_ "github.com/packwiz/packwiz/curseforge"
_ "github.com/packwiz/packwiz/modrinth"
```

Each module registers:
- CLI commands via `cmd.Add(myCommand)`
- Updaters via `core.Updaters["source"] = myUpdater{}`
- MetaDownloaders via `core.MetaDownloaders["source"] = myDownloader{}`

### Core Interfaces (core/interfaces.go)

**Updater** - Checks for and applies mod updates:
- `ParseUpdate()` - Parse update config from TOML
- `CheckUpdate()` - Check for available updates
- `DoUpdate()` - Apply the update

**MetaDownloader** - Downloads mods using metadata services (CurseForge API, direct URLs)

### Data Model

Three-layer TOML structure:
1. **Pack** (`pack.toml`) - Modpack metadata, Minecraft version, mod loaders
2. **Index** (`index.toml`) - File inventory with SHA256 hashes
3. **Mod** (`.pw.toml` files) - Individual mod metadata with download/update sources

### Key Directories

- `cmd/` - Cobra CLI commands (init, update, refresh, serve, list, etc.)
- `core/` - Data structures (Pack, Mod, Index) and interfaces
- `curseforge/` - CurseForge API integration and import/export
- `modrinth/` - Modrinth API integration and export
- `cmdshared/` - Shared CLI utilities (prompts, progress bars)

### Configuration Priority

1. CLI flags (`--pack-file`, `--cache`, `-y`)
2. Pack options (`pack.toml [options]` table)
3. Local config file (`~/.packwiz/.packwiz.toml`)
4. Environment variables (`PACKWIZ_*` prefix)

## Key Dependencies

- `github.com/spf13/cobra` - CLI framework
- `github.com/spf13/viper` - Configuration management
- `github.com/BurntSushi/toml` - TOML parsing
- `codeberg.org/jmansfield/go-modrinth` - Modrinth API client
- `github.com/unascribed/FlexVer/go/flexver` - Version comparison

## Common Patterns

### Adding a New Mod Source

1. Create a new package (e.g., `newsource/`)
2. Implement `Updater` and/or `MetaDownloader` interfaces
3. Register in `init()`: `core.Updaters["newsource"] = newsourceUpdater{}`
4. Add blank import to `main.go`
5. Add CLI commands via `cmd.Add()`

### Working with Pack Data

```go
pack, err := core.LoadPack()           // Load pack.toml
index, err := pack.LoadIndex()         // Load index.toml
mods, err := index.LoadAllMods()       // Load all .pw.toml files
mod.Write()                            // Save mod changes
index.Write()                          // Save index
pack.UpdateIndexHash()                 // Update hash in pack.toml
```
