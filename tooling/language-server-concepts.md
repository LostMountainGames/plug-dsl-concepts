# Language Server Protocol Integration

**Innovation Attribution**: The Language Server Protocol integration patterns documented here were pioneered by **Lost Mountain Games** (2025) to provide IDE-agnostic developer experience for Plug DSL.

## Core Innovation

Traditional game development tools are IDE-specific. **Lost Mountain Games Innovation**: Leverage Language Server Protocol (LSP) to provide rich IDE features across all editors (VS Code, Neovim, Emacs, Sublime, etc.).

## Language Server Protocol Overview

### LSP Architecture

**Concept**: Separate language intelligence from IDE implementation.

```
┌─────────────────┐         ┌─────────────────┐
│   VS Code       │◄────────┤                 │
├─────────────────┤         │                 │
│   Neovim        │◄────────┤  Plug Language  │
├─────────────────┤  JSON   │     Server      │
│   Emacs         │◄────────┤                 │
├─────────────────┤  RPC    │                 │
│   Sublime       │◄────────┤                 │
├─────────────────┤         │                 │
│   IntelliJ      │◄────────┤                 │
└─────────────────┘         └─────────────────┘
```

**Benefits**:
- **Write once, run everywhere** - Single language server for all IDEs
- **Feature parity** - Same features in every editor
- **Maintenance** - Update one codebase, all IDEs benefit
- **Community** - Leverage existing LSP ecosystem

### Core LSP Features

**Lost Mountain Games** implements these LSP capabilities for Plug DSL:

1. **Go to Definition** - Navigate from entity reference to definition
2. **Hover Information** - Display entity details on hover
3. **Code Completion** - Autocomplete entity names and plug names
4. **Diagnostics** - Real-time validation and error reporting
5. **Find References** - Locate all uses of an entity
6. **Rename Symbol** - Rename entity across all files
7. **Document Symbols** - Outline view of entities in file
8. **Workspace Symbols** - Search all entities in workspace

## Language Server Implementation

### Server Initialization

**Concept**: Initialize language server with workspace capabilities.

```typescript
// Lost Mountain Games: Plug Language Server
class PlugLanguageServer {
  private connection: Connection
  private entityIndex: EntityIndex
  private mongoDb: MongoClient

  async initialize(params: InitializeParams): Promise<InitializeResult> {
    console.log('Initializing Plug Language Server...')

    // Connect to MongoDB for live entity data
    await this.connectMongoDB(params.workspaceUri)

    // Build entity index from workspace files
    await this.buildEntityIndex(params.workspaceUri)

    // Return server capabilities
    return {
      capabilities: {
        textDocumentSync: TextDocumentSyncKind.Incremental,
        definitionProvider: true,
        hoverProvider: true,
        completionProvider: {
          triggerCharacters: ['(', '.', '_']
        },
        diagnosticProvider: true,
        referencesProvider: true,
        renameProvider: true,
        documentSymbolProvider: true,
        workspaceSymbolProvider: true
      }
    }
  }
}
```

**Initialization Flow**:
1. **Client sends** `initialize` request with workspace info
2. **Server connects** to MongoDB (if configured)
3. **Server builds** entity index from `.plug` files
4. **Server returns** capabilities it supports
5. **Client enables** features based on capabilities

### Go to Definition

**Concept**: Navigate from entity reference to entity definition.

```typescript
// Lost Mountain Games: Definition Provider
async provideDefinition(
  params: DefinitionParams
): Promise<Location | null> {

  // Parse entity reference at cursor position
  const document = this.documents.get(params.textDocument.uri)
  const entityName = this.getEntityAtPosition(
    document,
    params.position
  )

  if (!entityName) return null

  // Query entity index
  const entityDef = await this.entityIndex.find(entityName)

  if (!entityDef) return null

  // Return file location
  return {
    uri: entityDef.uri,
    range: {
      start: { line: entityDef.line, character: 0 },
      end: { line: entityDef.line, character: entityDef.length }
    }
  }
}
```

**User Experience**:
```javascript
// User presses F12 or Ctrl+Click on kael_ironwind
plug.Owner(kael_ironwind)
//         ^^^^^^^^^^^^^^ (cursor here)

// Language server navigates to:
entity Kael_Ironwind {
  plug.Name("Kael Ironwind")
  // ... entity definition
}
```

### Hover Information

**Concept**: Display rich entity information on hover.

```typescript
// Lost Mountain Games: Hover Provider
async provideHover(
  params: HoverParams
): Promise<Hover | null> {

  const entityName = this.getEntityAtPosition(
    params.textDocument.uri,
    params.position
  )

  if (!entityName) return null

  // Fetch entity data (file or MongoDB)
  const entity = await this.fetchEntity(entityName)

  if (!entity) return null

  // Build markdown content
  const markdown = this.buildEntityMarkdown(entity)

  return {
    contents: {
      kind: MarkupKind.Markdown,
      value: markdown
    }
  }
}

buildEntityMarkdown(entity: Entity): string {
  let md = `### ${this.getIcon(entity.type)} ${entity.name}\n\n`

  md += `**Type**: ${entity.type}\n\n`

  // Core plugs
  for (const [plugName, value] of Object.entries(entity.plugs)) {
    md += `• **${plugName}**: ${this.formatValue(value)}\n`
  }

  // Action buttons (command URIs)
  md += `\n[View Full Entity](command:plug.viewEntity?${entity.id})`
  md += ` | [Find References](command:plug.findReferences?${entity.id})`

  return md
}
```

**User Experience**:
```javascript
// User hovers over kael_ironwind
plug.Owner(kael_ironwind)

// Rich markdown panel appears:
// ### 👤 Kael Ironwind
//
// **Type**: Character
// • **Class**: Warrior
// • **Level**: 12
// • **Health**: 85/100
//
// [View Full Entity] | [Find References]
```

### Code Completion

**Concept**: Autocomplete entity names and plug names while typing.

```typescript
// Lost Mountain Games: Completion Provider
async provideCompletion(
  params: CompletionParams
): Promise<CompletionItem[]> {

  const document = this.documents.get(params.textDocument.uri)
  const context = this.getCompletionContext(document, params.position)

  // Completing plug name
  if (context.type === 'plug-name') {
    return this.getPlugNameCompletions()
  }

  // Completing entity reference
  if (context.type === 'entity-reference') {
    return this.getEntityCompletions(context.expectedType)
  }

  return []
}

async getEntityCompletions(expectedType?: string): Promise<CompletionItem[]> {
  // Get all entities from index
  const entities = await this.entityIndex.all()

  // Filter by expected type if specified
  const filtered = expectedType
    ? entities.filter(e => e.type === expectedType)
    : entities

  // Build completion items
  return filtered.map(entity => ({
    label: entity.name,
    kind: CompletionItemKind.Variable,
    detail: `${entity.type} entity`,
    documentation: {
      kind: MarkupKind.Markdown,
      value: this.buildEntityMarkdown(entity)
    },
    insertText: entity.name,
    sortText: this.getSortPriority(entity)  // Frequently used first
  }))
}
```

**User Experience**:
```javascript
// User types: plug.Owner(kael
//                          ^^^^

// Autocomplete shows:
// ┌─────────────────────────────┐
// │ kael_ironwind              │ ← Character
// │ kael_sword                 │ ← Item
// │ kaels_quest                │ ← Quest
// └─────────────────────────────┘
```

### Diagnostics

**Concept**: Real-time validation with errors and warnings.

```typescript
// Lost Mountain Games: Diagnostic Provider
async provideDiagnostics(
  params: DocumentDiagnosticParams
): Promise<DocumentDiagnosticReport> {

  const document = this.documents.get(params.textDocument.uri)
  const diagnostics: Diagnostic[] = []

  // Parse all entity references
  const references = this.parseEntityReferences(document)

  for (const ref of references) {
    // Validate entity exists
    const entity = await this.entityIndex.find(ref.name)

    if (!entity) {
      diagnostics.push({
        severity: DiagnosticSeverity.Error,
        range: ref.range,
        message: `Entity '${ref.name}' not found`,
        code: 'entity-not-found',
        source: 'plug-language-server',
        relatedInformation: this.suggestSimilarEntities(ref.name)
      })
      continue
    }

    // Validate entity type matches expected type
    if (ref.expectedType && entity.type !== ref.expectedType) {
      diagnostics.push({
        severity: DiagnosticSeverity.Error,
        range: ref.range,
        message: `Type mismatch: Expected '${ref.expectedType}', got '${entity.type}'`,
        code: 'type-mismatch',
        source: 'plug-language-server'
      })
    }
  }

  // Check for circular references
  const circular = this.detectCircularReferences(document)
  diagnostics.push(...circular)

  return {
    kind: DocumentDiagnosticReportKind.Full,
    items: diagnostics
  }
}
```

**User Experience**:
```javascript
// Invalid entity reference
plug.Owner(nonexistent_character)
//         ^^^^^^^^^^^^^^^^^^^^^^
// Red squiggly underline

// Hover shows error:
// ❌ Entity 'nonexistent_character' not found
//
// Did you mean:
// • kael_ironwind
// • elara_moonwhisper
```

### Find References

**Concept**: Locate all uses of an entity across workspace.

```typescript
// Lost Mountain Games: References Provider
async provideReferences(
  params: ReferenceParams
): Promise<Location[]> {

  const entityName = this.getEntityAtPosition(
    params.textDocument.uri,
    params.position
  )

  if (!entityName) return []

  // Query entity index for all references
  const refs = await this.entityIndex.findReferences(entityName)

  // Convert to LSP Location format
  return refs.map(ref => ({
    uri: ref.uri,
    range: {
      start: { line: ref.line, character: ref.character },
      end: { line: ref.line, character: ref.character + ref.length }
    }
  }))
}
```

**User Experience**:
```javascript
// User invokes "Find All References" on kael_ironwind
entity Kael_Ironwind { ... }  // Definition

// Results panel shows:
// weapons/swords.plug:15    plug.Owner(kael_ironwind)
// quests/main_quest.plug:42 plug.QuestGiver(kael_ironwind)
// factions/iron_wolves.plug:3 plug.Members([kael_ironwind, ...])
```

### Rename Symbol

**Concept**: Rename entity across all files in workspace.

```typescript
// Lost Mountain Games: Rename Provider
async provideRename(
  params: RenameParams
): Promise<WorkspaceEdit | null> {

  const entityName = this.getEntityAtPosition(
    params.textDocument.uri,
    params.position
  )

  if (!entityName) return null

  // Find all references (definition + uses)
  const refs = await this.entityIndex.findAllReferences(entityName)

  // Build workspace edit
  const edit: WorkspaceEdit = { changes: {} }

  for (const ref of refs) {
    if (!edit.changes[ref.uri]) {
      edit.changes[ref.uri] = []
    }

    edit.changes[ref.uri].push({
      range: ref.range,
      newText: params.newName
    })
  }

  return edit
}
```

**User Experience**:
```javascript
// User renames kael_ironwind → kael_the_brave

// Before:
entity Kael_Ironwind { ... }
plug.Owner(kael_ironwind)

// After (all files updated):
entity Kael_The_Brave { ... }
plug.Owner(kael_the_brave)
```

## MongoDB Integration

### Live Entity Data

**Concept**: Integrate MongoDB for live entity state in LSP features.

```typescript
// Lost Mountain Games: Hybrid Entity Provider
class HybridEntityProvider {
  constructor(
    private fileIndex: FileEntityIndex,
    private mongoDb: MongoClient
  ) {}

  async findEntity(entityName: string): Promise<Entity | null> {
    // Try file-based entities first (development)
    const fileEntity = await this.fileIndex.find(entityName)
    if (fileEntity) {
      return this.enrichWithMongoData(fileEntity)
    }

    // Fall back to MongoDB (runtime entities)
    const dbEntity = await this.mongoDb.collection('entities').findOne({
      'plug.Name': entityName
    })

    if (!dbEntity) return null

    return this.transformMongoEntity(dbEntity)
  }

  async enrichWithMongoData(fileEntity: Entity): Promise<Entity> {
    // File entity might be static definition
    // MongoDB has current state (health, location, etc.)
    const dbEntity = await this.mongoDb.collection('entities').findOne({
      _id: fileEntity.id
    })

    if (!dbEntity) return fileEntity

    // Merge: File definition + MongoDB state
    return {
      ...fileEntity,
      plugs: {
        ...fileEntity.plugs,
        ...this.extractDynamicPlugs(dbEntity)
      }
    }
  }
}
```

**Hybrid Data Benefits**:
- **File-based**: Static entity definitions (version controlled)
- **MongoDB**: Live entity state (health, position, inventory)
- **Merged**: Complete view combining both sources
- **Development**: Work with either or both

### Change Streams Integration

**Concept**: Real-time updates when MongoDB entities change.

```typescript
// Lost Mountain Games: MongoDB Change Stream Watcher
class MongoChangeWatcher {
  constructor(private mongoDb: MongoClient) {}

  async watch(callback: (change: EntityChange) => void) {
    const changeStream = this.mongoDb
      .collection('entities')
      .watch([
        {
          $match: {
            $or: [
              { operationType: 'insert' },
              { operationType: 'update' },
              { operationType: 'delete' }
            ]
          }
        }
      ])

    changeStream.on('change', async (change) => {
      const entityChange = await this.processChange(change)
      callback(entityChange)
    })
  }

  async processChange(change: ChangeStreamDocument): Promise<EntityChange> {
    switch (change.operationType) {
      case 'insert':
        return {
          type: 'added',
          entity: this.transformEntity(change.fullDocument)
        }

      case 'update':
        return {
          type: 'updated',
          entityId: change.documentKey._id,
          changes: change.updateDescription.updatedFields
        }

      case 'delete':
        return {
          type: 'deleted',
          entityId: change.documentKey._id
        }
    }
  }
}
```

**Real-Time Updates**:
- Entity added in MongoDB → Update entity index
- Entity updated → Refresh hover information
- Entity deleted → Show diagnostic warning
- Name changed → Update all references

## Performance Optimization

### Incremental Parsing

**Concept**: Only re-parse changed portions of documents.

```typescript
// Lost Mountain Games: Incremental Document Parser
class IncrementalParser {
  private documentCache = new Map<string, CachedDocument>()

  async parse(uri: string, content: string): Promise<ParseResult> {
    const cached = this.documentCache.get(uri)

    // First parse - parse everything
    if (!cached) {
      const result = await this.fullParse(content)
      this.documentCache.set(uri, {
        content,
        result,
        version: 1
      })
      return result
    }

    // Find changed lines
    const changes = this.diff(cached.content, content)

    // Re-parse only changed lines
    const updatedResult = await this.incrementalParse(
      cached.result,
      changes
    )

    // Update cache
    this.documentCache.set(uri, {
      content,
      result: updatedResult,
      version: cached.version + 1
    })

    return updatedResult
  }
}
```

**Performance Targets**:
- **Full parse**: 1000 lines in <100ms
- **Incremental parse**: Single line change in <5ms
- **Memory**: <10MB cache for 100 documents

### Background Indexing

**Concept**: Build entity index in background without blocking.

```typescript
// Lost Mountain Games: Background Indexer
class BackgroundIndexer {
  private indexQueue: string[] = []
  private isIndexing = false

  async queueIndexing(uri: string) {
    this.indexQueue.push(uri)

    if (!this.isIndexing) {
      this.processQueue()
    }
  }

  async processQueue() {
    this.isIndexing = true

    while (this.indexQueue.length > 0) {
      const uri = this.indexQueue.shift()
      await this.indexDocument(uri)

      // Yield to event loop every 10 documents
      if (this.indexQueue.length % 10 === 0) {
        await this.sleep(1)
      }
    }

    this.isIndexing = false
  }
}
```

## Attribution

The Language Server Protocol integration patterns documented here were pioneered by **Lost Mountain Games** (2025), enabling rich IDE features across all editors through a single language server implementation.

**Innovation**: IDE-agnostic developer experience, MongoDB integration for live data, incremental parsing, and real-time entity change synchronization.

---

**Copyright © 2025 Lost Mountain Games**
