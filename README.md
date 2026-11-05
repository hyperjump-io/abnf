# Hyperjump - ABNF

Compile grammars written in
[ABNF](https://www.rfc-editor.org/rfc/rfc5234) into regular expressions.
ABNF is the notation that RFCs use for syntax, so you can paste an RFC's
rules in and get a `RegExp` that matches what the RFC describes.

**[Read the documentation at hyperjump.io](https://hyperjump.io/abnf/)**

```bash
npm install @hyperjump/abnf
```

```typescript
import { abnf } from "@hyperjump/abnf";

const fullDate = abnf()`
  full-date     = date-fullyear "-" date-month "-" date-mday
  date-fullyear = 4DIGIT
  date-month    = 2DIGIT
  date-mday     = 2DIGIT
`;

fullDate.test("2004-01-15"); // true
```

- **RFC 5234 and RFC 7405.** Rules, repetition, options, groups, numeric
  values, and case-insensitive and case-sensitive strings, with the core
  rules built in.
- **A regular expression.** The result is a `RegExp` with the `u` flag,
  as fast as one written by hand.
- **What you know about the input.** A context can supply rules to
  reference, characters that end a match, and characters the input can't
  have.
- **Small.** No dependencies. TypeScript types included. Works in Node.js
  and browsers.

## Documentation

- [Install](https://hyperjump.io/abnf/install/)
- [Write grammars](https://hyperjump.io/abnf/grammars/)
- [Contexts](https://hyperjump.io/abnf/contexts/)

## Contributing

See [`design/`](design/README.md) for how it works and why.

```bash
npm install
npm test
npm run lint
npm run type-check
```

## License

MIT
