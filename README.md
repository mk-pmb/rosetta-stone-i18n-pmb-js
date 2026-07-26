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
  If a headword isn't defined for Canadian English, try US English.
  If we don't have that either, try if we have a generic English translation,
  and as a last resort, check the languange-independent dictionary.
  (You could use the latter to define your toolbar button icons using Emoji.)
* __Custom fallback template:__
  If the headword isn't found in any of the preferred languages,
  show a customizeable visible indication that something is missing.
* __Custom data template renderer:__
  Bring your own, if you even need one.



Concepts
--------

### Headword charset

Supported characters for the headwords in your dictionaries are:

* From the "Basic Latin" Unicode block:
  Letters A-Z, a-z, Digits 0-9,
  U+002B plus sign (+),
  U+002D hyphen-minus (-),
  U+002E full stop (.),
  U+003A colon (:),
  U+005F low line (_)



### Phrase templates

Sometimes you want to compose a message based on multiple headwords.
In that case, you can use word slot syntax (`&?word;`) to combine them.
The trailing `;` is optional when it's followed by whitespace,
and also at the end of the template.

* __Example:__ `stone('&?error;: &?missing_field;: &?email_addr')`

* __Phrase markup in user-provided params:__
  To avoid unexpected effects of user input that contains `&?` or `{{`,
  phrase templates are resolved without user inputs.
  Only the final resolved definition will then be `.render`ed.

* __Recursive phrase definitions:__
  The headwords referenced in a phrase template can themselves be defined
  using a phrase template. In order to avoid infinite recursion,
  there is a limit on how many replacement rounds are performed.
  (Option `maxPhraseDepth`.)

* __Pre-resolving recursive phrases:__
  If you want your recursive phrases resolved beforehand,
  you'll have to use another library for that.
  This library here is meant for UI in web apps,
  where usually users' RAM limits are more important than
  a few milliseconds of CPU time.



### Reserved language names

Normal language names should be valid headwords.
I recommend using ISO language codes.
A few reserved language names have special meaning:

* All reserved language names can be used in `stone.locales` as if they
  were normal language names. They can thus appear in `stone.order`.

* The language name `*` (U+002A asterisk) means "locale-independent".
  It is an automatic fallback for when none of the other languages in
  `stone.order` had a match.
  Independent of the fallback, you may always add `*` explicitly to give
  it priority over lesser fallbacks.
  I have no idea what the usecase could be, but it works.

* The language `=` (U+003D equals sign) means "verbatim".
  When encountered, it translates any input to exactly that input.

* The language `¬` (U+00AC not sign) means "give up".
  When encountered, `stone` skips all remaining potential fallback languages.



API
---

This module exports one function:

### makeRosettaStone(opt?)

Return a new independent function `stone`.
The `opt.defaultMethod` option decides what the `stone` function does.

`opt` is an optional options object that supports these optional keys:

* `defaultMethod`: Decides what the `stone` function does.
  * Certain strings will make `stone` be a function that behaves the same
    as its homonymous method (`stone.word` etc.) would:
    * `'lookup'`
    * `'phrase'`
    * `'word'`
  * `undefined`, missing, or any false-y value: Same as `'phrase'`.

* `order`: If truthy, it's fed to `stone.locales()`.
* `initDict`: If truthy, it's fed to `stone.multiLangLearn()`.



### stone(…)

The stone function itself acts as an alias for
the method configured via `.defaultMethod`.



### stone.learn(lang, vocab)

Before your `stone` can translate anything, it needs to learn at least one
language.

If `vocab` is the string `'\r'` (a single U+000D carriage return),
forget the entire language `lang`.
Chosen because you should never need that in a translation string,
it's easy to encode in JSON (e.g. when loading `vocab` via JSON-RPC),
and it has nice mnemonics: `\r` = retract, CR = completely retract.

Otherwise, merge all enumerable entries `{ headword: tr }`
(`tr` = translation | translator)
of dictionary object `vocab`
into the translation table for language `lang`.

If `Array.isArray(vocab)`, entries are expected as
`[headword, tr]` pairs instead.

The `tr`anslation/`tr`anslator can be:

* The string `'\r'` (a single U+000D carriage return)
  to retract (delete) an entry.
* A false-y value, including the empty string, is ignored.
* A literal string: The easiest way to define a translation.
* A translation function `tr(params)` that produces a translation
  (see `.word` for details).
* Any value not covered by the rules above will be stringfied using
  `tr.toString(stone, headword)` and stored.


💡 Tip:

* To keep your project more modular, you may want to start without
  any vocabularies, and have a separate JS file inject them later,
  just before your app starts using them.
* If your app exports a `window` global, this strategy makes it trivial
  to have your bundler run only once to generate a "no language" bundle,
  and then create language-specific bundles by just appending
  the appropriate vocabulary bundle file(s).



### stone.set(lang, vocab)

Alias for `stone.learn`.
This is for compatibility with the `rosetta` library.



### stone.lookup(lang, headword, params?)

Lookup the raw definition for `headword` in language `lang`,
without the post-processing that `.word` would do.

The `lang` parameter goes first so you can easily `.bind(null, null)`
this method to use it as a callback for Array's `.map()`.

If `lang` is false-y (e.g. omitted), try all languages in `stone.order`
until a translation is found.

If no translation was found, return `undefined`.



### stone.word(headword, params?)

* First, `.lookup` the headword.
* If no translation was found, continue with `stone.missing` instead.
* If the interim result is a function, continue with its result.
  * More specifically: Any value with a truthy `tr.call` property will be
    assumed to be a translation function that can be invoked as
    `tr.call(stone, headword, params, stone)`.
    The final `stone` paramseter is there for when you want to avoid using
    the `this` keyword for performance or policy reasons.
* If the interim result is a string,
  or something else that implements `.replace`,
  use that to replace `'\v'` (U+000B line tabulation) with `headword`.
* If `stone.render` is truthy, it's expected to be a function, and will be
  called with arguments `(template, params, headword)`,
  where `template` is the interim result.



### stone.phrase(tpl, params?)

In phrase template `tpl`, replace all word slots with their result from
`.word`, except the potential `.render` step is postponed until all word
slots are filled in.

If no word slots were found, `.phrase` behaves as an alias for `.word`,
using `tpl` as the `headword` and forwarding `params`.
This fallback behavior, combined with being the `.defaultMethod`,
makes it so you can usually just call the `stone` function itself with
either a headword or a phrase template and it will do the right thing.



### stone.maxPhraseDepth

See "Phrase templates" above. Default: 10



### stone.order

An array of preferred languages. You should treat it as read-only.
Use `stone.locales` to update it.



### stone.render

Set this to a string render function if you want support for inserting
custom data into your phrases. See `.word` for exact details.
Default: `false`



### stone.missing

What to use if the headword wasn't found in any of the translations.
Can be a string, or a translation function.
See `stone.word` for exact details.
Default: `'[?\v?]'`.



### stone.t(headword, params?)

Alias for `stone.word`.
This is for partial compatibility with the `rosetta` library.
("Partial" because ours doesn't support the `lang` paramseter.)



### stone.locale(langs?)

Compatibility only – you won't usually need this.

Return `stone.locales(langs).order[0]`, i.e. the (resulting, new)
preferred language after optially setting it.

The name suggests a mere getter for `stone.order[0]` without the optional
update step. The result-of-optional-update pattern is for compatibility
with the `rosetta` library's `.locale`.



### stone.locales(langs)

Update the list of preferred languages, then return the `stone`.

* If `langs` is false-y, just returns the `stone`, without any changes.
* `langs` should be a string with a list of names,
  separated by whitespace and/or U+002C comma (,).
* The most-preferred language goes first, optionally followed by fallbacks.
* Whatever truthy value you provide as `langs` will be `String()`ified first,
  so an array will work as well.



### stone.table(lang?)

Neither implemented nor planned, because it cannot honor `stone.order`.
This is an intentional incompatibility with the `rosetta` library,
to discourage compatibility band-aids that would later bite you when
you do need multi-language fallback.



Usage
-----

See [test/usage.mjs](test/usage.mjs).



Known issues
------------

* Needs more/better tests and docs.





<!--#toc stop="scan" -->

&nbsp;


License
-------
<!--#echo json="package.json" key="license" -->
MIT
<!--/#echo -->
