# Type System

## Overview

This document describes the type system concepts for Plug DSL, enabling type-safe entity definitions with compile-time validation.

## Core Types

### Primitive Types

**String Type**:
```javascript
plug.Name("Crystal Sword")
plug.Description("A legendary blade")

// Type definition
type: "string"
constraints: {
  minLength: 0,
  maxLength: 1000,
  pattern: /regex/  // Optional validation pattern
}
```

**Number Type**:
```javascript
plug.Weight(3.5)
plug.DamageMin(15)
plug.Level(10)

// Type definition
type: "number"
constraints: {
  min: -Infinity,
  max: Infinity,
  integer: false  // Set to true for integers only
}
```

**Boolean Type**:
```javascript
plug.IsEquippable(true)
plug.IsQuestItem(false)

// Type definition
type: "boolean"
```

**Enum Type**:
```javascript
plug.Rarity("Legendary")
plug.EquipSlot("Head")

// Type definition
type: "enum"
values: ["Common", "Uncommon", "Rare", "Legendary"]
```

### Complex Types

**Array Type**:
```javascript
plug.Tags(["magical", "weapon", "sword"])
plug.RequiredQuests([dragon_slayer, village_rescue])

// Type definition
type: "array"
elementType: "string" | "number" | "boolean" | "entity"
minLength: 0
maxLength: Infinity
```

**Entity Reference Type**:
```javascript
plug.Owner(kael_ironwind)
plug.Location(armory_chamber)

// Type definition
type: "entity"
entityType: "Character" | "Location" | "Item" | etc.
```

**Object Type**:
```javascript
plug.Attributes({
  strength: 18,
  dexterity: 14,
  constitution: 16
})

// Type definition
type: "object"
properties: {
  strength: { type: "number" },
  dexterity: { type: "number" },
  constitution: { type: "number" }
}
```

## Type Safety Concepts

### Compile-Time Type Checking

**Concept**: Validate types at parse time, not runtime.

```javascript
// Type-safe - validated at parse time
plug.Weight(3.5)              // number ✓
plug.IsEquippable(true)       // boolean ✓
plug.Rarity("Legendary")      // enum ✓

// Type errors - caught at parse time
plug.Weight("heavy")          // Type error: Expected number, got string
plug.IsEquippable("yes")      // Type error: Expected boolean, got string
plug.Rarity("Epic")           // Type error: "Epic" not in enum values
```

### Entity Type Constraints

**Concept**: Entity references must match expected types.

```javascript
// Schema defines expected entity types
const PropertySchemas = {
  Owner: {
    type: "entity",
    entityType: "Character"  // Must be Character entity
  },
  Location: {
    type: "entity",
    entityType: "Location"  // Must be Location entity
  },
  EquippedWeapon: {
    type: "entity",
    entityType: "Item"  // Must be Item entity
  }
}

// Type-safe entity references
plug.Owner(kael_ironwind)          // ✓ kael_ironwind is Character
plug.Location(armory_chamber)      // ✓ armory_chamber is Location
plug.EquippedWeapon(crystal_sword) // ✓ crystal_sword is Item

// Type errors
plug.Owner(crystal_sword)          // ✗ crystal_sword is Item, not Character
plug.Location(kael_ironwind)       // ✗ kael_ironwind is Character, not Location
```

### Type Inference

**Concept**: Infer types from context when possible.

```javascript
// Explicit typing
plug.Count(10)  // Inferred as number

// Context-based inference
plug.Name(crystal_sword)  // Inferred as entity reference (no quotes)
plug.Name("Crystal Sword") // Inferred as string (has quotes)

// Array type inference
plug.Tags(["magical", "weapon"])  // Inferred as string array
plug.Levels([1, 2, 3, 4, 5])     // Inferred as number array
```

## Schema-Driven Types

### Type Definitions in Schema

**Concept**: Schemas define property types and constraints.

```javascript
// Item entity schema (conceptual)
const ItemSchema = {
  entityType: "Item",
  properties: {
    Name: {
      type: "string",
      required: true,
      minLength: 1,
      maxLength: 100
    },
    Weight: {
      type: "number",
      required: false,
      min: 0,
      max: 1000
    },
    Rarity: {
      type: "enum",
      values: ["Common", "Uncommon", "Rare", "Legendary"],
      required: false,
      default: "Common"
    },
    Owner: {
      type: "entity",
      entityType: "Character",
      required: false
    },
    Tags: {
      type: "array",
      elementType: "string",
      required: false
    }
  }
}
```

### Custom Type Definitions

**Concept**: Define reusable custom types.

```javascript
// Define custom types (conceptual)
type Rarity = "Common" | "Uncommon" | "Rare" | "Legendary"
type EquipSlot = "Head" | "Chest" | "Hands" | "Feet" | "MainHand" | "OffHand"
type Damage = { min: number, max: number }

// Use in schemas
{
  Rarity: { type: "Rarity" },
  EquipSlot: { type: "EquipSlot" },
  Damage: { type: "Damage" }
}

// Usage
plug.Rarity("Legendary")  // Type: Rarity
plug.EquipSlot("Head")    // Type: EquipSlot
plug.Damage({ min: 10, max: 20 })  // Type: Damage
```

## Type Conversion and Coercion

### Automatic Type Conversion

**Concept**: Automatically convert compatible types.

```javascript
// Number to string (when schema allows)
plug.Description(42)  // Converted to "42" if property is string type

// String to number (when unambiguous)
plug.Count("10")  // Converted to 10 if property is number type

// Warning: Explicit conversions preferred
// Rely on schema for type enforcement
```

### Strict Type Mode

**Concept**: Disable automatic conversions for strict type safety.

```javascript
// Strict mode - no automatic conversions
strict: true

plug.Count("10")  // Type error: Expected number, got string
plug.Description(42)  // Type error: Expected string, got number

// Must use correct types
plug.Count(10)           // ✓
plug.Description("42")   // ✓
```

## Type Composition

### Union Types

**Concept**: Property can accept multiple types.

```javascript
// Union type definition (conceptual)
type StringOrNumber = string | number

// Schema with union type
{
  Value: {
    type: ["string", "number"]  // Can be either string or number
  }
}

// Valid values
plug.Value("text")   // ✓ string
plug.Value(42)       // ✓ number
plug.Value(true)     // ✗ boolean not in union
```

### Optional Types

**Concept**: Property may be undefined.

```javascript
// Optional property (conceptual)
type Weight = number | undefined

// Schema with optional property
{
  Weight: {
    type: "number",
    required: false  // Can be omitted
  }
}

// Valid entities
entity Item lightweight {
  plug.Name("Feather")
  // Weight omitted - valid because optional
}

entity Item heavy {
  plug.Name("Anvil")
  plug.Weight(100)  // Included - also valid
}
```

### Nullable Types

**Concept**: Property can be explicitly null.

```javascript
// Nullable type (conceptual)
type Owner = Character | null

// Schema with nullable property
{
  Owner: {
    type: "entity",
    entityType: "Character",
    nullable: true  // Can be null
  }
}

// Valid values
plug.Owner(kael_ironwind)  // ✓ Character entity
plug.Owner(null)           // ✓ Explicitly null
// Owner omitted            // ✗ Must be present (not optional, but nullable)
```

## Generic Types

### Parameterized Types

**Concept**: Types that take type parameters.

```javascript
// Array with generic element type (conceptual)
type Array<T> = T[]

// Usage
type StringArray = Array<string>
type NumberArray = Array<number>
type EntityArray = Array<Character>

// In schemas
{
  Tags: { type: "Array<string>" },
  Levels: { type: "Array<number>" },
  Party: { type: "Array<Character>" }
}
```

### Constrained Generics

**Concept**: Generic types with constraints.

```javascript
// Generic with constraint (conceptual)
type EntityArray<T extends Entity> = T[]

// Valid
type CharacterArray = EntityArray<Character>  // ✓ Character extends Entity
type LocationArray = EntityArray<Location>    // ✓ Location extends Entity

// Invalid
type StringArray = EntityArray<string>  // ✗ string doesn't extend Entity
```

## Type Validation Patterns

### Runtime Type Guards

**Concept**: Validate types at runtime for external data.

```javascript
// Type guard functions (conceptual)
function isString(value: any): value is string {
  return typeof value === "string"
}

function isNumber(value: any): value is number {
  return typeof value === "number" && !isNaN(value)
}

function isEntity(value: any): value is Entity {
  return value && typeof value.uuid === "string"
}

// Usage in validation
if (!isNumber(weight)) {
  throw new ValidationError("Weight must be a number")
}
```

### Schema Validation

**Concept**: Validate values against schema types.

```javascript
// Schema-based validation (conceptual)
function validateValue(value: any, typeDefinition: TypeDefinition): ValidationResult {
  switch (typeDefinition.type) {
    case "string":
      return validateString(value, typeDefinition.constraints)
    case "number":
      return validateNumber(value, typeDefinition.constraints)
    case "entity":
      return validateEntity(value, typeDefinition.entityType)
    case "array":
      return validateArray(value, typeDefinition.elementType)
    default:
      return { isValid: false, error: "Unknown type" }
  }
}
```

## Best Practices

### Strong Typing

**Use Specific Types**:
```javascript
// Good - specific enum type
plug.EquipSlot("Head")  // From predefined enum

// Poor - loose string type
plug.EquipSlot("anywhere")  // No validation
```

**Entity Type Constraints**:
```javascript
// Good - constrained entity type
plug.Owner(kael_ironwind)  // Must be Character type

// Poor - unconstrained reference
plug.Owner(any_entity)  // Could be any entity type
```

### Type Documentation

```javascript
// Document custom types
/**
 * Rarity indicates item rarity level
 * Values: Common, Uncommon, Rare, Legendary
 * Default: Common
 */
type Rarity = "Common" | "Uncommon" | "Rare" | "Legendary"
```

### Type Safety in Schemas

```javascript
// Define comprehensive type constraints
{
  Weight: {
    type: "number",
    min: 0,              // No negative weights
    max: 1000,           // Reasonable maximum
    integer: false,      // Allow decimals
    description: "Item weight in pounds"
  }
}
```

## Performance Considerations

### Type Checking Performance

**Concept**: Efficient type validation.

**Strategies**:
- Cache type validation results
- Use type inference to reduce checks
- Validate only on changes
- Batch validation operations

**Target**: Sub-1ms type validation per property

### Type Index

**Concept**: Indexed type lookup for fast validation.

```javascript
// Type index (conceptual)
const typeIndex = {
  "string": validateString,
  "number": validateNumber,
  "boolean": validateBoolean,
  "entity": validateEntity,
  "array": validateArray
}

// Fast type validation lookup
const validator = typeIndex[typeDefinition.type]
const result = validator(value, typeDefinition)
```

## Attribution

These type system concepts were developed by **Lost Mountain Games** as part of their schema-driven game development approach.

**Innovation Attribution**: The combination of compile-time type checking, entity type constraints, schema-driven types, and performance-optimized validation represents Lost Mountain Games' contribution to type-safe game development.

---

**Copyright © 2025 Lost Mountain Games**
