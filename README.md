# SanctraBans

GUI-first punishment plugin for **Paper, Purpur, Spigot, and Folia** 1.21.x, with built-in IP-based alt detection, Bedrock/Geyser/Floodgate platform support, cross-server network sync (BungeeCord, Waterfall, or Velocity), IP mute, Simple Voice Chat mute support, and full punishment history. Most staff workflows start with a command that opens a menu. Commands can also be used end-to-end without opening a GUI.

**Supported game servers:** Paper · Purpur · Spigot · Folia (Minecraft 1.21.x)

**Data folder:** `plugins/SanctraBans/`  
**Admin command:** `/sanctrabans`  
**Permission prefix:** `sanctrabans.*`

## Contents

1. [Features overview](#1-features-overview)
2. [Using the GUI](#2-using-the-gui)
3. [Commands only (no GUI)](#3-commands-only-no-gui)
4. [Other important notes](#4-other-important-notes)
   - [Supported platforms](#supported-platforms)
   - [Network setup (multi-server)](#network-setup-multi-server)
   - [BungeeCord](#bungeecord)
   - [Waterfall](#waterfall)
   - [Velocity](#velocity)
   - [Alt detection on networks](#alt-detection-on-networks)
5. [Config files](#5-config-files)
6. [Permissions reference](#6-permissions-reference)
7. [Suggested permission sets by role](#7-suggested-permission-sets-by-role)

---

## 1. Features overview

*What staff can do at a glance: punishments, menus, commands, and extras.*

### Punishment types

| Type | What it does |
|------|----------------|
| **Ban** | Blocks the account from joining |
| **Temp Ban** | Blocks joining for a set amount of time |
| **IP Ban** | Blocks the player's IP from joining (by player name or raw IPv4) |
| **Temp IP Ban** | Blocks the IP for a set amount of time (by player name or raw IPv4) |
| **Mute** | Blocks chat, configured commands (e.g. `/msg`), and voice chat (when Simple Voice Chat is installed) |
| **Temp Mute** | Mutes for a set amount of time (includes voice chat when Simple Voice Chat is installed) |
| **IP Mute** | Mutes all accounts sharing the IP (by player name or raw IPv4, including voice chat when Simple Voice Chat is installed) |
| **Temp IP Mute** | IP mute for a set amount of time (by player name or raw IPv4, including voice chat when Simple Voice Chat is installed) |
| **Warn** | Issues a warning (counts toward warn actions) |
| **Temp Warn** | Warning that lasts for a set amount of time |
| **Kick** | Removes an online player (not stored long-term) |
| **Note** | Internal staff record. The player is not notified |

---

### GUI features

*In-game menus for issuing and reviewing punishments.*

| GUI | How to open | What you can do |
|-----|-------------|-----------------|
| **Punish menu** | `/punish <player>` (full flow), or `/ban <player>`, `/tempban <player>`, etc. with only the target | Pick type → reason → duration (temps) → confirm. Type-only commands skip straight to the reason step |
| **Player check** | `/check <player>` | View join status, ban/mute/IP-mute state, warns/notes, linked alts. Kick (online only), punish, history, notes, alt management |
| **History** | `/history <player>` or from check/banlist | Browse all punishments with filters. Left-click to edit/revoke |
| **Banlist** | `/banlist` or `/banlist <search>` | View active punishments server-wide. Filter, search, browse pages. Left-click to manage, right-click for player history |
| **Warns** | `/warns` or `/warns <player>` | Multi-page list of warnings |
| **Notes** | `/notes` or `/notes <player>` | Multi-page list of staff notes |
| **Punishment edit** | Left-click a punishment in history or banlist | Change reason, change duration (active temps), revoke (with optional batch revoke for alts) |
| **Alt management** | `/check <player>` → Manage Alts | View linked accounts, manually link/unlink alts |
| **Duration layout browser** | Punish menu duration step (when enabled) | Pick any configured time layout |
| **Staff check** | `/staffcheck <staff>` | Audit punishments **issued by** a staff member: summary counts, paginated history with filters, optional bulk rollback and restore |

---

### Command features (no GUI required)

*Run punishments and lookups entirely from chat.*

| Command | Purpose |
|---------|---------|
| `/ban`, `/tempban`, `/mute`, `/tempmute`, `/warn`, `/tempwarn`, `/kick` | Issue punishments directly |
| `/banip`, `/tempipban`, `/ipmute`, `/tempipmute` | IP-based punishments (player name or raw IPv4, e.g. `1.2.3.4`) |
| `/note` | Add a staff note |
| `/unban`, `/unmute`, `/unipmute`, `/unwarn`, `/unnote` | Revoke active punishments |
| `/unpunish <id>` | Revoke any stored punishment by database ID |
| `/change-reason <id> [reason]` | Change a punishment's reason by ID |
| `/punish <player>` | Open punish menu |
| `/history <player>` | Open history GUI |
| `/banlist [search]` | Open banlist GUI |
| `/check <player>` | Open check GUI |
| `/staffcheck <staff>` | Open staff check GUI (punishments issued by that staff member) |
| `/punishments rollback <staff> <window> [confirm]` | Roll back punishments issued by a staff member within a time window (hard delete) |
| `/punishments restore <batchId>` | Restore a rollback batch within the configured undo window |
| `/punishments rollback-list [staff]` | Open GUI of restorable rollback batches (all staff, or filtered). Console prints a text list |
| `/warns [player]` | Open warns GUI |
| `/notes [player]` | Open notes GUI |
| `/sanctrabans reload` | Reload configs (admin) |
| `/cancel` | Cancel an active chat input prompt (while a GUI asked you to type something) |

---

### Extra systems

*Behaviours that apply across multiple commands and menus.*

- **Silent punishments:** `-s` flag on commands or toggle in the punish confirm step. Silent punishments do not broadcast to other staff (unless they have notify permissions).
- **Alt detection:** Accounts that have ever shared an IP are auto-linked on join. Linking is **IP-only** (never username-based), so it works on premium, cracked, and mixed servers without false-linking unrelated players who happen to share a name. Staff with permission can apply punishments to linked alts (`-a` / `--alts` on commands, or toggle in GUI). See [Alt detection](#alt-detection) below.
- **Bedrock / Geyser / Floodgate:** When Floodgate is installed, Bedrock players are tagged on join and staff menus show **Platform: Java**, **Bedrock**, or **Unknown** (with a note that it is decided on next join). See [Bedrock / Geyser / Floodgate](#bedrock--geyser--floodgate) below.
- **Escalation layouts:** Repeat offenses for the same reason can auto-suggest the next duration step (configured in `escalation.yml`).
- **Duration presets:** Quick-pick durations in the GUI (configured in `config.yml`).
- **Warn actions:** Automatic commands when warn count hits a threshold (e.g. temp-ban at 3 warns).
- **Staff duration limits:** `temp-perms` in config cap how long each staff rank can punish (by permission level).
- **Staff check & rollback:** `/staffcheck` audits punishments issued by a staff member and can bulk-remove them within a time window (hard delete, not revoke). Snapshots are kept for a configurable undo window (default 3 days). Use `/punishments restore` or the staff check **Restore Rollback** button to undo an accidental rollback. Rollback window limits use `staffcheck.rollback-perms` and `sanctrabans.staffcheck.rollback.dur.*` (see [Staff check](#staff-check-staffcheck-staff) and [Staff duration limits](#staff-duration-limits)).
- **Offline & never-joined players:** `/check`, `/history`, and punish commands resolve names via local cache and Mojang (same as issuing a ban). Never-joined targets show a clear status. Punishments apply on first login.
- **Chat prompts:** Some GUI buttons ask you to type in chat (custom reason, custom duration, banlist search, **rollback time window**). Type `/cancel` to abort.
- **Simple Voice Chat:** When [Simple Voice Chat](https://modrinth.com/plugin/simple-voice-chat) is installed on the server, SanctraBans mutes also block proximity voice for muted players. See [Simple Voice Chat integration](#simple-voice-chat-integration) below.

---

## 2. Using the GUI

*Step-by-step guides for each staff menu.*

### Punish menu (`/punish <player>`)

The menu has up to four steps:

#### Step 1: Punishment type
- Click the punishment type you want (ban, temp ban, mute, kick, etc.).
- Types you lack permission for appear locked.
- **Cancel** closes the menu.

#### Step 2: Reason
- Click a **preset reason** (book items from `reasons.yml`).
- **No reason** (gray dye): use when there is no specific reason. Stored as **`No reason specified`** (change the text via `default-reason` in `config.yml`).
- Reasons are split across pages automatically: **14 per page**, with Previous/Next when needed.
- **Specify in chat** (book & quill): type a custom reason.
- **Cancel** closes the menu.

#### Step 3: Duration (temporary types only)
- **Escalation item** (repeater): use the suggested next step for this reason/offense (if configured).
- **Browse layouts:** pick any time layout (on by default. Set `independent-time-layouts: false` in config to disable).
- **Duration presets:** click a preset (e.g. `1d`, `7d`, `perma`).
- Presets use multiple pages like reasons: **14 per page**.
- **Specify in chat:** type a duration (e.g. `30m`, `2d`, `perma`).
- **Cancel** aborts.

#### Step 4: Confirm
- Review the summary (type, target, reason, duration, alt status).
- **Silent toggle:** hide staff broadcast (requires `sanctrabans.silent`).
- **Apply to alts toggle:** also punish linked alt accounts (requires `sanctrabans.alts.apply` and linked alts exist).
- **Confirm** issues the punishment.
- **Cancel** aborts.
- If the target **never joined**, the summary notes that the punishment applies on first login.

**Shortcuts to the punish menu** (players only. The console runs the command immediately instead):
- `/check <player>` → Punish, Kick, or Add Note buttons (kick/note open at reason step with type preset)
- `/ban <player>`, `/tempban <player>`, `/mute <player>`, `/warn <player>`, `/kick <player>`, `/note <player>`, `/banip <player|IPv4>`, etc. with **no other arguments** open the punish menu at the **reason step** with that command's type already selected (requires the matching punishment permission)
- `/punish <player>` opens the full menu at the **type picker** (any punishment type you can use)

---

### Player check (`/check <player>`)

*One-screen overview of a player before you punish or review history.*

Shows the player's head with status lore:

| Line | What it shows |
|------|----------------|
| **Status** | `Online now`, `Offline (last seen …)`, `Never joined this server`, or `Unknown account` |
| **Platform** | `Java`, `Bedrock`, or `Unknown` (with a hint that unknown is decided on next join). Requires Floodgate for Bedrock detection. See [Bedrock / Geyser / Floodgate](#bedrock--geyser--floodgate) |
| **UUID** | Real UUID, or `Could not resolve`. Requires **`sanctrabans.check.uuid`** (not granted by `check` alone) |
| **IP** | Live or last-known IP, or `Never connected` / `Hidden`. Requires **`sanctrabans.check.ip`** (not granted by `check` alone) |
| **Ban / mute / IP mute** | Active punishment state for the account |
| **Warns & notes** | Counts |
| **Linked alts** | Names of linked accounts (requires `alts.view`) |

Without `sanctrabans.check.uuid` or `sanctrabans.check.ip`, those lines show **Hidden** (or unresolved / never connected where applicable). `sanctrabans.check` alone does not include either.

**Buttons:**
| Button | Action |
|--------|--------|
| Kick | Opens punish menu at kick step (**only when the target is online**). Shows a locked barrier when offline or never joined |
| Manage Alts | Opens alt link/unlink GUI |
| Add Note | Opens punish menu at note step |
| History | Opens full punishment history |
| Punish | Opens full punish menu |
| Close | Closes menu |

---

### History (`/history <player>`)

*Full punishment log for one player, with filters and edit access.*

- Each punishment is shown as the **player's head** with full lore (type, status, reason, staff, dates, **punishment ID**).
- **Platform** (Java / Bedrock / Unknown) is shown on the player head when viewing one player's history.
- **Punishment IDs** in history and banlist are the **global database ID** (e.g. `#542`). Use this number with `/unpunish <id>` and `/change-reason <id>`. The warns and notes menus still use per-player numbering (`Warning #1`, `#2`, etc.) for that player only.
- **Filter** (compass): All, Active, Expired, Bans, Mutes, Warns, Notes.
- **Previous / Next:** browse through pages of results.
- **Left-click** a punishment: open **Punishment Edit** (requires `sanctrabans.history` or `sanctrabans.banlist`).

---

### Staff check (`/staffcheck <staff>`)

*Audit punishments issued by a staff member and optionally roll them back in bulk.*

Use this when you need to review what a moderator did, or undo a batch of mistakes they made recently. This is **not** the same as `/check <player>` (which inspects a **target** player). Staff check lists punishments where the named staff member was the **operator**.

**Opening the menu:**
- `/staffcheck <staff>` requires view permission (see [Staff check & rollback](#staff-check--rollback))
- Staff with only `sanctrabans.staffcheck.own` can open `/staffcheck` for **their own name only**
- Player-only command (console cannot open the GUI)
- If the named staff member has **no recorded punishments**, you get a chat message and the menu does not open

**Summary head (top of menu):**
- Total punishments issued by that staff member
- Breakdown by type (bans, mutes, warns, notes, kicks) with active counts
- Time since their most recent recorded punishment

**History list:**
- Same pagination and **filters** as player history (All, Active, Expired, Bans, Mutes, Warns, Notes)
- Each entry shows the **target player** head and punishment details
- **Left-click** an entry to open **Punishment Edit** (same as history/banlist)

**Rollback (requires `sanctrabans.staffcheck.rollback`):**

Use the GUI or the chat command:

```
/punishments rollback <staff> <window> [confirm]
```

**GUI flow:**

1. Click the **Rollback** button (TNT by default in `gui.yml`)
2. Pick a time window preset (from `staffcheck.rollback-presets` in `config.yml`) or type a custom window in chat (e.g. `2h`, `1d`)
3. Review the preview (counts by type, including alt-batch expansions)
4. Confirm to execute (skipped when `gui.confirm-revoke: false` in `config.yml`, rollback runs immediately after preview)

**Command flow:** Same permissions and limits as the GUI. Works from console. When `gui.confirm-revoke: true`, the first run prints a preview and you must add `confirm` at the end to execute (example: `/punishments rollback Steve 1d confirm`).

The time window is measured from **when each punishment was issued** (`start_time`), not when it expires.

**Important: rollback is a hard delete from live tables, not a revoke:**
- Matching punishment rows are **removed from the database** (including history-only records such as kicks)
- **Active** punishments are cleared from enforcement first (players can rejoin / chat again if nothing else blocks them)
- When any punishment in an alt batch matches, the **full alt batch** is included
- On networked servers, a single sync refresh propagates the removal to all backends sharing MySQL
- Staff revoke broadcasts still fire for active punishments that were cleared
- Before delete, a **snapshot batch** is stored so the rollback can be undone within the restore window (default **3 days**, configurable)

**Restore accidental rollbacks (requires `sanctrabans.staffcheck.rollback.restore`):**

```
/punishments restore <batchId>
/punishments rollback-list [staff]
```

After rollback, success messages include a **batch ID** (for example `#12`). Restore re-inserts deleted rows with their original IDs, re-tracks active punishments, and re-enforces online players. If a target already has an active punishment of the same blocking type (ban, mute, IP), that row is **skipped** and you get a partial restore message. Kick history rows are restored as history only (players are not re-kicked).

**Restore GUI:**
- **`/punishments rollback-list`** opens a **global** menu of all restorable batches (every staff member)
- **`/punishments rollback-list <staff>`** opens the same menu filtered to one operator
- **`/staffcheck <staff>` → Restore Rollback** opens the filtered menu with a back button to staff check

Restore eligibility is **not** tied to rollback window tier limits (`dur.*`). It only requires the restore permission, a batch within the configured age window, and a batch that has not already been restored.

**Rollback window limits:** Presets longer than your rank's cap appear as **locked barrier items** (same style as punishment types you cannot use). Custom chat input over the limit shows an error with your max window. Limits work like temp punishment caps: grant `sanctrabans.staffcheck.rollback.dur.N` and map `N` to max seconds in `staffcheck.rollback-perms` (see [Staff duration limits](#staff-duration-limits)).

**Configuration** (`config.yml` → `staffcheck`):

```yaml
staffcheck:
  rollback-presets:
    - 5m
    - 10m
    - 15m
    - 20m
    - 30m
    - 45m
    - 1h
    - 2h
    - 3h
    - 6h
    - 12h
    - 1d
    - 7d
    - 30d
  rollback-perms:
    1: 3600
    2: 86400
    3: 604800
    4: 2592000
    5: -1
  rollback-restore:
    enabled: true
    window-days: 3
    max-batches-kept: 50
```

| Key | Description |
|-----|-------------|
| `rollback-presets` | Quick-pick windows in the rollback time menu |
| `rollback-perms` | Max window in **seconds** per permission tier (`-1` = unlimited for that tier) |
| `rollback-restore.enabled` | Store snapshot batches before delete so rollbacks can be undone |
| `rollback-restore.window-days` | How long batch snapshots remain restorable (default 3) |
| `rollback-restore.max-batches-kept` | Cap on restorable batches listed / kept before purge |

GUI slot layout: `gui.yml` → `staffcheck`, `staffcheck-rollback-time`, `staffcheck-rollback-confirm`, `staffcheck-rollback-restore`, `rollback-restore-list`. Messages: `messages.yml` → `staffcheck.*`, `punishments.restore-*`.

---

### Banlist (`/banlist` or `/banlist <search>`)

*Server-wide view of active punishments only.*

- Shows **active** punishments across the server (same **global database IDs** as history, e.g. `#542`).
- **Platform** (Java / Bedrock / Unknown) is shown on each entry's player head.
- **Filter:** All, Bans, Mutes, Warns, Recent.
- **Search** (spyglass): type a player name in chat to search (requires `banlist.search`).
- **Left-click:** manage punishment (edit menu).
- **Right-click:** open that player's full history.

---

### Punishment edit (from history or banlist)

*Change reason, duration, or revoke an existing punishment.*

Open by **left-clicking** a punishment in history or banlist. Requires `sanctrabans.history` or `sanctrabans.banlist`. You do **not** need edit permissions to open the menu.

Actions you lack permission for appear as **barrier blocks** with *"You do not have permission for this."* lore. Allowed actions use the normal button icons.

| Action | Permission needed |
|--------|-------------------|
| Change reason | `sanctrabans.change-reason` |
| Change duration | `sanctrabans.change-duration` (all temp types), or a per-type node such as `sanctrabans.change-duration.tempmute` |
| Revoke | Matching revoke permission (e.g. `sanctrabans.unban` for bans) |

Change duration only appears for **active temporary** punishments (temp ban, temp mute, temp warn, temp IP ban/mute).

| Button | What it does |
|--------|----------------|
| Change reason | Pick a preset, **No reason**, or type in chat |
| Change duration | Pick a preset or type a new duration (active temp punishments only) |
| Revoke | Confirm revoke, or optional **batch revoke** for all punishments in the same alt batch |
| Back | Return to the previous menu |

---

### Warns & notes (`/warns`, `/notes`)

*Browse warnings or internal notes for yourself or another player.*

- Multi-page lists of warnings or notes.
- `/warns` or `/notes` alone shows **your own** entries (if permitted).
- `/warns <player>` or `/notes <player>` shows another player's entries.

---

### Alt management (`/check <player>` → Manage Alts)

*View, link, or unlink accounts tied to the same player.*

- View accounts linked to the target (auto-linked by shared IP history, or manually linked by staff).
- Each linked account shows its **platform** (Java / Bedrock / Unknown).
- **Link:** type another player name in chat to manually link.
- **Unlink:** confirm removal of a link. If the accounts share an IP again on a later join, they may be **auto-linked again** (unless that IP is ignored or over the shared-IP cap).
- Requires `sanctrabans.alts.manage` and `sanctrabans.alts.link`.

---

### Chat prompts

*When a menu asks you to type in chat instead of clicking a button.*

When a GUI asks you to type in chat:
1. The GUI closes temporarily.
2. Type your input in chat (reason, duration, search query, etc.).
3. Type **`/cancel`** to go back without submitting.

---

## 3. Commands only (no GUI)

*Issue, revoke, and look up punishments without opening a menu.*

### Issuing punishments

*Syntax for bans, mutes, warns, kicks, notes, and IP punishments.*

**Permanent punishments:**
```
/ban <player> [reason]
/mute <player> [reason]
/warn <player> [reason]
/kick <player> [reason]
/banip <player|IPv4> [reason]
/ipmute <player|IPv4> [reason]
/note <player> [text]
```

Omitting `[reason]` uses the default text **`No reason specified`** (set `default-reason` in `config.yml`).

**Temporary punishments:**
```
/tempban <player> <duration> [reason]
/tempmute <player> <duration> [reason]
/tempwarn <player> <duration> [reason]
/tempipban <player|IPv4> <duration> [reason]
/tempipmute <player|IPv4> <duration> [reason]
```

**IP command targets:** For `/banip`, `/tempipban`, `/ipmute`, and `/tempipmute`, the first argument can be either a **player name** or a **raw IPv4 address** (e.g. `192.168.1.50`).

| Target | How the IP is resolved |
|--------|----------------------|
| **Online player** | Uses their current connection IP |
| **Offline player (name)** | Uses the last-known IP from the plugin cache (player must have joined before) |
| **Raw IPv4** | Uses that address directly. No Mojang lookup or prior join required |

IP ban/mute types affect **everyone currently on that IP**. Apply-to-alts (`-a`) does not apply to raw IP targets (no linked account UUID).

Examples:
```
/banip 203.0.113.42 VPN evasion
/tempipban 10.0.0.1 7d Alt farming
/ipmute 192.168.1.50 Spam
/unban 203.0.113.42
```

**Duration format** (commands and chat input):
| Input | Meaning |
|-------|---------|
| `30s` | 30 seconds |
| `15m` | 15 minutes |
| `6h` | 6 hours |
| `7d` | 7 days |
| `2w` | 2 weeks |
| `1mo` | 1 month (30 days) |
| `perma` | Permanent |

**Silent flag:** add `-s` anywhere after the player name:
```
/tempban Player -s 1d Griefing
```

**Apply to linked alts:** add `-a` or `--alts` as the **last argument**:
```
/tempban Player 1d Griefing -a
```

**Time layout token** (enabled by default. Disable with `independent-time-layouts: false`):
```
/tempban Player #HackingBan Cheating
```

**Open GUI instead of punishing:** as a **player**, run a punish command with **only** the target (player name or IPv4 for IP commands). Add any extra argument (duration, reason, `-s`) and the command runs fully in chat instead.

```
/ban Player
/tempban Player
/banip 203.0.113.42
```

Opens the menu at the **reason step** with that type pre-selected. `/punish Player` still opens at the type picker.

---

### Revoking punishments

*Remove active bans, mutes, warns, or notes by player name, IP, or database ID.*

```
/unban <player> [-a]
/unmute <player> [-a]
/unipmute <player> [-a]
/unwarn <player> [-a]
/unwarn clear <player>
/unnote clear <player>
/unnote <id>
/unpunish <id>
```

- `-a` on revoke commands also revokes matching active punishments on linked alts (requires `alts.apply`).
- `/unpunish <id>` revokes any stored punishment by its **database ID** (shown as `#542` in history and banlist).
- `/unban`, `/unmute`, and `/unipmute` accept a **player name or raw IPv4** for IP-related revokes.

---

### Staff rollback

*Bulk-delete punishments issued by a staff member within a time window (hard delete from live tables, not revoke). Requires `sanctrabans.staffcheck.rollback` and view access to that operator.*

```
/punishments rollback <staff> <window> [confirm]
/punishments restore <batchId>
/punishments rollback-list [staff]
```

Examples:
```
/punishments rollback Steve 1d
/punishments rollback Steve 6h confirm
/punishments restore 12
/punishments rollback-list Steve
```

When `gui.confirm-revoke: true` in `config.yml`, the first rollback command prints a preview. Add `confirm` to execute. When `gui.confirm-revoke: false`, the rollback runs immediately. Same rollback window tier limits as the staff check GUI (`staffcheck.rollback-perms` and `sanctrabans.staffcheck.rollback.dur.*`).

Each rollback stores a snapshot batch (when `staffcheck.rollback-restore.enabled` is true). Success output includes the batch ID. Restore within `rollback-restore.window-days` (default 3) using `/punishments restore`, **`/punishments rollback-list`** (global GUI), or the staff check **Restore Rollback** button. Requires `sanctrabans.staffcheck.rollback.restore`. Partial restore occurs when restored rows would conflict with existing active punishments of the same blocking type.

---

### Editing punishments

*Update reason or duration on an existing entry.*

**Command:**
```
/change-reason <id> [reason]
```

Omitting `[reason]` uses the default (`No reason specified`).

**GUI:** left-click a punishment in history or banlist (`sanctrabans.history` or `sanctrabans.banlist`). Use **Change reason** in the edit menu (`sanctrabans.change-reason`) or **Change duration** on active temp punishments (`sanctrabans.change-duration`, or e.g. `sanctrabans.change-duration.tempban` for temp bans only). Locked barrier buttons mean you can view the menu but not that action.

Syncs to alt batch punishments when `sync-reason-in-batch` or `sync-duration-in-batch` is enabled in config.

---

### Lookup commands

*Open history, check, banlist, warns, or notes from chat.*

```
/history <player>
/check <player>
/staffcheck <staff>
/banlist
/banlist <search query>
/warns [player]
/notes [player]
/punishments rollback <staff> <window> [confirm]
/punishments restore <batchId>
/punishments rollback-list [staff]
```

`/staffcheck` opens a GUI. `/punishments rollback-list` opens the global restore GUI for players (console gets a text list). `/punishments rollback` and `restore` work from chat or console (see [Staff check](#staff-check-staffcheck-staff)).

---

### Admin

*Plugin maintenance commands.*

```
/sanctrabans          - show plugin version
/sanctrabans reload   - reload all config files
```

---

## 4. Other important notes

*Server setup, integrations, and behaviour that is not tied to a single menu.*

### Supported platforms

SanctraBans supports **Paper**, **Purpur**, **Spigot**, and **Folia** on game server backends and proxies.

At startup the plugin detects your server software and logs the platform, for example:

```text
SanctraBans enabled on 1.21.11-... (MC: 1.21.11) (platform=PAPER, full features).
SanctraBans enabled on 1.21.11-... (MC: 1.21.11) (platform=PAPER, full features).   ← Purpur logs as PAPER
SanctraBans enabled on 1.21.11-... (MC: 1.21.11) (platform=SPIGOT, full features).
SanctraBans enabled on 1.21.11-... (MC: 1.21.11) (platform=FOLIA, full features).
```

| Platform | Support | Notes |
|----------|---------|-------|
| **Paper** | Full | Recommended. Default compile target, with all features. |
| **Purpur** | Full | Paper-compatible fork, detected and logged as `platform=PAPER`. |
| **Spigot** | Near-full | Punishments, GUIs, mutes, network sync, alt detection, textured heads (Mojang session fetch), and Simple Voice Chat mute. Uses legacy chat and text APIs where Spigot lacks Paper features. |
| **Folia** | Full | Region-safe scheduling built in (`folia-supported: true`). Same feature set as Paper for normal staff workflows. |
| **BungeeCord / Waterfall** | Full (proxy) | Login ban enforcement + sync relay. |
| **Velocity** | Full (proxy) | Login ban enforcement + sync relay. |

Cross-platform networks (e.g. Paper + Folia backends with shared MySQL) are supported.

**Minecraft version:** 1.21.x

**Bedrock players:** Requires Floodgate on the backend regardless of server software (see [Bedrock / Geyser / Floodgate](#bedrock--geyser--floodgate)).

---

### Config changes on a live server

- Edit files in `plugins/SanctraBans/`, save, then `/sanctrabans reload` or restart.
- **Smart config updates:** On startup and reload, SanctraBans merges in changes from the new jar automatically:
  - **New keys** from the jar are added to your config files.
  - **Updated jar defaults** replace your value only if you never customized that key (your value still matches the previous jar default).
  - **Your custom edits are kept.** If you changed a value away from the old default, it will not be overwritten.
- Snapshots of the last jar defaults are stored in `plugins/SanctraBans/.defaults/` (auto-managed. Do not edit). This is how the plugin tells the difference between “still on the old default” and “staff customized this”.
- Back up `plugins/SanctraBans/` before major updates (configs + `data.db`).

---

### Network setup (multi-server)

*Run SanctraBans across a proxy network with shared punishments and instant sync between game server backends.*

SanctraBans **supports BungeeCord, Waterfall, and Velocity**. Backends can run Paper, Purpur, Spigot, or Folia. Each side detects its platform at startup.

| Proxy | Supported | Setup guide |
|-------|-----------|-------------|
| **BungeeCord** | Yes | [BungeeCord](#bungeecord) |
| **Waterfall** | Yes | [Waterfall](#waterfall) |
| **Velocity** (3.x) | Yes | [Velocity](#velocity) |

Install SanctraBans on your proxy and on each **game server backend** where staff issue punishments (Paper, Purpur, Spigot, or Folia). A network runs **one** proxy at a time, so use the setup guide that matches yours.

#### What every network needs

| Requirement | Details |
|-------------|---------|
| **Shared database** | MySQL or MariaDB on the proxy and every backend. SQLite cannot sync across servers. |
| **Matching credentials** | Same `use-mysql`, `mysql.type`, host, database, username, and password on every node. |
| **IP forwarding** | Backends must receive each player's real IP from the proxy (see your proxy section below). Required for alt detection, IP bans, and IP mutes. |
| **Backend sync** | `network.enabled: true` on each game server backend. `network.enabled: false` on the proxy. |

Use the same `mysql.type` everywhere (`mysql` or `mariadb`). Do not mix driver types against one database.

#### SanctraBans config

**Every node** (proxy and backends):

```yaml
use-mysql: true
mysql:
  type: mariadb   # or mysql
  host: localhost
  port: 3306
  database: sanctra
  username: root
  password: 'your-password'
database:
  pool-size: 10
  connection-timeout-ms: 10000
  max-lifetime-ms: 1800000
```

#### Database connection pool (networks)

SanctraBans uses HikariCP for database connections. On **shared MySQL/MariaDB** networks, each node (proxy + every backend) opens its own pool. Tune `database.pool-size` so total connections stay within your MySQL `max_connections` limit.

| Setup | Suggested `pool-size` per node |
|-------|-------------------------------|
| Single Paper server | `10` (default) |
| Busy backend on a shared network | `20`-`30` |
| Lightweight proxy only | `5`-`10` |

Example: one proxy + four backends at `pool-size: 20` uses up to **100** connections to MySQL. Leave headroom for admin tools and other plugins.

Other `database` keys:

- `connection-timeout-ms`: how long to wait for a free connection (default `10000`)
- `max-lifetime-ms`: recycle connections periodically (default `1800000`, 30 minutes)

Keep the database **geographically close** to your game servers when possible, because ban cache misses still hit MySQL on pre-login. Use a **dedicated MySQL instance** for SanctraBans on large networks if you can.

**Game server backends only** (unique `server-name` per server: Paper, Purpur, Spigot, or Folia):

```yaml
network:
  enabled: true
  server-name: 'survival'   # optional, defaults to the Bukkit server name
```

**Proxy only:**

```yaml
network:
  enabled: false
```

The proxy uses the database for login ban checks and IP recording. It relays sync messages between backends.

Customize the ban screen shown at the proxy in `plugins/SanctraBans/layouts.yml` (`BanScreen` layout).

#### How sync works

1. Staff issue, revoke, or update a punishment on a **game server backend**. SanctraBans writes the change to the **shared database**.
2. That backend sends a **plugin message** to the **proxy** (BungeeCord, Waterfall, or Velocity).
3. The proxy **relays** the message to every other backend on the network.
4. Each receiving backend **refreshes its local cache** from the database and **kicks** online players if they are now banned or otherwise blocked.

The proxy also checks the shared database on **login** and denies banned players before they reach a backend. Backends perform their own checks as well, so enforcement works even if a player connects directly during maintenance (though players should normally join through the proxy).

#### What runs where

| Feature | Proxy | Backend (Paper / Purpur / Spigot / Folia) |
|---------|-------|---------------------------------------------|
| Login ban enforcement | Yes | Yes |
| Mute / IP mute enforcement | No | Yes |
| GUI, commands, voice mute | No | Yes |
| Alt detection (current IP) | Records identity | Full linking and enforcement |
| Instant cache sync | Relays messages | Sends and receives |

#### Alt detection on networks

Alt detection links accounts that share the same **current IP**. The proxy and backends write into the same database. If backends only see the proxy IP instead of each player's real IP, unrelated accounts can be linked together.

| Node | IP source |
|------|-----------|
| **Proxy** | Player's address when they connect to the proxy |
| **Game server backend** | Forwarded player IP (only if forwarding is configured correctly) |

After setup, verify with `/check <player>` and `sanctrabans.check.ip` on a backend. Different players should show different public IPs.

If forwarding is misconfigured, you can add the proxy or host IP to `alt-detection.ignored-ips` as a fallback, but fixing forwarding is the correct solution. See [Alt detection](#alt-detection) for `ignored-ips` and `max-shared-ip-accounts`.

---

#### BungeeCord

SanctraBans fully supports **BungeeCord** proxies: login ban checks, IP recording, and sync relay between game server backends.

Use this section if your network runs BungeeCord.

**1. Install SanctraBans**

| Location | Config |
|----------|--------|
| BungeeCord `plugins/` | `use-mysql: true`, `network.enabled: false` |
| Each game server backend `plugins/` | `use-mysql: true`, `network.enabled: true` |

**2. BungeeCord proxy** (`config.yml`):

```yaml
ip_forward: true
```

**3. Each game server backend** (`spigot.yml`):

```yaml
settings:
  bungeecord: true
```

**4. Each game server backend** (`config/paper-global.yml`):

```yaml
proxies:
  velocity:
    enabled: false
```

**5. Restart** the proxy and all Paper servers.

**Verify:** ban a player on one backend. They should be blocked on the proxy and on other backends. `/check` on a backend should show each player's real IP, not the proxy IP.

---

#### Waterfall

SanctraBans fully supports **Waterfall** proxies. Waterfall is a BungeeCord fork, so setup matches [BungeeCord](#bungeecord) with the same SanctraBans and IP forwarding settings.

Use this section if your network runs Waterfall.

- `ip_forward: true` in Waterfall `config.yml`
- `bungeecord: true` in each Paper `spigot.yml`
- `velocity.enabled: false` in each Paper `paper-global.yml`
- Same SanctraBans database and `network` settings as BungeeCord

Install SanctraBans on the Waterfall proxy and on each game server backend where staff punish players.

---

#### Velocity

SanctraBans fully supports **Velocity** proxies (Velocity 3.x): login ban checks, IP recording, and sync relay between game server backends.

Use this section if your network runs Velocity.

**1. Install SanctraBans**

| Location | Config |
|----------|--------|
| Velocity `plugins/` | `use-mysql: true`, `network.enabled: false` |
| Each game server backend `plugins/` | `use-mysql: true`, `network.enabled: true` |

**2. Velocity proxy** (`velocity.toml`):

```toml
online-mode = true
player-info-forwarding-mode = "modern"
forwarding-secret = "your-secret-here"
```

Velocity may also create a `forwarding.secret` file. The secret must match Paper on every backend.

**3. Each game server backend** (`server.properties`):

```properties
online-mode=false
```

**4. Each Paper backend** (`spigot.yml`):

```yaml
settings:
  bungeecord: false
```

**5. Each game server backend** (`config/paper-global.yml`):

```yaml
proxies:
  velocity:
    enabled: true
    online-mode: true
    secret: 'your-secret-here'
```

Backends use offline mode. The proxy stays in online mode for Mojang authentication. Do not leave `bungeecord: true` in `spigot.yml` when using Velocity.

**6. Restart** Velocity and all Paper servers. Look for `SanctraBans Velocity enabled.` on the proxy and `Network sync enabled for server '...'` on each backend.

**Verify:** same as BungeeCord (ban syncs network-wide, and `/check` shows real player IPs on backends).

---

#### Known limits

- Simple Voice Chat integration and staff GUIs run on **game server backends only** (Paper, Purpur, Spigot, or Folia). They do not work on proxies.
- The proxy does not run punish commands or staff menus.
- Simple Voice Chat is optional on backends. Without it, voice mute is skipped and the rest of SanctraBans still loads.
- Players should connect through the proxy address, not directly to a backend port.

On first start with an existing database, harmless MariaDB/MySQL warnings about duplicate columns or indexes may appear during migration. They can be ignored.

---

### Exempt players

Players listed in `exempt-players` in `config.yml` cannot be punished. Individual exemption permissions also exist (e.g. `sanctrabans.ban.exempt`).

---

### Staff duration limits

SanctraBans uses two parallel tier maps in `config.yml`. Each **level** (`1`, `2`, …) is unlocked by granting the matching permission node. Higher levels override lower ones when a staff member has multiple tier nodes. A value of `-1` means unlimited for that tier.

#### Temp punishment limits (`temp-perms`)

Maps levels to the **maximum temp punishment duration** (in seconds) a staff member can issue. Checked when issuing or editing temp punishments.

| Config key | Permission (example) | Typical use |
|------------|----------------------|-------------|
| `temp-perms.1` | `sanctrabans.tempban.dur.1` (and same level on other temp types) | e.g. 1 hour max |
| `temp-perms.2` | `sanctrabans.tempmute.dur.2` | e.g. 1 day max |
| … | `sanctrabans.<type>.dur.N` | One node per temp command type you want to cap |
| *(bypass)* | `sanctrabans.<type>.dur.max` | Ignore all temp duration caps for that type |

Default tiers in `config.yml`: `1` = 1h, `2` = 1d, `3` = 7d, `4` = 30d, `5` = unlimited.

#### Rollback window limits (`staffcheck.rollback-perms`)

Maps levels to the **maximum rollback time window** (in seconds) a staff member can select when using staff check rollback. Checked on preset click, custom chat input, and again at confirm. Over-limit presets are shown as locked barriers in the rollback time menu.

| Config key | Permission | Typical use |
|------------|------------|-------------|
| `staffcheck.rollback-perms.1` | `sanctrabans.staffcheck.rollback.dur.1` | e.g. roll back up to 1 hour |
| `staffcheck.rollback-perms.2` | `sanctrabans.staffcheck.rollback.dur.2` | e.g. roll back up to 1 day |
| … | `sanctrabans.staffcheck.rollback.dur.N` | Levels 1-10 |
| *(bypass)* | `sanctrabans.staffcheck.rollback.dur.max` | Any rollback window |
| *(no tier granted)* | (none) | **Unlimited** rollback window (only `staffcheck.rollback` required) |

Default tiers mirror `temp-perms`: `1` = 1h, `2` = 1d, `3` = 7d, `4` = 30d, `5` = unlimited.

Example LuckPerms setup for a senior mod who may roll back up to 1 hour:

```
sanctrabans.staffcheck
sanctrabans.staffcheck.rollback
sanctrabans.staffcheck.rollback.dur.1
```

With default config, `dur.1` caps the window at 3600 seconds (1 hour). Grant `dur.2` for 1 day, `dur.3` for 7 days, and so on.

---

### Warn actions

When a player reaches a warn count defined in `warn-actions`, the listed commands run automatically (e.g. auto temp-ban).

---

### Notifications

- Non-silent punishments broadcast to online staff with the matching `sanctrabans.notify.<type>` permission (or `sanctrabans.all`).
- Revokes broadcast to staff with `sanctrabans.notify.revoke` or per-type revoke notify permissions.

---

### Database

- Default storage is SQLite (`data.db` in the plugin folder).
- MySQL or MariaDB can be enabled in `config.yml` (`use-mysql: true`, `mysql.type`: `mysql` or `mariadb`). SanctraBans uses the matching JDBC driver for each type.
- For network sync, every node must use the same `mysql.type` and credentials.
- Alt links, the last-known IP per account, and IP history are stored in the database and migrate automatically on plugin update.

---

### Alt detection

*How accounts are linked, and how to tune it for your server type.*

SanctraBans links alternate accounts when they **currently share the same IP address** (their last-known address). Linking is **purely IP-based**. Usernames are never used to decide links. That means:

- **Premium servers:** different UUIDs on the same IP (e.g. siblings, alts) are linked correctly.
- **Cracked / offline servers:** offline UUIDs are linked by IP like any other account.
- **Mixed servers (cracked + premium):** the same person's premium UUID and cracked UUID can both link to each other when they share an IP, without false-linking two unrelated players who happen to use the same name.

**On each join**, the plugin records the player's UUID, name, IP, and (when Floodgate is present) platform. If auto-linking is enabled, it links the joining account to **other accounts whose last-known IP is the same current IP**, and creates `auto_ip` links. Only the **current IP** is used for linking, not the player's full IP history. This is deliberate: matching on every historical IP would mass-link unrelated players who happened to use the same shared VPN or proxy address at different times.

**Proxy networks:** the proxy and backends all write to the same database when using shared MySQL. Backends must receive each player's **real IP** via BungeeCord or Velocity forwarding. If they only see the proxy IP, unrelated accounts will be linked together. See [Alt detection on networks](#alt-detection-on-networks) in the network setup section.

**Manual links** (`/check` → Manage Alts → Link) are stored separately and can be removed with Unlink. Unlinking does not block future auto-links: if both accounts join again from the same current IP, they may be re-linked automatically.

**False-positive protection:**

| Setting | Purpose |
|---------|---------|
| `ignored-ips` | IPs that are never recorded or linked (default: `127.0.0.1`, `::1`). On proxy networks, add your proxy or host IP only if backends still report it for every player (fix forwarding first). |
| `max-shared-ip-accounts` | Skip **auto-linking** on an IP once more than this many accounts currently share it. IPs are still recorded. Only automatic linking is skipped. Default: `0` (disabled, no limit). Set e.g. `10` on busy or VPN-heavy networks to reduce false positives. |

**Configuration** (`config.yml` → `alt-detection`):

```yaml
alt-detection:
  enabled: true
  auto-link-ip: true
  record-ip-history: true
  ignored-ips:
    - 127.0.0.1
    - '::1'
  max-shared-ip-accounts: 0
  apply-to-alts-default: false
  sync-reason-in-batch: true
  sync-duration-in-batch: true
```

| Key | Description |
|-----|-------------|
| `enabled` | Master toggle for alt detection features |
| `auto-link-ip` | Auto-link on join when another account currently shares the same IP |
| `record-ip-history` | Store UUID/name/IP on join. Needed for `/check` and IP punishments. Turn off only if you do not want IPs stored at all |
| `ignored-ips` | Never record or link these IPs |
| `max-shared-ip-accounts` | VPN/CGNAT/proxy protection for auto-linking. `0` = no limit (default) |
| `apply-to-alts-default` | Default state of the "apply to alts" toggle in the punish GUI |
| `sync-reason-in-batch` / `sync-duration-in-batch` | Keep alt batch punishments in sync when editing reason or duration |

**Staff workflow:** Linked alts appear on `/check` (with `alts.view`), in the alt management GUI, and can receive the same punishment with `-a` / `--alts` or the GUI toggle (requires `alts.apply`). Punishments propagate to linked accounts even when names match (e.g. premium vs cracked versions of the same player).

---

### Bedrock / Geyser / Floodgate

*Bedrock player detection and platform labels in staff menus.*

SanctraBans integrates with **[Floodgate](https://geysermc.org/wiki/floodgate/)** (used alongside Geyser) as an optional soft dependency. **Geyser alone is not required** for SanctraBans. Only Floodgate's API is used for Bedrock detection. If Floodgate is not installed, the plugin behaves as before (all players treated as Java, with no errors).

**What it does when Floodgate is present:**

| Feature | Behaviour |
|---------|-----------|
| **Join tagging** | Each join is recorded as **Java** or **Bedrock** in `ip_cache` |
| **Platform in menus** | Check, alt management, banlist, and history show **Platform: Java**, **Bedrock**, or **Unknown** on the player head |
| **Unknown platform** | Accounts that joined before this feature, or with no stored platform, show **Unknown** plus *"Determined the next time this player joins"* |
| **Never-joined Bedrock names** | Punishing or checking a Bedrock-prefixed name that has **never joined** will not fabricate a false offline/Mojang UUID (Bedrock UUIDs are XUID-based and cannot be guessed from the name). Once the player joins once, they resolve normally from cache |

**Requirements:**

| Component | Required? |
|-----------|-----------|
| **Floodgate plugin** on the backend | Yes, for Bedrock detection |
| **Geyser** | Typical setup (Bedrock clients connect through Geyser → Floodgate on backend) |
| Floodgate bundled in SanctraBans jar | No, compile-time API only. Floodgate must be on the server |

**Configuration** (`config.yml` → `bedrock`):

```yaml
bedrock:
  enabled: true
  prefix-override: ''
```

| Key | Description |
|-----|-------------|
| `enabled` | Turn Bedrock detection and platform display on or off |
| `prefix-override` | Override Floodgate's username prefix (e.g. `.`). Leave blank to use Floodgate's live configured prefix (recommended) |

**Messages** (`messages.yml` → `platform.*`): customize the Java, Bedrock, Unknown, and unknown-hint lines shown in menus.

**Soft dependencies** (in `plugin.yml`): `Geyser-Spigot`, `floodgate`.

---

### Offline and never-joined players

- Punish commands (`/ban`, `/tempban`, etc.) work on players who are offline or have never joined, as long as the name resolves via Mojang or local cache.
- **Bedrock-prefixed names** (Floodgate prefix, e.g. `.PlayerName`) that have **never joined** cannot be resolved to a UUID until their first login. Staff will see unresolved/unknown account status instead of a wrong UUID.
- `/check` and `/history` use the same identity lookup, so ban/mute status and history stay accurate for those targets.
- Invalid names show a friendly error (`Could not find a Minecraft account named …`) instead of a raw engine message.

---

### Simple Voice Chat integration

*Mute enforcement for proximity voice when Simple Voice Chat is installed.*

SanctraBans can block **proximity voice chat** for muted players when [Simple Voice Chat](https://modrinth.com/plugin/simple-voice-chat) is present. This uses the same mute state as text chat. There is no separate voice-mute command or database entry.

**Requirements for voice mutes to work:**

| Requirement | Who needs it | Why |
|-------------|--------------|-----|
| **SanctraBans** | Server | Issues and enforces mutes |
| **Simple Voice Chat server plugin** | Server | Receives voice packets and exposes the voice chat API |
| **Simple Voice Chat client mod** | Each player who uses voice | Without the mod, a player cannot use voice chat at all |

Both the **server plugin** and the **player mod** are required for voice mute enforcement. SanctraBans alone is enough for text mutes. Voice blocking only applies when a muted player tries to speak through Simple Voice Chat. Voice mute works on **Paper, Purpur, Spigot, and Folia** whenever Simple Voice Chat is installed on that backend.

**If Simple Voice Chat is not installed on the server**, SanctraBans works normally and the voice hook is skipped. Text mutes and command blocking are unchanged.

**Configuration** (`config.yml`):

```yaml
voice-chat-mute:
  enabled: true
  notify-cooldown-seconds: 5
```

- `enabled`: turn voice mute integration on or off without removing Simple Voice Chat
- `notify-cooldown-seconds`: how often to show the muted-voice message when a player keeps trying to talk (reduces spam)

**Messages** (`messages.yml` → `mute.blocked-voice` and `mute.blocked-voice-temp`) control what muted players see when voice is blocked.

**Server setup note:** Simple Voice Chat uses a separate **UDP port** (default `24454`) in addition to your Minecraft port. On hosted panels (e.g. Pterodactyl), you may need an extra allocation and `voice_host` set in `plugins/voicechat/voicechat-server.properties`. See the [Simple Voice Chat wiki](https://modrepo.de/minecraft/voicechat/wiki/troubleshooting) if players see "voice chat not connected".

**Permissions:** SanctraBans does not add a "can speak in voice" permission. Use Simple Voice Chat's own nodes (e.g. `voicechat.speak`) if you manage voice access separately. SanctraBans mute exempt permissions (`sanctrabans.mute.exempt`, etc.) prevent being muted in the first place.

---

## 5. Config files

*What each file in `plugins/SanctraBans/` is for.*

All files live in **`plugins/SanctraBans/`**.

| File | Purpose |
|------|---------|
| **`config.yml`** | Main settings: database, `network`, `default-reason`, `independent-time-layouts`, exempt players, mute commands, voice-chat-mute, warn actions, **temp-perms** and **staffcheck.rollback-perms** duration limits, **staffcheck** rollback presets, **alt detection**, **bedrock**, duration presets, debug, prefix, and more |
| **`reasons.yml`** | Preset reasons shown as books in the punish menu and edit-reason menu. Add entries here to show new reasons in the GUI |
| **`escalation.yml`** | Time/duration **layouts** (escalation ladders) and **bindings** that map each reason + punishment type to a layout. Controls auto-escalation in the punish GUI |
| **`gui.yml`** | Inventory layouts: slot positions for history, banlist, check, **staff check**, **rollback time/confirm**, punish menu, reason picker, duration picker, filters, buttons, and theme materials |
| **`messages.yml`** | All in-game text: chat messages, GUI button labels, ban screen text references, broadcast templates |
| **`layouts.yml`** | Screen layouts shown to punished players (ban screen, mute screen, kick screen message lines) |
| **`data.db`** | SQLite database (created automatically). Stores all punishments, alt links, IP cache, **IP history**, and escalation progress. Not a YAML file you edit by hand |

---

### Common customization tasks

*Quick reference for the most common config edits.*

| Goal | Edit this file |
|------|----------------|
| Add a punish reason | `reasons.yml` + matching layouts/bindings in `escalation.yml` |
| Change the "no reason" text | `config.yml` → `default-reason` |
| Change quick duration buttons | `config.yml` → `gui.duration-presets` |
| Change GUI colors/materials | `gui.yml` → `theme` and `items` sections |
| Change chat/broadcast wording | `messages.yml` |
| Change voice mute blocked message | `messages.yml` → `mute.blocked-voice` / `mute.blocked-voice-temp` |
| Change ban screen text | `layouts.yml` |
| Change how many warns triggers auto-ban | `config.yml` → `warn-actions` |
| Change staff check rollback presets | `config.yml` → `staffcheck.rollback-presets` |
| Change max rollback window per rank | `config.yml` → `staffcheck.rollback-perms` + LuckPerms `staffcheck.rollback.dur.*` |
| Change staff check / rollback GUI layout | `gui.yml` → `staffcheck`, `staffcheck-rollback-time`, `staffcheck-rollback-confirm` |
| Change staff check / rollback messages | `messages.yml` → `staffcheck.*` |
| Tune alt auto-linking (ignored IPs, shared-IP cap) | `config.yml` → `alt-detection` |
| Enable/disable Bedrock platform tags | `config.yml` → `bedrock` |
| Enable cross-server punishment sync | `config.yml` → `network.enabled` (requires shared MySQL) |
| Change platform labels in menus | `messages.yml` → `platform.*` |

---

## 6. Permissions reference

*Every `sanctrabans.*` node, grouped by purpose.*

All permissions use the prefix **`sanctrabans.`**. Grant `sanctrabans.all` for full access to every node listed below.

### How permissions work

Each punishment or revoke permission covers **both commands and GUI**. There are no separate `gui.*` nodes.

| Example permission | Command access | GUI access |
|--------------------|----------------|------------|
| `sanctrabans.ban` | `/ban <player> <reason>` | Open punish menu and select Ban |
| `sanctrabans.tempban` | `/tempban <player> <duration> <reason>` | Open punish menu and select Temp Ban |
| `sanctrabans.unban` | `/unban <player>` | Revoke button in history/banlist edit menu |
| `sanctrabans.change-reason` | `/change-reason <id> [reason]` | Change reason button in edit menu |
| `sanctrabans.change-duration` | *(GUI only)* | Change duration button in edit menu (all temp types) |
| `sanctrabans.change-duration.tempmute` | *(GUI only)* | Change duration for temp mutes only |

**Opening the punish menu** (`/punish`, `/ban <player>` alone, check menu Punish button): requires at least one punishment-type permission (`ban`, `mute`, `kick`, etc.). Types you lack appear locked in the menu.

**Opening the punishment edit menu** (left-click in history/banlist): requires `sanctrabans.history` or `sanctrabans.banlist`. Individual edit actions (reason, duration, revoke) use their own permissions. Missing permissions show as locked barrier buttons.

`sanctrabans.punish` is an optional legacy alias that opens the punish menu without any specific punishment permission. Most servers should grant individual type permissions instead.

---

### Punishment types

| Permission | Description |
|------------|-------------|
| `sanctrabans.ban` | Permanent ban (`/ban`) and punish menu access for bans |
| `sanctrabans.tempban` | Temporary ban (`/tempban`) and punish menu access for temp bans |
| `sanctrabans.ipban` | Permanent IP ban (`/banip` with player name or IPv4) and punish menu access |
| `sanctrabans.tempipban` | Temporary IP ban (`/tempipban` with player name or IPv4) and punish menu access |
| `sanctrabans.mute` | Permanent mute (`/mute`) and punish menu access |
| `sanctrabans.tempmute` | Temporary mute (`/tempmute`) and punish menu access |
| `sanctrabans.ipmute` | Permanent IP mute (`/ipmute` with player name or IPv4) and punish menu access |
| `sanctrabans.tempipmute` | Temporary IP mute (`/tempipmute` with player name or IPv4) and punish menu access |
| `sanctrabans.warn` | Warn (`/warn`) and punish menu access |
| `sanctrabans.tempwarn` | Temporary warn (`/tempwarn`) and punish menu access |
| `sanctrabans.kick` | Kick (`/kick`) and punish menu access |
| `sanctrabans.note` | Add staff notes (`/note`) and punish menu access |

---

### Revoke commands

| Permission | Description |
|------------|-------------|
| `sanctrabans.unban` | Revoke bans (`/unban`) and revoke from history/banlist GUI |
| `sanctrabans.unmute` | Revoke mutes (`/unmute`) and revoke from GUI |
| `sanctrabans.unipmute` | Revoke IP mutes (`/unipmute`, alias behaviour same as unmute for IP mutes) and revoke from GUI |
| `sanctrabans.unwarn` | Revoke warnings (`/unwarn`, `/unwarn clear`) and revoke from GUI |
| `sanctrabans.unnote` | Revoke notes (`/unnote`, `/unnote clear`) and revoke from GUI |
| `sanctrabans.unpunish` | Revoke any punishment by ID (`/unpunish`) |

---

### Edit punishments

| Permission | Description |
|------------|-------------|
| `sanctrabans.change-reason` | Change reason via command (`/change-reason`) or history/banlist edit GUI |
| `sanctrabans.change-duration` | Change duration on any active temp punishment via edit GUI |
| `sanctrabans.change-duration.tempban` | Change duration on active temp bans only |
| `sanctrabans.change-duration.tempmute` | Change duration on active temp mutes only |
| `sanctrabans.change-duration.tempipban` | Change duration on active temp IP bans only |
| `sanctrabans.change-duration.tempipmute` | Change duration on active temp IP mutes only |
| `sanctrabans.change-duration.tempwarn` | Change duration on active temp warnings only |

Per-type `change-duration.*` nodes are optional. Grant `sanctrabans.change-duration` for all temp types, or only the specific nodes you need. Issue permissions (`tempban`, `tempmute`, etc.) do **not** grant duration editing.

---

### Lookup & list commands

| Permission | Description |
|------------|-------------|
| `sanctrabans.history` | Open player history GUI (`/history`) |
| `sanctrabans.banlist` | Open server banlist GUI (`/banlist`) |
| `sanctrabans.banlist.search` | Use banlist search (spyglass button / `/banlist <query>`) |
| `sanctrabans.check` | Open player check GUI (`/check`): status, punishments, alts. **Does not** include UUID or IP |
| `sanctrabans.check.uuid` | See UUID on check GUI (separate from `check`, included in `all`) |
| `sanctrabans.check.ip` | See IP address on check GUI (separate from `check`, included in `all`) |

---

### Warns & notes viewing

| Permission | Description |
|------------|-------------|
| `sanctrabans.warns.use` | Parent permission for `/warns` (includes own + other) |
| `sanctrabans.warns.own` | View your own warnings (`/warns`) |
| `sanctrabans.warns.other` | View another player's warnings (`/warns <player>`) |
| `sanctrabans.warns` | Legacy. Grants full warns access |
| `sanctrabans.notes.use` | Parent permission for `/notes` |
| `sanctrabans.notes.own` | View your own notes |
| `sanctrabans.notes.other` | View another player's notes |
| `sanctrabans.notes` | Legacy. Grants full notes access |

---

### Alt account permissions

| Permission | Description |
|------------|-------------|
| `sanctrabans.alts.view` | See linked alts on `/check` |
| `sanctrabans.alts.manage` | Open alt management GUI from `/check` |
| `sanctrabans.alts.link` | Manually link and unlink alt accounts |
| `sanctrabans.alts.apply` | Apply punishments to linked alts (`-a` flag, GUI toggle, batch revoke). Links are IP-based (see [Alt detection](#alt-detection)) |

---

### Staff check & rollback

| Permission | Default | Description |
|------------|---------|-------------|
| `sanctrabans.staffcheck` | op | View punishments issued by **any** staff member (`/staffcheck <staff>`) |
| `sanctrabans.staffcheck.own` | false | View only punishments you issued yourself (`/staffcheck` with your own name) |
| `sanctrabans.staffcheck.rollback` | op | Use rollback from staff check GUI or `/punishments rollback` (hard-delete punishments in a time window). Requires view access to that operator |
| `sanctrabans.staffcheck.rollback.restore` | op | Restore a rollback batch within the configured undo window (`/punishments restore` or staff check GUI) |
| `sanctrabans.staffcheck.rollback.restore.list` | op | List restorable rollback batches (`/punishments rollback-list`). Inherits from restore permission |

**Rollback window tiers** (levels `dur.1` … `dur.10` are enforced at runtime. Only `dur.max` is declared in `plugin.yml`):

| Permission | Description |
|------------|-------------|
| `sanctrabans.staffcheck.rollback.dur.1` … `.dur.10` | Max rollback window in seconds, level maps to `staffcheck.rollback-perms` in `config.yml` |
| `sanctrabans.staffcheck.rollback.dur.max` | Bypass rollback window limits (included in `sanctrabans.all`) |

If a staff member has `staffcheck.rollback` but **no** `dur.*` nodes, rollback windows are unlimited. Grant tier nodes to cap how far back they can undo another staff member's actions. Over-limit presets appear as barrier blocks. Custom windows show `rollback-window-exceeds-limit` in chat.

---

### Other staff permissions

| Permission | Description |
|------------|-------------|
| `sanctrabans.silent` | Use silent punishments (`-s` flag and GUI silent toggle) |
| `sanctrabans.admin` | Admin commands (`/sanctrabans reload`) |
| `sanctrabans.all` | **All permissions** (includes every node in this reference) |
| `sanctrabans.prompt.active` | Internal. Granted while a chat prompt is active (allows `/cancel`) |

---

### Notification permissions (optional)

Staff only receive broadcast messages if they have the matching notify permission. These are **not** listed in `plugin.yml` but work at runtime:

| Permission pattern | Description |
|--------------------|-------------|
| `sanctrabans.notify.ban` | Broadcast when a ban is issued |
| `sanctrabans.notify.tempban` | Broadcast when a temp ban is issued |
| `sanctrabans.notify.mute` | Broadcast when a mute is issued |
| `sanctrabans.notify.tempmute` | Broadcast when a temp mute is issued |
| `sanctrabans.notify.warn` | Broadcast when a warn is issued |
| `sanctrabans.notify.kick` | Broadcast when a kick is issued |
| `sanctrabans.notify.note` | Broadcast when a note is added |
| *(same pattern for `ipban`, `tempipban`, `ipmute`, `tempipmute`, `tempwarn`)* | |
| `sanctrabans.notify.revoke` | All revoke broadcast notifications |
| `sanctrabans.notify.revoke.<type>` | Revoke broadcast for a specific type only |

`sanctrabans.all` includes revoke notify access.

---

### Exemption permissions (optional)

Grant to staff or players who should be immune to a punishment type:

| Permission | Description |
|------------|-------------|
| `sanctrabans.ban.exempt` | Cannot be banned |
| `sanctrabans.tempban.exempt` | Cannot be temp-banned |
| `sanctrabans.mute.exempt` | Cannot be muted |
| `sanctrabans.tempmute.exempt` | Cannot be temp-muted |
| `sanctrabans.warn.exempt` | Cannot be warned |
| *(same pattern for other types)* | |

Players in `exempt-players` in `config.yml` are also protected.

---

## 7. Suggested permission sets by role

*Copy-paste examples for LuckPerms or similar. Adjust to your rank structure.*

### Helper / Trial Mod
```
sanctrabans.kick
sanctrabans.warn
sanctrabans.mute
sanctrabans.tempmute
sanctrabans.check
sanctrabans.history
sanctrabans.warns.own
```

---

### Moderator

Add to helper set:
```
sanctrabans.tempban
sanctrabans.unmute
sanctrabans.unwarn
sanctrabans.banlist
sanctrabans.change-reason
sanctrabans.change-duration
sanctrabans.notify.kick
sanctrabans.notify.warn
sanctrabans.notify.mute
sanctrabans.notify.tempmute
```

---

### Senior Moderator

Add to moderator set:
```
sanctrabans.ban
sanctrabans.tempwarn
sanctrabans.note
sanctrabans.unban
sanctrabans.unnote
sanctrabans.unpunish
sanctrabans.silent
sanctrabans.check.uuid
sanctrabans.check.ip
sanctrabans.banlist.search
sanctrabans.warns.other
sanctrabans.notes.other
sanctrabans.alts.view
sanctrabans.alts.manage
sanctrabans.alts.link
sanctrabans.alts.apply
sanctrabans.staffcheck.own
sanctrabans.notify.tempban
sanctrabans.notify.ban
sanctrabans.notify.note
sanctrabans.notify.revoke
```

---

### Admin
```
sanctrabans.all
```

Or explicitly for owners who need staff audit without full `all`:
```
sanctrabans.staffcheck
sanctrabans.staffcheck.rollback
sanctrabans.staffcheck.rollback.dur.max
```

`sanctrabans.all` includes every permission in this reference (punishments, lookup, alts, staff check, rollback, silent, check UUID/IP, notifications, and `/sanctrabans reload`).

---

*Moderation through menus, not memory.*
