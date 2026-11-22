# AGENT RULES: Project Dron

Read this file fully before every task. These rules override your defaults.

## 1. What this project is
Dron: single-player 3D action-RPG. A cloud LLM game master ("Dron", named after Dronacharya) gives NPCs persistent memory, drives world events, and powers free-text blacksmithing. Deterministic rules keep the AI from breaking balance.

## 2. Hard constraints (never violate)
- Budget: **$0**. No paid assets, APIs, or subscriptions.
- Hardware: i5-11260H, RTX 2050 (4 GB VRAM), **8 GB RAM (7.73 usable)**, Windows 11.
- **Banned:** Unreal Engine, local LLMs (Ollama, LM Studio), embedding models or vector DBs, heavy dependencies.
- Godot **4.7.2 stable, GDScript only** (no C#). Renderer: **Compatibility**. Dev resolution 720p/900p.
- Assets: low-poly CC0 only (Kenney.nl, Quaternius, Mixamo).

## 3. Architecture: three-layer authority
1. **AI layer** (cloud LLM behind an interchangeable `LLMProvider` interface): interprets text, writes dialogue, proposes hooks and whitelisted tags. Free tiers: Groq, Gemini.
2. **Rules layer** (Python FastAPI + SQLite): single source of truth. Validates all LLM JSON, clamps affinity to +/-5 per turn, checks inventory, computes weapon stats from a material table, keeps an append-only `events` table.
3. **Game layer** (Godot): renders, moves, fights, shows UI. Talks to FastAPI through one `ApiClient` autoload, asynchronously.

**The LLM never controls game state.** Godot never trusts unvalidated LLM output. Never execute LLM-generated code.

## 4. Repo layout
```
Project_dron/
  game/dron/          <- Godot project (project.godot lives here)
    scenes/{player,world,npcs,ui,combat,items}
    scripts/{player,npc,combat,systems,autoload}
    assets/
  backend/
    app/              (main.py, api, core, services, schemas, database, game_rules, memory, world_director)
    tests/
    requirements.txt
    .env.example
  docs/
  AGENT_RULES.md
```
Godot paths use `res://` relative to `game/dron/`.

## 5. Working rules
1. **One small, testable step at a time.** Stop after each step. Do not start the next phase.
2. **Never rewrite working modules.** Make the smallest diff that does the job. Do not refactor, rename, or reformat unrelated code.
3. **State exact file paths and the Godot node hierarchy before showing code.**
4. End every task with: what changed, how to test it, and the milestone that proves it works. Then wait for the user's confirmation.
5. Do not add features, dependencies, or files that were not requested.
6. If something is ambiguous or conflicts with these rules, ask one short question instead of guessing.
7. Say so plainly when unsure. Do not invent Godot APIs, node names, or model names. Verify against the 4.x docs or ask.

## 6. Godot rules
- Typed GDScript (`var x: float`, return types). Godot 4 syntax only (`@export`, `@onready`, `CharacterBody3D`, signals via `.connect()`).
- Prefer building scenes in the editor. If you edit `.tscn` files by hand, keep edits minimal and valid. A broken `.tscn` fails silently, so say exactly what you changed.
- Input actions live in Project Settings → Input Map. Current actions: `move_forward`, `move_back`, `move_left`, `move_right`. Planned: `interact` (E), `light_attack`, `heavy_attack`, `dodge` (Space), `sprint` (Shift), `inventory` (I).
- No blocking calls. All network I/O through `ApiClient` with timeouts and fallback lines (latency is 1-3 s).
- No API keys anywhere in the Godot project.
- Limited shadows, low-poly, no unnecessary assets loaded. Keep RAM low.

## 7. Backend rules
- Python, FastAPI, Pydantic, HTTPX, SQLite (FTS5 for long-term memory). Keep dependencies minimal.
- **Strict JSON schemas on every endpoint** (Pydantic models). Reject and log malformed input or LLM output. Never apply it.
- API keys only in `backend/.env` (never committed). Maintain `.env.example`.
- Whitelist everything: intent tags, event tags, weapon effect IDs (`FIRE_BURST`, `BLEED`, `POISON`, `FROST`, `KNOCKBACK`).
- Validate ownership and inventory before any state change.
- Authoritative numbers (affinity deltas, weapon damage/weight/stamina) are computed in Python, never by the LLM.
- `events` table is append-only.
- Add a test in `backend/tests/` for every validation rule.

## 8. NPC memory (4 tiers, no embeddings)
1. Personality: static text in the system prompt.
2. Relationship: trust/respect/affinity in SQLite, range -100..100.
3. Short-term: last N messages in SQLite.
4. Long-term: summarized facts retrieved with SQLite FTS5 keyword search.

## 9. Roadmap (strict order)
1. 3D foundation: Git, controller, camera, blacksmith shop, `E` interaction.
2. Backend bridge: `/health`, SQLite schema, `ApiClient` autoload.
3. Persistent AI NPC: JSON contract v1, FTS5 memory.
4. World Director: events log, 3 deterministic rules, quests.
5. Bounded AI blacksmith: description -> name/lore/effect ID -> Python stats -> spawn weapon.
6. Combat arena: light/heavy, dodge, stamina, 1 enemy.
7. Vertical slice polish and profiling on 8 GB.

**Current status:** Phase 1. Steps 1-2 done (repo, flat test world). Step 3 (player WASD) in progress. Update this line as steps are confirmed.

## 10. Git
- Commit after each confirmed step with a clear message (`Step N: what changed`).
- Never commit `.env`, `game/dron/.godot/`, `*.db`, or `venv/`.
- No force pushes. No destructive commands (`reset --hard`, deleting folders) without explicit approval.

## 11. Style of replies
Concise. Short bullets. No long essays. Code in blocks. Exact paths. No filler.
