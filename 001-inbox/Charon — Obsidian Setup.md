---
type: reference
status: active
created: 2026-10-17
source:
tags:
  - obsidian
  - knowledge-management
---
## Purpose

Establish a local, Git-backed Obsidian knowledge base for capturing thoughts, maintaining technical documentation and organizing project knowledge.

Charon is the reference implementation for future Windows clients, including Geonosis.
## Architecture

- **Editor:** Obsidian
- **Canonical format:** Markdown
- **Version control:** Git
- **Remote repository:** [castleofbones/knowledge](https://github.com/castleofbones/knowledge)
- **Repository visibility:** Private
- **Local vault:** `C:\src\Obsidian\knowledge`
- **Primary branch:** `main`

Each client will maintain an independent local clone of the repository. Git is responsible for synchronization and version history.

The active vault must not reside inside OneDrive or another cloud-synchronized directory.

## Initial installation

1. Install Obsidian for Windows from [https://obsidian.md/download](https://obsidian.md/download).
2. Create a vault named `knowledge` under `C:\src\Obsidian`.
3. Open PowerShell in the vault directory.
4. Initialize Git:

```powershell
git init -b main
```

5. Create a `.gitignore` file:

```gitignore
# Obsidian device-specific workspace state
.obsidian/workspace.json
.obsidian/workspaces.json

# Operating system files
.DS_Store
Thumbs.db
desktop.ini

# Temporary files
*.tmp
*.bak
```

6. Commit the initial configuration.
7. Create an empty private GitHub repository named `knowledge`.
8. Connect the local repository:

```powershell
git remote add origin https://github.com/castleofbones/knowledge.git
git push -u origin main
```

9. Verify the working tree is clean and `main` tracks `origin/main`.

## Quick capture

Create a folder named `00-inbox`.

Under **Settings → Files and links**, configure:

- Default location for new notes: **In the folder specified below**   
- Folder to create new notes in: `00-inbox`

Pressing `Ctrl+N` creates a new note in the inbox.

The inbox is intended for immediate capture without requiring classification or organization.

## Global keyboard shortcuts

AutoHotkey v2 provides system-wide shortcuts.

| Shortcut         | Behaviour                                               |
| ---------------- | ------------------------------------------------------- |
| `Ctrl+Alt+Space` | Launch or activate Obsidian                             |
| `Ctrl+Alt+N`     | Launch or activate Obsidian and create a new inbox note |

The Windows installation registers Obsidian using application identifier `md.obsidian`.

The AutoHotkey script uses `shell:AppsFolder\md.obsidian` rather than a hardcoded executable path.

### AutoHotkey v2 script

Save as `ObsidianHotkeys.ahk`:

```autohotkey
#Requires AutoHotkey v2.0
#SingleInstance Force

obsidianApp := "shell:AppsFolder\md.obsidian"

ActivateObsidian() {
    global obsidianApp

    if WinExist("ahk_exe Obsidian.exe") {
        WinActivate("ahk_exe Obsidian.exe")
    } else {
        Run('explorer.exe "' obsidianApp '"')
        if !WinWait("ahk_exe Obsidian.exe",, 10)
            return false

        WinActivate("ahk_exe Obsidian.exe")
    }

    return !!WinWaitActive("ahk_exe Obsidian.exe",, 5)
}

^!Space:: {
    ActivateObsidian()
}

^!n:: {
    if ActivateObsidian()
        Send("^n")
}
```

Both shortcuts have been tested successfully on Charon.

### Automatic startup

Place a shortcut to `ObsidianHotkeys.ahk` in the Windows Startup folder, accessible through `Win+R` → `shell:startup`.

Startup execution should be verified after a subsequent Windows sign-in.

## Git workflow

Git operations are currently manual.

Before editing on a secondary client:

```powershell
git pull --ff-only
```

After completing a meaningful set of changes:

```powershell
git status
git add .
git diff --cached
git commit -m "Describe documentation changes"
git push
```

Review staged changes before committing, particularly files inside `.obsidian`.

## Future clients

Geonosis (Surface Pro) will be configured from this reference implementation.

Because the GitHub repository already exists, Geonosis should clone it rather than initialize a new repository.

Device-specific paths, application registration and hotkey configuration must be verified independently.

## Deferred enhancements

- Geonosis installation and synchronization   
- Android capture from Pixel 10 Pro
- External capture inbox and agent-assisted classification
- Documentation templates
- Automated Git synchronization
- AI-assisted knowledge maintenance

## Design principles

- Capture first, organize later.
- Markdown remains the source of truth.
- Keep knowledge accessible independently of Obsidian.
- Prefer simple, inspectable workflows.
- Introduce automation only when its value is demonstrated.
- Preserve human review of agent-generated changes.