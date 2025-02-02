---
title: On Player Leave
category: On Event Systems
mentions:
    - BedrockCommands
    - zheaEvyline
nav_order: 3
description: This system will run your desired commands on the event that a player leaves the world.
---

## Introduction

[Sourced by the Bedrock Commands Community Discord](https://discord.gg/SYstTYx5G5)

This system will run your desired commands on the event that a player leaves the world.

> **Note:** You cannot execute commands on *players* that leave using target selectors. However, you can use the [On Player Join](/commands/on-player-join) system to execute commands when they rejoin.

## Setup

*To be typed in chat:*

`/scoreboard objectives add total dummy`

If you are working with functions and prefer to have the objective added automatically when the world initializes, follow the process outlined in [On First World Load.](/commands/on-first-world-load)

## System

<CodeHeader>BP/functions/events/player/on_leave.mcfunction</CodeHeader>

```yaml
## Entity Counter
### Reset current player count
scoreboard players reset NewPlayerCount total
### Get current player count 
execute as @a run scoreboard players add NewPlayerCount total 1

## Get Difference (Current - Previous)
scoreboard players operation NewPlayerCount total -= PlayerCount total

## Your Commands Here (Example)
### Message if there is a difference of -1 or less
execute if score NewPlayerCount total matches ..-1 run say One or more players have left the world

## Entity Counter
### Reset current player count
scoreboard players reset PlayerCount total
### Get current player count (to check the difference in the next game tick)
execute as @a run scoreboard players add PlayerCount total 1
```

![Chain of 6 Command Blocks](/assets/images/commands/commandBlockChain/6.png)

Here, we have used a `/say` command as an example, but you can use any command you prefer and as many as you need.

Just make sure to follow the given order and properly apply the `/execute if score` condition as shown for your desired commands.

## Explanation

- **`NewPlayerCount`**: The total number of players in the world at the start of the command loop (in the current game tick).
- **`PlayerCount`**: The total number of players in the world at the end of the command loop (in the current game tick).

Since `PlayerCount` is updated at the end of the command loop, it can be used at the start of the next game tick to compare with `NewPlayerCount`.

The count is obtained using the [Entity Counter](/commands/entity-counter) system. It may be beneficial to refer to that page for better understanding. You will notice that we have used the objective name `total` instead of `count` to prevent conflicts between systems.

By subtracting `PlayerCount total` from `NewPlayerCount total`, we can determine if the player count has:
- Decreased (`..-1`)
- Increased (`1..`)
- Remained unchanged (`0`)

If it has decreased, it means one or more players have left the game.
Using this, we can execute commands when `NewPlayerCount` is `-1` or lower.
- Example: If there were 10 players and one leaves:
    - `NewPlayerCount - PlayerCount = 9 - 10 = -1`
    - We detect this using `..-1`

- `NewPlayerCount` is obtained first, subtraction is performed next, commands are executed based on the result, and finally, `PlayerCount` is updated for the next game tick.

:::tip
All commands in a command-block chain or function will execute sequentially but within the same game tick, regardless of the number of commands involved. This system works because commands execute at the end of a game tick after all events (such as player logins, logouts, deaths, etc.) occur.

![Game Tick](/assets/images/commands/gametick.png)
:::

## Tick JSON

If you are using functions instead of command blocks, the `on_leave` function must be added to `tick.json` to ensure continuous execution. Multiple files can be added to `tick.json` by placing a comma after each string. Refer to the [Functions](/commands/mcfunctions#tick-json) documentation for more details.

<CodeHeader>BP/functions/tick.json</CodeHeader>
```json
{
  "values": [
    "events/player/on_leave"
  ]
}
```

If using functions, your pack folder structure should be as follows:

<FolderView
	:paths="[
    'BP',
    'BP/functions',
    'BP/pack_icon.png',
    'BP/manifest.json',
    'BP/functions/events',
    'BP/functions/events/player',
    'BP/functions/events/player/on_leave.mcfunction',
    'BP/functions/tick.json'
]"
></FolderView>

:::info **NOTE:**

The scoreboard names (e.g., `total`) might be used by other people. To minimize conflicts, you can append an underscore and a randomly generated set of characters. A similar technique can be used for `.mcfunction` filenames.  
Examples:
- `total_0fe678`
- `on_leave_0fe678.mcfunction`

:::
