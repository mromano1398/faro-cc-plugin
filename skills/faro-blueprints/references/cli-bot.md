# Blueprint — CLI / Bot

## CLI Structure
```
src/
├── index.ts                        # Entry point
├── commands/
│   ├── init.ts
│   ├── build.ts
│   └── deploy.ts
├── lib/
│   ├── config.ts                   # Config file handling
│   ├── logger.ts                   # Colored output
│   └── prompts.ts                  # Interactive prompts
├── types/
└── utils/
```

## Bot Structure (Telegram/Discord)
```
src/
├── index.ts                        # Entry point
├── bot/
│   ├── handlers/
│   │   ├── commands.ts
│   │   ├── messages.ts
│   │   └── callbacks.ts
│   ├── middleware/
│   │   ├── auth.ts
│   │   └── rate-limit.ts
│   └── bot.ts                      # Bot instance
├── lib/
│   ├── db.ts
│   └── ai.ts                       # LLM integration
└── types/
```

## Stack CLI
- commander o yargs (parsing argomenti)
- chalk (colori)
- ora (spinner)
- inquirer o prompts (input interattivo)

## Stack Bot
- node-telegram-bot-api o telegraf (Telegram)
- discord.js (Discord)
- Database: SQLite (leggero) o PostgreSQL
