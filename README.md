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

Six layers. Layer names are exposed through `display-name`, so they show up on
the nice!view and in ZMK Studio.

### 0 — Base

```
 TAB     |  Q  |  W  |  E  |  R  |  T  |     |  Y  |  U  |  I  |  O  |  P  |  BSPC
 HYP/ESC |  A  |  S  |  D  |  F  |  G  |     |  H  |  J  |  K  |  L  |  ;  |  HYP/ESC
 LSHFT   |  Z  |  X  |  C  |  V  |  B  |     |  N  |  M  |  ,  |  .  |  /  |  RSHFT
                     | GUI | SYM/ESC | SPC | | RET | NAV/BSPC | FR |
```

- **Home row mods** (on hold): `A`/`;` = GUI, `S`/`L` = ALT, `D`/`K` = SHIFT, `F`/`J` = CTRL.
- **Outer pinky columns**: Hyper (`GUI+ALT+SHIFT+CTRL`) on hold, `ESC` on tap.
- **Thumbs**: the two inner thumb keys are layer-taps — hold for the layer, tap for `ESC` (left) / `BSPC` (right). The right outer thumb holds the French layer.

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

### 3 / 4 / 5 — French

```
     |  ê  |  ô  |  î  |  û  |  â  |     |  «  |  »  |  €  |  …  |
     |  é  |  è  |  à  |  ç  |  ù  |     |
     |  ë  |  ï  |  ü  |  œ  |  æ  |     |                     OS
```

Held with the right outer thumb, so the left hand types. Shift gives the capital
on every letter (`É`, `À`, `Ç`, …) through a mod-morph on each key.

ZMK sends HID scancodes, not characters, so what actually appears depends
entirely on the host's keyboard layout. Both hosts are covered:

| Layer | Host | Mechanism |
|---|---|---|
| 3 `French` | macOS | Option dead keys of the standard US layout (`⌥E` `⌥\`` `⌥I` `⌥U`), plus direct `⌥C` `⌥Q` `⌥'` `⌥\` `⌥⇧2` `⌥;` |
| 4 `French (Win)` | Windows | CP1252 alt codes — hold Alt, type four keypad digits |
| 5 `OS: Windows` | — | empty all-transparent flag layer |

Layer 5 never shadows anything on its own; it is only a switch, toggled by the
bottom-right key of the French layer. A conditional layer wires it up:

```dts
french_windows {
    if-layers = <3 5>;
    then-layer = <4>;
};
```

When the flag is on *and* the French key is held, layer 4 activates and, being
higher, wins over layer 3 — same physical keys, right encoding for the connected
host. Because layer 5 stays active in Windows mode and the nice!view shows
`zmk_keymap_highest_layer_active()`, its name doubles as a permanent host
indicator on the display.

Two caveats on the Windows side: alt codes need **NumLock on**, and the leading
`0` is significant (ANSI/CP1252, not the OEM code page).

The macOS side needs only four parameterised macros — one per dead accent —
rather than one macro per accented letter:

```dts
mac_acu: mac_acu {
    compatible = "zmk,behavior-macro-one-param";
    #binding-cells = <1>;
    bindings = <&kp LA(E)>, <&macro_param_1to1>, <&kp MACRO_PLACEHOLDER>;
};
```

`&mac_acu E` gives `é`, `&mac_acu LS(E)` gives `É`.

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
| `config/corne.keymap` | layers, behaviors, combos, macros, French input |
| `config/corne.conf` | Kconfig (sleep, TX power, Studio, debounce) |
| `config/west.yml` | ZMK revision pin |
| `build.yaml` | CI build matrix |
| `boards/shields/` | drop-in point for a custom shield (`zephyr/module.yml` sets `board_root`) |
