# zmk-config-corne-ta

ZMK configuration for a 42-key **Corne** (foostan corne, 6-column) running on
two `nice_nano_v2` controllers with **nice!view** displays.

| | |
|---|---|
| Board | `nice_nano_v2` |
| Shields | `corne_left` / `corne_right` + `nice_view_adapter` + `nice_view` |
| ZMK | `v0.3` (pinned in `config/west.yml` and in the CI workflow) |
| Bluetooth name | `TA Corne` |
| ZMK Studio | **disabled on purpose** — see below |

## Layout

Four layers. Base and System carry a `display-name`, so they are named on the
nice!view.

### 0 — Base

```
 TAB     |  Q  |  W  |  E  |  R  |  T  |     |  Y  |  U  |  I  |  O  |  P  |  BSPC
 HYP/ESC |  A  |  S  |  D  |  F  |  G  |     |  H  |  J  |  K  |  L  |  ;  |  HYP/ESC
 LSHFT   |  Z  |  X  |  C  |  V  |  B  |     |  N  |  M  |  ,  |  .  |  /  |  RSHFT
                     | GUI | SYM/ESC | SPC | | RET | NAV/BSPC | FR |
```

- **Home row mods** (on hold): `A`/`;` = GUI, `S`/`L` = ALT, `D`/`K` = SHIFT, `F`/`J` = CTRL.
- **Outer pinky columns**: Hyper (`GUI+ALT+SHIFT+CTRL`) on hold, `ESC` on tap.
- **Thumbs**: the two inner thumb keys are layer-taps — hold for the layer, tap for `ESC` (left) / `BSPC` (right). The right outer thumb holds the System layer.

### 1 — Nav (right inner thumb)

```
 ESC |  BT0  |  7  |  8  |  9  |  F8  |     |  F1  | F2   |  F3  |  F4   |  F5  |  BSPC
 BT1 |  BT2  |  4  |  5  |  6  |  F9  |     | PGUP | HOME |  UP  |  END  |  F6  |  CAPSWD
 BT3 |  BT4  |  1  |  2  |  3  |  0   |     | PGDN | LEFT | DOWN | RIGHT |  F7  |  BTCLR
```

### 2 — Symbols (left inner thumb)

```
 ESC |  @  |  :  |  #  |  $  |  %  |     |  (  |  )  |  [  |  ]  |  \  |  DEL
     |  !  |  ?  |  "  |  '  |  _  |     |  {  |  }  |  <  |  >  |  |  |
     |  `  |  *  |  =  |  +  |  -  |     |  ^  |  &  |  ,  |  .  |  /  |
```

### 3 — System

```
     |     |     |     |     |     |     |  BRI- | BRI+ |      |      |     |
     |     |     |     |     |     |     |  VOL- | MUTE | VOL+ |      |     |
     |     |     |     |     |     |     |  PREV | PLAY | NEXT |      |     |
```

Reached by the right outer thumb, or by holding both thumb layer keys at once
(a conditional layer on `if-layers = <1 2>`).

## Hold-tap tuning

Both the home row mods and the thumb layer-taps use `balanced` flavor, which
resolves to the hold as soon as another key is pressed **and** released. The
built-in defaults (`hold-preferred` for `&mt`, `tap-preferred` for `&lt`) are
deliberately overridden — see the comments in `config/corne.keymap`.

The home row mods additionally use **positional hold-tap**: `hold-trigger-key-positions`
restricts a left-hand mod to right-hand (and thumb) partners and vice versa, so
same-hand rolls like `d`+`f` can never produce a stray modifier.
`require-prior-idle-ms` disables the hold entirely mid-word.

Thumb layer-taps intentionally do **not** set `require-prior-idle-ms`: a thumb is
legitimately pressed right after a letter, and suppressing the hold there would
emit `ESC`/`BSPC` instead of switching layers.

## Why ZMK Studio is disabled

`CONFIG_ZMK_STUDIO=y` selects both `ZMK_KEYMAP_SETTINGS_STORAGE` and
`ZMK_KEYMAP_LAYER_REORDERING` (`app/src/studio/Kconfig`). With those on, ZMK
loads the firmware keymap at boot and then **overwrites individual key positions
and the whole layer order with whatever Studio previously saved to flash**
(`keymap_handle_set` in `app/src/keymap.c`). Any position edited in Studio is
frozen, and layers added later are missing from the stored layer order entirely,
so they cannot be reached no matter what is flashed.

This repo is edited through git and keymap-editor, so it is the single source of
truth and Studio is turned off. What is flashed is what runs.

With `CONFIG_ZMK_STUDIO=n` the settings-storage code is not compiled in at all,
so anything Studio previously wrote to flash is simply ignored. **No settings
reset is needed for the keymap.** The `settings_reset` firmware stays in
`build.yaml` for Bluetooth trouble only; it also wipes every pairing.

## Building

Firmware is built by GitHub Actions on every push. Grab the artifact from the
run's summary page, then flash by double-tapping reset on each half and copying
the matching `.uf2` onto the `NICENANO` drive.

To build locally you need a ZMK west workspace in `.zmk/` (gitignored):

```sh
west build -s zmk/app -d build/left  -b nice_nano_v2 -- \
  -DSHIELD="corne_left nice_view_adapter nice_view" -DZMK_CONFIG="$PWD/config"
```

## Files

| Path | Purpose |
|---|---|
| `config/corne.keymap` | layers, behaviors, macros |
| `config/corne.conf` | Kconfig (sleep, TX power, Studio, debounce) |
| `config/west.yml` | ZMK revision pin |
| `build.yaml` | CI build matrix, including the `settings_reset` firmware |
| `boards/shields/` | drop-in point for a custom shield (`zephyr/module.yml` sets `board_root`) |
