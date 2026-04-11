# CLI Patterns — Faro

Pattern completi per CLI, bot e script di automazione.

## CLI NODE.JS + COMMANDER

### Setup

```bash
npm init -y
npm install commander chalk ora inquirer conf
npm install -D typescript @types/node tsx tsup
```

`package.json`:
```json
{
  "name": "faro-cli",
  "version": "1.0.0",
  "bin": {
    "faro": "./dist/index.js"
  },
  "files": ["dist"],
  "scripts": {
    "build": "tsup src/index.ts --format esm --dts",
    "dev": "tsx src/index.ts",
    "prepublishOnly": "npm run build"
  },
  "engines": {
    "node": ">=18"
  }
}
```

### Entry point

`src/index.ts`:
```ts
#!/usr/bin/env node
import { Command } from 'commander'
import chalk from 'chalk'
import { initCommand } from './commands/init.js'
import { deployCommand } from './commands/deploy.js'

const program = new Command()

program
  .name('faro')
  .description('CLI Faro per automazioni progetto')
  .version('1.0.0')

program
  .command('init')
  .description('Inizializza un nuovo progetto Faro')
  .option('-t, --template <name>', 'template da usare', 'default')
  .action(initCommand)

program
  .command('deploy')
  .description('Deploy del progetto')
  .option('-e, --env <env>', 'environment', 'staging')
  .action(deployCommand)

program.parseAsync().catch((error) => {
  console.error(chalk.red('Errore:'), error.message)
  process.exit(1)
})
```

### Comando con spinner

`src/commands/deploy.ts`:
```ts
import ora from 'ora'
import chalk from 'chalk'
import { exec } from 'node:child_process'
import { promisify } from 'node:util'

const execAsync = promisify(exec)

export async function deployCommand(options: { env: string }) {
  const spinner = ora('Building...').start()

  try {
    await execAsync('npm run build')
    spinner.text = 'Uploading...'
    await execAsync(`vercel deploy --prod=${options.env === 'production'}`)
    spinner.succeed(chalk.green(`Deploy su ${options.env} completato!`))
  } catch (error) {
    spinner.fail(chalk.red('Deploy fallito'))
    console.error(error)
    process.exit(1)
  }
}
```

### Config persistente

```ts
import Conf from 'conf'

const config = new Conf<{ apiKey?: string; team?: string }>({
  projectName: 'faro-cli',
})

export function setApiKey(key: string) {
  config.set('apiKey', key)
}

export function getApiKey() {
  return config.get('apiKey') ?? process.env.FARO_API_KEY
}
```

### Interattività

```ts
import inquirer from 'inquirer'

export async function initCommand() {
  const answers = await inquirer.prompt([
    {
      type: 'input',
      name: 'projectName',
      message: 'Nome progetto:',
      validate: (v) => v.length > 0,
    },
    {
      type: 'list',
      name: 'template',
      message: 'Template:',
      choices: ['next-app', 'astro-blog', 'cli-tool'],
    },
    {
      type: 'confirm',
      name: 'git',
      message: 'Inizializzare git?',
      default: true,
    },
  ])

  console.log(chalk.green('Setup in corso...'), answers)
}
```

## PUBBLICAZIONE NPM

```bash
# Dry run: verifica cosa verrebbe pubblicato
npm publish --dry-run

# Login
npm login

# Pubblica
npm publish --access public
```

## CLI PYTHON + TYPER

```bash
pip install typer[all] rich
```

`src/faro_cli/main.py`:
```python
import typer
from rich.console import Console
from rich.table import Table

app = typer.Typer(help="CLI Faro Python")
console = Console()


@app.command()
def init(
    nome: str = typer.Argument(..., help="Nome progetto"),
    template: str = typer.Option("default", "--template", "-t"),
):
    """Inizializza un nuovo progetto"""
    console.print(f"[green]Init[/green] {nome} con template [cyan]{template}[/cyan]")


@app.command()
def list_projects():
    """Lista progetti locali"""
    table = Table(title="Progetti")
    table.add_column("Nome", style="cyan")
    table.add_column("Path")
    table.add_row("faro-cli", "/tmp/faro-cli")
    console.print(table)


if __name__ == "__main__":
    app()
```

`pyproject.toml`:
```toml
[project]
name = "faro-cli"
version = "0.1.0"

[project.scripts]
faro = "faro_cli.main:app"
```

Installa in dev:
```bash
pip install -e .
faro --help
```

## BOT DISCORD — discord.js v14

```bash
npm install discord.js @discordjs/builders dotenv
```

`src/index.ts`:
```ts
import 'dotenv/config'
import {
  Client,
  GatewayIntentBits,
  REST,
  Routes,
  SlashCommandBuilder,
  Events,
} from 'discord.js'

const client = new Client({
  intents: [GatewayIntentBits.Guilds, GatewayIntentBits.GuildMessages],
})

const commands = [
  new SlashCommandBuilder()
    .setName('ping')
    .setDescription('Risponde pong'),
  new SlashCommandBuilder()
    .setName('stats')
    .setDescription('Mostra statistiche Faro'),
].map(c => c.toJSON())

const rest = new REST({ version: '10' }).setToken(process.env.DISCORD_TOKEN!)

client.once(Events.ClientReady, async (c) => {
  console.log(`Bot connesso come ${c.user.tag}`)
  await rest.put(
    Routes.applicationCommands(process.env.DISCORD_CLIENT_ID!),
    { body: commands }
  )
})

client.on(Events.InteractionCreate, async (interaction) => {
  if (!interaction.isChatInputCommand()) return

  if (interaction.commandName === 'ping') {
    await interaction.reply('pong')
  } else if (interaction.commandName === 'stats') {
    await interaction.deferReply()
    const stats = await fetchStats()
    await interaction.editReply(`Utenti: ${stats.users}\nProgetti: ${stats.progetti}`)
  }
})

// Graceful shutdown
process.on('SIGINT', () => {
  console.log('Stopping bot...')
  client.destroy()
  process.exit(0)
})
process.on('SIGTERM', () => {
  client.destroy()
  process.exit(0)
})

client.login(process.env.DISCORD_TOKEN)

async function fetchStats() {
  return { users: 42, progetti: 7 }
}
```

## BOT TELEGRAM — Telegraf v4

```bash
npm install telegraf dotenv
```

```ts
import 'dotenv/config'
import { Telegraf, session, Markup } from 'telegraf'

interface Session {
  step?: 'askName' | 'askEmail'
  data?: { name?: string; email?: string }
}

const bot = new Telegraf<{ session: Session }>(process.env.TELEGRAM_TOKEN!)

bot.use(session({ defaultSession: () => ({}) }))

bot.start((ctx) => {
  ctx.reply('Benvenuto in Faro Bot. Usa /nuovo per creare un progetto.')
})

bot.command('nuovo', (ctx) => {
  ctx.session.step = 'askName'
  ctx.session.data = {}
  ctx.reply('Qual è il nome del progetto?')
})

bot.on('text', (ctx) => {
  if (ctx.session.step === 'askName') {
    ctx.session.data!.name = ctx.message.text
    ctx.session.step = 'askEmail'
    ctx.reply('Qual è la tua email?')
  } else if (ctx.session.step === 'askEmail') {
    ctx.session.data!.email = ctx.message.text
    ctx.reply(
      `Creato progetto ${ctx.session.data!.name} per ${ctx.session.data!.email}`,
      Markup.inlineKeyboard([
        Markup.button.callback('Conferma', 'confirm'),
        Markup.button.callback('Annulla', 'cancel'),
      ])
    )
    ctx.session.step = undefined
  }
})

bot.action('confirm', async (ctx) => {
  await ctx.answerCbQuery('Confermato!')
  await ctx.editMessageText('Progetto creato correttamente.')
})

bot.action('cancel', async (ctx) => {
  await ctx.answerCbQuery('Annullato')
  ctx.session = {}
})

bot.catch((err) => console.error('Bot error:', err))

bot.launch()
process.once('SIGINT', () => bot.stop('SIGINT'))
process.once('SIGTERM', () => bot.stop('SIGTERM'))
```

## GITHUB ACTION — Custom Action

`action.yml`:
```yaml
name: 'Faro Deploy'
description: 'Deploy Faro project'
inputs:
  environment:
    description: 'Environment name'
    required: true
    default: 'staging'
  token:
    description: 'Faro API token'
    required: true
runs:
  using: 'node20'
  main: 'dist/index.js'
```

`src/index.ts`:
```ts
import * as core from '@actions/core'
import * as github from '@actions/github'

async function run() {
  try {
    const env = core.getInput('environment')
    const token = core.getInput('token')

    core.info(`Deploying to ${env}`)

    const res = await fetch(`https://api.faro.example/deploy`, {
      method: 'POST',
      headers: { 'Authorization': `Bearer ${token}` },
      body: JSON.stringify({ env, ref: github.context.sha }),
    })

    if (!res.ok) throw new Error(`Deploy failed: ${res.status}`)
    const { url } = await res.json()

    core.setOutput('url', url)
    core.info(`Deployed to ${url}`)
  } catch (error) {
    core.setFailed((error as Error).message)
  }
}

run()
```

## CRON JOB — GitHub Schedule

`.github/workflows/cron.yml`:
```yaml
name: Cleanup sessioni scadute
on:
  schedule:
    - cron: '0 3 * * *' # 3 AM UTC ogni giorno
  workflow_dispatch: # manuale

jobs:
  cleanup:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20 }
      - run: npm ci
      - run: node scripts/cleanup-sessions.js
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
```

`scripts/cleanup-sessions.js`:
```js
import { db } from '../src/lib/db.js'

async function main() {
  const cutoff = new Date(Date.now() - 7 * 24 * 60 * 60 * 1000)
  const { count } = await db.session.deleteMany({
    where: { expiresAt: { lt: cutoff } },
  })
  console.log(`Cancellate ${count} sessioni scadute`)
}

main().catch((e) => {
  console.error(e)
  process.exit(1) // importante per far fallire la action
})
```

## CRON SERVERLESS — Next.js + cron-job.org

```ts
// app/api/cron/cleanup/route.ts
export async function GET(request: Request) {
  const authHeader = request.headers.get('authorization')
  if (authHeader !== `Bearer ${process.env.CRON_SECRET}`) {
    return new Response('Unauthorized', { status: 401 })
  }

  const count = await cleanupExpiredSessions()
  return Response.json({ ok: true, count })
}
```

Configura su cron-job.org:
- URL: `https://app.faro.example/api/cron/cleanup`
- Header: `Authorization: Bearer xxx`
- Schedule: ogni giorno alle 3:00

## TEST CLI

```ts
// tests/cli.test.ts
import { describe, it, expect } from 'vitest'
import { execSync } from 'node:child_process'

describe('faro CLI', () => {
  it('--help ritorna testo', () => {
    const out = execSync('node dist/index.js --help').toString()
    expect(out).toContain('CLI Faro')
  })

  it('init fallisce senza nome', () => {
    expect(() => execSync('node dist/index.js init')).toThrow()
  })
})
```

## RELEASE AUTOMATICA — GitHub Action

```yaml
name: Release
on:
  push:
    tags: ['v*']

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          registry-url: https://registry.npmjs.org
      - run: npm ci
      - run: npm run build
      - run: npm test
      - run: npm publish --access public
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
      - uses: softprops/action-gh-release@v2
        with:
          generate_release_notes: true
```
