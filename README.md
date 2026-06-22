# SWTOR Launcher Autofill

Small local helper for launching SWTOR through Steam and filling the launcher
password and one-time-password fields from 1Password.

It does not store secrets and it does not press Login.

## Requirements

- Steam with SWTOR installed as app `1286830`
- `xdotool`
- `notify-send` for desktop notifications
- 1Password CLI, `op`
- A signed-in 1Password CLI session
- A 1Password item named `SWTOR` with:
  - a password field
  - a one-time password/TOTP field

On Arch, the missing local dependency from this project is currently expected to
be the 1Password CLI. Install it separately, then run:

```sh
op signin
```

## Setup

Create the default config:

```sh
./swtor-login --init-config
```

Edit `~/.config/swtor-launcher/config` if your item name, vault, window title, or
Tab sequence differs.

Useful settings:

```sh
SWTOR_1P_ITEM=SWTOR
SWTOR_1P_VAULT=
SWTOR_TABS_TO_PASSWORD=0
SWTOR_TABS_TO_OTP=1
SWTOR_NOTIFY=1
```

Set `SWTOR_NOTIFY=0` to keep terminal output only.

## Use

Check dependencies and 1Password access:

```sh
./swtor-login --check
```

Send a test desktop notification:

```sh
./swtor-login --test-notify
```

Launch SWTOR and fill the launcher:

```sh
./swtor-login
```

Fill an already-open launcher, useful while tuning the Tab sequence:

```sh
./swtor-login --no-launch
```

If the password or OTP goes into the wrong field, adjust
`SWTOR_TABS_TO_PASSWORD` and `SWTOR_TABS_TO_OTP` in the config and retry with
`--no-launch`.
