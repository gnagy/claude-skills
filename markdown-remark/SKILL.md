---
name: markdown-remark
description: Use this skill when formatting, linting, or validating markdown — running `mdfmt` (the markdown-toolbox CLI), setting up remark (the unified/mdast toolchain) in a project, adding plugins, writing a custom rule, or debugging output a formatter mangled. Covers the `mdfmt` command and its config, the remark plugin map, the escaping hazards that silently corrupt documents, matching IntelliJ's table style, and how to verify a formatter before letting it near real files. Also read it before running any bulk markdown reformat, with any tool.
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

## Reach for `mdfmt` first

A per-project remark install is usually the wrong move: it duplicates `node_modules` into every
project that has docs, and collides with whatever build the project already has. **`mdfmt` is
installed once per machine and is on `PATH`** — it ships remark, its plugins, and their config
together in one package (`markdown-toolbox`), so it formats any directory without that directory
gaining a single dependency:

```shell
mdfmt docs                    # format in place
mdfmt --check docs            # CI; exit 1 on unformatted files or lint problems
mdfmt --stdin < note.md       # format one document to stdout
```

It formats **byte-identically to IntelliJ's own formatter**, so the IDE's table inspection stays
quiet and the two never overwrite each other. It also exports an mdast toolkit (`read`, `write`,
`selectAll`, `getFrontmatter`, `setFrontmatter`) — import it by absolute path from a throwaway script
and reach for that instead of regexes when editing markdown structurally. Read the rest of this skill
anyway: everything below about escaping, table hazards, and verification applies to it, because it
*is* remark.

Optional per-project settings live in a `markdown-toolbox.config.mjs` at the project root — target
globs, serialiser overrides, front-matter JSON Schemas, link checking. It exports plain data, so the
project still needs no dependencies. Run `mdfmt --help` for the current flags.

> **If `mdfmt` is not on `PATH`** (check with `mdfmt --version`), this machine hasn't got it yet. It
> is **not on npm** — it lives at [github.com/gnagy/markdown-toolbox](https://github.com/gnagy/markdown-toolbox).
> From a checkout, `npm install` then `ln -s "$PWD/bin/mdfmt.mjs" ~/.local/bin/mdfmt` is enough —
> Node resolves the symlink to its real path, so the package's own dependencies still resolve. With
> no checkout, `npx --package=github:gnagy/markdown-toolbox -- mdfmt …` runs it without installing
> anything into the project; the `--package=` form is required, because `npx <spec>` treats a bare
> path or repo as a command to execute rather than a package to fetch.

> **Why can't remark itself be installed globally?** It resolves plugins relative to the *config
> file*, and ESM ignores `NODE_PATH`. A project-local `.remarkrc.mjs` can therefore only load plugins
> installed in that same project, and `npm install -g remark-cli` is invisible to it. The config and
> the plugins have to sit in one directory — which is precisely what `markdown-toolbox` packages, and
> why it works where a global remark doesn't.

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

| Hazard                       | Symptom                                                                                                     | Fix                                              |
|------------------------------|-------------------------------------------------------------------------------------------------------------|--------------------------------------------------|
| Custom inline syntax escaped | `[[link]]` → `\[\[link]]`, `{{var}}` → `\{{var}}`. Renders fine, means nothing to the tool that consumed it | Load the plugin that teaches remark the syntax   |
| Front matter destroyed       | YAML becomes a thematic break plus paragraphs                                                               | `remark-frontmatter`                             |
| Wiki embed escaped           | `![[note]]` → `!\[\[note]]`, even *with* `remark-wiki-link`, which tokenises only `[[…]]`                   | No plugin covers it; post-process (`mdfmt` does) |
| Lone `~` escaped             | "~15 rows" → "\\~15 rows"                                                                                   | No option exists; post-process or live with it   |
| Whole-document restyle       | Every `-` bullet becomes `*`, every list reindents                                                          | Pin `settings`                                   |
| Table misalignment           | Columns skew on any row with wide glyphs                                                                    | Custom `stringLength` (see below)                |
| `validate-links` aborts      | "Cannot find remote `origin`" on every file                                                                 | `{repository: false}`                            |

**A construct the parser never tokenises can't be fixed with an option.** Escaping is decided per
*text* node while serialising, so the levers are a plugin that turns the syntax into a node of its
own, or a post-pass over the serialised text. Embeds are the case where the plugin isn't enough:
`remark-wiki-link` matches `[[…]]` only, so `![[note]]` arrives as text and `![` has to be escaped or
it would re-read as an image. A post-pass must skip code spans and fences, or it corrupts the
documents most likely to contain the escaped form — the ones *about* escaping. `markdown-toolbox`'s
`lib/intellij-tables.mjs` is the worked example, undoing `\~` and `!\[\[` that way.

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

A formatter that reflows the whole document deserves an actual test, not a hopeful first run. Work on
a copy and read the diff before anything touches the real tree:

```shell
cp -R docs /tmp/verify
mdfmt /tmp/verify
diff -r docs /tmp/verify | grep -E '^[<>]' | grep -v '^[<>] *|'   # non-table changes
mdfmt --check /tmp/verify                                         # exit 0 ⇒ idempotent
```

With a hand-rolled per-project pipeline instead of `mdfmt`, the equivalent is
`npx remark /tmp/verify --rc-path .remarkrc.mjs --output` — and `--rc-path` is not optional there,
because config discovery walks up from the *input* files, so a copy outside the project would
otherwise be formatted with no config at all.

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

With `mdfmt` the same thing lives in `markdown-toolbox.config.mjs` and needs no plugin wiring:

```js
export default {
  files: ['docs/**/*.md'],
  schemas: {
    './.remark/note.json': ['docs/**/*.md'],
    './.remark/plan.json': ['docs/planned/*.md'],
  },
}
```

Schema paths resolve relative to the project, not to the tool, so the rules live next to the notes
they govern. They run under `--check` only; formatting stays purely mechanical.

## The worked example

`markdown-toolbox` is the reference implementation of everything above — pinned settings, a compiler
post-pass with fence skipping, IntelliJ-matched table layout, and glob-mapped schemas. If you need to
write your own pass, read `lib/intellij-tables.mjs` and the fixtures beside it first; the
`*.expected.md` files are real IntelliJ output, which is the only reliable way to know what the IDE
actually does rather than what its docs imply.

For a Foam wiki specifically, see the `wiki-docs` skill: Foam owns the link graph, remark owns the
documents.
