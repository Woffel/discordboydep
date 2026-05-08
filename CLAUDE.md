# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

This is an **AMP (Application Management Panel) module** for deploying Node.js Discord bots. It contains no bot code — only AMP configuration files that define how AMP downloads, installs, configures, and runs a Discord bot on a managed server.

## File roles and relationships

| File | Purpose |
|------|---------|
| `discordbot.kvp` | Root config file AMP reads; references all other JSON files via `@IncludeJson[filename]` |
| `discordbotconfig.json` | User-facing settings UI (download type, Node.js version, npm install mode, env vars, etc.) |
| `discordbotupdates.json` | Ordered update pipeline: App Download → Node.js Download → Install npm Packages → Run App Setup Commands |
| `discordbotstart.json` | Pre-start pipeline: runs `AppPrestartCommands` (default: `npm run deploy-commands`) before each bot start |
| `discordbotports.json` | Port definitions (currently empty — Discord bots don't expose ports) |
| `discordbotmetaconfig.json` | Meta config overrides (currently empty) |
| `manifest.json` | AMP module identity (`id: "discordbot"`) |

## Template variable system

AMP substitutes `{{VariableName}}` placeholders at runtime using values from `discordbotconfig.json`. Special builtins use a `$` prefix: `{{$FullRootDir}}`, `{{$FullBaseDir}}`. Conditions on update/start stages are expressed as `UpdateSourceConditionSetting` + `UpdateSourceConditionValue` pairs that gate which stage runs.

## Update pipeline logic

Stages in `discordbotupdates.json` run sequentially on install/update. Each stage specifies `UpdateSourcePlatform` (`Linux`, `Windows`, or `All`) so the same JSON covers both OSes. The Git repo download stage handles both first-time clone and subsequent pulls in a single shell command.

## Key config values in discordbot.kvp

- `App.RootDir` = `./discordbot-server/` — AMP's working root
- `App.BaseDirectory` = `./discordbot-server/app/` — where the bot code lives after download
- `App.CommandLineArgs` = `{{NodeArgs}} {{ApplicationName}} {{CommandLineArgs}}` — how Node.js is invoked
- `App.EnvironmentVariables` injects Node.js binary into PATH at runtime
- `App.SmartExcludeExemptions` lists config file extensions preserved across updates
- `Meta.MinAMPVersion` = `2.6.0.0` — minimum AMP version required
- `Meta.ExtraContainerPackages` = `["python-is-python3", "make", "g++"]` — needed for native npm modules (e.g. `node-gyp`)
