This is the last time anyone from these games see me (This was the original plan I had of making this N-A). Here's Naruto-Arena. You will probably need to edit the rank names in server.js. Host it on render. Missions and Edit missions aren't linked, you can link it yourself in the sidebar or just change the address to missions.html/editmission.html. To role yourself as Admin when you first use this, make your mongo cloud atlas, connect it in the .env, use npm start and role yourself as admin via mongodb compass on your computer.

Some design editing is needed since I merged C-A and N-A into one. Example files are included.


## Features

- Express server for API routes and game/session handling
- MongoDB-backed storage for users, matches, app state, news, and other game data
- JWT and cookie-based authentication
- WebSocket battle sessions for live matches
- Character roster and battle logic stored in local JavaScript data files
- Mission, profile, ladder, clan, and admin/editor pages
- Render deployment config included

## Stack

- Node.js
- Express
- MongoDB
- WebSockets via `ws`
- Vanilla HTML, CSS, and JavaScript

## Getting Started

### 1. Install dependencies

```bash
npm install
```

### 2. Create `.env`

Add the environment variables your local server needs:

```env
MONGODB_URI=your_mongodb_connection_string
MONGODB_DB=naruto-arena
MONGODB_USERS_COLLECTION=users
MONGODB_MATCHES_COLLECTION=matches
JWT_SECRET=replace_with_a_secure_secret
JWT_EXPIRY=7d
SESSION_COOKIE_NAME=naruto_session
SESSION_MAX_AGE_MS=604800000
CORS_ORIGIN=http://localhost:4000
```

Optional variables used by the server:

```env
MONGODB_APP_STATE_COLLECTION=app_state
MONGODB_NEWS_POSTS_COLLECTION=news_posts
ENABLE_BATTLE_BOTS=true
ALLOW_INSECURE_HTTP=true
HTTPS_KEY_PATH=
HTTPS_CERT_PATH=
PORT=4000
```

### 3. Start the app

```bash
npm start
```

Default local port:

```text
http://localhost:4000
```

## Scripts

```bash
npm start   # run the server
npm test    # run Node test runner
```

## Deployment

This repo includes a [`render.yaml`](./render.yaml) for Render. The current config expects:

- `MONGODB_URI`
- `JWT_SECRET`
- `CORS_ORIGIN`

# BattleLogic Effects Dictionary

This file documents the `effects` entries used by `characters.js` and handled by `battleLogic.js`.

Each skill can include:

```js
effects: [
  {
    type: "damage",
    amount: 20,
    scope: "target"
  }
]
```

## Common Effect Fields

These fields work on many effect types.

- `type`: Required. Must match one of the effect types below.
- `scope`: Optional. If omitted, the effect applies to the selected target or targets.
- `amount`: A number used by damage, healing, chakra, cooldown, and duration effects.
- `condition`: Optional. Makes the effect happen only when a condition matches.
- `metadata`: Optional. Extra flags used by damage, statuses, tooltips, and custom behavior.
- `chance`: Optional percent chance from `0` to `100`. If present, the effect can fail.
- `rollPerRecipient`: Optional boolean. If true, chance is rolled separately for each recipient.
- `activationChancePercent`: Optional percent chance from `0` to `100`.

## Scope Values

These values are supported inside effects:

- `target`: The selected target or reflected selected targets.
- `self`: The character using the skill.
- `all-allies`: The user's whole team.
- `all-enemy`: The enemy team.
- `all-units`: Both teams.
- `random-enemy`: One random enemy.
- `random-other-enemy`: One random enemy that was not the selected target.
- `other-enemies`: Every enemy except the selected target.

If `scope` is omitted, BattleLogic behaves like `target`.

## Conditions

Common `condition` fields:

```js
condition: {
  scope: "target",
  statusId: "example_status",
  consumeOnMatch: true
}
```

- `scope`: `self` or `target`. Defaults to `self` in most condition checks.
- `statusId`: Requires this status on the scoped unit.
- `statusIdsAny`: Requires at least one status from this list.
- `missingStatusId`: Requires this status to be missing.
- `targetRelation`: `self`, `ally`, or `enemy`.
- `consumeOnMatch`: Removes `statusId` after the condition succeeds.
- `conditionalAmount`: For damage-style effects, replaces `amount` when the condition matches.
- `statusMetadataAtLeast`: Checks a metadata number on a status.
- `sourceCurrentHpAtMost`: Requires the scoped unit's HP to be at or below this value.
- `sourceCurrentHpAtLeast`: Requires the scoped unit's HP to be at or above this value.
- `sourceSkillUsesAtLeast`: For `self`; checks how many times a skill has been used.
- `sourceSkillUsesAtMost`: For `self`; checks max uses of a skill.

## Metadata Basics

Common `metadata` fields:

- `harmful`: Marks a status or effect as harmful.
- `tooltipText`: Text shown on status/tooltips.
- `afflictionDamage`: Treats damage as affliction.
- `cannotBeEvaded`: Bypasses evasion.
- `ignoreDamageReduction`: Damage ignores damage reduction.
- `ignoreDestructibleDefense`: Damage bypasses destructible defense.
- `ignoreHelpfulInvulnerability`: Helpful effects can bypass helpful-skill immunity.
- `skillClasses`: Overrides the classes used for triggered damage.
- `randomScopeGroupKey`: Reuses the same random target for multiple effects with the same key.
- `onSuccessfulDamageApplyStatusToTarget`: Applies a status if damage was actually dealt.

Example:

```js
{
  type: "damage",
  amount: 20,
  metadata: {
    cannotBeEvaded: true,
    ignoreDamageReduction: true
  }
}
```

## Effect Types

### `damage`

Deals normal damage.

```js
{
  type: "damage",
  amount: 25,
  scope: "target"
}
```

Useful fields:

- `amount`: Damage amount.
- `scope`: Who receives the damage.
- `condition`: Optional condition.
- `metadata.afflictionDamage`: Makes it affliction damage.
- `metadata.ignoreDamageReduction`: Ignores damage reduction.
- `metadata.ignoreDestructibleDefense`: Ignores destructible defense.
- `metadata.cannotBeEvaded`: Cannot be evaded.
- `metadata.amountFromSourceMissingHp`: Adds the user's missing HP to damage.
- `metadata.amountFromSourceCurrentHp`: Adds the user's current HP to damage.
- `metadata.amountFromTargetCurrentHp`: Adds the target's current HP to damage.
- `metadata.amountFromTargetMissingHp`: Adds the target's missing HP to damage.

### `health_steal_damage`

Deals damage and heals the skill user by the damage dealt.

```js
{
  type: "health_steal_damage",
  amount: 20,
  scope: "target"
}
```

Useful fields are the same as `damage`.

### `apply_status`

Applies a status to the target, self, allies, or enemies.

```js
{
  type: "apply_status",
  statusId: "example_status",
  duration: 2,
  scope: "target",
  metadata: {
    harmful: true,
    tooltipText: "This character is affected."
  }
}
```

Required fields:

- `statusId`: Unique status id.
- `duration`: Number of turns.

Useful fields:

- `scope`: Who gets the status.
- `metadata.tooltipText`: Text shown for the status.
- `metadata.harmful`: Marks it as harmful.
- `metadata.triggerOnApply`: Makes turn-trigger statuses fire immediately.
- `metadata.turnEndTrigger`: Common values include `owner_turn` and `source_turn`.
- `metadata.turnEndDamage`: Damage dealt when the status triggers.
- `metadata.turnEndHeal`: Healing done when the status triggers.
- `metadata.destructibleDefense`: Adds destructible defense.
- `metadata.damageReductionFlat`: Flat damage reduction.
- `metadata.damageReductionPercent`: Percent damage reduction.
- `metadata.invulnerable`: Invulnerable to skills.
- `metadata.invulnerableToHelpfulSkills`: Blocks helpful skills.
- `metadata.invulnerableToHarmfulEffects`: Blocks harmful effects.
- `metadata.cannotUseSkills`: Full stun.
- `metadata.cannotUseHarmfulSkills`: Stuns harmful skills.
- `metadata.cannotUseNonMentalSkills`: Stuns non-mental skills.
- `metadata.cannotUseSkillClasses`: Stuns listed classes.
- `metadata.cannotUseSkillIndices`: Stuns listed skill slots.
- `metadata.cannotReduceDamage`: Stops damage reduction.
- `metadata.cannotBecomeInvulnerable`: Stops invulnerability.
- `metadata.skillReplacements`: Replaces skill ids while status is active.
- `metadata.facePictureOverride`: Changes the unit face while status is active.
- `metadata.onExpireEffects`: Effects to run when triggered or expired.
- `metadata.onExpireEffectsToEnemiesOfSource`: Effects to run on enemies of the source.

### `heal`

Heals a unit.

```js
{
  type: "heal",
  amount: 15,
  scope: "self"
}
```

Useful fields:

- `amount`: Healing amount.
- `scope`: Who receives healing.
- `metadata.amountFromSourceMissingHp`: Adds the user's missing HP.
- `metadata.amountFromSourceCurrentHp`: Adds the user's current HP.
- `metadata.amountFromTargetCurrentHp`: Adds the target's current HP.
- `metadata.amountFromTargetMissingHp`: Adds the target's missing HP.

### `HealthLoss`

Direct HP loss. This is not normal damage and is usually used for recoil or self-loss.

```js
{
  type: "HealthLoss",
  amount: 15,
  scope: "self"
}
```

Useful fields:

- `amount`: HP lost.
- `scope`: Who loses HP.

### `execute_below_hp`

Kills the recipient if their current HP is at or below `threshold`.

```js
{
  type: "execute_below_hp",
  threshold: 25,
  scope: "target"
}
```

Useful fields:

- `threshold`: HP cutoff for execution.
- `scope`: Who can be executed.

### `set_hp_from_snapshot`

Sets HP from a stored snapshot key. Your current code stores `ownerTurnEndHp` at turn end.

```js
{
  type: "set_hp_from_snapshot",
  scope: "self",
  snapshotKey: "ownerTurnEndHp"
}
```

Useful fields:

- `snapshotKey`: The saved HP snapshot name.
- `scope`: Who has HP restored to the snapshot.

### `destroy_destructible_defense`

Removes all destructible defense from the recipient.

```js
{
  type: "destroy_destructible_defense",
  scope: "target"
}
```

### `cleanse_harmful`

Removes harmful statuses from the recipient.

```js
{
  type: "cleanse_harmful",
  count: 1,
  scope: "self"
}
```

Useful fields:

- `count`: Number of harmful statuses to remove. If omitted or `0`, behavior depends on the cleanse helper.
- `scope`: Who is cleansed.

### `cleanse_statuses`

Removes statuses with filters.

```js
{
  type: "cleanse_statuses",
  scope: "self",
  count: 1,
  harmfulOnly: true,
  sourceRelation: "enemy"
}
```

Useful fields:

- `count`: Max statuses removed. `0` or omitted means all matching statuses.
- `scope`: Whose statuses are removed.
- `sourceRelation`: `any`, `enemy`, or `ally`.
- `statusId`: Only remove one exact status id.
- `statusIdsAny`: Remove any status id in the list.
- `metadataAny`: Remove statuses that have any listed metadata key.
- `harmfulOnly`: Only remove statuses with `metadata.harmful`.
- `sourceSkillClassesAny`: Only remove statuses stamped by skills with matching classes.

### `extend_status`

Adds turns to an existing status.

```js
{
  type: "extend_status",
  targetStatusId: "example_status",
  amount: 1,
  scope: "target"
}
```

Useful fields:

- `targetStatusId`: Status to extend.
- `amount`: Number of turns to add.
- `scope`: Whose status is extended.
- `condition`: Optional condition.

### `remove_random_chakra`

Removes random chakra from the recipient's chakra pool.

```js
{
  type: "remove_random_chakra",
  amount: 1,
  scope: "target"
}
```

Useful fields:

- `amount`: Number of chakra removed.
- `scope`: Which player's chakra pool is hit.

### `gain_chakra`

Gives chakra to the skill user.

```js
{
  type: "gain_chakra",
  chakraType: "random",
  amount: 1
}
```

Useful fields:

- `chakraType`: `taijutsu`, `ninjutsu`, `bloodline`, `genjutsu`, or `random`.
- `amount`: Number gained.

### `gain_chakra_by_last_skill`

Looks for a status on the skill user, reads `metadata.lastSkillId`, maps that skill id to a chakra type, and gives chakra.

```js
{
  type: "gain_chakra_by_last_skill",
  statusId: "konohamaru_last_skill_used",
  amount: 1,
  map: {
    "konohamaru-sexy-jutsu": "genjutsu",
    "konohamaru-shuriken": "taijutsu"
  }
}
```

Useful fields:

- `statusId`: Status that stores `metadata.lastSkillId`.
- `amount`: Number gained.
- `map`: Object from skill id to chakra type.

### `spend_all_chakra`

Clears all chakra from the skill user.

```js
{
  type: "spend_all_chakra"
}
```

### `drain_chakra_non_bloodline_from_target_to_self`

Randomly drains taijutsu, ninjutsu, or genjutsu from the target and gives it to the skill user.

```js
{
  type: "drain_chakra_non_bloodline_from_target_to_self",
  amount: 1,
  scope: "target"
}
```

Useful fields:

- `amount`: Max chakra drained.
- `scope`: Usually `target`.

### `drain_chakra_specific_from_target_to_self`

Drains a specific chakra type from the target and gives it to the skill user.

```js
{
  type: "drain_chakra_specific_from_target_to_self",
  chakraType: "ninjutsu",
  amount: 1,
  scope: "target"
}
```

Useful fields:

- `chakraType`: `taijutsu`, `ninjutsu`, `bloodline`, or `genjutsu`.
- `amount`: Max chakra drained.
- `scope`: Usually `target`.

### `modify_cooldowns`

Changes cooldowns on skill ids.

```js
{
  type: "modify_cooldowns",
  scope: "target",
  amount: 1,
  operation: "add",
  includeAllCharacterSkills: true
}
```

Useful fields:

- `amount`: Cooldown amount.
- `operation`: `add`, `set`, `max`, or `min`. Omitted means `add`.
- `skillIds`: Optional explicit skill ids to change.
- `includeAllCharacterSkills`: If true, includes every skill on that character.
- `scope`: Whose cooldowns are changed.

### `trigger_status_effects`

Finds matching active statuses and triggers their stored effects, then usually consumes the matched status.

```js
{
  type: "trigger_status_effects",
  scope: "target",
  statusId: "example_status",
  consumeMatchedStatus: true
}
```

Useful fields:

- `statusId`: One status id to trigger.
- `statusIdsAny`: Any status id in this list.
- `consumeMatchedStatus`: Defaults to true.

The matched status can store:

```js
metadata: {
  onExpireEffects: [
    {
      type: "damage",
      amount: 20
    }
  ],
  onExpireEffectsToEnemiesOfSource: [
    {
      type: "damage",
      amount: 10
    }
  ]
}
```

Triggered status effects currently support:

- `damage`
- `health_steal_damage`
- `apply_status`
- `cleanse_statuses`

### `swap_positions`

Swaps the user's board position with the recipient.

```js
{
  type: "swap_positions",
  scope: "target"
}
```

## Skill-Level Fields That Work With Effects

These are not effect types, but they are important when coding skills:

```js
{
  id: "example-character-example-skill",
  name: "Example Skill",
  skillimage: "https://example.com/image.png",
  skilldescription: "Deal 20 damage to one enemy.",
  energy: ["Taijutsu", "Random"],
  target: "single-enemy",
  damage: 20,
  cooldown: 1,
  classes: ["Physical", "Melee", "Instant"],
  effects: []
}
```

Supported skill targets seen in the current roster:

- `self`
- `single-enemy`
- `all-enemy`
- `single-ally`
- `all-allies`
- `self-or-single-ally`
- empty string for non-target/helper skills

Supported energy entries seen in the current roster:

- `Taijutsu`
- `Ninjutsu`
- `Bloodline`
- `Genjutsu`
- `Random`
- `none`
- empty array `[]`

Supported classes seen in the current roster:

- `Action`
- `Affliction`
- `Affliction*`
- `Chakra`
- `Control`
- `Instant`
- `Instant*`
- `Invisible`
- `Melee`
- `Mental`
- `Physical`
- `Ranged`
- `Unique`


## Notes
