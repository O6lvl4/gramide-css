# gramide-css 0.1.0

A separately buildable CSS **structural reader** for gramide v0.2.11.
Original implementation informed by [CSS Syntax Level 3](https://www.w3.org/TR/css-syntax-3/); no third-party parser source is vendored.

## Supported core

- Stylesheet rule lists, selector lists, type/class/id/attribute/pseudo selectors, namespaces and explicit combinators
- Declaration blocks and custom properties, including nested component-value blocks
- At-rules with preludes and semicolon/block forms: import, media, supports, layer, font-face and keyframe-shaped contents
- Nested style rules, `&` nesting selectors, and at-rules within style blocks
- Recursive `()`, `[]`, `{}` component values; comments and strings are atomic so delimiter-looking text inside them is harmless
- CSS escapes in names/strings, UTF-8 identifiers, numbers/dimensions/percentages, CRLF-aware byte coordinates

Rules, properties and at-rules appear in symbols/outline/tags. At-rule names are their written `@keyword`; parameters are available in source spans. The shared engine joins selector tokens for symbol names, so selector names omit inter-token whitespace (`article .card` becomes `article.card`). Treat these as display labels, not round-trippable selectors or unique semantic keys. Byte spans remain exact.

## Deliberate limits

Supported source line endings are LF and CRLF. Strict scanning rejects lone CR anywhere, including literals/comments, because the pinned engine otherwise gives inconsistent complete symbol line ranges. Byte offsets remain unchanged; recovery uses LF-based source coordinates.

This validates useful structural shape, not browser CSS acceptance. It does not validate property vocabularies or values, at-rule-specific semantics, selector pseudo-function arguments, whitespace-sensitive selector adjacency, units, media-query semantics or URL validity. CSS numbers, dimensions and percentages share the engine's `number` kind. The lexer retains original escapes rather than decoding them or applying CSS input preprocessing. Raw C0 controls except tab are rejected in quoted strings to protect selector-name JSON output (CSS escapes remain supported). It rejects unterminated comments/strings in strict mode even where browsers recover at EOF. Unknown at-rules follow generic syntax, but arbitrary future block grammars are not promised. No broad browser/WPT oracle suite has been run.

Capabilities: `tokens`, `parse`, `outline`, `symbols`, `tags`, `map`. No `check` or `symbols-recovered`; strict symbols require a complete structural parse, and outline recovery is only an editing aid.

## Build and validate

```sh
almide test
almide build cli/main.almd -o gramide_css
./gramide_css gen-table | diff -u src/table.almd -
./gramide_css symbols fixtures/components.css
```

Regenerate with `./gramide_css gen-table > src/table.almd` after grammar edits, then rebuild. `src/mod.almd` imports the generated table only. `src/fixture_test.almd` checks representative valid/invalid files, symbol names, and generated/dynamic tree equivalence. `ci/check.sh` exercises CLI byte ranges and capability gates. These fixtures demonstrate specific coverage, not full CSS conformance.

## Repository contract

This repository owns this language package and its tests. `src/mod.almd` exports
`definition()` using the shared gramide package API. The `gramide-cli` repository
composes it as a git dependency; no grammar source is vendored into the CLI.
`bash ci/check.sh` runs the complete package gate with an explicit test entry
point, avoiding recursive parallel compiler fan-out. CI pins Almide and Rust
in `.github/workflows/quality.yml`.
