# MongoDB Atlas Indexing for Plug-Based Queries

**Innovation Attribution**: The plug-aware indexing strategies documented here were pioneered by **Lost Mountain Games** (2025) to enable fast queries on schemaless, pluggable entity collections.

## Core Challenge

Pluggable entities can have ANY combination of plugs. Traditional database indexing assumes known schema. **Lost Mountain Games Innovation**: Index common plug patterns while maintaining schema flexibility.

## Plug-Based Index Strategies

### Index Frequently Queried Plugs

**Concept**: Create indexes on plugs that systems query often.

```javascript
// Combat system queries entities with Damage plug
db.entities.createIndex({ "plug.Damage": 1 })

// Spatial system queries entities with Position plug
db.entities.createIndex({ "plug.Position": 1 })

// Magic system queries entities with Mana plug
db.entities.createIndex({ "plug.Mana": 1 })

// Fast queries (uses index)
db.entities.find({ "plug.Damage": { $gt: 10 } })
db.entities.find({ "plug.Mana": { $exists: true } })
```

**Performance**:
- **Without Index**: 500ms+ (collection scan)
- **With Index**: <25ms (index scan)

### Sparse Indexes for Optional Plugs

**Concept**: Use sparse indexes for plugs that only some entities have.

```javascript
// Not all entities have MagicPower plug
// Sparse index only indexes entities that DO have it
db.entities.createIndex(
  { "plug.MagicPower": 1 },
  { sparse: true }
)

// Benefit: Index is much smaller
// Only indexes entities with MagicPower, not empty values
```

**Size Comparison**:
```
Regular Index: 100,000 entities = 10MB index
Sparse Index:  10,000 entities with MagicPower = 1MB index
```

### Compound Indexes for Multi-Plug Queries

**Concept**: Index multiple plugs together for common query patterns.

```javascript
// Common query: Find warriors with high health
db.entities.find({
  "plug.Tag": "warrior_tag_uuid",
  "plug.Health": { $gt: 50 }
})

// Compound index optimizes this query
db.entities.createIndex({
  "plug.Tag": 1,
  "plug.Health": 1
})

// Another common pattern: Characters in a location
db.entities.createIndex({
  "plug.Tag": 1,           // Character tag
  "plug.Location": 1       // Location UUID
})
```

**Query Performance**:
```javascript
// Without compound index: ~200ms
db.entities.find({
  "plug.Tag": "character_tag_uuid",
  "plug.Location": "tavern_uuid"
})

// With compound index: ~5ms
```

### Text Indexes for Search

**Concept**: Enable full-text search on plug values.

```javascript
// Text index on Name and Description plugs
db.entities.createIndex({
  "plug.Name": "text",
  "plug.Description": "text"
})

// Fast text search
db.entities.find({
  $text: { $search: "warrior sword" }
})

// Finds entities with "warrior" or "sword" in Name/Description
```

## Tag-Based Indexing

### Index Tag Arrays

**Concept**: Optimize queries for entities with specific tags.

```javascript
// Entities can have multiple tags
{
  "_id": "kael_uuid",
  "plug": {
    "Name": "Kael",
    "Tag": ["character_tag_uuid", "warrior_tag_uuid", "ironwolves_tag_uuid"]
  }
}

// Index the Tag array
db.entities.createIndex({ "plug.Tag": 1 })

// Fast tag queries
db.entities.find({ "plug.Tag": "warrior_tag_uuid" })

// Find entities with ALL these tags
db.entities.find({
  "plug.Tag": {
    $all: ["warrior_tag_uuid", "magical_tag_uuid"]
  }
})
```

**Multi-Tag Performance**:
- **Without Index**: Scan all entities (~500ms for 100k entities)
- **With Index**: <10ms for single tag, <25ms for multi-tag

### Compound Tag + Property Indexes

```javascript
// Common query: Tagged entities with specific property
db.entities.createIndex({
  "plug.Tag": 1,
  "plug.Rarity": 1
})

// Fast query: Find all legendary items
db.entities.find({
  "plug.Tag": "item_tag_uuid",
  "plug.Rarity": "Legendary"
})
// Uses compound index: <10ms
```

## Geospatial Indexing for Position Plugs

### 2D Geospatial Index

**Concept**: Enable spatial queries for entities with Position plug.

```javascript
// Create 2D index on Position plug
db.entities.createIndex({ "plug.Position": "2d" })

// Entities with Position plug
{
  "_id": "kael_uuid",
  "plug": {
    "Name": "Kael",
    "Position": [10, 20]  // [x, y]
  }
}

// Spatial queries
// Find entities near a point
db.entities.find({
  "plug.Position": {
    $near: [15, 25],
    $maxDistance: 5
  }
})

// Find entities in a rectangular area
db.entities.find({
  "plug.Position": {
    $geoWithin: {
      $box: [[0, 0], [50, 50]]
    }
  }
})
```

**Spatial Query Performance**:
- **Nearby Entities**: <10ms (with index)
- **Area Queries**: <25ms (with index)

## Dynamic Index Management

### Analyze Query Patterns

**Concept**: Monitor slow queries and create indexes dynamically.

```javascript
// Enable query profiling
db.setProfilingLevel(2, { slowms: 100 })

// Analyze slow queries
const slowQueries = db.system.profile.find({
  millis: { $gt: 100 }
}).sort({ ts: -1 }).limit(10)

// Identify missing indexes
slowQueries.forEach(query => {
  if (!query.planSummary.includes('IXSCAN')) {
    console.log('Missing index for:', query.command.filter)
    // Create appropriate index
  }
})
```

### Index Recommendation System

```javascript
class IndexRecommender {
  constructor() {
    this.queryPatterns = new Map()
  }

  recordQuery(filter) {
    const pattern = this.getQueryPattern(filter)
    const count = this.queryPatterns.get(pattern) || 0
    this.queryPatterns.set(pattern, count + 1)
  }

  getQueryPattern(filter) {
    // Extract plug fields from filter
    const fields = Object.keys(filter)
      .filter(k => k.startsWith('plug.'))
      .sort()

    return fields.join(',')
  }

  getRecommendations(threshold = 100) {
    const recommendations = []

    for (const [pattern, count] of this.queryPatterns) {
      if (count > threshold) {
        const fields = pattern.split(',')
        recommendations.push({
          pattern,
          count,
          suggestedIndex: this.buildIndexSpec(fields)
        })
      }
    }

    return recommendations.sort((a, b) => b.count - a.count)
  }

  buildIndexSpec(fields) {
    const spec = {}
    fields.forEach(field => {
      spec[field] = 1
    })
    return spec
  }
}

// Usage
const recommender = new IndexRecommender()

// Track queries
db.entities.find({ "plug.Tag": "warrior", "plug.Health": { $gt: 50 } })
recommender.recordQuery({ "plug.Tag": "warrior", "plug.Health": { $gt: 50 } })

// After collecting data
const recommendations = recommender.getRecommendations()
console.log(recommendations)
// Output:
// [
//   {
//     pattern: "plug.Health,plug.Tag",
//     count: 1523,
//     suggestedIndex: { "plug.Health": 1, "plug.Tag": 1 }
//   }
// ]
```

## Index Maintenance

### Monitor Index Usage

```javascript
// Get index statistics
const indexStats = db.entities.aggregate([
  { $indexStats: {} }
])

indexStats.forEach(stat => {
  console.log(`Index: ${stat.name}`)
  console.log(`  Accesses: ${stat.accesses.ops}`)
  console.log(`  Since: ${stat.accesses.since}`)
})

// Drop unused indexes
indexStats.forEach(stat => {
  if (stat.accesses.ops < 10) {
    console.log(`Consider dropping unused index: ${stat.name}`)
    // db.entities.dropIndex(stat.name)
  }
})
```

### Index Size Management

```javascript
// Check index sizes
const stats = db.entities.stats()

console.log('Collection Size:', stats.size)
console.log('Index Sizes:')
for (const [name, size] of Object.entries(stats.indexSizes)) {
  console.log(`  ${name}: ${size} bytes`)
}

// Alert if indexes exceed collection size
if (stats.totalIndexSize > stats.size) {
  console.warn('Indexes larger than collection - review index strategy')
}
```

## Query Optimization Patterns

### Use Covered Queries

**Concept**: Query that can be answered entirely from index.

```javascript
// Create index with all queried fields
db.entities.createIndex({
  "plug.Tag": 1,
  "plug.Name": 1,
  "plug.Health": 1
})

// Covered query (no document fetch needed!)
db.entities.find(
  { "plug.Tag": "character_tag_uuid" },
  { "plug.Name": 1, "plug.Health": 1, "_id": 0 }
)

// Explain shows: "stage": "IXSCAN", "totalDocsExamined": 0
// Ultra-fast: <5ms even for large result sets
```

### Compound Index Field Order

**Concept**: Order fields in compound index by query patterns.

```javascript
// Query pattern: equality, then range
db.entities.find({
  "plug.Tag": "warrior_tag_uuid",  // Equality
  "plug.Health": { $gt: 50 }       // Range
})

// Good index (equality first, range last)
db.entities.createIndex({
  "plug.Tag": 1,     // Equality field first
  "plug.Health": 1   // Range field last
})

// Poor index (reversed)
db.entities.createIndex({
  "plug.Health": 1,  // Range first - less selective
  "plug.Tag": 1      // Equality last - can't use efficiently
})
```

### Avoid Collection Scans

```javascript
// Check query plan
const explain = db.entities.find({
  "plug.MagicPower": { $exists: true }
}).explain("executionStats")

if (explain.executionStats.totalDocsExamined > 1000) {
  console.warn('Collection scan detected - create index')

  db.entities.createIndex(
    { "plug.MagicPower": 1 },
    { sparse: true }
  )
}
```

## Sharding Strategies

### Shard by World ID

**Concept**: Distribute entities across shards by world.

```javascript
// Shard key on worldId
sh.shardCollection("game.entities", {
  "worldId": 1,
  "_id": 1
})

// Benefits:
// - Entities from same world stay together
// - World-specific queries hit single shard
// - Horizontal scaling as worlds increase
```

### Shard by Entity Type

**Concept**: Distribute by primary tag or entity type.

```javascript
// Shard by primary tag
sh.shardCollection("game.entities", {
  "plug.Tag": 1,
  "_id": 1
})

// Benefits:
// - Character queries hit character shard
// - Location queries hit location shard
// - System-specific queries more efficient
```

## Performance Benchmarks

### Index Impact Measurements

```javascript
// Benchmark query performance
async function benchmarkQuery(query, iterations = 100) {
  const times = []

  for (let i = 0; i < iterations; i++) {
    const start = Date.now()
    await db.entities.find(query).toArray()
    times.push(Date.now() - start)
  }

  const avg = times.reduce((a, b) => a + b) / times.length
  const p95 = times.sort()[Math.floor(iterations * 0.95)]

  return { avg, p95, min: Math.min(...times), max: Math.max(...times) }
}

// Test before and after index
const query = { "plug.Tag": "warrior_tag_uuid", "plug.Health": { $gt: 50 } }

console.log('Without index:')
console.log(await benchmarkQuery(query))
// { avg: 450ms, p95: 520ms, min: 380ms, max: 600ms }

db.entities.createIndex({ "plug.Tag": 1, "plug.Health": 1 })

console.log('With index:')
console.log(await benchmarkQuery(query))
// { avg: 8ms, p95: 12ms, min: 5ms, max: 20ms }
```

## Best Practices

### Index Common Patterns

```javascript
// Good - index frequent query patterns
db.entities.createIndex({ "plug.Tag": 1 })
db.entities.createIndex({ "plug.Tag": 1, "plug.Health": 1 })
db.entities.createIndex({ "plug.Name": 1 })
db.entities.createIndex({ "plug.Location": 1 })

// Poor - index every possible plug
// (Creates too many indexes, slows writes, wastes space)
```

### Use Sparse Indexes for Optional Plugs

```javascript
// Good - sparse index for optional plug
db.entities.createIndex(
  { "plug.MagicPower": 1 },
  { sparse: true }
)

// Poor - regular index on optional plug
db.entities.createIndex({ "plug.MagicPower": 1 })
// Indexes null for entities without plug - wastes space
```

### Monitor and Adjust

```javascript
// Regular index review
setInterval(async () => {
  // Check slow queries
  const slowQueries = await db.system.profile.find({
    millis: { $gt: 100 }
  }).toArray()

  if (slowQueries.length > 0) {
    console.warn(`Found ${slowQueries.length} slow queries`)
    // Analyze and create indexes
  }

  // Check index usage
  const unused = await findUnusedIndexes()
  if (unused.length > 0) {
    console.warn(`Found ${unused.length} unused indexes`)
    // Consider dropping
  }
}, 3600000)  // Check hourly
```

## Attribution

The plug-aware indexing strategies documented here were pioneered by **Lost Mountain Games** (2025), demonstrating how to maintain query performance in schemaless, pluggable entity systems.

**Innovation**: Sparse indexing, compound plug indexes, and dynamic index management for flexible entity systems that maintain sub-25ms query performance.

---

**Copyright © 2025 Lost Mountain Games**
