# Local git: use the git that ships with TopArrow

`AGENTS.md` says to commit locally when a change is done, with TopArrow's own git first. Read this before the first git command in a task, and when a commit fails because the user name or email is missing.

## Which git to use

1. **TopArrow's git.** TopArrow ships its own git, and TopArrow itself uses only that one on the app's repository (for example, for the baseline tag of an installed app). Use the same one, so both sides run the same git.
2. **The system `git`**, only when TopArrow is not installed on this machine (for example, the project came from GitHub and the user has no TopArrow).
3. **Neither**: tell the user. Do not download or install git.

On a Mac, `/usr/bin/git` exists on every machine, but without the Xcode Command Line Tools it only opens an install prompt. If `git --version` fails or opens that prompt, treat it as no git. Do not install the Command Line Tools for the user.

## Windows

TopArrow's git (MinGit) is at:

`<TopArrow program root>\runtime\git\cmd\git.exe`

With the default install this is `C:\Program Files\TopArrow\runtime\git\cmd\git.exe`. The user may install on another drive (for example `D:\TopArrow`). Use the real install location on this machine. Do not hard-code `C:\Program Files`.

Find the program root in this order (the root contains `runtime\`, `tactl.exe`, and `TopArrow.exe`):

1. The environment variable `TOPARROW_RUNTIME_DIR` (its parent is the program root).
2. `InstallLocation` under `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\TopArrow`. If that key is missing, check the same name under `HKCU`. The value may include quotes; strip them.
3. If TopArrow is running, the directory of the `TopArrow.exe` process.
4. If none of the above exist, try the default path `C:\Program Files\TopArrow`.

```powershell
$root = $null
if ($env:TOPARROW_RUNTIME_DIR) { $root = Split-Path $env:TOPARROW_RUNTIME_DIR -Parent }
if (-not $root) {
  foreach ($k in @(
    'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\TopArrow',
    'HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\TopArrow'
  )) {
    $loc = (Get-ItemProperty $k -ErrorAction SilentlyContinue).InstallLocation
    if ($loc) { $root = $loc.Trim('"'); break }
  }
}
if (-not $root) {
  $p = Get-Process TopArrow -ErrorAction SilentlyContinue | Select-Object -First 1
  if ($p) { $root = Split-Path $p.Path -Parent }
}
if (-not $root) { $root = 'C:\Program Files\TopArrow' }
$git = Join-Path $root 'runtime\git\cmd\git.exe'
if (Test-Path $git) { $env:Path = (Split-Path $git) + ';' + $env:Path }
git --version
```

When TopArrow's git exists, this puts it in front of PATH for the current terminal session only, so plain `git` is TopArrow's. When it does not exist, plain `git` stays the system one. Do not change the system or user PATH variable.
If your tool starts a new shell for every command, run the snippet again, or call TopArrow's git by its full path: `& $git status`.

You can also self-check with `& (Join-Path $root 'tactl.exe') runtime`. The output has a `git  ok` line.

## macOS

TopArrow's git is at `<TopArrow program root>/runtime/git/bin/git`. The program root is `TopArrow.app/Contents/Resources`, normally `/Applications/TopArrow.app/Contents/Resources`. The user may have moved the app (for example to `~/Applications`).

Find the program root in this order:

1. The environment variable `TOPARROW_RUNTIME_DIR` (its parent is the program root).
2. If TopArrow is running, the path of its `TopArrow.app/Contents/MacOS/TopArrow` (or `TopArrowAgent`) process.
3. If neither exists, try `/Applications/TopArrow.app` and `~/Applications/TopArrow.app`.

```sh
root=""
if [ -n "$TOPARROW_RUNTIME_DIR" ]; then root="$(dirname "$TOPARROW_RUNTIME_DIR")"; fi
if [ -z "$root" ]; then
  exe="$(ps -axo comm= | grep -m1 -E '/TopArrow\.app/Contents/MacOS/TopArrow(Agent)?$')"
  if [ -n "$exe" ]; then root="${exe%/MacOS/*}/Resources"; fi
fi
for app in /Applications/TopArrow.app "$HOME/Applications/TopArrow.app"; do
  if [ -z "$root" ] && [ -d "$app/Contents/Resources/runtime" ]; then root="$app/Contents/Resources"; fi
done
git_bin="$root/runtime/git/bin/git"
if [ -x "$git_bin" ]; then export PATH="$(dirname "$git_bin"):$PATH"; fi
git --version
```

As on Windows, this changes PATH for the current shell only. If your tool starts a new shell for every command, run the snippet again, or call `"$git_bin"` directly.
Self-check: `"$root/../MacOS/tactl" runtime`. The output has a `git` line.

## How to use it

- After the snippet, use plain `git` for every git command in this task: `git status`, `git diff`, `git log -5`, `git add ...`, `git commit -m "..."`, `git init`.
- Always pass the commit message with `-m`. Do not run commands that open an editor (for example `commit` without `-m`, or `rebase -i`). TopArrow's git on Windows has no editor and no pager at all.

## Commit identity (name and email)

From the project root, check whether this repository already has an identity:

```sh
git config user.name
git config user.email
```

If both have values, use them and do not change them. If either is missing, fill in both with these rules:

- **Name**: default to the TopArrow user name. It lives in the `settings` table of `platform.sqlite` in TopArrow's user data directory (Windows: `%LOCALAPPDATA%\TopArrow\`; Mac: `~/Library/Application Support/TopArrow/`). The row with `key = 'account.profile'` has a JSON `value`; use its `nickname`.
- **Not signed in to TopArrow** (after sign-out that row's `value` is an empty string; the row may also be missing, or `nickname` may be empty), or TopArrow is not installed: use `TAU` followed by 10 random digits, for example `TAU2045491692`.
- **Email**: `<name>@users.noreply.toparrow.local`, for example `zhangsan@users.noreply.toparrow.local`.
  If the name contains Chinese, spaces, or other characters an email address cannot hold (only ASCII letters, digits, `_`, `-`, and `.` between them are allowed), use `TAU` followed by the TopArrow user id (the `id` in the same JSON) before the `@`. For example, with nickname `张三` and id `1234`, the name stays `张三` and the email is `TAU1234@users.noreply.toparrow.local`.

Write them to this repository only (`--local`). Do not touch `--global` or `--system`. Once is enough; later commits reuse it. This is also what keeps a generated `TAU` name stable, so do not generate a new one for each commit.

Open `platform.sqlite` read-only and never write to it. The scripts below read it with TopArrow's Node (`$root` is the program root found above; run them from the project root). When TopArrow is not installed, skip the script and write the `TAU` name and its email by hand with `git config --local`.

Windows (PowerShell):

```powershell
if (-not (git config user.name) -or -not (git config user.email)) {
  $node = Join-Path $root 'runtime\node\node.exe'
  [Console]::OutputEncoding = [Text.Encoding]::UTF8
  $js = @'
const { DatabaseSync } = require('node:sqlite');
const os = require('node:os');
const path = require('node:path');
const dataRoot = process.platform === 'darwin'
  ? path.join(os.homedir(), 'Library', 'Application Support', 'TopArrow')
  : path.join(process.env.LOCALAPPDATA || '', 'TopArrow');
let name = '';
let id = '';
try {
  const db = new DatabaseSync(path.join(dataRoot, 'platform.sqlite'), { readOnly: true });
  const row = db.prepare("select value from settings where key = 'account.profile'").get();
  if (row && row.value) {
    const profile = JSON.parse(row.value);
    name = String(profile.nickname || '').trim();
    id = String(profile.id ?? '').trim();
  }
} catch (e) {}
const random = 'TAU' + Array.from({ length: 10 }, () => Math.floor(Math.random() * 10)).join('');
if (!name) name = random;
const local = /^[A-Za-z0-9_-]+(\.[A-Za-z0-9_-]+)*$/.test(name) ? name : (id ? 'TAU' + id : random);
console.log(name);
console.log(local + '@users.noreply.toparrow.local');
'@
  $out = @($js | & $node -)
  git config --local user.name $out[0].Trim()
  git config --local user.email $out[1].Trim()
}
```

macOS (zsh or bash):

```sh
if [ -z "$(git config user.name)" ] || [ -z "$(git config user.email)" ]; then
  out="$("$root/runtime/node/bin/node" - <<'EOF'
const { DatabaseSync } = require('node:sqlite');
const os = require('node:os');
const path = require('node:path');
const dataRoot = process.platform === 'darwin'
  ? path.join(os.homedir(), 'Library', 'Application Support', 'TopArrow')
  : path.join(process.env.LOCALAPPDATA || '', 'TopArrow');
let name = '';
let id = '';
try {
  const db = new DatabaseSync(path.join(dataRoot, 'platform.sqlite'), { readOnly: true });
  const row = db.prepare("select value from settings where key = 'account.profile'").get();
  if (row && row.value) {
    const profile = JSON.parse(row.value);
    name = String(profile.nickname || '').trim();
    id = String(profile.id ?? '').trim();
  }
} catch (e) {}
const random = 'TAU' + Array.from({ length: 10 }, () => Math.floor(Math.random() * 10)).join('');
if (!name) name = random;
const local = /^[A-Za-z0-9_-]+(\.[A-Za-z0-9_-]+)*$/.test(name) ? name : (id ? 'TAU' + id : random);
console.log(name);
console.log(local + '@users.noreply.toparrow.local');
EOF
)"
  git config --local user.name "$(printf '%s\n' "$out" | sed -n 1p)"
  git config --local user.email "$(printf '%s\n' "$out" | sed -n 2p)"
fi
```
