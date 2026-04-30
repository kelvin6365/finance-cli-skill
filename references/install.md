# Installing finance-cli

Use this when `finance schema` returns `command not found` / `ENOENT`, or
when the user explicitly asks how to install. Drive the flow yourself —
detect, recommend, run, verify — instead of dumping all four install
methods at once.

## Step 1 — detect the user's environment

Pick the right install method by checking what they have:

```bash
uname -s              # Darwin | Linux  (Windows shows MINGW/CYGWIN/MSYS)
uname -m              # arm64 | aarch64 | x86_64
command -v bun        # exit 0 if Bun is installed
command -v brew       # exit 0 if Homebrew is installed
```

Then pick **one** method and present **only that one** to the user.
Offer alternatives only if they balk.

## Step 2 — pick the method

Decision tree:

1. **Bun ≥ 1.2 already installed** → use the Bun-from-GitHub method.
   Cleanest, no compile, runs from source.
2. **macOS or Linux x86_64, no Bun** → use the one-line install script.
   Downloads the prebuilt binary from GitHub Releases.
3. **Windows** → manual download. The install script doesn't support
   Windows.
4. **Asks for "the official package manager"** → only homebrew tap and
   npm wrappers exist as community options today; recommend the install
   script unless they specifically want one.

### Method A — one-line install script (default for macOS / Linux)

```bash
curl -fsSL https://raw.githubusercontent.com/kelvin6365/finance-cli/main/install.sh | bash
```

What it does:
- Detects OS/arch (Darwin arm64/x64, Linux x86_64).
- Downloads the latest release binary from
  `https://github.com/kelvin6365/finance-cli/releases/latest`.
- Installs to `/usr/local/bin/finance` (uses sudo only if not writable).
- Override location with `INSTALL_DIR=$HOME/.local/bin` exported before
  the curl pipe, e.g. for systems without sudo.

If the user reports "tmp: unbound variable" or any trap-related error,
GitHub's raw CDN is serving a stale cached script — bust it with:

```bash
curl -fsSL "https://raw.githubusercontent.com/kelvin6365/finance-cli/main/install.sh?v=$(date +%s)" | bash
```

### Method B — Bun from GitHub (for Bun users)

```bash
bun add -g github:kelvin6365/finance-cli
```

Runs from source via the user's local Bun. No prebuilt binary needed.
Requires Bun ≥ 1.2. Verify with `bun --version`.

### Method C — manual download (Windows or air-gapped)

1. Go to https://github.com/kelvin6365/finance-cli/releases/latest.
2. Download the asset matching the user's platform:
   - macOS Apple Silicon → `finance-macos-arm64`
   - macOS Intel → `finance-macos-x64`
   - Linux x86_64 → `finance-linux-x64`
   - Windows x86_64 → `finance-windows-x64.exe`
3. `chmod +x` the file (skip on Windows) and move it onto `PATH`
   (e.g. `/usr/local/bin/finance` on Unix, `%USERPROFILE%\bin\` on
   Windows).

## Step 3 — verify the install

Always run both checks before proceeding:

```bash
finance --help              # confirms binary is on PATH
finance schema --json       # confirms it can produce a JSON envelope
```

If `finance --help` works but `schema --json` errors with `ENOENT`, that
means the data file is missing — go to step 4.

If `finance --help` says "command not found", PATH isn't picking up the
install location. The script prints a hint when this happens; tell the
user to add the install dir to PATH:

```bash
echo 'export PATH="/usr/local/bin:$PATH"' >> ~/.zshrc   # or ~/.bashrc
source ~/.zshrc
```

## Step 4 — initialise the data file

```bash
finance init                                  # USD by default
finance init --currency HKD --symbol HK$      # any other currency
finance init --currency JPY --symbol ¥
```

Creates `~/.finance/data.json` from the bundled seed. Idempotent — safe
to run twice; the second run reports `created: false`.

To pick a currency, ask the user *what symbol they want their amounts
formatted with* (e.g. `$`, `HK$`, `€`, `¥`). The ISO code (`USD`, `HKD`,
…) is informational; the symbol is what `money()` actually prints. If
they're unsure, default to USD/$.

## Step 5 — first useful query

After init succeeds, demonstrate the tool by running one read-only
command relevant to whatever the user originally asked. Examples:

- They asked "can I afford X" → `finance afford <X> --json`
- They asked "log this expense" → `finance add ... --json`
- No specific intent → `finance status --json` (month overview)

## Troubleshooting cheat sheet

| Symptom | Likely cause | Fix |
|---|---|---|
| `command not found: finance` | Binary not installed or not on PATH | Run install script; check PATH |
| `tmp: unbound variable` | Stale CDN-cached install script | Cache-bust with `?v=$(date +%s)` |
| `ENOENT` from any command | Data file missing | `finance init` |
| `Permission denied` writing to `/usr/local/bin` | No sudo / unsupported | `INSTALL_DIR=$HOME/.local/bin curl … \| bash`, then add to PATH |
| Install script fails on architecture | Unsupported (e.g. Linux arm64) | Build from source: `git clone …; bun install; bun run build:linux` |
| `bun add -g` errors with "unable to resolve" | Bun < 1.2 | Upgrade Bun: `curl -fsSL https://bun.sh/install \| bash` |
