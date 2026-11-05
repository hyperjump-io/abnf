# Write grammars

`abnf()` returns a tag. Write a grammar in a template literal after it,
and the tag returns a `RegExp` with the `u` flag:

```typescript
import { abnf } from "@hyperjump/abnf";

const time = abnf()`2DIGIT ":" 2DIGIT`;
time.test("09:30"); // true
```

## Rule lists

To write more than one rule, write a rule list. The first rule is the one
that the pattern matches, and the others are rules that it references:

```typescript
const fullDate = abnf()`
  full-date     = date-fullyear "-" date-month "-" date-mday
  date-fullyear = 4DIGIT
  date-month    = 2DIGIT
  date-mday     = 2DIGIT
`;
```

A grammar that starts with an expression, instead of a rule, matches that
expression. Rules after it are rules that it references:

```typescript
const fullDate = abnf()`
  date-fullyear "-" date-month "-" date-mday

  date-fullyear = 4DIGIT
  date-month    = 2DIGIT
  date-mday     = 2DIGIT
`;
```

Lines and indentation don't matter. A rule starts where `=` or `=/`
follows a name, so you can paste an RFC's rules in as they're written,
with their comments.

## What's supported

The grammar is ABNF as RFC 5234 defines it, with RFC 7405's
case-sensitive strings:

| ABNF | Matches |
| --- | --- |
| `"abc"` | `abc`, in any case |
| `%s"abc"` | `abc`, in this case only |
| `%i"abc"` | `abc`, in any case |
| `%x41`, `%d65`, `%b1000001` | The character with that code |
| `%x41-5A` | A character in that range of codes |
| `%x41.42.43` | Those characters in order |
| `a b` | `a`, then `b` |
| `a / b` | `a` or `b` |
| `( a b )` | A group |
| `[ a ]` | `a`, or nothing |
| `*a`, `1*a`, `*3a`, `2*3a`, `3a` | Repetitions of `a` |
| `a =/ b` | Adds `b` to the rule `a` as another alternative |

Rule names don't depend on case: `ALPHA` and `alpha` are the same rule.
RFC 5234's core rules, like `ALPHA`, `DIGIT`, `HEXDIG`, and `VCHAR`, are
always available. A rule that you define replaces a core rule with the same name.

## Share rules

To use the same rules in more than one pattern, put them in a string and
splice it in with `${}`:

```typescript
const dateRules = `
  date-fullyear = 4DIGIT
  date-month    = 2DIGIT
  date-mday     = 2DIGIT
`;

const fullDate = abnf()`
  date-fullyear "-" date-month "-" date-mday ${dateRules}
`;
const yearMonth = abnf()`date-fullyear "-" date-month ${dateRules}`;
```

## Errors

The tag throws a `SyntaxError` for a grammar that it can't compile:

- Text that isn't ABNF
- A rule that's defined twice
- A rule that's referenced but not defined
- A rule that references itself, either directly or through other
  rules. A regular expression can't match a recursive grammar.
- A prose value, like `<a date>`, which describes what it matches in
  words
- A grammar that can't match anything without a context's delimiters. For more information, see [Contexts](contexts.md#delimiters).

The tag throws a `TypeError` for a spliced value that isn't a string.
