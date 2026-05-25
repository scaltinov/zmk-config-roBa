# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository overview

Personal ZMK firmware config for the **roBa** split keyboard.
- Hardware: two **Seeeduino XIAO nRF52840 Sense** boards
- Right side (`roBa_R`) — split central, hosts the **PMW3610 trackball**, paired to host over BLE
- Left side (`roBa_L`) — split peripheral, hosts the **EC11 rotary encoder**, connects via the right side
- Latest keymap is a custom layout based on the fadotech blog reference (commit `e67c482`).

## Build, flash, visualize

- **CI build** — pushing to any branch triggers `.github/workflows/build.yml`, which calls `zmkfirmware/zmk/.github/workflows/build-user-config.yml@v0.3-branch`. The matrix is defined in `build.yaml` (top-level): currently `roBa_R` (with `studio-rpc-usb-uart` snippet), `roBa_L`, and `settings_reset`. UF2 artifacts are downloaded from the workflow run.
- **Adding/removing a board+shield combo** — edit `build.yaml`. ZMK upstream or driver-fork pinning lives in `config/west.yml` (currently ZMK `v0.3-branch` + `kumamuk-git/zmk-pmw3610-driver` `main`).
- **Flashing** — double-tap the reset button on the target XIAO to mount `XIAO-SENSE`, then copy the matching `.uf2` to `/Volumes/XIAO-SENSE/`. The board reboots automatically.
- **Rollback to stock firmware** — `factory_backup/roBa_{L,R}_factory.uf2` are the full-flash dumps captured before flashing ZMK. Flash the same way.
- **Keymap visualization** — `.github/workflows/draw.yml` runs `caksoylar/keymap-drawer` automatically whenever `config/roBa.keymap` (or `config/roBa.dtsi`) changes, regenerating `keymap-drawer/roBa.svg` and committing it back. Do not edit the SVG by hand; do not run keymap-drawer locally.
- **No local toolchain is set up in this repo.** Default workflow is: edit → push → download CI artifact → flash. Use ZMK Studio (`CONFIG_ZMK_STUDIO=y` on `roBa_R`) for live keymap tweaks without rebuilding.

## Architecture

Two-tier config split:

- **`config/`** — user-facing keymap layer
  - `roBa.keymap` — devicetree keymap (behaviors, macros, combos, 7 layers)
  - `roBa.json` — physical layout + sensor metadata used by keymap-drawer
  - `west.yml` — west manifest (ZMK fork pin + PMW3610 driver)
- **`boards/shields/roBa/`** — hardware shield definition (Kconfig + devicetree)
  - `roBa.dtsi` — shared devicetree: 49-key physical layout, matrix transform, kscan GPIO matrix, encoder node, trackball node
  - `roBa.zmk.yml` — shield metadata (registers `roBa` for `build.yaml`, requires `seeeduino_xiao_ble`)
  - `roBa_L.conf` / `roBa_L.overlay` — left side: EC11 encoder, battery proxy to right, `CONFIG_ZMK_POINTING=y`, 30 min idle sleep, GPIO/encoder pin assignments
  - `roBa_R.conf` / `roBa_R.overlay` — right side: PMW3610 trackball (CPI 1200, automouse timeout 1000 ms, orientation 180°, smart algorithm), SPI init, `CONFIG_ZMK_STUDIO=y` (locking off), 30 min idle sleep
  - `Kconfig.shield` / `Kconfig.defconfig` — declares `SHIELD_ROBA_L`/`SHIELD_ROBA_R`, sets split central/peripheral roles and `ZMK_KEYBOARD_NAME`
- **`zephyr/module.yml`** — declares this repo as a Zephyr module with `board_root: .` so the in-repo shield is discoverable by upstream ZMK builds.

### Keymap layers (`config/roBa.keymap`)

| # | Name | Purpose |
|---|------|---------|
| 0 | `default_layer` | QWERTY base + hold-taps (`S`→TAB, `Z`→LSHFT) + thumb cluster (LALT / LGUI / caps_word / LCTRL / `&mo 1` / LSHFT / RGUI / `&lt 2 SPACE` / `&lt 5 ESC`) |
| 1 | `FUNCTION` | ASCII symbols (`! @ # $ %`, brackets, operators, etc.) |
| 2 | `NUM` | F1–F15 top row + numpad cluster + arithmetic operators |
| 3 | `ARROW` | macOS text navigation (Cmd+A/E, Alt/Ctrl+arrows), mouse buttons MB1–5, arrow keys |
| 4 | `MOUSE` | Auto-mouse layer activated by trackball motion (`automouse-layer = 4`) — mostly transparent except a mouse-button row |
| 5 | `SCROLL` | Trackball scroll layer (`scroll-layers = <5>`) — fully transparent so trackball generates scroll deltas |
| 6 | `layer_6` | BT pairing (`BT_SEL 0–4`, `BT_CLR`, `BT_CLR_ALL`), macOS shortcuts, bootloader, numeric layer selector |

### Behaviors / combos / sensors

- `&mt` mod-tap — `flavor = "balanced"`, `quick-tap-ms = 0`
- `&trackball` — wraps the PMW3610 input with `automouse-layer = 4` and `scroll-layers = <5>`
- `to_layer_0` ZMK macro and `lt_to_layer_0` hold-tap — both return to the base layer from any other layer (used for escape-style transitions)
- Combos: `[11,12]`=TAB, `[12,13]`=Shift+TAB, `[20,21]`=`"`, `[24,25]`=`=`
- Left-side sensor binding: `&inc_dec_kp PAGE_UP PAGE_DOWN` (EC11)

## Conventions / gotchas

- **Indentation is enforced by `.editorconfig`**: `*.keymap`, `*.overlay`, `*.dtsi` use **hard tabs, width 8**. Do not introduce spaces — the existing files mix-and-match in a few places but new edits should stay on tab.
- **Only `roBa_R` pairs with the host.** Never try to pair `roBa_L` directly; it talks to the host through the right side.
- **ZMK Studio over USB** — to use Studio, connect `roBa_R` via USB-C; the `studio-rpc-usb-uart` snippet in `build.yaml` is what makes that work. Studio locking is off (`CONFIG_ZMK_STUDIO_LOCKING=n`).
- **`factory_backup/*.uf2` are ~1.9 MB full-flash dumps** (SoftDevice + bootloader + app). Do not compare sizes against ZMK build artifacts (~350–525 KB, application only).
- **Commit style** — short, single-line, often Japanese (e.g. `v0.3-branchを参照`, `build.ymlの更新`, `add CONFIG_ZMK_POINTING=y in roBa_L.conf`). Match it.
- **`.claude/settings.local.json`** only allows the `Bash(rtk ls *)` permission pattern. No hooks configured.
