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



API
---

:TODO:



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
MIT
<!--/#echo -->
