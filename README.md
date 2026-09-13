# cTrader CLI — official Docker image

[![GHCR](https://img.shields.io/badge/ghcr.io-spotware%2Fctrader--console-blue?logo=docker&logoColor=white)](https://github.com/spotware/ctrader-console-docker/pkgs/container/ctrader-console)
[![Latest release](https://img.shields.io/github/v/release/spotware/ctrader-console-docker?label=latest%20image)](https://github.com/spotware/ctrader-console-docker/releases)
[![Platforms](https://img.shields.io/badge/platforms-linux%2Famd64%20%7C%20linux%2Farm64-informational)](#supported-platforms)
[![Docs](https://img.shields.io/badge/docs-help.ctrader.com-0b7285)](https://help.ctrader.com/ctrader-cli/)

Run cBots, backtests, optimizations and trading commands on a headless Linux box — no cTrader Desktop,
no GUI, no X server.

```bash
docker run --rm ghcr.io/spotware/ctrader-console:latest --version
```

---

## Quick reference

|                                  |                                                                                                          |
| -------------------------------- | -------------------------------------------------------------------------------------------------------- |
| **Image**                        | `ghcr.io/spotware/ctrader-console`                                                                          |
| **Maintained by**                | [Spotware Systems](https://www.spotware.com/)                                                               |
| **Documentation**                | [help.ctrader.com/ctrader-cli](https://help.ctrader.com/ctrader-cli/)                                       |
| **Full command reference**       | [spotware/CLI-references](https://github.com/spotware/CLI-references) · or `docker run … --commands`         |
| **Where to file image issues**   | [github.com/spotware/ctrader-console-docker/issues](https://github.com/spotware/ctrader-console-docker/issues) |
| **Where to get help**            | [cTrader community forum](https://ctrader.com/forum/) · [Algo docs](https://help.ctrader.com/ctrader-algo/)  |
| **Supported platforms**          | `linux/amd64`, `linux/arm64`                                                                                 |
| **Entrypoint**                   | the CLI itself — pass CLI arguments straight after the image name                                            |
| **Working directory**            | `/app`                                                                                                       |
| **Runs as**                      | `root` (no `USER` set) — see [Running as a non-root user](#running-as-a-non-root-user)                        |
| **Exposed ports**                | none                                                                                                         |
| **Compressed size**              | ≈ 250 MB (amd64), ≈ 240 MB (arm64)                                                                            |

## Supported tags

| Tag                                      | Contents                                                                 |
| ---------------------------------------- | ------------------------------------------------------------------------ |
| `latest`                                 | the newest stable release — currently **5.9.11**                          |
| `5.9.11`, `5.9.10`, `5.9`                | cTrader 5.9 line                                                          |
| `5.7.10`, `5.6.8`, `5.5.17`, `5.5.8`, `5.5`, `5.4` | older stable lines, kept for reproducibility                     |
| `5.10.0-alpha`, `…-alpha`                | pre-release builds — **not for production**                               |

Tags are **immutable except `latest`**: once `5.9.11` is published it always resolves to the same digest.
`latest` moves on every stable release. Pin a version tag in production, or a digest for byte-exact
reproducibility:

```bash
docker pull ghcr.io/spotware/ctrader-console:5.9.11
docker pull ghcr.io/spotware/ctrader-console@sha256:<digest>
```

Every tag is a multi-arch index with build **provenance attestations** attached:

```bash
docker buildx imagetools inspect ghcr.io/spotware/ctrader-console:latest
```

## What is in the image

The image is the complete cTrader Algo runtime for Linux, not just a thin CLI wrapper:

| Component                    | Why it is there                                                               |
| ---------------------------- | ----------------------------------------------------------------------------- |
| `ctrader-cli`                | the CLI itself — every command below                                           |
| .NET runtime **and SDK**     | so `create` and `build` can compile C# algo projects **inside** the container   |
| .NET reference packs         | so `.algo` files targeting older .NET versions still compile                    |
| Python 3.12 + `ash` (busybox) | so Python cBots and indicators run the same way they do in cTrader Cloud       |
| `algohost.netcore`           | the out-of-process algo host that executes your cBot, per architecture          |

> **Version note.** From **5.10** the image moves to .NET 10 on Ubuntu 24.04 and gains a small
> entrypoint wrapper that pins `XDG_DOCUMENTS_DIR` (a container has no `~/Documents`, and without it
> `create`/`build` would scaffold into the wrong place). Behaviour from your side is unchanged: you
> still pass CLI arguments straight after the image name.

---

## Quickstart

### 1. Put your credentials in a file

The CLI reads your cTrader ID password from a file, never from the command line.

```bash
mkdir -p ~/ctrader/algos
printf '%s' 'my-cTrader-ID-password' > ~/ctrader/ctrader-cli.pwd
chmod 600 ~/ctrader/ctrader-cli.pwd
```

The file holds the password on a single line, no trailing spaces. See
[Credentials & security](#credentials--security) before you put it on a shared host.

### 2. List your accounts

```bash
docker run --rm \
  -v ~/ctrader:/mnt/ctrader:ro \
  ghcr.io/spotware/ctrader-console:latest \
  accounts \
    --ctid=user@example.com \
    --pwd-file=/mnt/ctrader/ctrader-cli.pwd
```

### 3. Backtest a cBot

```bash
docker run --rm \
  -v ~/ctrader:/mnt/ctrader \
  ghcr.io/spotware/ctrader-console:latest \
  backtest "/mnt/ctrader/algos/Sample Breakout cBot.algo" \
    --ctid=user@example.com \
    --pwd-file=/mnt/ctrader/ctrader-cli.pwd \
    --account=9102302 \
    --symbol=EURUSD --period=h1 \
    --start=01/01/2026 --end=01/03/2026 \
    --data-mode=m1 \
    --balance=10000 --commission=15 --spread=1 \
    --report=/mnt/ctrader/report.html \
    --report-json=/mnt/ctrader/report.json \
    --exit-on-stop
```

### 4. Run the cBot live

```bash
docker run -d --name my-bot --restart=unless-stopped \
  -v ~/ctrader:/mnt/ctrader \
  ghcr.io/spotware/ctrader-console:latest \
  run "/mnt/ctrader/algos/My bot.algo" \
    --ctid=user@example.com \
    --pwd-file=/mnt/ctrader/ctrader-cli.pwd \
    --account=9102302 \
    --symbol=EURUSD --period=h1 \
    --exit-on-stop
```

```bash
docker logs -f my-bot     # the cBot's Print() output, live
docker stop my-bot        # graceful: SIGTERM reaches the CLI directly
```

---

## How the container sees your files

The CLI needs three kinds of path, and **every one of them must live under a mount** — anything written
elsewhere disappears with the container.

| What                      | Typical container path              | Notes                                                                     |
| ------------------------- | ----------------------------------- | ------------------------------------------------------------------------- |
| `.algo` files             | `/mnt/ctrader/algos/My bot.algo`    | quote the path if it contains spaces                                       |
| password file             | `/mnt/ctrader/ctrader-cli.pwd`      | mount it read-only                                                         |
| reports (`--report`, …)   | `/mnt/ctrader/report.html`          | written by the container, so the mount must be writable                    |
| historical data cache     | `--data-dir=/mnt/ctrader/data`      | **the directory must already exist** — the CLI does not create it          |
| algo projects (`create`)  | `/root/cAlgo/Sources/…`             | scaffolded under the home directory; mount a volume there to keep them     |

A single mount for all of them is the simplest setup:

```bash
-v ~/ctrader:/mnt/ctrader
```

### Cache historical data between runs

Backtests download tick or m1 history on every cold run. Point `--data-dir` at a persistent volume and
repeated runs start in seconds:

```bash
mkdir -p ~/ctrader/data
docker run --rm -v ~/ctrader:/mnt/ctrader ghcr.io/spotware/ctrader-console:latest \
  backtest … --data-dir=/mnt/ctrader/data
```

Add `--redownload-data` once when you want the cache refreshed.

---

## Recipes

> **The CLI has two authentication conventions, and they are not interchangeable.**
>
> - **Batch commands** — `accounts`, `symbols`, `metadata`, `run`, `backtest`, `optimize` — take
>   `--pwd-file`. They reject `--password`.
> - **Interactive-route commands** — everything else: orders, positions, prices, candles, indicators,
>   alerts, `account-stats` — take `--password=…` together with `-q`, which runs one command and exits.
>   They reject `--pwd-file`.
>
> Mixing them produces `Missing --pwd-file` or `Missing --ctid in non-interactive mode`. Full rules:
> [CLI-references → Mode routing](https://github.com/spotware/CLI-references#mode-routing).

All examples assume these shell variables:

```bash
CT="docker run --rm -v $HOME/ctrader:/mnt/ctrader ghcr.io/spotware/ctrader-console:latest"

# batch commands
BATCH="--ctid=user@example.com --pwd-file=/mnt/ctrader/ctrader-cli.pwd --account=9102302"
# interactive-route commands (one-shot)
ONESHOT="--ctid=user@example.com --password=$CTRADER_PASSWORD --account=9102302 -q"
```

### Read-only market and account queries

```bash
$CT periods                                          # no credentials needed
$CT metadata "/mnt/ctrader/algos/My bot.algo"        # parameters of an .algo, no credentials
$CT accounts --ctid=user@example.com --pwd-file=/mnt/ctrader/ctrader-cli.pwd
$CT symbols  $BATCH                                  # tradable symbols

$CT account-stats $ONESHOT                           # balance, equity, margin
$CT price         $ONESHOT --symbol=EURUSD
$CT candles       $ONESHOT --symbol=EURUSD --period=h1 --count=200
$CT indicator history $ONESHOT \
      --indicator="Simple Moving Average" --symbol=EURUSD --period=h1 --count=50
```

### Trade

```bash
$CT order place-market $ONESHOT --symbol=EURUSD --side=buy --volume=1000 --sl=1.0800 --tp=1.0950
$CT positions          $ONESHOT
$CT position close     $ONESHOT --position=123456 --yes
```

`--volume` is in units of the base currency; add `--volume-type=lots` to pass lots instead.

### Optimize (parameter sweep)

`optimize` uses `--timeframe`, not `--period`, and needs a parameter-settings JSON file:

```bash
$CT optimize "/mnt/ctrader/algos/My bot.algo" $BATCH \
    --params=/mnt/ctrader/params.json \
    --symbol=EURUSD --timeframe=h1 \
    --start=01/01/2026 --end=01/06/2026 --data-mode=m1 \
    --method=genetic --cores=4 \
    --criteria=NetProfit:max,MaxEquityDrawdownPercentages:min \
    --optres=/mnt/ctrader/result.optres \
    --passes-dir=/mnt/ctrader/passes
```

Give the container the cores it is allowed to use, or `--cores` will oversubscribe the host:

```bash
docker run --cpus=4 --memory=8g …
```

### Monte Carlo robustness check

Add `--mc-method` to a `backtest` — `--mc-output` is then required:

```bash
$CT backtest "/mnt/ctrader/algos/My bot.algo" $BATCH \
    --symbol=EURUSD --period=h1 --start=01/01/2026 --end=01/06/2026 --data-mode=m1 \
    --mc-method=bootstrap --mc-iterations=1000 --mc-seed=42 \
    --mc-output=/mnt/ctrader/montecarlo.json
```

Every seed — `0` included — is fully reproducible; change it to draw a different simulation.

### Create and build an algo inside the container

The image carries the .NET SDK and Python, so it is a complete build box:

```bash
docker run --rm -v ~/ctrader/cAlgo:/root/cAlgo ghcr.io/spotware/ctrader-console:latest \
  create cbot MyFirstBot csharp

docker run --rm -v ~/ctrader/cAlgo:/root/cAlgo ghcr.io/spotware/ctrader-console:latest \
  build /root/cAlgo/Sources/Robots/MyFirstBot
```

Both print a JSON object — project paths for `create`, a success flag plus compiler diagnostics for
`build` — so they drop straight into CI. Neither needs credentials. Pass `python` as the third
argument to `create` for a Python cBot.

---

## Configuration by environment variable

Every option can come from an environment variable instead of the command line. This is what makes the
image pleasant in Compose, Kubernetes and CI, where secrets arrive as env vars.

**Two things are required:**

1. `--environment-variables` (short form `-e`) **must** be on the command line. Without it the CLI
   ignores the environment entirely.
2. The variable name is the option name with `-` written as `_`, matched case-insensitively.

| Option          | Environment variable |
| --------------- | -------------------- |
| `--ctid`        | `CTID`               |
| `--pwd-file`    | `PWD_FILE`           |
| `--account`     | `ACCOUNT`            |
| `--symbol`      | `SYMBOL`             |
| `--period`      | `PERIOD`             |
| `--start`       | `START`              |
| `--end`         | `END`                |
| `--data-mode`   | `DATA_MODE`          |
| `--report-json` | `REPORT_JSON`        |
| `--exit-on-stop` | `EXIT_ON_STOP`      |

```bash
docker run --rm \
  -v ~/ctrader:/mnt/ctrader \
  -e CTID='user@example.com' \
  -e PWD_FILE='/mnt/ctrader/ctrader-cli.pwd' \
  -e ACCOUNT='9102302' \
  -e SYMBOL='EURUSD' \
  -e PERIOD='h1' \
  -e START='01/01/2026' \
  -e END='01/03/2026' \
  -e DATA_MODE='m1' \
  -e BALANCE='10000' \
  -e REPORT='/mnt/ctrader/report.html' \
  ghcr.io/spotware/ctrader-console:latest \
  backtest "/mnt/ctrader/algos/My bot.algo" --environment-variables --exit-on-stop
```

> **`-e` twice, two different meanings.** Docker's `-e` comes **before** the image name and sets a
> variable; the CLI's `-e` comes **after** it and is the short form of `--environment-variables`. The
> examples spell the CLI flag out in full to keep the two apart.

### cBot parameters from the environment

Your cBot's own parameters are readable from the environment too — but their names are matched
**case-sensitively**, exactly as declared in the `.algo`:

```bash
-e RiskPercent='1.5' -e TakeProfitPips='40'
```

That is deliberate: it is why the container's own `HOME`, `PATH` and `HOSTNAME` can never be mistaken
for a cBot parameter named `Home` or `Path`.

### What each command accepts

Only the options a command actually supports are read from the environment; anything else is ignored.

<details>
<summary><code>run</code></summary>

`CTID` · `PWD_FILE` · `ACCOUNT` · `SYMBOL` · `PERIOD` · `ALGO_FILE` · `CBOT_SET` · `BROKER` ·
`FULL_ACCESS` · `EXIT_ON_STOP` · plus every cBot parameter name.
</details>

<details>
<summary><code>backtest</code></summary>

`CTID` · `PWD_FILE` · `ACCOUNT` · `SYMBOL` · `PERIOD` · `ALGO_FILE` · `CBOT_SET` · `BROKER` ·
`FULL_ACCESS` · `EXIT_ON_STOP` · `START` · `END` · `BALANCE` · `DATA_MODE` · `DATA_FILE` ·
`DATA_DIR` · `COMMISSION` · `COMMISSION_TYPE` · `COMMISSION_AUTO` · `SPREAD` · `EQUITY_X_AXIS` ·
`REPORT` · `REPORT_JSON` · `PRECISE_CONVERSION` · `REDOWNLOAD_DATA` · `MC_METHOD` · `MC_ITERATIONS` ·
`MC_SEED` · `MC_STARTING_EQUITY` · `MC_RUIN_FLOOR` · `MC_BLOCK_SIZE` · `MC_OUTPUT` · plus every cBot
parameter name.
</details>

<details>
<summary><code>optimize</code></summary>

`CTID` · `PWD_FILE` · `ACCOUNT` · `SYMBOL` · `TIMEFRAME` · `ALGO_FILE` · `CBOT_SET` · `BROKER` ·
`FULL_ACCESS` · `EXIT_ON_STOP` · `START` · `END` · `BALANCE` · `DATA_MODE` · `DATA_FILE` ·
`DATA_DIR` · `COMMISSION` · `COMMISSION_TYPE` · `COMMISSION_AUTO` · `SPREAD` · `PARAMS` · `METHOD` ·
`CORES` · `CRITERIA` · `FITNESS` · `AUTO_SELECT_BEST` · `SKIP_EMPTY_PASSES` · `OPTRES` ·
`PASSES_DIR` · `PRECISE_CONVERSION` · `REDOWNLOAD_DATA`.
</details>

<details>
<summary><code>accounts</code> / <code>symbols</code></summary>

`CTID` · `PWD_FILE` · `BROKER` (and `ACCOUNT` for `symbols`).
</details>

---

## Compose, Kubernetes and CI

### Docker Compose — a cBot that stays up

```yaml
services:
  eurusd-breakout:
    image: ghcr.io/spotware/ctrader-console:5.9.11
    restart: unless-stopped
    stop_grace_period: 30s
    command: >
      run "/mnt/ctrader/algos/My bot.algo"
      --environment-variables --exit-on-stop
    environment:
      CTID: user@example.com
      PWD_FILE: /run/secrets/ctrader_password
      ACCOUNT: "9102302"
      SYMBOL: EURUSD
      PERIOD: h1
      RiskPercent: "1.5"
    secrets:
      - ctrader_password
    volumes:
      - ./ctrader:/mnt/ctrader
    logging:
      driver: json-file
      options: { max-size: "10m", max-file: "5" }
    deploy:
      resources:
        limits: { cpus: "1.0", memory: 2G }

secrets:
  ctrader_password:
    file: ./secrets/ctrader-cli.pwd
```

### Kubernetes — a nightly backtest

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: nightly-backtest
spec:
  schedule: "0 2 * * *"
  jobTemplate:
    spec:
      backoffLimit: 1
      template:
        spec:
          restartPolicy: Never
          containers:
            - name: ctrader-cli
              image: ghcr.io/spotware/ctrader-console:5.9.11
              args:
                - backtest
                - /algos/My bot.algo
                - --environment-variables
                - --exit-on-stop
              env:
                - name: CTID
                  value: user@example.com
                - name: PWD_FILE
                  value: /secrets/password
                - name: ACCOUNT
                  value: "9102302"
                - name: SYMBOL
                  value: EURUSD
                - name: PERIOD
                  value: h1
                - name: DATA_MODE
                  value: m1
                - name: REPORT_JSON
                  value: /reports/report.json
              volumeMounts:
                - { name: algos,   mountPath: /algos,   readOnly: true }
                - { name: secrets, mountPath: /secrets, readOnly: true }
                - { name: reports, mountPath: /reports }
              resources:
                requests: { cpu: "500m", memory: 1Gi }
                limits:   { cpu: "2",    memory: 4Gi }
          volumes:
            - { name: algos,   persistentVolumeClaim: { claimName: ctrader-algos } }
            - { name: secrets, secret: { secretName: ctrader-cid, defaultMode: 0400 } }
            - { name: reports, persistentVolumeClaim: { claimName: ctrader-reports } }
```

`START` and `END` are absolute dates, so a scheduled job normally computes them and passes them as
arguments rather than hard-coding them.

### GitHub Actions — backtest every push

```yaml
- name: Backtest
  run: |
    printf '%s' '${{ secrets.CTRADER_PASSWORD }}' > pwd.txt
    docker run --rm -v "$PWD:/w" ghcr.io/spotware/ctrader-console:5.9.11 \
      backtest "/w/MyBot.algo" \
        --ctid='${{ secrets.CTRADER_ID }}' --pwd-file=/w/pwd.txt \
        --account='${{ secrets.CTRADER_ACCOUNT }}' \
        --symbol=EURUSD --period=h1 \
        --start=01/01/2026 --end=01/06/2026 --data-mode=m1 \
        --report-json=/w/report.json --exit-on-stop
    rm -f pwd.txt
```

---

## Operating the container

**Signals and shutdown.** The CLI is PID 1, so `docker stop` delivers `SIGTERM` straight to it and the
cBot shuts down cleanly. Give long-running bots a grace period (`--stop-timeout`, or
`stop_grace_period` in Compose).

**`--exit-on-stop`.** Without it the process keeps running after the cBot stops itself, and a
`restart: unless-stopped` container will never restart. Use it for every unattended run.

**Logs.** Everything the cBot prints goes to stdout — `docker logs`, and whatever log driver you
configure. Cap the size; a chatty cBot fills a disk. The image defines no `HEALTHCHECK`: liveness is
"the process is still running", which `--restart` already covers.

**Resources.** Backtests and optimizations are CPU- and memory-hungry. Set `--cpus`/`--memory` and
match `--cores` to them.

**Time.** Backtest `--start`/`--end` are UTC regardless of the container's timezone.

### Running as a non-root user

The image sets no `USER`, so it runs as `root` by default. To drop privileges, give the process a home
directory it can write to — the CLI keeps its algo tree and caches there:

```bash
docker run --rm \
  --user "$(id -u):$(id -g)" \
  -e HOME=/mnt/ctrader/home \
  -v ~/ctrader:/mnt/ctrader \
  ghcr.io/spotware/ctrader-console:latest \
  backtest …
```

Make sure `~/ctrader` is writable by that uid. The same applies to `--read-only` containers: mount a
writable volume for `HOME`, for `--data-dir` and for every report path.

---

## Credentials & security

- **The password file is plaintext.** Treat it as a secret: `chmod 600`, mount it read-only, keep it
  out of image layers and out of version control. Prefer Docker/Kubernetes secrets over a bind mount.
- **Never bake credentials into a derived image.** `ENV CTID=…` or `COPY pwd.txt` puts them in a layer
  that anyone who pulls the image can read.
- **Avoid `--password=` on the command line** — it lands in shell history, `docker inspect` and `ps`.
  `--pwd-file` exists for this reason.
- **Scope the account.** Use a dedicated trading account for automation.
- **Linux has no algo sandbox.** On Windows `--full-access` is a gate; on Linux there is none, so a
  cBot can do anything the container user can. Omitting `--full-access` only prints a warning and
  delays the start by five seconds — it does not restrict the cBot. Run untrusted algos in a
  locked-down container (dropped capabilities, no host mounts, no host network).
- **Pin by digest** in production so a moved `latest` cannot change what you run.

---

## Troubleshooting

| Symptom                                                                                 | Cause and fix                                                                                                           |
| --------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| `Missing --password or --pwd-file in non-interactive mode.`                               | The password file is not visible inside the container. Check the mount and that `--pwd-file` uses the **container** path. |
| `Missing --ctid in non-interactive mode`                                                  | You used an interactive-route command without `--password` and `-q`, or a batch command without `--pwd-file`.              |
| Options are ignored although the env vars are set                                         | `--environment-variables` is missing from the command line, or the name uses `-` where it needs `_`.                        |
| A cBot parameter from the environment has no effect                                       | Parameter names are **case-sensitive** — match the `.algo` declaration exactly. `metadata <file>` prints the real names.    |
| `Parameter --spread is not allowed with tick data`                                        | `--spread` cannot be combined with `--data-mode=ticks` or `tick-csv`; the spread comes from the tick data itself. Drop it.  |
| Backtest dates rejected                                                                   | `--start`/`--end` are `dd/MM/yyyy [hh:mm]`. The `--from`/`--to` of `candles`/`deals` are `yyyy-MM-dd` or `dd/MM/yyyy`.       |
| `--data-dir` errors out                                                                   | The directory must already exist inside the container; create it on the host before mounting.                               |
| The report file is missing after the run                                                  | It was written to a path outside any mount and died with the container. Point `--report*` inside the mount.                 |
| The container exits immediately after the cBot stops — or never exits                     | That is `--exit-on-stop`: present, the process exits with the cBot; absent, it keeps running.                               |
| `optimize` rejects `--period`                                                             | `optimize` has no `--period`. Use `--timeframe` (comma-separated to sweep several).                                        |
| Exit code `85`                                                                            | The authentication token expired. Re-run; if it persists, re-check the cTID and password file.                              |
| A cBot fails on `arm64` but works on `amd64`                                              | A native dependency inside the algo is x64-only. Rebuild it for `linux-arm64` or run on `linux/amd64`.                     |
| Everything hangs with no output                                                           | Usually the algo host died before producing a report. Re-run with `--exit-on-stop`, check `docker logs`, and reduce the date range — very long tick ranges are the common trigger. |

Getting more detail: `docker run --rm ghcr.io/spotware/ctrader-console:latest --help` for the option
reference and `--commands` for the full command reference. Neither needs credentials.

---

## Supported platforms

`linux/amd64` and `linux/arm64` — the same tag serves both, so an Apple-silicon laptop, a Graviton
instance and an x86 VPS all pull the right image with no flags. Force one with
`--platform=linux/amd64` if an algo needs it.

The CLI is designed to be comfortable on a small VPS; memory and CPU needs scale with the date range
and data mode of a backtest, not with the image.

---

## Links

- [cTrader CLI documentation](https://help.ctrader.com/ctrader-cli/) — setup, how-tos, FAQ, AI-agent integration
- [Full command reference](https://github.com/spotware/CLI-references)
- [cTrader Algo documentation](https://help.ctrader.com/ctrader-algo/) — writing cBots, indicators and plugins
- [Package on GHCR](https://github.com/spotware/ctrader-console-docker/pkgs/container/ctrader-console)
- [Releases](https://github.com/spotware/ctrader-console-docker/releases)
- [cTrader community forum](https://ctrader.com/forum/)

## License

The cTrader CLI is proprietary software of Spotware Systems Ltd and is distributed under the
[cTrader terms of use](https://ctrader.com/terms-of-use/). The image also bundles third-party software
— the .NET runtime and SDK, and CPython — each under its own licence; by pulling the image you agree
to the licence terms of everything it contains.
