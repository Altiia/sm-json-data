# Logical Requirements
Logical requirements are an important part of this project. They are used to represent any and all conditions Samus needs to fulfill to be able to perform actions. This can include having items, performing techs, and consuming ammo or health.

## Structure
A logical requirement is an array of logical elements, which are implicitly linked by a logical AND. Those logical elements can be a number of things:
* _The name of a helper._ Helpers are defined in [helpers.json](helpers.json) and they themselves represent a logical requirement. Those exist to reduce duplication and make logical requirements more readable. By convention, a helper's name should start with `h_`.
* _The name of a tech._ Techs are defined in [tech.json](tech.json).  Those represent a technique that players can perform, which may also imply logical requirements of their own. By convention, a tech's name should start with `can`.
* _The name of an item._ Those are defined in [items.json](items.json).
* _The name of a game flag._ Those are defined in [items.json](items.json), and are used to represent game events such as defeating a boss, or breaking the Maridia tube. By convention, a game flag's name should start with `f_`.
* _"never"._ This indicates a logical element that cannot be fulfilled under any conditions.
* More complex elements which will be defined in their own sub-sections

### Structural Logical Elements
Structural logical elements are arrays of other logical elements, and are fulfilled by fulfilling a given amount of those

#### and object
An `and` object is fulfilled only if all of the logical elements it contains are fulfilled

__Example:__
```json
{"and": [
  "canTunnelCrawl",
  "ScrewAttack"
]}
```

Requirements are applied in the order in which they are listed; this can matter in cases where "refill",
"ammoDrain", or "energyAtMost" requirements are involved with other resource usage requirements.

#### or object
An `or` object is fulfilled if at least one of the logical elements it contains is fulfilled

__Example:__
```json
{"or": [
  "canSwim",
  "canSuitlessMaridia"
]}
```

#### not object
A `not` object is fulfilled if the logical element given is not fulfilled

__Example:__
```json
{"not": "f_KilledMetroidRoom1"}
```

### Ammo Management Objects
This section contains logical elements that are fulfilled by spending ammo. Encountering many of those successively without having a chance to refill can impact how many power bomb/missile/super tanks can be required to get through.

#### ammo object
An `ammo` object represents the need for Samus to spend a set amount of a specific ammo. It can have the following properties:
* _type:_ The type of ammo to spend. Can have the following values:
  * PowerBomb
  * Missile
  * Super
* _count:_ The amount of ammo to spend

__Example:__
```json
{"ammo": {
  "type": "PowerBomb",
  "count": 1
}}
```

#### ammoDrain object
An `ammoDrain` object works very much like `ammo`, except that spending the ammo isn't mandatory. The ammo is just always spent if it's there. This has the same properties as an `ammo` object.

__Example:__
```json
{"ammoDrain": {
  "type": "Missile",
  "count": 75
}}
```

#### enemyKill object
An `enemyKill` object communicates the need to kill a given set of enemies, and is satisfied by having the necessary items to use one of the valid [weapons](weapons/weapons-readme.md) that will kill each of the enemies (as well as enough ammo, if applicable).

Determining what items can fulfill an `enemyKill` object should be done by doing the following:
* Identify the weapons that are valid for this `enemyKill`.
  * By default, all weapons that are not `situational` are valid
  * If the `enemyKill` object has an `explicitWeapons` property, its contents entirely overrides those default weapons
  * Weapons that are found in the `enemyKill` object's `excludedWeapons` property are rendered invalid
* For each enemy (or group of enemies), identify which valid weapons, and how many shots of them, will work. This can be determined by using the weapon's base damage and the enemy's health, damage multipliers, and invulnerabilities.
* Use the identified weapons' `useRequires` and `shotRequires` requirements to build an effective logical requirement for killing the enemies.
  * If a weapon's `shotRequires` uses ammo that is flagged as farmable in this `enemyKill`, remove the ammo cost.
  * If one of the enemies to kill has an applicable [boss scenario](enemies/bossScenarios-readme.md), that enemy's ammo cost should be obtained from the scenario calculation instead.
* If one of the enemies to kill has an applicable [boss scenario](enemies/bossScenarios-readme.md), calculate the energy cost.

An `enemyKill` object can have the following properties:
* _enemies:_ An array of groups of enemies. Those groups are themselves represented as an array. Putting enemies together in a group communicates that they can all be hit by the same shot of a weapon that `hitsGroup` (most notably a Power Bomb).
* _explicitWeapons:_ An array of weapons. These must be weapon names, weapon categories are not allowed. If this is present, defines the only weapons that may be used to fulfill this object (assuming they are actually effective to kill the enemies). If this is not present, all non-`situational` weapons may be used.
* _excludedWeapons:_ An array of weapons. These must be weapon names, weapon categories are not allowed. If this property is present, all weapons found in it may not be used to fulfill the object, regardless of whether the enemies are vulnerable to them.
* _farmableAmmo:_ An array of ammo types, which are considered farmable in the context of the `enemyKill` object. No ammo cost should be considered if using weapons that consumes that ammo type. If using a [boss scenario](enemies/bossScenarios-readme.md), this flag should be ignored and replaced by an ammo cost that takes drops and fight duration into account.

__Example:__
```json
  {"enemyKill":{
    "enemies": [
      [
        "Yellow Space Pirate (wall)",
        "Yellow Space Pirate (standing)"
      ],
      [
        "Yellow Space Pirate (wall)"
      ]
    ],
    "excludedWeapons": ["Bombs"]
  }}
```
Since Yellow Space pirates have 900 health and are immune to uncharged beam shots, this object would be fulfilled by either Charge, Screw Attack, 27 Missiles, 9 Supers, or Morph + 6 Power Bombs (3 per group, expecting that they will double-hit for 400 damage each).

__Additional considerations__

When trying to determine the ammo needed to fulfill an `enemyKill` object, it could be a good idea to factor in a player accuracy variable, to represent different levels of leniency.

### Health Management Objects
This section contains logical elements that are fulfilled by spending health. Encountering enough of those successively without having a chance to refill can impact the number of e-tanks that are needed to get through.

#### shinespark
A `shinespark` object represents the need for Samus to spend energy performing a shinespark for a given number of frames. This implicitly requires the `canShinespark` tech. It has the following properties:

* _frames_: The duration of the shinespark in frames, assuming the spark is completed without being interrupted by reaching 29 energy. The shinespark frames equals the amount of energy spent, as a shinespark uses 1 energy per frame. 
* _excessFrames_: The shinespark duration (in frames) that is not required to complete the objective of the shinespark. Subtracting this from the `frames` defines the lower limit of the shinespark energy cost.

#### acidFrames object
An `acidFrames` object represents the need for Samus to spend time (measured in frames) in a pool of acid. This is meant to be converted to a flat health value based on item loadout. The vanilla damage for acid is 6 damage every 4 frames, halved by Varia (3 damage every 4 frames), and halved again by Gravity Suit (3 damage every 8 frames).

__Example:__
```json
{"acidFrames": 35}
```

__Additional considerations__

Much like the other logical elements that represent environmental frame damage, the acid frame counts listed in this project might not be stricly "perfect" play, but they are very much unforgiving. Their most significant value is to provide relative lengths to different acid runs. It's recommended to apply a leniency factor to those, possibly as an option that can vary by difficulty.

#### gravitylessAcidFrames object
A `gravitylessAcidFrames` object represents Samus interacting with acid physics with Gravity Suit turned off, even if it is available. The number of frames here needs to be represented as `acidFrames` without the reduction effects given by Gravity Suit.

__Example:__
```json
{"gravitylessAcidFrames": 70}
```

#### draygonElectricityFrames object
A `draygonElectricityFrames` object represents the need for Samus to spend time (measured in frames) grappled to a broken turret in Draygon's room. This is meant to be converted to a flat health value based on item loadout. The vanilla damage for electricity is 1 damage per frame, halved by Varia (1 damage every 2 frames), and halved again by Gravity Suit (1 damage every 4 frames).

__Example:__
```json
{"draygonElectricityFrames": 240}
```

__Additional considerations__

While the demands on the player are very flat for the Draygon kill, execution plays a huge factor in the Draygon grapple jump. Like with other types of environmental frame damage, it might be a good idea to apply a leniency factor (possibly as an option that can vary by difficulty), although that could make the Grapple kill excessively lenient as a side effect.

#### enemyDamage object
An `enemyDamage` object represents the need for Samus to intentionally take damage from an enemy. This is meant to be converted to a flat health value based on item loadout. It has the following properties:
* _enemy:_ The name of the enemy that will damage Samus
* _type:_ The name of the attack that Samus will take damage from
* _hits:_ The number of hits Samus will take from that attack

__Example:__
```json
{"enemyDamage": {
  "enemy": "Sidehopper",
  "type": "contact",
  "hits": 3
}}
```

#### energyAtMost object

There are situations where progress causes Samus' energy to be set to a specific value, regardless of how much she had coming in. An `energyAtMost` object communicates a logical requirement that Samus' energy is reduced to a maximum of the accompanying value. Fulfilling this object does not require draining reserve tanks.

__Example:__
```json
{"energyAtMost": 1}
```

#### autoReserveTrigger object

An `autoReserveTrigger` object represents a logical requirement for "auto" reserves to be triggered, which results in Samus' energy becoming equal to the amount of energy in reserves (capped to energy capacity), and reserve energy becoming zero. It has two optional properties:

* _minReserveEnergy_: The minimum amount of energy in reserves which will satisfy this requirement (default: 1).
* _maxReserveEnergy_: The maximum amount of energy in reserves which will satisfy this requirement (default: 400).

__Example:__
```json
{"autoReserveTrigger": {}}
```

#### heatFrames object
A `heatFrames` object represents the need for Samus to spend time (measured in frames) in a heated room. This is meant to be converted to a flat health value based on item loadout. The vanilla damage for heated rooms is 1 damage every 4 frames, negated by Varia or Gravity Suit. The effect of Gravity suit on heat damage may be modified by randomizers. A `heatFrames` object implicitly includes a requirement `{"or": ["h_heatProof", "canHeatRun"]}`.

__Example:__
```json
{"heatFrames": 100}
```

__Additional considerations__

Much like the other logical elements that represent environmental frame damage, the heat frame counts listed in this project might not be stricly "perfect" play, but they are very much unforgiving. Their most significant value is to provide relative lengths to different heat runs. It's recommended to apply a leniency factor to those, possibly as an option that can vary by difficulty.

#### gravitylessHeatFrames object
A `gravitylessHeatFrames` object represents Samus in a heated environment with Gravity Suit turned off, even if it is available. The number of frames here needs to be represented as `heatFrames` without the reduction effects given by Gravity Suit.

__Example:__
```json
{"gravitylessHeatFrames": 70}
```

#### hibashiHits object
A `hibashiHits` object represents the need for Samus to intentionally take a number of hits from the Norfair flame bursts (also called hibashi). This is meant to be converted to a flat health value based on item loadout. The vanilla damage per hibashi hit is 30 with Power Suit, 15 with Varia, and 7 with Gravity Suit.

__Example:__
```json
{"hibashiHits": 1}
```

#### lavaFrames object
A `lavaFrames` object represents the need for Samus to spend time (measured in frames) in a pool of lava. This is meant to be converted to a flat health value based on item loadout. The vanilla damage for lava is 2 damage every 4 frames, halved by Varia, and negated by Gravity Suit.

__Example:__
```json
{"lavaFrames": 70}
```

__Additional considerations__

Much like the other logical elements that represent environmental frame damage, the lava frame counts listed in this project might not be stricly "perfect" play, but they are very much unforgiving. Their most significant value is to provide relative lengths to different lava runs. It's recommended to apply a leniency factor to those, possibly as an option that can vary by difficulty.

#### gravitylessLavaFrames object
A `gravitylessLavaFrames` object represents Samus interacting with lava physics with Gravity Suit turned off, even if it is available. The number of frames here needs to be represented as `lavaFrames` without the reduction effects given by Gravity Suit.

__Example:__
```json
{"gravitylessLavaFrames": 70}
```

#### samusEaterFrames object
A `samusEaterFrames` object represents the need for Samus to take damage from the environmental enemy known as a Samus Eater.  When captured, fixed damage is dealt over a set number of frames. The frame amount is 320 for ceiling and 160 for floor variations.  The vanilla damage is 2 per 20 frames in Power Suit, 1 per 20 frames in Varia Suit, 1 per 40 frames in Gravity Suit.

__Example:__
```json
{"samusEaterFrames": 160}
```

#### metroidFrames object
A `metroidFrames` object represents the need for Samus to spend time (measured in frames) grappled by a Metroid. When captured, the Metroid deals 3 damage every 4 frames. This is halved by Varia (3 damage every 8 frames), and halved again by Gravity Suit (3 damage every 16 frames).

__Example:__
```json
{"metroidFrames": 260}
```

#### spikeHits object
A `spikeHits` object represents the need for Samus to intentionally take a number of hits from spikes. This is meant to be converted to a flat health value based on item loadout. The vanilla damage per spike hit is 60 with Power Suit, 30 with Varia, and 15 with Gravity Suit.

__Example:__
```json
{"spikeHits": 6}
```

__Additional considerations__

While this is true to some extent for every strat that requires taking intentional punctual damage, spike hits especially are featured quite prominently in some unforgiving strats. It might be a good idea to apply a leniency factor to those, possibly as an option that can vary by difficulty.

#### thornHits object
A `thornHits` object represents the need for Samus to intentionally take a number of hits from the game's weaker spikes, found mainly in Brinstar. This is meant to be converted to a flat health value based on item loadout. The vanilla damage per thorn hit is 16 with Power Suit, 8 with Varia, and 4 with Gravity Suit.

__Example:__
```json
{"thornHits": 1}
```

#### resourceCapacity object
A `resourceCapacity` object represents the need for Samus to be capable of holding at least a set amount of a specific resource. It can have the following properties:
* _type:_ The type of resource. Can have the following values:
  * Missile
  * Super
  * PowerBomb
  * RegularEnergy
  * ReserveEnergy
* _count:_ The amount of capacity that Samus must have.

__Example:__
```json
{"resourceCapacity": [
    {"type": "Missile", "count": 10},
    {"type": "Super", "count": 10},
    {"type": "PowerBomb", "count": 11},
    {"type": "RegularEnergy", "count": 899}
]}
```

#### resourceAvailable object
A `resourceAvailable` object represents the need for Samus to be holding at least a set amount of a specific resource. It has the following properties:
* _type:_ The type of resource. Can have the following values:
  * Missile
  * Super
  * PowerBomb
  * RegularEnergy
  * ReserveEnergy
  * Energy (total of RegularEnergy + ReserveEnergy)
* _count:_ The amount of the resource that Samus must have.

This requirement does not consume the resource.

__Example:__
```json
{"resourceAvailable": [
    {"type": "Energy", "count": 99}
]}
```

#### resourceMissingAtMost object
A `resourceMissingAtMost` object represents the need for Samus to be missing at most a certain amount of a specific resource, relative to being full capacity. It has the following properties:
* _type:_ The type of resource. Can have the following values:
  * Missile
  * Super
  * PowerBomb
  * RegularEnergy
  * ReserveEnergy
  * Energy (total of RegularEnergy + ReserveEnergy)
* _count:_ The amount of the resource that Samus must have.

For example, a count of zero would indicate that Samus must be full on that resource type.

__Example:__
```json
{"resourceMissingAtMost": [
    {"type": "Energy", "count": 0}
]}
```

#### refill object
A `refill` object is always fulfilled and represents a process that refills certain resource types to their current maximum capacity. Possible uses could include a farm, recharge station, or Crystal Flash. The
refilled resource types are listed as an array having the following possible values:

* Missile
* Super
* PowerBomb
* RegularEnergy
* ReserveEnergy
* Energy (shorthand for RegularEnergy + ReserveEnergy)

__Example:__
```json
{"refill": ["Energy", "Missile"]}
```

#### partialRefill object

A `partialRefill` object represents a process that refills a certain resource type up to a certain level, while having no effect if already at or above that level. It has two properties:

* _type_: The resource type being partially refilled, one of the following values:
  * Missile
  * Super
  * PowerBomb
  * RegularEnergy
  * ReserveEnergy
  * Energy (combination of RegularEnergy + ReserveEnergy)
* _limit_: The level of resource amount that the refill stops at.

When applied to `Energy` type, the refill applies first to regular energy; if the refill `limit` exceeds the regular energy capacity, then regular energy will be fully refilled and the remaining amount (after subtracting regular energy capacity) will be applied as a partial refill to reserve energy.

__Example:__
```json
{"partialRefill": {"type": "Energy", "limit": 1500}}
```

### Momentum-Based Objects
This section contains logical elements centered around available running room, as well as the charging (and subsequent execution) of shinesparks.

#### canShineCharge object
A `canShineCharge` object represents the need for Samus to be able to charge a shinespark within the current room. It has the following special properties:
* _usedTiles:_ The number of tiles that are available to charge the shinespark. Smaller amounts of tiles require increasingly more difficult short charging techniques.
* The following properties further define the tiles in `usedTiles`, by indicating how many of them have some particularities. Sloped tiles impact the required number of tiles to charge a shinespark. Those properties will be missing if there are no such tiles. In places with more than 33 tiles where it's not relevant, that information will also be ommitted. All up/down tile counts assume Samus is running in the most convenient direction.
  * _gentleUpTiles:_ Indicates how many tiles gently slope upwards (like in Speed Booster Hall).
  * _gentleDownTiles:_ Indicates how many tiles gently slope downwards (like in Speed Booster Hall).
  * _steepUpTiles:_ Indicates how many tiles steeply slope upwards (like in Landing Site).
  * _steepDownTiles:_ Indicates how many tiles steeply slope downwards (like in Landing Site).
  * _startingDownTiles:_  Indicates how many tiles slope downwards at the expected start of the running space. A stutter can't be executed on those tiles.
* _openEnd:_ Any runway that is used to gain momentum has two ends. An open end is when a platform drops off into nothingness, as opposed to ending against a wall. Since those offer a bit more room, this property indicates the number of open ends that are available for charging (between 0 and 2).

__Example:__
```json
{"canShineCharge": {
  "usedTiles": 25,
  "steepUpTiles": 3,
  "steepDownTiles": 3,
  "openEnd": 1
}},
```

__Additional considerations__

* A `canShineCharge` object implicitly requires the Speed Booster. Energy requirements for the shinespark (if applicable) are specified separately using a `shinespark` object.

#### itemNotCollectedAtNode object
An `itemNotCollectedAtNode` object represents the need to have not yet collected the item at a given node in the same room. For example, such
an item could be used to overload PLMs in G-mode assuming the item has spawned. Note that any conditions for the item to spawn (e.g. for
Wrecked Ship items) are not included in this requirement and would need to be specified separately if applicable. In many situations,
an `itemNotCollectedAtNode` requirement should be accompanied by a `canRiskPermanentLossOfAccess`, if it is possible to prematurely obtain
the item and then get stuck from being unable to do the strat.

__Example:__
```json
{"requires": [
  {"itemNotCollectedAtNode": 1},
  "canRiskPermanentLossOfAccess"
]}
```

### Lock-related objects
This section contains logical elements that are affected by Lock type Objects attached to Nodes.

#### doorUnlockedAtNode object

A `doorUnlockedAtNode` object represents the need for a door to be unlocked, i.e. to be free of a lock such a red, green, yellow, or gray door shell. An example would be if the space in the door frame is needed as runway for a jump or shinecharge. 

In order to support randomizers that may modify door colors, a `doorUnlockedAtNode` requirement should be used when appropriate even if the vanilla door color is blue. If a strat has an `exitCondition`, then there is an implicit `doorUnlockedAtNode` requirement on the destination door node, unless the strat has [`bypassesDoorShell`](strats.md#bypasses-door-shell) set to `true`.

If the door node in a `doorUnlockedAtNode` also appears in the strat's [`unlocksDoors`](strats.md#unlocks-doors) property, then the `doorUnlockedAtNode` may also be fulfilled by unlocking the door as part of executing the strat (as an alternative to having been previously unlocked), and any `requires` for that locked door in the `unlocksDoors` then become part of the requirements for the strat.

__Example:__
```json
{"requires": [
  {"doorUnlockedAtNode": 1},
  "HiJump",
  "SpeedBooster"
]}
```

### Obstacle-related objects
This section contains logical elements that are affected by Obstacle type Objects within a room.

#### obstaclesCleared

An `obstaclesCleared` object represents a requirement that all of the listed obstacles must be already cleared.

__Example:__
```json
{"obstaclesCleared": ["A"]}
```

#### obstaclesNotCleared

An `obstaclesNotCleared` object represents a requirement that none of the listed obstacles be already cleared.

__Example:__
```json
{"obstaclesNotCleared": ["A"]}
```

__Additional considerations__
Entering a room does not count as executing a strat, so this logical element cannot be fulfilled instantly upon entering a room.

#### resetRoom object
A `resetRoom` object represents the need for the room to be in an initial state in order to perform a strat. A `resetRoom` object can have the following properties:
* _nodes:_ An array containing the in-room ID of nodes at which entering the room can work.
* _nodesToAvoid:_ An array containing the in-room ID of nodes that Samus must not visit after resetting the room. If any of those nodes have to be visited, the `resetRoom` object cannot be fulfilled, regardless of where Samus entered the room.
* _mustStayPut:_ This property is mutually exclusive with `nodesToAvoid` and is only meaningful for `resetRoom` objects whose only `nodes` is the one they are at. If it is present and `true`, it is equivalent to having a `nodesToAvoid` property containing all other nodes in the room.

In order to fulfill a `resetRoom` object, Samus must be able to do all of the following:
* Enter the room at one of the listed `nodes`
* Reach the node where the logic contains the `resetRoom` object
  * If `mustStayPut` is true, Samus should be entering the room at the correct node and staying there
* Do this while visiting none of the listed `nodesToAvoid`. However, it's ok if a node to avoid ends up being visited directly afterwards, as a result of fulfilling the `resetRoom` object.
* If Samus is already in the room and has done one of the actions to avoid, she must be able to exit at one of the listed `nodes` and re-enter, following all other rules.
  * Please note that if fulfilling a `resetRoom` object involves exiting and re-entering, it will indeed reset the room and cause all obstacles to respawn.

__Example:__
```json
{"resetRoom":{
  "nodes": [1, 2]
}}
```
