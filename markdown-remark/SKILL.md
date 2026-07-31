---
name: markdown-remark
description: Use this skill when formatting, linting, or validating markdown with remark (the unified/mdast toolchain) — setting it up in a project, adding plugins, writing a custom rule, or debugging output remark mangled. Covers the config, the plugin map, the escaping hazards that silently corrupt documents, table formatting, and how to verify a formatter before letting it near real files. Also read it before running any bulk markdown reformat, remark or otherwise.
---

# Markdown editing with remark

[remark](https://github.com/remarkjs/remark) is the markdown half of the
[unified](https://unifiedjs.com/) ecosystem: it parses markdown to an AST (**mdast**), runs plugins
over it, and serialises it back out. Use it for formatting, linting, link checking, front-matter
validation, and any structural rule you want enforced mechanically rather than by review.

## The one thing to understand first

**remark does not patch files — it re-serialises them.** Every run reparses the document to an AST
and writes a *fresh* document from that AST. Anything not represented in the AST is lost, and
anything the serialiser has an opinion about gets normalised whether you asked or not: list markers,
emphasis characters, fence style, escaping, table padding.

Two consequences that drive everything below:

- **A first run on an existing project produces a large diff** unless you pin `settings` to match the
  prevailing style. Budget for that; don't discover it on 200 files.
- **Any syntax remark doesn't know about is at risk.** It doesn't preserve unknown inline syntax
  verbatim — it escapes what looks ambiguous, so custom constructs come back as literal text.

## Before installing anything: `mdfmt`

A per-project remark install is often the wrong move — it duplicates `node_modules` into every
project that has docs, and collides with whatever build the project already has. Prefer
[`markdown-toolbox`](https://github.com/) (`tools/markdown-toolbox` in this workspace), which ships
remark, its plugins, and their config together in one package:

```shell
npx markdown-toolbox docs           # format in place
npx markdown-toolbox --check docs   # CI; exit 1 on unformatted files or lint problems
```

It formats **byte-identically to IntelliJ's own formatter**, so the IDE's table inspection stays
quiet and the two never overwrite each other. It also exports an mdast toolkit (`read`, `write`,
`selectAll`, `getFrontmatter`, `setFrontmatter`) for structural edits — reach for that instead of
regexes when editing markdown programmatically. Read the rest of this skill anyway: everything below
about escaping, table hazards, and verification applies to it, because it *is* remark.

> **Why not a global install?** remark resolves plugins relative to the *config file*, and ESM
> ignores `NODE_PATH`. A project-local `.remarkrc.mjs` can therefore only load plugins installed in
> that same project, and `npm install -g` is invisible to it. The config and the plugins have to live
> in one directory — which is exactly what the package is.

## Minimal setup

Only when a project genuinely needs its own pipeline (custom plugins, a bespoke lint rule):

```shell
npm install --save-dev remark-cli remark-gfm remark-frontmatter
```

```json
{
  "type": "module",
  "scripts": {
    "docs:check": "remark docs --frail --quiet",
    "docs:format": "remark docs --output --quiet"
  }
}
```

```js
// .remarkrc.mjs
import remarkFrontmatter from 'remark-frontmatter'
import remarkGfm from 'remark-gfm'

export default {
  settings: { bullet: '-', emphasis: '*', strong: '*', fence: '`', fences: true, rule: '-' },
  plugins: [remarkFrontmatter, remarkGfm],
}
```

`--frail` makes warnings exit non-zero, which is what you want in CI. Without `--output`, remark
writes nothing — **always start read-only**.

> **Config discovery walks up from each input file, not from the cwd.** Running remark over a path
> outside the project silently applies *no* config — no plugins, default style, maximum damage. Pass
> `--rc-path` when the target lives elsewhere, which is exactly the case when testing on a copy.

## Plugin map

| Plugin                           | Does                                                       | Notes                                                        |
|----------------------------------|------------------------------------------------------------|--------------------------------------------------------------|
| `remark-frontmatter`             | Keeps YAML front matter as an opaque node                  | **Without it front matter is parsed as prose and destroyed** |
| `remark-gfm`                     | Tables, strikethrough, autolinks, task lists, footnotes    | Without it, tables aren't tables — they're paragraphs        |
| `remark-lint` + a preset         | Style and consistency rules                                | `remark-preset-lint-recommended` is a sane start             |
| `remark-validate-links`          | Checks relative file links and heading anchors             | Needs `{repository: false}` with no git remote               |
| `remark-lint-frontmatter-schema` | Validates front matter against JSON Schema, mapped by glob | The best way to enforce a closed vocabulary                  |
| `remark-toc`                     | Generates/updates a table of contents                      | Writes into a designated heading                             |

## Hazards

These are the failure modes that cost real time. Most are silent — the document still *renders*
correctly, so review doesn't catch them.

| Hazard                       | Symptom                                                                                                     | Fix                                            |
|------------------------------|-------------------------------------------------------------------------------------------------------------|------------------------------------------------|
| Custom inline syntax escaped | `[[link]]` → `\[\[link]]`, `{{var}}` → `\{{var}}`. Renders fine, means nothing to the tool that consumed it | Load the plugin that teaches remark the syntax |
| Front matter destroyed       | YAML becomes a thematic break plus paragraphs                                                               | `remark-frontmatter`                           |
| Lone `~` escaped             | "~15 rows" → "\\~15 rows"                                                                                   | No option exists; post-process or live with it |
| Whole-document restyle       | Every `-` bullet becomes `*`, every list reindents                                                          | Pin `settings`                                 |
| Table misalignment           | Columns skew on any row with wide glyphs                                                                    | Custom `stringLength` (see below)              |
| `validate-links` aborts      | "Cannot find remote `origin`" on every file                                                                 | `{repository: false}`                          |

**Fenced code is safe** — code nodes are opaque, so mermaid diagrams, examples, and anything else
inside fences round-trip byte for byte. This is the one thing you don't have to worry about, and
it's worth knowing because mermaid edge labels (`A -->|"x"| B`) look alarmingly like table rows.

## Tables

`remark-gfm` aligns tables by default, and this is where it most often disagrees with an editor.
**Decide whose style wins before the first bulk run** — losing that fight quietly is worse than
either style, because every save then produces a diff.

**Width measurement.** The default is `String#length`. A `stringLength` that returns rendered columns
(wide characters count 2, combining marks 0) makes CJK and emoji tables line up in a fixed-width
font — but IntelliJ counts UTF-16 code units, so **adding it is what makes the IDE flag every table
containing a wide glyph.** Keep the default unless nothing else touches the files.

**Delimiter padding.** `tableCellPadding` applies to *every* row, so remark can produce `| --- |`
everywhere or `|---|` everywhere — but not the padded-cells-with-flush-delimiter hybrid that
IntelliJ emits:

```markdown
| Zone             | Hosted at    |
|------------------|--------------|
| `ol1.webhejj.hu` | DigitalOcean |
```

**Minimum width.** remark pads a delimiter to at least three dashes and widens the column to fit, so
`|:-:|` becomes `|:---:|`. IntelliJ keeps narrow aligned columns at content width. It also biases
centered content left where remark biases it right.

No combination of `settings` produces IntelliJ's output; a post-pass has to re-lay out the table.
`markdown-toolbox` already does all of the above — use it rather than rebuilding it, and if you must
write your own, its `lib/intellij-tables.mjs` and the IntelliJ-generated fixtures beside it are the
reference.

## Custom behaviour

For rules mdast can express, write a transform plugin. For output-shape tweaks the serialiser won't
do, wrap the compiler:

```js
function myPostPass() {
  const compile = this.compiler
  if (typeof compile !== 'function') throw new Error('use after remark-stringify')
  this.compiler = (tree, file) => transform(String(compile(tree, file)))
}
```

Two rules for anything in that pass:

- **Skip fenced code.** Track both fence markers (three backticks, or `~~~`) and pass those lines
  through, or you will corrupt the one part of the document that was previously safe. A mermaid edge
  label such as `A -->|"x"| B` is indistinguishable from a table row.
- **Keep width measurement consistent with the final output.** If the pass changes a line's length,
  any alignment remark already computed is now wrong. This is subtle and produces off-by-one column
  skew that's easy to miss — and it makes the formatter non-idempotent, so it thrashes git forever.
  Run anything that changes line length as its own pass *before* the pass that lays out tables.
- **Treat inline code spans as verbatim.** A backslash inside `` `\~` `` is literal text, not an
  escape; a pass that rewrites escapes without tracking backticks will corrupt documentation about
  escaping — which is exactly the kind of file you are most likely to be running it on.

## Verify before you let it write

A formatter that reflows the whole document deserves an actual test, not a hopeful `--output`:

```shell
cp -R docs /tmp/verify
npx remark /tmp/verify --rc-path .remarkrc.mjs --output    # note --rc-path
diff -r docs /tmp/verify | grep -E '^[<>]' | grep -v '^[<>] *|'   # non-table changes
```

Then check, in order of how badly each bites:

1. **Only intended lines changed** — grep the diff for anything you didn't expect to touch.
2. **Line counts unchanged**, if the pass shouldn't add or remove any.
3. **Custom syntax intact** — grep for the escaped form (`\[\[`, `\{{`).
4. **Fenced blocks byte-identical** — diff just the fences.
5. **Idempotent** — run twice, hash both times. A formatter that isn't idempotent will thrash git.

Step 5 catches the most bugs. Two formatters that disagree by one space will fight forever.

## Enforcing project rules

`remark-lint-frontmatter-schema` maps JSON Schemas to globs, which lets different directories carry
different requirements:

```js
[remarkLintFrontmatterSchema, {
  schemas: {
    './schemas/note.json': ['docs/**/*.md'],
    './schemas/plan.json': ['docs/planned/*.md'],
  },
}]
```

Use `additionalProperties: false` to reject invented fields, `enum` for closed vocabularies, and
`{"not": {"required": ["x"]}}` to forbid a field in one directory while requiring it in another.
This converts written-down conventions into failures, which is the entire point — prose conventions
decay, schema violations don't.

## In this repo

`.remarkrc.mjs` at the root is a worked example of all of the above: pinned settings, a custom
`stringLength`, a compiler post-pass with fence skipping, and glob-mapped schemas in `.remark/`. It
targets `docs/wiki/`, a Foam wiki — see the `wiki-docs` skill for what that adds.

```shell
npm run docs:check     # lint + validate + schema, no writes
npm run docs:format    # rewrite in place
```
