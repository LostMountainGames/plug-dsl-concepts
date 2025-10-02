# Item Systems Examples

**Innovation Attribution**: The item system patterns documented here were pioneered by **Lost Mountain Games** (2025) to demonstrate living items, crafting systems, and unlimited item composition.

## Basic Items

### Simple Weapon

**Concept**: Standard weapon with combat stats.

```javascript
entity Iron_Sword {
  // Identity
  plug.Name("Iron Sword")
  plug.Description("A well-crafted iron blade, standard issue for soldiers")
  plug.Tag([item_tag, weapon_tag, common_tag])

  // Item Properties
  plug.ItemType("Weapon")
  plug.WeaponType("Sword")
  plug.Rarity("Common")
  plug.Value(50)        // Gold pieces
  plug.Weight(3)        // Pounds

  // Combat Stats
  plug.Damage("1d8+1")
  plug.CriticalRange("20")
  plug.CriticalMultiplier("x2")
  plug.DamageType("Slashing")

  // Requirements
  plug.RequiresStrength(10)
  plug.RequiresProficiency("Sword")

  // Current State
  plug.Durability(100)
  plug.MaxDurability(100)
  plug.Owner(null)      // Not owned
  plug.Location(merchants_shop)
}
```

## Magical Items

### Enchanted Weapon

**Concept**: Weapon with magical properties and enchantments.

```javascript
entity Crystal_Sword_of_Flame {
  // Identity
  plug.Name("Crystal Sword of Flame")
  plug.Description("A crystalline blade that burns with eternal fire")
  plug.Tag([item_tag, weapon_tag, magical_tag, rare_tag])

  // Item Properties
  plug.ItemType("Weapon")
  plug.WeaponType("Sword")
  plug.Rarity("Rare")
  plug.Value(5000)
  plug.Weight(3.5)

  // Combat Stats (enhanced)
  plug.Damage("2d6+2")
  plug.BonusDamage("1d6", "Fire")
  plug.CriticalRange("19-20")
  plug.AttackBonus("+2")

  // Enchantments
  plug.Enchantment([
    flame_blade,
    light_source
  ])

  // Special Abilities
  plug.SpecialAbility([
    ignite_weapon,      // Set blade on fire (extra damage)
    light_spell,        // Illuminates area
    fire_resistance     // Grants wielder resistance
  ])

  // Magical Properties
  plug.RequiresAttunement(true)
  plug.AttunementSlots(1)
  plug.MagicBonus("+2")

  // Visual Effects
  plug.VisualEffect("Blade glows with orange flame")
  plug.SoundEffect("Crackling fire")

  // Current State
  plug.Durability(200)
  plug.MaxDurability(200)
  plug.IsAttuned(false)
  plug.Owner(null)
  plug.Location(ancient_armory)
}
```

## Living Item (Sentient Artifact)

### Talking Sword with Personality

**Concept**: Weapon that is also a character with goals.

```javascript
entity Frostbite_Soulreaver {
  // ITEM PLUGS
  plug.Name("Frostbite, the Soulreaver")
  plug.Description("A legendary blade forged in the heart of an ice dragon")
  plug.Tag([item_tag, weapon_tag, legendary_tag, character_tag, sentient_tag, artifact_tag])

  // Weapon Stats
  plug.ItemType("Weapon")
  plug.WeaponType("Greatsword")
  plug.Rarity("Legendary")
  plug.Value(100000)
  plug.Weight(6)

  // Combat Stats (extremely powerful)
  plug.Damage("3d8+5")
  plug.BonusDamage("2d6", "Cold")
  plug.CriticalRange("17-20")
  plug.AttackBonus("+3")
  plug.MagicBonus("+5")

  // Legendary Abilities
  plug.SpecialAbility([
    freeze_strike,           // Freeze enemies solid
    ice_storm_summon,        // Cast ice storm 1/day
    cold_immunity,           // Wielder immune to cold
    soul_reaving,            // Trap souls of slain
    dragon_aspect            // Transform wielder
  ])

  // CHARACTER PLUGS (sentient item)
  plug.Personality("Cold, calculating, values strength and ambition")
  plug.Will(22)              // Extremely powerful will
  plug.Intelligence(18)
  plug.Wisdom(16)
  plug.Charisma(14)
  plug.Alignment("Lawful Evil")

  // Communication
  plug.CanSpeak(true)
  plug.CanTelepathy(true)
  plug.Languages(["Common", "Draconic", "Primordial"])
  plug.Dialogue([
    greeting_worthy_wielder,
    scorn_weak_wielder,
    tactical_advice,
    ancient_dragon_lore,
    soul_collection_demands
  ])

  // Goals & Motivations
  plug.Goal("Collect souls of legendary warriors")
  plug.Desire("Find wielder worthy of dragon power")
  plug.Fear("Being sealed away again")
  plug.Memory("Created from heart of ancient ice dragon")
  plug.Age(5000)

  // Sentient Item Mechanics
  plug.EgoScore(20)          // Will contest with wielder
  plug.DominationCheck("DC 22")  // Resist being controlled
  plug.BondRequirement("Must defeat in single combat")
  plug.Loyalty(0)            // Starts at 0, builds over time

  // Relationship with Wielder
  plug.CurrentWielder(null)
  plug.FormerWielders([
    dragon_king_ancient,
    lich_lord_frozen,
    hero_of_ice_age
  ])
  plug.SoulsCollected(347)
  plug.RequiresAttunement(true)
  plug.AttunementCondition("Prove your strength in combat")

  // Curse Properties
  plug.Curse("Cannot be discarded once bonded")
  plug.CurseEffect("Wielder slowly becomes cold and emotionless")
  plug.CurseLift("Complete sword's quest for 1000 souls")

  // Visual & Audio
  plug.VisualEffect("Blade constantly shimmers with frost, mist emanates")
  plug.SoundEffect("Whispers of trapped souls, crackling ice")
  plug.Aura("Freezing cold surrounds wielder")

  // Current State
  plug.Durability(999)
  plug.MaxDurability(999)   // Nearly indestructible
  plug.Location(frozen_tomb)
  plug.IsSealed(true)        // Requires quest to unseal
}
```

**Pattern**: Weapon has its own agenda. Might refuse to help, demand worthy targets, or take control of wielder in dire situations. Rich dialogue and relationship system.

## Crafting and Materials

### Crafting Recipe

**Concept**: Recipe defines how to create items from materials.

```javascript
entity Recipe_Enchanted_Longsword {
  // Identity
  plug.Name("Recipe: Enchanted Longsword")
  plug.Description("Instructions for forging a magically enhanced longsword")
  plug.Tag([item_tag, recipe_tag, crafting_tag])

  // Recipe Properties
  plug.ItemType("Recipe")
  plug.RecipeType("Blacksmithing")
  plug.Difficulty("Expert")
  plug.RequiresSkill("Blacksmithing", 15)

  // Materials Required
  plug.Materials([
    {item: mithril_ingot, quantity: 5},
    {item: dragon_scale, quantity: 2},
    {item: magic_crystal, quantity: 1},
    {item: leather_grip, quantity: 1}
  ])

  // Crafting Requirements
  plug.RequiresTool(master_forge)
  plug.RequiresLocation(enchanting_station)
  plug.CraftingTime(8)   // Hours
  plug.SuccessChance(75) // Percent

  // Output
  plug.Creates(enchanted_longsword)
  plug.OutputQuantity(1)

  // Learning
  plug.LearnedBy([
    master_smith_thane,
    elven_weaponsmith
  ])
  plug.TeachableFrom(ancient_smithing_tome)
}
```

### Crafting Materials

**Concept**: Materials used in crafting recipes.

```javascript
entity Mithril_Ingot {
  plug.Name("Mithril Ingot")
  plug.Description("A bar of refined mithril, light and incredibly strong")
  plug.Tag([item_tag, material_tag, rare_tag])

  plug.ItemType("Material")
  plug.MaterialType("Metal")
  plug.Rarity("Rare")
  plug.Value(500)
  plug.Weight(0.5)   // Very light for metal

  // Crafting Uses
  plug.UsedInRecipes([
    recipe_mithril_sword,
    recipe_mithril_armor,
    recipe_enchanted_longsword,
    recipe_elven_chain
  ])

  // Properties
  plug.MaterialProperties([
    "Lightweight",
    "Extremely Durable",
    "Magic Conductive",
    "Corrosion Resistant"
  ])

  // Sources
  plug.ObtainedFrom([
    mithril_mine_deep_mountains,
    ancient_dwarven_ruins,
    merchant_rare_goods
  ])
}
```

## Container Systems

### Bag of Holding

**Concept**: Container item with inventory management.

```javascript
entity Bag_of_Holding {
  plug.Name("Bag of Holding")
  plug.Description("A magical bag that holds far more than it appears")
  plug.Tag([item_tag, container_tag, magical_tag])

  plug.ItemType("Container")
  plug.ContainerType("Magical Bag")
  plug.Rarity("Uncommon")
  plug.Value(1000)
  plug.Weight(15)    // Bag itself

  // Container Properties
  plug.Capacity(500)   // Pounds of items
  plug.VolumeCapacity("64 cubic feet")
  plug.IgnoreItemWeight(true)  // Items inside don't add weight

  // Magical Properties
  plug.MagicType("Extradimensional Space")
  plug.RequiresAttunement(false)

  // Contents
  plug.Contains([
    healing_potion,
    healing_potion,
    mana_potion,
    rope_50ft,
    rations_7days,
    torch,
    torch,
    torch,
    lockpicks,
    gold_pieces_1500
  ])

  // Container Mechanics
  plug.RetrievalAction("Bonus Action")
  plug.AccessRestriction("Owner only")
  plug.DestructionRisk("If torn, contents lost in void")

  // Current State
  plug.Owner(player_character)
  plug.Location(player_character)  // In inventory
  plug.CurrentWeight(150)   // Contents weight (ignored)
}
```

## Living Container

### Mimic That Is Also a Backpack

**Concept**: Container that is also a monster/companion.

```javascript
entity Faithful_Mimic_Backpack {
  // ITEM PLUGS (container)
  plug.Name("Faithful Mimic Backpack")
  plug.Description("A domesticated mimic that serves as a loyal backpack")
  plug.Tag([item_tag, container_tag, character_tag, creature_tag, companion_tag])

  // Container Properties
  plug.ItemType("Container")
  plug.ContainerType("Living Backpack")
  plug.Rarity("Very Rare")
  plug.Value(10000)
  plug.Weight(20)

  plug.Capacity(100)   // Pounds
  plug.Contains([
    // Player items stored inside
  ])

  // CHARACTER PLUGS (living creature)
  plug.Personality("Loyal, protective, likes being fed treats")
  plug.CreatureType("Aberration")
  plug.Size("Small")

  // Stats
  plug.Health(30)
  plug.MaxHealth(30)
  plug.ArmorClass(14)

  // Abilities
  plug.Might(12)
  plug.Agility(10)
  plug.Endurance(14)
  plug.Intelligence(6)    // Not very smart
  plug.Will(8)
  plug.Presence(10)

  // Companion Abilities
  plug.CanAttack(true)
  plug.AttackType("Bite")
  plug.Damage("1d6+1")
  plug.DefendOwner(true)

  // Living Container Features
  plug.CanSortItems(true)          // Organizes contents
  plug.CanEatJunk(true)            // Disposes unwanted items
  plug.WarnsDanger(true)           // Senses danger, warns owner
  plug.RequiresFeeding(true)       // Needs food daily
  plug.FavoriteFood("Gold coins")  // Eats 1 gold per day

  // Bonding
  plug.RequiresAttunement(true)
  plug.BondedWith(player_character)
  plug.Loyalty(80)

  // Special Abilities
  plug.SpecialAbility([
    treasure_sense,       // Detects valuable items
    guardian_bite,        // Bites thieves
    item_preservation     // Keeps food fresh
  ])

  // Dialogue (limited)
  plug.CanCommunicate("Through emotions and sounds")
  plug.Dialogue([
    happy_fed_sounds,
    hungry_grumbles,
    danger_warning_hiss,
    content_purr
  ])

  // Current State
  plug.Hunger(50)       // 0-100 scale
  plug.Mood("Content")
  plug.Owner(player_character)
  plug.Location(player_character)  // Worn as backpack
}
```

**Innovation**: Backpack that helps in combat, organizes items, and has emotional bonds with owner. Requires feeding but provides unique abilities.

## Consumable Items

### Healing Potion with Stack System

**Concept**: Consumable item that stacks.

```javascript
entity Healing_Potion {
  plug.Name("Healing Potion")
  plug.Description("A red liquid that restores health when consumed")
  plug.Tag([item_tag, consumable_tag, potion_tag, common_tag])

  plug.ItemType("Consumable")
  plug.ConsumableType("Potion")
  plug.Rarity("Common")
  plug.Value(50)
  plug.Weight(0.5)

  // Consumable Properties
  plug.Effect("Restore 2d4+2 health")
  plug.UsageAction("Bonus Action")
  plug.SingleUse(true)

  // Stacking
  plug.Stackable(true)
  plug.MaxStackSize(99)

  // Crafting
  plug.CanBeCrafted(true)
  plug.Recipe(recipe_healing_potion)

  // Current State
  plug.StackCount(1)    // Single potion
  plug.Owner(player_character)
  plug.Location(player_character)
}
```

## Legendary Artifact

### World-Changing Artifact

**Concept**: Extremely powerful item with world-affecting abilities.

```javascript
entity The_Heart_of_Eternity {
  plug.Name("The Heart of Eternity")
  plug.Description("A crystalline heart that pulses with the power of time itself")
  plug.Tag([item_tag, artifact_tag, legendary_tag, plot_item_tag, character_tag])

  // Artifact Properties
  plug.ItemType("Artifact")
  plug.ArtifactType("Cosmic Relic")
  plug.Rarity("Artifact")  // Beyond legendary
  plug.Value("Priceless")
  plug.Weight(5)

  // Power Level
  plug.PowerLevel("World-Altering")
  plug.ThreatLevel("Extreme")

  // Abilities (extremely powerful)
  plug.SpecialAbility([
    stop_time,              // Freeze time for 1 minute
    rewind_time,            // Undo last hour
    time_travel,            // Travel to past/future
    age_manipulation,       // Age/de-age targets
    see_all_timelines,      // View alternate realities
    temporal_anchor         // Fix point in time
  ])

  // CHARACTER ASPECTS (sentient artifact)
  plug.Personality("Ancient, omniscient, speaks in temporal riddles")
  plug.Intelligence(25)   // Beyond mortal comprehension
  plug.Will(25)
  plug.Alignment("True Neutral")

  plug.CanSpeak(true)
  plug.Languages(["All Languages, Simultaneously, Across All Time"])
  plug.Dialogue([
    greetings_from_all_times,
    warnings_of_paradox,
    knowledge_of_history,
    visions_of_future
  ])

  // Goals
  plug.Goal("Preserve the timeline's integrity")
  plug.Desire("Find worthy guardian")
  plug.Fear("Temporal paradox destroying reality")

  // Artifact Mechanics
  plug.RequiresAttunement(true)
  plug.AttunementCondition("Survive witnessing all possible futures")
  plug.EgoScore(25)       // Will dominate most wielders
  plug.Loyalty(null)      // Serves timeline, not wielder

  // Consequences of Use
  plug.SideEffect("Each use risks temporal paradox")
  plug.Corruption("Wielder slowly becomes unstuck in time")
  plug.Madness("Seeing all timelines drives wielder insane")

  // Plot Integration
  plug.IsQuestItem(true)
  plug.RequiredForQuest(quest_prevent_apocalypse)
  plug.CannotBeDestroyed(true)
  plug.CannotBeDiscarded(true)

  // Visual Effects
  plug.VisualEffect("Constantly shifts between past and future states")
  plug.SoundEffect("Echoes of all moments in time")
  plug.Aura("Time distorts around wielder")

  // Current State
  plug.Location(vault_of_eternity)
  plug.IsSealed(true)
  plug.SealRequirement("Unlock with keys from three timelines")
}
```

**Pattern**: World-changing artifact with extreme power and consequences. Tied to main plot. Using it risks breaking reality.

## Attribution

The item system patterns documented here were pioneered by **Lost Mountain Games** (2025), demonstrating living items with personalities, complex crafting systems, sentient containers, and world-altering artifacts through unlimited plug composition.

**Innovation**: Items as characters (talking swords, living backpacks), crafting recipe systems, container hierarchies, and cosmic artifacts using unified plug architecture.

---

**Copyright © 2025 Lost Mountain Games**
