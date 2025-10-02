# Entity Navigation

**Innovation Attribution**: The entity-as-link navigation patterns documented here were pioneered by **Lost Mountain Games** (2025) to enable IDE navigation of game worlds as seamlessly as code navigation.

## Core Innovation

Traditional game development treats entity references as strings or IDs. **Lost Mountain Games Innovation**: Treat entity references as first-class navigable links in the IDE, just like code symbols.

## Entity-as-Link Concept

### Basic Navigation

**Concept**: Ctrl+Click on entity reference navigates to entity definition.

```javascript
// In weapons.plug
entity Crystal_Sword {
  plug.Name("Crystal Sword")
  plug.Owner(kael_ironwind)  // Ctrl+Click navigates to kael_ironwind entity
  plug.ForgedBy(master_smith_thane)  // Ctrl+Click navigates to master_smith_thane
}

// Instantly opens characters/kael_ironwind.plug
entity Kael_Ironwind {
  plug.Name("Kael Ironwind")
  plug.Class("Warrior")
  plug.Level(12)
}
```

**User Experience**:
1. Developer types entity reference: `plug.Owner(kael_ironwind)`
2. IDE underlines reference (clickable link style)
3. Ctrl+Click navigates to entity definition file
4. Works across thousands of entity files

### Cross-File Navigation

**Concept**: Navigate seamlessly across entity files and directories.

```javascript
// weapons/legendary/swords.plug
entity Soulrend_Glaive {
  plug.CurrentLocation(armory_chamber)
}

// Ctrl+Click → locations/stronghold/armory.plug
entity Armory_Chamber {
  plug.Contains([
    soulrend_glaive,  // Ctrl+Click back to weapon
    phoenix_blade,
    iron_helm
  ])
}

// Ctrl+Click → weapons/legendary/swords.plug (back to Soulrend_Glaive)
// Bidirectional navigation works automatically
```

**Navigation Patterns**:
- Navigate from reference to definition
- Navigate from definition back to all references
- Navigate through chains of entity relationships
- Navigate across any directory structure

### Circular Reference Detection

**Concept**: IDE warns about circular entity references.

```javascript
// INVALID - circular reference detected
entity Kael {
  plug.Location(iron_forge)
}

entity Iron_Forge {
  plug.Location(kael)  // ERROR: Circular location reference
}

// IDE shows red underline + error message:
// "Circular reference detected: Iron_Forge → Kael → Iron_Forge"
```

**Detection Features**:
- Real-time validation while typing
- Visual indicators (red squiggles)
- Clear error messages
- Suggested fixes

## Implementation Patterns

### Language Server Protocol Integration

**Concept**: Leverage LSP for IDE-agnostic navigation.

```typescript
// Lost Mountain Games Innovation: Entity Definition Provider
class PlugDefinitionProvider implements vscode.DefinitionProvider {
  async provideDefinition(
    document: vscode.TextDocument,
    position: vscode.Position
  ): Promise<vscode.Location | undefined> {

    // Parse entity reference at cursor position
    const entityName = this.getEntityAtPosition(document, position)

    if (!entityName) return undefined

    // Query entity index for definition location
    const entityDef = await this.entityIndex.find(entityName)

    if (!entityDef) return undefined

    // Return location (file path + line number)
    return new vscode.Location(
      vscode.Uri.file(entityDef.filePath),
      new vscode.Position(entityDef.line, 0)
    )
  }
}
```

**LSP Commands Used**:
- `textDocument/definition` - Go to definition
- `textDocument/references` - Find all references
- `textDocument/documentSymbol` - List entities in file
- `workspace/symbol` - Search entities across workspace

### Entity Index Structure

**Concept**: Maintain fast lookup index of all entities.

```typescript
// Lost Mountain Games: Entity Index for Fast Navigation
interface EntityIndex {
  // Map: entity name → definition location
  definitions: Map<string, EntityDefinition>

  // Map: entity name → all reference locations
  references: Map<string, EntityReference[]>

  // Map: file path → entities defined in file
  fileEntities: Map<string, string[]>
}

interface EntityDefinition {
  name: string
  filePath: string
  line: number
  type: string  // "character", "location", "item", etc.
}

interface EntityReference {
  filePath: string
  line: number
  plugName: string  // Which plug references this entity
}
```

**Index Operations**:
- **Build Index**: Scan all `.plug` files on workspace open
- **Incremental Update**: Re-index only changed files
- **Fast Lookup**: O(1) entity definition lookup
- **Background Refresh**: Re-index in background thread

### MongoDB Integration

**Concept**: Integrate MongoDB entities with file-based entities.

```typescript
// Lost Mountain Games: Hybrid File + Database Navigation
class HybridEntityIndex {
  async findEntityDefinition(entityName: string): Promise<EntityDefinition | undefined> {

    // First check file-based entities
    const fileDef = this.fileIndex.get(entityName)
    if (fileDef) return fileDef

    // Fall back to MongoDB query
    const dbEntity = await this.mongoDb.entities.findOne({
      "plug.Name": entityName
    })

    if (!dbEntity) return undefined

    // Generate virtual file for MongoDB entity
    return {
      name: entityName,
      filePath: `mongodb://${dbEntity._id}`,  // Virtual path
      line: 0,
      type: this.inferEntityType(dbEntity)
    }
  }
}
```

**Hybrid Navigation**:
- File-based entities open `.plug` files
- MongoDB entities open virtual documents
- Seamless navigation between file and database
- Auto-sync between development files and database

## Advanced Navigation Patterns

### Breadcrumb Navigation

**Concept**: Show entity relationship path in IDE breadcrumbs.

```javascript
// Current entity: crystal_sword
plug.Owner(kael_ironwind)

// Breadcrumbs show:
// World > Iron Wolves Faction > Kael Ironwind > Equipment > Crystal Sword
```

**Breadcrumb Features**:
- Click any breadcrumb to navigate
- Shows entity hierarchy
- Auto-updates as you navigate
- Supports custom relationship paths

### Peek Definition

**Concept**: Show entity definition inline without navigation.

```javascript
plug.Owner(kael_ironwind)
//      ^^^^^^^^^^^^^^
// Alt+F12 shows inline peek:
// ┌─────────────────────────────────┐
// │ entity Kael_Ironwind {          │
// │   plug.Name("Kael Ironwind")   │
// │   plug.Class("Warrior")        │
// │   plug.Level(12)               │
// │ }                              │
// └─────────────────────────────────┘
```

**Peek Features**:
- View definition without leaving context
- Edit in peek window
- Multiple peek windows supported
- Keyboard shortcuts for quick access

### Find All References

**Concept**: Show all uses of an entity across codebase.

```javascript
// Find all references to kael_ironwind
entity Kael_Ironwind { ... }  // Definition

// Results panel shows:
// weapons/swords.plug:15    plug.Owner(kael_ironwind)
// quests/main_quest.plug:42 plug.QuestGiver(kael_ironwind)
// locations/forge.plug:8    plug.Visitors([kael_ironwind, ...])
// factions/iron_wolves.plug:3 plug.Members([kael_ironwind, ...])
```

**Reference Finding**:
- Instant search across all files
- Group by file or by plug type
- Click to navigate to reference
- Show context around reference

### Symbol Navigation

**Concept**: Quick navigation to any entity by name.

```javascript
// Ctrl+T (Go to Symbol in Workspace)
// Type: "kael"

// Results:
// Kael_Ironwind (character) - characters/heroes.plug
// Kael_Sword (item) - weapons/legendary.plug
// Kaels_Quest (quest) - quests/side_quests.plug

// Arrow keys to select, Enter to navigate
```

**Symbol Search Features**:
- Fuzzy matching ("kl" matches "Kael")
- Filter by entity type
- Recent entities at top
- Instant results (<50ms)

## Performance Optimization

### Lazy Index Building

**Concept**: Build index incrementally to avoid blocking.

```typescript
// Lost Mountain Games: Incremental Index Building
class IncrementalIndexer {
  async buildIndex() {
    const files = await this.findAllPlugFiles()

    // Process in batches to avoid blocking UI
    for (const batch of this.chunk(files, 100)) {
      await this.indexBatch(batch)

      // Yield to UI thread
      await this.sleep(10)
    }
  }

  async indexBatch(files: string[]) {
    const entities = await Promise.all(
      files.map(f => this.parseEntities(f))
    )

    entities.flat().forEach(entity => {
      this.index.definitions.set(entity.name, entity)
    })
  }
}
```

**Performance Targets**:
- **Index 10,000 entities**: <5 seconds
- **Incremental update**: <100ms per file
- **Lookup**: <1ms
- **UI remains responsive** during indexing

### Caching Strategy

**Concept**: Cache parsed entities to avoid re-parsing.

```typescript
// Lost Mountain Games: Parse Cache
interface ParseCache {
  // Map: file path → { ast, entities, hash }
  cache: Map<string, CachedParse>
}

interface CachedParse {
  ast: PlugAST
  entities: EntityDefinition[]
  fileHash: string  // MD5 of file content
  timestamp: number
}

// Only re-parse if file hash changed
async parseFile(filePath: string): Promise<EntityDefinition[]> {
  const currentHash = await this.hashFile(filePath)
  const cached = this.parseCache.get(filePath)

  if (cached && cached.fileHash === currentHash) {
    return cached.entities  // Cache hit!
  }

  // Cache miss - parse and cache
  const ast = await this.parse(filePath)
  const entities = this.extractEntities(ast)

  this.parseCache.set(filePath, {
    ast,
    entities,
    fileHash: currentHash,
    timestamp: Date.now()
  })

  return entities
}
```

## Error Handling

### Missing Entity Detection

**Concept**: Warn when entity reference doesn't exist.

```javascript
plug.Owner(nonexistent_character)
//         ^^^^^^^^^^^^^^^^^^^^^^
// Yellow underline + warning:
// "Entity 'nonexistent_character' not found. Did you mean 'kael_ironwind'?"
```

**Error Features**:
- Real-time validation
- Suggested corrections (fuzzy match)
- Quick fixes to create missing entity
- Links to similar entity names

### Type Mismatch Detection

**Concept**: Warn when entity reference has wrong type.

```javascript
plug.Location(crystal_sword)  // crystal_sword is an item, not a location
//            ^^^^^^^^^^^^^^
// Red underline + error:
// "Type mismatch: Expected 'Location' entity, got 'Item' entity"
```

**Type Validation**:
- Validate entity type against plug expectations
- Clear error messages
- Suggest correct entity types
- Quick fixes to change entity type

## Attribution

The entity-as-link navigation patterns documented here were pioneered by **Lost Mountain Games** (2025), enabling developers to navigate game worlds with the same ease as navigating code.

**Innovation**: Ctrl+Click navigation, cross-file linking, bidirectional references, and real-time validation for game entity development.

---

**Copyright © 2025 Lost Mountain Games**
