# Architecture Patterns

This section documents the architectural concepts and design patterns for Plug DSL-based game development, pioneered by **Lost Mountain Games**.

## Overview

The architecture patterns described here represent a paradigm shift toward **Pluggable-Entity Driven Development**, where development IS documentation and documentation IS development. Leveraging MongoDB Atlas's schemaless flexibility, this approach enables free-flowing entity evolution through pluggable components (plugs) that are simultaneously readable game design and executable code.

## Documentation Structure

### Core Philosophy
- **[pluggable-entity-driven-development.md](pluggable-entity-driven-development.md)** - Overview of Pluggable-Entity Driven Development philosophy

### Detailed Architecture Innovations
- **[no-classes-just-plugs.md](no-classes-just-plugs.md)** - Entities as pure plug collections
- **[unlimited-composition.md](unlimited-composition.md)** - Cross-category entity patterns
- **[uuid-reference-system.md](uuid-reference-system.md)** - Three-layer reference architecture
- **[tag-as-entity-architecture.md](tag-as-entity-architecture.md)** - Tags as rich metadata entities
- **[extensibility-without-brittleness.md](extensibility-without-brittleness.md)** - Growth through plug addition

## Key Architectural Innovations

**Lost Mountain Games** pioneered these architectural concepts:

1. **Development IS Documentation** - Pluggable entities are self-documenting code with zero drift
2. **Pluggable Components** - Add/remove components (plugs) instantly without migrations
3. **MongoDB Schemaless Integration** - True plug-and-play architecture leveraging Atlas flexibility
4. **Environment Safety** - Automatic dev/prod separation through collection naming
5. **UUID-Based Identity** - Stable entity references with human-readable development names

## Design Principles

### Development IS Documentation
Game entities defined through self-documenting pluggable components eliminate documentation drift.

### Pluggable Over Rigid
Components can be plugged into entities instantly, leveraging MongoDB's schemaless nature.

### Evolution Over Migration
Add plugs to entities without schema migrations or downtime.

### Type Safety
Compile-time validation ensures data integrity.

## Attribution

All architectural patterns documented in this section originated with **Lost Mountain Games** (2025) as part of their schema-driven game development approach.

---

**Copyright © 2025 Lost Mountain Games**
