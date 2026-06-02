# VoidAuth

**Mandatory Authentication Mod for Minecraft 26.1–26.1.2 | Fabric | Both-Side Install**

VoidAuth is a security authentication mod designed for the Minecraft 26.1 series, requiring both server and client installation. Beyond basic login/registration, it architecturally solves long-standing issues in traditional auth mods — flickering,  death-state lockups, and elytra disconnect void traps — while providing full GUI management, fine-grained permissions, and a developer API.

---

## 🚀 Quick Start

1. Install [Fabric Loader](https://fabricmc.net/use/) 0.19.2+ and [Fabric API](https://modrinth.com/mod/fabric-api)
2. Place VoidAuth in **both server and client** `mods` folders
3. Start the server — config auto-generates at `config/voidauth.json`
4. Players join and type `/voidauth register <password>` to register, then `/voidauth login <password>` each visit
5. Type `/voidauth help` to see all commands (clickable!)

> 💡 Uses JSON storage by default — zero dependencies, works out of the box. For SQLite/MySQL, just drop the driver jar into the mods folder.

---

## 📋 Requirements

| Item | Requirement |
|------|-------------|
| Minecraft | **26.1 / 26.1.1 / 26.1.2** |
| Java | **25** |
| Fabric Loader | **0.19.2+** |
| Fabric API | **0.145.1+26.1** |

> ⚠️ **Mandatory both-side install** — Clients without VoidAuth will be rejected and prompted to install it.

---

## ✨ Unique Features at a Glance

These features are not found in other Minecraft authentication mods:

| # | Feature | One-Liner |
|---|---------|-----------|
| 🔒 | 4-Layer Position Lock | Client input freeze + server position sync + move packet intercept + deviation correction — eliminates flickering and X-ray |
| 🌑 | Darkness Anti-Bypass | Darkness effect + forced gamma=0 + F3 screen block — even Fullbright can't see through |
| 🪂 | Elytra Disconnect Protection | Auto-loads 7×7 chunks on join, prevents void trap after elytra disconnect |
| 💀 | Death-State Auto-Recovery | Detects death on logout and auto-respawns on rejoin — no admin intervention needed |
| ⏱️ | Flight Check Grace Period | Configurable grace period after login suppresses fly-hack detection during falling/elytra glide |
| 🛡️ | BCrypt Truncation Guard | Validates password UTF-8 length ≤ 72 bytes, closes the silent BCrypt truncation vulnerability |
| 🔐 | Sequential Character Detection | Blocks abc/123/cba/321 patterns — rejects weak passwords |
| 📝 | Admin Audit Log | All admin actions auto-logged, one-click export with timestamp |
| 🔑 | Fine-Grained Permission Nodes | 7 independent permission nodes, supports Fabric Permissions API |
| 🔌 | Developer API | Callback system + data queries + exception isolation for third-party mod integration |
| 🔄 | Auto Migration | Config and data auto-migrate on upgrade or storage type switch |
| 🛡️ | MySQL Password Protection | Config sync uses placeholder `***`, prevents password leak via network |
| ⚙️ | bcryptCostFactor Validation | Server enforces range 10–14, prevents DoS via malicious cost factor |

---

## 🔍 Unique Features in Detail

### 🔒 4-Layer Position Lock — Eliminates Login Flickering & X-Ray

Traditional auth mods only intercept move packets server-side, leaving the client free to move — causing flickering and X-ray. VoidAuth implements four coordinated layers:

1. **ClientInputMixin** — Freezes all keyboard/mouse input
2. **tickFreeze() + LockPositionPacket** — Locks client to server-specified position, zero velocity + no gravity + render interpolation lock
3. **ServerPlayHandlerMixin** — Intercepts all move and vehicle move packets
4. **ServerPlayerEntityMixin** — Actively corrects position deviation, resets fly-check counters

Key innovation: A dedicated `LockPositionPayload` network packet syncs the server-side coordinates to the client, preventing block-embedding caused by position desync.

### 🌑 Darkness Anti-Bypass

Traditional mods use Blindness, which Fullbright resource packs easily bypass. VoidAuth uses triple protection:

- **Darkness effect** — Cannot be penetrated even with max brightness
- **Client gamma forced to 0** — Locks render brightness during login, restores after
- **F3 debug screen blocked** — Prevents coordinate and entity info leaks

### ⏱️ Flight Check Grace Period

After login, players may still be falling or gliding with elytra. Traditional mods immediately trigger fly-hack detection and kick them. VoidAuth provides a configurable grace period (`flightCheckGracePeriodSeconds`) that suppresses flight checks, letting players land safely.

### 🪂 Elytra Disconnect Protection

When a player disconnects mid-elytra-flight and rejoins, traditional mods leave them stuck in the void (chunks not loaded). VoidAuth proactively loads a 7×7 chunk area around the player on join.

### 💀 Death-State Auto-Recovery

Players who die and disconnect get stuck in a "dead" state on rejoin with traditional mods. VoidAuth records the death state on logout and auto-executes respawn on rejoin.

### 🛡️ BCrypt 72-Byte Truncation Guard

BCrypt silently truncates passwords beyond 72 bytes. Attackers can exploit this prefix collision to access other accounts. VoidAuth validates UTF-8 byte length and rejects passwords exceeding 72 bytes.

### 🔐 Sequential Character Detection

Beyond requiring letters + digits, VoidAuth detects ascending/descending sequences (e.g. `abc`, `123`, `cba`, `321`) to reject weak passwords.

### 📝 Admin Audit Log

All admin actions are automatically logged to `logs/voidauth-admin.log`. Export a timestamped copy with `/voidauth exportlog`.

### 🔑 Fine-Grained Permission Nodes

| Permission Node | Action |
|----------------|--------|
| `voidauth.admin.reset` | Reset player password |
| `voidauth.admin.setpassword` | Force-set password |
| `voidauth.admin.delete` | Delete player account |
| `voidauth.admin.list` | View player list |
| `voidauth.admin.reload` | Reload config |
| `voidauth.admin.config` | Modify config |
| `voidauth.admin.exportlog` | Export admin log |

Falls back to OP check when Fabric Permissions API is not installed.

### 🔌 Developer API & Callback System

```java
VoidAuthAPI.registerCallback(new AuthCallbackAdapter() {
    @Override
    public void onAuthenticated(UUID uuid) { /* Login success */ }
});

Set<UUID> authenticated = VoidAuthAPI.getAuthenticatedPlayers();
Optional<PlayerData> data = VoidAuthAPI.getPlayerData(uuid);
```

- `AuthCallbackAdapter` provides empty implementations for selective override
- Callback exceptions are auto-isolated — one failing callback won't affect others

### 🔄 Auto Migration

- **Config**: Automatic version migration on mod upgrade — no lost settings
- **Data**: Automatic migration when switching storage type (JSON → SQLite → MySQL)

### 🛡️ MySQL Password Protection

Config sync packets replace the MySQL password with `***`. The server only updates the password when the client sends a non-empty, non-placeholder value.

### ⚙️ bcryptCostFactor Validation

Server enforces BCrypt cost factor range 10–14, preventing DoS via maliciously high values.

---

## 🎮 Full Feature List

### Player Commands

| Command | Description | Click Behavior |
|---------|-------------|----------------|
| `/voidauth login <password>` | Login | Fills command, type password |
| `/voidauth register <password>` | Register | Fills command, type password |
| `/voidauth info` | View personal info | Click to execute |
| `/voidauth changepassword` | Open change password GUI | Click to execute |
| `/voidauth email` | Open email management GUI | Click to execute |
| `/voidauth unregister` | Delete account (requires confirmation) | Fills command, confirm after prompt |
| `/voidauth help` | Show help menu | Click to execute |

### Admin Commands

| Command | Description | Permission | Click Behavior |
|---------|-------------|-----------|----------------|
| `/voidauth reset <player>` | Reset player password | `voidauth.admin.reset` | Fills command, type player name |
| `/voidauth setpassword <player> <password>` | Force-set password | `voidauth.admin.setpassword` | Fills command, type name & password |
| `/voidauth list [page]` | View registered players | `voidauth.admin.list` | Click to execute |
| `/voidauth delete <player>` | Delete player account | `voidauth.admin.delete` | Fills command, type player name |
| `/voidauth reload` | Reload config | `voidauth.admin.reload` | Click to execute |
| `/voidauth config` | Open config GUI | `voidauth.admin.config` | Click to execute |
| `/voidauth exportlog` | Export admin log | `voidauth.admin.exportlog` | Click to execute |

> 💡 All commands in the help menu are clickable. Commands requiring parameters auto-fill into the chat bar.

### GUI Screens

- **Login Screen** — With countdown timer, auto-kick on timeout
- **Register Screen** — Optional email binding during registration
- **Change Password Screen** — Old + new + confirm password fields
- **Email Management Screen** — Bind / change / unbind modes
- **Config Management Screen** — Visual editor for all settings
- **Account Management Screen** — Unified entry point

### Security

- BCrypt password hashing (configurable cost factor 10–14)
- Password strength validation (min 6 chars + letters & digits + sequential char detection + 72-byte cap)
- Login failure lockout (configurable attempts & duration)
- Same-IP auto-login (configurable validity period)
- Account deletion with confirmation + effect cleanup + data purge

### Data Storage

| Storage | Description | Dependency |
|---------|-------------|------------|
| **JSON** | Default, zero-dependency | None |
| **SQLite** | For small-medium servers | Install sqlite-jdbc in mods folder |
| **MySQL** | For large servers / multi-server | Install mysql-connector-j in mods folder |

> Data auto-migrates when switching storage type. Falls back to JSON with a warning if driver is missing.

---

## 🔧 Optional Dependencies

| Dependency | Purpose | Installation |
|------------|---------|-------------|
| [sqlite-jdbc](https://github.com/xerial/sqlite-jdbc) | SQLite storage | Place jar in mods folder |
| [mysql-connector-j](https://github.com/mysql/mysql-connector-j) | MySQL storage | Place jar in mods folder |
| [Fabric Permissions API](https://github.com/lucko/fabric-permissions-api) | Fine-grained permissions | Place in mods folder |

Auto-fallback to defaults (JSON storage + OP permission check) when not installed. Core features unaffected.

---

## ⚙️ Configuration

| Setting | Default | Description |
|---------|---------|-------------|
| `loginTimeoutSeconds` | 120 | Login timeout (seconds) |
| `maxPasswordAttempts` | 5 | Max password attempts |
| `lockDurationMinutes` | 30 | Account lock duration (minutes) |
| `bcryptCostFactor` | 10 | BCrypt cost factor (10–14) |
| `ipAutoLoginEnabled` | true | Enable same-IP auto-login |
| `ipAutoLoginDurationHours` | 24 | Auto-login validity (hours) |
| `storageType` | json | Storage type (json/sqlite/mysql) |
| `flightCheckGracePeriodSeconds` | 30 | Flight check grace period (seconds) |
| `mysqlHost` | localhost | MySQL host |
| `mysqlPort` | 3306 | MySQL port |
| `mysqlDatabase` | voidauth | MySQL database name |
| `mysqlUsername` | root | MySQL username |
| `mysqlPassword` | | MySQL password |
| `mysqlPoolSize` | 5 | MySQL connection pool size |

---

## 📜 Developer API

```java
// Add dependency
dependencies {
    modImplementation "com.voidauth:voidauth:1.0.0"
}

// Register callback
VoidAuthAPI.registerCallback(new AuthCallbackAdapter() {
    @Override
    public void onAuthenticated(UUID uuid) { /* Login success */ }
});

// Data queries
Optional<PlayerData> data = VoidAuthAPI.getPlayerData(uuid);
Set<UUID> authenticated = VoidAuthAPI.getAuthenticatedPlayers();
boolean isAuth = VoidAuthAPI.isAuthenticated(uuid);
```

---

## ❓ FAQ

**Q: Can players join without VoidAuth installed?**
A: No. Clients without VoidAuth are rejected and prompted to install it.

**Q: Do I need to login in singleplayer?**
A: No. The host player auto-authenticates in singleplayer (including LAN).

**Q: How do I switch storage type?**
A: Change `storageType` in `config/voidauth.json` and restart. Existing data auto-migrates.

**Q: What if I forget my password?**
A: Admins can use `/voidauth reset <player>` to reset it.

**Q: Which Minecraft versions are supported?**
A: 26.1, 26.1.1, and 26.1.2. Not compatible with older versions.

---

## 📄 License

© 2025 VoidAuth. All Rights Reserved.

This mod is closed-source software. All rights are reserved. Without prior written authorization from the author, the following are prohibited:

- Decompiling, reverse engineering, or disassembling this software
- Modifying, adapting, or creating derivative works
- Distributing, re-licensing, or selling the source code of this software
- Removing or altering any copyright notices in this software

Users are granted a non-exclusive, non-transferable license to install and use this software solely for personal, non-commercial purposes.
