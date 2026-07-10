```markdown
# linux-steam-launch-opts

A Linux-only dispatcher for Steam launch options. Offload complex command 
strings into modular, per-game bash configuration files. Manage environment 
variables, command wrappers, and conditional logic for native, Proton, and 
non-Steam titles. Keep your game launch configurations clean, modular, and 
maintainable.

## Installation

1. **Download**: Place the script in a directory in your `$PATH` 
   (e.g., `~/.local/bin/`).
2. **Permissions**:
   ```bash
   chmod +x steam-launch-opts

```

3. **Symlink (Optional, for shorter commands)**:
```bash
ln -s "$(pwd)/steam-launch-opts" ~/.local/bin/slopts

```


4. **Steam Integration**:
* Right-click any game in your Steam Library → **Properties**.
* In **Launch Options**, enter:
```bash
steam-launch-opts %command%
or
slopts %command%
```





## How to Migrate Your Launch Options

If you have existing Steam launch options, they likely look like a long
string of text. This tool replaces that messy string with a clean,
organized file.

### Understanding the Mapping

Steam launch options usually follow this pattern:
`[Wrappers] %command% [Arguments]`

* **Wrappers (PRE)**: Tools that run *before* the game, such as
`gamemoderun` or `mangohud`.
* **Arguments (ARGS)**: Settings passed *to* the game, such as `-novid`,
`-windowed`, or `-vulkan`.

**Example:**
If your current Steam launch string is:
`gamemoderun mangohud %command% -vulkan -novid`

**You would map it to your configuration file like this:**

```bash
# Anything before %command% goes in PRE
PRE=(gamemoderun mangohud)

# Anything after %command% goes in ARGS
ARGS=(-vulkan -novid)

```

## Architecture

Configurations are stored in `~/.config/steam-launch-opts/`. The script
automatically generates a commented stub file the first time a game
is launched.

* `global.conf`: Sourced before every game. Ideal for environment
variables or dynamic logic that applies to your entire library.
* `<appid>-<executable>.conf`: Per-game overrides. Sourced after
`global.conf`.

## Syntax Reference

Configurations are standard bash scripts. You can use any valid bash logic.

| Variable | Description |
| --- | --- |
| `PRE` | Array. Commands/wrappers prepended to the executable |
| `ARGS` | Array. Arguments appended to the executable |

### Basic Usage Example

`~/.config/steam-launch-opts/12345-GameName.conf`:

```bash
# Export variables
export PROTON_ENABLE_WAYLAND=1

# Add wrappers (PRE) and arguments (ARGS)
PRE=(gamemoderun mangohud)
ARGS=(-novid -console)

```

## Advanced Examples

### Dynamic Display Topology

You can run conditional logic (like checking monitor settings via
`kscreen-doctor`) inside `global.conf`.

```bash
# ~/.config/steam-launch-opts/global.conf

# Example: Dynamic frame rate tuning based on display output
if kscreen-doctor -o 2>/dev/null | grep -q "Vrr: Always"; then
    export DXVK_FRAME_RATE=224
else
    export MANGOHUD_CONFIG="vsync=1,fps_limit=120"
fi

```

### Pre-launch Script Execution

You can execute local scripts or commands before the game launches.

```bash
# ~/.config/steam-launch-opts/99999-MMO.conf

# Cache clearing script
if [[ -x "./clear-cache.sh" ]]; then
    ./clear-cache.sh
fi

export PROTON_USE_NTSYNC=1

```

## Debugging

If a game fails to launch, check the log file:

```bash
cat ~/.config/steam-launch-opts/last-launch.log

```

This file records the resolved `appid`, the configuration file sourced,
and the exact final execution command string.

## License

Licensed under the **GPLv3**. See `LICENSE` for details.

## Acknowledgements

The structure, documentation, and logic for this project were developed
with the extensive assistance of large language models. This
project has been manually curated, tested, and maintained to ensure
functionality and reliability.
