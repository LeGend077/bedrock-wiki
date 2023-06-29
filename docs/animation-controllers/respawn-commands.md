---
title: Respawn Commands
tags:
    - recipe
mentions:
    - SirLich
    - solvedDev
    - Joelant05
    - BlueFrog130
    - SmokeyStack
    - MedicalJewel105
    - cda94581
---

<BButton color="blue" link="animation-controllers-intro">Learn more about Animation Controllers</BButton>

This animation controller can be used to run commands, such as re-adding potion effects or giving items when the player respawns.

Simply add the animation controller to the `player.json`, and

<CodeHeader>BP/animation_controllers/respawn.ac.json</CodeHeader>

```json
{
	"format_version": "1.10.0",
	"animation_controllers": {
		"controller.animation.death": {
			"initial_state": "initialization",
			"states": {
				"initialization": {
					"transitions": [
						{
							"has_died": "!query.is_alive"
						}
					],
					"on_exit": [
						"variable.delay = 0.2 + query.life_time;",
						"/<death command or animation>"
					]
				},
				"has_died": {
					"on_exit": ["/<Respawn command or animation>"],
					"transitions": [
						{
							"initialization": "query.is_alive && (query.life_time >= variable.delay)"
						}
					]
				}
			}
		}
	}
}
```
