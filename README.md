lv_i18n - Internationalization for LittlevGL
============================================

[![Build Status](https://img.shields.io/travis/littlevgl/lv_i18n/master.svg?style=flat)](https://travis-ci.org/littlevgl/lv_i18n)
[![NPM version](https://img.shields.io/npm/v/lv_i18n.svg?style=flat)](https://www.npmjs.org/package/lv_i18n)

> Translate your firmware with ease!

**Work in progress**

Usage example
-------------

Create `translations.h` header for generated translation:

```c
// Library header
#include "lv_i18n.h"

// "translations" fill be generated by compiler tool
lv_i18n_translations_struct translations;

// Define shortcuts to wrap your texts
#define _(t)     lv_i18n_get_text(t)
#define _p(t, p) lv_i18n_get_text_plural(t, p)
```

Initialize i18n module, somewhere in your code:

```c
#include "translations.h"

// Load translations & default locale (usually, done once)
// Comment if translations are not compiled yet
lv_i18n_init(i18n_translations);

// Set active locale (can be switched anytime)
lv_i18n_set_locale("ru-RU");
```

Then, in all your sources:

```c
#include "translations.h"

const char* mytext = _("my text to translate");
const char* my_plural_text = _p("my text to translate", number);
```

CLI tools install ([node.js](https://nodejs.org/en/download/) required):

```sh
npm install -g lv_i18n
```


Translation workflow
--------------------

Here are simplified steps:

- Wrap texts in your code with `_()` and `_p()` wrappers. Without
  translations your program will continue work "as is", returning wrapped texts.
- Anytime, run "extractor" to analyze existing code, translations, and fill
  translation files with texts to translate.
- Translate (edit `.yml` files, replace `~` with translated texts).
- Run "compiler" to generate `.c` file and rebuild your program

If you change base phrases in source code, their existing translations become
"lost". CLI tools allow search such phrases, and "rename" or remove keys in all
translation at once, to minimize handwork.


C API
-----

### bool lv_i18n_init(lv_i18n_translations_struct translations)

Attach generated translations to be used by `lv_i18n_get_text()`.


### bool lv_i18n_set_locale(char* locale_name)

Set locale to be used by `lv_i18n_get_text()`.

- _locale_name_ - locale name, case insensitive (`en-GB`, `ru-RU`). If empty
  string passed - switch to default locale.
  - Locale names are not restricted to standard list, but only ASCII chars
    allowed (for simple normalization).

Returns `false` if locale not found (use default one then).


### char* lv_i18n_get_text(char* text)

Mapped to `_(...)` or `_t(...)` via `#define`

Get translated text. If not translated, return fallback (try default locale
first, then input param if default not exists)


### char* lv_i18n_get_text_plural(char* text, int32_t plural)

Mapped to `_p(...)` or `_tp(...)` via `#define`

Get plural form of translated text. Use current local to select proper plural
algorythm. If not translated, fallback to default locale first, then to input
param.


### CLI tools

CLI tools are used to simplify translation process and generate `.c` file with
optimized data. Utilities are written in JS and require
[node.js](https://nodejs.org/en/download/) to work. After node.js been
installed, type this:

```sh
npm install -g lv_i18n
```

This will install utilities and create shourtcut `lv_i18n` to call from terminal.
You can also use local install and type explicit path to `lv_i18n.js`:

```sh
npm install lv_i18n
```

All "actions" use similar pattern:

- Load source C/CPP files and extract phrases.
- Load yaml translations to understand existing progress.
- Do something useful (command):
  - Fill yaml files with new phrases to translate
  - Check errors and inconsistencies
  - Generate ("compile") `.c` file with optimized translation data.

See help:

```sh
lv_i18n -h
lv_i18n extract -h
lv_i18n compile -h
```

Examples:

```sh
lv_i18n extract -s 'src/**.+(c|cpp|h|hpp)' -s inc/** -t src/i18n/*.yml
lv_i18n compile -t src/i18n/*.yml -o src/translations.c
lv_i18n rename -t src/i18n/*.yml --from 'Hillo wold' --to 'Hello world!'
```

Translation files format
------------------------

Translations are stored in `.yaml` (or `.json`) files. Structure is similar to
one in [Ruby on Rails](https://guides.rubyonrails.org/i18n.html), with minor
simplifications:

```yaml
# Default locale
'en-GB':
  # May be useful if shortcuts used instead of real texts.
  # For example: `msg_file_missed`. When real texts used, default locale
  # doesn't need translation for singular phrases (leave "null" here)
  Settings: ~
  # Plurals - should be filled
  Nut:
    # Valid plural form names are `zero`, `one`, `two`, `few`, `many`, `other`
    # Each language has specific set of required forms.
    one: Nut
    many: Nuts
  'LVGL is awesome!': ~

# Example of additional locale. Usually, placed to separate file.
'ru-RU':
  Settings: Настройки
  Nut:
    one: Гайка
    few: Гайки
    many: Гаек
  'LVGL is awesome!': ~ # null (still untranslated), added by extractor
```

- No nesting subtrees, `foo.bar.baz` will be just a plain string

It's possible to store everything in one file, but we strongly recommend keep
each locale in separate one. When you run CLI tools and pass multiple
files, all data will be joined automatically.

If you wish add new locale - just create a new file with this content:

```yaml
your_new_locale_name: ~
```

Then run extractor to fill missed keys.


References:
-----------

To understand i18n principles better, you may find useful links below:

- [gettext](https://www.gnu.org/software/gettext/)
- [Rails Internationalization (I18n) API](https://guides.rubyonrails.org/i18n.html)
