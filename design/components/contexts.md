# Contexts

A context says what's known about the input before the pattern runs.
It's a value:

```ts
type Context<T extends RegExp = RegExp> = {
  readonly rules?: string;          // a rule list the text can reference
  readonly alphabet?: string;       // ABNF for the characters the input can have
  readonly delimiters?: string;     // ABNF for the characters that end what's matched
  readonly percentEncoded?: boolean; // each "%" starts a pct-encoded
  readonly anchored?: boolean;      // whether to match a whole string
  readonly [result]?: T;            // only says what the tag returns
};
```

`alphabet` and `delimiters` are ABNF that matches one character, like
`"?" / "#"`, resolved among the context's rules. So they can say
`iunreserved / sub-delims`, and there's no second notation for sets. One
that doesn't match one character, like `"ab"`, is a `TypeError` when the
tag is made.

- **`delimiters` are removed from every character set.** A pattern can't
  match one, so it ends where they start. That's how a URI component's
  pattern stays in its component. A pattern that can't match anything once
  they're removed throws, like a path rule that only allows `?`. An
  alternative that can't is left out: `tag`'s
  `specific = *( pchar / "/" / "?" )` compiles for a path, without its
  `"?"`.
- **`alphabet` lets sets be widened.** A character the input can't have is
  free to be in a set or not, without changing what matches, so each set
  is compiled as whichever class is shortest. That's what lets `*pchar`
  compile to `[^/?#]*` where a URI's generic grammar has already checked
  the characters.
- **`percentEncoded`** says the input has already been checked to have two
  hex digits after each `%`, so `*( unreserved / pct-encoded )` can match
  `%` as one character (see [Compiling](compiling.md)).
- **`rules`** are the rules the text can reference, like RFC 3986's.
- **`anchored`** is `true` unless it's `false`. A pattern that goes inside
  a larger one, like a URI component's, isn't anchored.

The phantom `[result]` is how a context says what type the tag returns, so
`@hyperjump/uri`'s contexts make tags that return its `ComponentPattern`
brand, and neither package imports the other's code. `@hyperjump/uri`
imports the `Context` type.

A context is a value, not a configuration of the compiler, so the compiler
knows nothing about what a context is for. `@hyperjump/uri`'s contexts are
data: RFC 3986's and RFC 3987's rules, and each component's characters and
delimiters.

## Rejected

- **Knowing a rule by its name**, like leaving `:` in a URI host's sets
  only inside RFC 3986's `IP-literal`, which a prototype did. A compiler
  that knows the names of another specification's rules isn't general,
  and a context can't say "except inside this rule" without a notion of
  scope that only hosts would use. `@hyperjump/uri` leaves `:` out of a
  host's delimiters instead.
- **Renaming rules in the compiler**, like RFC 3986's to RFC 3987's, so
  that `pchar` means `ipchar` in an IRI. It's data: a context's rules can
  define `pchar = ipchar`.
- **A notation of its own for character sets**, like a string of
  characters. Sets that are ranges, like RFC 3987's `ucschar`, would need
  one too, and ABNF already has both.
