# Closeout Notes for Future AI Agents

This repository contains a small Bash helper for launching and autofilling the
Steam SWTOR launcher on Gavin's Linux desktop.

## Current State

- GitHub repo: `https://github.com/gavin-bender/SWTOR-Launcher`
- Main executable: `swtor-login`
- Documentation: `README.md`
- Current branch: `main`
- The app is not compiled. It is a Bash script that Steam or the shell runs
  directly.

The tool:

- launches SWTOR through Steam app ID `1286830`
- waits for the SWTOR launcher window with `xdotool`
- reads password and TOTP from a 1Password item, default title `SWTOR`
- fills password and one-time-code fields using a configurable Tab sequence
- does not press Login
- reports status and errors through `notify-send` desktop notifications

## Important Design Decisions

- Keep it as a single Bash script. There is no build, packaging, or generated
  artifact step.
- Do not store secrets. The config file only stores non-secret settings.
- Fetch secrets at runtime with the 1Password CLI:
  - password: `op item get "$SWTOR_1P_ITEM" --fields password --reveal`
  - TOTP: `op item get "$SWTOR_1P_ITEM" --otp`
- Pipe secrets into `xdotool type --file -` so passwords and codes are not
  exposed as process command-line arguments.
- Use `notify-send` only for desktop UI. Popup experiments with `kdialog` and
  `zenity` were rejected because they were distracting and interfered with the
  automation flow.
- Preserve `SWTOR_NOTIFY=0` as the opt-out for desktop notifications.

## Steam Integration

For the original SWTOR Steam entry, use this Launch Options value:

```sh
bash -lc '"/home/gavin/Documents/SWTOR-Launcher/swtor-login" --no-launch & exec "$@"' -- %command%
```

Do not replace the SWTOR app command with `swtor-login` directly. In normal mode
the script launches Steam app `1286830`, so using it as the SWTOR executable
would recurse.

## Config

Default config path:

```text
~/.config/swtor-launcher/config
```

Create it with:

```sh
./swtor-login --init-config
```

Key settings:

```sh
SWTOR_1P_ITEM=SWTOR
SWTOR_1P_VAULT=
SWTOR_STEAM_APP_ID=1286830
SWTOR_WINDOW_REGEX='STAR WARS|The Old Republic|SWTOR|launcher'
SWTOR_TABS_TO_PASSWORD=0
SWTOR_TABS_TO_OTP=1
SWTOR_NOTIFY=1
```

The Tab settings are intentionally configurable because launcher focus and
layout can change. Tune with `./swtor-login --no-launch` while the launcher is
already open.

## Verification Commands

Run these before committing changes:

```sh
bash -n swtor-login
./swtor-login --help
./swtor-login --test-notify
SWTOR_NOTIFY=0 ./swtor-login --test-notify
./swtor-login --check
```

`shellcheck` is useful if installed, but it was not installed during the initial
work:

```sh
if command -v shellcheck >/dev/null 2>&1; then shellcheck swtor-login; fi
```

## Timeline of Notable Changes

- `8b345a5` initial SWTOR launcher autofill tool.
- `b8e0456` expanded the README for GitHub.
- `2dd6532` tried KDE passive popups with `kdialog`.
- `985e439` tried real dialog popups with `zenity`.
- `8cb0258` reverted popups back to `notify-send` notifications after user
  feedback that popups were disruptive and broke the flow.

## Pitfalls

- Do not reintroduce blocking or focus-stealing dialogs for normal status.
- Avoid modal error UI unless the user explicitly asks; `notify-send` critical
  urgency is the current accepted behavior.
- Do not press Enter or click Login automatically. The current behavior is fill
  only.
- Do not add config values that store secrets.
- Be careful with `op` output and `xdotool`; keep secrets out of argv and logs.
- This project is not a package manager integration. If a future change adds
  install scripts or desktop entries, keep the direct script path working.

## Current Dependencies

- `bash`
- `steam`
- `xdotool`
- `notify-send` from `libnotify`
- `op` from the 1Password CLI
- an X/XWayland `DISPLAY`

The known local environment during development was Arch Linux on KDE Wayland
with XWayland available.
