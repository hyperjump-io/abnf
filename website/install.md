# Install

To install the package, run the following command:

```bash
npm install @hyperjump/abnf
```

The package is an ES module with no dependencies, and it works the same in
Node.js and browsers. It exports one function, `abnf`:

```typescript
import { abnf } from "@hyperjump/abnf";
```

## TypeScript

The package includes its type declarations. It exports the following
types:

| Type | Description |
| --- | --- |
| `Context<T>` | What a pattern can know about its input. `T` is the type that the tag returns, a `RegExp` by default. |
| `Tag<T>` | The tag that `abnf(context)` returns: `(strings: TemplateStringsArray, ...values: string[]) => T` |

A context can say that its patterns have a more specific type than
`RegExp`. For more information, see [Contexts](contexts.md#types).

## Compile at build time

Compiling takes time when your module loads. If that matters, compile at
build time and save each pattern's `source`:

```typescript
const source = abnf()`4DIGIT "-" 2DIGIT "-" 2DIGIT`.source;
```

Then make the pattern from it, with the `u` flag:

```typescript
const fullDate = new RegExp(source, "u");
```
