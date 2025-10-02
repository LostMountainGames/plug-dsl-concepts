# Caching Strategies

**Innovation Attribution**: The plug-aware caching strategies documented here were pioneered by **Lost Mountain Games** (2025) to achieve sub-25ms response times for pluggable entity systems.

## Core Innovation

Traditional caching strategies cache specific fields or queries. **Lost Mountain Games Innovation**: Cache entire entity plug collections with intelligent invalidation based on plug modifications.

## Multi-Layer Caching Architecture

### Layer 1: Redis Entity Cache

**Concept**: Cache complete entity documents in Redis for instant retrieval.

```javascript
// Cache key structure
const cacheKey = `plug:${worldId}:${entityType}:${entityId}`

// Example: "plug:world-123:character:kael-uuid"

// Cache value: Complete entity plug collection
{
  "_id": "kael-uuid",
  "plug": {
    "Name": "Kael Ironwind",
    "Health": 100,
    "Damage": 15,
    "Mana": 80,
    // ... all plugs
  }
}
```

**Benefits**:
- **Sub-5ms retrieval** from Redis
- **Complete entity** available immediately
- **No MongoDB query** for cache hits

### Layer 2: Query Result Cache

**Concept**: Cache results of plug-based queries.

```javascript
// Cache complex queries
const queryKey = `plug:query:${worldId}:${queryHash}`

// Query: "Find all warriors with magic"
const query = {
  "plug.Tag": "warrior_tag_uuid",
  "plug.Mana": { $exists: true }
}

// Cache key: "plug:query:world-123:hash(query)"
// Cache value: Array of entity IDs
["kael-uuid", "elara-uuid", "theron-uuid"]

// TTL: 5 minutes (queries change frequently)
```

**Benefits**:
- **Fast query responses** without MongoDB aggregation
- **Reduced database load** for common queries
- **Batch retrieval** of matching entities

### Layer 3: Computed Value Cache

**Concept**: Cache expensive computations based on plugs.

```javascript
// Cache computed values that depend on multiple plugs
const computedKey = `plug:computed:${entityId}:${computationType}`

// Example: Total combat power calculation
function getCombatPower(entity) {
  const cacheKey = `plug:computed:${entity._id}:combatPower`

  // Check cache
  const cached = redis.get(cacheKey)
  if (cached) return cached  // <1ms

  // Compute from plugs
  const base = entity.getPlug("Damage") || 0
  const magic = entity.getPlug("MagicPower") || 0
  const multiplier = entity.getPlug("PowerMultiplier") || 1
  const total = (base + magic) * multiplier

  // Cache for 30 minutes
  redis.setex(cacheKey, 1800, total)

  return total
}
```

**Benefits**:
- **Avoid repeated calculations** of complex formulas
- **Plug-dependent caching** invalidates when plugs change
- **Performance improvement** for stat-heavy systems

## Cache Invalidation Strategies

### Plug-Based Invalidation

**Concept**: Invalidate cache when specific plugs change.

```javascript
function updateEntityPlug(entityId, plugName, newValue) {
  // Update MongoDB
  db.entities.updateOne(
    { "_id": entityId },
    { $set: { [`plug.${plugName}`]: newValue } }
  )

  // Invalidate entity cache
  redis.del(`plug:*:*:${entityId}`)

  // Invalidate computed caches that depend on this plug
  const dependentComputations = getComputationsDependingOn(plugName)
  for (const comp of dependentComputations) {
    redis.del(`plug:computed:${entityId}:${comp}`)
  }

  // Invalidate query caches that filter on this plug
  const affectedQueries = getQueriesFilteringOn(plugName)
  for (const queryHash of affectedQueries) {
    redis.del(`plug:query:*:${queryHash}`)
  }
}
```

**Example - Changing Health plug**:
```javascript
// Change Health from 100 to 80
updateEntityPlug("kael-uuid", "Health", 80)

// Invalidates:
// - Entity cache: plug:world-123:character:kael-uuid
// - Computed: plug:computed:kael-uuid:survivalChance
// - Computed: plug:computed:kael-uuid:effectiveHealth
// - Query: plug:query:world-123:hash("Health > 50")
// - Query: plug:query:world-123:hash("Health < 100")
```

### Time-Based Expiration (TTL)

**Concept**: Different plug types have different cache durations.

```javascript
const CacheTTL = {
  // Static plugs (rarely change)
  Name: 3600,           // 1 hour
  Description: 3600,
  Tag: 1800,            // 30 minutes

  // Dynamic plugs (change frequently)
  Health: 300,          // 5 minutes
  Mana: 300,
  Position: 60,         // 1 minute

  // Computed values
  CombatPower: 1800,    // 30 minutes

  // Query results
  QueryResults: 300     // 5 minutes
}

function cacheEntity(entity) {
  const cacheKey = `plug:${entity.worldId}:${entity.type}:${entity._id}`

  // Determine TTL based on plugs
  const hasDynamicPlugs = ["Health", "Mana", "Position"].some(
    plug => entity.hasPlug(plug)
  )

  const ttl = hasDynamicPlugs ? CacheTTL.Health : CacheTTL.Name

  redis.setex(cacheKey, ttl, JSON.stringify(entity))
}
```

### Write-Through Caching

**Concept**: Update cache immediately when writing to database.

```javascript
async function updateEntity(entityId, updates) {
  // 1. Update MongoDB
  await db.entities.updateOne(
    { "_id": entityId },
    { $set: updates }
  )

  // 2. Update cache immediately (write-through)
  const entity = await db.entities.findOne({ "_id": entityId })
  const cacheKey = `plug:${entity.worldId}:${entity.type}:${entityId}`

  await redis.setex(
    cacheKey,
    getCacheTTL(entity),
    JSON.stringify(entity)
  )

  // Cache is now consistent with database
}
```

**Benefits**:
- **Immediate consistency** between cache and database
- **Next read is cached** without waiting
- **Reduced cache misses** after writes

## Performance Patterns

### Batch Cache Retrieval

**Concept**: Retrieve multiple entities from cache in single operation.

```javascript
async function getEntitiesBatch(entityIds, worldId) {
  // Build cache keys
  const cacheKeys = entityIds.map(id =>
    `plug:${worldId}:entity:${id}`
  )

  // Single Redis MGET operation
  const cached = await redis.mget(cacheKeys)

  // Separate hits from misses
  const hits = []
  const misses = []

  cached.forEach((value, index) => {
    if (value) {
      hits.push(JSON.parse(value))
    } else {
      misses.push(entityIds[index])
    }
  })

  // Fetch misses from MongoDB
  if (misses.length > 0) {
    const fromDb = await db.entities.find({
      "_id": { $in: misses }
    }).toArray()

    // Cache the fetched entities
    const pipeline = redis.pipeline()
    fromDb.forEach(entity => {
      const key = `plug:${worldId}:entity:${entity._id}`
      pipeline.setex(key, getCacheTTL(entity), JSON.stringify(entity))
    })
    await pipeline.exec()

    hits.push(...fromDb)
  }

  return hits
}

// Performance: 100 entities
// All cached: ~5ms (single MGET)
// 10 misses: ~15ms (MGET + single MongoDB query)
// vs 100 individual queries: ~1000ms+
```

### Predictive Caching

**Concept**: Pre-cache entities likely to be accessed soon.

```javascript
async function enterLocation(characterId, locationId) {
  // Character enters location
  await updateEntityPlug(characterId, "Location", locationId)

  // Predictive caching: Pre-cache nearby entities
  const location = await getEntity(locationId)
  const nearbyEntities = location.getPlug("Contains") || []

  // Pre-cache all entities in location
  await warmCache(nearbyEntities)

  // When player queries location contents, cache is ready
}

async function warmCache(entityIds) {
  // Fetch entities that aren't cached
  const uncached = []

  for (const id of entityIds) {
    const key = `plug:*:*:${id}`
    const exists = await redis.exists(key)
    if (!exists) uncached.push(id)
  }

  if (uncached.length > 0) {
    const entities = await db.entities.find({
      "_id": { $in: uncached }
    }).toArray()

    // Cache in background
    const pipeline = redis.pipeline()
    entities.forEach(entity => {
      const key = `plug:${entity.worldId}:${entity.type}:${entity._id}`
      pipeline.setex(key, getCacheTTL(entity), JSON.stringify(entity))
    })
    await pipeline.exec()
  }
}
```

### Cache Warming on Startup

**Concept**: Pre-populate cache with frequently accessed entities.

```javascript
async function warmCacheOnStartup() {
  console.log("Warming entity cache...")

  // Cache all active characters
  const activeChars = await db.entities.find({
    "plug.Tag": "character_tag_uuid",
    "plug.IsActive": true
  }).limit(1000).toArray()

  // Cache all locations
  const locations = await db.entities.find({
    "plug.Tag": "location_tag_uuid"
  }).toArray()

  // Cache common items
  const commonItems = await db.entities.find({
    "plug.Tag": "item_tag_uuid",
    "plug.Rarity": "Common"
  }).limit(500).toArray()

  // Batch cache
  const pipeline = redis.pipeline()
  const allEntities = [...activeChars, ...locations, ...commonItems]

  allEntities.forEach(entity => {
    const key = `plug:${entity.worldId}:${entity.type}:${entity._id}`
    pipeline.setex(key, getCacheTTL(entity), JSON.stringify(entity))
  })

  await pipeline.exec()

  console.log(`Warmed cache with ${allEntities.length} entities`)
}
```

## Cache Key Patterns

### Hierarchical Keys

```javascript
// Pattern: plug:{world}:{type}:{id}
"plug:world-123:character:kael-uuid"
"plug:world-123:location:grotto-uuid"
"plug:world-123:item:sword-uuid"

// Enables:
// - Delete all caches for world: redis.del("plug:world-123:*")
// - Delete all character caches: redis.del("plug:*:character:*")
// - Delete specific entity: redis.del("plug:*:*:kael-uuid")
```

### Query Fingerprinting

```javascript
function getQueryCacheKey(worldId, query) {
  // Create stable hash of query
  const queryStr = JSON.stringify(query, Object.keys(query).sort())
  const hash = crypto.createHash('md5').update(queryStr).digest('hex')

  return `plug:query:${worldId}:${hash}`
}

// Same query always produces same key
const q1 = { "plug.Tag": "warrior", "plug.Health": { $gt: 50 } }
const q2 = { "plug.Health": { $gt: 50 }, "plug.Tag": "warrior" }

// Both produce same cache key (sorted keys)
getQueryCacheKey("world-123", q1) === getQueryCacheKey("world-123", q2)
```

## Monitoring and Metrics

### Cache Performance Metrics

```javascript
class CacheMetrics {
  constructor() {
    this.hits = 0
    this.misses = 0
    this.writes = 0
    this.invalidations = 0
  }

  recordHit() {
    this.hits++
  }

  recordMiss() {
    this.misses++
  }

  getHitRate() {
    const total = this.hits + this.misses
    return total > 0 ? (this.hits / total) * 100 : 0
  }

  getStats() {
    return {
      hits: this.hits,
      misses: this.misses,
      hitRate: this.getHitRate().toFixed(2) + '%',
      writes: this.writes,
      invalidations: this.invalidations
    }
  }
}

// Usage
async function getEntity(entityId) {
  const cacheKey = `plug:*:*:${entityId}`
  const cached = await redis.get(cacheKey)

  if (cached) {
    metrics.recordHit()
    return JSON.parse(cached)
  }

  metrics.recordMiss()
  const entity = await db.entities.findOne({ "_id": entityId })

  // Cache for next time
  await redis.setex(cacheKey, 1800, JSON.stringify(entity))
  metrics.writes++

  return entity
}

// Log metrics every minute
setInterval(() => {
  console.log("Cache Stats:", metrics.getStats())
}, 60000)
```

## Best Practices

### Cache Appropriately

```javascript
// Good - cache static/slow-changing data
plug.Name("Kael")           // Cache: 1 hour
plug.Description("...")     // Cache: 1 hour
plug.Tag(warrior_tag)       // Cache: 30 minutes

// Good - short TTL for dynamic data
plug.Health(85)             // Cache: 5 minutes
plug.Position({x: 10, y: 20}) // Cache: 1 minute

// Avoid caching - real-time data
plug.LastSeen(Date.now())   // Don't cache (always changing)
plug.OnlineStatus(true)     // Don't cache (needs real-time)
```

### Invalidate Precisely

```javascript
// Good - precise invalidation
updateEntityPlug("kael-uuid", "Health", 80)
// Invalidates: plug:*:*:kael-uuid

// Poor - over-invalidation
redis.flushall()  // Nukes entire cache!
```

### Monitor Cache Health

```javascript
// Track cache hit rate
const targetHitRate = 90  // 90%

setInterval(async () => {
  const hitRate = metrics.getHitRate()

  if (hitRate < targetHitRate) {
    console.warn(`Cache hit rate ${hitRate}% below target ${targetHitRate}%`)

    // Investigate:
    // - Are TTLs too short?
    // - Are invalidations too aggressive?
    // - Is cache size sufficient?
  }
}, 300000)  // Check every 5 minutes
```

## Attribution

The plug-aware caching strategies documented here were pioneered by **Lost Mountain Games** (2025), achieving sub-25ms response times for pluggable entity systems through intelligent multi-layer caching.

**Innovation**: Caching entire entity plug collections with intelligent invalidation based on plug modifications, enabling high performance without sacrificing pluggable architecture flexibility.

---

**Copyright © 2025 Lost Mountain Games**
