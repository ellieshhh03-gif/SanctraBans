# SanctraBans

GUI-first punishment plugin for Paper 1.21.x with built-in alt detection, IP mute, staff vanish, Simple Voice Chat mute support, and full punishment history. Most staff workflows start with a command that opens a menu. Commands can also be used end-to-end without opening a GUI.

**Data folder:** `plugins/SanctraBans/`  
**Admin command:** `/sanctrabans`  
**Permission prefix:** `sanctrabans.*`

## Contents

1. [Features overview](#1-features-overview)
2. [Using the GUI](#2-using-the-gui)
3. [Commands only (no GUI)](#3-commands-only-no-gui)
4. [Other important notes](#4-other-important-notes)
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
| **IP Mute** | Mutes all accounts sharing the IP (by player name or raw IPv4; includes voice chat when Simple Voice Chat is installed) |
| **Temp IP Mute** | IP mute for a set amount of time (by player name or raw IPv4; includes voice chat when Simple Voice Chat is installed) |
| **Warn** | Issues a warning (counts toward warn actions) |
| **Temp Warn** | Warning that lasts for a set amount of time |
| **Kick** | Removes an online player (not stored long-term) |
| **Note** | Internal staff record; player is not notified |

---

### GUI features

*In-game menus for issuing and reviewing punishments.*

| GUI | How to open | What you can do |
|-----|-------------|-----------------|
| **Punish menu** | `/punish <player>` (full flow), or `/ban <player>`, `/tempban <player>`, etc. with only the target | Pick type → reason → duration (temps) → confirm; type-only commands skip straight to the reason step |
| **Player check** | `/check <player>` | View join status, ban/mute/IP-mute state, warns/notes, linked alts; kick (online only), punish, history, notes, alt management |
| **History** | `/history <player>` or from check/banlist | Browse all punishments with filters; left-click to edit/revoke |
| **Banlist** | `/banlist` or `/banlist <search>` | View active punishments server-wide; filter, search, browse pages; left-click to manage, right-click for player history |
| **Warns** | `/warns` or `/warns <player>` | Multi-page list of warnings |
| **Notes** | `/notes` or `/notes <player>` | Multi-page list of staff notes |
| **Punishment edit** | Left-click a punishment in history or banlist | Change reason, change duration (active temps), revoke (with optional batch revoke for alts) |
| **Alt management** | `/check <player>` → Manage Alts | View linked accounts, manually link/unlink alts |
| **Duration layout browser** | Punish menu duration step (when enabled) | Pick any configured time layout |

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
| `/warns [player]` | Open warns GUI |
| `/notes [player]` | Open notes GUI |
| `/vanish` | Toggle staff vanish on yourself (hidden from tab list and other players) |
| `/vanish <player>` | Toggle vanish on another online player |
| `/sanctrabans reload` | Reload configs (admin) |
| `/cancel` | Cancel an active chat input prompt (while a GUI asked you to type something) |

---

### Extra systems

*Behaviours that apply across multiple commands and menus.*

- **Silent punishments:** `-s` flag on commands or toggle in the punish confirm step. Silent punishments do not broadcast to other staff (unless they have notify permissions).
- **Alt detection:** Accounts on the same IP can be auto-linked. Staff with permission can apply punishments to linked alts (`-a` / `--alts` on commands, or toggle in GUI).
- **Escalation layouts:** Repeat offenses for the same reason can auto-suggest the next duration step (configured in `escalation.yml`).
- **Duration presets:** Quick-pick durations in the GUI (configured in `config.yml`).
- **Warn actions:** Automatic commands when warn count hits a threshold (e.g. temp-ban at 3 warns).
- **Staff duration limits:** `temp-perms` in config cap how long each staff rank can punish (by permission level).
- **Offline & never-joined players:** `/check`, `/history`, and punish commands resolve names via local cache and Mojang (same as issuing a ban). Never-joined targets show a clear status; punishments apply on first login.
- **Chat prompts:** Some GUI buttons ask you to type in chat (custom reason, custom duration, banlist search). Type `/cancel` to abort.
- **Simple Voice Chat:** When [Simple Voice Chat](https://modrinth.com/plugin/simple-voice-chat) is installed on the server, SanctraBans mutes also block proximity voice for muted players. See [Simple Voice Chat integration](#simple-voice-chat-integration) below.
- **Staff vanish:** `/vanish` fully hides staff from other players (body, armor, tab list). Staff with bypass permission can still see vanished players. See [Staff vanish](#staff-vanish) below.

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
- **Browse layouts:** pick any time layout (on by default; set `independent-time-layouts: false` in config to disable).
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

**Shortcuts to the punish menu** (players only; console runs the command immediately instead):
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
| **UUID** | Real UUID, or `Could not resolve`. Requires **`sanctrabans.check.uuid`** (not granted by `check` alone) |
| **IP** | Live or last-known IP, or `Never connected` / `Hidden`. Requires **`sanctrabans.check.ip`** (not granted by `check` alone) |
| **Ban / mute / IP mute** | Active punishment state for the account |
| **Warns & notes** | Counts |
| **Linked alts** | Names of linked accounts (requires `alts.view`) |

Without `sanctrabans.check.uuid` or `sanctrabans.check.ip`, those lines show **Hidden** (or unresolved / never connected where applicable). `sanctrabans.check` alone does not include either.

**Buttons:**
| Button | Action |
|--------|--------|
| Kick | Opens punish menu at kick step (**only when the target is online**); shows a locked barrier when offline or never joined |
| Manage Alts | Opens alt link/unlink GUI |
| Add Note | Opens punish menu at note step |
| History | Opens full punishment history |
| Punish | Opens full punish menu |
| Close | Closes menu |

---

### History (`/history <player>`)

*Full punishment log for one player, with filters and edit access.*

- Each punishment is shown as the **player's head** with full lore (type, status, reason, staff, dates, **punishment ID**).
- **Punishment IDs** in history and banlist are the **global database ID** (e.g. `#542`). Use this number with `/unpunish <id>` and `/change-reason <id>`. The warns and notes menus still use per-player numbering (`Warning #1`, `#2`, etc.) for that player only.
- **Filter** (compass): All, Active, Expired, Bans, Mutes, Warns, Notes.
- **Previous / Next:** browse through pages of results.
- **Left-click** a punishment: open **Punishment Edit** (requires `sanctrabans.history` or `sanctrabans.banlist`).

---

### Banlist (`/banlist` or `/banlist <search>`)

*Server-wide view of active punishments only.*

- Shows **active** punishments across the server (same **global database IDs** as history, e.g. `#542`).
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
| Revoke | Confirm revoke; optional **batch revoke** for all punishments in the same alt batch |
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

- View accounts linked to the target.
- **Link:** type another player name in chat to manually link.
- **Unlink:** confirm removal of a link.
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
| **Raw IPv4** | Uses that address directly; no Mojang lookup or prior join required |

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

**Time layout token** (enabled by default; disable with `independent-time-layouts: false`):
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
/banlist
/banlist <search query>
/warns [player]
/notes [player]
```

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

### Config changes on a live server

- Edit files in `plugins/SanctraBans/`, save, then `/sanctrabans reload` or restart.
- **Smart config updates:** On startup and reload, SanctraBans merges in changes from the new jar automatically:
  - **New keys** from the jar are added to your config files.
  - **Updated jar defaults** replace your value only if you never customized that key (your value still matches the previous jar default).
  - **Your custom edits are kept.** If you changed a value away from the old default, it will not be overwritten.
- Snapshots of the last jar defaults are stored in `plugins/SanctraBans/.defaults/` (auto-managed; do not edit). This is how the plugin tells the difference between “still on the old default” and “staff customized this”.
- Back up `plugins/SanctraBans/` before major updates (configs + `data.db`).

---

### Exempt players

Players listed in `exempt-players` in `config.yml` cannot be punished. Individual exemption permissions also exist (e.g. `sanctrabans.ban.exempt`).

---

### Staff duration limits

`temp-perms` in `config.yml` maps permission **levels** to max duration in seconds. Staff without a high enough level cannot issue longer temp punishments.

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
- MySQL can be enabled in `config.yml` for multi-server setups.

---

### Offline and never-joined players

- Punish commands (`/ban`, `/tempban`, etc.) work on players who are offline or have never joined, as long as the name resolves via Mojang.
- `/check` and `/history` use the same identity lookup, so ban/mute status and history stay accurate for those targets.
- Invalid names show a friendly error (`Could not find a Minecraft account named …`) instead of a raw engine message.

---

### IP visibility on `/check`

- The IP shown is the **real connection address** as the server sees it (used for IP bans/mutes and alt auto-linking).
- Requires **`sanctrabans.check.ip`** to display; otherwise the check GUI shows **Hidden**.
- On Bungee/Velocity networks, the address depends on IP forwarding being configured correctly on the proxy.

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

Both the **server plugin** and the **player mod** are required for voice mute enforcement. SanctraBans alone is enough for text mutes; voice blocking only applies when a muted player tries to speak through Simple Voice Chat.

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

### Staff vanish

*Hide staff from other players in-world and on the tab list.*

SanctraBans includes a staff **vanish** command that hides players from others in-world and on the tab list (armor, held items, and name tag included). Vanish is session-only and clears on disconnect.

| Command | Permission | Description |
|---------|------------|-------------|
| `/vanish` | `sanctrabans.vanish` | Toggle vanish on yourself |
| `/vanish <player>` | `sanctrabans.vanish.others` | Toggle vanish on another online player |

| Permission | Description |
|------------|-------------|
| `sanctrabans.vanish.see` | See vanished players in-world and on the tab list |
| `sanctrabans.vanish.exempt` | Cannot be vanished by other staff |

If another plugin also uses `/vanish`, you can still run `/sanctrabans:vanish`.

---

## 5. Config files

*What each file in `plugins/SanctraBans/` is for.*

All files live in **`plugins/SanctraBans/`**.

| File | Purpose |
|------|---------|
| **`config.yml`** | Main settings: database, `default-reason`, `independent-time-layouts`, exempt players, mute commands, voice-chat-mute, vanish, warn actions, temp-perms duration limits, alt detection, duration presets, debug, prefix, and more |
| **`reasons.yml`** | Preset reasons shown as books in the punish menu and edit-reason menu. Add entries here to show new reasons in the GUI |
| **`escalation.yml`** | Time/duration **layouts** (escalation ladders) and **bindings** that map each reason + punishment type to a layout. Controls auto-escalation in the punish GUI |
| **`gui.yml`** | Inventory layouts: slot positions for history, banlist, check, punish menu, reason picker, duration picker, filters, buttons, and theme materials |
| **`messages.yml`** | All in-game text: chat messages, GUI button labels, ban screen text references, broadcast templates |
| **`layouts.yml`** | Screen layouts shown to punished players (ban screen, mute screen, kick screen message lines) |
| **`data.db`** | SQLite database (created automatically). Stores all punishments, alt links, and escalation progress. Not a YAML file you edit by hand |

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

---

## 6. Permissions reference

*Every `sanctrabans.*` node, grouped by purpose.*

All permissions use the prefix **`sanctrabans.`**. Grant `sanctrabans.all` for full access to every node listed below.

### How permissions work

Each punishment or revoke permission covers **both commands and GUI**. There are no separate `gui.*` nodes.

| Example permission | Command access | GUI access |
|--------------------|----------------|------------|
| `sanctrabans.ban` | `/ban <player> <reason>` | Open punish menu; select Ban |
| `sanctrabans.tempban` | `/tempban <player> <duration> <reason>` | Open punish menu; select Temp Ban |
| `sanctrabans.unban` | `/unban <player>` | Revoke button in history/banlist edit menu |
| `sanctrabans.change-reason` | `/change-reason <id> [reason]` | Change reason button in edit menu |
| `sanctrabans.change-duration` | *(GUI only)* | Change duration button in edit menu (all temp types) |
| `sanctrabans.change-duration.tempmute` | *(GUI only)* | Change duration for temp mutes only |

**Opening the punish menu** (`/punish`, `/ban <player>` alone, check menu Punish button): requires at least one punishment-type permission (`ban`, `mute`, `kick`, etc.). Types you lack appear locked in the menu.

**Opening the punishment edit menu** (left-click in history/banlist): requires `sanctrabans.history` or `sanctrabans.banlist`. Individual edit actions (reason, duration, revoke) use their own permissions; missing permissions show as locked barrier buttons.

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
| `sanctrabans.unipmute` | Revoke IP mutes (`/unipmute`; alias behaviour same as unmute for IP mutes) and revoke from GUI |
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
| `sanctrabans.check` | Open player check GUI (`/check`): status, punishments, alts; **does not** include UUID or IP |
| `sanctrabans.check.uuid` | See UUID on check GUI (separate from `check`; included in `all`) |
| `sanctrabans.check.ip` | See IP address on check GUI (separate from `check`; included in `all`) |

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
| `sanctrabans.alts.apply` | Apply punishments to linked alts (`-a` flag, GUI toggle, batch revoke) |

---

### Vanish permissions

| Permission | Description |
|------------|-------------|
| `sanctrabans.vanish` | Toggle vanish on yourself (`/vanish`) |
| `sanctrabans.vanish.others` | Toggle vanish on other players (`/vanish <player>`) |
| `sanctrabans.vanish.see` | See vanished players in-world and on the tab list |
| `sanctrabans.vanish.exempt` | Cannot be vanished by other staff |

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
| `sanctrabans.vanish.exempt` | Cannot be vanished by other staff |
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
sanctrabans.vanish
sanctrabans.vanish.see
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

`sanctrabans.all` includes every permission in this reference (punishments, lookup, alts, vanish, silent, check UUID/IP, notifications, `/sanctrabans reload`, and `sanctrabans.vanish.exempt`).

---

*Moderation through menus, not memory.*
