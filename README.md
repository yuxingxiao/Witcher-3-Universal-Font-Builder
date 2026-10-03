# Witcher 3 Universal Font Builder

A standalone Python tool for building a **Universal Font MOD** for **The
Witcher 3: Wild Hunt** from a compatible TrueType font and Witcher 3
font-resource template.

The builder converts font glyphs into Scaleform `DefineFont3` data,
replaces the main embedded font in the template SWF, recompresses the
SWF, writes it into a Witcher 3 bundle, and creates a ready-to-use MOD
directory.

> This is an unofficial fan-made development tool and is not affiliated
> with or endorsed by CD PROJEKT RED.

## What it does

The build pipeline is:

``` text
TTF / TTC
   ↓
Read Unicode BMP glyphs
   ↓
Convert TrueType outlines to SWF shape records
   ↓
Build DefineFont3
   ↓
Replace the main DefineFont3 in the template SWF
   ↓
Recompress the SWF as CWS
   ↓
Write the new SWF into the compatible bundle template
   ↓
Copy the matching metadata.store
   ↓
Generate a Witcher 3 Universal Font MOD
```

The tool can use either:

-   all compatible BMP characters available in the source font; or
-   a custom character set supplied with `--chars`.

## Requirements

-   Windows, Linux, or another environment capable of running Python 3
-   Python 3
-   `fontTools`

Install the Python dependency with:

``` cmd
py -m pip install fonttools
```

## Input files

The builder requires three inputs:

### 1. Font

A TrueType font containing the characters you want to embed.

Supported input containers:

``` text
.ttf
.ttc
```

For TTC files, the first font in the collection is used.

The current builder requires TrueType `glyf` outlines. CFF/CFF2 outlines
are not converted by this tool.

### 2. Template bundle

A compatible Witcher 3 font-resource bundle containing an embedded
compressed SWF font resource.

The template provides the Witcher 3 resource/container structure into
which the newly generated SWF is written.

### 3. metadata.store

The `metadata.store` associated with the compatible template bundle.

It is copied into the generated MOD together with the rebuilt bundle.

## Basic usage

``` cmd
py Witcher3_Universal_Font_Builder_Clean.py FONT TEMPLATE_BUNDLE METADATA_STORE
```

Example:

``` cmd
py Witcher3_Universal_Font_Builder_Clean.py ^
  "UniversalFont.ttf" ^
  "template.bundle" ^
  "metadata.store"
```

By default, the output directory is:

``` text
modUniversalWitcherFont\
```

You can choose another output directory with `-o`:

``` cmd
py Witcher3_Universal_Font_Builder_Clean.py ^
  "UniversalFont.ttf" ^
  "template.bundle" ^
  "metadata.store" ^
  -o "MyUniversalFontMod"
```

## Building only selected characters

By default, the builder uses all compatible Unicode BMP characters
available in the font.

You can instead provide a literal character set:

``` cmd
py Witcher3_Universal_Font_Builder_Clean.py ^
  "UniversalFont.ttf" ^
  "template.bundle" ^
  "metadata.store" ^
  --chars "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789ąćęłńóśźż中文"
```

Only characters that are both:

1.  included in `--chars`; and
2.  available in the source font

will be added.

## Output

A successful build creates a MOD directory similar to:

``` text
modUniversalWitcherFont\
├─ content\
│  ├─ bilingual.bundle
│  └─ metadata.store
├─ font_manifest.json
└─ build_report.json
```

### `bilingual.bundle`

Contains the rebuilt font resource with the generated `DefineFont3`.

### `metadata.store`

The matching metadata file used by the template.

### `font_manifest.json`

Contains SHA-256 information for the generated MOD resources.

### `build_report.json`

Contains technical information about the build, including:

-   builder version;
-   source font;
-   glyph count;
-   units per em;
-   SWF scale;
-   generated `DefineFont3` size;
-   first and last included codepoints;
-   incompatible glyphs skipped by the builder;
-   SWF and bundle sizes;
-   resource offset/capacity information;
-   SHA-256 hashes.

## DefineFont3 generation

The builder generates Scaleform/SWF `DefineFont3` data directly from
TrueType glyph information.

For each usable character it processes:

-   Unicode codepoint;
-   glyph outline;
-   line and quadratic curve segments;
-   advance width;
-   glyph bounds;
-   font ascent;
-   font descent;
-   line leading.

Font coordinates are converted to the coordinate scale used by
`DefineFont3`.

The resulting font uses wide offsets and wide character codes so that
Unicode BMP characters can be embedded.

## Unicode range

The builder works with Unicode codepoints in the **Basic Multilingual
Plane (BMP)**:

``` text
U+0000 – U+FFFF
```

Characters outside the BMP are not included.

The source font's `cmap` determines which characters are available.

## Glyph compatibility

`DefineFont3` stores glyph advance values as signed 16-bit integers.

If a glyph's scaled advance cannot be represented in that range, the
builder skips that glyph and records it in the build report instead of
failing the entire font build.

This allows the remaining compatible glyphs to be generated normally.

## SWF processing

The template bundle must contain a compressed `CWS` SWF resource.

The builder:

1.  locates the embedded CWS resource;
2.  decompresses it to FWS;
3.  parses the SWF tag stream;
4.  locates the first `DefineFont3` tag;
5.  replaces its payload with the generated font;
6.  updates the SWF length;
7.  recompresses it as CWS;
8.  writes it back into the template's resource area.

The generated SWF must fit within the resource capacity provided by the
template bundle. If it is too large, the build stops with an error
rather than overwriting data outside the available area.

## Installing the generated MOD

Copy the generated MOD directory into:

``` text
The Witcher 3\Mods\
```

For example:

``` text
The Witcher 3\
└─ Mods\
   └─ modUniversalWitcherFont\
      ├─ content\
      │  ├─ bilingual.bundle
      │  └─ metadata.store
      ├─ font_manifest.json
      └─ build_report.json
```

Whether the generated resource is loaded correctly depends on the
compatibility of the supplied template resources with the target
game/font slot.

Keep a backup of existing font MODs before testing a newly generated
build.

## Limitations

-   The current implementation supports TrueType `glyf` outlines.
-   CFF/CFF2 font outlines are not supported directly.
-   Only Unicode BMP codepoints (`U+0000`--`U+FFFF`) are included.
-   TTC input uses the first font in the collection.
-   The first/main `DefineFont3` in the template SWF is replaced.
-   The generated SWF must fit inside the template bundle's available
    resource area.
-   The supplied bundle and `metadata.store` must be compatible with the
    intended Witcher 3 font resource.

## Related tool

If you need a single source font containing both Latin and CJK
characters, the separate **Witcher 3 Noto Font Merger** can be used to
prepare a combined TrueType source font before running this builder.

The font merger is separate from this builder and is not required when
your input TTF already contains all characters you need.

## License and third-party material

The source code for this tool may be distributed under the license
provided with the project.

That license does not automatically grant redistribution rights for:

-   The Witcher 3 game resources;
-   CD PROJEKT RED / REDkit material or trademarks;
-   template bundles or metadata taken from the game;
-   third-party fonts.

Do not include or redistribute game resources or fonts unless you have
the appropriate rights to do so.

## Author

**Yuxing In Łódź**# Witcher-3-Universal-Font-Builder
Converts a compatible TrueType font into a DefineFont3-based Universal Font resource and builds it into a Witcher 3 font MOD using compatible template resources.
