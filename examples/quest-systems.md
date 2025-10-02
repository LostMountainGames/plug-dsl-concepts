# Quest System Examples

**Innovation Attribution**: The quest system patterns documented here were pioneered by **Lost Mountain Games** (2025) to demonstrate dynamic narrative structures and branching quest design.

## Basic Quest

### Simple Fetch Quest

**Concept**: Straightforward quest with single objective.

```javascript
entity Quest_Retrieve_Heirloom {
  // Identity
  plug.Name("Retrieve the Ironwind Heirloom")
  plug.Description("War Chief Ironwind's family sword was stolen by bandits")
  plug.Tag([quest_tag, side_quest_tag, retrieval_tag])

  // Quest Properties
  plug.QuestType("Retrieval")
  plug.Difficulty("Easy")
  plug.RecommendedLevel(5)
  plug.EstimatedTime("30 minutes")

  // Quest Giver
  plug.QuestGiver(war_chief_ironwind)
  plug.QuestLocation(great_hall_iron_wolves)

  // Objectives
  plug.Objectives([
    {
      id: "find_bandits",
      description: "Locate the bandit camp",
      type: "location",
      target: bandit_camp,
      complete: false
    },
    {
      id: "defeat_bandits",
      description: "Defeat the bandit leader",
      type: "combat",
      target: bandit_leader,
      count: 1,
      complete: false
    },
    {
      id: "retrieve_sword",
      description: "Retrieve the Ironwind Heirloom",
      type: "item",
      target: ironwind_family_sword,
      complete: false
    },
    {
      id: "return_sword",
      description: "Return the sword to War Chief Ironwind",
      type: "delivery",
      target: war_chief_ironwind,
      complete: false
    }
  ])

  // Rewards
  plug.RewardGold(500)
  plug.RewardExperience(1000)
  plug.RewardItems([
    iron_wolves_emblem,
    healing_potion
  ])
  plug.RewardReputation(iron_wolves_clan, 25)

  // Quest State
  plug.QuestStatus("Available")
  plug.CurrentObjective("find_bandits")
  plug.AcceptedBy(null)
  plug.CompletedBy([])

  // Requirements
  plug.RequiresLevel(5)
  plug.RequiresQuest(null)  // No prerequisite
  plug.RequiresReputation(iron_wolves_clan, "Friendly", 25)

  // Failure Conditions
  plug.FailConditions([
    "War Chief Ironwind dies",
    "Bandit leader escapes with sword"
  ])
}
```

## Branching Quest

### Quest with Multiple Paths

**Concept**: Quest with meaningful choices that affect outcome.

```javascript
entity Quest_The_Hostage_Dilemma {
  plug.Name("The Hostage Dilemma")
  plug.Description("Bandits have taken hostages. Choose how to resolve the crisis.")
  plug.Tag([quest_tag, main_quest_tag, branching_tag, moral_choice_tag])

  plug.QuestType("Branching Narrative")
  plug.Difficulty("Medium")
  plug.RecommendedLevel(8)

  plug.QuestGiver(guard_captain_stone)

  // Main Objective
  plug.MainObjective("Resolve the hostage situation")

  // Branching Paths
  plug.QuestBranches([
    {
      id: "diplomatic_path",
      name: "Negotiate for Release",
      description: "Attempt to negotiate with the bandits",
      requirements: {
        skill: "Persuasion",
        level: 10
      },
      objectives: [
        "Gather ransom money (1000 gold)",
        "Negotiate with bandit leader",
        "Secure hostage release"
      ],
      outcome: "peaceful_resolution",
      consequences: [
        "Hostages safe",
        "Bandits escape",
        "Gold lost"
      ]
    },
    {
      id: "stealth_path",
      name: "Stealth Rescue",
      description: "Sneak into camp and free hostages",
      requirements: {
        skill: "Stealth",
        level: 12
      },
      objectives: [
        "Scout bandit camp",
        "Disable guards silently",
        "Free hostages undetected"
      ],
      outcome: "stealth_success",
      consequences: [
        "Hostages safe",
        "Some bandits apprehended",
        "No casualties"
      ]
    },
    {
      id: "combat_path",
      name: "Direct Assault",
      description: "Attack the bandit camp head-on",
      requirements: {
        minLevel: 8
      },
      objectives: [
        "Assault bandit camp",
        "Defeat all bandits",
        "Free hostages"
      ],
      outcome: "violent_resolution",
      consequences: [
        "Some hostages may die",
        "All bandits killed",
        "Heavy combat"
      ]
    }
  ])

  // Current State
  plug.QuestStatus("Available")
  plug.ChosenPath(null)
  plug.BranchLocked(false)

  // Path-Specific Rewards
  plug.RewardsByPath({
    diplomatic_path: {
      gold: 200,         // Less reward (spent 1000 on ransom)
      experience: 2000,
      reputation: {faction: town_guard, amount: 50},
      title: "Peacemaker"
    },
    stealth_path: {
      gold: 800,
      experience: 2500,  // Highest XP
      items: [shadow_cloak],
      reputation: {faction: thieves_guild, amount: 25}
    },
    combat_path: {
      gold: 600,
      experience: 2000,
      items: [bandit_leaders_axe],
      reputation: {faction: iron_wolves_clan, amount: 40}
    }
  })

  // Consequences Affect World State
  plug.WorldStateChanges({
    diplomatic_path: [
      "Bandits remain active in region",
      "Hostages grateful but town poorer",
      "Guard captain questions your choice"
    ],
    stealth_path: [
      "Bandits scattered, some captured",
      "Hostages safe, town intact",
      "Thieves guild respects your skills"
    ],
    combat_path: [
      "Bandits eliminated",
      "Some hostage casualties",
      "Town sees you as warrior, not diplomat"
    ]
  })
}
```

**Pattern**: Choice of path locks out other options. Each path has different requirements, rewards, and world consequences.

## Dynamic Quest

### Quest That Adapts to Player Actions

**Concept**: Quest state changes based on player choices and world events.

```javascript
entity Quest_The_Rising_Darkness {
  plug.Name("The Rising Darkness")
  plug.Description("Strange shadows spread across the land. Investigate the source.")
  plug.Tag([quest_tag, main_quest_tag, dynamic_tag, world_threat_tag])

  plug.QuestType("Dynamic Investigation")
  plug.Difficulty("Hard")
  plug.RecommendedLevel(12)

  // Dynamic Quest Properties
  plug.AdaptsToPlayerActions(true)
  plug.WorldStateDependent(true)
  plug.TimeeSensitive(true)

  // Investigation Phases
  plug.QuestPhases([
    {
      phase: 1,
      name: "Initial Reports",
      trigger: "Quest accepted",
      objectives: [
        "Investigate shadow sightings in 3 locations",
        "Interview witnesses",
        "Collect evidence"
      ],
      timeLimit: null
    },
    {
      phase: 2,
      name: "Growing Threat",
      trigger: "Complete Phase 1 OR 7 days pass",
      objectives: [
        "Defend villages from shadow attacks",
        "Trace shadow source",
        "Research ancient texts"
      ],
      timeLimit: "10 days",
      escalation: "If time runs out, shadows spread to major cities"
    },
    {
      phase: 3,
      name: "The Source Revealed",
      trigger: "Complete Phase 2 OR shadows reach capital",
      objectives: [
        "Confront the shadow source",
        "Choose: Seal the portal OR Destroy the source",
        "Deal with consequences"
      ],
      finalPhase: true
    }
  ])

  // Dynamic Objectives (change based on player actions)
  plug.DynamicObjectives([
    {
      condition: "Player allies with Shadow Empire",
      addsObjective: "Infiltrate rebel forces",
      removesObjective: "Defend villages"
    },
    {
      condition: "Player discovers shadow magic is natural",
      addsObjective: "Find balance between light and shadow",
      removesObjective: "Destroy the source"
    },
    {
      condition: "Elara joins party",
      addsObjective: "Use elven knowledge of shadows",
      bonusReward: "elven_shadow_ward"
    }
  ])

  // World State Tracking
  plug.WorldState({
    shadowSpreadLevel: 0,        // 0-100
    villagesDestroyed: 0,
    alliesGained: [],
    knowledgeDiscovered: []
  })

  // Consequences Based on Performance
  plug.OutcomesByPerformance({
    excellent: {
      condition: "Complete in <14 days, all villages safe",
      outcome: "Shadows banished, peace restored",
      rewards: {
        gold: 5000,
        experience: 10000,
        items: [legendary_light_blade],
        reputation: "Hero status in all factions"
      }
    },
    good: {
      condition: "Complete in <21 days, most villages safe",
      outcome: "Shadows contained, some damage",
      rewards: {
        gold: 3000,
        experience: 7500,
        items: [rare_shadow_ward],
        reputation: "Respected hero"
      }
    },
    poor: {
      condition: "Complete after 21 days OR many villages lost",
      outcome: "Shadows beaten back but world scarred",
      rewards: {
        gold: 1000,
        experience: 5000,
        reputation: "Victory at great cost"
      }
    },
    failure: {
      condition: "Shadows reach capital and overwhelm defenses",
      outcome: "Shadow apocalypse",
      consequences: "Game over OR post-apocalyptic world state"
    }
  })

  // Current State
  plug.QuestStatus("Available")
  plug.CurrentPhase(1)
  plug.DaysElapsed(0)
  plug.PlayerChoices([])
}
```

**Pattern**: Quest evolves over time. Player speed and choices affect outcome. Multiple endings based on performance.

## Quest Chain

### Multi-Quest Storyline

**Concept**: Series of connected quests forming narrative arc.

```javascript
entity Quest_Chain_Unite_The_Clans {
  plug.Name("Unite the Clans")
  plug.Description("A multi-part epic to unite the warring northern clans")
  plug.Tag([quest_tag, quest_chain_tag, main_storyline_tag, epic_tag])

  plug.QuestType("Quest Chain")
  plug.TotalQuests(7)
  plug.EstimatedTime("15 hours")

  // Quest Sequence
  plug.Quests([
    {
      order: 1,
      quest: quest_prove_your_worth,
      description: "Prove yourself to War Chief Ironwind",
      status: "Available"
    },
    {
      order: 2,
      quest: quest_gain_iron_wolves_trust,
      description: "Complete trials of the Iron Wolves",
      status: "Locked",
      unlockCondition: "Complete quest_prove_your_worth"
    },
    {
      order: 3,
      quest: quest_meet_rival_chieftain,
      description: "Meet with rival Storm Clan chieftain",
      status: "Locked",
      unlockCondition: "Complete quest_gain_iron_wolves_trust"
    },
    {
      order: 4,
      quest: quest_ancient_grudge,
      description: "Uncover the ancient grudge between clans",
      status: "Locked",
      unlockCondition: "Complete quest_meet_rival_chieftain",
      branches: ["forgive_grudge", "avenge_grudge"]  // Branching!
    },
    {
      order: 5,
      quest: quest_gather_the_clans,
      description: "Convince all clan leaders to meet",
      status: "Locked",
      unlockCondition: "Complete quest_ancient_grudge"
    },
    {
      order: 6,
      quest: quest_the_great_moot,
      description: "Survive the Great Moot and broker peace",
      status: "Locked",
      unlockCondition: "Complete quest_gather_the_clans"
    },
    {
      order: 7,
      quest: quest_defend_united_clans,
      description: "Defend the united clans from external threat",
      status: "Locked",
      unlockCondition: "Complete quest_the_great_moot",
      finalQuest: true
    }
  ])

  // Chain Progress
  plug.CurrentQuest(quest_prove_your_worth)
  plug.CompletedQuests([])
  plug.FailedQuests([])

  // Chain-Wide Rewards
  plug.ChainRewards({
    onCompletion: {
      gold: 10000,
      experience: 50000,
      items: [crown_of_united_clans],
      reputation: "Legendary status among northern clans",
      title: "Uniter of Clans",
      worldChange: "Northern clans united under single banner"
    },
    perQuest: {
      gold: 500,
      experience: 5000
    }
  })

  // Failure Consequences
  plug.ChainFailCondition("Any critical quest failed")
  plug.FailureOutcome("Clans remain divided, vulnerable to invasion")
}
```

## Procedurally Generated Quest

### Dynamic Quest Template

**Concept**: Quest template that generates unique instances.

```javascript
entity Quest_Template_Bounty_Hunt {
  plug.Name("Bounty Hunt Template")
  plug.Description("Template for generating bounty hunting quests")
  plug.Tag([quest_tag, template_tag, procedural_tag])

  plug.QuestType("Procedural Bounty Hunt")

  // Template Variables (filled at generation)
  plug.Variables({
    target_name: null,         // Generated NPC name
    target_type: null,         // "Bandit", "Monster", "Renegade", etc.
    target_location: null,     // Random location
    bounty_amount: null,       // 100-1000 gold
    difficulty: null,          // "Easy", "Medium", "Hard"
    quest_giver: null         // Random from available NPCs
  })

  // Generation Rules
  plug.GenerationRules({
    target_type: {
      weights: {
        "Bandit": 40,
        "Monster": 30,
        "Renegade": 20,
        "Deserter": 10
      }
    },
    target_location: {
      mustBe: "wilderness_or_dungeon",
      distanceFromTown: "1-3 regions away"
    },
    bounty_amount: {
      formula: "difficulty_tier * (100 + random(1, 500))",
      minimum: 100,
      maximum: 5000
    },
    difficulty: {
      basedOn: "player_level",
      range: "player_level ± 2"
    }
  })

  // Generated Quest Structure
  plug.QuestStructure({
    objectives: [
      "Locate {target_name} at {target_location}",
      "Defeat or capture {target_name}",
      "Return to {quest_giver} for reward"
    ],
    optional_objectives: [
      "Bring {target_name} alive (2x bounty)",
      "Recover {target_name}'s stolen goods"
    ],
    rewards: {
      gold: "{bounty_amount}",
      experience: "{bounty_amount} * 2",
      reputation: "{quest_giver_faction}, 10"
    }
  })

  // Example Generated Instance
  plug.ExampleGeneration({
    target_name: "Razor Jack the Cruel",
    target_type: "Bandit",
    target_location: "abandoned_mine_northern_pass",
    bounty_amount: 750,
    difficulty: "Medium",
    quest_giver: "guard_captain_stone",
    fullDescription: "Guard Captain Stone offers 750 gold for the capture or elimination of Razor Jack the Cruel, a bandit terrorizing the northern trade routes. Last seen near the Abandoned Mine in Northern Pass."
  })
}

// Auto-Generated Instance
entity Quest_Bounty_Razor_Jack {
  plug.Name("Wanted: Razor Jack the Cruel")
  plug.Description("Guard Captain Stone offers 750 gold for dealing with a bandit")
  plug.Tag([quest_tag, bounty_hunt_tag, generated_quest_tag])

  plug.GeneratedFrom(quest_template_bounty_hunt)
  plug.GenerationSeed(1234567)  // For reproducibility

  plug.QuestGiver(guard_captain_stone)
  plug.Target(razor_jack_the_cruel)  // Dynamically created NPC
  plug.TargetLocation(abandoned_mine_northern_pass)

  plug.Objectives([
    "Track down Razor Jack in the Abandoned Mine",
    "Defeat or capture Razor Jack",
    "Return to Guard Captain Stone"
  ])

  plug.OptionalObjective("Bring Razor Jack alive for double bounty")

  plug.RewardGold(750)
  plug.BonusGold(750)  // If brought alive
  plug.RewardExperience(1500)

  plug.QuestStatus("Available")
  plug.ExpiresAfter(30)  // Days
}
```

**Pattern**: Template generates unlimited unique quests. Keeps world feeling alive with fresh content.

## Companion Quest

### Character-Specific Personal Quest

**Concept**: Quest tied to companion character's personal story.

```javascript
entity Quest_Thanes_Legacy {
  plug.Name("Thane's Legacy: Reforge the Masterwork")
  plug.Description("Help Thane recover his father's legendary forge and create a masterwork")
  plug.Tag([quest_tag, companion_quest_tag, crafting_quest_tag, personal_story_tag])

  plug.QuestType("Companion Quest")
  plug.Companion(thane_the_smith)
  plug.RequiresCompanion(true)  // Thane must be in party

  // Unlock Condition
  plug.UnlockCondition("Thane loyalty ≥ 50")
  plug.TriggeredBy("Camp conversation with Thane")

  // Personal Story Objectives
  plug.Objectives([
    {
      description: "Listen to Thane's story about his father",
      type: "dialogue",
      target: thane_the_smith,
      complete: false
    },
    {
      description: "Travel to the Abandoned Forge in Dragon Peak",
      type: "location",
      target: abandoned_forge_dragon_peak,
      complete: false
    },
    {
      description: "Clear the forge of monsters",
      type: "combat",
      enemies: [fire_elementals, corrupted_constructs],
      complete: false
    },
    {
      description: "Find Thane's father's smithing notes",
      type: "item",
      target: master_smiths_journal,
      complete: false
    },
    {
      description: "Gather rare materials for the masterwork",
      type: "collection",
      materials: [
        dragon_scale_red,
        mithril_ingot,
        phoenix_feather,
        eternal_flame
      ],
      complete: false
    },
    {
      description: "Help Thane forge the masterwork",
      type: "crafting",
      recipe: thanes_masterwork_hammer,
      complete: false
    }
  ])

  // Companion Character Development
  plug.CharacterGrowth({
    loyaltyIncrease: 50,
    newDialogue: [
      thane_backstory_complete,
      thane_gratitude,
      thane_master_smith_dialogue
    ],
    skillIncrease: {
      crafting: 5,
      combat: 2
    },
    unlocksAbility: "Master Smithing"
  })

  // Rewards
  plug.RewardExperience(5000)
  plug.RewardItems([thanes_masterwork_hammer])
  plug.RewardCompanionPerk("Master Crafting", "Thane can now craft legendary items")
  plug.RewardReputation(smiths_guild, 100)

  // Personal Story Impact
  plug.CompanionStoryResolution("Thane honors his father's legacy")
  plug.UnlocksCompanionEnding(true)

  // Current State
  plug.QuestStatus("Locked")
  plug.WatchingForTrigger(true)
}
```

**Pattern**: Quest deepens relationship with companion. Unlocks unique abilities and dialogue. Contributes to companion's personal story arc.

## World Event Quest

### Large-Scale Event Affecting Entire World

**Concept**: Major event that changes world state permanently.

```javascript
entity Quest_The_Dragon_Awakening {
  plug.Name("The Dragon Awakening")
  plug.Description("An ancient dragon awakens, threatening all civilizations")
  plug.Tag([quest_tag, world_event_tag, main_storyline_tag, apocalyptic_tag])

  plug.QuestType("World Event")
  plug.Scale("World-Altering")
  plug.AffectsAllPlayers(true)  // In multiplayer

  // World State Changes
  plug.InitiationEvent("Ancient seal breaks on Dragon Peak")

  plug.WorldChanges({
    phase1_awakening: [
      "Dragon emerges, destroys nearby settlements",
      "Refugees flood major cities",
      "Trade routes disrupted",
      "NPCs flee or prepare defenses"
    ],
    phase2_rampage: [
      "Dragon attacks major cities",
      "Factions forced to cooperate",
      "Resources become scarce",
      "Some locations permanently destroyed"
    ],
    phase3_final_battle: [
      "All factions unite against dragon",
      "Epic siege battle",
      "World fate decided"
    ]
  })

  // Global Objectives (all players contribute)
  plug.GlobalObjectives([
    {
      description: "Evacuate civilians from danger zones",
      type: "rescue",
      progress: 0,
      goal: 10000,  // 10,000 NPCs rescued
      reward: "Civilian support in final battle"
    },
    {
      description: "Gather ancient dragon-slaying weapons",
      type: "collection",
      items: [dragon_bane_sword, dragon_scale_armor, wyvern_killer_lance],
      progress: 0,
      goal: 100,  // 100 weapons collected
      reward: "Bonus damage vs dragon"
    },
    {
      description: "Research dragon weaknesses",
      type: "knowledge",
      progress: 0,
      goal: 50,  // 50 lore items found
      reward: "Reveal dragon weak points"
    }
  ])

  // Personal Quest Line
  plug.PersonalObjectives([
    "Survive initial dragon attack",
    "Join faction's defense efforts",
    "Participate in final battle",
    "Land killing blow OR support those who do"
  ])

  // Multiple Endings Based on World Performance
  plug.Outcomes({
    victory_united: {
      condition: "All global objectives complete, minimal casualties",
      outcome: "Dragon slain, world united in peace",
      worldState: "golden_age_begins",
      rewards: {
        title: "Dragon Slayer",
        monument: "statue_in_capital",
        permanentBuff: "blessing_of_the_alliance"
      }
    },
    victory_pyrrhic: {
      condition: "Dragon slain but heavy casualties",
      outcome: "Dragon defeated but world scarred",
      worldState: "rebuilding_era",
      rewards: {
        title: "Survivor",
        reputation: "war_hero"
      }
    },
    stalemate: {
      condition: "Dragon driven back but not killed",
      outcome: "Dragon retreats, uneasy peace",
      worldState: "age_of_fear",
      consequences: "Dragon may return"
    },
    defeat: {
      condition: "Dragon triumphant",
      outcome: "Dragon rules, civilizations fall",
      worldState: "age_of_dragons",
      gameChanges: "Post-apocalyptic survival mode"
    }
  })

  // Timeline
  plug.EventTimeline([
    {day: 0, event: "Dragon awakens"},
    {day: 7, event: "First city falls"},
    {day: 14, event: "Factions begin cooperation"},
    {day: 21, event: "Dragon attacks capital"},
    {day: 30, event: "Final battle begins"}
  ])

  plug.CurrentDay(0)
  plug.QuestStatus("Active")  // Automatically active for all players
}
```

**Pattern**: Massive world event that changes game state permanently. All players contribute to global objectives. Multiple possible world states based on collective performance.

## Attribution

The quest system patterns documented here were pioneered by **Lost Mountain Games** (2025), demonstrating branching narratives, dynamic quests, procedural generation, and world-altering events through pluggable quest architecture.

**Innovation**: Adaptive quests, quest chains, procedural generation, companion integration, and world-scale events using unified plug system for flexible narrative design.

---

**Copyright © 2025 Lost Mountain Games**
