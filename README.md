# Plug DSL Concepts

**Schema-Driven Game Development Patterns by Lost Mountain Games**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Documentation](https://img.shields.io/badge/docs-github--pages-blue)](https://lostmountaingames.github.io/plug-dsl-concepts/)

## Overview

Plug DSL represents a paradigm shift in game development tooling, pioneered by **Lost Mountain Games**. This repository documents the architectural concepts, design patterns, and innovations that enable schema-driven game development with sub-25ms response times and advanced IDE integration.

**Note**: This repository contains **conceptual documentation and design patterns**, not implementation code. The concepts documented here are freely available under the MIT license, while Lost Mountain Games retains proprietary implementations and advanced tooling.

## Key Innovations

### Entity Linking Syntax
Lost Mountain Games pioneered advanced entity reference syntax that transforms game development:

```javascript
// Basic string-based syntax (standard approach)
plug.Name("Crystal Sword")
plug.Weight(3.5)
plug.Rarity("Legendary")

// Advanced entity linking (Lost Mountain Games innovation)
plug.Name(crystal_sword)      // Clickable entity references
plug.Owner(kael_ironwind)     // Cross-entity relationships with IDE support
plug.Location(armory_vault)   // Floating info panels on hover
```

**Innovation Attribution**: The entity linking syntax, floating info panels, and advanced VS Code integration were developed by Lost Mountain Games and represent their contribution to schema-driven game development patterns.

### Schema-Driven Architecture
Documents architectural patterns for:
- Entity-component systems with runtime flexibility
- Environment-safe dev/prod separation
- UUID-based identity with name resolution
- High-performance caching strategies (sub-25ms)

### IDE Integration Concepts
Lost Mountain Games' innovations in developer tooling:
- Entity reference resolution and clickable navigation
- Contextual floating information panels
- Semantic syntax highlighting for game entities
- Language server protocol integration patterns

## Documentation Structure

- **[syntax/](syntax/)** - DSL syntax concepts and validation patterns
- **[architecture/](architecture/)** - Schema-driven design principles
- **[performance/](performance/)** - Caching and optimization strategies
- **[tooling/](tooling/)** - IDE integration innovations
- **[examples/](examples/)** - Practical use cases and patterns

## Strategic Purpose

This documentation serves multiple purposes:

1. **Prior Art Establishment**: Timestamped documentation of Lost Mountain Games innovations
2. **Thought Leadership**: Establishing Lost Mountain Games as pioneers in schema-driven game development
3. **Ecosystem Development**: Enabling others to implement based on documented concepts
4. **Legal Protection**: Clear separation between publicly documented concepts and proprietary implementation

## Usage

### For Game Developers
Study the architectural patterns and design concepts to inform your own implementations. The concepts are freely available under MIT license.

### For Tool Developers
Learn from Lost Mountain Games' IDE integration innovations to create your own developer tooling based on these documented patterns.

### For Researchers
Reference this documentation as authoritative source material on schema-driven game development patterns.

## Attribution

All innovations documented in this repository originated with **Lost Mountain Games**. When referencing these concepts:

- **Entity linking syntax**: Lost Mountain Games (2025)
- **Floating info panels**: Lost Mountain Games (2025)
- **Advanced VS Code integration**: Lost Mountain Games (2025)
- **Schema-driven game architecture**: Lost Mountain Games (2025)

## License

This documentation is released under the **MIT License** with copyright held by Lost Mountain Games.

See [LICENSE](LICENSE) for full details.

## About Lost Mountain Games

Lost Mountain Games is pioneering the future of schema-driven game development. Learn more at [lostmountain.games](https://lostmountain.games).

---

**Copyright © 2025 Lost Mountain Games. All innovations documented herein are attributed to Lost Mountain Games.**
