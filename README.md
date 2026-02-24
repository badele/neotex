![neotex](./logo.png)

**Resurrecting 80s ANSI art into a clear, modern file format.**

Neotex is a file format designed to make viewing and sharing ANSI art files
(often encoded in CP437) easier. The concept is simple: separate the text
content from the style sequences, allowing clean copy/paste of the text while
preserving all formatting information.

## Why Neotex?

- **Clean copy/paste**: Raw text is directly readable and copyable
- **Human-readable format**: Styles are expressed in a compact, readable format
- **Easy to parse**: Simple line-by-line structure, ideal for implementation
- **Preserves ANSI art**: All color and effect information is retained

## Example

<img src="./neotex-format.png" align="center"/>

This example generated from this [neotex example file](./neotex-format.neo)

## Format Structure

Each line in a `.neo` file follows this structure:

```
<TEXT> | <METADATA_AND_STYLES>
```

- **Left part**: the text content (Unicode characters)
- **Separator**: the exact sequence `|` (space, pipe, space)
- **Right part**: metadata and style sequences

Required rules:

- The `|` is placed at column `yy` defined by `!TWxx/yy`; pad the text so the
  separator lands there.
- The first line's right part starts with `!V…; !TW…; !NL…;` in that order, with
  a required space after each `;`. Subsequent lines may omit these labels.
- A space is required after every `;` and after every `,`; no space after `:`.
- Style sequences are optional.

### Simple Example

```
Hello World | !V1; !TW11/11; !NL1; 1:FR; 7:FG;
```

This line displays "Hello World" with:

- Position 1: "Hello" in bright red (FR = Foreground Red bright)
- Position 7: "World" in bright green (FG = Foreground Green bright)

## Complete Specification

### 1. Metadata

Metadata starts with `!`. Only the first line must begin its right part with the
three required labels `!V…; !TW…; !NL…;` (in that order, with a space after each
`;`). Subsequent lines may include optional metadata or none at all.

| Metadata        | Format                         | Description                                                 |
| --------------- | ------------------------------ | ----------------------------------------------------------- |
| `!V<semver>`    | `!V1.0.0;`                     | Neotex format version (accepts `1`, `1.2`, or full `1.0.0`) |
| `!TW<t>/<m>`    | `!TW80/120;`                   | Width: `t` = trimmed width, `m` = max width                 |
| `!NL<n>`        | `!NL25;`                       | Total number of lines                                       |
| `!ST<text>`     | `!ST<My Art>;`                 | SAUCE title (stored on a dedicated metadata line)           |
| `!SA<text>`     | `!SA<Author>;`                 | SAUCE author                                                |
| `!SG<text>`     | `!SG<Group>;`                  | SAUCE group                                                 |
| `!SD<YYYYMMDD>` | `!SD<20260119>;`               | SAUCE creation date                                         |
| `!SF<text>`     | `!SF<Web437_ToshibaSat_8x14>;` | SAUCE font info (TInfoS)                                    |
| `!SI`           | `!SI;`                         | SAUCE iCE colors / NonBlink flag                            |

Only line 1 enforces the `!V…; !TW…; !NL…;` prefix; other lines can carry the
optional metadata above (including SAUCE tags) or simply style sequences.

`splitans` writes SAUCE labels on a trailing empty line; width/height are still
carried by `!TW`/`!NL`.

**Example:**

```
Hello | !V1; !TW5/5; !NL1; 1:FR;
```

### 2. Style Sequences

General format of a sequence:

```
<POSITION>:<STYLE1>, <STYLE2>, ...;
```

- **POSITION**: Character index (1-based) where the style applies
- **STYLES**: List of style codes separated by commas
- Each sequence ends with `;`
- A space is required after each `;` (except end of line) and after each `,`; no
  space after `:`.
- Sequences are optional: a line may have none.

### 3. Color Codes

#### ANSI 16 Colors

| Normal                                            | Code | Color   | Bright                                            | Code | Color (bright)      |
| ------------------------------------------------- | ---- | ------- | ------------------------------------------------- | ---- | ------------------- |
| ![](https://placehold.co/15x15/000000/000000.png) | `Fk` | Black   | ![](https://placehold.co/15x15/555555/555555.png) | `FK` | Bright Black (Gray) |
| ![](https://placehold.co/15x15/AA0000/AA0000.png) | `Fr` | Red     | ![](https://placehold.co/15x15/FF5555/FF5555.png) | `FR` | Bright Red          |
| ![](https://placehold.co/15x15/00AA00/00AA00.png) | `Fg` | Green   | ![](https://placehold.co/15x15/55FF55/55FF55.png) | `FG` | Bright Green        |
| ![](https://placehold.co/15x15/AA5500/AA5500.png) | `Fy` | Yellow  | ![](https://placehold.co/15x15/FFFF55/FFFF55.png) | `FY` | Bright Yellow       |
| ![](https://placehold.co/15x15/0000AA/0000AA.png) | `Fb` | Blue    | ![](https://placehold.co/15x15/5555FF/5555FF.png) | `FB` | Bright Blue         |
| ![](https://placehold.co/15x15/AA00AA/AA00AA.png) | `Fm` | Magenta | ![](https://placehold.co/15x15/FF55FF/FF55FF.png) | `FM` | Bright Magenta      |
| ![](https://placehold.co/15x15/00AAAA/00AAAA.png) | `Fc` | Cyan    | ![](https://placehold.co/15x15/55FFFF/55FFFF.png) | `FC` | Bright Cyan         |
| ![](https://placehold.co/15x15/AAAAAA/AAAAAA.png) | `Fw` | White   | ![](https://placehold.co/15x15/FFFFFF/FFFFFF.png) | `FW` | Bright White        |
|                                                   | `FD` | Default |                                                   |      |                     |

**Foreground (text)**: Prefix `F` **Background**: Prefix `B`

| Color                                             | Code | Description        |
| ------------------------------------------------- | ---- | ------------------ |
| ![](https://placehold.co/15x15/000000/000000.png) | `Bk` | Background Black   |
| ![](https://placehold.co/15x15/AA0000/AA0000.png) | `Br` | Background Red     |
| ![](https://placehold.co/15x15/00AA00/00AA00.png) | `Bg` | Background Green   |
| ![](https://placehold.co/15x15/AA5500/AA5500.png) | `By` | Background Yellow  |
| ![](https://placehold.co/15x15/0000AA/0000AA.png) | `Bb` | Background Blue    |
| ![](https://placehold.co/15x15/AA00AA/AA00AA.png) | `Bm` | Background Magenta |
| ![](https://placehold.co/15x15/00AAAA/00AAAA.png) | `Bc` | Background Cyan    |
| ![](https://placehold.co/15x15/AAAAAA/AAAAAA.png) | `Bw` | Background White   |
|                                                   | `BD` | Background Default |

> **Note**: Background colors do not have bright variants (uppercase). The
> "bright" effect is achieved through letter case for foreground only.

#### 256 Colors (Indexed Palette)

```
F<index>   Foreground indexed (0-255)
B<index>   Background indexed (0-255)
```

**Examples:**

- `F123`: Foreground color index 123
- `B200`: Background color index 200

#### RGB Colors (True Color)

```
F<RRGGBB>   Foreground RGB (hex)
B<RRGGBB>   Background RGB (hex)
```

**Examples:**

- `FF0080`: Foreground RGB(255, 0, 128) - pink
- `B00FF00`: Background RGB(0, 255, 0) - green

#### Link Hover Colors

Use the same formats as foreground/background but prefix with `HF` (hover
foreground) or `HB` (hover background). Hover colors persist until changed and
apply when a renderer supports pointer hover on hyperlinks.

- `HFY`: Hover foreground bright yellow
- `HBk`: Hover background black
- `HF123`: Hover foreground indexed color 123
- `HB00FF00`: Hover background RGB green

### 4. Effect Codes

| Code | Effect    | Code | Deactivation  |
| ---- | --------- | ---- | ------------- |
| `EM` | Dim       | `Em` | Dim OFF       |
| `EI` | Italic    | `Ei` | Italic OFF    |
| `EU` | Underline | `Eu` | Underline OFF |
| `EB` | Blink     | `Eb` | Blink OFF     |
| `ER` | Reverse   | `Er` | Reverse OFF   |

> **Note**: There is no Bold effect. The "bright" version of a color (uppercase)
> replaces this effect.

### 5. Hyperlink Codes

Hyperlinks (OSC 8) are encoded alongside other styles:

| Code       | Description                                               |
| ---------- | --------------------------------------------------------- |
| `HL:<url>` | Start hyperlink at this position (URL wrapped in `<...>`) |
| `Hl`       | End the current hyperlink                                 |

- Pair with `HF`/`HB` hover colors to define hover palette; hover colors are not
  reset by `R0`.
- Hyperlink codes can be combined with other styles at the same position.

**Example:**

```
Click me | !V1; !TW80/80; !NL101; 1:HL:<https://example.com>, FG, HFY, HBk; 9:Hl;
```

### 6. Special Code

| Code | Description                               |
| ---- | ----------------------------------------- |
| `R0` | Full reset (resets all styles to default) |

## Developer Guide (implementers)

### Line layout

- Each line is `<TEXT> | <RIGHT_PART>` with the exact separator `|` (space,
  pipe, space).
- The `|` must be placed at the column declared by `!TWt/m`; pad the left text
  accordingly.
- A space is mandatory after every `;` and `,` (except at end of line). No space
  after `:`.

### First line header (mandatory)

- The right part of line 1 must start with: `!V…; !TWt/m; !NLn;`
- Keep this exact order and a space after each `;`.

### Metadata (two families)

- Simple key/value: `!KEYvalue;` (no colon). Example: `!TW80/120;`, `!NL25;`,
  `!V1.0.0;`.
- SAUCE tags (values wrapped in `< >`):
  `!ST<title>; !SA<author>; !SG<group>; !SD<YYYYMMDD>; !SF<font>; !SI;`
- Optional labels can appear on any line; only `!V`, `!TW`, `!NL` are mandatory
  on the first line.

### Style sequences

- Syntax: `position:STYLE1, STYLE2;` (positions are 1-based).
- Styles are comma-separated with a required space after each comma; each
  sequence ends with `;` plus a space unless end of line.

### Style codes (recap)

- Colors: `Fk/FR`… for 16 colors (foreground), `Bk`… for background; indexed
  `F123`/`B200`; RGB `F00FF00`/`BFF0080`; hover `HF…`/`HB…`.
- Effects: `EM` (dim), `EI` (italic), `EU` (underline), `EB` (blink), `ER`
  (reverse); lowercase versions disable (`Em`/`Ei`/`Eu`/`Eb`/`Er`).
- Links: `HL:<url>` starts a link (URL inside `< >`); `Hl` ends it. Combine with
  hover colors if desired.
- Reset: `R0` resets all styles to defaults.

### Parsing checklist

1. Split each line at `|` (space, pipe, space) using the column from `!TW` to
   position the separator.
2. On the first line, read `!V…; !TW…; !NL…;` in that order; they must be
   present.
3. Split the right part on `;` (semicolon + space) to list entries. Each entry
   starting with `!` is metadata; others are style sequences.
4. For a style sequence, split on `:` (no spaces). Left = position; right =
   styles split on `,` (comma + space).
5. Apply styles in order; `R0` resets state. Hover colors persist until changed.

## Annotated Complete Example

```
Hello | !V1; !TW80/80; !NL101; 1:FR, Bg; 4:R0; 6:FG;
```

**Parsing:**

1. Text: `Hello`
2. Metadata (required, in this order):
   - `!V1`
   - `!TW80/80`
   - `!NL101`

3. Styles (sequences optional):
   - Position 1: `FR` (bright red), `Bg` (green background)
   - Position 4: `R0` (reset)
   - Position 6: `FG` (bright green)

**Rendering:**

- `Hel`: Bright red on green background
- `lo`: Default colors
- `` (space after "Hello"): Bright green

## Real Example: The Neotex Logo

Here is the content of the `logo.neo` file that generates the logo above:

```
▀▄          █▀▀▀▀█ ▐▀▀▀▀▀▀▀▀▀▀▀▀█      ▀▀▀▀▀▀▀▀▀▀     ▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀ ▐▀▀▀▀▀▀▀▀▀▀▀▀█ ▐▀▀▀▀█     █▀▀▀▀█   | !V1; !TW104/104; !NL12; 1:FM, Bk; 3:R0; 13:FM; 14:Bm; 18:Bk; 21:Bm; 33:Bk; 35:R0; 40:FM; 50:R0; 55:FM; 72:Bm; 84:Bk; 87:Bm; 91:Bk; 92:R0; 97:FM; 98:Bm; 102:Bk; 104:R0
▄ ▀▄        ██████ ▐ ████████████   ▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀  ▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀ ▐ ████████████ ▐ ████     █ ████   | 1:FM; 2:R0; 3:FM; 5:R0; 13:FM; 14:R0, Fg, Bk; 15:FM; 21:Bm; 22:Bk; 35:R0; 37:FM; 53:R0; 55:FM; 72:Bm; 73:Bk; 87:Bm; 88:Bk; 92:R0; 97:FM; 98:Bm; 99:Bk; 104:R0
 ▀▄ ▀▄      ██████ ▐ ████▀▀▀▀▀▀▀▀  ▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀ ▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀ ▐ ████▀▀▀▀▀▀▀▀ ▐ ████     █ ████   | 2:FM; 4:R0; 5:FM; 7:R0; 13:FM; 14:R0, Fg, Bk; 15:FM; 21:Bm; 22:Bk; 35:R0; 36:FM; 72:Bm; 73:Bk; 87:Bm; 88:Bk; 92:R0; 97:FM; 98:Bm; 99:Bk; 104:R0
▀▄ ▀▄ ▀▄    ██████ ▐ ████▄▄▄▄▄▄▄▄ ▐█▀▀▀▀█▀ ▀▀ ▀██████ ▀▀▀▀ █▀▀▀▀█ ▀▀▀ ▐ ████▄▄▄▄▄▄▄▄ ▐██████▄▄▄█▀▄███▌   | 1:FM; 3:R0; 4:FM; 6:R0; 7:FM; 9:R0; 13:FM; 14:R0, Fg, Bk; 15:FM; 21:Bm; 22:Bk; 37:Bm; 41:Bk; 43:R0; 44:FK; 46:R0; 47:FM; 55:FK; 59:R0; 60:FM; 61:Bm; 65:Bk; 66:R0; 67:FK; 71:FM; 72:Bm; 73:Bk; 97:Bm; 99:Bk; 104:R0
  ▀▄ ▀▄ ▀▄  █▄████ ▐ ████▄▄▄▄▄▄▄█ ▐█ ████      █ ████      █ ████     ▐ ████▄▄▄▄▄▄▄█  ▀█████▄▄▄▄████▀    | 3:FM; 5:R0; 6:FM; 8:R0; 9:FM; 11:R0; 13:FM; 14:Bm; 15:Bk; 21:Bm; 22:Bk; 25:Bm; 33:Bk; 37:Bm; 38:Bk; 42:R0; 48:FM; 49:Bm; 50:Bk; 55:R0; 60:FM; 61:Bm; 62:Bk; 66:R0; 71:FM; 72:Bm; 73:Bk; 76:Bm; 84:Bk; 86:R0; 87:FM; 91:Bm; 92:Bk; 93:Bm; 97:Bk; 102:R0
▐█▄ ▀▄ ▀▄ ▀▄ ▀████ ▐ ████████████ ▐█ ████      █ ████      █ ████     ▐ ████████████ ▀ ▄███████████▄ ▀   | 1:FM; 4:R0; 5:FM; 7:R0; 8:FM; 10:R0; 11:FM; 13:R0; 14:FM; 21:Bm; 22:Bk; 37:Bm; 38:Bk; 42:R0; 48:FM; 49:Bm; 50:Bk; 55:R0; 60:FM; 61:Bm; 62:Bk; 66:R0; 71:FM; 72:Bm; 73:Bk; 86:FK; 87:R0; 88:Fg; 89:FM; 100:R0, Fg, Bk; 101:R0; 102:FK; 104:R0
▐ ██▄ ▀▄ ▀▄ ▀▄ ▀██ ▐▄████         ▐█ ████▄    ▄█ ████      █ ████     ▐▄████          ▄████▀▀ ▀▀████▄▄   | 1:FM; 2:Bm; 3:Bk; 6:R0; 7:FM; 9:R0; 10:FM; 12:R0; 13:FM; 15:R0; 16:FM; 21:Bm; 22:Bk; 26:R0; 35:FM; 37:Bm; 38:Bk; 43:R0; 47:FM; 49:Bm; 50:Bk; 55:R0; 60:FM; 61:Bm; 62:Bk; 66:R0; 71:FM; 72:Bm; 73:Bk; 77:R0; 87:FM, Bm; 88:Bk; 92:Bm; 93:R0, Fg, Bk; 94:R0; 95:FM; 96:Bm; 97:Bk; 101:Bm; 102:R0, Fg, Bk; 104:R0
▐ ████  ▀▄ ▀▄ ▀▄ ▀ ▄▄▄▄▄▄▄▄▄▄▄▄▄▄  █▄▀█████████▄█████      █ ████     ▄▄▄▄▄▄▄▄▄▄▄▄▄▄ ▄▄▄▄▄▄  ▀▄  █████   | 1:FM; 2:Bm; 3:Bk; 7:R0; 9:FM; 11:R0; 12:FM; 14:R0; 15:FM; 17:R0; 18:FM; 35:R0; 36:FM; 37:Bm; 39:Bk; 46:Bm; 49:Bk; 55:R0; 60:FM; 61:Bm; 62:Bk; 66:R0; 71:FM; 92:R0; 94:FK; 96:R0; 97:FM, Bm; 98:Bk; 104:R0
▐ ████ ▀▄ ▀▄ ▀▄ ▀▄ ▄▄▄▄▄▄▄▄▄▄▄▄▄▄   ▀█▄▀▀██████████▀       █ ████     ▄▄▄▄▄▄▄▄▄▄▄▄▄▄ ▄▄▄▄▄▄   █ ▄▄▄▄▄▄   | 1:FM; 2:Bm; 3:Bk; 7:R0; 8:FK; 10:R0; 11:FM; 13:R0; 14:FM; 16:R0; 17:FM; 35:R0; 37:FM; 39:Bm; 42:Bk; 47:Bm; 48:Bk; 53:R0; 60:FM; 61:Bm; 62:Bk; 66:R0; 71:FM; 92:R0; 95:FK; 96:R0; 97:FM; 104:R0
▐▄████   ▀▄ ▀▄ █ █ ▄▄▄▄▄▄▄▄▄▄▄▄▄▄  ▀▄ ▀▀████████▀▀ ▄▀      █▄████     ▄▄▄▄▄▄▄▄▄▄▄▄▄▄ ▄▄▄▄▄▄   █ ▄▄▄▄▄▄   | 1:FM; 2:Bm; 3:Bk; 7:R0; 10:FK; 12:R0; 13:FM; 15:R0; 16:FM; 17:R0; 18:FM; 35:R0; 36:FK; 38:R0; 39:FM; 51:R0; 52:FK; 55:R0; 60:FM; 61:Bm; 62:Bk; 66:R0; 71:FM; 92:R0; 95:FK; 96:R0; 97:FM; 104:R0
▄▄▄▄▄▄     ▀▄▄ ▄ ▄ ▄▄▄▄▄▄▄▄▄▄▄▄▄▄    ▀▀▄▄▄▄▄▄▄▄▄▄▀▀        ▄▄▄▄▄▄     ▄▄▄▄▄▄▄▄▄▄▄▄▄▄ ▄▄▄▄▄▄   █▄▄▄▄▄▄▄   | 1:FK; 7:R0; 12:FK; 15:R0; 16:FK; 17:R0; 18:FK; 19:R0; 20:FK; 34:R0; 38:FK; 52:R0; 60:FK; 66:R0; 71:FK; 85:R0; 86:FK; 92:R0; 95:FK; 103:R0
```

This example shows:

- Metadata on the first line: `!V1; !TW104/104; !NL12;`
- Extensive use of `FM` (bright magenta) and `Bm` (magenta background) to create
  the gradient effect
- `R0` resets to return to default colors between colored zones

## Projects Using Neotex

- [splitans](https://github.com/badele/splitans) - Parses ANSI/ASCII files and
  separates text from CSI sequences

## Resources

### ANSI/VT Documentation

- [16colo.rs](https://16colo.rs) - ANSI art archive
- [WezTerm - SGR](https://wezterm.org/escape-sequences.html#graphic-rendition-sgr)
- [VT510 Reference](https://vt100.net/docs/vt510-rm/chapter4.html)
- [XTerm Control Sequences](https://invisible-island.net/xterm/ctlseqs/ctlseqs.html)
- [ECMA-48 Standard](https://ecma-international.org/wp-content/uploads/ECMA-48_5th_edition_june_1991.pdf)

### Tools

- [CODEF Ansi Logo Maker](https://n0namen0.github.io/CODEF_Ansi_Logo_Maker/) -
  ZETRAX font used for the logo
