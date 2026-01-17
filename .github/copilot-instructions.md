# WWM Utils - Copilot Instructions

## Project Overview
Binary parser and packer for "Where Winds Meet" (WWM) game localization files. This CLI tool unpacks proprietary `.map` files into editable JSON and repacks them after modification.

## Architecture

### File Format Structure
- **MapHeader** (12 bytes): Magic `0xDEADBEEF`, version, entry count
- **Seek table**: Array of offsets to compressed blocks
- **Blocks**: Each contains a BlockHeader + zstd-compressed table data
- **Tables**: Table 0 is raw data; tables 1+ contain localization strings with hash-based lookup

### Core Workflow
1. **Unpack**: [core.rs](src/core.rs#L10) reads binary → extracts zstd blocks → parses string tables → outputs JSON + raw tables to `output/{filename}/`
2. **Pack**: [core.rs](src/core.rs#L130) reads modified JSON + original tables → recompresses → writes to `merged/` subdirectory

### Key Components
- [structs.rs](src/structs.rs): `#[repr(C, packed)]` binary layouts using `bytemuck` for zero-copy parsing
- [prelude.rs](src/prelude.rs): Imports from `porter-utils` crate for binary I/O (`StructReadExt`, `ArrayReadExt`, `BufferReadExt`)
- [utils/io_ext.rs](src/utils/io_ext.rs): `SeekReadExt` trait adds `read_struct_at()` for position-specific reads
- [main.rs](src/main.rs): Drag-and-drop interface - files unpack, directories pack

## Critical Patterns

### Binary I/O
```rust
// Reading packed structs
let header = reader.read_struct::<MapHeader>()?;
// Reading at specific offset
let entry = reader.read_struct_at::<TableEntry>(pos)?;
```
**Always** use `bytemuck::{Pod, Zeroable}` for structs read from disk. Use `#[repr(C, packed)]` to match game's layout.

### String Table Parsing
- Strings stored with 64-bit hash IDs
- Entry offset is relative to **current entry position** + 8 bytes (after id field)
- Skip entries where first byte is `0xFF` (invalid markers)
- Bucket size calculation: `((entry_count + 1) + 16).max(24)` (see [core.rs](src/core.rs#L57))

### Error Handling
Currently uses `.unwrap()` everywhere. **TODO** comment in [main.rs](src/main.rs#L9) notes this needs proper `Result` returns.

## Build & Development

### Commands
```bash
cargo build --release              # Production build
cargo test                         # Run tests (requires game files)
```

### Platform-Specific
- **Windows**: [build.rs](build.rs) embeds UTF-8 manifest and resources via `embed-manifest` + `winresource`
- **Other**: Build script is no-op

### Dependencies
- `porter-utils`: Custom binary I/O from https://github.com/echo000/porter-lib (not on crates.io)
- `zstd`: Compression (game uses level 0 for speed)
- `walkdir`: Directory traversal for pack operations
- `serde_json`: JSON serialization with `preserve_order` feature

## Usage Patterns

### Output Structure
```
output/
  {filename}/
    entries.json        # Editable string map (id → text)
    tables/             # Raw decompressed blocks (0, 1, 2...)
    modified/           # Repacked tables (for debugging)
    merged/             # Final .map file here
```

### Known Limitations
- Patch files (`_diff` suffix) cannot be merged automatically (see [README.md](README.md))
- Only edit maps with trivial (1KB) patch files
- No validation of string lengths or encoding

## Conventions
- Use `Path::new()` for cross-platform paths
- Sort tables by index when packing (see [core.rs](src/core.rs#L151))
- Preserve bucket structure even if not fully understood
- Set CWD to exe directory at startup ([utils/utils.rs](src/utils/utils.rs#L3))

## Testing
Tests in [tests.rs](src/tests.rs) use hardcoded Windows paths. Update `ZONE_PATH` constants to match your game installation.
