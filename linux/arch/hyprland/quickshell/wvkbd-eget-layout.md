# wvkbd: Creating your own keyboard layout

A standalone, self-contained guide — not dependent on the rest of this
documentation site. Written in English since it's meant to be
downloadable and usable by anyone, regardless of language.

Based on the exact process used to build
[wvkbd-norsk](https://github.com/archruud/wvkbd-norsk) (Norwegian).
See that repo for a worked example.


This recipe follows the exact same method used to build the Norwegian
layout, and can be reused for any language with a handful of keys that
differ from the US layout (Swedish, Danish, German, French, etc).

### 1. Copy the starting point

```bash
cd wvkbd
cp keymap.deskintl.h keymap.YOURLANG.h
cp layout.deskintl.h layout.YOURLANG.h
cp config.deskintl.h config.YOURLANG.h
```

### 2. Find the XKB keysym names for your special characters

Every standard Unicode letter has a fixed, documented XKB name (this isn't
something you guess — they're defined in X11's `keysymdef.h` and have been
stable for decades). Some examples:

| Letter | XKB name (lowercase) | XKB name (uppercase) |
|---|---|---|
| æ / Æ | `ae` | `AE` |
| ø / Ø | `oslash` | `Ooblique` |
| å / Å | `aring` | `Aring` |
| ü / Ü | `udiaeresis` | `Udiaeresis` |
| ñ / Ñ | `ntilde` | `Ntilde` |
| ç / Ç | `ccedilla` | `Ccedilla` |

Full list: run `grep -i "^#define XK_" /usr/include/X11/keysymdef.h` on
any Linux machine with the X11 headers installed.

### 3. Edit the `xkb_symbols` section in the keymap file

Find the line for the key you want to change (search for the physical key
position, e.g. `<AC10>` for the key right of L). The format is:

```
key <AC10>               {	[       semicolon,           colon ] };\
```

Replace it with your character (remember: an actual TAB character between
`{` and `[`, not a space or the literal text `\t` — use a small Python
script to guarantee the correct byte format instead of editing by hand):

```python
with open('keymap.YOURLANG.h', 'r', encoding='utf-8') as f:
    lines = f.readlines()
TAB = '\t'
lines[LINE_NUMBER - 1] = f'        key <AC10>               {{{TAB}[                  ae,              AE ] }};\\\n'
with open('keymap.YOURLANG.h', 'w', encoding='utf-8') as f:
    f.writelines(lines)
```

**Important:** this text is part of one continuous C string spanning
thousands of lines. Every line MUST end with exactly one `\` character
(line continuation) — not zero, not two. Use `cat -A filename.h` to compare
your line ending against an unmodified line in the same file before you're
satisfied.

### 4. Edit the visible button label in the layout file

Find the same physical key in `layout.YOURLANG.h` (search for the same
`KEY_SEMICOLON`/`KEY_APOSTROPHE`/etc. — the scancode name, not the keysym
name):

```c
{";", ":", 1.0, Code, KEY_SEMICOLON},
```
becomes:
```c
{"æ", "Æ", 1.0, Code, KEY_SEMICOLON},
```

### 5. Build and test

```bash
make LAYOUT=YOURLANG
./wvkbd-YOURLANG   # test directly, without installing first
```

Does the right character show up when you press the right key? You're
done — `sudo make LAYOUT=YOURLANG install`.

### Contributing back

Made a layout for another language? Feel free to open a pull request with
your `keymap.YOURLANG.h` / `layout.YOURLANG.h` / `config.YOURLANG.h` files
in this repo, so we can collect several languages in one place.
