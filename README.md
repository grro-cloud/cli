# Grro Cloud CLI

`grro` is the command line tool for [Grro Cloud](https://usegrro.com), the
African-first platform for running web apps, APIs, workers and databases.
Deploy and watch the build, follow logs, roll back, set environment
variables, scale, add domains and databases, take backups, schedule jobs and
invite your team, from your terminal or from CI.

```console
$ grro deploy
Deploying 3f9c2a1b...
   #1 [internal] load build definition from Dockerfile
   #6 [build 4/6] RUN npm ci
   #9 [build 6/6] RUN npm run build
   #12 exporting to image
 ▸  deploying
 ▸  running
Live at https://shop-438824.kigali.usegrro.app
```

- [Install](#install)
- [Quick start](#quick-start)
- [Examples](#examples)
- [Deploy from CI](#deploy-from-ci)
- [Settings, JSON and exit codes](#settings-json-and-exit-codes)
- [Troubleshooting](#troubleshooting)
- [Help and links](#help-and-links)

## Install

Pick one. Every way installs the same program and checks it against its
published SHA-256 checksum.

| How | Command | Works on |
|---|---|---|
| Homebrew | `brew install grro-cloud/tap/grro` | macOS, Linux |
| npm | `npm install -g @grrocloud/cli` | macOS, Linux, Windows (Node 16 or newer) |
| Install script | `curl -fsSL https://usegrro.com/cli/install.sh \| sh` | macOS, Linux |
| PowerShell | `irm https://usegrro.com/cli/install.ps1 \| iex` | Windows |

Check it worked:

```console
$ grro version
grro v1.0.0 (darwin/arm64)
```

**Run it once without installing** (handy in CI or to try it):

```bash
npx @grrocloud/cli version
```

**Download it yourself**: get the file for your computer from the
[latest release](https://github.com/grro-cloud/cli/releases/latest), check it
against `SHA256SUMS`, make it runnable and put it on your PATH.

| File | Computer |
|---|---|
| `grro-darwin-arm64` | Mac with an Apple chip (M1, M2, M3, M4) |
| `grro-darwin-amd64` | Mac with an Intel chip |
| `grro-linux-amd64` | Linux on most PCs and servers |
| `grro-linux-arm64` | Linux on ARM (Raspberry Pi, ARM servers) |
| `grro-windows-amd64.exe` | Windows |

```bash
shasum -a 256 -c SHA256SUMS --ignore-missing
chmod +x grro-darwin-arm64
sudo mv grro-darwin-arm64 /usr/local/bin/grro
```

### Updating

| Installed with | Update with |
|---|---|
| Homebrew | `brew upgrade grro` |
| npm | `npm install -g @grrocloud/cli@latest` |
| Install script, PowerShell or a download | `grro update` |

Not sure? Run `grro update`: it updates itself, or tells you which command to
use.

## Quick start

**1. Sign in.** Your browser opens. Check that the code matches the one in
your terminal and press **Allow**.

```console
$ grro login
Your sign-in code: WXKD-4QPT

Signed in as ada@example.com until 2026-12-12.
```

On a server or over SSH, run `grro login --no-browser` and open the link it
prints on any device.

**2. Pick a project.** List your projects, then link one to this folder so
you don't have to name it in every command.

```console
$ grro projects
$ grro link shop
```

Or create a new one and link it in one go:

```bash
grro projects:create shop --region kigali --link
```

**3. Deploy.** The build log streams to your terminal and the command waits
until the new version is live.

```bash
grro deploy --source https://github.com/you/shop   # the first deploy of a new project
grro deploy                                         # every deploy after that
```

**4. Look at it.**

```bash
grro open --url   # the app in your browser
grro logs -f      # its output, live (Ctrl+C to stop)
grro status       # is it up? traffic, errors and tips
```

## Examples

Every project command uses the folder's linked project. Add `-p NAME` to use
another one: `grro logs -p api`.

### Deploy and watch the build

```bash
grro deploy                              # build the latest commit of the project's repository
grro deploy --ref v1.4.0                 # a branch, tag or commit
grro deploy --source https://github.com/you/monorepo --subpath apps/web
grro deploy --image ghcr.io/you/api:1.5  # deploy a ready image, no build
grro deploy --detach                     # start it and return straight away
grro redeploy                            # build the current source again
grro deploys                             # recent deployments and their status
grro deploys:info                        # the latest one in detail
grro deploys:cancel                      # stop a build that's still running
```

### Logs

```bash
grro logs                       # the app's latest lines
grro logs -f                    # keep following new lines
grro logs -n 500                # the last 500 lines
grro logs --build               # the build log of the latest deployment
grro logs --build --deploy 3f9c2a1b
grro logs -f | grep --line-buffered "POST /checkout"
```

### Environment variables

New variables are secret (nobody can read them back) unless you add
`--plain`. Setting a variable redeploys the app so it gets the new value.

```bash
grro config                                       # list them
grro config:set STRIPE_KEY=sk_live_... MAIL_FROM=hi@shop.com
grro config:set LOG_LEVEL=debug --plain           # readable
grro config:set GREETING="Hello Kigali" --no-redeploy
grro config:get LOG_LEVEL                         # print a plain value
grro config:import .env.production                # every line of a .env file
grro config:unset LOG_LEVEL
```

### Roll back

No build, so it takes seconds.

```bash
grro rollback            # back to the previous deployment that ran
grro rollback 3f9c2a1b   # back to a specific one (from grro deploys)
```

### Scale and settings

```bash
grro scale                     # how many instances run now
grro scale min=1               # always on
grro scale min=0               # sleep when nobody uses it, wake on the next request
grro scale min=1 max=5         # grow to 5 under load
grro settings:set health=/healthz
grro settings:set protocol=h2c # gRPC and HTTP/2 apps
```

### Custom domains

```bash
grro domains:add shop.com              # prints the DNS records to create
grro domains:add www.shop.com
grro domains:check shop.com            # check DNS and HTTPS now
grro domains:primary shop.com          # the others redirect to it
grro domains:www shop.com on           # www.shop.com redirects to shop.com
grro domains
```

HTTPS certificates are set up automatically once the DNS records are in place.

### Databases and backups

Postgres, MySQL, Redis and MongoDB. A new database is attached to your app as
`DATABASE_URL` (or `REDIS_URL`, `MONGODB_URL`) and backed up daily.

```bash
grro db:plans                                    # sizes and prices
grro db:create main --engine postgres
grro db:create cache --engine redis --as REDIS_URL
grro db                                          # the project's databases
grro db:info main
grro db:backup main                              # back it up now
grro db:backups main
grro db:restore 7b1e... --new main-restored      # safe: restores into a new database
grro db:rotate main                              # new password, the app is updated
```

### Scheduled jobs

A job runs a command in a copy of your app, with the same image and
variables.

```bash
grro cron:create cleanup --schedule "0 2 * * *" --command "npm run cleanup"
grro cron:create report --schedule "0 8 * * 1" --command "python report.py" --tz Africa/Lagos
grro cron                  # list jobs
grro cron:run cleanup      # run it now
grro cron:runs cleanup     # recent runs and their output
grro cron:pause cleanup
```

### Your team

```bash
grro members:add ada@example.com --role developer      # deploy, roll back, logs, variables
grro members:add sam@example.com --role viewer         # deployments and logs
grro members:add owner@client.com --role client --weekly-report
grro members
grro teams
```

### Templates

Start WordPress, Next.js, an AI chat app and many more in one command.

```bash
grro templates                          # browse them
grro templates:info wordpress           # what it asks for
grro templates:deploy wordpress --name blog --region kigali --link
```

### Billing

```bash
grro billing            # plan, this month's usage and credit
grro billing:invoices
```

See every command with `grro commands`, and any command's flags and examples
with `grro help <command>`.

## Deploy from CI

In the dashboard, create an **API token** for the project. In your
repository's settings, save it as the secret `GRRO_TOKEN`, and save the
project's name as `GRRO_PROJECT`.

**GitHub Actions**

```yaml
# .github/workflows/deploy.yml
name: Deploy
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm test
      - run: npx @grrocloud/cli deploy --ref "$GITHUB_SHA"
        env:
          GRRO_TOKEN: ${{ secrets.GRRO_TOKEN }}
          GRRO_PROJECT: ${{ secrets.GRRO_PROJECT }}
```

**GitLab CI**

```yaml
# .gitlab-ci.yml (set GRRO_TOKEN and GRRO_PROJECT as masked CI variables)
deploy:
  stage: deploy
  image: alpine:3.20
  script:
    - apk add --no-cache curl
    - curl -fsSL https://usegrro.com/cli/install.sh | sh
    - grro deploy --ref "$CI_COMMIT_SHA"
  only: [main]
```

`grro deploy` waits until the deployment is live and fails the step when the
deployment fails, with the build log in the job output. A project token can
deploy and read its one project only.

If you deploy from CI, turn off push-to-deploy for that project in the
dashboard so each commit isn't deployed twice.

## Settings, JSON and exit codes

**Environment variables grro reads**

| Variable | What it does |
|---|---|
| `GRRO_TOKEN` | Use this token instead of the saved sign-in (CI) |
| `GRRO_PROJECT` | The project, when the folder isn't linked and there's no `-p` |
| `GRRO_CONFIG_DIR` | Keep the sign-in and settings in this folder |
| `GRRO_API` | API address (default `https://api.usegrro.com`) |

**JSON for scripts**: add `--json` to any command.

```bash
grro deploys --json | jq -r '.[0].status'
grro projects --json | jq -r '.[].name'
id=$(grro deploy --detach --json | jq -r .id)
```

**Commands that delete something** ask you to type its name. In a script,
pass it with `--confirm`:

```bash
grro db:delete old-db --confirm old-db
```

**Exit codes**

| Code | Means |
|---|---|
| `0` | It worked |
| `1` | It failed; the message says why |
| `3` | Not signed in, or the sign-in or token expired: run `grro login` |
| `130` | You pressed Ctrl+C |

## Troubleshooting

**"you're not signed in"**, or exit code 3: run `grro login`.
A browser sign-in lasts 30 days. In CI, check that `GRRO_TOKEN` is set and
the token hasn't been revoked.

**"which project?"**: run `grro link NAME` in your app's folder, or add
`-p NAME` to the command. **"no project called ..."**: check the name with
`grro projects`.

**The deploy failed**: read the build log with `grro logs --build`. The last
lines usually say what went wrong (a missing file, a failing test, a wrong
start command). Fix it, push, and run `grro deploy` again. Your previous
version keeps serving traffic until a new one is live.

**The app deployed but doesn't answer**: run `grro logs` for its output and
`grro status` for health checks. Make sure the app listens on the `PORT`
environment variable, and that the health path (`grro settings`) answers 200.

**Typed a command wrong?** grro suggests the closest one:

```console
$ grro deplyo
 ▸  "deplyo" isn't a grro command. Did you mean deploy?
```

**npm install fails with EACCES (permission denied)**: your npm global
folder needs administrator rights. Install with Homebrew or the install
script instead, or
[set npm up to install global packages without sudo](https://docs.npmjs.com/resolving-eacces-permissions-errors-when-installing-packages-globally).

**npm install can't download the program** (behind a proxy or offline): use
the install script, or download the file from the
[latest release](https://github.com/grro-cloud/cli/releases/latest) yourself.

## Help and links

- `grro help`: topics and where to start
- `grro help deploy`: one topic or command, with examples
- `grro commands`: every command
- Full guide: [usegrro.com/docs/cli](https://usegrro.com/docs/cli)
- Command reference: [usegrro.com/docs/cli-reference](https://usegrro.com/docs/cli-reference)
- Releases and downloads: [github.com/grro-cloud/cli/releases](https://github.com/grro-cloud/cli/releases)
- Support: [support@usegrro.com](mailto:support@usegrro.com)

MIT License. Copyright Grro Cloud.
