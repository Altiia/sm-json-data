# Strats
Strats are a concept that encompasses performing a series of maneuvers in a room.

Strats are usually presented within an array of strats, where all individual strats are a way to attain roughly the same goal.

## Structure

A `strat` can have the following properties:
  * _name_: The name of the strat.
  * _notable_: Indicates whether the strat is notable.
  * _reusableRoomwideNotable_: The name of the reusable roomwide notable strat. This must share an identical name with an entry in the `reusableRoomwideNotable` array and the other strats that are connected to this one. This is only applicable for strats where `notable` is `true`.
  * _entranceCondition_: Indicates that this strat requires entering through the door transition in a special way, combining with a strat in the previous room.
  * _requires_: The [logical requirements](logicalRequirements.md) that must be fulfilled to execute that strat.
  * _exitCondition_: Indicates that this strat leaves through the door transition in a special way that combines with a strat in the next room. 
  * _clearsObstacles_: An array containing the ID of obstacles that will be cleared by executing this strat (if they are not already cleared).
  * _resetsObstacles_: An array containing the ID of obstacles that will be reset (i.e. returned to their original state) by executing this strat.
  * _comesThroughToilet_: Indicates whether this strat is applicable if the Toilet comes between this room and the other room.
  * _gModeRegainMobility_: Indicates that this strat allows regaining mobility when entering with G-mode immobile.
  * _bypassesDoorShell_: Indicates that this strat allows exiting without opening the door.
  * _unlocksDoors_: An array describing possible doors that can be unlocked as part of this strat.
  
These properties are described below in more detail.
### Example

Here's a simple example of a strat:

```json
{
  "name": "Base",
  "notable": false,
  "requires": ["Morph"]
}
```

## Notable Strats

Some strats are flagged as "notable". this means a few things:
  * The strat's name must be unique among all existing strats in the project.
  * Just having the required items and the ability to perform the required techs may not be enough to execute a notable strat. It might require:
    * A higher difficulty than what the tech would typically imply
    * Knowledge of the strat or specific to the strat (such as an obscure strat or unique setup)
  * An algorithm using this project to drive its logic should consider allowing to toggle off (or on) any given notable strat.

## Logical Requirements

A strat has [logical requirements](logicalRequirements.md) which must be fulfilled in order to execute it. These must always be specified, although they may be explicitly set to an empty array if there are no requirements.

## Clears Obstacles

Execution of a strat may have an effect of clearing one or more [obstacles](region/region-readme.md#obstacles) in the room. This is indicated by a `clearsObstacles` property on the strat. It can represent, for example, that certain enemies or special blocks are destroyed by executing this strat. This allows later performing strats in the room that have an [`obstaclesCleared`](logicalRequirements.md#obstaclescleared) logical requirement on this obstacle, while disallowing strats that have an [`obstaclesNotCleared`] requirement.

Similarly, the `resetsObstacles` property is used to indicate that a strat results in an obstacle returning to its original, uncleared state.

## Reusable Roomwide Notable Strats

Some notable strats may be very similar to each other. If similar notable strats exist in different rooms, a tech might be made for them. Otherwise, if similar notable strats exist in the same room, a [reusable notable strat](region/region-readme.md#reusable) can exist to connect them with a common name and description. A typical example is for symmetric links with strats of notable difficulty, with one notable strat for each of two directions the room can be traversed in, and the two strats being connected by sharing the same reusable notable.

## Cross-room strats

Some strats involve leaving the room in a special way that allows a corresponding strat in the next room to be executed. Common examples include leaving the room with a shinespark or shinecharge, or running out of the room in order to complete a shinecharge in the next room. Strats that exit a room in such a way are identified by a strat property `exitCondition`, containing a property describing the type of exit condition, such as `leaveWithRunway`, `leaveShinecharged`, or `leaveWithSpark`. Likewise, strats that require entering the room in a special way are identified by a strat property `entranceCondition`, containing a property describing the type of entrance condition required, such as `comeInRunning`, `comeInShinecharged`, or `comeInWithSpark`.

## Runway geometry

Many cross-room strats involve the use of runways. The geometry of a runway is summarized by an object having the following properties:

* _length:_ The number of tiles in the runway. 
* _openEnd:_ Any runway that is used to gain momentum has two ends. An open end is when a platform drops off into nothingness, as opposed to ending against a wall or a door transition tile. An open end offers a bit more room to run. This property indicates the number of open ends that are available to use (0 or 1).
* The following properties further define the tiles in `length`, by indicating how many of them have some particularities. Samus temporarily moves more slowly over sloped tiles, which effectively gives a longer distance over which to gain speed on the runway. These properties will be missing if there are no such tiles. In places with more than 45 tiles where it's not relevant, that information will also be omitted.
  * _gentleUpTiles:_ Indicates how many tiles gently slope upwards (like in Speed Booster Hall).
  * _gentleDownTiles:_ Indicates how many tiles gently slope downwards (like in Speed Booster Hall).
  * _steepUpTiles:_ Indicates how many tiles steeply slope upwards (like in Landing Site).
  * _steepDownTiles:_ Indicates how many tiles steeply slope downwards (like in Landing Site).
  * _startingDownTiles:_  Indicates how many tiles slope downwards at the expected start of the running space. A stutter can't be executed on those tiles.

In most cases, what matters is the effective runway length (ERL), which takes into account how slopes temporarily slow down Samus' horizontal movement:

`ERL = length - startingDownTiles - 9/16 * (1 - openEnd) + 1/3 * steepUpTiles + 1/7 * steepDownTiles + 5/27 * gentleUpTiles + 5/59 * gentleDownTiles`

There is some variance in how much downward slopes slow Samus' movement, depending on specific alignment of Samus' X position. The amount of variance is small enough to be neglected as long as some small amount of lenience is included in how runways are required to be used.

## Exit conditions

In all strats with an `exitCondition`, the `to` node of the strat must be a door node or exit node. An `exitCondition` object must contain exactly one property, which indicates the type of exit condition provided by the strat:

- _leaveNormally_: This indicates that can Samus leave through this door in a "normal" way, by walking, falling, or jumping.
- _leaveWithRunway_: This indicates that a runway of a certain length is connected to the door, with which Samus can gain speed and run or jump through the door, among other possible actions. 
- _leaveShinecharged_: This indicates that it is possible to charge a shinespark and leave the room with a certain amount of time remaining on the shinecharge timer (e.g., so that a shinespark can be activated in the next room). 
- _leaveWithTemporaryBlue_: This indicates that Samus may leave through this door by jumping with temporary blue.
- _leaveWithSpark_: This indicates that it is possible to shinespark through the door transition.
- _leaveWithStoredFallSpeed_: This indicates that is is possible to walk through the door with the stored velocity to clip through floor tiles using a Moonfall.
- _leaveWithGModeSetup_: This indicates that Samus can take enemy damage through the door transition, to set up R-mode or direct G-mode in the next room.
- _leaveWithGMode_: This indicates that Samus can carry G-mode into the next room (where it will become indirect G-mode).
- _leaveWithDoorFrameBelow_: This indicates that Samus can go up through this door with momentum by jumping in the door frame, e.g. using a wall-jump or Space Jump.
- _leaveWithPlatformBelow_: This indicates that Samus can go up through this door with momentum by jumping from a platform below, possibly with run speed.
- _leaveWithGrappleTeleport_: This indicates that Samus can leave through this door while grappling, which can enable a teleport in the next room.

Each of these properties is described in more detail below.

A strat with an exit condition implicitly has a `doorUnlockedAtNode` requirement on its `to` node, unless it has the `bypassesDoorShell` property set to `true`. This means that if the `to` door has a lock on it, it must either be unlocked before the strat can be executed, or the door's requirements under the strat property `unlocksDoors` must be satisfied. 

### Leave Normally

A `leaveNormally` object indicates that Samus can leave the room through this door, with no other particular requirements.

#### Example
```json
{
  "name": "Leave Normally",
  "exitCondition": {
    "leaveNormally": {}
  },
  "requires": []
}
```

### Leave With Runway
A `leaveWithRunway` object indicates that a strat exits the current room using a runway. The `leaveWithRunway` exit condition is unique in that it describes available geometry rather than a specific way to leave the room. This is done in order to reduce the amount of redundant boilerplate that would otherwise be required, since every door node in the game will have at least one strat with `leaveWithRunway`. The specific way that the runway is used depends on the entrance condition in the destination room. A `leaveWithRunway` condition implies that the area around the door must be clear of any enemies that would interfere with using the runway.

A `leaveWithRunway` exit condition can satisfy the following entrance conditions in the next room: `comeInRunning`, `comeInJumping`, `comeInShinecharging`, `comeInShinecharged`, `comeInWithSpark`, `comeInWithBombBoost`, `comeInWithStutter`, `comeInWithDoorStuckSetup`, `comeInSpeedballing`, and `comeInWithTemporaryBlue`. Details are given under the corresponding entrance conditions below.

`leaveWithRunway` has the following properties describing the runway geometry (see [runway geometry](#runway-geometry) above for details) :

* _length_, _openEnd_, _gentleUpTiles_, _gentleDownTiles_, _steepUpTiles_, _steepDownTiles_, _startingDownTiles_

The runway length should not include the transition tile, but it should include the door shell tile if applicable.

In a heated room, a `leaveWithRunway` exit condition implicitly includes `heatFrames` needed to use the runway. The amount of `heatFrames` required depends on the entrance condition in the next room, and the details of how these `heatFrames` may be calculated are described under each entrance condition [below](#entrance-conditions). If the `from` and `to` nodes of the strat are different nodes, then it is assumed that the strat property `requires` already includes any heat frames needed to reach the starting point of the runway (the side furthest from the door), in which case the implicit `heatFrames` in `leaveWithRunway` will only account for the heat frames needed to use the runway in one direction, moving towards the door. If the `from` and `to` nodes of the strat are the same node, then the implicit `heatFrames` includes all heat frames needed to position appropriately on the runway (starting from the door), execute the required movement, and exit the room through the door.

When a `leaveWithRunway` condition is used to charge a shinespark, it implicitly includes a `canShinecharge` requirement with `usedTiles` based on the runway length (and also based on a continuation of the runway in the destination room where applicable). Again, this is described in more detail under the applicable entrance conditions below.

When a `leaveWithRunway` conditions occurs on a door in a water environment, it may implicitly include a `Gravity` requirement, depending on whether this is needed for the corresponding entrance condition.

#### Example
```json
{
  "name": "Leave With Runway",
  "notable": false,
  "requires": [],
  "exitCondition": {
    "leaveWithRunway": {
      "length": 20,
      "openEnd": 1
    }
  }
}
```

### Leave Shinecharged

A `leaveShinecharged` object represents that Samus can leave through this door with a shinecharge (shinespark charge).

`leaveShinecharged` has a single property:
- _framesRemaining_: The number of frames remaining in the shinecharge when leaving the room. A special value "auto" may be used when the strat has a `comeInShinecharged` entrance condition: in this case, the frames remaining when leaving the room depends on how many frames are remaining when entering the room, with the `framesRequired` property of the `comeInShinecharged` indicating the amount of frames by which the shinecharge timer decreases between entering and exiting the room.

A strat with a `leaveShinecharged` condition should either include a `canShinecharge` requirement in its `requires` or have a `comeInShinecharged` entrance condition.

*Note*: Using a runway connected to a door to leave the room with a shinecharge is already covered by `leaveWithRunway`, so `leaveShinecharged` only needs to be used in cases where the shinecharge is obtained in another part of the room and then carried through the door.

#### Position and Momentum Details

A `leaveShinecharged` object does not provide any way to specify Samus' position or momentum through the door transition, but these details can affect the execution of the strat in the next room. Instead, as a way of normalizing these requirements, we make the following assumptions:

- For a horizontal door transition, the `framesRemaining` (and any other strat requirements such as heat frames) should be based on Samus touching the door transition while running, with an unspecified amount of horizontal momentum but no vertical momentum.

- For a vertical door transition, the `framesRemaining` (and other strat requirements) should be based on the worst-case position, among all possible horizontal positions within the door frame, with an unspecified amount of horizontal and vertical momentum. Samus must enter the transition in a normal "jumping" or "falling" pose. For example, entering an upward transition with a jump into mid-air morph would not be valid, because in the next room it would not be possible to control the effects of the jump in the expected way. Breaking spin before the transition would be valid, and it is recommended to do so since it allows reaching the transition more quickly.

#### Example
```json
{
  "name": "Leave Shinecharged",
  "notable": false,
  "requires": [
    {"canShinecharge": {
      "usedTiles": 20,
      "openEnd": 0
    }}
  ],
  "exitCondition": {
    "leaveShinecharged": {
      "framesRemaining": 90
    }
  }
}
```

### Leave With Temporary Blue

A `leaveWithTemporaryBlue` exit condition represents that Samus can leave through this door by jumping with temporary blue. It has no properties.

The `leaveWithTemporaryBlue` object has the following property:
- _direction_: This takes two possible values "left" and "right", indicating the direction that Samus is facing through the transition. It should be specified for all vertical transitions but not horizontal ones.

*Note*: Using a runway connected to a door to leave the room with temporary blue is already covered by `leaveWithRunway`.

#### Example
```json
{
  "name": "Leave With Temporary Blue",
  "notable": false,
  "requires": [
    {"canShinecharge": {
      "usedTiles": 20,
      "openEnd": 0
    }}
  ],
  "exitCondition": {
    "leaveWithTemporaryBlue": {}
  }
}
```

### Leave With Spark

A `leaveWithSpark` exit condition represents that Samus can leave through this door while shinesparking. A strat with a `leaveWithSpark` condition should include a `canShinecharge` and `shinespark` requirements in its `requires`.

The `leaveWithSpark` object has the following property:
- _position_: For a horizontal transition, if specified, this takes two possible values, "top" or "bottom". The value "top" represents that the strat sparks through the doorway high enough to clear a single-tile block level with the bottom of the doorway. The value "bottom" represents that the strat sparks through the doorway low enough to clear a single-tile block level with the top of the doorway. If unspecified, it is understood that the strat can exit through either the top or bottom of the doorway, whichever is needed in the next room.

The direction of the spark is assumed to be horizontal when sparking through horizontal door transitions, or vertical when sparking through vertical door transitions. There is an implicit requirement of `canHorizontalShinespark` when sparking through a horizontal door.

*Note*: Using a runway connected to a door to leave the room with a shinespark is already covered by `leaveWithRunway`. Likewise `leaveShinecharged` implicitly includes the possibility of leaving the room with a shinespark. It is only necessary to use `leaveWithSpark` in cases where it would not be possible to reach the door before the shinecharge timer expires.

#### Example
```json
{
  "name": "Leave With Spark",
  "notable": false,
  "requires": [
    {"canShinecharge": {
      "usedTiles": 20,
      "openEnd": 0
    }},
    {"shinespark": {
      "frames": 200
    }}
  ],
  "exitCondition": {
    "leaveWithSpark": {}
  }
}
```

### Leave With Stored Fall Speed

A `leaveWithStoredFallSpeed` exit condition represents that Samus can leave through this door with stored fall speed.

The `leaveWithStoredFallSpeed` object has a single property:
- _fallSpeedInTiles_: The number of tiles Samus would clip through by Moonfalling on top of a solid floor.

A `leaveWithStoredFallSpeed` entrance condition must match with a `comeInWithStoredFallSpeed` condition on the other side of the door.  A strat with a `leaveWithStoredFallSpeed` condition must either include a method of storing fall speed within its requirements, such as a Moondance.  Entering a room with a `comeInWithStoredFallSpeed` condition would also be possible exiting with a `leaveWithStoredFallSpeed` condition so long as the stored speed is not lost.  For this to happen, both doors must be connected by one `Runway`, and Samus must not Crouch or become Knocked back.

#### Example
```json
{
  "name": "Moondance to Store Fall Speed",
  "notable": false,
  "requires": [
    "h_canUseBombs",
    "canMoondance"
  ],
  "exitCondition": {
    "leaveWithStoredFallSpeed": {
      "fallSpeedInTiles": 1
    }
  }
}
```

### Leave with G-Mode Setup

A `leaveWithGModeSetup` exit condition represents that Samus can leave through this door while taking damage through the transition, in a pose that would allow using X-Ray on the first frame after the transition. This sets up the player to enter R-mode or direct G-mode in the next room. The only known way to achieve this is to use an enemy that can follow Samus into the doorway during the transition. It will not work with enemy projectiles since these do not move during transitions, and environmental damage such as heat, lava, acid do not work as these are not active during the transition. Also note that the damage must happen *during* (not *before*) the transition, so being able to take a hit that knocks Samus into the door transition does not work. The enemy damage through the transition should _not_ be included in the `requires`, as the type/amount of enemy damage is irrelevant since Samus' energy will always reach zero here in order to trigger reserves.

A `leaveWithGModeSetup` object contains the following property:
- _knockback_: If true, then Samus gets knockback frames through the transition. This makes it possible for Samus to retain mobility in the next room if reserve energy is at most 4. Most enemies provide knockback, so this property is true by default if unspecified. Certain enemies such as Beetoms, Metroids, and Mochtroids do not provide knockback, so this property should be set to false for strats that use these enemies for the transition damage.

A `leaveWithGModeSetup` comes with implicit requirements, which are described in detail under the entrance conditions `comeInWithRMode` and `comeInWithGMode`. The implicit requirements depend on the specifics of the entrance condition in the next room, but they always include at least the following:
- The `XRayScope` item requirement.
- A requirement to have at least 1 reserve energy.
- A requirement to damage down to 0 energy, triggering reserves.

#### Example
```json
{
  "name": "Leave With G-Mode Setup",
  "notable": false,
  "requires": [],
  "exitCondition": {
    "leaveWithGModeSetup": {}
  }
}
```

### Leave with G-Mode

A `leaveWithGMode` exit condition represents that Samus can leave through this door while in G-mode, resulting in indirect G-mode in the next room. A strat with a `leaveWithGMode` exit condition should always also have a `comeInWithGMode` entrance condition since the only (known) way to enter G-mode is while (or before) coming into the room.

A `leaveWithGMode` object has the following property:
- _morphed_: If true, then this strat results in leaving the room in a morphed state, either by maintaining artificial morph or by having the Morph item.

For most doors in the game, it is possible to enter the room with a G-mode setup and then immediately exit back through the same door. This is because in direct G-mode the door does not close behind Samus. To avoid the need to write out tedious boilerplate, these strats are understood to be included implicitly. Specifically, for every door node without a `spawnAt` property, there are two implicit strats, with `leaveWithGMode` exit conditions, one of the form

```json
{
  "name": "G-Mode Go Back Through Door",
  "notable": false,
  "entranceCondition": {
    "comeInWithGMode": {
      "mode": "direct",
      "morphed": false
    }
  },
  "requires": [],
  "exitCondition": {
    "leaveWithGMode": {
      "morphed": false
    }
  }
}
```

and another where the `false` values for `morphed` are replaced with `true`. Here we are referring to nodes with `"nodeType": "door"`, which excludes sand entrances (which instead have `"nodeType": "entrance"`). These implicit strats have the same door node for both their `from` and `to` nodes.

Aside from the implicit strats, there are a limited amount of `leaveWithGMode` strats possible. Normally entering a room with G-mode (or a G-mode setup) and then leaving with G-mode through a different door is not possible, since door shells cannot be opened while in G-mode. However, some door transitions do not have door shells (e.g. in Crateria Tube, Glass Tunnel, Crab Hole, Big Pink; also elevators and sand transitions), and some door shells are possible to bypass using glitches, so `leaveWithGMode` can be used in these situations.

### Leave With Door Frame Below

A `leaveWithDoorFrameBelow` exit condition represents that Samus can go up through this door with momentum by jumping in the door frame, e.g. using a wall-jump or Space Jump. A `leaveWithDoorFrameBelow` exit condition can satisfy `comeInWithWallJumpBelow` and `comeInWithSpaceJumpBelow` entrance conditions in the room above.

A `leaveWithDoorFrameBelow` object has the following property:

- _height_: The number of tiles beneath the door transition (not including the transition tiles) usable for wall-jumping.

In a heated room, heat frames must be explicitly included in the strat `requires`, based on an assumption of wall-jumping up through the door. If the strat in the neighboring room has a `comeInWithSpaceJumpBelow` entrance condition, then additional heat frames will be implicitly included to use Space Jump, so that does not need to be included in the `leaveWithDoorFrameBelow` strat. Likewise, requirements for `canWalljump` or `SpaceJump` do not need to included, as these will be implicitly included in the corresponding entrance conditions. If a strat starts at the same (door) node that it ends at, then heat frames should include the time required to fall down from the door, shoot it open, and then wall-jump back out.

#### Example
```json
{
  "name": "Leave With Door Frame Below",
  "requires": [],
  "exitCondition": {
    "leaveWithDoorFrame": {
      "height": 2
    }
  }
}
```

### Leave With Platform Below

A `leaveWithPlatformBelow` exit condition represents that that Samus can go up through this door with momentum by jumping from a platform below, possibly with run speed. A `leaveWithPlatformBelow` exit condition can satisfy a `comeInWithPlatformBelow` entrance condition in the room above.

A `leaveWithPlatformBelow` object has the following properties:

- _height_: The number of tiles between the door transition and the platform, not including the transition tiles or platform itself. A horizontal slope tile (as in Blue Hopper Room) counts as a half tile.
- _leftPosition_: This indicates the position of the furthest left usable tile of the platform, relative to the center of the door. A negative values indicates a position to the left of the door center, while a positive value indicates a position to the right of the door center. An open end, if applicable, is represented by an extra half tile.
- _rightPosition_: This indicates the position of the furthest right usable tile of the platform, relative to the center of the door. A negative values indicates a position to the left of the door center, while a positive value indicates a position to the right of the door center. An open end, if applicable, is represented by an extra half tile.

In a heated room, heat frames must be explicitly included in the strat `requires`, based on a worst-case assumption of how the platform could need to be used. If a strat starts at the same (door) node that it ends at, then heat frames should include the time required to fall down from the door, shoot it open, position at the far end of the runway, and jump out.

#### Example
```json
{
  "name": "Leave With Platform Below",
  "requires": [],
  "exitCondition": {
    "leaveWithDoorFrame": {
      "height": 7,
      "leftPosition": -2.5,
      "rightPosition": 2.5,
    }
  }
}
```

## Leave With Grapple Teleport

A `leaveWithGrappleTeleport` exit condition represents that Samus can leave through this door while grappled, which can enable a teleport in the next room. The position of the block that Samus is grappled to (counted in tiles from the top left corner of the room) determines the destination of the teleport in the next room. For the teleport to work, the next room must have a block in a corresponding position that Samus can remain grappled to; e.g. a solid tile will work but air will not.

A `leaveWithGrappleTeleport` object has the following property:

- _blockPositions_: A list of tile coordinates of (grapple) blocks that Samus could be grappled to while exiting the room. Coordinates `[x, y]` are represented as tile counts with `[0, 0]` representing the top-left corner of the room.

A `leaveWithGrappleTeleport` comes with an implicit tech requirement `canGrappleTeleport`.

#### Example
```json
{
  "name": "Leave with Grapple Teleport",
  "requires": [],
  "exitCondition": {
    "leaveWithGrappleTeleport": {
      "blockPositions": [[5, 3]]
    }
  }
}
```

## Entrance conditions

In all strats with an `entranceCondition`, the `from` node of the strat must be a door node or entrance node. An `entranceCondition` object must contain exactly one of the following properties:

- _comeInNormally_: This indicates that Samus must come into the room through the specified door, with no other particular requirements.
- _comeInRunning_: This indicates that Samus must run into the room, with speed in a certain range.
- _comeInJumping_: This indicates that Samus must run and jump just before hitting the transition, with speed in a certain range.
- _comeInShinecharging_: This indicates that Samus must run into the room with enough space to complete a shinecharge.
- _comeInShinecharged_: This indicates that Samus must enter the room with a shinecharge with a certain amount of frames remaining.
- _comeInShinechargedJumping_: This indicates that Samus must jump into the the room with a shinecharge with a certain amount of frames remaining.
- _comeInWithSpark_: This indicates that Samus must shinespark into the room.
- _comeInWithBombBoost_: This indicates that Samus must come into the room with a horizontal bomb boost.
- _comeInStutterShinecharging_: This indicates that Samus must run into the room with a stutter immediately before the transition.
- _comeInWithDoorStuckSetup_: This indicates that Samus must enter the room in a way that allows getting stuck in the door as it closes.
- _comeInSpeedballing_: This indicates that Samus must enter the room either in a speedball from the previous room, or in a process of running, jumping, or falling into a speedball.
- _comeInWithTemporaryBlue_: This indicates that Samus must come in by jumping through this door with temporary blue.
- _comeInWithStoredFallSpeed_: This indicates that Samus must enter the room with fall speed stored, and is able to clip through a floor with a Moonfall.
- _comeInWithRMode_: This indicates that Samus must have or obtain R-mode while coming through this door.
- _comeInWithGMode_: This indicates that Samus must have or obtain G-mode (direct or indirect) while coming through this door. 
- _comeInWithWallJumpBelow_: This indicates that Samus must come up through this door with momentum by wall-jumping in the door frame below.
- _comeInWithSpaceJumpBelow_: This indicates that Samus must come up through this door with momentum by using Space Jump in the door frame below.
- _comeInWithPlatformBelow_: This indicates that Samus must come up through this door with momentum by jumping from a platform below, possibly with run speed.
- _comeInWithGrappleTeleport_: This indicates that Samus must come into the room while grappling, teleporting Samus to a position in this room corresponding to the location of the (grapple) block in the other room.

In addition it may contain the following property:

- _comesThroughToilet_: This indicates whether the strat is applicable if the Toilet comes between this room and the other room.

Each of these properties is described in more detail below.

### Come In Normally

A `comeInNormally` entrance condition represents the need for Samus to enter the room through this door, with no other particular requirements. It has no properties.

A `comeInNormally` condition can be satisfied by a matching strat on the other side of the door with a `leaveNormally` or `leaveWithRunway` exit condition. 

When matching with a `leaveWithRunway` condition in a heated room on a strat having distinct `from` and `to` nodes, heat frame requirements must be included for the run performed in the other room; for details, see the `comeInRunning` condition below.

#### Example
```json
{
  "name": "Come In Normally",
  "notable": false,
  "entranceCondition": {
    "comeInNormally": {}
  },
  "requires": []
}
```

### Come In Running

A `comeInRunning` entrance condition represents the need for Samus to be able to run into the room with speed in a certain range. It has the following properties:
* _speedBooster_: If true, then Speed Booster must be used while running into the room. If false, then Speed Booster must not be used. If "any", then Speed Booster may or may not be used.
* _minTiles_: The minimum horizontal speed that will satisfy the condition, measured in effective runway tiles with dash held.
* _maxTiles_: The maximum horizontal speed that will satisfy the condition, measured in effective runway tiles with dash held.

A `comeInRunning` condition can be satisfied only by a matching strat on the other side of the door with a `leaveWithRunway` exit condition: a match is valid if all of the following are true:
- The effective runway length of the `leaveWithRunway` is at least as long as the `minTiles` in the `comeInRunning` condition.

Where applicable, a `comeInRunning` condition also includes implicit requirements for actions to be performed in the previous room, which are effectively prepended to the start of the strat's `requires` (or equivalently but more properly, onto the end of the `requires` of the `leaveWithRunway` strat in the other room):
- If `speedBooster` is true, then there is an implicit `SpeedBooster` item requirement.
- If `speedBooster` is false, then there is an implicit `canDisableEquipment` tech requirement.
- If the previous room is heated, then `heatFrames` are required based on the time needed to perform the run. 
- If the previous door environment is water, then `Gravity` is required.

Heat frames may be calculated based on the following table of approximations (which include almost no lenience):

| Action                                                                  | Heat frames             |
| ----------------------------------------------------------------------  | ----------------------- |
| Run from a standstill in one direction, without Speedbooster, 0-6 tiles | `9 + 4 * tiles`         |
| Run from a standstill in one direction, without Speedbooster, 7+ tiles  | `13 + 64 / 19 * tiles`  |
| Run from a standstill in one direction, with Speedbooster, 0-6 tiles    | `9 + 4 * tiles`         |
| Run from a standstill in one direction, with Speedbooster, 7-16 tiles   | `15 + 3 * tiles`        |
| Run from a standstill in one direction, with Speedbooster, 17-42 tiles  | `32 + 2 * tiles`        |
| Run from a standstill in one direction, with Speedbooster, 43+ tiles    | `47 + 64 / 39 * tiles`  |
| Turn around                                                             | `8`                     |
| Position using moonwalk (and shoot open the door)                       | `12`                    |

The way to calculate minimally required heat frames depends on the type of `leaveWithRunway`:

- If the `from` node of the `leaveWithRunway` is the same as the `to` node, then this represents that the runway is used starting from the door. In this case heat frames must be included for running in both directions. The distance to run is determined by the `minTiles` in the `comeInRunning` object. The heat frames for this run length should be doubled; then heat frames should be added to allow time to turn around and position. Even with optimal movement, time to position is necessary if the full length of the runway is required; in theory this time could be reduced by using X-Ray Scope to buffer the positioning and to turn-around in place.
- If the `from` node of the `leaveWithRunway` is different from the `to` node, then this represents that the runway is used starting at the end opposite the door. In this case heat frames only need to be included for running in one direction, toward the door. However, if the `comeInRunning` condition has a `maxTiles` and it is less than the effective runway length in the `leaveWithRunway`, then it will not work to run through the entire length of the runway; there are a couple options in this case:
  - The player can hold run until at a distance of `maxTiles` from the transition, then stop and restart running (by briefly releasing forward while holding an angle button). In this case the runway is partitioned into two runs, and the heat frames from the two individual runs must be added, along with any time for positioning.
  - The player can hold dash at the beginning of the runway for a distance between `minTiles` and `maxTiles` (with the optimal strategy being to run as close to `maxTiles` as possible without going over), then release dash for the remainder of the run to maintain a constant speed.

#### Example
```json
{
  "name": "Come In Running",
  "notable": false,
  "entranceCondition": {
    "comeInRunning": {
      "speedBooster": "any",
      "minTiles": 2
    }
  },
  "requires": []
}
```

### Come In Jumping

A `comeInJumping` entrance condition represents the need for Samus to be able to run toward the door in the previous room, with speed in a certain range, and spin jump just before hitting the transition. It has the following properties:
* _speedBooster_: If true, then Speed Booster must be used while running into the room. If false, then Speed Booster must not be used. If "any", then Speed Booster may or may not be used.
* _minTiles_: The minimum horizontal speed that will satisfy the condition, measured in effective runway tiles with dash held.
* _maxTiles_: The maximum horizontal speed that will satisfy the condition, measured in effective runway tiles with dash held.

`comeInJumping` is very similar to `comeInRunning`, the only difference being the jump which happens just before the transition. Heat frames and other implicit requirements are generated in exactly the same way.

#### Example
```json
{
  "name": "Cross Room Jump",
  "notable": false,
  "entranceCondition": {
    "comeInJumping": {
      "speedBooster": "any",
      "minTiles": 2
    }
  },
  "requires": []
}
```

### Come In Shinecharging

A `comeInShinecharging` entrance condition represents the need for Samus to run into the room with enough space to complete a shinecharge. It has the following properties describing the geometry of the runway in the current room which can be used to help complete the shinecharge (see [runway geometry](#runway-geometry) above for details):

* _length_, _openEnd_, _gentleUpTiles_, _gentleDownTiles_, _steepUpTiles_, _steepDownTiles_

The length should not include any tiles that Samus skips over through the transition (e.g. door transition tiles and door shell tiles). It also has the following property:

* _minTiles_: If specified, this is the minimum number of tiles of runway in the other room required to be used, in order to gain enough momentum.

A `comeInShinecharging` must match with a corresponding `leaveWithRunway` condition on the other side of the door. 

A `comeInShinecharging` object also includes implicit requirements for actions to be performed in the previous room, which are effectively prepended to the start of the current strat's `requires` (or equivalently but more properly, onto the end of the `requires` of the `leaveWithRunway` strat in the other room):
- A `canShinecharge` requirement is included based on the combined runway length. This includes a `SpeedBooster` item requirement as well as a check that the combined runway length is long enough that charging a shinespark is possible.
- If the previous room is heated, then `heatFrames` are included based on the time spent running in that room.
- If the current room is heated, then `heatFrames` are included based on the time spent running in this room.
- If the previous door environment is water, then `Gravity` is required.

The way to calculate minimally required heat frames depends on the type of `leaveWithRunway` and on which of the two rooms (or both) are heated:

- If the `from` node of the `leaveWithRunway` is the same as the `to` node, then this represents that the runway in the other room is used starting from the door. In this case Samus will need to run in both directions. The way to calculate heat frames then depends on which rooms are heated:
  - If both rooms are heated, then it is best to use smallest amount of runway possible in the other room. If the required shinecharge tiles
  (based on the desired difficulty) is no more than the effective runway length in the current room (based on the `comeInShinecharging` properties), then there is no need to add heat frames for running in the other room. Otherwise, the amount of runway to use in the other room is the difference between the required shinecharge tiles and the effective runway length in the current room (but at least `minTiles`, if specified). The run in the other room to get positioned on the runway can be done at full speed, so the [table](#come-in-running) can be used to determine the required heat frames for this run, including heat frames for turning around and positioning. For the run back to charge the spark, there are two cases:
     - If the combined effective runway length is at least 31.3 tiles, then dash can be held through the entire run, so the [table](#come-in-running) can be used to get the total heat frames. 
     - If the combined effective runway length is less than 31.3 tiles, then run back to charge the spark requires a constant 85 frames (essentially independent of shortcharging technique).

  - If only the current room is heated, then heat frames are minimized by using the full available runway in the other room to gain as much speed as possible before the transition. 
    - If the combined effective runway length is at least 31.3 tiles, then dash can be held through the entire run. In this case, the frames spent on the combined run can be found in the [table](#come-in-running), as can the frames spent in the other room; subtracting these two values gives the frames spent on the heated part of the run (i.e. the part in the current room).
    - If the combined effective runway length is less than 31.3 tiles, then the run can be completed in the constant 85 frames needed to perform the shinecharge (regardless of shortcharging technique). The precise amount of frames spent in the current room is complicated to determine as it depends on details of how the shortcharge is performed. We can get a reasonable approximation by modeling as the movement as starting at a speed of 0.125 tiles/frame (somewhat less than the full walking speed of 0.171875 tiles/frame) and having constant acceleration up to the end of the combined runway. This is a quadratic model which has the following solution:

      $$T = \text{total time} = 85 \text{ frames}$$
      $$C = \text{initial speed} = 0.125 \text{ tiles per frame}$$
      $$L = \text{combined effective runway length (in tiles)}$$
      $$L_o = \text{effective runway length in other room}$$
      $$a = \text{acceleration} = 2(L - CT) / T^2$$
      $$T_o = \text{time in other room} = \frac{\sqrt{C^2 + 2aL_o} - C}{a}$$
      $$T_c = \text{time in current room} = T - T_o$$

    For medium-length runways (ones significantly longer than the player's minimum shortcharge length, but significantly less than 31.3 tiles), this model is somewhat lenient. It assumes essentially that the player would perform the shortcharge over the full length of the runway, like doing a 4-tap with relatively long, safe taps. If instead the player did a more precise start to the shortcharge and then held dash for the remainder of the run (e.g. like a stutter 2-tap), then the heat frames in the current room could be slightly reduced. This could be modeled, but it is probably not worth the hassle for the small difference it would make.

  - If only the other room is heated, then it is again best to use smallest amount of runway possible in the other room. The heat frames for the first run in the other room (if it is needed) and to turn-around and position can be obtained using the [table](#come-in-running). Heat frames for the run back can be approximated using either the [table](#come-in-running) or the formula for $T_o$ in the quadratic model above, depending on if the combined runway has length greater 31.3 tiles. In the case of medium-length runways, this model is technically a bit lenient. We are assuming a uniform rate of acceleration, but it could be possible to get out of the other room faster by holding dash at the start of the short-charge for as long as possible and then switch to tapping; this type of movement would be highly unintuitive and only save a few heat frames, so we don't try to model it.

- If the `from` node of the `leaveWithRunway` is different from the `to` node, then this represents that the runway in the other room is used starting at the end opposite the door. The way to calculate heat frames in this case is similar to above, except the full combined runway is used in every case, and it is only necessary to consider the one run from the other room to the current room (there being no need to start by doing a separate run in the opposite direction).


#### Example
```json
{
  "name": "Come In Shinecharging",
  "notable": false,
  "entranceCondition": {
    "comeInShinecharging": {
      "length": 5,
      "openEnd": 1
    }
  },
  "requires": [
    {"shinespark": {
      "frames": 40,
      "excessFrames": 3
    }}
  ]
}
```

### Come In Shinecharged

A `comeInShinecharged` entrance condition represents the need for Samus to run or jump into the room with a shinecharge with a certain amount of time remaining before it would expire. It has the following property:

- _framesRequired_: The number of frames that must be left on the shinespark charge when coming in. This must be a value between 1 and 179. Note that the shinecharge timer begins at 180 frames, and at least one frame must elapse between obtaining the shinecharge in the other room and crossing the door transition.

A strat with a `comeInShinecharged` condition should include a `shinespark` requirement in its `requires`.

A `comeInShinecharged` must match with either a `leaveShinecharged` condition or a `leaveWithRunway` condition on the other side of the door. 

- In order for `comeInShinecharged` to have a valid match with a `leaveShinecharged` condition, the `framesRequired` in the `comeInShinecharged` must be less than or equal to the `framesRemaining` of the `leaveShinecharged` condition. Aside from this, `comeInShinecharged` condition has no implicit requirements when matched with a `leaveShinecharged` conditions: all requirements in the other room are assumed to be explicitly accounted for in the strat with the `leaveShinecharged`. The frame counts in `comeInShinecharged` and `leaveShinecharged` are based on highly skilled (but humanly viable) play; leniency could be added by adjusting these counts (to increase `framesRequired` or decrease `framesRemaining`).

- A `comeInShinecharged` condition may also match with a `leaveWithRunway` condition. In this case it is assumed that the runway in the other room is used to obtain a shinecharge just before entering the transition, with 170 frames remaining. This comes with implicit requirements for actions to be performed in the previous room:
  - A `canShinecharge` requirement is included based on the runway length. This includes a `SpeedBooster` item requirement as well as a check that the effective runway length is enough that charging a shinespark is possible.
  - If the previous room is heated, then `heatFrames` are included based on the time spent running in that room. The minimally required heat frames are calculated the same way as in `comeInShinecharging`, except here with `comeInShinecharged` there is no second runway to combine with. An extra 10 heat frames are assumed for leaving the room after the shinecharge is obtained.
  - If the previous door environment is water, then `Gravity` is required.

#### Position and Momentum Details

A `comeInShinecharged` object does not provide any way to specify Samus' position or momentum through the door transition, but these details can affect the execution of the strat. As a way of normalizing the requirements, we make the following assumptions:

- For a horizontal door transition, the `framesRequired` (and any other strat requirements such as heat frames) should be based on an assumption that Samus enters the room either while running or spin-jumping just before the transition, with an unspecified amount of momentum. So the requirements should be based on the worst-case scenario, which in most cases means entering the room with no momentum. If entering with a spin jump is required, keep in mind that a `comeInShinecharged` condition does not require air physics in the previous room, so the jump may be low; if a spin jump with more momentum is required then the `comeInShinechargedJumping` condition should be used.

- For an vertical door transition, the `framesRemaining` (and other strat requirements) should be based on an assumption that Samus can enter through any horizontal position of the doorway (whichever is most favorable), in a jumping or falling pose, but with an unspecified amount of horizontal and vertical momentum. The requirements should be based on the worst-case momentum scenario, which generally means entering the room with no momentum, since any unwanted vertical momentum can be cancelled by releasing jump through the transition.

#### Example
```json
{
  "name": "Come In Shinecharged",
  "notable": false,
  "entranceCondition": {
    "comeInShinecharged": {
      "framesRequired": 65
    }
  },
  "requires": [
    {"shinespark": {
      "frames": 40,
      "excessFrames": 3
    }}
  ]
}
```

### Come In Shinecharged Jumping

A `comeInShinechargedJumping` entrance condition represents the need for Samus to jump into the room with a shinecharge with a certain amount of time remaining before it would expire. The jump must occur from an air environment on the other side of the door. It has the following property:

- _framesRequired_: The number of frames that must be left on the shinespark charge when coming in. This must be a value between 1 and 179. Note that the shinecharge timer begins at 180 frames, and at least one frame must elapse between obtaining the shinecharge in the other room and crossing the door transition.

A strat with a `comeInShinechargedJumping` condition should include a `shinespark` requirement in its `requires`.

The conditions for `comeInShinechargedJumping` are the same as for `comeInShinecharged`, with the added condition that the other side of the door must be an air environment.

### Come In With Spark

A `comeInWithSpark` entrance condition indicates that Samus must shinespark into the room.

The `comeInWithSpark` object has the following property:
- _position_: For a horizontal transition, if specified, this takes two possible values, "top" or "bottom". The value "top" represents that the strat requires sparking through the doorway high enough to clear a single-tile block level with the bottom of the doorway. The value "bottom" represents that the strat requires sparking through the doorway low enough to clear a single-tile block level with the top of the doorway. If unspecified, it is understood that sparking in any position will work.

The direction of the spark is assumed to be horizontal when sparking through horizontal door transitions, or vertical when sparking through vertical door transitions.

A strat with a `comeInWithSpark` condition should include a `shinespark` requirement in its `requires`.

A `comeInWithSpark` condition must match with either a `leaveWithSpark`, `leaveShinecharged`, or `leaveWithRunway` condition on the other side of the door:

- A match with `leaveWithSpark` is valid as long as the `position` properties are compatible. The `position` properties of a `leaveWithSpark` and `comeInWithSpark` are compatible if they are equal or if at least one of them are unspecified.
- A match with `leaveShinecharged` is always valid.
- A match with `leaveWithRunway` comes with the following implicit requirements (the same as for `comeInShinecharged`) for actions to be performed in the previous room:
  - A `canShinecharge` requirement is included based on the runway length. This includes a `SpeedBooster` item requirement as well as a check that the effective runway length is enough that charging a shinespark is possible.
  - If the previous room is heated, then `heatFrames` are included based on the time spent running in that room. The minimally required heat frames are calculated the same way as in `comeInShinecharging`, except here with `comeInShinecharged` there is no second runway to combine with.
  - If the previous door environment is water, then `Gravity` is required.

In all three cases, there is an implicit requirement of `canHorizontalShinespark` when sparking through a horizontal door.

#### Example
```json
{
  "name": "Come In With Spark",
  "notable": false,
  "entranceCondition": {
    "comeInWithSpark": {}
  },
  "requires": [
    {"shinespark": {
      "frames": 50,
      "excessFrames": 3
    }}
  ]
}
```

### Come In Stutter Shinecharging

A `comeInStutterShinecharging` entrance condition indicates that Samus must run into the room with SpeedBooster equipped, with a stutter immediately before the transition. This is used when entering a water room in order to obtain a shinecharge in a shorter amount of space than would otherwise be possible. It has the following property:
- _minTiles_: The minimum amount of effective runway tiles in other room needed for this strat.

A `comeInStutterShinecharging` condition must match with a `leaveWithRunway` condition on the other side of the door, which must have an "air" environment and an effective length of at least `minTiles`. A match comes with the following implicit requirements for actions to be performed in the previous room:
- The tech `canStutterWaterShineCharge`, which includes a requirement for the `SpeedBooster` item.
- If the previous room is heated, then heat frame requirements are included based on `minTiles`, in the same way as for a `comeInRunning` requirement.

#### Example
```json
{
  "name": "Come In Stutter Shinecharging",
  "notable": false,
  "entranceCondition": {
    "comeInStutterShinecharging": {
      "minTiles": 2
    }
  },
  "requires": [
    {"shinespark": {"frames": 60}}
  ]
}
```

### Come In With Bomb Boost

A `comeInWithBombBoost` entrance condition indicates that Samus must come into the room with a horizontal bomb boost. This object has no properties.

A `comeInWithBombBoost` condition must match with a `leaveWithRunway` condition on the other side of the door, which must have an "air" environment. A match comes with the following implicit requirements for actions to be performed in the previous room:
- A requirement `h_canBombThings`, to be able to use Bombs or a Power Bomb, as well as the tech `canBombHorizontally`.
- If the previous room is heated, a requirement of `{"heatFrames": 100}` is included, for positioning, placing the bomb, and waiting for it to detonate.

#### Example
```json
{
  "name": "Come In With Bomb Boost",
  "notable": false,
  "entranceCondition": {
    "comeInWithBombBoost": {}
  },
  "requires": []
}
```

### Come In With Door Stuck Setup

A `comeInWithDoorStuckSetup` entrance condition indicates that Samus must enter the room in a way that allows getting stuck in the door as it closes. This object has no properties.

A `comeInWithDoorStuckSetup` condition must match with a `leaveWithRunway` condition on the other side of the door. A match comes with the following implicit requirements for actions to be performed in the previous room:
- If the previous room is heated, a minimum requirement of `{"heatFrames": 100}` is included, for positioning and executing the door stuck setup; depending on the desired difficulty, this requirement can be increased to add leniency. Note that if X-Ray Scope is available (and it always should be for a strat with this condition, since the only known application of door stuck setups is for X-Ray climbing), then minimizing heat damage is made easier by using X-Ray to buffer the positioning near the door and to turn around in place.
- If the current room is heated and leniency is desired for failed attempts, then a minimum requirement of `{"heatFrames": 50}` should be included per potential failed attempt.
- If the previous room is to the left, then a tech requirement `canStationarySpinJump` is included.
- If the previous room is to the right, then a tech requirement `canRightSideDoorStuck` is included.
- If the previous door environment is water and is to the right, then a requirement `{"or": ["Gravity", "canRightSideDoorStuckFromWater"]}` is included.

#### Example
```json
{
  "name": "X-Ray Climb",
  "notable": false,
  "entranceCondition": {
    "comeInWithDoorStuckSetup": {}
  },
  "requires": [
    "canXRayClimb"
  ]
}
```

### Come In Speedballing

A `comeInSpeedballing` entrance condition indicates that Samus must enter the room either in a speedball from the previous room, or in a process of running, jumping, or falling into a speedball. It has the following property:

- _runway_: A [runway geometry](#runway-geometry) object describing the tiles available in the current room to complete the speedball. The end of this runway represents the point by which the speedball must be complete (i.e. when Samus must be morphed and on the ground with blue speed). A runway length of 0 would represent that the speedball must be completed before the transition.

It is assumed that the runway in the current room is level or sloping up; adjustments would be needed to handle a case where it sloped down.

Note that a `comeInSpeedballing` entrance condition can always be satisfied by coming into the room while already in a speedball. Therefore, a different entrance condition must be used if the strat specifically requires obtaining the speedball after entering the room (either by jumping through the transition, or by running through the transition and jumping afterward).

A `comeInSpeedballing` entrance condition must match with a `leaveWithRunway` condition on the other side of the door. A match with a `leaveWithRunway` comes with implicit requirements:
- A `canSpeedball` tech requirement.
- A `canShineCharge` based on the combined runway length, minus the amount of tiles needed to perform the jump into the speedball. The amount of tiles needed for the jump depends on the player's shortcharge ability (as well as ability to short-hop mockball): obtaining blue speed with lower run speed means less space needed to perform the jump. This can be approximated in a simple way with the following assumptions:
  - If the tech `canSlowShortCharge` is enabled, then 5 tiles are needed for the jump into the speedball.
  - Otherwise 14 tiles are needed.
- If the previous door environment is water, then `Gravity` is required.
- If the previous room is heated, then `heatFrames` are required based on the time needed. This can be calculated in the same way as for `comeInShinecharging`.

### Come In With Temporary Blue

A `comeInWithTemporaryBlue` entrance condition indicates that Samus must come in by jumping through this door with temporary blue. It has no properties.

The `comeInWithTemporaryBlue` object has the following property:
- _direction_: This takes two possible values "left" and "right", indicating the direction that Samus must be facing through the transition. It should be specified for all vertical transitions but not horizontal ones.

A `comeInWithTemporaryBlue` entrance condition must match with either a `leaveWithTemporaryBlue` or `leaveWithRunway` condition on the other side of the door. 

For a `comeInWithTemporaryBlue` to match with a `leaveWithTemporaryBlue`, their `direction` properties must be equal, if they are both specified and not "any".

A match with `leaveWithTemporaryBlue` comes only with an implicit requirement of the tech `canTemporaryBlue`.

A match with `leaveWithRunway` comes with implicit requirements for actions to be performed in the previous room:
  - The tech `canTemporaryBlue` is required.
  - A `canShinecharge` requirement is included based on the runway length. This includes a `SpeedBooster` item requirement as well as a check that the effective runway length is enough that gaining a shinecharge is possible.
  - If the previous room is heated, then `heatFrames` are included based on the time spent running in that room. The minimally required heat frames are calculated the same way as in `comeInShinecharging`, except here with `comeInShinecharged` there is no second runway to combine with. An extra 200 heat frames are assumed for gaining temporary blue and leaving the room after the shinecharge is obtained.
  - If the previous door environment is water, then `Gravity` is required.

### Come In With Stored Fall Speed

A `comeInWithStoredFallSpeed` entrance condition represents that Samus can enter through this door with stored fall speed. 

The `comeInWithStoredFallSpeed` object has a single property:
- _fallSpeedInTiles_: The number of tiles Samus would clip through by Moonfalling on top of a solid floor.

A `comeInWithStoredFallSpeed` entrance condition must match with a `leaveWithStoredFallSpeed` condition on the other side of the door.  The `comeInWithStoredFallSpeed` can lead to another `leaveWithStoredFallSpeed` so long as the stored speed is not lost.  For this to happen, both doors must be connected by one `Runway`, and Samus must not Crouch or become Knocked back.  A strat with a `comeInWithStoredFallSpeed` condition should only include requirements needed to position Samus for the clip.

*Note*: There is an implicit `canMoonfall` requirement on strats which have the `comeInWithStoredFallSpeed` condition as a means of using the stored fall speed.

#### Example
```json
{
  "name": "Stored Fall Speed Clip",
  "notable": false,
  "entranceCondtion": {
    "comeInWithStoredFallSpeed": {
      "fallSpeedInTiles": 1
    }
  },
  "requires": []
}
```
### Come In With R-Mode

A `comeInWithRMode` entrance condition indicates that Samus must obtain R-mode while coming through this door.

A `comeInWithRMode` object does not have any properties.

A `comeInWithRMode` entrance condition must match with a `leaveWithGModeSetup` entrance condition on the other side of the door. It comes with the following implicit requirements:
- The tech requirement `canEnterRMode`.
- The `XRayScope` item requirement.
- A requirement to have at least 1 reserve energy.
- A requirement to damage down to 0 energy, triggering reserves (causing the reserve energy to become zero and the regular energy to become what the reserve energy was).

```json
{
  "name": "Red Tower R-Mode Frozen Beetom X-Ray Climb",
  "notable": true,
  "entranceCondition": {
    "comeInWithRMode": {}
  },
  "requires": [
    "canWalljump",
    {"enemyDamage": {
      "enemy": "Beetom",
      "type": "contact",
      "hits": 1
    }},
    {"enemyDamage": {
      "enemy": "Ripper",
      "type": "contact",
      "hits": 1
    }},
    "canWallIceClip",
    "canXRayClimb"
  ]
}
```

### Come In With G-Mode

A `comeInWithGMode` entrance condition indicates that Samus must have or obtain G-mode (direct or indirect) while coming through this door. 

A `comeInWithGMode` object has the following properties:
* _mode_: Takes one of three possible values, "direct", "indirect", or "any", indicating whether Samus must enter in direct G-mode, indirect G-mode, or either. Direct G-mode is the state obtained when G-mode is first entered (i.e., the next room after the G-mode setup is performed), while indirect G-mode is the state after passing a door transition with G-mode (usually back into the room where the G-mode setup was performed).
* _morphed_: A boolean indicating whether Samus must enter the room in a morphed state. This can be satisfied by obtaining or already being in an artificially morphed state, or by having collected the Morph item.
* _mobility_: Takes one of three possible values, "mobile", "immobile", or "any", indicating whether or not Samus is
required to be mobile (or immobile) after entering the room. The default value is "any". When entering with indirect G-mode, Samus is always mobile. With direct G-mode, Samus can be mobile if she takes knockback through the door transition and the reserve energy is low enough (<= 4 energy) so that knockback frames do not expire until after reserves finish filling.

A `comeInWithGMode` entrance condition must match with either a `leaveWithGModeSetup` or `leaveWithGMode` entrance condition on the other side of the door:
- If `mode` is "direct", then it can only match with a `leaveWithGModeSetup`. If `mode` is "indirect", then it can only match with a `leaveWithGMode`.

When matching with a `leaveWithGModeSetup`, a `comeInWithGMode` has implicit requirements:
- The tech requirement `canEnterGMode`.
- The `XRayScope` item requirement.
- A requirement to have at least 1 reserve energy.
- A requirement to damage down to 0 energy, triggering reserves (causing the reserve energy to become zero and the regular energy to become what the reserve energy was).
- The requirement `{"or": ["Morph", "canArtificialMorph"]}`, if the `morphed` property of `comeInWithGMode` is true.
- The Toilet must not come between the room with the `leaveWithGModeSetup` and the `comeInWithGMode`, because obtaining direct G-mode in the Toilet is not possible, due to Samus not having an ability to use X-Ray while entering the room.
- Requirements for regaining mobility (there may be multiple options here, and they should be treated either as separate strats or the requirements should be joined as though with an `or`):
  - G-mode mobile: Samus uses the knockback from the previous room to retain mobility. Here the `knockback` property of the `leaveWithGModeSetup` must be true, and reserve energy is assumed to have been drained to at most 4 before doing the strat.
  - G-mode immobile: Samus uses knockback from a hit in the current room to regain mobility. The possibility of doing this is indicated by the presence of a strat with a `gModeRegainMobility` property with `from` and `to` node matching entrance/door node of the current strat (the `from` node of the strat with `comeInWithGMode`). Any requirements of the `gModeRegainMobility` strat should be included as requirements to execute the current strat; it is important that these requirements be applied *after* the requirement to damage down to 0 energy and trigger reserves. If there are multiple such `gModeRegainMobility` strats, then these should be treated as multiple options.

When matching with a `leaveWithGMode`, a `comeInWithGMode` has an implicit requirement in one scenario:
- If `morphed` property of `comeInWithGMode` is true but a matching `leaveWithGMode` has `morphed` false, then the `Morph` item is implicitly required.

__Example:__
```json
{
  "name": "G-Mode Morph",
  "entranceCondition": {
    "comeInWithGMode": {
      "mode": "any",
      "morphed": true
    }
  },
  "requires": []
}
```

### Come In With Wall Jump Below

A `comeInWithWallJumpBelow` entrance condition indicates that Samus must come up through this door with momentum by wall-jumping in the door frame below.

A `comeInWithWallJumpBelow` object has the following property:

- _minHeight_: Minimum height of door frame (tiles below the transition tiles) that will satisfy the condition.

A `comeInWithWallJumpBelow` entrance condition must match with a `leaveWithDoorFrameBelow` exit condition on the other side of the door:
- A match is valid provided the `height` property on the `leaveWithDoorFrameBelow` is at least as large as the `minHeight` property on the `comeInWithWallJumpBelow`.

A `comeInWithWallJumpBelow` implicitly includes a `canWalljump` tech requirement.

__Example:__
```json
{
  "name": "Cross Room Jump - Wall Jump",
  "entranceCondition": {
    "comeInWithWallJumpBelow": {
      "minHeight": 2
    }
  },
  "requires": [
    "canCrossRoomJumpIntoWater"
  ]
}
```

### Come In With Space Jump Below

A `comeInWithSpaceJumpBelow` entrance condition indicates that Samus must come up through this door with momentum by using Space Jump in the door frame below. It has no properties.

A `comeInWithSpaceJumpBelow` entrance condition must match with a `leaveWithDoorFrameBelow` exit condition on the other side of the door.

A `comeInWithSpaceJumpBelow` implicitly includes a `SpaceJump` item requirement. If the room below is heated, then a requirement of `{"heatFrames": 30}` is implicitly included.

__Example:__
```json
{
  "name": "Cross Room Jump - Space Jump",
  "entranceCondition": {
    "comeInWithSpaceJumpBelow": {}
  },
  "requires": [
    "canCrossRoomJumpIntoWater"
  ]
}
```

### Come In With Platform Below

A `comeInWithPlatformBelow` entrance condition indicates that Samus must come up through this door with momentum by jumping from a platform below, possibly with run speed. It has the following properties:

* _minHeight:_ Minimum height of the platform below that can satisfy this condition. It expresses that the platform must be positioned at least a certain distance below the door transition (in tiles, not including the transition tiles or platform tiles).
* _maxHeight:_ Maximum height of the platform below that can satisfy this condition. It expresses that the platform must be positioned at most a certain distance below the door transition.
* _maxLeftPosition:_ Maximum value of "leftPosition" of the platform below that can satisfy this condition. It expresses that the platform extends at least a certain distance to the left (in tiles, relative to the center of the door, with negative values indicating a position to the left of the door center).
* _minRightPosition:_ Minimum value of "rightPosition" of the platform below that can satisfy this condition. It expresses that the platform extends at least a certain distance to the right (in tiles, relative to the center of the door, with negative values indicating a position to the left of the door center).

A `comeInWithPlatformBelow` entrance condition must match with a `leaveWithPlatformBelow` exit condition on the other side of the door. A match is valid provided the `height`, `leftPosition`, and `rightPosition` properties on the `leaveWithDoorFrameBelow` satisfy all applicable constraints indicated by properties in the `comeInWithPlatformBelow`:
$$\text{minHeight} \leq \text{height} \leq \text{maxHeight}$$
$$\text{leftPosition} \leq \text{maxLeftPosition}$$
$$\text{rightPosition} \geq \text{minRightPosition}$$

A `comeInWithPlatformBelow` entrance condition has no implicit requirements.

__Example:__
```json
{
  "name": "Cross Room Jump - Standing Jump",
  "entranceCondition": {
    "comeInWithPlatformBelow": {
      "minHeight": 6,
      "maxHeight": 6,
      "maxLeftPosition": 1,
      "minRightPosition": 2
    }
  },
  "requires": [
    "canCrossRoomJumpIntoWater"
  ]
}
```


### Come In With Grapple Teleport

A `comeInWithGrappleTeleport` entrance condition represents that Samus must come into the room while grappling, teleporting Samus to a position in this room corresponding to the location of the (grapple) block in the other room.

A `comeInWithGrappleTeleport` object has the following property:

- _blockPositions_: A list of tile coordinates of (grapple) blocks that Samus could be grappled to while exiting the other room. Coordinates `[x, y]` are represented as tile counts with `[0, 0]` representing the top-left corner of the room.

A `comeInWithGrappleTeleport` comes with an implicit tech requirement `canGrappleTeleport`.

#### Example
```json
{
  "name": "Come in With Grapple Teleport",
  "requires": [],
  "exitCondition": {
    "leaveWithGrappleTeleport": {
      "blockPositions": [[5, 3]]
    }
  }
}
```

### Comes Through Toilet

Inside an `entranceCondition` object, a `comesThroughToilet` property indicates if the strat is applicable when the Toilet comes between this room and the other room (one with a matching `exitCondition`). This property should be specified on every strat having an `entranceCondition` through a vertical transition. It has three possible values:

- "yes": The strat is applicable only if the Toilet comes between this room and the other room.
- "no": The strat is applicable only if the Toilet *does not* come between this room and the other room.
- "any": The strat is applicable regardless of whether the Toilet comes between this room and the other room.

### Example
```json
{
  "name": "Cross Room Jump - Toilet HiJump Wall Jump",
  "entranceCondition": {
    "comeInWithWallJumpBelow": {
      "minHeight": 2
    },
    "comesThroughToilet": "yes"
  },
  "requires": [
    "canCrossRoomJumpIntoWater",
    "HiJump",
    "canPreciseWalljump",
    "canDownGrab"
  ]
}
```

## G-Mode Regain Mobility

A `gModeRegainMobility` property on a strat indicates that the strat allows Samus to regain mobility (by taking knockback from an enemy) after entering with G-mode immobile. In all strats with a `gModeRegainMobility` property, the `from` node of the strat must be the same as the `to` node and must be a door or entrance node. A strat with `gModeRegainMobility` should normally include an `enemyDamage` requirement in its `requires`.

A `gModeRegainMobility` object has no properties.

### Example
```json
{
  "name": "G-Mode Regain Mobility",
  "notable": false,
  "requires": [
    "h_ZebesIsAwake",
    {"enemyDamage": {
      "enemy": "Geemer (blue)",
      "type": "contact",
      "hits": 1
    }}
  ],
  "gModeRegainMobility": {}
}
```

## Bypasses Door Shell

A `bypassesDoorShell` property on a strat indicates that Samus can leave through the door transition in the `to` node
without first unlocking or opening the door. For this to be valid, the `to` node must have `"nodeType": "door"`. This can be used even for doors that are easy to open (e.g. blue doors), to support randomizers that may alter door colors. A strat with `"bypassesDoorShell": true` may also have an exit condition, but it is not required to have one.

A strat with `"bypassesDoorShell": true` has an implicit tech requirement of `canSkipDoorLock`.

### Example
```json
{
  "name": "Bypass Door Shell",
  "notable": false,
  "requires": [
    "canWallIceClip"
  ],
  "bypassesDoorShell": true
}
```

## Unlocks Doors

An `unlocksDoors` array lists possibilities of doors that can be unlocked as part of executing this strat. The objects in the array have the following properties:

- _nodeId_: The node ID of the door that can be unlocked. If unspecified, it is assumed to be the destination node of the strat.
- _types_: A list of door unlock types, among "missiles", "super", "powerbomb", and "gray":
    - "missiles": A door which can be opened using 5 Missiles, e.g. red doors.
    - "super": A door which can be opened using a single Super, e.g. red or green doors.
    - "powerbomb": A door which can be opened using a single Power Bomb, e.g. a yellow door.
    - "gray": A door with a room-specific condition to unlock it.
    - "ammo": A door which can be opened with ammo. This is a shorthand for ["missiles", "super", "powerbomb"].
- _requires_: A list of additional logical requirements which must be satisfied in order for the door to be unlocked using this strat. 
- _useImplicitRequires_: A boolean, true by default, indicating whether standard requirements should be implicitly appended to the `requires` in this object. This can be set this to false if the standard requirements are already accounted for in the strat `requires`, for example if the strat involves using a Power Bomb which would already unlock the door as a side effect, or if it uses a Super as a hero shot to open the door. If this property is set to true, the implicit standard requirements are based on the door type, as follows:
    - For "missiles", the implicit requirement is `{"ammo": {"type": "Missile", "count": 5}}`.
    - For "super", the implicit requirement is `{"ammo": {"type": "Super", "count": 1}}`.
    - For "powerbomb", the implicit requirement is `h_canUsePowerBombs`.
    - For "gray", there is no implicit requirement.
    
In general the `requires` in an `unlocksDoors` object do not need to be satisfied in order to perform the strat; if satisfied, they provide a way to unlock the door. However, if the strat has a [`doorUnlockedAtNode`](logicalRequirements.md#doorunlockedatnode-object) requirement and the door is locked, then these requirements become part of the strat requirements; this applies, in particular, if the strat has an exit condition, in which case there is an implicit `doorUnlockedAtNode` requirement on the destination door except if [`bypassesDoorShell`](strats.md#bypasses-door-shell) is set to `true`.

If an `unlocksDoors` property is not specified, then it is assumed to be an empty array. If a strat has any `doorUnlockedAtNode` requirements (including an implicit one based on having an exit condition without a `bypassesDoorShell`), then the `unlocksDoors` property should be specified explicitly and include items for each of the three possible types "missiles", "super", and "powerbomb" (or the catch-all "any") for each applicable node. The only exception is if the strat has no entrance condition then the starting node of the strat does not need to be included in the `unlocksDoors` property; in this case, the door could be unlocked immediately prior to the strat being executed (e.g. by an implicit unlock strat; see below), so generally it would not be necessary to describe how to unlock it as part of the strat. Where applicable, cases should be included for all three types of door unlock methods, "missiles", "super", and "powerbomb" (or using "ammo" as a catch-all), in order to support randomizers which may modify the door colors. The case of gray doors only needs to be included in places where the vanilla game has gray doors.

### Implicit Unlock Strats

By default every door node has an implicit strat from the node to itself, for unlocking the door in a standard way. In an unheated room, this implicit strat has an `unlocksDoors` of the following form:

```json
{
  "link": [1, 1],
  "name": "Unlock Door",
  "requires": [],
  "unlocksDoors": [{"types": ["ammo"], "requires": []}]
}
```

In a heated room, it instead has the form:

```json
{
  "link": [1, 1],
  "name": "Unlock Door",
  "requires": [],
  "unlocksDoors": [
    {"types": ["missiles"], "requires": [{"heatFrames": 50}]},
    {"types": ["super"], "requires": []},
    {"types": ["powerbomb"], "requires": [{"heatFrames": 110}]}
  ]
}
```

The implicit unlock strats can be disabled by setting the node property `useImplicitDoorUnlocks` to false.
