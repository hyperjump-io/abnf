# Design

`@hyperjump/abnf` compiles ABNF (RFC 5234, with RFC 7405's case-sensitive
strings) into regular expressions. It was written in `@hyperjump/uri`,
for its schemes' patterns, and moved here so that `json-schema-formats`
can use it too. Nothing in it knows about URIs. What a caller knows about
its input is passed to it as a value (see
[Contexts](components/contexts.md)).

These documents record the decisions made so far, and why. The `.d.ts`
files in `src/` are the source of truth for the API. They're written by
hand and have JSDoc explaining the details. Documentation for users is in
[`website/`](../website/README.md).

## Goals

- Compile grammars the way their RFCs write them, so an RFC's rules can be
  pasted in.
- Compiled patterns as fast as hand-written ones.
- Small, with no dependencies, for Node.js and browsers. Compile at build
  time where loading matters.

## Use cases

- **`@hyperjump/uri`'s scheme patterns**: custom schemes' patterns, and
  the built-in schemes', which are generated from their RFCs' ABNF. See
  [ABNF in `@hyperjump/uri`](https://github.com/hyperjump-io/uri/blob/v2/design/components/abnf.md).
- **`json-schema-formats`**, whose formats are defined in ABNF by their
  RFCs, like RFC 3339's `date-time`.

Not supported on purpose:

- **Grammars a regular expression can't match.** A recursive rule throws.
- **Capturing or parsing.** A pattern only says whether a string matches.

## How it's organized

- **[Style](style/README.md)**: how code here is laid out and written.
- **Components**: how each part works, what was decided, and what was
  rejected. Open questions and known limitations are with the component
  they're about.

## Components

- [Text](components/text.md): rule lists and expressions, splicing,
  scopes, and errors
- [Contexts](components/contexts.md): what a caller knows about its input
- [Compiling](components/compiling.md): the rewrites that make patterns
  fast
