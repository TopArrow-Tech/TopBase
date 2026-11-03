# Runtime choice and finding it on this machine

Read this when a new project starts, when you write a plan, or when you need to run Node, PHP, or Python locally. Skip it for ordinary feature work.

## How to choose a runtime

Set `runtime.type` in `manifest.json` first. **Do not default to PHP.** If the user named a runtime, use that one.

| Use | Fits |
| --- | --- |
| **Node** | Small games, Canvas, lots of images, audio, or level files, WebSocket. A game like Gold Miner uses Node, not PHP. |
| **PHP** | Admin systems, forms, reports, and CRUD. |
| **Python** | Scripts, data processing, and small local HTTP tools with a few third-party libraries. The platform provides the interpreter. Dependencies live in the app's own `vendor/`. |
| **Electron** | Desktop capabilities such as system file dialogs, printing, or native windows. An ordinary web page or small game does not need Electron. |

On Windows, PHP handles requests one at a time. A small game that pulls many assets on the first screen will stall, and tuning will not get around that.
A static site only needs a few dozen lines of Node to serve files.

The `manifest.json` in this template is the PHP shape. For Node, Python, or Electron, replace `entry.public_dir` with `entry.main` (the js or py file to run, or the Electron app root).

## Finding TopArrow's Node, PHP, Python, and SQLite on this machine

`node`, `php`, and `python` are usually **not** on the system PATH. Do not treat that as "not installed", and do not download another copy. Do not `npm install sqlite3` or `better-sqlite3`.
TopArrow ships the runtimes. **Do not look them up on PATH.**

The default install path is `C:\Program Files\TopArrow\`. The user may install on another drive (for example `D:\TopArrow`). Use the real install location on this machine. Do not hard-code `C:\Program Files`.

Find the program root in this order (the root contains `runtime\`, `tactl.exe`, and `TopArrow.exe`):

1. The environment variable `TOPARROW_RUNTIME_DIR` (injected when TopArrow launches the app; its parent is the program root).
2. `InstallLocation` in the uninstall registry (written by the installer, so a custom directory still works):
   `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\TopArrow`
   If that key is missing, check the same name under `HKCU`. The value may include quotes; strip them.
3. If TopArrow is running, the directory of the `TopArrow.exe` process is the program root.
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
$node = Join-Path $root 'runtime\node\node.exe'
```

Then use `$node`, not a `node` from the system. Next to it are `runtime\php\php.exe`, `runtime\python\python.exe`, `runtime\electron\electron.exe`, and `tactl.exe`. Self-check: `& (Join-Path $root 'tactl.exe') runtime`.

SQLite is built into the runtimes. It is not a separate install:

- Node: `DatabaseSync` from `require('node:sqlite')` (platform Node 24; no flag and no native module).
- PHP: the `pdo_sqlite` and `sqlite3` extensions are already enabled.
- Python: the standard-library `sqlite3` module (platform Python 3.13).

Probe Node + SQLite (`$node` is the path found above):

```powershell
& $node -e "const {DatabaseSync}=require('node:sqlite'); const db=new DatabaseSync(':memory:'); console.log(db.prepare('select sqlite_version() as v').get());"
```

During development, run scripts with the `node.exe`, `php.exe`, or `python.exe` you found. When the app actually runs, TopArrow starts it and injects `TOPARROW_*`.
User data still lives in `%LOCALAPPDATA%\TopArrow\`, no matter which drive the program is installed on.

## Extra rules for Python

- **Do not ship `python.exe`, and do not create a `.venv`.** Only the platform may provide the interpreter.
- **The platform does not include pip.** On the development machine, run `pip install --target vendor <package>` and put the unpacked site-packages into `vendor/` at the app root. Do not drop raw `.whl` files in there.
- Packages with native extensions must be `cp313` + `win_amd64` (built on Windows against this same Python 3.13). A wheel built for another platform will fail to import on the user's machine.
- The platform starts the app as `python.exe -X utf8 -B toparrow_bootstrap.py <app root> <entry.main>`. Do not set `PYTHONPATH` yourself.
- After editing a `.py` file, stop briefly and start the app again. Do not add a bundler.
