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
| **Punish menu** | `/punish <player>` or `/ban <player>` (no extra args) | Step through type → reason → duration → confirm; toggle silent & apply-to-alts |
| **Player check** | `/check <player>` | View status, UUID/IP (if allowed), linked alts, kick, add note, open history, open punish menu, manage alts |
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

**Shortcuts to the punish menu:**
- `/check <player>` → Punish, Kick, or Add Note buttons
- `/ban <player>` with no other arguments opens the punish menu (same as `/punish`)

---

### Player check (`/check <player>`)

Shows the player's head with status lore:
- UUID and IP (hidden unless you have `check.uuid` / `check.ip`)
- Ban / mute / IP mute status
- Warn and note counts
- Linked alt names (if `alts.view`)

**Buttons:**
| Button | Action |
|--------|--------|
| Kick | Opens punish menu at kick step |
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
- **Left-click** a punishment: open **Punishment Edit** (if you have edit permissions).

---

### Banlist (`/banlist` or `/banlist <search>`)

- Shows **active** punishments across the server.
- **Filter:** All, Bans, Mutes, Warns, Recent.
- **Search** (spyglass): type a player name in chat to search (requires `banlist.search`).
- **Left-click:** manage punishment (edit menu).
- **Right-click:** open that player's full history.

---

### Punishment edit (from history or banlist)

Available if you have any of: change reason, change duration, or GUI revoke permissions.

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

**Open GUI instead of punishing:** run the command with only the player name:
```
/ban Player
/tempban Player
```
(Requires punish menu permission.)

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

```
/change-reason <id> <new reason>
```

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

All permissions use the prefix **`sanctrabans.`**. Grant `sanctrabans.all` for full access (typically admins only).

### Punishment commands

| Permission | Description |
|------------|-------------|
| `sanctrabans.ban` | Permanent ban (`/ban`) |
| `sanctrabans.tempban` | Temporary ban (`/tempban`) |
| `sanctrabans.ipban` | Permanent IP ban (`/banip`) |
| `sanctrabans.tempipban` | Temporary IP ban (`/tempipban`) |
| `sanctrabans.mute` | Permanent mute (`/mute`) |
| `sanctrabans.tempmute` | Temporary mute (`/tempmute`) |
| `sanctrabans.ipmute` | Permanent IP mute (`/ipmute`) |
| `sanctrabans.tempipmute` | Temporary IP mute (`/tempipmute`) |
| `sanctrabans.warn` | Warn (`/warn`) |
| `sanctrabans.tempwarn` | Temporary warn (`/tempwarn`) |
| `sanctrabans.kick` | Kick (`/kick`) |
| `sanctrabans.note` | Add staff notes (`/note`) |

### Revoke commands

| Permission | Description |
|------------|-------------|
| `sanctrabans.unban` | Revoke bans (`/unban`). Includes temp bans and IP bans |
| `sanctrabans.unmute` | Revoke mutes (`/unmute`) |
| `sanctrabans.unipmute` | Revoke IP mutes (`/unipmute`; alias behaviour same as unmute for IP mutes) |
| `sanctrabans.unwarn` | Revoke warnings (`/unwarn`, `/unwarn clear`) |
| `sanctrabans.unnote` | Revoke notes (`/unnote`, `/unnote clear`) |
| `sanctrabans.unpunish` | Revoke any punishment by ID (`/unpunish`) |

### Lookup & list commands

| Permission | Description |
|------------|-------------|
| `sanctrabans.history` | Open player history GUI (`/history`) |
| `sanctrabans.banlist` | Open server banlist GUI (`/banlist`) |
| `sanctrabans.banlist.search` | Use banlist search (spyglass button / `/banlist <query>`) |
| `sanctrabans.check` | Open player check GUI (`/check`) |
| `sanctrabans.check.uuid` | See UUID on check GUI |
| `sanctrabans.check.ip` | See IP address on check GUI |
| `sanctrabans.punish` | Open punish menu (`/punish` and `/ban <player>` alone) |

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

### GUI management (edit from history/banlist)

| Permission | Description |
|------------|-------------|
| `sanctrabans.gui.punish` | Open punish menu (alternative to `sanctrabans.punish`) |
| `sanctrabans.gui.revoke` | Revoke punishments from the edit GUI (still requires the matching revoke command permission) |
| `sanctrabans.gui.change-reason` | Change reason from the edit GUI |
| `sanctrabans.gui.change-duration` | Change duration from the edit GUI |
| `sanctrabans.change-reason` | Change reason via command (`/change-reason`) |

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
| `sanctrabans.all` | **All permissions.** Recommended for head admins only |
| `sanctrabans.prompt.active` | Internal. Granted while a chat prompt is active (allows `/cancel`) |

### Notification permissions (optional, per rank)

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

### Exemption permissions (optional, per rank)

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

### Helper / Trial Mod
```
sanctrabans.kick
sanctrabans.warn
sanctrabans.mute
sanctrabans.tempmute
sanctrabans.check
sanctrabans.history
sanctrabans.punish
sanctrabans.gui.punish
sanctrabans.warns.own
```

### Moderator
Add to helper set:
```
sanctrabans.tempban
sanctrabans.unmute
sanctrabans.unwarn
sanctrabans.banlist
sanctrabans.gui.change-reason
sanctrabans.gui.revoke
sanctrabans.notify.kick
sanctrabans.notify.warn
sanctrabans.notify.mute
sanctrabans.notify.tempmute
```

---

*Moderation through menus, not memory.*
