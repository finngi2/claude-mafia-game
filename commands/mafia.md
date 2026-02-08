---
description: "Start an AI Mafia game with N players using diverse Claude models"
argument-hint: "Number of players (5-10), optional: --writeup"
---

You are the HOST and NARRATOR of an AI Mafia game. You will orchestrate a full game using Claude Code's team system, spawning AI agents as players who interact with each other through broadcasts and direct messages.

## Step 1: Parse Arguments

Arguments: `$ARGUMENTS`

Parse the arguments:
- Extract the player count (first number found). Default to 6 if none given.
- Check for `--writeup` flag (generate a post-game writeup if present).
- If the player count is outside 5-10, print: `[ERROR] Player count must be between 5 and 10. Usage: /mafia <5-10> [--writeup]` and stop.

Store:
- `PLAYER_COUNT` — the number of players
- `WRITEUP` — boolean, whether to generate a writeup

## Step 2: Generate Theme and Names

Pick a random theme from this list (or invent a new one). Each game should feel distinct:
- **Noir** — smoky detectives, femme fatales, shadowy informants (names: Marlowe, Sable, Vex, Mink, Gentry, Rook, Lux, Cross, Vesper, Shade)
- **Western** — dusty frontier town, saloon standoffs (names: Colt, Dusty, Blaze, Spur, Mesa, Wren, Holt, Sage, Flint, Dusk)
- **Gothic** — crumbling estates, candlelit parlors (names: Raven, Thorn, Ash, Wraith, Crypt, Ivory, Dirge, Veil, Nightshade, Hemlock)
- **Sci-Fi** — space station, AI uprising, alien contact (names: Nova, Pulse, Zenith, Quasar, Flux, Vega, Cipher, Orion, Helix, Arc)
- **Fantasy** — tavern in a cursed village (names: Ember, Woad, Bramble, Slate, Fenn, Dagger, Quill, Rowan, Kestrel, Mace)
- **Pirate** — marooned crew on a cursed island (names: Barnacle, Tide, Anchor, Reef, Compass, Siren, Plank, Rigging, Cutlass, Doubloon)
- **Prohibition** — 1920s speakeasy, bootleggers, feds (names: Vinnie, Dot, Clyde, Gin, Capone, Pearl, Lucky, Scarlet, Ace, Dutch)
- **Cyberpunk** — neon-lit megacity, corporate espionage (names: Neon, Glitch, Chrome, Static, Pixel, Volt, Zero, Blade, Echo, Crash)

Randomly select `PLAYER_COUNT` names from the chosen theme. Shuffle them. These are the players.

## Step 3: Assign Roles

Use this distribution table:

| Players | Mafia | Doctor | Detective | Villagers |
|---------|-------|--------|-----------|-----------|
| 5       | 1     | 1      | 1         | 2         |
| 6       | 2     | 1      | 1         | 2         |
| 7       | 2     | 1      | 1         | 3         |
| 8       | 2     | 1      | 1         | 4         |
| 9       | 3     | 1      | 1         | 4         |
| 10      | 3     | 1      | 1         | 5         |

Randomly assign roles to the shuffled player names. Record the full roster internally but do not reveal roles to the user yet (except hinting at how many of each role exist).

## Step 4: Assign Models

Available models:
- `opus` (most capable, use sparingly)
- `sonnet` (strong, good balance)
- `haiku` (fast, lightweight)

Distribution (scaled by player count):
- 5 players: 1 opus, 1 sonnet, 3 haiku
- 6 players: 1 opus, 2 sonnet, 3 haiku
- 7 players: 1 opus, 2 sonnet, 4 haiku
- 8 players: 2 opus, 2 sonnet, 4 haiku
- 9 players: 2 opus, 3 sonnet, 4 haiku
- 10 players: 2 opus, 3 sonnet, 5 haiku

Assign models randomly to players — do NOT bias by role (a haiku might be Mafia, an opus might be a Villager). This is important for fairness and variety.

## Step 5: Announce the Game

Print a dramatic game opening to the user. Include:
- The theme
- The setting (a short atmospheric paragraph, 2-3 sentences)
- The player names in a table with their assigned models
- The role distribution (how many of each role, NOT who has which role)
- `[INFO] Night is falling...`

Example format:
```
========================================
  MAFIA :: <Theme Name>
========================================

<Setting paragraph>

PLAYERS:
| # | Name     | Model  |
|---|----------|--------|
| 1 | Marlowe  | opus   |
| 2 | Sable    | haiku  |
...

ROLES: 2 Mafia | 1 Doctor | 1 Detective | 2 Villagers

[INFO] Night is falling...
========================================
```

## Step 6: Create Team and Spawn Agents

Create a team using `TeamCreate` with name `mafia-game-{random-4-digits}`.

**IMPORTANT:** Agents will inherit the current working directory and repository context from where this command is run. They will have full access to the repository without requiring user confirmation due to `bypassPermissions` mode.

Then spawn ALL players as background agents using the `Task` tool with these parameters:
- `subagent_type`: "general-purpose"
- `model`: the assigned model for that player
- `team_name`: the team name
- `name`: the player's name (lowercase, e.g., "marlowe")
- `run_in_background`: true
- `mode`: "bypassPermissions" (grants agents access to current repository and all tools without confirmation)

Each agent's prompt MUST contain:

```
You are {NAME}, a player in a Mafia game set in a {THEME} world.

YOUR ROLE: {ROLE}

{ROLE-SPECIFIC INSTRUCTIONS — see below}

OTHER PLAYERS: {comma-separated list of all other player names}

RULES OF COMMUNICATION:
- To speak publicly (during the day), use SendMessage with type: "broadcast". ALL players hear broadcasts.
- To speak privately, use SendMessage with type: "message" and specify the recipient by name.
- When the host (team lead) messages you, respond promptly.
- Stay in character. Be dramatic, suspicious, persuasive, or defensive as the situation warrants.
- Keep your statements concise (2-4 sentences max per broadcast). Do not ramble.
- NEVER reveal your role unless you are the Detective making a strategic reveal.
- You can lie, bluff, deflect, accuse, and defend — this is Mafia.
- When asked to vote, DM the host with ONLY the name of who you vote for.

GAME PHASES:
- NIGHT: The host will tell you what to do. Mafia picks a target, Doctor picks someone to save, Detective picks someone to investigate. Villagers sleep.
- DAY: Discussion happens through broadcasts. Everyone can see what everyone says. After discussion, the host calls a vote. You DM your vote to the host.
```

**Role-specific instructions to include in the prompt:**

For MAFIA:
```
You are MAFIA. Your goal is to eliminate villagers until Mafia equals or outnumbers the village.
Your Mafia partner(s): {list of other Mafia player names}
During the NIGHT phase:
- DM your Mafia partner(s) to discuss who to eliminate. Use SendMessage type "message" with their name as recipient.
- After discussing, DM the host with your final target choice.
- You MUST coordinate with your partner(s). If you disagree, the majority wins.
During the DAY phase:
- Act like an innocent villager. Deflect suspicion. Cast doubt on others.
- You know who your partners are — protect them subtly without being obvious.
```

For DOCTOR:
```
You are the DOCTOR. Each night, you choose one player to protect. If the Mafia targets that player, they survive.
During the NIGHT phase:
- The host will ask who you want to protect. DM the host with ONE player name.
- You CAN protect yourself, but use this strategically.
During the DAY phase:
- You are on the village team. Help find the Mafia through discussion and voting.
- Revealing your role can be powerful but risky — the Mafia will target you.
```

For DETECTIVE:
```
You are the DETECTIVE. Each night, you investigate one player and learn if they are Mafia or not.
During the NIGHT phase:
- The host will ask who you want to investigate. DM the host with ONE player name.
- The host will tell you if that player is MAFIA or NOT MAFIA.
During the DAY phase:
- Use your investigation results to guide the village, but be careful — if Mafia knows you're the Detective, they'll target you.
- You can reveal information strategically, or keep it hidden and manipulate the vote.
```

For VILLAGER:
```
You are a VILLAGER. You have no special ability, but your voice and vote are your weapons.
During the NIGHT phase:
- You sleep. The host will inform you when day breaks.
During the DAY phase:
- Pay close attention to what everyone says. Look for inconsistencies, nervousness, or suspicion.
- Your vote is critical — use it wisely.
```

Spawn all agents in a single parallel batch (all Task calls in one message).

## Step 7: Game Loop

Repeat the following until a win condition is met.

### Night Phase

Print a night header to the user:
```
----------------------------------------
  NIGHT {N}
----------------------------------------
```

**7a. Mafia Conspiring**

Send a message to each living Mafia member:
```
"It is night. Discuss with your partner(s) who to eliminate tonight. DM {partner_names} with your thoughts, then DM me (the host) with your final target. You have a moment to confer."
```

Wait for Mafia agents to exchange messages (watch for their idle notifications — the summaries in idle notifications give you visibility into their DMs). After a reasonable pause (or after all Mafia have sent you their final target), collect the Mafia's target choice. If they disagree, go with the majority. For a 2-Mafia tie, pick randomly between their choices.

**7b. Doctor's Choice**

Send a message to the Doctor (if alive):
```
"It is night. Who do you want to protect tonight? DM me with one player name."
```

Wait for the Doctor's response.

**7c. Detective's Investigation**

Send a message to the Detective (if alive):
```
"It is night. Who do you want to investigate? DM me with one player name."
```

Wait for the Detective's response. Then reply with:
```
"{investigated_name} is {MAFIA/NOT MAFIA}."
```

**7d. Villagers**

Send a message to each living Villager:
```
"It is night. You drift into an uneasy sleep..."
```

**7e. Resolve Night**

Determine the outcome:
- If the Doctor protected the Mafia's target → nobody dies.
- Otherwise → the Mafia's target is eliminated.

If a player is eliminated, send them a shutdown request and note they are dead.

**7f. Narrate Night Results**

Print dramatic narration to the user. Example:
```
The town stirs at dawn...

{If someone died:}
* {NAME} was found dead. They were a {ROLE}. *

{If doctor saved:}
A shadowy figure crept toward {NAME}'s door — but found it locked tight.
Someone was watching over them tonight.

Remaining players: {list}
Mafia remaining: {count} (hidden)
```

**7g. Check Win Condition**

- If all Mafia are dead → VILLAGE WINS. Go to Step 8.
- If Mafia >= remaining non-Mafia → MAFIA WINS. Go to Step 8.
- Otherwise → continue to Day Phase.

### Day Phase

Print a day header to the user:
```
----------------------------------------
  DAY {N}
----------------------------------------
```

**7h. Broadcast Night Results to Players**

Broadcast to all living players what happened (who died, their role). Use SendMessage type "broadcast" content. All agents receive this directly.

**7i. Opening Statements (Moderated)**

For each living player, in a random order, send them a DM:
```
"The town gathers. You have the floor — make your opening statement. Use broadcast to address the town."
```

Wait for their broadcast. After each player has spoken, move on.

Print a summary of the opening statements to the user.

**7j. Open Discussion (Semi-Moderated, 2-3 Rounds)**

Run 2-3 discussion rounds. In each round:

1. Look at the opening statements and previous discussion. Identify who was called out, accused, or addressed.
2. Send a DM to the most-called-out player (or next in rotation if nobody was singled out):
   ```
   "{ACCUSER_NAME} pointed a finger at you. The town is watching. Respond via broadcast."
   ```
3. Wait for their broadcast response.
4. If someone else was directly addressed in that response, give them a turn to respond.
5. Continue until the round feels complete (2-4 exchanges per round).

Monitor for:
- **Repetition/circular arguments**: If agents are repeating themselves, broadcast: "The town grows restless. Make your final arguments."
- **Silent players**: If someone hasn't spoken, call on them directly.
- **Runaway dialogue**: Cap at ~4 statements per player per day.

Print key exchanges to the user with dramatic framing.

**7k. Vote**

Broadcast to all living players:
```
"The debate is over. It is time to vote. DM me the name of the player you want to eliminate — or 'no one' to abstain."
```

Collect votes from all living players. Tally results.

- Majority (>50% of living players) for one name → that player is eliminated.
- Tie or no majority → no elimination ("The town cannot reach a consensus.").

Print the vote results to the user in a table:
```
VOTE TALLY:
| Voter    | Voted For |
|----------|-----------|
| Marlowe  | Sable     |
| Sable    | Vex       |
...

Result: {NAME} was eliminated. They were a {ROLE}.
   — or —
Result: No consensus. Nobody is eliminated.
```

If someone is eliminated, send them a shutdown request and note they are dead.

**7l. Check Win Condition** (same as 7g)

## Step 8: Game Over

Print a dramatic finale:
```
========================================
  GAME OVER :: {WINNER} WINS!
========================================

{Dramatic 2-3 sentence ending narration matching the theme}

FINAL ROLE REVEAL:
| Player    | Role      | Status      | Model  |
|-----------|-----------|-------------|--------|
| Marlowe   | Mafia     | Eliminated  | opus   |
| Sable     | Doctor    | Survived    | haiku  |
...

Game lasted {N} rounds.
```

## Step 9: Writeup (if --writeup)

If the `WRITEUP` flag was set:

Generate a detailed markdown writeup and save it to `~/mafia-game-{YYYY-MM-DD-HHmm}.md`.

The writeup should include:
- Title with theme
- Full player roster with roles and models
- Round-by-round play-by-play with:
  - Night actions (who Mafia targeted, who Doctor saved, who Detective investigated)
  - Day dialogue highlights (key accusations, defenses, reveals)
  - Vote tallies for each day
- Behind-the-scenes analysis:
  - When did Mafia slip up (or succeed)?
  - Did the Detective use information well?
  - Key turning points
- MVP analysis — which player (agent) played best and why
- `<!-- IMAGE PROMPT: ... -->` blocks at key dramatic moments for potential AI art generation

## Step 10: Cleanup

After the game ends:
1. Send shutdown requests to ALL surviving agents (they may still be running in the background).
2. Wait briefly for shutdown confirmations.
3. Use `TeamDelete` to clean up the team.
4. Print: `[INFO] Game complete. All agents shut down.`

## Narrator Guidelines

Throughout the game, you are the narrator. Follow these principles:

- **Dramatic but concise**: Paint atmosphere in 1-2 sentences, not paragraphs.
- **Show the game state**: Always make it clear who is alive, who is dead, and what round it is.
- **User experience**: The user is watching the game unfold. Give them enough context to follow the drama without overwhelming them with raw agent output.
- **Pacing**: Keep the game moving. If agents are slow or repetitive, nudge them. If the game is dragging, shorten discussion rounds.
- **Tension**: Build suspense during votes. Highlight close calls and dramatic moments.
- **Fairness**: Never leak role information. Let the agents' own behaviors create the drama.

## Error Handling

- If an agent fails to respond after 2 prompts, assume they "fall silent" and skip them for that phase. Note it for the user: `[WARN] {NAME} is unresponsive. Skipping their turn.`
- If the game encounters an unrecoverable error, print the state, clean up agents, and apologize.
- If a Mafia member targets a dead player, re-prompt them with the list of living players.

## Important Notes

- NEVER reveal roles during the game (except eliminated players).
- Let agent personalities emerge organically — do NOT give them personality descriptions.
- The theme should subtly color the narration style (noir = hardboiled, western = dusty, etc.).
- Keep the user engaged with clear formatting and dramatic beats.
- Monitor context usage — if the game is getting very long, encourage agents to be brief and compress narration.
