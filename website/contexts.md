# Contexts

A context describes what you already know about the input. Give it to
`abnf`, and every pattern that the tag makes uses it:

```typescript
import { abnf } from "@hyperjump/abnf";

const tag = abnf({
  rules: `
    date-fullyear = 4DIGIT
    date-month    = 2DIGIT
    date-mday     = 2DIGIT
  `
});

const fullDate = tag`date-fullyear "-" date-month "-" date-mday`;
const yearMonth = tag`date-fullyear "-" date-month`;
```

A context has the following properties, all optional:

| Property | Description |
| --- | --- |
| `rules` | A rule list that patterns can reference. Defaults to none. |
| `alphabet` | ABNF that matches one character: every character the input can have. Defaults to every character. |
| `delimiters` | ABNF that matches one character: the characters that end a match. Defaults to none. |
| `percentEncoded` | Whether you already know that two hex digits follow each `%` in the input. Defaults to `false`. |
| `anchored` | Whether a pattern matches a whole string, with `^` and `$`. Defaults to `true`. |

## Rules

A context's rules are rules that patterns can reference, like a standard
that other grammars build on. A rule in a pattern replaces a context's
rule with the same name, but only in that pattern. The context's own
rules still use the context's rule.

`alphabet` and `delimiters` can reference the context's rules too.

## Delimiters

A pattern can't match a delimiter. `abnf` removes the delimiters from
every set of characters in the grammar, so a pattern ends where a
delimiter starts:

```typescript
const path = abnf({ delimiters: `"?" / "#"` })`*VCHAR`;

path.test("/a/b"); // true
path.test("/a?b"); // false
```

`abnf` leaves out an alternative that can only match a delimiter. A
grammar that can't match anything without the delimiters throws a
`SyntaxError`.

## Alphabet

When you've already checked your input, some characters can't be in
it. Give the characters it can have as the `alphabet`, and `abnf` can
compile a set of characters to a shorter regular expression, which can be
faster. A pattern matches the same strings that are in the alphabet
either way.

`percentEncoded` is for input where you've already checked that two hex
digits follow each `%`, like a valid URI. Then a repetition of
characters and `pct-encoded` can match `%` as one character, which is
faster.

## Anchoring

By default, a pattern matches a whole string. With `anchored: false`, a
pattern matches anywhere in a string, without `^` and `$`. Use it for a
pattern that you put inside a larger one by its `source`.

## Types

`Context<T>` has a type parameter for what the tag returns. A context can
say that its patterns are a more specific type than `RegExp`, like a type
that another function requires:

```typescript
import { abnf } from "@hyperjump/abnf";
import type { Context } from "@hyperjump/abnf";

type DatePattern = RegExp & { readonly date: true };
const dates: Context<DatePattern> = { rules: `year = 4DIGIT` };

abnf(dates)`year "-" 2DIGIT`; // DatePattern
```

The type is only in the declarations. At runtime, every pattern is a
`RegExp`.
