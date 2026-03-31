# Chapter 12

# eggs-gui: unified GUI for penguins-eggs

[eggs-gui](https://github.com/Interested-Deving-1896/eggs-gui) is a unified
graphical frontend for penguins-eggs. It merges three existing projects
([pengui](https://github.com/pieroproietti/pengui),
[eggsmaker](https://github.com/pieroproietti/eggsmaker), and
[eggsmaker fork](https://github.com/jlendres/eggsmaker)) into one system with a
shared Go backend and three frontend options.

## Frontends

| Frontend | Framework | Use case |
|----------|-----------|----------|
| **TUI** | BubbleTea (Go) | Terminal power users, SSH sessions |
| **Desktop** | NodeGUI (Qt6/TypeScript) | Native desktop with CSS styling |
| **Web** | NiceGUI (Python) | Remote or headless access via browser |

All frontends connect to a single Go daemon via JSON-RPC over a Unix socket
(`/tmp/eggs-gui.sock`).

## Architecture

```
Frontend (TUI / Desktop / Web)
        │
   JSON-RPC over Unix socket (/tmp/eggs-gui.sock)
        │
   eggs-daemon (Go)
        │
   penguins-eggs CLI
```

The daemon exposes JSON-RPC methods grouped by domain:

- `config.*` — read and write `eggs.yaml` and `tools.yaml`
- `eggs.*` — run eggs commands (produce, dad, tools, etc.)
- `wardrobe.*` — browse and apply costumes and accessories
- `system.*` — system info (distro, kernel, disk, eggs version)
- `iso.*` — ISO management (list, copy to USB, delete)
- `ai.*` — AI assistant methods (proxied to eggs-ai server)

## Features

- Produce ISOs with full option control (prefix, basename, compression, theme, excludes, clone)
- AUTO mode — one-click prepare + produce workflow
- Dad configuration editor (`eggs.yaml`)
- Tools configuration editor (`tools.yaml`)
- Wardrobe browser — costumes, accessories, servers
- Calamares installer management
- PPA, Skel, Yolk tools
- ISO copy to USB/directory with progress
- Version display (eggs, Calamares, distro)
- i18n support (es, en, pt, it)
- AI assistant panel (via eggs-ai HTTP API)

## Installation

### Prerequisites

- Go 1.22+ (daemon and TUI — required)
- Node.js 20+ (desktop frontend — optional)
- Python 3.11+ (web frontend — optional)

### Build

```
git clone https://github.com/Interested-Deving-1896/eggs-gui
cd eggs-gui
make all
```

Build individual components:

```
make daemon      # Go backend daemon
make tui         # BubbleTea terminal UI
make desktop     # NodeGUI desktop app (requires: cd desktop && npm install)
make web         # Instructions for web frontend
```

## Running

The daemon must be running for any frontend to work.

```
# Terminal 1: start daemon
make run-daemon

# Terminal 2: pick a frontend
make run-tui       # Terminal UI
make run-desktop   # Native desktop
make run-web       # Web UI at http://localhost:8080
```

Or use the combined shortcut (starts daemon + TUI):

```
make run
```

## TUI frontend

The BubbleTea TUI is the default frontend. It runs in any terminal and works
over SSH. Navigate with arrow keys, select options with Enter, and use keyboard
shortcuts shown at the bottom of each screen.

Main screens:

- **Dashboard** — system status, eggs version, last ISO info
- **Produce** — ISO build options with live flag preview
- **Config** — `eggs.yaml` editor
- **Wardrobe** — costume browser
- **Tools** — skel, yolk, clean, PPA management
- **AI** — eggs-ai chat panel (requires eggs-ai server running)

## Desktop frontend

The NodeGUI desktop app provides a native Qt6 window with CSS styling. It
connects to the same daemon as the TUI.

```
cd desktop
npm install
npm start
```

## Web frontend

The NiceGUI web frontend runs a Python server accessible from any browser. It
is useful for managing penguins-eggs on a headless or remote machine.

```
cd web
pip install -r requirements.txt
python main.py
```

Access at `http://localhost:8080`.

## Integration with eggs-ai

When eggs-ai is running as an HTTP API server, eggs-gui connects to it
automatically and adds an AI panel to all frontends:

```
# Terminal 1: eggs-gui daemon
make run-daemon

# Terminal 2: eggs-ai API server
eggs-ai serve --port 3737

# Terminal 3: any frontend
make run-tui
```

The AI panel lets you ask questions, run diagnostics, and get build guidance
without leaving the GUI.

## Credits

- [pengui](https://github.com/pieroproietti/pengui) by Piero Proietti — PySide6 GUI
- [eggsmaker](https://github.com/pieroproietti/eggsmaker) by Jorge Luis Endres — customtkinter GUI
- [eggsmaker fork](https://github.com/jlendres/eggsmaker) by Jorge Luis Endres — enhanced + web UI
