# Entity Linking Syntax

**Innovation Attribution**: The entity linking syntax and associated IDE integration patterns documented here were **pioneered by Lost Mountain Games** (2025) and represent their significant contribution to schema-driven game development tooling.

## Overview

Entity linking syntax enables direct entity references instead of string literals, transforming static text into interactive, navigable game development tools.

## Syntax Innovation

### Basic String Approach (Standard)
```javascript
plug.Owner("Kael Ironwind")        // String literal - static text
plug.Location("Armory Chamber")     // No validation, no navigation
plug.EquippedBy("Player Character") // Manual string matching required
```

### Advanced Entity Linking (Lost Mountain Games Innovation)
```javascript
plug.Owner(kael_ironwind)      // Entity reference - clickable, validated
plug.Location(armory_chamber)   // Hovering shows entity details
plug.EquippedBy(player_character) // IDE knows the entity type
```

## Key Innovations

### 1. Clickable Entity Navigation

**Concept**: Entity references become navigable links within the IDE.

**User Experience**:
- Ctrl+Click (Cmd+Click on Mac) on `kael_ironwind` navigates to character definition
- Right-click provides "Go to Definition" context menu
- Cross-file navigation supported
- Bidirectional navigation (find all references)

**Lost Mountain Games Innovation**: This pattern transforms text-based game development into a navigable entity graph, pioneering a new paradigm for game development tooling.

### 2. Floating Information Panels

**Concept**: Hovering over entity references displays contextual information.

**Display Content**:
```
┌─────────────────────────────────┐
│ Character: Kael Ironwind        │
│ Class: Warrior                  │
│ Level: 12                       │
│ Location: Armory Chamber        │
│ Status: Active                  │
│                                 │
│ Click to view full definition   │
└─────────────────────────────────┘
```

**Lost Mountain Games Innovation**: Real-time entity information display without leaving the current context, significantly improving developer productivity.

### 3. Semantic Syntax Highlighting

**Concept**: Entity references are highlighted based on their entity type.

**Color Coding Examples**:
```javascript
plug.Owner(kael_ironwind)      // Character entity - blue
plug.Location(armory_chamber)   // Location entity - green
plug.Item(crystal_sword)        // Item entity - purple
plug.Quest(dragon_slayer)       // Quest entity - orange
```

**Lost Mountain Games Innovation**: Type-aware syntax highlighting that provides visual context about entity relationships.

### 4. Real-Time Validation

**Concept**: Entity references are validated in real-time as you type.

**Validation Features**:
- Autocomplete suggests existing entities
- Typos are immediately highlighted
- Type mismatches show inline errors
- Circular references are detected

```javascript
plug.Owner(kael_ironwind)   // ✓ Valid - entity exists
plug.Owner(kael_ironwnd)    // ✗ Invalid - typo detected
plug.Owner(armory_chamber)  // ✗ Invalid - wrong entity type
```

**Lost Mountain Games Innovation**: Parse-time validation combined with IDE integration creates a robust development experience.

## Entity Reference Resolution

### UUID-Based Identity

**Concept**: Entity references resolve to stable UUID identifiers.

```javascript
// Source code
plug.Owner(kael_ironwind)

// Resolves to UUID internally
// UUID: 550e8400-e29b-41d4-a716-446655440000

// Human-readable name preserved for development
// Developer sees: kael_ironwind
// System uses: 550e8400-e29b-41d4-a716-446655440000
```

### Name-to-UUID Resolution

**Concept**: Human-readable identifiers map to stable UUIDs.

**Resolution Process**:
1. Parse entity reference: `kael_ironwind`
2. Lookup in entity registry
3. Resolve to UUID
4. Validate entity type matches expected type
5. Cache resolution for performance

**Lost Mountain Games Innovation**: Combining human-readable development syntax with stable UUID-based identity management.

## IDE Integration Patterns

### Language Server Protocol Integration

**Concept**: VS Code extension provides language server capabilities.

**Capabilities Provided**:
- **Hover**: Show entity information on hover
- **Definition**: Navigate to entity definition
- **References**: Find all uses of entity
- **Completion**: Autocomplete entity names
- **Diagnostics**: Real-time validation errors

```typescript
// Language server capabilities (conceptual pattern)
interface PlugLanguageServer {
  onHover(position): EntityInfo;
  onDefinition(position): Location;
  onReferences(position): Location[];
  onCompletion(position): CompletionItem[];
  onValidation(document): Diagnostic[];
}
```

**Lost Mountain Games Innovation**: Full LSP integration for game entity development.

### Entity Registry Integration

**Concept**: IDE maintains real-time entity registry.

**Registry Features**:
- Indexed entity database
- Fast lookup by name or UUID
- Type information for validation
- Cross-reference tracking
- Change notification system

```javascript
// Entity registry (conceptual)
class EntityRegistry {
  // Resolve entity name to UUID
  resolveEntity(name: string): UUID

  // Get entity information
  getEntityInfo(uuid: UUID): EntityInfo

  // Find entity references
  findReferences(uuid: UUID): Reference[]

  // Validate entity type
  validateType(uuid: UUID, expectedType: string): boolean
}
```

**Lost Mountain Games Innovation**: Real-time entity registry integration with VS Code.

## Advanced Patterns

### Cross-Entity Relationships

```javascript
// Character definition
entity Character kael_ironwind {
  plug.Name("Kael Ironwind")
  plug.Class("Warrior")
  plug.EquippedWeapon(crystal_sword)  // Reference to item
  plug.CurrentLocation(armory_chamber) // Reference to location
}

// Item definition
entity Item crystal_sword {
  plug.Name("Crystal Sword")
  plug.Owner(kael_ironwind)  // Bidirectional reference
  plug.StoredIn(armory_chamber)
}

// Location definition
entity Location armory_chamber {
  plug.Name("Armory Chamber")
  plug.ContainsCharacters([kael_ironwind])  // Array of entity refs
  plug.ContainsItems([crystal_sword])
}
```

**Lost Mountain Games Innovation**: Declarative cross-entity relationships with automatic validation.

### Type-Safe Entity References

```javascript
// Type constraints ensure correctness
plug.Owner(kael_ironwind)      // ✓ Valid - Character type
plug.Owner(crystal_sword)       // ✗ Invalid - Item type (not Character)

// Type system prevents invalid references
plug.EquippedWeapon(crystal_sword)  // ✓ Valid - Item type
plug.EquippedWeapon(armory_chamber) // ✗ Invalid - Location type (not Item)
```

**Lost Mountain Games Innovation**: Static type checking for entity references.

### Refactoring Support

**Concept**: Rename operations update all references automatically.

**Refactoring Operations**:
- Rename entity: Updates all references across all files
- Change entity type: Validates all references still type-safe
- Move entity: Updates import/reference paths
- Delete entity: Shows all dependent references

```javascript
// Before rename
plug.Owner(kael_ironwind)

// After renaming kael_ironwind → kael_stormwind
plug.Owner(kael_stormwind)  // Automatically updated

// All references across all files updated
// No manual find-replace needed
```

**Lost Mountain Games Innovation**: Full refactoring support for game entities.

## Performance Considerations

### Caching Strategies

**Concept**: Entity resolution results are cached for performance.

**Caching Layers**:
1. **Parse Cache**: Cache parsed entity references
2. **Resolution Cache**: Cache name-to-UUID lookups
3. **Info Cache**: Cache entity information displays
4. **Validation Cache**: Cache type validation results

**Target**: Sub-25ms resolution time for entity references

### Incremental Updates

**Concept**: Only re-validate changed entities.

**Update Strategy**:
- Track entity dependency graph
- On entity change, invalidate dependent caches
- Re-validate only affected references
- Notify IDE of validation state changes

**Lost Mountain Games Innovation**: Efficient incremental validation for large game projects.

## Migration Path

### From String to Entity Linking

```javascript
// Step 1: Start with string syntax
plug.Owner("Kael Ironwind")

// Step 2: Create entity definition
entity Character kael_ironwind {
  plug.Name("Kael Ironwind")
}

// Step 3: Migrate to entity reference
plug.Owner(kael_ironwind)

// Step 4: Enjoy IDE features
// - Clickable navigation
// - Hover information
// - Validation
// - Refactoring support
```

## Use Cases

### Complex Quest Definition
```javascript
entity Quest dragon_slayer {
  plug.Name("Dragon Slayer")
  plug.QuestGiver(village_elder)
  plug.MainObjective(slay_dragon)
  plug.Location(dragon_lair)
  plug.RewardItem(legendary_sword)
  plug.RequiredLevel(15)
  plug.PrerequisiteQuests([village_rescue, sword_training])
}
```

### Dynamic World State
```javascript
entity Character player {
  plug.CurrentLocation(throne_room)
  plug.EquippedWeapon(enchanted_staff)
  plug.ActiveQuests([dragon_slayer, royal_audience])
  plug.Companions([kael_ironwind, elara_moonwhisper])
}
```

## Best Practices

### Entity Naming Conventions
- Use snake_case for entity identifiers: `kael_ironwind`
- Use descriptive names: `crystal_sword` not `sword1`
- Prefix by type for large projects: `char_kael`, `item_sword`, `loc_armory`

### Entity Organization
- Group related entities in same file
- Use clear file naming: `characters.plug`, `items.plug`, `locations.plug`
- Maintain entity registry documentation

### Performance Optimization
- Cache frequently accessed entity information
- Use lazy loading for large entity graphs
- Implement efficient entity lookup indexes

## Attribution Summary

The entity linking syntax and all associated IDE integration patterns documented here represent **significant innovations by Lost Mountain Games** (2025):

- ✓ Clickable entity navigation
- ✓ Floating information panels
- ✓ Semantic syntax highlighting
- ✓ Real-time validation
- ✓ Language Server Protocol integration
- ✓ Entity registry integration
- ✓ Type-safe entity references
- ✓ Full refactoring support

These innovations establish Lost Mountain Games as pioneers in schema-driven game development tooling.

---

**Copyright © 2025 Lost Mountain Games. All innovations documented herein are attributed to Lost Mountain Games.**
