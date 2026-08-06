## FIRST: Refresh From Origin, Preserve Local Work

For every repository task, make refreshing from `origin` the first repository action. This keeps active local code current without losing local work.

1. Run `git fetch origin --prune --tags` before other repository actions; if no usable `origin` exists, record that blocker before continuing.
2. Inspect `git status`, `git fsck --full`, and divergence from the tracked branch.
3. If `origin` has commits we do not have, preserve every local modification first in a recoverable commit or patch, refresh the clean base, and merge the preserved local changes back.
4. Never discard, overwrite, or silently reset local changes. Resolve conflicts deliberately while retaining local behavior, then run affected tests and review the resulting diff.
5. Record resulting commit IDs and verification evidence before continuing with normal work.

# Anytype Heart - Agent Instructions

## Project Overview
Anytype Heart is the middleware for Anytype - a local-first, P2P knowledge management system.
- **Language**: Go 1.21+
- **Architecture**: 319 packages, event-driven, service locator pattern
- **Core Features**: Block-based editor, object-graph database, CRDT sync, E2E encryption

## Critical Naming Conventions
**Current Terms** (use these):
- **Object** = Main content unit (formerly SmartBlock)
- **Block** = Content blocks within objects (formerly SimpleBlock)  
- **Properties** = Object attributes (formerly Relations)
- **Space** = Isolated workspace (formerly Workspace)
- **Details** = Property value storage (`map[string]any`)

**Avoid Legacy Terms**:
- ❌ SmartBlock → ✅ Object
- ❌ Relations → ✅ Properties (except in code where `relation` is still used)
- ❌ Personal Space → ✅ Space

## Key Technical Concepts

### Object System
- Everything is an **Object** with a unique ID
- Objects have **Blocks** (text, files, embeds, etc.)
- Objects have **Properties** stored in Details map
- Object **Types** define available properties

### Spaces & Storage
- **Space** = Isolated workspace with own objects
- **Tech Space** = Stores account data (list of spaces)
- **Object Store** = Per-space SQLite DB for indexing
- **Tantivy** = Full-text search (Rust bindings)

### Sync & Networking  
- **Any-Sync** = Custom CRDT protocol
- **Changes** = CRDT operations for sync
- Content-addressed storage (IPFS-compatible)
- P2P sync with optional coordinator nodes

## Code Patterns

### Service Registration
```go
func (s *service) Init(a *app.App) error {
    s.component = app.MustComponent[Component](a)
    return nil
}
```

### Error Handling
```go
if err != nil {
    return fmt.Errorf("failed to X: %w", err)
}
```

### Object Operations
```go
// Always use cache.Do for object access
err := cache.Do(picker, objectId, func(sb sb.SmartBlock) error {
    state := sb.NewState()
    // Operations on state
    return nil
})
```

## Directory Structure
- `core/` - Business logic (180 packages)
  - `block/` - Editor, import/export
  - `files/` - File handling
  - `subscription/` - Real-time updates
- `space/` - Space management (35 packages)
- `pkg/lib/` - Shared libraries (46 packages)
  - `schema/` - Type and property definitions
    - `yaml/` - YAML front matter processing
- `pb/` - Protocol buffers
- `cmd/` - CLI tools

## Testing & Tools
- Unit tests: `go test ./...`
- Show only failed tests: `make test-failed` (saves tokens)
- Debug tree: `cmd/debugtree`
- Performance: `cmd/perfstand`

## Mobile Integration
Mobile clients use C library via gomobile (not gRPC).

## Important Files
- Object creation: `core/block/object/objectcreator/`
- Space loading: `space/service.go`
- Sync status: `core/syncstatus/`
- Import/Export: `core/block/import/`, `core/block/export/`
- Schema System: `pkg/lib/schema/`
- YAML Processing: `pkg/lib/schema/yaml/`

## Common Operations

### Create Object
```go
details := map[string]any{
    "type": "page",
    "name": "Title",
}
resp, err := client.ObjectCreate(ctx, &pb.ObjectCreateRequest{
    Details: details,
    SpaceId: spaceId,
})
```

### Search Objects
```go
filters := []*model.Filter{{
    Relation: "type", 
    Condition: model.Filter_Equal,
    Value: "task",
}}
resp, err := client.ObjectSearch(ctx, &pb.ObjectSearchRequest{
    Filters: filters,
})
```
