# Easy Blueprint Subsystem — User Manual

---

## Enable the Plugin

**Edit → Plugins** → search **Easy Blueprint Subsystem** → enable → restart the editor if prompted.

---

## What This Plugin Does

Use **Blueprints** to author **subsystem** logic: global services, per-level services, or editor-only tools. After you **compile and save** your subsystem Blueprint, the plugin creates instances automatically. Other Blueprints use a **Get** node (under the same categories as the engine’s **Get Subsystem** nodes) to obtain your subsystem and call its functions or variables.

You only need three workflows:

1. **Content Browser right-click** — create a subsystem Blueprint (pick one of four parent types).  
2. **Inside the subsystem Blueprint** — implement **On Initialize** / **On Deinitialize** (optional **Tick**, editor events).  
3. **In Actor / UI / other Blueprints** — **Get &lt;YourSubsystemName&gt;** to use it.

---

## Choosing One of Four Subsystem Types

| Content Browser item | Scope | Typical use |
|----------------------|-------|-------------|
| **Easy Blueprint Engine Subsystem** | One instance for the whole game / editor process | Global config, cross-level managers |
| **Easy Blueprint Game Instance Subsystem** | One set per new game session (including PIE) | Saves, player session, run-scoped data |
| **Easy Blueprint World Subsystem** | One set per world / level run (including PIE) | Level rules, flow, logic only in that map |
| **Easy Blueprint Editor Subsystem** | Editor only; **not** in shipped game builds | Post-import hooks, save notifications, editor utilities |

You may have **multiple** Blueprints of the same scope (e.g. two Engine subsystems: `BP_Audio`, `BP_Config`); each class gets its own instance.

**Editor** types exist only in the editor. **Engine / Game Instance / World** types ship with the game.

---

## Creating a Subsystem Blueprint

### Steps

1. **Right-click** in the **Content Browser** (target folder).  
2. Open **Easy Blueprint Subsystem**.  
3. Pick one of the four items above (parent class is fixed; **no** parent picker dialog).  
4. Name the asset (e.g. `BP_Engine_XXX`, `BP_GI_Save`, `BP_World_Level`, `BP_Editor_Import`).  
5. Open the Blueprint.

Alternatively: **Right-click → Blueprint Class**, search **Easy Blueprint**, and pick one of the four **Easy Blueprint … Subsystem** parents.

### After Opening the Blueprint

1. In **Event Graph**:  
   - **On Initialize** — runs when the subsystem instance starts (cache refs, load data, register delegates, etc.).  
   - **On Deinitialize** — runs before the instance is torn down (unbind, clear variables).  
2. For per-frame logic: **Class Defaults → Tick → Can Ever Tick**, then add a **Tick** event in Event Graph (Editor type has no Tick; see below).  
3. **Compile** → **Save**.

Other Blueprints usually will **not** show the **Get** node until the subsystem Blueprint is compiled and saved.

### When On Initialize Runs

| Type | Rough timing |
|------|----------------|
| **Engine** | After editor load, or after PIE / game start (often early) |
| **Editor** | After editor load (editor only) |
| **Game Instance** | **After PIE or game start** (often no instance while only editing assets) |
| **World** | **After entering a level and PIE / game start** (same) |

For Game Instance / World, **Play (PIE)** or **Run** before testing On Initialize or breakpoints.

### Delete, Rename, or Major Edits

- **Delete** the subsystem Blueprint like any other asset in the Content Browser.  
- After **rename or large changes**, **Compile** + **Save** again; if Get nodes or behavior look wrong, close and reopen related Blueprints or restart the editor.

---

## Using a Subsystem in Other Blueprints

### Add a Get Node

1. Open the consumer Blueprint’s **Event Graph** (Actor, Level Blueprint, Widget, etc.).  
2. **Right-click** on the graph.  
3. Find your node by **subsystem type** (name = your **subsystem Blueprint class name**):

| Subsystem type | Context menu path (same as engine) |
|----------------|-----------------------------------|
| Engine | **Engine Subsystems** → **Get BP_XXX** |
| Game Instance | **GameInstance Subsystems** → **Get BP_XXX** |
| World | **World Subsystems** → **Get BP_XXX** |
| Editor | **Editor Subsystems** → **Get BP_XXX** |

Or search `Get BP_YourAssetName`.

4. From the return value, call **functions** or read **variables** on your subsystem Blueprint.

### When the Get Node Appears

1. Subsystem Blueprint is **compiled** and **saved**.  
2. The graph supports this node type (normal Event Graphs do).

If it still missing: save subsystem → close/reopen consumer Blueprint → restart editor.

### Usage Tips

- Call **Get** when you need it (e.g. **BeginPlay**, button events); avoid storing the instance on a short-lived Actor as a long-lived “global” reference.  
- After PIE ends or the world changes, do not keep using an old reference from a previous Get.

---

## On Initialize, On Deinitialize, Tick

### On Initialize / On Deinitialize

In the **subsystem Blueprint’s** Event Graph:

| Event | Meaning |
|-------|---------|
| **On Initialize** | Called once when the plugin creates the instance |
| **On Deinitialize** | Called once before the instance is destroyed |

Use **On Deinitialize** to undo what **On Initialize** set up; do not start new gameplay logic there.

### Tick (Game Instance / World only)

1. Open subsystem Blueprint → **Class Defaults**.  
2. **Tick**:  
   - **Can Ever Tick** — must be enabled for **Tick** events (off by default).  
   - **Tick Interval** — `0` = every frame; e.g. `0.5` ≈ twice per second.  
   - **Tick Even When Paused** — tick while game is paused.  
3. Implement the **Tick** event in Event Graph.

**Engine** and **Editor** subsystems have no Tick (Engine is a global singleton without a stable per-frame world; Editor uses **Editor Events** below).

Tick is meaningful during **PIE / game**; Game Instance / World usually do not tick while only editing without play.

---

## Editor Subsystem and Events

Only for **Easy Blueprint Editor Subsystem** — editor automation (import, save, etc.).

### Create

Same as other types: **Content Browser → Easy Blueprint Subsystem → Easy Blueprint Editor Subsystem**.

### Bind Events (pick one approach)

**A — Class Defaults (good for always-on listeners)**

1. Open the Editor subsystem Blueprint.  
2. **Class Defaults**.  
3. **Editor Events**.  
4. On an event (e.g. **On Asset Added**), click **+** and bind to a custom event.  
5. Implement logic in that event (log path, refresh UI, etc.).

**B — Bind in On Initialize**

1. In **On Initialize**, drag from **On Asset Added** (etc.) → **Assign** → your custom event.  
2. In **On Deinitialize**, **Clear** or unbind the same delegate.

### Event Reference

| Event | When it fires | Parameters (in Blueprint) |
|-------|---------------|---------------------------|
| **On Asset Added** | Asset registered in Content Browser (create, import, save) | Asset, Asset Path, Asset Class |
| **On Asset Removed** | Asset removed from project | Same |
| **On Asset Updated** | Rename, move, resave (frequent; consider throttling) | Same |
| **On Asset Editor Opened** | Asset editor opened for asset | Asset |
| **On Asset Editor Closed** | Asset editor closed | Asset |
| **On Asset Post Import** | Import / reimport finished | Factory, Imported Object |
| **On World Saved** | Level / map save completed | World |

**Asset** may be null; use **Asset Path** (string) to identify the asset.

### Get Editor Subsystem Elsewhere

In Blueprints that run **in the editor only** (e.g. Editor Utility): **Right-click → Editor Subsystems → Get &lt;YourEditorSubsystemName&gt;**.

---

## FAQ

**Q: Cannot find the Get node?**  
A: **Compile + Save** the subsystem Blueprint; reopen the consumer Blueprint; restart the editor.

**Q: Game Instance / World On Initialize never runs?**  
A: Start **PIE** or **Run**; instances often do not exist while only editing assets.

**Q: Multiple Engine subsystems?**  
A: Yes — one instance per subsystem Blueprint class.

**Q: Editor subsystem in a packaged game?**  
A: No — editor-only.

**Q: Can the subsystem Blueprint have functions and variables?**  
A: Yes — same as any Blueprint; other graphs Get the instance and call them.

---
