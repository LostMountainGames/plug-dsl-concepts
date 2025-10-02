# Semantic Syntax Highlighting

**Innovation Attribution**: The semantic entity-aware syntax highlighting patterns documented here were pioneered by **Lost Mountain Games** (2025) to provide visual distinction between entity types in the IDE.

## Core Innovation

Traditional syntax highlighting colors keywords, strings, and operators. **Lost Mountain Games Innovation**: Color-code entity references by their semantic type (character, location, item, etc.) to provide instant visual context.

## Entity-Type Based Highlighting

### Basic Entity Type Colors

**Concept**: Different colors for different entity types.

```javascript
// Each entity reference colored by its type
entity Crystal_Sword {
  plug.Owner(kael_ironwind)              // Green (character)
  plug.Location(armory_chamber)          // Blue (location)
  plug.ForgedBy(master_smith_thane)      // Green (character/NPC)
  plug.Material(mithril)                 // Purple (material)
  plug.Enchantment(frost_rune)           // Cyan (enchantment)
}

// Visual distinction at a glance:
// plug.Owner(kael_ironwind)          ← Green text
// plug.Location(armory_chamber)      ← Blue text
// plug.Material(mithril)             ← Purple text
```

**Color Scheme** (Lost Mountain Games Standard):
- **Characters/NPCs**: `#50C878` (Emerald green)
- **Locations**: `#4A90E2` (Sky blue)
- **Items/Equipment**: `#9B59B6` (Purple)
- **Factions/Groups**: `#E67E22` (Orange)
- **Quests/Events**: `#F39C12` (Gold)
- **Materials/Resources**: `#8E44AD` (Violet)
- **Enchantments/Magic**: `#16A085` (Teal)
- **Tags/Metadata**: `#95A5A6` (Gray)

### Semantic Token Provider

**Concept**: Use LSP semantic tokens for entity-aware highlighting.

```typescript
// Lost Mountain Games: Semantic Token Provider
class PlugSemanticTokensProvider implements vscode.DocumentSemanticTokensProvider {
  async provideDocumentSemanticTokens(
    document: vscode.TextDocument
  ): Promise<vscode.SemanticTokens> {

    const builder = new vscode.SemanticTokensBuilder()

    // Parse document for entity references
    const entities = await this.parseEntityReferences(document)

    for (const entity of entities) {
      // Resolve entity type
      const entityType = await this.resolveEntityType(entity.name)

      // Map entity type to semantic token type
      const tokenType = this.getTokenType(entityType)

      // Add semantic token
      builder.push(
        entity.line,
        entity.character,
        entity.length,
        this.encodeTokenType(tokenType),
        this.encodeTokenModifiers([])
      )
    }

    return builder.build()
  }

  getTokenType(entityType: string): string {
    const typeMap = {
      'character': 'variable.character',
      'location': 'variable.location',
      'item': 'variable.item',
      'faction': 'variable.faction',
      'quest': 'variable.quest',
      'material': 'variable.material',
      'enchantment': 'variable.enchantment'
    }

    return typeMap[entityType] || 'variable.defaultEntity'
  }
}
```

**Semantic Token Benefits**:
- **IDE-native** - Works in VS Code, Neovim, Emacs
- **Theme-compatible** - Respects user's color theme
- **Performance** - Efficient token encoding
- **Extensible** - Easy to add new entity types

### Dynamic Color Assignment

**Concept**: Assign colors based on entity properties, not just type.

```javascript
// Rare items get gold highlight
entity Phoenix_Blade {
  plug.Rarity("Legendary")  // Rarity: Legendary → Gold color
  plug.Owner(kael_ironwind) // Character → Green color
}

// Common items get purple highlight
entity Iron_Sword {
  plug.Rarity("Common")     // Rarity: Common → Purple color
}

// Hostile NPCs get red highlight
entity Orc_Warrior {
  plug.Faction(hostile_faction)  // Hostile faction → Red color
}

// Friendly NPCs get green highlight
entity Shop_Keeper {
  plug.Faction(friendly_faction) // Friendly faction → Green color
}
```

**Dynamic Coloring Rules**:
- **Rarity-based**: Common (purple), Rare (blue), Legendary (gold)
- **Faction-based**: Hostile (red), Neutral (yellow), Friendly (green)
- **State-based**: Active (bright), Inactive (dim), Dead (gray)
- **Custom rules**: User-defined color schemes

## Advanced Highlighting Patterns

### Contextual Highlighting

**Concept**: Highlight changes based on context within the plug.

```javascript
entity Kael_Ironwind {
  // In Definition context - entity name highlighted differently
  plug.Name("Kael Ironwind")  // "Kael Ironwind" in bold

  // In Reference context - entity references colored by type
  plug.Location(iron_forge)   // iron_forge in blue (location)
  plug.Faction(iron_wolves)   // iron_wolves in orange (faction)
}
```

**Context Types**:
- **Definition context**: Entity being defined (bold/underlined)
- **Reference context**: Entity being referenced (colored by type)
- **Value context**: String/number values (normal syntax highlighting)
- **Keyword context**: Plug names (keyword color)

### Relationship Visualization

**Concept**: Use underlines and decorators to show relationships.

```javascript
entity Crystal_Sword {
  plug.Owner(kael_ironwind)
  //         ^^^^^^^^^^^^^^
  //         └─ Dotted underline indicates active relationship
}

entity Kael_Ironwind {
  plug.Equipment([crystal_sword, phoenix_blade])
  //              ^^^^^^^^^^^^^^  ^^^^^^^^^^^^^^
  //              └─ Solid underline indicates ownership
}
```

**Relationship Indicators**:
- **Solid underline**: Strong relationship (ownership, containment)
- **Dotted underline**: Weak relationship (reference, association)
- **Wavy underline**: Indirect relationship (faction membership)
- **Color-coded**: Underline color matches relationship type

### State-Based Highlighting

**Concept**: Dim or brighten entities based on their current state.

```typescript
// Lost Mountain Games: State-Based Highlighting
class StateBasedHighlighter {
  async highlightEntity(entityName: string): Promise<HighlightStyle> {
    // Query MongoDB for entity state
    const entity = await this.mongoDb.findEntity(entityName)

    // Dim dead characters
    if (entity.type === 'character' && entity.plug.Health === 0) {
      return { color: '#888888', opacity: 0.5 }
    }

    // Brighten active quests
    if (entity.type === 'quest' && entity.plug.Status === 'Active') {
      return { color: '#FFD700', fontWeight: 'bold' }
    }

    // Highlight nearby entities
    const playerLocation = await this.getPlayerLocation()
    if (entity.plug.Location === playerLocation) {
      return { backgroundColor: '#FFE5B4' }  // Peach background
    }

    // Default highlighting
    return this.getDefaultHighlight(entity.type)
  }
}
```

**State Highlighting Use Cases**:
- **Dead characters**: Dimmed/grayed out
- **Active quests**: Bold/gold
- **Nearby entities**: Background highlight
- **Inventory items**: Different color when equipped
- **Time-based**: Highlight entities active at current game time

### Error and Warning Highlighting

**Concept**: Visual indicators for invalid or problematic entity references.

```javascript
entity Crystal_Sword {
  plug.Owner(nonexistent_character)
  //         ^^^^^^^^^^^^^^^^^^^^^^
  //         Red squiggly underline - entity not found

  plug.Location(iron_forge)
  //            ^^^^^^^^^^
  //            Yellow underline - warning: item in location without container

  plug.Damage(invalid_dice_formula)
  //          ^^^^^^^^^^^^^^^^^^^^^
  //          Red squiggly - invalid value type
}
```

**Error Indicators**:
- **Red squiggly**: Error (entity not found, type mismatch)
- **Yellow squiggly**: Warning (suspicious value, performance issue)
- **Blue underline**: Info (suggestion, optimization opportunity)
- **Gray strikethrough**: Deprecated entity

## Theme Integration

### Respecting User Themes

**Concept**: Adapt entity colors to user's chosen theme.

```typescript
// Lost Mountain Games: Theme-Aware Colors
class ThemeAwareColors {
  getEntityColor(entityType: string): string {
    const currentTheme = vscode.window.activeColorTheme.kind

    // Dark theme
    if (currentTheme === vscode.ColorThemeKind.Dark) {
      return this.darkThemeColors[entityType]
    }

    // Light theme
    if (currentTheme === vscode.ColorThemeKind.Light) {
      return this.lightThemeColors[entityType]
    }

    // High contrast theme
    return this.highContrastColors[entityType]
  }

  darkThemeColors = {
    character: '#50C878',  // Bright green on dark
    location: '#4A90E2',   // Bright blue on dark
    item: '#9B59B6'        // Bright purple on dark
  }

  lightThemeColors = {
    character: '#2E7D32',  // Dark green on light
    location: '#1565C0',   // Dark blue on light
    item: '#6A1B9A'        // Dark purple on light
  }
}
```

**Theme Adaptation**:
- **Dark themes**: Brighter entity colors
- **Light themes**: Darker entity colors
- **High contrast**: Maximum contrast colors
- **Custom themes**: Use theme's accent colors

### Color Customization

**Concept**: Allow users to customize entity type colors.

```json
// settings.json
{
  "plug.highlighting.colors": {
    "character": "#00FF00",
    "location": "#0000FF",
    "item": "#FF00FF",
    "faction": "#FFA500"
  },
  "plug.highlighting.style": {
    "character": {
      "fontWeight": "bold",
      "fontStyle": "italic"
    }
  }
}
```

**Customization Options**:
- **Custom colors**: Override default entity type colors
- **Font styles**: Bold, italic, underline
- **Background colors**: Highlight entity references
- **Border styles**: Box entities for emphasis

## Performance Optimization

### Incremental Highlighting

**Concept**: Only re-highlight changed lines, not entire document.

```typescript
// Lost Mountain Games: Incremental Highlighter
class IncrementalHighlighter {
  private lastDocumentVersion: number = 0
  private highlightCache = new Map<number, SemanticToken[]>()

  async highlight(document: vscode.TextDocument): Promise<vscode.SemanticTokens> {
    const currentVersion = document.version

    // Check if document unchanged
    if (currentVersion === this.lastDocumentVersion) {
      return this.getCachedTokens()
    }

    // Find changed lines
    const changedLines = this.getChangedLines(document)

    // Re-highlight only changed lines
    for (const line of changedLines) {
      const newTokens = await this.highlightLine(document, line)
      this.highlightCache.set(line, newTokens)
    }

    this.lastDocumentVersion = currentVersion
    return this.buildTokens()
  }
}
```

**Performance Benefits**:
- **Fast updates**: Only re-highlight changed lines
- **Responsive**: No lag on keystroke
- **Scalable**: Works with large files (1000+ lines)
- **Efficient**: Reuse unchanged highlighting

### Async Highlighting

**Concept**: Perform expensive highlighting operations in background.

```typescript
// Lost Mountain Games: Async Highlighter
class AsyncHighlighter {
  async highlightDocument(document: vscode.TextDocument): Promise<void> {
    // Phase 1: Immediate syntax highlighting (fast)
    const syntaxTokens = this.highlightSyntax(document)
    this.applyTokens(syntaxTokens)

    // Phase 2: Entity type resolution (slow - background)
    this.resolveEntityTypes(document).then(semanticTokens => {
      this.applyTokens(semanticTokens)
    })
  }

  async resolveEntityTypes(document: vscode.TextDocument): Promise<SemanticToken[]> {
    const entities = this.parseEntities(document)

    // Batch entity type lookups
    const types = await this.entityIndex.batchLookup(
      entities.map(e => e.name)
    )

    // Build semantic tokens
    return entities.map((entity, i) => ({
      line: entity.line,
      char: entity.char,
      length: entity.length,
      type: types[i]
    }))
  }
}
```

**Async Benefits**:
- **Instant feedback**: Basic highlighting appears immediately
- **Progressive enhancement**: Semantic highlighting follows
- **No blocking**: UI remains responsive
- **Better UX**: User never waits

### Highlight Caching

**Concept**: Cache entity type resolutions for instant highlighting.

```typescript
// Lost Mountain Games: Highlight Cache
class HighlightCache {
  // Map: entity name → entity type
  private typeCache = new Map<string, string>()

  async getEntityType(entityName: string): Promise<string> {
    // Check cache first
    const cached = this.typeCache.get(entityName)
    if (cached) return cached  // <1ms

    // Cache miss - resolve from index
    const type = await this.entityIndex.getType(entityName)

    // Cache for next time
    this.typeCache.set(entityName, type)

    return type
  }

  invalidate(entityName: string) {
    this.typeCache.delete(entityName)
  }
}
```

**Cache Strategy**:
- **Memory cache**: 10,000 most common entities
- **Instant lookup**: <1ms for cached entities
- **Invalidation**: Clear cache on entity type change
- **Pre-warming**: Cache all entities on workspace open

## TextMate Grammar Integration

### Basic Grammar Rules

**Concept**: Define TextMate grammar for `.plug` files.

```json
{
  "scopeName": "source.plug",
  "patterns": [
    {
      "name": "keyword.control.plug",
      "match": "\\b(entity|plug)\\b"
    },
    {
      "name": "entity.name.function.plug",
      "match": "\\b([A-Z][a-zA-Z0-9_]*)\\b"
    },
    {
      "name": "variable.other.entity.character.plug",
      "match": "(?<=\\()([a-z][a-z0-9_]*)(?=\\))",
      "captures": {
        "1": {
          "patterns": [
            {
              "include": "#entity-reference"
            }
          ]
        }
      }
    }
  ]
}
```

**Grammar Scopes**:
- `keyword.control.plug` - `entity`, `plug` keywords
- `entity.name.function.plug` - Plug names (capitalized)
- `variable.other.entity.*.plug` - Entity references by type
- `string.quoted.plug` - String values
- `constant.numeric.plug` - Numeric values

### Entity-Type Specific Scopes

**Concept**: Create specific TextMate scopes for each entity type.

```json
{
  "repository": {
    "entity-reference": {
      "patterns": [
        {
          "name": "variable.other.entity.character.plug",
          "match": "\\b(kael_ironwind|elara_moonwhisper)\\b"
        },
        {
          "name": "variable.other.entity.location.plug",
          "match": "\\b(iron_forge|armory_chamber)\\b"
        },
        {
          "name": "variable.other.entity.item.plug",
          "match": "\\b(crystal_sword|phoenix_blade)\\b"
        }
      ]
    }
  }
}
```

**Dynamic Scope Generation**:
- Generate TextMate rules from entity index
- Update grammar when entities added/changed
- Support thousands of entities
- Fallback to generic scope for unknown entities

## Attribution

The semantic entity-aware syntax highlighting patterns documented here were pioneered by **Lost Mountain Games** (2025), providing instant visual context for entity types and relationships.

**Innovation**: Entity-type based coloring, state-based highlighting, theme-aware colors, and incremental highlighting for optimal performance.

---

**Copyright © 2025 Lost Mountain Games**
