# Local git: use TopArrow's own git when this machine has none

`AGENTS.md` says to commit locally when a change is done. Read this when `git` cannot be found on this machine, or when a commit fails because the user name or email is missing. Skip it if git already commits normally.

## Where TopArrow's git is

If `git` is not on the system PATH, do not skip the commit because of that, and do not download and install git. TopArrow ships its own git (MinGit) at:

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
& $git --version
```

You can also self-check with `& (Join-Path $root 'tactl.exe') runtime`. The output has a `git  ok` line.

## How to use it

- Run every git command in this task as `& $git ...`: `& $git status`, `& $git diff`, `& $git log -5`, `& $git add ...`, `& $git commit -m "..."`, `& $git init`.
- Or put it on PATH for the current terminal session only: `$env:Path = (Split-Path $git) + ';' + $env:Path`, then use plain `git`. Do not change the system or user PATH variable.
- This git has no editor and no pager. Always pass the commit message with `-m`. Do not run commands that open an editor (for example `commit` without `-m`, or `rebase -i`).

## Commit identity (name and email)

From the project root, check whether this repository already has an identity:

```powershell
& $git config user.name
& $git config user.email
```

If both have values, use them and do not change them. If either is missing, fill in both with these rules:

- **Name**: default to the TopArrow user name. It lives in the `settings` table of `%LOCALAPPDATA%\TopArrow\platform.sqlite`: the row with `key = 'account.profile'` has a JSON `value`; use its `nickname`.
- **Not signed in to TopArrow** (after sign-out that row's `value` is an empty string; the row may also be missing, or `nickname` may be empty): use `TAU` followed by 10 random digits, for example `TAU2045491692`.
- **Email**: `<name>@users.noreply.toparrow.local`, for example `zhangsan@users.noreply.toparrow.local`.
  If the name contains Chinese, spaces, or other characters an email address cannot hold (only ASCII letters, digits, `_`, `-`, and `.` between them are allowed), use `TAU` followed by the TopArrow user id (the `id` in the same JSON) before the `@`. For example, with nickname `张三` and id `1234`, the name stays `张三` and the email is `TAU1234@users.noreply.toparrow.local`.

Write them to this repository only (`--local`). Do not touch `--global` or `--system`. Once is enough; later commits reuse it. This is also what keeps a generated `TAU` name stable, so do not generate a new one for each commit.

Open `platform.sqlite` read-only and never write to it. This reads it with TopArrow's Node (`$root` and `$git` are the values found above; run it from the project root):

```powershell
if (-not (& $git config user.name) -or -not (& $git config user.email)) {
  $node = Join-Path $root 'runtime\node\node.exe'
  [Console]::OutputEncoding = [Text.Encoding]::UTF8
  $js = @'
const { DatabaseSync } = require('node:sqlite');
const path = require('node:path');
let name = '';
let id = '';
try {
  const db = new DatabaseSync(path.join(process.env.LOCALAPPDATA, 'TopArrow', 'platform.sqlite'), { readOnly: true });
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
  & $git config --local user.name $out[0].Trim()
  & $git config --local user.email $out[1].Trim()
}
```
