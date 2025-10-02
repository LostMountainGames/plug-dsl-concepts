# Validation Patterns

## Overview

This document describes validation concepts and patterns for Plug DSL, ensuring data integrity and type safety at parse time.

## Validation Layers

### 1. Syntax Validation

**Concept**: Validate correct Plug DSL syntax structure.

```javascript
// Valid syntax
plug.Name("Crystal Sword")

// Invalid syntax - detected immediately
plug.Name("Crystal Sword"  // Missing closing parenthesis
plug.Name()                // Missing required value
plug.Name                  // Missing function call syntax
```

### 2. Type Validation

**Concept**: Validate value types match schema expectations.

```javascript
// Schema definition (conceptual)
const PropertySchemas = {
  Name: { type: "string", required: true },
  Weight: { type: "number", required: false },
  IsEquippable: { type: "boolean", required: false }
}

// Valid - types match schema
plug.Name("Crystal Sword")    // string ✓
plug.Weight(3.5)              // number ✓
plug.IsEquippable(true)       // boolean ✓

// Invalid - type mismatches
plug.Weight("heavy")          // string when number expected ✗
plug.IsEquippable("yes")      // string when boolean expected ✗
```

### 3. Constraint Validation

**Concept**: Validate values meet defined constraints.

```javascript
// Constraint definitions (conceptual)
const WeightConstraints = {
  type: "number",
  min: 0,
  max: 1000
}

const RarityConstraints = {
  type: "string",
  enum: ["Common", "Uncommon", "Rare", "Legendary"]
}

// Valid - within constraints
plug.Weight(3.5)              // 0 <= 3.5 <= 1000 ✓
plug.Rarity("Legendary")      // In enum list ✓

// Invalid - violates constraints
plug.Weight(-5)               // Below minimum ✗
plug.Weight(2000)             // Above maximum ✗
plug.Rarity("Epic")           // Not in enum list ✗
```

### 4. Reference Validation

**Concept**: Validate entity references exist and have correct types.

```javascript
// Valid entity reference
plug.Owner(kael_ironwind)     // Entity exists, type is Character ✓

// Invalid entity references
plug.Owner(nonexistent_entity)  // Entity doesn't exist ✗
plug.Owner(crystal_sword)       // Entity exists but wrong type (Item, not Character) ✗
```

## Validation Timing

### Parse-Time Validation

**Concept**: Validate as Plug DSL is parsed.

**Benefits**:
- Immediate feedback to developer
- Prevents invalid data from entering system
- Reduces runtime errors
- Improves developer experience

**Validation Steps**:
1. Tokenize Plug DSL syntax
2. Validate syntax structure
3. Resolve entity references
4. Validate types against schema
5. Check constraints
6. Report errors with line/column information

### Schema-Driven Validation

**Concept**: Validation rules defined in schemas, not hardcoded.

```javascript
// Schema defines validation rules
const ItemSchema = {
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
      type: "string",
      required: false,
      enum: ["Common", "Uncommon", "Rare", "Legendary"]
    }
  }
}

// Validation uses schema rules
validatePlugProperty("Name", "Crystal Sword", ItemSchema)  // ✓
validatePlugProperty("Weight", 3.5, ItemSchema)            // ✓
validatePlugProperty("Weight", -5, ItemSchema)             // ✗
```

## Error Handling Concepts

### Error Message Design

**Concept**: Clear, actionable error messages.

**Good Error Message**:
```
Error: Invalid property value
  Location: items.plug:15:14
  Property: Weight
  Expected: number between 0 and 1000
  Received: -5
  Fix: Change -5 to a value >= 0

  15 |   plug.Weight(-5)
     |              ^^^
```

**Poor Error Message**:
```
Error: Validation failed
```

### Error Categories

**Syntax Errors**:
```javascript
plug.Name("Crystal Sword"  // Missing closing parenthesis

// Error: Syntax error - expected closing parenthesis
//   Line 1, Column 27
```

**Type Errors**:
```javascript
plug.Weight("heavy")  // String when number expected

// Error: Type mismatch
//   Property: Weight
//   Expected: number
//   Received: string
```

**Constraint Errors**:
```javascript
plug.Rarity("Epic")  // Not in allowed enum values

// Error: Invalid enum value
//   Property: Rarity
//   Allowed: "Common", "Uncommon", "Rare", "Legendary"
//   Received: "Epic"
```

**Reference Errors**:
```javascript
plug.Owner(nonexistent_entity)  // Entity doesn't exist

// Error: Entity reference not found
//   Property: Owner
//   Reference: nonexistent_entity
//   Available entities: kael_ironwind, elara_moonwhisper, ...
```

## Validation Patterns

### Required Properties

**Concept**: Ensure required properties are present.

```javascript
// Schema with required properties
const CharacterSchema = {
  properties: {
    Name: { type: "string", required: true },
    Class: { type: "string", required: true },
    Level: { type: "number", required: false }
  }
}

// Valid - all required properties present
entity Character kael_ironwind {
  plug.Name("Kael Ironwind")
  plug.Class("Warrior")
}

// Invalid - missing required property
entity Character broken_character {
  plug.Name("Broken Character")
  // Missing required Class property ✗
}
```

### Conditional Validation

**Concept**: Validation rules depend on other property values.

```javascript
// Conceptual conditional validation
if (entity.IsEquippable === true) {
  // When equippable, EquipSlot is required
  require(entity.EquipSlot)
}

// Valid
entity Item helmet {
  plug.IsEquippable(true)
  plug.EquipSlot("Head")  // Required when IsEquippable=true
}

// Invalid
entity Item broken_item {
  plug.IsEquippable(true)
  // Missing required EquipSlot ✗
}
```

### Cross-Property Validation

**Concept**: Validate relationships between properties.

```javascript
// Conceptual cross-property validation
validate(entity.MinDamage <= entity.MaxDamage)

// Valid
plug.MinDamage(10)
plug.MaxDamage(20)  // 10 <= 20 ✓

// Invalid
plug.MinDamage(20)
plug.MaxDamage(10)  // 20 <= 10 ✗
```

### Custom Validation Functions

**Concept**: Schema-defined custom validation logic.

```javascript
// Custom validator (conceptual)
const validators = {
  isValidItemName: (name) => {
    // No special characters except spaces and hyphens
    return /^[a-zA-Z0-9 -]+$/.test(name)
  }
}

// Schema with custom validator
const ItemSchema = {
  properties: {
    Name: {
      type: "string",
      validators: ["isValidItemName"]
    }
  }
}

// Valid
plug.Name("Crystal Sword")        // ✓
plug.Name("Sword-of-Light")       // ✓

// Invalid
plug.Name("Sword@#$")             // ✗ Contains invalid characters
```

## Performance Validation

### Validation Caching

**Concept**: Cache validation results for unchanged entities.

**Caching Strategy**:
1. Compute hash of entity definition
2. Check validation cache for hash
3. If cached, return cached result
4. If not cached, validate and cache result
5. Invalidate cache on entity modification

**Performance Target**: Sub-5ms validation for cached entities

### Incremental Validation

**Concept**: Re-validate only changed properties.

**Strategy**:
1. Track which properties changed
2. Re-validate only changed properties
3. Re-validate dependent properties
4. Keep validation results for unchanged properties

**Example**:
```javascript
// Initial entity
entity Item sword {
  plug.Name("Crystal Sword")    // Validated ✓
  plug.Weight(3.5)               // Validated ✓
  plug.Rarity("Legendary")       // Validated ✓
}

// Modify one property
plug.Weight(4.0)  // Only re-validate Weight property

// Name and Rarity validation results reused from cache
```

## Validation API Concepts

### Validation Functions

```typescript
// Conceptual validation API
interface ValidationAPI {
  // Validate entire entity
  validateEntity(entity: Entity): ValidationResult

  // Validate single property
  validateProperty(propertyName: string, value: any, schema: Schema): ValidationResult

  // Validate entity reference
  validateReference(reference: EntityRef, expectedType: string): ValidationResult

  // Custom validation
  validateCustom(value: any, validator: ValidatorFunction): ValidationResult
}
```

### ValidationResult Structure

```typescript
// Conceptual validation result
interface ValidationResult {
  isValid: boolean
  errors: ValidationError[]
  warnings: ValidationWarning[]
}

interface ValidationError {
  severity: "error"
  message: string
  location: Location
  expectedType?: string
  receivedType?: string
  suggestedFix?: string
}
```

## Best Practices

### Schema Design for Validation

**Clear Constraints**:
```javascript
// Good - explicit constraints
{
  type: "number",
  min: 0,
  max: 100,
  description: "Percentage value from 0 to 100"
}

// Poor - unclear constraints
{
  type: "number",
  description: "Some percentage"
}
```

**Meaningful Enums**:
```javascript
// Good - clear enum values
{ enum: ["Common", "Uncommon", "Rare", "Legendary"] }

// Poor - unclear enum values
{ enum: ["0", "1", "2", "3"] }
```

### Error Message Guidelines

1. **Be Specific**: Indicate exactly what's wrong
2. **Show Location**: Line and column numbers
3. **Suggest Fixes**: Provide actionable suggestions
4. **Include Context**: Show surrounding code
5. **Use Examples**: Demonstrate correct usage

### Validation Testing

```javascript
// Test validation logic with edge cases
testValidation({
  // Valid cases
  valid: [
    { value: 0, constraint: { min: 0, max: 100 } },
    { value: 100, constraint: { min: 0, max: 100 } },
    { value: 50, constraint: { min: 0, max: 100 } }
  ],
  // Invalid cases
  invalid: [
    { value: -1, constraint: { min: 0, max: 100 } },
    { value: 101, constraint: { min: 0, max: 100 } }
  ]
})
```

## Attribution

These validation patterns and error handling concepts were developed by **Lost Mountain Games** as part of their schema-driven game development approach.

**Innovation Attribution**: The combination of parse-time validation, schema-driven constraints, entity reference validation, and developer-friendly error messaging represents Lost Mountain Games' contribution to making game development more robust and productive.

---

**Copyright © 2025 Lost Mountain Games**
