# File Structure Templates

Use these templates when organizing or auditing project structure (Phase 5).
Pick the template that matches the project type. Adapt, don't force.

---

## Table of Contents
1. [Node.js / Express API](#1-nodejs--express-api)
2. [Discord.js Bot (v14 Modular)](#2-discordjs-bot-v14-modular)
3. [Telegram Bot (Node.js)](#3-telegram-bot-nodejs)
4. [SA-MP / Pawn Gamemode](#4-sa-mp--pawn-gamemode)
5. [Vanilla Web App (HTML/CSS/JS)](#5-vanilla-web-app-htmlcssjs)
6. [Python Script / Bot](#6-python-script--bot)
7. [General Rules](#7-general-rules)

---

## 1. Node.js / Express API

```
project/
├── src/
│   ├── routes/          # Express route handlers (one file per resource)
│   │   ├── auth.js
│   │   ├── users.js
│   │   └── index.js     # aggregates all routes
│   ├── controllers/     # Business logic (called by routes)
│   │   ├── authController.js
│   │   └── userController.js
│   ├── models/          # DB schemas / ORM models
│   │   └── User.js
│   ├── middleware/      # Express middleware
│   │   ├── auth.js
│   │   └── errorHandler.js
│   ├── services/        # External API clients, complex logic
│   │   └── emailService.js
│   ├── utils/           # Pure helper functions (no side effects)
│   │   └── formatDate.js
│   └── app.js           # Express app setup (no listen() here)
├── config/
│   ├── database.js
│   └── constants.js
├── tests/
│   └── users.test.js
├── .env                 # secrets — never commit
├── .env.example         # template with empty values — always commit
├── .gitignore
├── package.json
└── server.js            # entry point: imports app.js, calls listen()
```

**Key rules:**
- `server.js` only does: `app.listen(port)` — nothing else
- Routes call controllers; controllers call services/models
- No DB calls in routes directly
- Environment config in `config/` not scattered in files

---

## 2. Discord.js Bot (v14 Modular)

```
bot/
├── src/
│   ├── commands/
│   │   ├── admin/
│   │   │   └── ban.js
│   │   ├── general/
│   │   │   ├── help.js
│   │   │   └── ping.js
│   │   └── registration/
│   │       └── register.js
│   ├── events/
│   │   ├── ready.js
│   │   ├── interactionCreate.js
│   │   └── messageCreate.js
│   ├── handlers/        # Auto-loaders for commands/events
│   │   ├── commandHandler.js
│   │   └── eventHandler.js
│   ├── database/
│   │   ├── connection.js
│   │   └── queries/
│   │       ├── userQueries.js
│   │       └── guildQueries.js
│   ├── utils/
│   │   ├── embeds.js    # reusable embed builders
│   │   ├── permissions.js
│   │   └── logger.js
│   └── index.js         # entry point
├── deploy-commands.js   # slash command registration script
├── .env
├── .env.example
├── .gitignore
└── package.json
```

**Each command file exports:**
```js
module.exports = {
  data: new SlashCommandBuilder()...,
  async execute(interaction) { ... }
};
```

**Each event file exports:**
```js
module.exports = {
  name: 'interactionCreate',
  once: false,
  async execute(interaction, client) { ... }
};
```

**Key rules:**
- Commands auto-loaded by category directory
- No logic in `index.js` — delegate to handlers
- Database queries isolated in `database/queries/`
- Embed builders centralized in `utils/embeds.js`

---

## 3. Telegram Bot (Node.js)

```
bot/
├── src/
│   ├── commands/
│   │   ├── start.js
│   │   ├── help.js
│   │   └── balance.js
│   ├── callbacks/       # Inline keyboard callbacks
│   │   └── confirmPayment.js
│   ├── scenes/          # Multi-step conversations (telegraf)
│   │   └── registerScene.js
│   ├── services/
│   │   └── ivasmsService.js   # External SMS API wrapper
│   ├── database/
│   │   └── index.js
│   ├── utils/
│   │   ├── keyboard.js  # Keyboard builders
│   │   └── messages.js  # Message templates
│   └── bot.js           # Bot initialization + middleware
├── .env
├── .env.example
├── .gitignore
└── package.json
```

**Key rules:**
- Each command in its own file, registered in `bot.js`
- Inline keyboards built by `utils/keyboard.js` (reusable)
- Message templates in `utils/messages.js` (not hardcoded in handlers)
- External API calls wrapped in `services/` (never inline in commands)

---

## 4. SA-MP / Pawn Gamemode

```
gamemode/
├── gamemodes/
│   └── MainGamemode.pwn    # Main file with includes & entry
├── includes/
│   ├── core/
│   │   ├── PlayerManager.inc
│   │   ├── VehicleManager.inc
│   │   └── Database.inc
│   ├── systems/
│   │   ├── Economy.inc
│   │   ├── Inventory.inc
│   │   ├── Housing.inc
│   │   └── Faction.inc
│   ├── commands/
│   │   ├── AdminCmds.inc
│   │   ├── PlayerCmds.inc
│   │   └── VehicleCmds.inc
│   └── utils/
│       ├── Strings.inc
│       ├── Colors.inc
│       └── Dialogs.inc
├── scriptfiles/
│   └── config.cfg
├── filterscripts/          # Optional addons
├── plugins/
├── .gitignore
└── README.md
```

**Key rules:**
- One system per `.inc` file
- `MainGamemode.pwn` only contains `#include` statements and callback forwarding
- All constants (colors, dialog IDs) in dedicated files
- Never hardcode dialog IDs — use `#define DIALOG_REGISTER 1`
- Commands file never contains business logic — calls functions from system includes
- All MySQL queries in `Database.inc`, not scattered across files

**MainGamemode.pwn pattern:**
```pawn
#include <a_samp>
#include <a_mysql>
#include <zcmd>

#include "includes/utils/Colors.inc"
#include "includes/utils/Strings.inc"
#include "includes/core/Database.inc"
#include "includes/core/PlayerManager.inc"
#include "includes/systems/Economy.inc"
// ... etc

main() {}

public OnGameModeInit() {
    DB_Connect();
    PlayerManager_Init();
    return 1;
}
```

---

## 5. Vanilla Web App (HTML/CSS/JS)

```
app/
├── index.html
├── pages/
│   ├── login.html
│   ├── chat.html
│   └── profile.html
├── css/
│   ├── base.css         # reset, variables, typography
│   ├── components.css   # buttons, inputs, cards
│   ├── layout.css       # grid, flexbox, page structure
│   └── pages/
│       ├── login.css
│       └── chat.css
├── js/
│   ├── api.js           # all fetch() calls here
│   ├── auth.js          # login/logout/session logic
│   ├── utils.js         # pure helpers (formatDate, escapeHtml)
│   └── pages/
│       ├── login.js
│       └── chat.js
├── assets/
│   ├── images/
│   └── icons/
└── .gitignore
```

**Key rules:**
- CSS variables defined in `:root` in `base.css`, used everywhere else
- Each page has its own JS file — no mega `main.js`
- All API calls in `api.js` — never `fetch()` inline in page scripts
- `utils.js` is pure functions only — no DOM access, no API calls

---

## 6. Python Script / Bot

```
project/
├── src/
│   ├── __init__.py
│   ├── bot.py           # entry point
│   ├── handlers/        # Message/command handlers
│   │   └── start.py
│   ├── services/
│   │   └── database.py
│   └── utils/
│       └── helpers.py
├── tests/
│   └── test_handlers.py
├── .env
├── .env.example
├── .gitignore
├── requirements.txt     # pip freeze > requirements.txt
└── README.md
```

**Key rules:**
- `requirements.txt` always present and up to date
- Never hardcode credentials — use `python-dotenv`
- One module per file — no 500-line monoliths
- `if __name__ == '__main__':` only in entry point file

---

## 7. General Rules

### Files to ALWAYS have at project root
| File | Purpose |
|------|---------|
| `.gitignore` | Exclude node_modules, .env, build/, *.pyc |
| `.env.example` | Template for env vars (no real values) |
| `README.md` | How to install, configure, and run |
| `package.json` / `requirements.txt` | Dependency manifest |

### Files to NEVER commit
```
node_modules/
.env
*.log
dist/ or build/
__pycache__/
*.pyc
.DS_Store
```

### Naming Anti-Patterns (fix these when found)
| ❌ Bad | ✅ Good |
|--------|--------|
| `utils.js` (generic dump) | `formatters.js`, `validators.js` |
| `index.js` (everything inside) | Separate files per concern |
| `temp.js`, `test2.js` | Delete or rename properly |
| `FINAL_v3_REAL.pwn` | Use git tags instead |
| `helpers/helpers.js` | `utils/stringUtils.js` |
| Numbered files: `command1.js` | Named by purpose: `banCommand.js` |

### When to Split a File
Split any file that:
- Exceeds **300 lines** (Pawn: 500 lines)
- Contains **more than 2 exported functions/classes** with unrelated purposes
- Has **multiple sections** separated by large block comments (`// ====`)
- Is imported by **every other file** in the project (extract shared core)
