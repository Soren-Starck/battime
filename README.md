# battime

Battery time remaining, power draw and health for Mac laptops, in the terminal.

```
$ battime
83% · on battery · 7h 48m left (macOS estimate)
Drawing 8.7 W from the battery (12.50 V, 699 mA)
Health: Normal · maximum capacity 100% · 93 cycles of 1000 rated
```

The battery menu in macOS shows a percentage. macOS still computes a time estimate, and `pmset -g batt` prints it.
battime puts that estimate next to the live power draw and the battery's health, in one line each.

It is a single zsh script. No dependencies, no root, no network access, nothing written to disk.

## Install

```sh
mkdir -p ~/.local/bin
curl -fsSL https://raw.githubusercontent.com/Soren-Starck/battime/main/battime -o ~/.local/bin/battime
chmod +x ~/.local/bin/battime
```

Make sure `~/.local/bin` is on your `PATH` (add `export PATH="$HOME/.local/bin:$PATH"` to `~/.zshrc` if it is not).
Or put it anywhere else on your `PATH`; it is one file. Read it before you run it: it is about 130 lines.

## Usage

```sh
battime            # one summary
battime --watch    # refresh every 5 seconds; battime --watch 2 for every 2 seconds
battime --json     # machine-readable output
```

`--json` prints one object:

```json
{"percent":83,"state":"discharging","external_power":false,"time_remaining_minutes":468,"time_source":"pmset",
 "battery_time_to_empty_minutes":377,"power_watts":8.7,"voltage_mv":12501,"current_ma":-699,"cycle_count":93,
 "design_cycle_count":1000,"condition":"Normal","maximum_capacity_percent":100}
```

`time_remaining_minutes` is `null` while macOS is still calculating, and when the Mac is plugged in and not charging.
`battery_time_to_empty_minutes` is `null` while charging, while plugged in and not charging, and while the battery has
no estimate yet. `condition` and `maximum_capacity_percent` are `null` if System Information does not report them.
`current_ma` is negative while the battery discharges and positive while it charges.

## Where each number comes from

| Shown | Source |
|---|---|
| Percentage, state, time left or time to full | `pmset -g batt`: the estimate macOS computes from your current load |
| `battery_time_to_empty_minutes` (JSON only) | `AvgTimeToEmpty` in the `AppleSmartBattery` registry entry (`ioreg -rn AppleSmartBattery`): the battery's own figure, which often differs from macOS's |
| Power draw | `Voltage` × `InstantAmperage` from the same registry entry: a momentary reading |
| Cycles | `CycleCount`, and the rated count the battery reports (`DesignCycleCount9C`) |
| Condition, maximum capacity | `system_profiler SPPowerDataType`: the values System Information shows |

Both time estimates follow the current workload, so they move as you work: compiling or a video call shortens them,
an idle screen lengthens them. Treat them as "at this rate", not as a promise.

## Requirements

- A Mac with an internal battery. On a desktop Mac, battime prints a message and exits with status 1.
- zsh, the default shell since macOS 10.15 Catalina.
- Tested on an Apple Silicon MacBook Air with macOS 27, on battery. Intel MacBooks should expose the same registry
  keys, but this version has not been tested on one; reports are welcome in the issues.

## A menu bar version

battime is free and stays a terminal tool. If you would rather see the time remaining in your menu bar all day,
[TurtleBar](https://www.turtlebar.app/?ref=github.com) is a paid app ($4.99, one time) from the same developer. It
also warns you when the Mac will not last until a time you set and can turn on Low Power Mode by itself.

## License

MIT, see [LICENSE](LICENSE).
