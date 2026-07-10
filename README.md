
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

   Right-click any game in your Steam Library → **Properties**.
   
   In **Launch Options**, enter:
      ```bash
      steam-launch-opts %command%
      ```
   or (with symlink)
      ```bash
      slopts %command%
      ```



## First-Time Setup & Initialization

This script creates its own environment upon your first launch:

1. **Launch the game** once using the Launch Option string specified above.
2. The script will automatically create the configuration directory at
`~/.config/steam-launch-opts/` and generate the following:
* `global.conf`: Created automatically if it does not exist.
* `<appid>-<executable>.conf`: A stub file generated specifically for
this game.


3. **Configure**: Once the files exist, edit them in your preferred
text editor. The changes apply on the *next* launch.




## How to Migrate Your Launch Options

If you have existing Steam launch options, they likely look like a long
string of text. This tool replaces that messy string with a clean,
organized file.


### Understanding the Mapping

Steam launch options usually follow this pattern:
`[Variables] [Wrappers] %command% [Arguments]`

* **Variables**: Environment variables that define behavior (e.g., `PROTON_ENABLE_NVAPI=1`).
* **Wrappers (PRE)**: Tools that run *before* the game, such as
`gamemoderun` or `mangohud`.
* **Arguments (ARGS)**: Settings passed *to* the game, such as `-novid`,
`-windowed`, or `-vulkan`.

**Example:**
If your current Steam launch string is:
`PROTON_ENABLE_NVAPI=1 gamemoderun mangohud %command% -vulkan -novid`

**You would map it to your configuration file like this:**

```bash
# AppId: 00000000
# Executable: /mnt/games/Games/GameFolder/GameName.exe

# Environment variables
export PROTON_ENABLE_NVAPI=1

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
variables or dynamic logic that applies to your entire game library.
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

### Dynamic Display Variables

You can incorporate conditional logic (such as checking monitor configuration via `kscreen-doctor`) directly within `global.conf` or any game-specific `<appid>-<executable>.conf` file. The following example demonstrates how to dynamically adjust frame rate and VSync settings based on your active display topology.

```bash
# ~/.config/steam-launch-opts/global.conf

# Example: Dynamic frame rate tuning based on display output
if kscreen-doctor -o 2>/dev/null | grep -q "Vrr: Always"; then
    export MANGOHUD_CONFIG="vsync=1,fps_limit=224"
else
    export MANGOHUD_CONFIG="vsync=3,fps_limit=120,fps_limit_method=early"
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
