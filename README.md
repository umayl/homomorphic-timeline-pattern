Author: Oladapo Oyelaja  
Date: 05 September 2026  

# Homomorphic Timeline Pattern (HTP): Validating Interval-Scoped Streams in Functional Entity Component Systems

**Theoretical Continuity:** Direct discrete application framework of *[A Declarative Interval-Scoped Notation System](https://github.com/umayl/interval-scoped-notation)* (Oyelaja, 2026).

---

Traditionally, managing time-dependent game state tracking introduces a pervasive trade-off between human cognitive readability and hardware-level performance. This repository presents the computational implementation of the Homomorphic Timeline Pattern (HTP), establishing an architectural framework where the textual geometry of source code directly mirrors a timeline's chronological progression. 

Serving as a discrete computational projection of the abstract interval-scoped primitives established in Oyelaja (2026), HTP utilizes higher-order functional composition over a decoupled Entity Component System (ECS) database. We demonstrate how this pattern collapses traditional visual editor toolchains, executing complex temporal mechanics in $O(1)$ constant time relative to unreached events without dynamic serialization or runtime conditional branching loops.

---

## I. The Engineering Problem

When writing time-dependent gameplay systems on top of an Entity Component System (ECS), developers face a fundamental architectural conflict:

* **The Readability Deficit:** Because ECS decouples data from logic, a single multi-frame ability containing staggered hitboxes, status updates, and conditional triggers gets fractured across multiple stateless system files and separate timer components. This makes it incredibly difficult to audit the timing of a move from a single source file.
* **The Tooling Tax:** Traditional game engines bypass this by building massive, visual graphical editors (like Unreal GAS or Unity Timeline). While designer-friendly, these tools require specialized UI maintenance and serialize configurations into heavy data assets that an engine loop must actively parse and step-evaluate at runtime.

HTP completely collapses this toolchain. It establishes a fully homomorphic layout where standard text indentation and block nesting map directly to the timeline's chronological flow:
* The **vertical axis** dictates chronological frame progression ($t$).
* The **horizontal nesting** isolates local conditional scopes.

Upon initialization, the configuration tree evaluates once into a sequential array of stateless function chains. At runtime, the system runner advances a single array pointer (`nextEvent`), passing the ECS global state context through the pre-composed closures. This allows the timeline checker to run at $O(1)$ constant time, entirely eliminating runtime data-parsing loops and frame-by-frame conditional testing.

### I.A. High-Level Declarative Expressiveness
By shifting execution mechanics entirely behind higher-order wrappers, the front-end layout functions as a clean, human-readable specification. A non-technical game designer can read down the vertical axis of the script and audit the exact mechanical progression of a move as if reading a structured sentence, removing the traditional necessity for a standalone graphical interface without forcing the engine team to sacrifice performance budgets.

---

## II. Technical Architecture and Code Implementation

The following sections illustrate the end-to-end framework, proving how high-level spatial syntax cleanly maps to an optimized, low-level ECS memory pool placeholder.

### II.A. Script Configuration Example (execution_protocol.luau)
This script demonstrates how multi-frame sequences, branching conditions, and status changes are managed inside a single, human-readable file.

```luau
--!strict

const atFrame = require '../engine/timeline/atFrame'
const chain = require '../engine/timeline/chain'

const damage = require '../gameplay/skills/effects/damage'
const heal = require '../gameplay/skills/effects/heal'
const status = require '../gameplay/skills/effects/status'
const onCondition = require '../gameplay/skills/effects/onCondition'

const targets = require '../gameplay/skills/targets'
const damageConditions = require '../gameplay/damage/conditions'

const buffs = require '../content/buffs'
const debuffs = require '../content/debuffs'

return {
	name = 'Execution Protocol',
	description = 'Full execution protocol test',

	cooldown = 3,
	targetType = 'Enemy',

	timeline = {
		atFrame(
			10,
			damage(targets.target, {
				multiplier = 1.5,
				scalingStat = 'attack',
			})
		),

		atFrame(
			20,
			chain {
				damage(targets.targetEnemies, {
					multiplier = 0.75,
					scalingStat = 'attack',
				}),

				onCondition(
					damageConditions.wasCritical,
					damage(targets.targetLastDamaged, {
						multiplier = 2,
						scalingStat = 'attack',
						flags = {
							cannotCrit = true,
						},
					})
				),
			}
		),

		atFrame(
			40,
			chain {
				damage(targets.targetEnemies, {
					multiplier = 0.5,
					scalingStat = 'attack',
				}),

				status(targets.targetEnemies, debuffs.decreaseAttack, 2),
				status(targets.targetAllies, buffs.increaseDefense, 2),
			}
		),

		atFrame(50, heal(targets.targetSelf, 50)),

		atFrame(
			60,
			damage(targets.targetLastDamaged, {
				multiplier = 1.25,
				scalingStat = 'attack',
			})
		),

		atFrame(
			70,
			damage(targets.targetAll, {
				multiplier = 0.4,
				scalingStat = 'attack',
			})
		),

		atFrame(80, heal(targets.targetAllies, 25)),

		atFrame(90, status(targets.targetEnemies, debuffs.decreaseAttack, 3)),

		atFrame(
			100,
			chain {
				damage(targets.target, {
					multiplier = 3,
					scalingStat = 'attack',
				}),

				onCondition(
					damageConditions.wasCritical,
					chain {
						damage(targets.targetLastDamaged, {
							multiplier = 4,
							scalingStat = 'attack',
						}),

						heal(targets.targetSelf, 50),
					}
				),
			}
		),
	},
}
```
### II.B. Composition Wrappers (chain.luau, onCondition.luau)
Instead of running continuous evaluation conditions inside the main game tick loop, logic boundaries are isolated into higher-order wrappers that forward local evaluation contexts.

```luau
--!strict

-- Combines an array of actions into a single step-through execution path
const function chain<T>(actions: { TimelineAction<T> }): TimelineAction<T>
	return function(world, context, previousResult)
		local result = previousResult
    
		for _, action in actions do
			result = action(world, context, result)
		end
    
		return result
	end
end

-- Evaluates conditional states locally by tracking context variables
const function onCondition(condition: SkillCondition, action: SkillEffect): SkillEffect
	return function(world, context, previousResult)
		if not previousResult then
			return
		end

		const previousEvaluatedResult = context.evaluatedResult
		for _, evaluatedResult in previousResult.results do
			context.evaluatedResult = evaluatedResult

			if condition(evaluatedResult) then
				action(world, context, previousResult)
			end
		end

		context.evaluatedResult = previousEvaluatedResult

		return previousResult
	end
end
```

### II.C. Generic Backend Mutator (heal.luau)
Closures unpack structural parameters, interacting directly with a generic ECS `World` abstraction.

```luau
--!strict

type World = any -- Framework-agnostic ECS World placeholder
type Entity = number

const function heal(targets: SkillTargetSelector, amount: number): TimelineAction<SkillContext>
	return function(world: World, context, previousResult)
		for _, target in targets(world, context) do
			const health = world:get(target, spiritComponents.health)
			const maxHealth = world:get(target, spiritComponents.maxHealth)
			const alive = world:get(target, spiritComponents.alive)

			if alive() then
				health(math.min(maxHealth(), health() + amount))
			end
		end
    
		return previousResult
	end
end

return heal
```

### II.D. The O(1) Constant-Time Engine Runner (update.luau)
Because timeline events are pre-sorted chronologically when loaded, the runtime ticker uses a single index pointer (`timeline.nextEvent`). It skips unreached frames immediately, executing actions only when the current frame passes the target block.

```luau
--!strict

type World = any

const function update(world: World, deltaTime: number)
	for index = #activeTimelines, 1, -1 do
		const timeline = activeTimelines[index]
		timeline.accumulator += deltaTime

		while timeline.accumulator >= FRAME_DURATION do
			timeline.accumulator -= FRAME_DURATION
			timeline.frame += 1

			while true do
				const event = timeline.events[timeline.nextEvent]
				
				-- Early-out exit point
				if not event or event.frame > timeline.frame then
					break
				end

				event.action(world, timeline.context)
				timeline.nextEvent += 1
			end
		end
	end
end
```

---

## III. Domain-Agnostic Temporal Specifications

While active combat moves provide an intuitive visualization of HTP, the pattern is entirely domain-agnostic. Any system that transitions states over an axis of progression ($t$) can utilize the exact same engine runner logic.

### III.A. Passive / Status Decay Timelines
Instead of running a dedicated frame tick system for every status effect, status rules are mapped as finite or infinite geographical coordinate intervals.

```luau
--!strict

return {
	name = "Regeneration Buff",
	type = "PassiveModifier",
	
	timeline = {
		-- Ticks healing sequentially across distinct interval markers
		atFrame(60, heal(targets.self, 10)),
		atFrame(120, heal(targets.self, 10)),
		atFrame(180, chain {
			heal(targets.self, 10),
			clearStatus(targets.self, debuffs.poison) -- Drops out cleanly at terminal horizon
		})
	}
}
```

### III.B. Time-Sensitive World & Quest State Progression
HTP functions identically for large-scale, low-frequency systems like environmental cycles or macro mission states by altering the scale of the universal anchor step duration.

```luau
--!strict

return {
	name = "Ambush Quest Sequence",
	type = "StateTimeline",
	
	timeline = {
		atFrame(1, spawnProps(locations.townSquare, assets.barricades)),
		atFrame(1200, playSound(audio.distantScream)), -- Ticked cleanly without polling logic gates
		atFrame(3600, chain {
			spawnEnemies(locations.townSquare, enemyGroups.raiders),
			updateObjectiveText("Defend the town square")
		})
	}
}
```

---

## IV. Known Limitations & Architectural Trade-offs

* **Garbage Collection (GC) Allocations:** Instantiating anonymous function closures dynamically at runtime can introduce garbage collection pressure in high-throughput combat loops. For production use at high scale, flattening the timeline layout into a pre-compiled, static data-index registry or pooling closures is recommended to protect cache locality and memory budgets.
* **Rigid Sequential Stepping:** The engine runner loops linearly through the sequential event pointer. Dynamic runtime adjustments to active timelines (such as frame skipping, fast-forwarding, or interrupting the move early) require explicit pointer manipulation inside the active registry.
* **Tooltip Generation Complexity:** While writing configurations declaratively allows you to theoretically parse the syntax tree to generate player-facing UI text tooltips, doing so requires building a custom text-compiler or string-transformer to accurately translate functional closures into plain language descriptions.

---

# Acknowledgements & Conceptual Independence

The Homomorphic Timeline Pattern (HTP) and its underlying spatial-nesting grammar were conceptualized and developed independently of existing data-driven game engine toolchains.

While the backend architecture naturally builds upon established paradigms—specifically data-oriented Entity Component Systems and higher-order functional pipeline composition—the frontend layout was engineered from first principles as a direct structural alternative to non-textual graphical sequencers.

This framework was developed entirely without referencing or drawing from existing proprietary infrastructure, such as Unreal Engine's Gameplay Ability System (GAS) or Unity's Timeline serialization models. The choice to map lexical indentation constraints directly to chronological coordinate lifetimes was arrived at independently to resolve the human cognitive friction and data-representation desynchronization encountered within traditional procedural gameplay scripts.
