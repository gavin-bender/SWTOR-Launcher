# SWTOR Launcher Autofill

Launches **STAR WARS: The Old Republic** through Steam, reads your SWTOR
password and one-time password from 1Password, and types them into the launcher.

The tool is intentionally small: it is one Bash script, stores no secrets, and
does not click Login for you.

## What It Does

- Starts SWTOR through Steam app `1286830`
- Waits for the SWTOR launcher window
- Reads your password from a 1Password item
- Reads the current TOTP/one-time password from the same item
- Types both values into the launcher with `xdotool`
- Sends desktop notifications for progress and failures

## Requirements

- Linux with Steam
- SWTOR installed through Steam
- `xdotool`
- `notify-send` for desktop notifications
- 1Password CLI, `op`
- A signed-in 1Password CLI session

On Arch Linux:

```sh
sudo pacman -S xdotool libnotify
```

Install the 1Password CLI separately, then sign in:

```sh
op signin
```

## 1Password Item

By default, the script looks for a 1Password item named `SWTOR`.

That item must have:

- a password field
- a one-time password/TOTP field

You can use a different item name or vault in the config.

## Setup

Clone the repo:

```sh
git clone https://github.com/gavin-bender/SWTOR-Launcher.git
cd SWTOR-Launcher
```

Create the default config:

```sh
./swtor-login --init-config
```

The config is written to:

```text
~/.config/swtor-launcher/config
```

It only stores settings, never your password or OTP.

## Configuration

Common settings:

```sh
SWTOR_1P_ITEM=SWTOR
SWTOR_1P_VAULT=

SWTOR_STEAM_APP_ID=1286830
SWTOR_WINDOW_REGEX='STAR WARS|The Old Republic|SWTOR|launcher'

SWTOR_TABS_TO_PASSWORD=0
SWTOR_TABS_TO_OTP=1

SWTOR_NOTIFY=1
```

Set `SWTOR_NOTIFY=0` to disable desktop notifications.

If the password or OTP lands in the wrong field, adjust:

```sh
SWTOR_TABS_TO_PASSWORD=0
SWTOR_TABS_TO_OTP=1
```

Then retry with `--no-launch` while the launcher is already open.

## Usage

Check dependencies and 1Password access:

```sh
./swtor-login --check
```

Send a test notification:

```sh
./swtor-login --test-notify
```

Launch SWTOR and fill the launcher:

```sh
./swtor-login
```

Fill an already-open launcher:

```sh
./swtor-login --no-launch
```

## Steam Launch Options

To run this from the original SWTOR Steam entry, open SWTOR properties in Steam
and set Launch Options to:

```sh
bash -lc '"/home/gavin/Documents/SWTOR-Launcher/swtor-login" --no-launch & exec "$@"' -- %command%
```

This starts the autofill helper in the background, then lets Steam run SWTOR
normally.

Do not set the SWTOR launch option to only `swtor-login`; the script launches
Steam itself in normal mode, so that would recurse.

## Troubleshooting

### Missing 1Password CLI

If you see:

```text
missing 'op'. Install the 1Password CLI, then run 'op signin'.
```

Install the 1Password CLI and sign in with:

```sh
op signin
```

### Wrong Field Is Filled

Run SWTOR normally, leave the launcher open, then tune the Tab settings:

```sh
./swtor-login --no-launch
```

Edit `~/.config/swtor-launcher/config` until the password and OTP land in the
right fields.

### No Notifications

Check that `notify-send` exists:

```sh
command -v notify-send
```

Then test:

```sh
./swtor-login --test-notify
```

### Launcher Window Not Found

If the script times out waiting for the launcher, inspect window titles:

```sh
xdotool search --onlyvisible --name . getwindowname %@ 2>/dev/null
```

Then update `SWTOR_WINDOW_REGEX` in the config.

## Security Notes

- The script does not store your SWTOR password or OTP.
- Secrets are read from 1Password at runtime.
- Secrets are piped into `xdotool` through stdin instead of being passed as
  command-line arguments.
- The tool fills fields but does not press Login.
