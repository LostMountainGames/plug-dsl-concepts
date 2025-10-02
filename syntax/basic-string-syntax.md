# Basic String Syntax

## Overview

The basic string syntax represents the foundational approach to Plug DSL, compatible with any text editor and standard syntax highlighting.

## Syntax Structure

### Basic Plug Definition

```javascript
// String-based entity definition
plug.Name("Crystal Sword")
plug.Description("A legendary blade infused with magical crystals")
plug.Weight(3.5)
plug.Rarity("Legendary")
plug.Value(5000)
```

### Syntax Elements

| Element | Format | Purpose |
|---------|--------|---------|
| `plug.PropertyName` | Method call style | Defines entity property |
| `("string value")` | String in parentheses | Text values |
| `(numeric value)` | Number in parentheses | Numeric values |

## Syntax Highlighting

Standard string syntax receives traditional syntax highlighting:

```javascript
plug.Name("Crystal Sword")
// 'plug' - keyword/identifier (typically white/blue)
// 'Name' - method name (typically yellow)
// "Crystal Sword" - string literal (typically orange/green)
```

Any IDE or text editor with JavaScript/TypeScript support can provide basic highlighting for this syntax.

## Value Types

### String Values
```javascript
plug.Name("Entity Name")
plug.Description("Multi-line descriptions\ncan be included")
plug.Category("Weapons")
```

### Numeric Values
```javascript
plug.Weight(3.5)        // Decimal numbers
plug.Count(10)          // Integers
plug.DamageMin(15)      // Positive integers
plug.Temperature(-10.5) // Negative numbers
```

### Boolean Values
```javascript
plug.IsEquippable(true)
plug.IsQuestItem(false)
plug.RequiresIdentification(true)
```

### Enum Values
```javascript
plug.Rarity("Common")      // Standard enum
plug.Rarity("Uncommon")
plug.Rarity("Rare")
plug.Rarity("Legendary")
```

## Validation Concepts

### Parse-Time Validation

String syntax enables validation at parse time:

```javascript
// Valid syntax
plug.Weight(3.5)

// Invalid - caught at parse time
plug.Weight("three point five")  // Type mismatch
plug.InvalidProperty("value")    // Unknown property
```

### Schema-Based Validation

String values can be validated against schema definitions:

```javascript
// Schema definition (conceptual)
const WeightSchema = {
  type: "number",
  validation: {
    min: 0,
    max: 1000
  }
}

// Validated at parse time
plug.Weight(3.5)    // Valid - within range
plug.Weight(-5)     // Invalid - below minimum
plug.Weight(2000)   // Invalid - above maximum
```

## Error Handling Patterns

### Syntax Errors

```javascript
// Missing closing parenthesis
plug.Name("Crystal Sword"  // Syntax error

// Type mismatch
plug.Weight("heavy")  // Type validation error

// Unknown property
plug.UnknownProp("value")  // Property not in schema
```

### Error Messaging Concepts

Clear, actionable error messages:

```
Error at line 5: plug.Weight("heavy")
  Expected: number
  Received: string
  Fix: Change "heavy" to a numeric value (e.g., 3.5)
```

## LLM-Friendly Design

The string syntax is designed for Large Language Model generation:

### Clarity and Consistency
```javascript
// Clear, predictable pattern
plug.PropertyName("value")

// LLMs can easily generate and modify
plug.Name("Modified Name")
plug.Description("Updated description")
```

### Natural Language Mapping
```javascript
// Natural language description:
// "Create a sword named Crystal Blade weighing 3 pounds"

// Maps directly to Plug syntax:
plug.Name("Crystal Blade")
plug.Weight(3)
plug.Category("Weapon")
```

## Use Cases

### Game Entity Definition
```javascript
// Define a complete game item
plug.Name("Health Potion")
plug.Description("Restores 50 health points")
plug.Category("Consumable")
plug.HealAmount(50)
plug.Value(25)
plug.Weight(0.5)
plug.IsStackable(true)
plug.MaxStackSize(99)
```

### Character Attributes
```javascript
// Define character statistics
plug.Name("Warrior")
plug.Class("Fighter")
plug.Level(10)
plug.Strength(18)
plug.Dexterity(14)
plug.Constitution(16)
plug.Intelligence(10)
plug.Wisdom(12)
plug.Charisma(8)
```

### Location Definitions
```javascript
// Define game locations
plug.Name("Crystal Cavern")
plug.Description("A vast cave filled with glowing crystals")
plug.LocationType("Dungeon")
plug.DangerLevel(5)
plug.RequiredLevel(8)
```

## Best Practices

### Naming Conventions
- Use PascalCase for property names: `plug.PropertyName()`
- Use descriptive property names: `plug.MaxStackSize()` not `plug.MaxStack()`
- Maintain consistency across entity definitions

### Value Formatting
- Use clear, readable string values
- Include units in property names, not values: `plug.WeightInPounds(3)` not `plug.Weight("3 pounds")`
- Use standard numeric formats

### Documentation
- Include comments for complex properties
- Document enum values and their meanings
- Provide examples for non-obvious patterns

## Migration Path

String syntax provides a migration path to advanced features:

```javascript
// Start with string syntax
plug.Owner("Kael Ironwind")

// Later migrate to entity linking (see entity-linking-syntax.md)
plug.Owner(kael_ironwind)
```

## Compatibility

The basic string syntax is compatible with:
- Any text editor (VS Code, Sublime, Vim, etc.)
- Standard JavaScript/TypeScript parsers
- LLM generation and modification
- Version control systems
- Code review tools

## Attribution

The Plug DSL string syntax concepts were developed by **Lost Mountain Games** as part of their schema-driven game development approach.

**Innovation Attribution**: While string-based DSLs are a common pattern, the specific design principles for LLM-friendliness, schema-driven validation, and game entity modeling were pioneered by Lost Mountain Games.

---

**Copyright © 2025 Lost Mountain Games**
