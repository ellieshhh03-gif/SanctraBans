# SanctraBans

GUI-first punishment plugin for Paper 1.21.x with built-in alt detection, IP mute, and full punishment history. Most staff workflows start with a command that opens a menu. Commands can also be used end-to-end without opening a GUI.

**Data folder:** `plugins/SanctraBans/`  
**Admin command:** `/sanctrabans`  
**Permission prefix:** `sanctrabans.*`

---

## 1. Features overview (what staff can do)

### Punishment types

| Type | What it does |
|------|----------------|
| **Ban** | Blocks the account from joining |
| **Temp Ban** | Blocks joining for a set amount of time |
| **IP Ban** | Blocks the player's IP from joining |
| **Temp IP Ban** | Blocks the IP for a set amount of time |
| **Mute** | Blocks chat and configured commands (e.g. `/msg`) |
| **Temp Mute** | Mutes for a set amount of time |
| **IP Mute** | Mutes all accounts sharing the IP |
| **Temp IP Mute** | IP mute for a set amount of time |
| **Warn** | Issues a warning (counts toward warn actions) |
| **Temp Warn** | Warning that lasts for a set amount of time |
| **Kick** | Removes an online player (not stored long-term) |
| **Note** | Internal staff record; player is not notified |

### GUI features

| GUI | How to open | What you can do |
|-----|-------------|-----------------|
| **Punish menu** | `/punish <player>`, `/ban <player>` (no extra args), or any punish command with only the player name | Step through type → reason → duration → confirm; toggle silent & apply-to-alts. Types you can use match your punishment permissions |
| **Player check** | `/check <player>` | View join status, ban/mute/IP-mute state, warns/notes, linked alts; kick (online only), punish, history, notes, alt management |
| **History** | `/history <player>` or from check/banlist | Browse all punishments with filters; left-click to edit/revoke |
| **Banlist** | `/banlist` or `/banlist <search>` | View active punishments server-wide; filter, search, browse pages; left-click to manage, right-click for player history |
| **Warns** | `/warns` or `/warns <player>` | Multi-page list of warnings |
| **Notes** | `/notes` or `/notes <player>` | Multi-page list of staff notes |
| **Punishment edit** | Left-click a punishment in history or banlist | Change reason, change duration (active temps), revoke (with optional batch revoke for alts) |
| **Alt management** | `/check <player>` → Manage Alts | View linked accounts, manually link/unlink alts |
| **Duration layout browser** | Punish menu duration step (when enabled) | Pick any configured time layout |

### Command features (no GUI required)

| Command | Purpose |
|---------|---------|
| `/ban`, `/tempban`, `/mute`, `/tempmute`, `/warn`, `/tempwarn`, `/kick` | Issue punishments directly |
| `/banip`, `/tempipban`, `/ipmute`, `/tempipmute` | IP-based punishments |
| `/note` | Add a staff note |
| `/unban`, `/unmute`, `/unipmute`, `/unwarn`, `/unnote` | Revoke active punishments |
| `/unpunish <id>` | Revoke any stored punishment by database ID |
| `/change-reason <id> <reason>` | Change a punishment's reason by ID |
| `/punish <player>` | Open punish menu |
| `/history <player>` | Open history GUI |
| `/banlist [search]` | Open banlist GUI |
| `/check <player>` | Open check GUI |
| `/warns [player]` | Open warns GUI |
| `/notes [player]` | Open notes GUI |
| `/sanctrabans reload` | Reload configs (admin) |
| `/cancel` | Cancel an active chat input prompt (while a GUI asked you to type something) |

### Extra systems staff should know about

- **Silent punishments:** `-s` flag on commands or toggle in the punish confirm step. Silent punishments do not broadcast to other staff (unless they have notify permissions).
- **Alt detection:** Accounts on the same IP can be auto-linked. Staff with permission can apply punishments to linked alts (`-a` / `--alts` on commands, or toggle in GUI).
- **Escalation layouts:** Repeat offenses for the same reason can auto-suggest the next duration step (configured in `escalation.yml`).
- **Duration presets:** Quick-pick durations in the GUI (configured in `config.yml`).
- **Warn actions:** Automatic commands when warn count hits a threshold (e.g. temp-ban at 3 warns).
- **Staff duration limits:** `temp-perms` in config cap how long each staff rank can punish (by permission level).
- **Offline & never-joined players:** `/check`, `/history`, and punish commands resolve names via local cache and Mojang (same as issuing a ban). Never-joined targets show a clear status; punishments apply on first login.
- **Chat prompts:** Some GUI buttons ask you to type in chat (custom reason, custom duration, banlist search). Type `/cancel` to abort.

---

## 2. Using the GUI

### Punish menu (`/punish <player>`)

The menu has up to four steps:

#### Step 1: Punishment type
- Click the punishment type you want (ban, temp ban, mute, kick, etc.).
- Types you lack permission for appear locked.
- **Cancel** closes the menu.

#### Step 2: Reason
- Click a **preset reason** (book items from `reasons.yml`).
- Reasons are split across pages automatically: **14 per page**, with Previous/Next when needed.
- **Specify in chat** (book & quill, above Cancel): type a custom reason in chat.
- **Cancel** closes the menu.

#### Step 3: Duration (temporary types only)
- **Escalation item** (repeater): use the suggested next step for this reason/offense (if configured).
- **Browse layouts:** pick any time layout (only when `independent-time-layouts` is enabled in config).
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

**Shortcuts to the punish menu:**
- `/check <player>` → Punish, Kick, or Add Note buttons
- `/ban <player>`, `/tempban <player>`, `/mute <player>`, etc. with no other arguments (requires the matching punishment permission)
- `/punish <player>` (requires at least one punishment permission, e.g. `sanctrabans.ban` or `sanctrabans.mute`)

---

### Player check (`/check <player>`)

Shows the player's head with status lore:

| Line | What it shows |
|------|----------------|
| **Status** | `Online now`, `Offline (last seen …)`, `Never joined this server`, or `Unknown account` |
| **UUID** | Real UUID, or `Could not resolve` — requires **`sanctrabans.check.uuid`** (not granted by `check` alone) |
| **IP** | Live or last-known IP, or `Never connected` / `Hidden` — requires **`sanctrabans.check.ip`** (not granted by `check` alone) |
| **Ban / mute / IP mute** | Active punishment state for the account |
| **Warns & notes** | Counts |
| **Linked alts** | Names of linked accounts (requires `alts.view`) |

Without `sanctrabans.check.uuid` or `sanctrabans.check.ip`, those lines show **Hidden** (or unresolved / never connected where applicable). `sanctrabans.check` alone does not include either.

**Buttons:**
| Button | Action |
|--------|--------|
| Kick | Opens punish menu at kick step — **only when the target is online**; shows a locked barrier when offline or never joined |
| Manage Alts | Opens alt link/unlink GUI |
| Add Note | Opens punish menu at note step |
| History | Opens full punishment history |
| Punish | Opens full punish menu |
| Close | Closes menu |

---

### History (`/history <player>`)

- Each punishment is shown as the **player's head** with full lore (type, status, reason, staff, dates, ID).
- **Filter** (compass): All, Active, Expired, Bans, Mutes, Warns, Notes.
- **Previous / Next:** browse through pages of results.
- **Left-click** a punishment: open **Punishment Edit** (if you have permission to change reason, change duration, or revoke that type).

---

### Banlist (`/banlist` or `/banlist <search>`)

- Shows **active** punishments across the server.
- **Filter:** All, Bans, Mutes, Warns, Recent.
- **Search** (spyglass): type a player name in chat to search (requires `banlist.search`).
- **Left-click:** manage punishment (edit menu).
- **Right-click:** open that player's full history.

---

### Punishment edit (from history or banlist)

Available when you have permission for at least one action on that punishment:

| Action | Permission needed |
|--------|-------------------|
| Change reason | `sanctrabans.change-reason` |
| Change duration | Matching temp permission (e.g. `sanctrabans.tempban` for temp bans) |
| Revoke | Matching revoke permission (e.g. `sanctrabans.unban` for bans) |

| Button | What it does |
|--------|----------------|
| Change reason | Pick a preset or type a custom reason |
| Change duration | Pick a preset or type a new duration (active temp punishments only) |
| Revoke | Confirm revoke; optional **batch revoke** for all punishments in the same alt batch |
| Back | Return to the previous menu |

---

### Warns & notes (`/warns`, `/notes`)

- Multi-page lists of warnings or notes.
- `/warns` or `/notes` alone shows **your own** entries (if permitted).
- `/warns <player>` or `/notes <player>` shows another player's entries.

---

### Alt management (`/check <player>` → Manage Alts)

- View accounts linked to the target.
- **Link:** type another player name in chat to manually link.
- **Unlink:** confirm removal of a link.
- Requires `sanctrabans.alts.manage` and `sanctrabans.alts.link`.

---

### Chat prompts

When a GUI asks you to type in chat:
1. The GUI closes temporarily.
2. Type your input in chat (reason, duration, search query, etc.).
3. Type **`/cancel`** to go back without submitting.

---

## 3. Commands only (no GUI)

### Issuing punishments

**Permanent punishments:**
```
/ban <player> [reason]
/mute <player> [reason]
/warn <player> [reason]
/kick <player> [reason]
/banip <player> [reason]
/ipmute <player> [reason]
/note <player> [text]
```

**Temporary punishments:**
```
/tempban <player> <duration> [reason]
/tempmute <player> <duration> [reason]
/tempwarn <player> <duration> [reason]
/tempipban <player> <duration> [reason]
/tempipmute <player> <duration> [reason]
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

**Time layout token** (only when `independent-time-layouts: true` in config):
```
/tempban Player #HackingBan Cheating
```

**Open GUI instead of punishing:** run any punish command with only the player name:
```
/ban Player
/tempban Player
/mute Player
```
Requires the matching punishment permission (e.g. `sanctrabans.ban` for `/ban Player`). Staff with any punishment permission can also use `/punish Player`.

---

### Revoking punishments

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
- `/unpunish <id>` revokes any stored punishment by its database ID (shown in history lore).

---

### Editing punishments

**Command:**
```
/change-reason <id> <new reason>
```

**GUI:** left-click a punishment in history or banlist (requires `sanctrabans.change-reason`). To change duration on an active temp punishment, use the edit menu (requires the matching temp permission, e.g. `sanctrabans.tempban`).

Syncs to alt batch punishments when `sync-reason-in-batch` is enabled in config.

---

### Lookup commands

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

```
/sanctrabans          - show plugin version
/sanctrabans reload   - reload all config files
```

---

## 4. Other important notes

### Config changes on a live server

- Edit files in `plugins/SanctraBans/`, save, then `/sanctrabans reload` or restart.
- Updating the JAR does **not** overwrite your existing config files. New default keys are merged in automatically.
- Back up `plugins/SanctraBans/` before major updates (configs + `data.db`).

### Exempt players

Players listed in `exempt-players` in `config.yml` cannot be punished. Individual exemption permissions also exist (e.g. `sanctrabans.ban.exempt`).

### Staff duration limits

`temp-perms` in `config.yml` maps permission **levels** to max duration in seconds. Staff without a high enough level cannot issue longer temp punishments.

### Warn actions

When a player reaches a warn count defined in `warn-actions`, the listed commands run automatically (e.g. auto temp-ban).

### Notifications

- Non-silent punishments broadcast to online staff with the matching `sanctrabans.notify.<type>` permission (or `sanctrabans.all`).
- Revokes broadcast to staff with `sanctrabans.notify.revoke` or per-type revoke notify permissions.

### Database

- Default storage is SQLite (`data.db` in the plugin folder).
- MySQL can be enabled in `config.yml` for multi-server setups.

### Offline and never-joined players

- Punish commands (`/ban`, `/tempban`, etc.) work on players who are offline or have never joined, as long as the name resolves via Mojang.
- `/check` and `/history` use the same identity lookup, so ban/mute status and history stay accurate for those targets.
- Invalid names show a friendly error (`Could not find a Minecraft account named …`) instead of a raw engine message.

### IP visibility on `/check`

- The IP shown is the **real connection address** as the server sees it (used for IP bans/mutes and alt auto-linking).
- Requires **`sanctrabans.check.ip`** to display; otherwise the check GUI shows **Hidden**.
- On Bungee/Velocity networks, the address depends on IP forwarding being configured correctly on the proxy.

---

## 5. Config files (what each one does)

All files live in **`plugins/SanctraBans/`**.

| File | Purpose |
|------|---------|
| **`config.yml`** | Main settings: database, exempt players, mute commands, UUID lookup URLs, warn actions, temp-perms duration limits, alt detection, GUI presets (`duration-presets`), debug, prefix, and more |
| **`reasons.yml`** | Preset reasons shown as books in the punish menu and edit-reason menu. Add entries here to show new reasons in the GUI |
| **`escalation.yml`** | Time/duration **layouts** (escalation ladders) and **bindings** that map each reason + punishment type to a layout. Controls auto-escalation in the punish GUI |
| **`gui.yml`** | Inventory layouts: slot positions for history, banlist, check, punish menu, reason picker, duration picker, filters, buttons, and theme materials |
| **`messages.yml`** | All in-game text: chat messages, GUI button labels, ban screen text references, broadcast templates |
| **`layouts.yml`** | Screen layouts shown to punished players (ban screen, mute screen, kick screen message lines) |
| **`data.db`** | SQLite database (created automatically). Stores all punishments, alt links, and escalation progress. Not a YAML file you edit by hand |

### Common customization tasks

| Goal | Edit this file |
|------|----------------|
| Add a punish reason | `reasons.yml` + matching layouts/bindings in `escalation.yml` |
| Change quick duration buttons | `config.yml` → `gui.duration-presets` |
| Change GUI colors/materials | `gui.yml` → `theme` and `items` sections |
| Change chat/broadcast wording | `messages.yml` |
| Change ban screen text | `layouts.yml` |
| Change how many warns triggers auto-ban | `config.yml` → `warn-actions` |

---

## 6. Permissions reference

All permissions use the prefix **`sanctrabans.`**. Grant `sanctrabans.all` for full access to every node listed below.

### How permissions work

Each punishment or revoke permission covers **both commands and GUI**. There are no separate `gui.*` nodes.

| Example permission | Command access | GUI access |
|--------------------|----------------|------------|
| `sanctrabans.ban` | `/ban <player> <reason>` | Open punish menu; select Ban |
| `sanctrabans.tempban` | `/tempban <player> <duration> <reason>` | Open punish menu; select Temp Ban; change temp ban duration in edit menu |
| `sanctrabans.unban` | `/unban <player>` | Revoke button in history/banlist edit menu |
| `sanctrabans.change-reason` | `/change-reason <id> <reason>` | Change reason button in edit menu |

**Opening the punish menu** (`/punish`, `/ban <player>` alone, check menu Punish button): requires at least one punishment-type permission (`ban`, `mute`, `kick`, etc.). Types you lack appear locked in the menu.

`sanctrabans.punish` is an optional legacy alias that opens the punish menu without any specific punishment permission. Most servers should grant individual type permissions instead.

### Punishment types

| Permission | Description |
|------------|-------------|
| `sanctrabans.ban` | Permanent ban (`/ban`) and punish menu access for bans |
| `sanctrabans.tempban` | Temporary ban (`/tempban`) and punish menu access for temp bans |
| `sanctrabans.ipban` | Permanent IP ban (`/banip`) and punish menu access |
| `sanctrabans.tempipban` | Temporary IP ban (`/tempipban`) and punish menu access |
| `sanctrabans.mute` | Permanent mute (`/mute`) and punish menu access |
| `sanctrabans.tempmute` | Temporary mute (`/tempmute`) and punish menu access |
| `sanctrabans.ipmute` | Permanent IP mute (`/ipmute`) and punish menu access |
| `sanctrabans.tempipmute` | Temporary IP mute (`/tempipmute`) and punish menu access |
| `sanctrabans.warn` | Warn (`/warn`) and punish menu access |
| `sanctrabans.tempwarn` | Temporary warn (`/tempwarn`) and punish menu access |
| `sanctrabans.kick` | Kick (`/kick`) and punish menu access |
| `sanctrabans.note` | Add staff notes (`/note`) and punish menu access |

### Revoke commands

| Permission | Description |
|------------|-------------|
| `sanctrabans.unban` | Revoke bans (`/unban`) and revoke from history/banlist GUI |
| `sanctrabans.unmute` | Revoke mutes (`/unmute`) and revoke from GUI |
| `sanctrabans.unipmute` | Revoke IP mutes (`/unipmute`; alias behaviour same as unmute for IP mutes) and revoke from GUI |
| `sanctrabans.unwarn` | Revoke warnings (`/unwarn`, `/unwarn clear`) and revoke from GUI |
| `sanctrabans.unnote` | Revoke notes (`/unnote`, `/unnote clear`) and revoke from GUI |
| `sanctrabans.unpunish` | Revoke any punishment by ID (`/unpunish`) |

### Edit punishments

| Permission | Description |
|------------|-------------|
| `sanctrabans.change-reason` | Change reason via command (`/change-reason`) or history/banlist edit GUI |

### Lookup & list commands

| Permission | Description |
|------------|-------------|
| `sanctrabans.history` | Open player history GUI (`/history`) |
| `sanctrabans.banlist` | Open server banlist GUI (`/banlist`) |
| `sanctrabans.banlist.search` | Use banlist search (spyglass button / `/banlist <query>`) |
| `sanctrabans.check` | Open player check GUI (`/check`) — status, punishments, alts; **does not** include UUID or IP |
| `sanctrabans.check.uuid` | See UUID on check GUI (separate from `check`; included in `all`) |
| `sanctrabans.check.ip` | See IP address on check GUI (separate from `check`; included in `all`) |

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

### Alt account permissions

| Permission | Description |
|------------|-------------|
| `sanctrabans.alts.view` | See linked alts on `/check` |
| `sanctrabans.alts.manage` | Open alt management GUI from `/check` |
| `sanctrabans.alts.link` | Manually link and unlink alt accounts |
| `sanctrabans.alts.apply` | Apply punishments to linked alts (`-a` flag, GUI toggle, batch revoke) |

### Other staff permissions

| Permission | Description |
|------------|-------------|
| `sanctrabans.silent` | Use silent punishments (`-s` flag and GUI silent toggle) |
| `sanctrabans.admin` | Admin commands (`/sanctrabans reload`) |
| `sanctrabans.all` | **All permissions** (includes every node in this reference) |
| `sanctrabans.prompt.active` | Internal. Granted while a chat prompt is active (allows `/cancel`) |

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

## Suggested permission sets by role

Examples only — adjust to match your server's rank structure and permission plugin.

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

### Moderator
Add to helper set:
```
sanctrabans.tempban
sanctrabans.unmute
sanctrabans.unwarn
sanctrabans.banlist
sanctrabans.change-reason
sanctrabans.notify.kick
sanctrabans.notify.warn
sanctrabans.notify.mute
sanctrabans.notify.tempmute
```

---

*Moderation through menus, not memory.*
