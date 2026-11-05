# Text

```js
import { abnf } from "@hyperjump/abnf";

const date = abnf()`
  full-date     = date-fullyear "-" date-month "-" date-mday
  date-fullyear = 4DIGIT
  date-month    = 2DIGIT
  date-mday     = 2DIGIT
`;

date.test("2004-01-15"); // true
```

- `abnf(context?)` returns a tag, and the tag returns a `RegExp` with the
  `u` flag. A tag, because ABNF has no backslashes, so a template literal
  holds it as written, and it's compiled where it's written.
- The text is a rule list, and the first rule is the one matched. RFC 5234
  doesn't name a start rule, so this is the convention, and it's what lets
  an RFC's rules be pasted in with one line on top.
- Text that doesn't start with a rule is an expression, like
  `` abnf(path)`path-absolute` ``, and it can be followed by the rules it
  references: `` abnf(asciiPath)`NID ":" NSS ${urnRules}` ``. It's
  unambiguous, since a rule list starts with a name and `=`, and an
  expression ends where the first rule starts. So a pattern doesn't need a
  rule named only to be first, like `urn-path = NID ":" NSS`. It was
  written that way twice by mistake while building, when it was a
  `SyntaxError`.
- `${}` splices text in, so a rule list can be shared, like `urn`'s rules
  for its path and its query. A value that isn't a string is a
  `TypeError`.
- Rule names match without case, and so do strings, as RFC 5234 says.
  `%s"…"` is case-sensitive and `%i"…"` isn't (RFC 7405).
- Without a context, the result matches a whole string: it's anchored at
  both ends.

Parsing doesn't depend on lines: a rule starts where a name is followed by
`=` or `=/`, which can't happen inside a rule, so the text can be indented
any way.

## Scopes

A rule defined in the text replaces one of the same name from the context
or RFC 5234's core rules. RFC 9562 defines `DIGIT` and `HEXDIG` again, the
same way, and its rules have to paste in. Rules from a context are
resolved among the context's own rules, so a caller's `DIGIT` doesn't
change them.

## Errors

It throws a `SyntaxError` for text that isn't ABNF, a rule that's defined
twice in one rule list, a rule that isn't defined, a recursive rule, a
prose value (`<…>`), which can't be compiled, and a pattern that can't
match anything once a context's delimiters are removed (see
[Contexts](contexts.md)). A prose value repeated zero times is empty,
which RFC 3986's own grammar needs: `path-empty = 0<pchar>`.

## Rejected

- **Only a rule list or an expression**, with a rule named only to be
  first, like `urn-path = NID ":" NSS`. It's one shape fewer, but a
  caller reaches for an expression with its rules after it, and the
  start rule's name adds nothing.
- **Public functions for building patterns**, like `seq`, `alt`, and
  `rep`. They'd be checked by editors and types, but they're an API to
  design and name, they read like neither ABNF nor a regular expression,
  and a `raw(/…/u)` escape hatch would be wanted, which brings back
  writing regular expressions by hand. The parsed tree is made of the
  same parts, and it stays internal.
- **A string argument**, like `abnf(text, context)`. It works the same,
  but a template literal is where ABNF reads best, and `${}` is how rule
  lists are shared.
