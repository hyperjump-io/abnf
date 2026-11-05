# Compiling

The text is parsed, its references resolved into one tree of character
sets, sequences, alternatives, and repetitions, the tree rewritten into
what's faster and matches the same, and the tree compiled to a regular
expression source.

- Alternatives are flattened, and character sets among them merged into
  one.
- What's kept while compiling, like the compiled character sets, is kept
  for one compilation and dropped after it. Nothing is kept between calls,
  so nothing depends on what was compiled before.

## Rewrites

Each of these matches the same strings as what it replaces, on input the
context describes. They're what make compiled patterns as fast as
hand-written ones. They were measured with `@hyperjump/uri`'s scheme
patterns (see
[ABNF](https://github.com/hyperjump-io/uri/blob/v2/design/benchmarks/abnf.md)).

- **Percent-encoding as one character.** With `percentEncoded`,
  `( set / pct-encoded )` repeated without a bound matches the same as
  `( set / "%" )` when the set has every hex digit, unless a hex digit can
  follow the repetition: `*( unreserved / pct-encoded ) "1"` matches
  `%41` once `%` is a character, and doesn't before. So does one
  `( set / pct-encoded )` followed by a repetition whose set has every hex
  digit, which takes them.
- **Star identities.** `*( b *a )` is `[ b *( a / b ) ]`, and
  `n*a [ b *( a / b ) ]` is `n(a) *( a / b )`, for sets `a` and `b`. These
  take RFC 3986's `path-absolute`, `"/" [ segment-nz *( "/" segment ) ]`,
  from nested loops to one.
- **An option as a lookahead.** `[ a *b ]`, where `b` has every character
  of `a`, is `*b` not starting with what's only in `b`, if that can't
  follow it. `path-absolute` becomes `\/(?!\/)[^?#]*`, which is what a
  hand-written pattern would be. It was 5% faster than
  `\/(?:[^/?#][^?#]*)?`.
- **Repetitions of what can be empty.** A repetition without a bound of a
  repetition that can be empty is any number of its item: `1*( *a )` and
  `1*( [ a ] )` are `*a`. It's what `1*query` is, since RFC 3986's `query`
  can be empty. Nested, they backtrack through every way of splitting a
  string that doesn't match: `^(?:[^#]*)+$` took 15 milliseconds on 16
  characters and a `#`, and 606 on 24, where `^[^#]*$` took less than 1.
- **Classes.** A set is compiled as a class or a negated class, whichever
  has fewer ranges, and a negated class when they have as many. A range to
  the end of Unicode starts as low as the alphabet allows: V8 checks a
  class of all non-ASCII characters by checking for ASCII. A `urn:uuid`
  pattern took 282 milliseconds for 2 million tests when its range started
  at U+A0, and 254 when it started at U+7F.

## Rejected

- **Widening every range of a class**, not only the last. It turned single
  characters into classes, like `"?"` into `[>?]`, and the `urn:uuid`
  pattern was slower.
- **Caches of compiled sets or rules kept between calls**, keyed by a
  context. They'd be hidden channels. Kept for one compilation, the
  compiled sets were what brought `@hyperjump/uri`'s `http` host pattern,
  which inlines RFC 3986's `IP-literal`, from 22 milliseconds to 6.
