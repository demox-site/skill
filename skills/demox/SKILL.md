---
name: demox
description: Deploy and manage static websites on the Demox platform using the demox CLI. Use this when the user wants to deploy a static site, build output, or ZIP to Demox, or list, inspect, update, or delete their Demox-hosted websites. Triggers include mentions of "demox", "deploy my site", or pushing a built frontend (dist/, build/, out/) to hosting.
---

# Demox CLI

The `demox` CLI deploys static websites (a directory, a `.zip`, or an `https://…​.zip` URL) to the Demox platform and manages them from the command line.

## Setup

The CLI is published as `@demox-site/cli` and requires Node.js >= 18.

```bash
# Install globally (preferred for repeated use)
npm install -g @demox-site/cli

# Or run without installing
npx @demox-site/cli <command>
```

Verify it is available before relying on it:

```bash
demox --version
```

If `demox` is not found, fall back to `npx @demox-site/cli` for every command in this guide.

## Authentication (do this first)

Every command except `logout`, `status`, and `clean` needs a valid token. Tokens are stored at `~/.demox/token.json` and last 30 days.

**`demox login` opens a browser for OAuth and blocks for up to 5 minutes waiting for the user to complete it. It cannot be completed by the agent.** So:

1. Run `demox status` first to check whether a valid token already exists.
2. If it reports "未登录" (not logged in) or that the token is expired, **ask the user to run `demox login` themselves** (in Claude Code, suggest they type `! demox login`). Do not run `demox login` unattended and wait — it will hang until it times out.
3. Once the user confirms login, continue with the deploy/manage commands.

```bash
demox status   # check login state and token expiry
demox login    # USER runs this — opens browser OAuth on localhost:39897
demox logout   # remove the local token
```

## Deploying

```bash
demox deploy <path> [options]
```

`<path>` can be:
- a **directory** — it is zipped automatically (e.g. `./dist`, `./build`, `./out`)
- a **`.zip` file**
- an **`https://…​.zip` URL** (must end in `.zip`)

Options:
- `-n, --name <name>` — display name. Defaults to the directory/zip base name.
- `-i, --id <id>` — **update an existing site** instead of creating a new one. Omit this to create a new site (a random 8-char ID is generated).

```bash
# New deploy from a build directory
demox deploy ./dist --name my-app

# Update an existing site in place (keeps the same URL)
demox deploy ./dist --id ABC123XY
```

On success it prints the website ID and URL. Capture the ID — it is needed to update or delete the site later.

### Constraints — check before deploying
- **Max upload size is 8 MB** (the zipped payload). If a build directory is larger, the deploy fails. Trim assets, or build a production bundle, before deploying.
- Only directories, `.zip` files, and `.zip` URLs are accepted. A single non-zip file is rejected.
- Always point `deploy` at the **build output** (e.g. `dist/`), not the project root with `node_modules/`.

## Managing sites

```bash
demox list                # list all sites (alias: demox ls)
demox info <websiteId>    # show one site's details (name, URL, path, timestamps)
demox delete <websiteId>  # delete a site (alias: demox rm)
```

**`demox delete` prompts for interactive y/N confirmation by default and will hang in a non-interactive shell.** When deleting on the user's behalf, either:
- confirm with the user first, then run `demox delete <id> --force` (`-f`) to skip the prompt, or
- have the user run it themselves.

Deletion is irreversible.

```bash
demox delete ABC123XY --force
```

## Utility commands

```bash
demox test          # verify auth + API connectivity (lists site count)
demox clean         # delete the local token
demox clean --all   # delete the whole ~/.demox config dir
```

## Configuration (advanced)

Defaults target the public Demox service. Override via environment variables only when pointing at a different deployment:

| Variable | Purpose | Default |
|----------|---------|---------|
| `DEMOX_CLIENT_ID` | OAuth client ID | `demox-mcp-client` |
| `DEMOX_AUTH_URL` | Authorization URL | `https://demox.site/#/mcp-authorize` |
| `DEMOX_API_BASE` | API base URL | `https://demox.site` |
| `DEMOX_CLOUD_FUNCTION_URL` | Deploy/API cloud function endpoint | Tencent SCF endpoint |

## Typical workflow

1. `demox status` — confirm a valid token (else ask the user to `demox login`).
2. Build the site (e.g. `npm run build`) so there is a `dist/` (or equivalent).
3. Verify the output dir is under 8 MB.
4. `demox deploy ./dist --name <name>` for a new site, or add `--id <id>` to update.
5. Report the returned URL to the user; save the website ID for future updates.

## Troubleshooting

- **"认证失败，请重新登录" / AuthError** — token expired or invalid. Ask the user to run `demox login` again.
- **"文件过大 … 最大支持 8MB"** — the zipped output exceeds 8 MB. Reduce bundle size.
- **"路径不存在" / "不支持的文件类型"** — bad `<path>`; ensure it is an existing directory, `.zip`, or `.zip` URL.
- **Command hangs** — likely `demox login` (waiting on browser) or `demox delete` without `--force` (waiting on the confirm prompt).
- **`demox: command not found`** — use `npx @demox-site/cli <command>` instead, or `npm install -g @demox-site/cli`.
