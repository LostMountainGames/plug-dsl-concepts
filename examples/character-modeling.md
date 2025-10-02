# Character Modeling Examples

**Innovation Attribution**: The character modeling patterns documented here were pioneered by **Lost Mountain Games** (2025) to demonstrate unlimited composition in character design.

## Basic Player Character

### Simple Warrior

**Concept**: Core character attributes for combat-focused character.

```javascript
entity Kael_Ironwind {
  // Identity
  plug.Name("Kael Ironwind")
  plug.Description("A grizzled warrior from the Iron Wolves clan")

  // Classification
  plug.Tag([character_tag, warrior_tag, player_tag])

  // Core Attributes
  plug.Class("Warrior")
  plug.Level(12)
  plug.Experience(45000)

  // Combat Stats
  plug.Health(85)
  plug.MaxHealth(100)
  plug.Stamina(60)
  plug.MaxStamina(80)

  // Attributes
  plug.Might(16)        // Strength
  plug.Agility(12)      // Dexterity
  plug.Endurance(14)    // Constitution
  plug.Cunning(10)      // Intelligence
  plug.Will(11)         // Wisdom
  plug.Presence(13)     // Charisma

  // World State
  plug.Location(iron_forge)
  plug.Faction(iron_wolves_clan)
}
```

**MongoDB Document**:
```json
{
  "_id": "kael_ironwind_uuid",
  "plug": {
    "Name": "Kael Ironwind",
    "Description": "A grizzled warrior from the Iron Wolves clan",
    "Tag": ["character_tag_uuid", "warrior_tag_uuid", "player_tag_uuid"],
    "Class": "Warrior",
    "Level": 12,
    "Experience": 45000,
    "Health": 85,
    "MaxHealth": 100,
    "Might": 16,
    "Location": "iron_forge_uuid",
    "Faction": "iron_wolves_clan_uuid"
  }
}
```

## Intermediate Character with Equipment

### Warrior with Inventory

**Concept**: Character with equipment and inventory management.

```javascript
entity Elara_Moonwhisper {
  // Identity
  plug.Name("Elara Moonwhisper")
  plug.Description("A ranger skilled in archery and wilderness survival")
  plug.Tag([character_tag, ranger_tag, player_tag])

  // Progression
  plug.Class("Ranger")
  plug.Level(10)
  plug.Experience(32000)

  // Stats
  plug.Health(70)
  plug.MaxHealth(80)
  plug.Mana(50)
  plug.MaxMana(60)

  // Attributes
  plug.Might(12)
  plug.Agility(16)       // High agility for ranger
  plug.Endurance(11)
  plug.Cunning(14)       // Perception and tracking
  plug.Will(13)
  plug.Presence(10)

  // Equipment (entity references)
  plug.EquippedWeapon(longbow_of_the_hunt)
  plug.EquippedArmor(leather_armor_of_shadows)
  plug.EquippedAccessory([elven_cloak, ring_of_protection])

  // Inventory (container system)
  plug.Inventory([
    healing_potion,
    healing_potion,      // Multiple instances
    rope_50ft,
    torch,
    torch,
    rations_7days,
    gold_pieces_500
  ])

  // Skills
  plug.Skills([
    archery_mastery,
    stealth_movement,
    animal_handling,
    wilderness_survival
  ])

  // Location & Faction
  plug.Location(moonwood_forest)
  plug.Faction(moonwhisper_elves)
}
```

**Pattern**: Equipment as entity references enables:
- Click equipment to view item details
- Drag-drop inventory management
- Equipment stat bonuses calculated automatically
- Hover shows equipped item stats

## Advanced Cross-Category Character

### NPC Merchant Who is Also a Location

**Concept**: Character that also functions as a shop/trading post.

```javascript
entity Merchant_Gilda {
  // CHARACTER PLUGS
  plug.Name("Merchant Gilda's Trading Post")
  plug.Description("A traveling merchant who set up permanent shop")
  plug.Tag([character_tag, npc_tag, merchant_tag, location_tag])

  // Character Stats
  plug.Class("Merchant")
  plug.Level(8)
  plug.Health(50)
  plug.Personality("Shrewd, friendly, loves to haggle")
  plug.Dialogue([
    greeting_new_customer,
    pitch_rare_items,
    haggling_responses
  ])

  // Attributes (minimal combat, high social)
  plug.Might(8)
  plug.Cunning(15)      // Business savvy
  plug.Presence(16)     // Persuasion

  // LOCATION PLUGS (functioning as shop)
  plug.LocationType("Shop")
  plug.Contains([
    weapons_display_case,
    armor_rack,
    potion_shelf,
    rare_items_vault
  ])

  // Shop Inventory (for sale)
  plug.ShopInventory([
    healing_potion,
    mana_potion,
    iron_sword,
    leather_armor,
    magic_scroll_fireball
  ])

  // Shop Mechanics
  plug.BuyPriceMultiplier(0.5)   // Buys at 50% value
  plug.SellPriceMultiplier(1.5)  // Sells at 150% value
  plug.RestockInterval("daily")
  plug.Gold(5000)                // Shop currency

  // Location in World
  plug.Location(market_district)  // Shop is INSIDE market district
  plug.Connections([
    market_square,
    back_alley
  ])
}
```

**Innovation**: Same entity is BOTH a character (you can talk to Gilda) AND a location (you can enter her shop). No class restrictions enable this flexibility.

## Living Weapon Character

### Sentient Sword with Personality

**Concept**: Weapon that is also a character with goals and personality.

```javascript
entity Soulrend_Glaive {
  // ITEM PLUGS
  plug.Name("Soulrend Glaive")
  plug.Description("A legendary polearm that hungers for worthy souls")
  plug.Tag([item_tag, weapon_tag, legendary_tag, character_tag, sentient_tag])

  // Weapon Stats
  plug.ItemType("Polearm")
  plug.Damage("3d6+3")
  plug.CriticalRange("19-20")
  plug.WeaponReach(10)  // Feet
  plug.Weight(8)        // Pounds
  plug.Rarity("Legendary")
  plug.Value(50000)     // Gold pieces

  // Special Abilities
  plug.SpecialAbility([
    soul_drain,         // Heal on kill
    extended_reach,     // 10ft range
    armor_piercing
  ])

  // CHARACTER PLUGS (sentient weapon)
  plug.Personality("Arrogant, bloodthirsty, seeks worthy wielders")
  plug.Will(18)         // Strong willpower
  plug.Intelligence(14)
  plug.Alignment("Chaotic Neutral")

  // Communication
  plug.CanSpeak(true)
  plug.Languages(["Common", "Abyssal", "Ancient Warrior Tongue"])
  plug.Dialogue([
    greeting_new_wielder,
    demands_worthy_foe,
    mocks_weak_opponent,
    celebrates_victory
  ])

  // Goals & Motivations
  plug.Goal("Find wielder worthy of my power")
  plug.Desire("Claim souls of legendary warriors")
  plug.Memory("Recalls every soul claimed across 500 years")

  // Sentient Item Mechanics
  plug.BondWithWielder(kael_ironwind)
  plug.SoulsClaimedCount(47)
  plug.RequiresAttunement(true)
  plug.EgoScore(16)     // Will contest with wielder

  // Current State
  plug.Owner(kael_ironwind)
  plug.Location(kael_ironwind)  // Equipped by owner
  plug.EquippedSlot("main_hand")
}
```

**Pattern**: Weapon you can have conversations with. Might refuse to fight weak opponents. Can provide advice or historical knowledge.

## NPC with Complex Relationships

### Faction Leader with Network

**Concept**: NPC with relationships, reputation, and political influence.

```javascript
entity War_Chief_Ironwind {
  // Identity
  plug.Name("War Chief Ironwind")
  plug.Description("Legendary chieftain of the Iron Wolves clan")
  plug.Tag([character_tag, npc_tag, faction_leader_tag, quest_giver_tag])

  // Stats
  plug.Class("Chieftain")
  plug.Level(20)        // High-level NPC
  plug.Health(150)
  plug.MaxHealth(150)

  // Combat Prowess
  plug.Might(18)
  plug.Endurance(16)
  plug.Will(15)
  plug.Presence(17)     // Leadership

  // Leadership & Influence
  plug.LeadsF action(iron_wolves_clan)
  plug.Influences([
    northern_trade_consortium,
    mountain_guard,
    elder_council
  ])

  // Relationships (with reputation values)
  plug.Relationship(kael_ironwind, "Mentor", 85)
  plug.Relationship(elara_moonwhisper, "Ally", 70)
  plug.Relationship(rival_chieftain, "Enemy", -50)
  plug.Relationship(king_valorian, "Respected", 60)

  // Reputation by Faction
  plug.Reputation(iron_wolves_clan, "Revered", 95)
  plug.Reputation(moonwhisper_elves, "Trusted", 70)
  plug.Reputation(orcish_horde, "Feared", -80)

  // Quest Interactions
  plug.QuestsAvailable([
    main_quest_unite_clans,
    side_quest_recover_heirloom,
    faction_quest_defend_territory
  ])
  plug.QuestsCompleted([
    historical_quest_founding_iron_wolves
  ])

  // Dialogue & Personality
  plug.Personality("Wise, fierce, values honor above all")
  plug.Dialogue([
    greeting_clan_member,
    greeting_outsider,
    quest_briefing_unite_clans,
    wisdom_about_history
  ])

  // Equipment (legendary gear)
  plug.EquippedWeapon(ironwind_warhammer)
  plug.EquippedArmor(chieftains_battle_plate)

  // Location
  plug.Location(great_hall_iron_wolves)
}
```

**Pattern**: Rich NPC with web of relationships enables:
- Dynamic quest availability based on reputation
- Faction interactions affect multiple NPCs
- Political intrigue and alliance systems
- Context-aware dialogue

## Companion Character

### Party Member with AI Behavior

**Concept**: Companion NPC that joins player's party.

```javascript
entity Thane_the_Smith {
  // Identity
  plug.Name("Thane the Smith")
  plug.Description("Master blacksmith and reluctant adventurer")
  plug.Tag([character_tag, npc_tag, companion_tag, craftsman_tag])

  // Progression (grows with player)
  plug.Class("Artificer")
  plug.Level(9)
  plug.Experience(28000)

  // Stats
  plug.Health(65)
  plug.MaxHealth(75)
  plug.Mana(40)
  plug.MaxMana(50)

  // Attributes (specialized)
  plug.Might(14)        // Strong from smithing
  plug.Cunning(16)      // Crafting intelligence
  plug.Endurance(13)
  plug.Agility(10)      // Not very agile

  // Combat Role
  plug.CombatRole("Tank")
  plug.EquippedWeapon(master_crafted_warhammer)
  plug.EquippedArmor(forge_masters_plate)

  // Companion Behavior
  plug.PartyMember(player_party)
  plug.CompanionType("Craftsman")
  plug.Loyalty(75)      // 0-100 scale
  plug.AIBehavior([
    defend_player,
    craft_repairs_after_battle,
    suggest_equipment_upgrades
  ])

  // Crafting Abilities
  plug.CraftingSkill("Master")
  plug.CanCraft([
    weapons_all_types,
    armor_heavy,
    enchanted_items_basic
  ])
  plug.KnownRecipes([
    recipe_iron_sword,
    recipe_steel_plate_armor,
    recipe_enchanted_hammer
  ])

  // Relationship with Player
  plug.Relationship(player_character, "Trusted Friend", 75)
  plug.CompanionQuests([
    quest_recover_smithing_tools,
    quest_legendary_forge
  ])

  // Personality & Dialogue
  plug.Personality("Gruff exterior, heart of gold, proud of craft")
  plug.Dialogue([
    combat_barks,
    crafting_advice,
    companionship_growth,
    personal_story_reveals
  ])

  // Companion Perks (benefits to player)
  plug.PartyBuff("Equipment Repair", "Can repair damaged gear")
  plug.PartyBuff("Crafting Discount", "Reduces crafting cost 25%")

  // Current State
  plug.Location(player_character)  // Following player
  plug.CurrentMood("Content")
}
```

**Pattern**: Companion who provides both combat support and crafting services. Loyalty system affects behavior and quest availability.

## Attribution

The character modeling patterns documented here were pioneered by **Lost Mountain Games** (2025), demonstrating unlimited composition where characters can also be locations, items can be characters, and entities freely combine capabilities through plugs.

**Innovation**: Cross-category characters (merchant-locations, sentient weapons), relationship networks, companion AI, and unlimited plug composition for flexible character design.

---

**Copyright © 2025 Lost Mountain Games**
