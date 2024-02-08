
<!--#echo json="package.json" key="name" underline="=" -->
rosetta-stone-i18n-pmb
======================
<!--/#echo -->

<!--#echo json="package.json" key="description" -->
An internationalization (translation, i18n) library inspired by `rosetta`, but
with multi-language fallback support (e.g. en_CA || en_US || en || global).
<!--/#echo -->



Motivation
----------

There is a neat little translation library
[`rosetta`](https://github.com/lukeed/rosetta)
by [lukeed](https://github.com/lukeed).
It is optimized for tiny code size,
and thus it doesn't provide the extra features I want:

* __Multi-language fallback:__
  If a term isn't defined for Canadian English, try US English.
  If we don't have that either, try the languange-independent dictionary.
* __Custom fallback template:__
  If the term isn't found in any of the preferred languages,
  show a customizeable visible indication that something is missing.
* __Custom data template renderer:__
  Bring your own, if you even need one.



API
---

This module exports one function:

### makeRosettaStone(initDict?)

Return a new independent `function stone(key, params)` as described below.

* For compatibility with the `rosetta` library, if `initDict` is truthy,
  it's expected to be a multi-language dictionary object,
  and will be fed to `stone.multiLangLearn`.
  * However, to keep your project more modular, you may want to start without
    any vocabularies, and have a separate JS file inject them later,
    just before your app starts using them.
    * If your app exports a `window` global, this strategy makes it trivial
      to have your bundler run only once to generate a "no language" bundle,
      and then create language-specific bundles by just appending
      the appropriate vocabulary bundle file(s).



### stone(term, params?)

`term` is a string that says what to translate:

* If `term` contains `&?` (U+0026 ampersand, U+003F question mark),
  it's treated as a phrase template (see below).
* Otherwise, try `stone.t(term, params, lang)` for all languages `lang`
  in `stone.order` until a translation is found.

If a translation is found, render it using `stone.render`,
and return the result.

Otherwise, return `stone.missing`.
If it's a string (or something else that implements `.replace`),
`'\v'` (U+000B line tabulation) will be replaced with `term`.


### Phrase templates

Sometimes you want to compose a message based on multiple vocabulary terms.
In that case, you can use `&?word;` syntax to combine them.
The trailing `;` is optional when it's followed by whitespace,
and also at the end of the template.

Example: `stone('&?error;: &?missing_field;: &?email_addr')`

The words referenced here can themselves be defined using a phrase template.
In order to avoid infinite recursion, there is a limit (`stone.maxPhraseDepth`)
on how many replacement rounds are performed.

To avoid unexpected effects of user input that contains `&?` or `{{`,
phrase templates are resolved without user inputs.
Only the final resolved definition will then be `.render`ed.


### Reserved language names

Normal language names should start with a letter.
I recommend using ISO language codes.
A few reserved language names have special meaning:

* All reserved language names can be used in `stone.order` as if they
  were normal language names.

* The language name `*` (U+002A asterisk) means "locale-independent".
  You could use this to use Emoji to define your toolbar button icons.
  It is an automatic fallback for when none of the other languages in
  `stone.order` had a match.
  Independent of the fallback, you may always add `*` explicitly to give
  it priority over lesser fallbacks.
  I have no idea what the usecase could be, but it works.

* The language `=` (U+003D equals sign) means "verbatim".
  When encountered, it translates any input to exactly that input.

* The language `¬` (U+00AC not sign) means "give up".
  When encountered, `stone` skips all remaining potential fallback languages.


### stone.maxPhraseDepth

See "Phrase templates" above. Default: 10


### stone.order

An array of preferred languages. You should treat it as read-only.
Use `stone.locale` to update it.


### stone.render

Set this to a string render function if you want support for inserting
custom data into your phrases.
If truthy, `stone.render` is expected to be a function, and will be called
with arguments `(template, param, term)`,
where `template` is the translation text, `param` is the custom data,
and `term` is what `stone` got, potentially useful for debugging.

Default: `null`


### stone.missing

What to use if the term wasn't found in any of the translations.
This may be a translation function as described in `stone.set`.
Default: `'[?\v?]'`.



### stone.t(term, param?)

Alias for `stone`.
This is for compatibility with the `rosetta` library.

Note that there is no `lang` parameter here.


### stone.locale(langs?)

Return `stone.locales(langs).order[0]`.
This is for compatibility with the `rosetta` library.


### stone.locales(langs)

Update the list of preferred languages, then return the `stone`.

* If `langs` is false-y, just returns the `stone`, without any changes.
* `langs` should be a string with a list of names,
  separated by whitespace and/or U+002C comma (,).
* The most-preferred language goes first, optionally followed by fallbacks.
* Whatever truthy value you provide as `langs` will be `String()`ified first,
  so an array will work as well.


### stone.set(lang, vocab)

If `vocab` is the string `'\r'` (a single U+000D carriage return),
forget the entire language `lang`.

Otherwise, merge all enumerable entries `{ term: val }`
of dictionary object `vocab`
into the translation table for language `lang`.

If `Array.isArray(vocab)`, entries are expected as `[term, val]` pairs instead.

The `val`ues can be:

* The string `'\r'` (a single U+000D carriage return)
  to retract (delete) an entry.
* A false-y value, including the empty string, is ignored.
* A literal string: The easiest way to define a translation.
* A translation function `val(params)` that produces a translation.
  * More specifically: Anything value with a truthy `.call` property
    will be assumed to be a translation function that can be invoked as
    `val.call(stone, term, params, stone)`.
    The `stone` parameter is there for when you want to avoid using the
    `this` keyword for performance or policy reasons.
* Any value not covered by the rules above will be stringfied using
  `val.toString(stone, term)` and stored.


### stone.multiLangLearn(multiVoc)

Feed each enumerable entry `{ lang: vocab }` of dictionary object `multiVoc`
to `stone.set` .


### stone.table(lang?)

Neither implemented nor planned.
This is an intentional incompatibility with the `rosetta` library.



Usage
-----

:TODO:



Known issues
------------

* Needs more/better tests and docs.





<!--#toc stop="scan" -->

&nbsp;


License
-------
<!--#echo json="package.json" key="license" -->
ISC
<!--/#echo -->
