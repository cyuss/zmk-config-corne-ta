# zmk-config-corne-ta

ZMK configuration for a 42-key **Corne** (foostan corne, 6-column) running on
two `nice_nano_v2` controllers with **nice!view** displays.

| | |
|---|---|
| Board | `nice_nano_v2` |
| Shields | `corne_left` / `corne_right` + `nice_view_adapter` + `nice_view` |
| ZMK | `v0.3` (pinned in `config/west.yml` and in the CI workflow) |
| Bluetooth name | `TA Corne` |
| ZMK Studio | enabled, unlock combo = **G + H** |

## Layout

Three layers. Layer names are exposed through `display-name`, so they show up on
the nice!view and in ZMK Studio.

### 0 — Base

```
 TAB     |  Q  |  W  |  E  |  R  |  T  |     |  Y  |  U  |  I  |  O  |  P  |  BSPC
 HYP/ESC |  A  |  S  |  D  |  F  |  G  |     |  H  |  J  |  K  |  L  |  ;  |  HYP/ESC
 LSHFT   |  Z  |  X  |  C  |  V  |  B  |     |  N  |  M  |  ,  |  .  |  /  |  RSHFT
                     | GUI | SYM/ESC | SPC | | RET | NAV/BSPC | ALT |
```

- **Home row mods** (on hold): `A`/`;` = GUI, `S`/`L` = ALT, `D`/`K` = SHIFT, `F`/`J` = CTRL.
- **Outer pinky columns**: Hyper (`GUI+ALT+SHIFT+CTRL`) on hold, `ESC` on tap.
- **Thumbs**: the two inner thumb keys are layer-taps — hold for the layer, tap for `ESC` (left) / `BSPC` (right).

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
| `config/corne.keymap` | layers, behaviors, combos, macros |
| `config/corne.conf` | Kconfig (sleep, TX power, Studio, debounce) |
| `config/west.yml` | ZMK revision pin |
| `build.yaml` | CI build matrix |
| `boards/shields/` | drop-in point for a custom shield (`zephyr/module.yml` sets `board_root`) |
