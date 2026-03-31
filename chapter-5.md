![](/media/chapter-5/chapter-5.jpg)

# Chapter 5

# Installing penguins-eggs

penguins-eggs is the package name of the package containing the eggs tool.

There are multiple ways to install penguins-eggs. The most practical is
[get-eggs](https://github.com/pieroproietti/get-eggs), but several alternatives
exist depending on your distribution and preferences.

## Method 1: using `get-eggs` (Arch/Debian/Devuan/Ubuntu)

The `get-eggs` script adds the `penguins-eggs-ppa` repository, the nodesource
repository when needed, and configures penguins-eggs on Arch, Debian, Devuan,
Ubuntu, or their derivatives.

Prerequisite: `git`. Install it first if needed:

```
sudo apt install git       # Debian/Ubuntu/Devuan
sudo pacman -Sy git        # Arch
```

Then:

```
git clone https://github.com/pieroproietti/get-eggs
cd get-eggs
sudo ./get-eggs
```

On Arch Linux, `get-eggs` adds the [chaotic-aur](https://aur.chaotic.cx/)
repository. On Debian-based systems it adds
[penguins-eggs-ppa](https://github.com/pieroproietti/penguins-eggs-ppa). It
also adds the [nodesource repository](https://github.com/nodesource/distributions)
when Node.js > 18 is not available in the distro's own repositories.

Run scripts with root privileges only when you trust the source and understand
what they do.

## Method 2: download the package from SourceForge (Debian/Devuan/Ubuntu)

If you prefer not to add the `penguins-eggs-ppa` repository:

1. Visit the [SourceForge page](https://sourceforge.net/projects/penguins-eggs/files/DEBS/)
   and download the latest `.deb` for your architecture.

2. Install it:

```
sudo dpkg -i eggs-10.0.x-amd64.deb
```

3. Fix any missing dependencies:

```
sudo apt install -f
```

4. Alternatively, use `gdebi` for a graphical install: right-click the `.deb`
   file and select "Open with GDebi Package Installer".

Once installed, you can add the PPA for future updates:

```
sudo eggs tools ppa --add
```

## Method 3: adding chaotic-aur manually (Arch Linux)

Follow the instructions at [aur.chaotic.cx/docs](https://aur.chaotic.cx/docs)
to configure the chaotic-aur repository, then:

```
sudo pacman -Sy penguins-eggs
```

## Method 4: using yay (Arch Linux)

```
sudo pacman -Sy yay
sudo yay -S penguins-eggs
```

yay fetches the package from the AUR and resolves dependencies automatically.

## Method 5: build from AUR (Arch Linux)

```
git clone https://aur.archlinux.org/packages/penguins-eggs
cd penguins-eggs
sudo makepkg -si
```

## Method 6: installing on Manjaro Linux

```
sudo pamac upgrade
sudo pamac install penguins-eggs
```

pamac fetches the package from the Manjaro community repository.

## Method 7: eggs love (automated workflow)

`eggs love` is the simplest path from a running system to a finished ISO. It
reads a `love.yaml` configuration file and runs the full eggs workflow
automatically — no need to call `eggs dad`, `eggs produce`, and related commands
individually.

```
sudo eggs love
```

With options:

```
sudo eggs love --clone
sudo eggs love --homecrypt
sudo eggs love --fullcrypt
sudo eggs love --dtbdir /path/to/dtbs
sudo eggs love --nointeractive
```

See Chapter 6 for the `love.yaml` configuration format.

## Installing eggs-ai (AI assistant)

[eggs-ai](https://github.com/Interested-Deving-1896/eggs-ai) is an optional AI
assistant for penguins-eggs. It requires Node.js 18+.

```
git clone https://github.com/Interested-Deving-1896/eggs-ai
cd eggs-ai
npm install
npm run build
```

Or run directly in development mode:

```
npm run dev -- doctor
npm run dev -- ask "How do I create a minimal rescue ISO?"
```

A Linux installer script is also provided:

```
bash install.sh
```

To configure LLM providers, run:

```
eggs-ai providers init
```

This creates a sample `~/.eggs-ai.yaml`. See Chapter 11 for full eggs-ai
documentation.

## Installing eggs-gui (unified GUI)

[eggs-gui](https://github.com/Interested-Deving-1896/eggs-gui) provides TUI,
desktop, and web frontends for penguins-eggs. Prerequisites vary by frontend:

- Go 1.22+ (daemon and TUI)
- Node.js 20+ (desktop, optional)
- Python 3.11+ (web, optional)

```
git clone https://github.com/Interested-Deving-1896/eggs-gui
cd eggs-gui
make all
```

Start the daemon and a frontend:

```
# Terminal 1
make run-daemon

# Terminal 2 — pick one
make run-tui
make run-desktop
make run-web
```

See Chapter 12 for full eggs-gui documentation.

## Building penguins-eggs from source

Building from source is useful for debugging or reviewing the code directly.

Prerequisites: Node.js > 18 and pnpm.

On Debian/Devuan/Ubuntu, configure the NodeSource repository if Node.js > 18 is
not available. On Arch: `sudo pacman -Syu nodejs`. On Manjaro:
`sudo pamac install nodejs pnpm devel-base`.

Install pnpm globally:

```
sudo npm i pnpm -g
```

Clone and build:

```
git clone https://github.com/pieroproietti/penguins-eggs
cd penguins-eggs
sudo pnpm install
```

Run directly from source:

```
sudo ./eggs produce --verbose
```

Build a `.deb` package:

```
cd penguins-eggs
pnpm deb
```

The package will be under `./perrisbrewery/work...`.

# Updating penguins-eggs

**Debian/Devuan/Ubuntu** (with penguins-eggs-ppa configured):

```
sudo apt update && sudo apt upgrade
```

**Arch / Manjaro** (with chaotic-aur or community repo):

```
sudo pacman -Syu
```

![image30.jpg](media/image30.jpg)
