# CLAUDE.md

Personal fork of [Vonng/Capslock](https://github.com/Vonng/Capslock) — a Karabiner-Elements complex modification that turns CapsLock into a Hyper key (`right_cmd + right_ctrl + right_shift + right_option`).

## Source of truth

- `mac_v3/capslock.yml` — authoritative; edit here
- `mac_v3/capslock.json` — compiled output; must stay in sync with the YAML
- Personal customizations live in their own sections appended to `capslock.yml`

## Build & deploy

### Standard flow (requires `yq`)
```sh
cd mac_v3 && make install
```
This compiles YAML → JSON and copies to `~/.config/karabiner/assets/complex_modifications/`.

Use `make compile` to only regenerate `capslock.json` without touching the live
Karabiner config. Never hand-edit `capslock.json` — it is generated output.

### Critical gotcha: assets ≠ active rules
The file at `~/.config/karabiner/assets/complex_modifications/capslock.json` only lists *available* rules for the Karabiner UI to import. Karabiner does **not** read it live. Active rules live in `~/.config/karabiner/karabiner.json` under `profiles[0].complex_modifications.rules`.

To activate a new rule after editing the source files, either:
1. Open Karabiner-Elements → "Complex Modifications" → "Add rule", or
2. Edit `~/.config/karabiner/karabiner.json` directly (matches a more compact inline style with single-line modifiers). Karabiner auto-reloads on save.

Always back up before editing the active config:
```sh
cp ~/.config/karabiner/karabiner.json ~/.config/karabiner/karabiner.json.bak-$(date +%Y%m%d)
```

### Validation
```sh
python3 -c "import json; json.load(open('PATH')); print('valid')"
```

## Custom sections

### Apptivate (Cmd+number app launchers)
Cmd+1..5 → Chrome / Things3 / Claude / Obsidian / iTerm via `shell_command: open -a`. `open -a` natively launches if closed or focuses if open — same behavior as Apptivate.

**Trade-off:** hijacks plain Cmd+1..5 globally, so browser tab switching and Slack workspace switching stop working. To exclude an app, add `conditions` with `frontmost_application_unless` on its bundle ID.

### Mouse Thumb Buttons to Middle Click
button4/button5 → button3, with `optional: [any]` so it fires regardless of modifiers.

**One-time setup per mouse:** open Karabiner-Elements → "Devices" tab → toggle "Modify events" on for the mouse. Without this, Karabiner won't intercept mouse events. If your mouse uses vendor software (Logitech Options, Razer Synapse, etc.) that may intercept buttons first — set the thumb buttons to "Generic Button 4 / Button 5" in the vendor app.

## Git remotes

- `origin` → `m-kronberg/Capslock` (personal fork; push target) over SSH
- `upstream` → `Vonng/Capslock` (read-only; do not push)
- Work happens on `master`, which tracks `origin/master`. Plain `git push` is correct.
- Upstream's default branch is `main`; pull upstream changes with
  `git fetch upstream && git merge upstream/main`.

## Updating from Apptivate

If the Apptivate mapping changes and you want to re-sync, the hotkeys live at:
```
~/Library/Application Support/Apptivate/hotkeys
```
It's an NSKeyedArchiver binary plist. Decode with:
```sh
plutil -convert xml1 -o - ~/Library/Application\ Support/Apptivate/hotkeys
```
macOS virtual keycodes used by Apptivate entries: `18`=1, `19`=2, `20`=3, `21`=4, `23`=5, `22`=6, `26`=7, `28`=8, `25`=9, `29`=0. `mods: 256` = Command-only.
