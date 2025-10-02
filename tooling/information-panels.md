# Floating Information Panels

**Innovation Attribution**: The floating entity information panel concepts documented here were pioneered by **Lost Mountain Games** (2025) to provide rich contextual information without disrupting developer flow.

## Core Innovation

Traditional IDEs show simple tooltips on hover. **Lost Mountain Games Innovation**: Display rich, structured entity information in floating panels that provide complete context without leaving the current file.

## Hover Information Architecture

### Basic Entity Information

**Concept**: Hover over entity reference to see complete entity details.

```javascript
// Hover over kael_ironwind
plug.Owner(kael_ironwind)
//         ^^^^^^^^^^^^^^

// Floating panel appears:
// ┌─────────────────────────────────────────────┐
// │ 👤 Kael Ironwind                           │
// │ ───────────────────────────────────────────│
// │ Type: Character                            │
// │ Class: Warrior                             │
// │ Level: 12                                  │
// │ Health: 85/100                             │
// │ Location: Iron Forge                       │
// │ Faction: Iron Wolves                       │
// │                                            │
// │ [View Full Entity] [Find References]       │
// └─────────────────────────────────────────────┘
```

**Panel Contents**:
- **Entity name and icon** - Visual identification
- **Core attributes** - Most important plugs
- **Current state** - Dynamic values (health, location, etc.)
- **Relationships** - Links to related entities
- **Action buttons** - Quick navigation and operations

### Rich Formatted Information

**Concept**: Use markdown and formatting for readable information display.

```javascript
// Hover over soulrend_glaive
plug.Weapon(soulrend_glaive)

// ┌──────────────────────────────────────────────┐
// │ ⚔️ **Soulrend Glaive**                      │
// │ ────────────────────────────────────────────│
// │ *A legendary polearm that hungers for souls*│
// │                                             │
// │ **Combat Stats**                            │
// │ • Damage: 3d6+3                            │
// │ • Range: 10 feet                           │
// │ • Weight: 8 lbs                            │
// │                                             │
// │ **Special Abilities**                       │
// │ • Soul Drain: Heal on kill                 │
// │ • Reach: Extended melee range              │
// │                                             │
// │ **Current State**                           │
// │ • Owner: Kael Ironwind                     │
// │ • Location: Equipped (main hand)           │
// │ • Souls Claimed: 47                        │
// │                                             │
// │ [View Full Entity] [View Owner]            │
// └──────────────────────────────────────────────┘
```

**Formatting Features**:
- **Markdown rendering** - Bold, italic, lists, headers
- **Emoji icons** - Visual category indicators
- **Grouping** - Logical sections for different plug types
- **Color coding** - Highlight important values
- **Interactive links** - Click to navigate to related entities

### Dynamic State Information

**Concept**: Show real-time entity state from MongoDB.

```javascript
// Hover over iron_forge
plug.Location(iron_forge)

// Panel queries MongoDB for current state:
// ┌──────────────────────────────────────────────┐
// │ 🏛️ **Iron Forge**                           │
// │ ────────────────────────────────────────────│
// │ Type: Location (Settlement)                 │
// │                                             │
// │ **Current Occupants** (Live from DB)        │
// │ • Kael Ironwind (character)                │
// │ • Elara Moonwhisper (character)            │
// │ • Master Smith Thane (NPC)                 │
// │                                             │
// │ **Weather**: Clear skies, 72°F              │
// │ **Time**: Morning, Day 14                   │
// │                                             │
// │ **Notable Items**                           │
// │ • Ancient Forge (crafting station)         │
// │ • Weapon Rack (storage)                    │
// │                                             │
// │ [View Full Entity] [View Map]              │
// └──────────────────────────────────────────────┘
```

**Real-Time Features**:
- **MongoDB queries** - Fetch current entity state
- **Live updates** - Show dynamic values (health, position, time)
- **Relationship resolution** - Display related entities
- **Cache-friendly** - Cache queries for performance

## Advanced Panel Patterns

### Nested Entity Information

**Concept**: Hover within hover panel to see nested entity details.

```javascript
// Hover over kael_ironwind
plug.Owner(kael_ironwind)

// ┌─────────────────────────────────────────────┐
// │ 👤 Kael Ironwind                           │
// │ ───────────────────────────────────────────│
// │ Faction: Iron Wolves                       │
// │          ^^^^^^^^^^^^ (hover)              │
// └─────────────────────────────────────────────┘

// Nested panel appears:
// ┌─────────────────────────────────────────────┐
// │ 🛡️ **Iron Wolves**                         │
// │ ────────────────────────────────────────────│
// │ Type: Faction (Warrior Guild)              │
// │ Members: 47 warriors                       │
// │ Leader: War Chief Ironwind                 │
// │ Territory: Northern Mountains              │
// │                                            │
// │ [View Full Faction]                        │
// └─────────────────────────────────────────────┘
```

**Nested Navigation**:
- Hover over entity name within panel
- Show nested panel for that entity
- Stack multiple panels for deep exploration
- Keyboard shortcuts to navigate panel stack

### Comparison Panels

**Concept**: Compare multiple entities side-by-side.

```javascript
// Ctrl+Hover over crystal_sword and phoenix_blade
plug.Equipment([crystal_sword, phoenix_blade])

// Side-by-side comparison panel:
// ┌──────────────────────┬──────────────────────┐
// │ ⚔️ Crystal Sword     │ ⚔️ Phoenix Blade     │
// │ ────────────────────│ ────────────────────│
// │ Damage: 2d6+2       │ Damage: 3d6+3       │
// │ Weight: 3.5 lbs     │ Weight: 4.2 lbs     │
// │ Rarity: Rare        │ Rarity: Legendary   │
// │ Value: 500 gold     │ Value: 5000 gold    │
// │                     │                     │
// │ **Special**         │ **Special**         │
// │ • Crystal Shards    │ • Phoenix Fire      │
// │                     │ • Resurrection      │
// └──────────────────────┴──────────────────────┘
```

**Comparison Features**:
- Select multiple entities to compare
- Aligned attribute display
- Highlight differences
- Sort by attribute values

### Historical State

**Concept**: Show entity state at different points in time.

```javascript
// Shift+Hover over kael_ironwind
plug.Owner(kael_ironwind)

// Panel with timeline:
// ┌─────────────────────────────────────────────┐
// │ 👤 Kael Ironwind                           │
// │ ───────────────────────────────────────────│
// │ [Now] [1 hour ago] [1 day ago] [1 week ago]│
// │                                            │
// │ **Current State**                          │
// │ Health: 85/100                             │
// │ Level: 12                                  │
// │ Location: Iron Forge                       │
// │                                            │
// │ **1 Day Ago**                              │
// │ Health: 100/100                            │
// │ Level: 11                                  │
// │ Location: Mountain Path                    │
// │                                            │
// │ Changes: +1 Level, -15 Health, Location    │
// └─────────────────────────────────────────────┘
```

**Historical Features**:
- Timeline scrubber
- Diff view (show changes)
- Playback mode (animate changes)
- Query MongoDB change streams

## Implementation Patterns

### Language Server Protocol Integration

**Concept**: Use LSP hover provider for entity information.

```typescript
// Lost Mountain Games: Entity Hover Provider
class PlugHoverProvider implements vscode.HoverProvider {
  async provideHover(
    document: vscode.TextDocument,
    position: vscode.Position
  ): Promise<vscode.Hover | undefined> {

    // Parse entity reference at cursor
    const entityName = this.getEntityAtPosition(document, position)
    if (!entityName) return undefined

    // Fetch entity data (file-based or MongoDB)
    const entity = await this.fetchEntity(entityName)
    if (!entity) return undefined

    // Build rich markdown content
    const markdown = this.buildEntityMarkdown(entity)

    // Return hover with markdown content
    return new vscode.Hover(markdown)
  }

  buildEntityMarkdown(entity: Entity): vscode.MarkdownString {
    const md = new vscode.MarkdownString()
    md.supportHtml = true
    md.isTrusted = true

    // Header with icon and name
    md.appendMarkdown(`### ${this.getIcon(entity.type)} ${entity.name}\n\n`)

    // Core attributes
    md.appendMarkdown(`**Type**: ${entity.type}\n\n`)

    // Dynamic plugs
    for (const [plugName, value] of Object.entries(entity.plugs)) {
      md.appendMarkdown(`• **${plugName}**: ${this.formatValue(value)}\n`)
    }

    // Action buttons (command links)
    md.appendMarkdown(`\n[View Full Entity](command:plug.viewEntity?${entity.id})`)
    md.appendMarkdown(` | [Find References](command:plug.findReferences?${entity.id})`)

    return md
  }
}
```

**LSP Integration**:
- `textDocument/hover` - Request hover information
- **Markdown rendering** - VS Code renders markdown in hover
- **Command links** - Clickable actions in hover panel
- **Async loading** - Fetch entity data asynchronously

### MongoDB Integration

**Concept**: Query MongoDB for live entity state in hover panels.

```typescript
// Lost Mountain Games: Live Entity Data Provider
class LiveEntityDataProvider {
  constructor(private mongoDb: Db) {}

  async fetchLiveEntity(entityId: string): Promise<Entity | undefined> {
    // Query MongoDB for current entity state
    const doc = await this.mongoDb.collection('entities').findOne({
      _id: entityId
    })

    if (!doc) return undefined

    // Transform MongoDB document to Entity interface
    return {
      id: doc._id,
      name: doc.plug.Name,
      type: this.inferType(doc),
      plugs: doc.plug
    }
  }

  async fetchRelatedEntities(entityId: string): Promise<Entity[]> {
    // Find entities that reference this entity
    const references = await this.mongoDb.collection('entities').find({
      $or: [
        { 'plug.Owner': entityId },
        { 'plug.Location': entityId },
        { 'plug.Contains': entityId }
      ]
    }).toArray()

    return references.map(doc => this.transformEntity(doc))
  }
}
```

**Live Data Features**:
- **Real-time queries** - Fetch current entity state
- **Relationship traversal** - Navigate entity graph
- **Cache layer** - Cache frequent queries
- **Fallback** - Use file-based data if MongoDB unavailable

### Incremental Rendering

**Concept**: Render panel incrementally for fast response.

```typescript
// Lost Mountain Games: Incremental Panel Rendering
class IncrementalPanelRenderer {
  async renderPanel(entityId: string): Promise<vscode.MarkdownString> {
    const md = new vscode.MarkdownString()

    // Phase 1: Render immediately available data (cached)
    const cached = this.cache.get(entityId)
    if (cached) {
      md.appendMarkdown(this.renderBasicInfo(cached))
    } else {
      md.appendMarkdown(`Loading ${entityId}...\n`)
    }

    // Phase 2: Fetch live data in background
    this.fetchLiveData(entityId).then(liveData => {
      // Update panel with live data
      this.updatePanel(entityId, liveData)
    })

    return md
  }
}
```

**Performance Pattern**:
1. **Instant display** - Show cached data immediately (<50ms)
2. **Background fetch** - Query MongoDB in background
3. **Progressive update** - Update panel when live data arrives
4. **User sees response** - No waiting for database queries

## Advanced Features

### Entity Graph Visualization

**Concept**: Visualize entity relationships in hover panel.

```javascript
// Hover over kael_ironwind
plug.Owner(kael_ironwind)

// Panel with relationship graph:
// ┌─────────────────────────────────────────────┐
// │ 👤 Kael Ironwind                           │
// │ ───────────────────────────────────────────│
// │ **Relationship Graph**                      │
// │                                            │
// │       [Iron Wolves]                        │
// │            ↑                               │
// │      (member of)                           │
// │            ↑                               │
// │     [Kael Ironwind]                        │
// │        ↙      ↘                           │
// │   (owns)    (located in)                  │
// │      ↙          ↘                         │
// │ [Crystal     [Iron                        │
// │  Sword]      Forge]                       │
// │                                            │
// │ [View Full Graph]                          │
// └─────────────────────────────────────────────┘
```

**Graph Features**:
- ASCII art graph for simple relationships
- Webview panel for complex graphs
- Interactive nodes (click to navigate)
- Expand/collapse relationship chains

### Inline Code Examples

**Concept**: Show code examples in hover panel.

```javascript
// Hover over dice_roll_3d6
plug.DiceRoll(dice_roll_3d6)

// ┌─────────────────────────────────────────────┐
// │ 🎲 3d6 Dice Roll                           │
// │ ───────────────────────────────────────────│
// │ Formula: Roll 3 six-sided dice, sum result │
// │                                            │
// │ **Usage Examples**                         │
// │ ```javascript                              │
// │ plug.SkillCheck(dice_roll_3d6)            │
// │ plug.Damage(dice_roll_3d6, modifier: +2)  │
// │ ```                                        │
// │                                            │
// │ **Expected Results**                       │
// │ • Minimum: 3 (all ones)                   │
// │ • Average: 10-11                          │
// │ • Maximum: 18 (all sixes)                 │
// └─────────────────────────────────────────────┘
```

**Example Features**:
- Syntax-highlighted code
- Copy code to clipboard
- Run example in debugger
- Link to full documentation

### Custom Panel Renderers

**Concept**: Support custom rendering for specific entity types.

```typescript
// Lost Mountain Games: Custom Panel Renderers
interface PanelRenderer {
  canRender(entity: Entity): boolean
  render(entity: Entity): vscode.MarkdownString
}

class CharacterPanelRenderer implements PanelRenderer {
  canRender(entity: Entity): boolean {
    return entity.type === 'character'
  }

  render(entity: Entity): vscode.MarkdownString {
    const md = new vscode.MarkdownString()

    // Character-specific rendering
    md.appendMarkdown(`### 👤 ${entity.name}\n\n`)
    md.appendMarkdown(`**Class**: ${entity.plugs.Class}\n`)
    md.appendMarkdown(`**Level**: ${entity.plugs.Level}\n`)

    // Health bar visualization
    const healthBar = this.renderHealthBar(
      entity.plugs.Health,
      entity.plugs.MaxHealth
    )
    md.appendMarkdown(`\n${healthBar}\n`)

    return md
  }

  renderHealthBar(current: number, max: number): string {
    const percent = (current / max) * 100
    const filled = Math.floor(percent / 10)
    const empty = 10 - filled

    return `Health: [${'█'.repeat(filled)}${'░'.repeat(empty)}] ${current}/${max}`
  }
}

// Register custom renderers
const renderers: PanelRenderer[] = [
  new CharacterPanelRenderer(),
  new LocationPanelRenderer(),
  new ItemPanelRenderer(),
  new QuestPanelRenderer()
]
```

**Custom Renderer Benefits**:
- **Type-specific layouts** - Optimized for each entity type
- **Rich visualizations** - Health bars, stat blocks, maps
- **Extensible** - Add new renderers without modifying core
- **Fallback** - Default renderer for unknown types

## Performance Optimization

### Lazy Data Loading

**Concept**: Load expensive data only when panel is expanded.

```typescript
// Lost Mountain Games: Lazy Panel Loading
class LazyPanelLoader {
  async renderPanel(entity: Entity): Promise<vscode.MarkdownString> {
    const md = new vscode.MarkdownString()

    // Always render: Basic info (fast)
    md.appendMarkdown(this.renderBasicInfo(entity))

    // Render on demand: Relationships (slow)
    md.appendMarkdown(`\n[Show Relationships](command:plug.loadRelationships?${entity.id})`)

    // Render on demand: Full history (very slow)
    md.appendMarkdown(` | [Show History](command:plug.loadHistory?${entity.id})`)

    return md
  }
}
```

**Lazy Loading Benefits**:
- **Fast initial render** - Only show essential info
- **User controls loading** - Click to load expensive data
- **Better UX** - No waiting for unused data
- **Reduced database load** - Only query when needed

### Panel Caching

**Concept**: Cache rendered panels for instant display.

```typescript
// Lost Mountain Games: Panel Cache
class PanelCache {
  private cache = new Map<string, CachedPanel>()

  async getPanel(entityId: string): Promise<vscode.MarkdownString> {
    const cached = this.cache.get(entityId)

    // Return cached panel if fresh (<30 seconds old)
    if (cached && Date.now() - cached.timestamp < 30000) {
      return cached.content
    }

    // Render new panel
    const panel = await this.renderPanel(entityId)

    // Cache for next time
    this.cache.set(entityId, {
      content: panel,
      timestamp: Date.now()
    })

    return panel
  }
}
```

**Cache Strategy**:
- **30-second TTL** - Balance freshness vs performance
- **LRU eviction** - Keep most-used panels
- **Invalidation** - Clear cache on entity changes
- **Size limit** - Max 1000 cached panels

## Attribution

The floating entity information panel concepts documented here were pioneered by **Lost Mountain Games** (2025), providing rich contextual information without disrupting developer workflow.

**Innovation**: Rich markdown panels, nested entity information, live MongoDB integration, and incremental rendering for optimal developer experience.

---

**Copyright © 2025 Lost Mountain Games**
