# ABNF

`@hyperjump/abnf` compiles grammars written in Augmented Backus-Naur Form
(ABNF) into regular expressions. ABNF is the notation that RFCs use for
syntax, so you can paste an RFC's rules in and get a `RegExp` that matches
what the RFC describes.

The following example compiles RFC 3339's `full-date`:

```typescript
import { abnf } from "@hyperjump/abnf";

const fullDate = abnf()`
  full-date     = date-fullyear "-" date-month "-" date-mday
  date-fullyear = 4DIGIT
  date-month    = 2DIGIT
  date-mday     = 2DIGIT
`;

fullDate.test("2004-01-15"); // true
fullDate.test("2004-1-15"); // false
```

The package offers the following:

- **RFC 5234 and RFC 7405.** Rules, repetition, options, groups,
  numeric values, and case-insensitive and case-sensitive strings, with
  RFC 5234's core rules, like `ALPHA` and `DIGIT`, built in.
- **A regular expression.** The result is a `RegExp` with the `u` flag,
  as fast as one written by hand, which you can test strings with or
  compile once at build time.
- **What you know about the input.** A context can supply rules to
  reference, characters that end a match, and characters the input can't
  have. For more information, see [Contexts](contexts.md).
- **Small.** No dependencies. It has TypeScript types and works in Node.js
  and browsers.

[`@hyperjump/uri`](https://github.com/hyperjump-io/uri/blob/v2/website/README.md)
uses it for the patterns of URI schemes.

## Contents

- [Install](install.md): setup for Node.js, browsers, and TypeScript
- [Write grammars](grammars.md): rule lists, expressions, sharing rules,
  and errors
- [Contexts](contexts.md): rules to reference, delimiters, and what the
  input can have
