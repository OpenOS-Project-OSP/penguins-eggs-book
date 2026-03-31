![](/media/chapter-6/chapter-6.jpg)

# Chapter 6

# eggs configuration

## Part 1: eggs configuration

You can configure eggs automatically using `sudo eggs dad --default`,
automatically with custom values using `sudo eggs dad --file custom.yaml`,
or interactively using `sudo eggs dad`.

It is also possible to configure eggs by editing the files directly:

- `/etc/penguins-eggs.d/eggs.yaml` — main configuration
- `/etc/penguins-eggs.d/tools.yaml` — tools configuration

## Part 2: manual configuration

During installation, eggs generates configuration files in:

```
/etc/penguins-eggs.d
```

### eggs.yaml

The main configuration file. Example values:

```yaml
force_installer: false
initrd_img: /boot/initrd.img-6.8.8-1-pve
make_efi: true
make_isohybrid: true
make_md5sum: false
root_passwd: evolution
snapshot_basename: father
snapshot_dir: /home/eggs/
snapshot_excludes: /etc/penguins-eggs.d/exclude.list
snapshot_mnt: /home/eggs/.mnt/
snapshot_prefix: egg-of_debian-bookworm-
user_opt: live
user_opt_passwd: evolution
vmlinuz: /boot/vmlinuz-6.8.8-1-pve
```

### /etc/penguins-eggs.d/exclude.list

Built from templates in `/etc/penguins-eggs.d/exclude.list.d/`. Available
templates:

- `master.list`
- `homes.list`
- `usr.list`
- `var.list`

These templates are combined each time `eggs produce` runs. To prevent
rebuilding the exclude list on every run, use the `--static` flag.

## eggs status

`eggs status` prints a summary of the current configuration values.

## love.yaml — automated workflow configuration

`eggs love` reads a `love.yaml` file to run the full eggs workflow
automatically. This is the configuration format for the `eggs love` command
(see Chapter 5, Method 7).

A minimal `love.yaml`:

```yaml
# love.yaml — eggs love configuration
produce:
  basename: my-distro
  prefix: egg-of_
  theme: default
  compression: fast   # fast | standard | max | pendrive
  clone: false
  homecrypt: false
  fullcrypt: false
```

With recovery tools:

```yaml
produce:
  basename: rescue-distro
  recovery: true
  recovery-gui: minimal
```

For ARM/RISC-V targets:

```yaml
produce:
  basename: my-arm-distro
  dtbdir: /usr/lib/linux-image-6.6.0/broadcom
```

## eggs-ai configuration (~/.eggs-ai.yaml)

[eggs-ai](https://github.com/Interested-Deving-1896/eggs-ai) uses
`~/.eggs-ai.yaml` to configure LLM providers. Generate a sample file with:

```
eggs-ai providers init
```

Example configuration:

```yaml
default_provider: gemini

providers:
  # Any OpenAI-compatible API
  - name: deepseek
    type: custom
    baseUrl: https://api.deepseek.com/v1
    envKey: DEEPSEEK_API_KEY
    model: deepseek-chat

  # Local LM Studio
  - name: lmstudio
    type: custom
    baseUrl: http://localhost:1234/v1
    model: local-model

  # Alias a built-in provider with a preset model
  - name: claude
    type: anthropic
    model: claude-sonnet-4-20250514
    envKey: ANTHROPIC_API_KEY
```

Built-in providers (no config needed, just set the environment variable):

| Provider | Environment variable | Default model |
|---|---|---|
| `gemini` | `GEMINI_API_KEY` | gemini-2.0-flash |
| `openai` | `OPENAI_API_KEY` | gpt-4o-mini |
| `anthropic` | `ANTHROPIC_API_KEY` | claude-sonnet-4-20250514 |
| `mistral` | `MISTRAL_API_KEY` | mistral-large-latest |
| `groq` | `GROQ_API_KEY` | llama-3.3-70b-versatile |
| `ollama` | *(none — local)* | llama3.2 |

eggs-ai auto-detects the provider by checking environment variables in order
and falls back to Ollama for fully offline operation.

See Chapter 11 for the full eggs-ai guide.

## eggs-gui daemon configuration

[eggs-gui](https://github.com/Interested-Deving-1896/eggs-gui) communicates
via a Unix socket at `/tmp/eggs-gui.sock`. The daemon reads the standard
`eggs.yaml` and `tools.yaml` files — no separate daemon config is required.

To start the daemon:

```
make run-daemon
```

The daemon exposes JSON-RPC methods for `config.*`, `eggs.*`, `wardrobe.*`,
`system.*`, and `iso.*` operations. All GUI frontends (TUI, desktop, web)
connect to this socket.

See Chapter 12 for the full eggs-gui guide.
