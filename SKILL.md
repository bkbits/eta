---
name: eta
description: Use when building, reviewing, migrating, or debugging Eta v4 templates in Node.js, Deno, browsers, Express, or Fastify. Covers ESM setup, rendering APIs, template syntax, escaping, partials, layouts and blocks, helpers, custom tags, configuration, integrations, and security.
---

# Eta v4

Use this skill for projects that use the Eta embedded JavaScript template engine, especially Eta v4.

## Core Rules

- Treat template source as executable JavaScript. Never pass untrusted or user-controlled template strings to Eta.
- Eta v4 is ESM-only. Use `import`, not `require()`.
- Keep `autoEscape` enabled unless the application has a reviewed escaping strategy.
- Use `<%=` for untrusted dynamic values. Use `<%~` only for trusted HTML or output already escaped by the application.
- Keep synchronous and asynchronous rendering consistent. Async templates, partials, captures, or blocks require the async APIs.
- Check the installed Eta version before applying v4 guidance. Do not copy v2 or v3 APIs into a v4 project.
- Preserve the project's package manager, module format, directory layout, and existing Eta conventions.

## Inspect the Project First

Before changing code:

1. Inspect `package.json`, lockfiles, runtime versions, and the installed `eta` version.
2. Find the shared `Eta` instance, its configuration, and all render call sites.
3. Locate the `views` directory and existing `.eta` templates.
4. Determine whether templates run in Node.js, Deno, a browser, or a framework integration.
5. Determine whether rendering is synchronous or asynchronous.
6. Check whether any template source, raw interpolation, custom tag, or helper handles untrusted data.

## Install and Initialize

Install Eta with the project's package manager. For npm:

```sh
npm install eta
```

### Node.js ESM

```js
import { Eta } from "eta"
import path from "node:path"

const eta = new Eta({
  views: path.join(import.meta.dirname, "templates"),
})

const html = eta.render("./simple", { name: "Ben" })
```

`import.meta.dirname` requires Node.js 20.11 or newer. For older supported Node.js versions, derive the directory from `import.meta.url`:

```js
import path from "node:path"
import { fileURLToPath } from "node:url"

const dirname = path.dirname(fileURLToPath(import.meta.url))
```

### Browser

Use the core build because browsers do not provide filesystem template loading:

```js
import { Eta } from "eta/core"

const eta = new Eta()
const html = eta.renderString("Hi <%= it.name %>!", { name: "Ben" })
```

Use a bundler or import map when the browser cannot resolve the bare `eta/core` specifier. Prefer `renderString()` or templates loaded with `loadTemplate()` in browser code.

### Deno

Prefer the JSR package:

```js
import { Eta } from "jsr:@bgub/eta"

const eta = new Eta({
  views: `${Deno.cwd()}/views/`,
  cache: true,
})
```

## Choose the Correct Rendering API

| API | Use |
| --- | --- |
| `eta.render(name, data)` | Render a filesystem or named template synchronously. |
| `eta.renderAsync(name, data)` | Render a filesystem or named template asynchronously. Await the result. |
| `eta.renderString(source, data)` | Render a trusted template string synchronously. |
| `eta.renderStringAsync(source, data)` | Render a trusted template string asynchronously. Await the result. |
| `eta.loadTemplate(name, source, options)` | Register a template programmatically. Use a name beginning with `@`. |

Template names passed to `render()` are resolved relative to `views`. Programmatic templates and other non-filesystem templates must use a leading `@`:

```js
eta.loadTemplate("@header", "<header><h1><%= it.title %></h1></header>")
const html = eta.render("@header", { title: "Home" })
```

Pass `{ async: true }` as the third `loadTemplate()` argument when the registered template is asynchronous.

## Template Syntax

Template data is available as `it` by default.

```eta
<!-- Escaped output: use for normal dynamic data -->
<h1><%= it.title %></h1>

<!-- Raw output: use only for trusted HTML -->
<main><%~ it.html %></main>

<!-- Execute JavaScript -->
<% const visibleItems = it.items.filter((item) => item.visible) %>

<!-- Comment -->
<% /* This does not appear in the output. */ %>
```

Use normal JavaScript for conditions and loops:

```eta
<% if (it.user) { %>
  <p>Hello, <%= it.user.name %></p>
<% } else { %>
  <p>Hello, guest</p>
<% } %>

<ul>
<% it.items.forEach((item) => { %>
  <li><%= item.label %></li>
<% }) %>
</ul>
```

### Whitespace Control

Use `-` to trim one adjacent newline and `_` to trim all adjacent whitespace. These markers follow the opening delimiter or precede the closing delimiter. Use them only when output formatting requires it; aggressive trimming can make templates difficult to read.

## Partials

Render partials as raw output because they return rendered markup:

```eta
<%~ include("./header") %>
<%~ include("./header", { title: "Home" }) %>
```

For asynchronous partials, use an async rendering API and await `includeAsync()`:

```eta
<%~ await includeAsync("./header") %>
```

Filesystem partials resolve from `views`. Use names beginning with `@` for templates registered programmatically:

```eta
<%~ include("@header", { title: "Home" }) %>
```

## Layouts and Blocks

A child template selects one parent layout with `layout()`. Layouts may themselves use parent layouts.

Child template:

```eta
<% layout("./base", { title: "Account" }) %>

<% block("sidebar", () => { %>
  <nav>Account navigation</nav>
<% }) %>

<h1><%= it.heading %></h1>
```

Layout template:

```eta
<!doctype html>
<html>
  <head>
    <title><%= it.title %></title>
  </head>
  <body>
    <aside><%~ block("sidebar", () => { %>Default sidebar<% }) %></aside>
    <main><%~ it.body %></main>
  </body>
</html>
```

Important behavior:

- Child output becomes `it.body` in the layout.
- A child-defined block overrides the layout fallback.
- An undefined block returns its fallback or nothing.
- Without an active layout, a block renders inline.
- Use `blockAsync()` and await it when block content is asynchronous.

## Built-in Helpers

### `output()`

`output(value)` appends directly to template output. Do not concatenate untrusted values into HTML passed to `output()` unless they are escaped first.

```eta
<% for (const item of it.items) {
  output("<li>" + item + "</li>")
} %>
```

Prefer ordinary `<%=` interpolation when values need Eta's automatic escaping.

### `capture()` and `captureAsync()`

Use `capture()` to store a rendered fragment for reuse:

```eta
<% const greeting = capture(() => { %>
  <h1>Hello, <%= it.name %>!</h1>
<% }) %>

<%= greeting %>
```

Use `captureAsync()` inside templates rendered through `renderAsync()` or `renderStringAsync()`:

```eta
<% const data = await captureAsync(async () => { %>
  <%= await it.fetchData() %>
<% }) %>

<%~ data %>
```

## Configuration

Create and reuse an `Eta` instance instead of constructing one per render.

```js
const eta = new Eta({
  views: templatesDirectory,
  cache: process.env.NODE_ENV === "production",
  autoEscape: true,
  debug: process.env.NODE_ENV !== "production",
})
```

Common options:

| Option | Meaning |
| --- | --- |
| `views` | Directory containing filesystem templates. |
| `defaultExtension` | Default template extension; default is `.eta`. |
| `autoEscape` | Automatically XML-escape interpolations; default is `true`. |
| `cache` | Cache templates; default is `false`. Enable deliberately in production. |
| `cacheFilepaths` | Cache resolved file paths. Set to `false` to disable. |
| `debug` | Improve runtime error formatting at a performance cost; default is `false`. |
| `tags` | Opening and closing delimiters; default is `["<%", "%>"]`. |
| `varName` | Template data variable name; default is `it`. |
| `useWith` | Expose data properties without `it`. Avoid it because of collisions and performance costs. |
| `functionHeader` | JavaScript inserted at the start of compiled functions. Use only trusted, static code. |
| `autoTrim` | Automatic whitespace trimming; default is `[false, "nl"]`. |
| `rmWhitespace` | Remove empty lines and inter-line whitespace. |
| `autoFilter` | Apply `filterFunction` to interpolated values. |
| `escapeFunction` | Replace the interpolation escaping function. Review security before changing it. |
| `customTags` | Map custom tag prefixes to handler functions. |
| `parse` | Configure evaluation, interpolation, and raw interpolation prefixes. |
| `plugins` | Add parser/compiler hooks. Review third-party plugins before use. |

Prefer `functionHeader` over `useWith` when selected fields need short local names:

```js
const eta = new Eta({
  functionHeader: "const { name, age } = it",
})
```

## Custom Tags

Custom tags receive static tag content and the full data object:

```js
const eta = new Eta({
  customTags: {
    "#": () => "",
    "*": (key, data) => translations[data.lang][key.trim()],
  },
})
```

```eta
<%# This emits nothing %>
<p><%* greeting %></p>
```

Custom tag handlers concatenate their return values directly into output without automatic escaping. Escape untrusted values in the handler. Prefixes cannot conflict with built-in prefixes (`=`, `~`, or the empty prefix) or whitespace markers (`-`, `_`).

## Framework Integrations

### Express

Eta v4 does not support passing `eta.render` directly to `app.engine()`. Prefer explicit rendering:

```js
app.get("/", (req, res) => {
  const html = eta.render("index", { title: "Home" })
  res.status(200).send(html)
})
```

When an application requires `res.render()`, provide an Express-compatible callback adapter that reads the file, calls `eta.renderString()`, and forwards errors to the callback. Do not register `app.engine("eta", eta.render)` directly.

### Fastify

Use `@fastify/view`:

```js
import fastify from "fastify"
import fastifyView from "@fastify/view"
import { Eta } from "eta"
import path from "node:path"

const eta = new Eta()
const server = fastify()

server.register(fastifyView, {
  engine: { eta },
  templates: path.join(import.meta.dirname, "views"),
})
```

Check framework plugin compatibility with Eta v4. Community integrations may lag behind Eta releases and are not necessarily security-vetted.

## Security Review

Eta compiles templates into JavaScript functions and does not sandbox execution.

Never do this:

```js
eta.renderString(req.body.template, data)
```

Pass user input as data instead:

```js
eta.renderString("Hello <%= it.name %>!", { name: req.body.name })
```

During review:

- Trace every template source to a trusted developer-controlled file or string.
- Audit every `<%~`, `output()`, custom tag, custom escape function, and helper that emits HTML.
- Keep user-controlled values behind `<%=` or an equivalent reviewed escaping function.
- Do not treat Eta as a sandbox.
- Use a logic-less template engine or an isolated runtime when templates must be user-authored.

## Troubleshooting

### `ERR_REQUIRE_ESM`

Eta v4 is ESM-only. Convert the call site to `import`, ensure the package is configured for ESM, or use the project's supported ESM interop strategy.

### `import.meta.dirname` is undefined

Use Node.js 20.11 or newer, or derive the path from `import.meta.url` with `fileURLToPath()`.

### Template not found

Check:

- `views` is an absolute path to the expected directory.
- The render name is relative to `views`.
- The file uses the configured `defaultExtension`.
- Programmatic templates use a leading `@`.
- Browser code is not attempting filesystem rendering.

### HTML appears escaped

Use `<%=` for text and untrusted values. Use `<%~` only when the value is intentional, trusted HTML. Do not disable global escaping to fix one raw fragment.

### Promise text or async syntax errors appear

Use `renderAsync()` or `renderStringAsync()`, await the call, and use `includeAsync()`, `captureAsync()`, or `blockAsync()` as appropriate.

### Template changes do not appear

Disable `cache` during development or clear the relevant Eta instance cache according to the installed version's API.

### Express rendering fails

Do not pass `eta.render` directly to `app.engine()`. Render explicitly or use an Express-compatible callback adapter.

## Validation Checklist

After changes:

1. Render representative templates with normal, empty, and special-character data.
2. Verify `<`, `>`, `&`, quotes, and user-provided HTML are escaped where expected.
3. Exercise partials, layouts, block fallbacks, and nested layouts when used.
4. Exercise async paths with real awaits and rejected promises.
5. Run the project's focused tests, type checks, lint checks, and build.
6. Test production cache behavior separately from development behavior.
7. Confirm no user-controlled string is compiled as a template.

## Official References

- Quickstart: https://eta.js.org/docs/4.x.x/intro/quickstart
- API overview: https://eta.js.org/docs/4.x.x/api/overview
- Configuration: https://eta.js.org/docs/4.x.x/api/configuration
- Security: https://eta.js.org/docs/4.x.x/intro/security
- Template syntax: https://eta.js.org/docs/4.x.x/syntax/template-syntax
- Layouts and blocks: https://eta.js.org/docs/4.x.x/syntax/layouts-and-blocks
- Helpers: https://eta.js.org/docs/4.x.x/syntax/helpers
- Custom tags: https://eta.js.org/docs/4.x.x/syntax/custom-tags
- Cheatsheet: https://eta.js.org/docs/4.x.x/syntax/cheatsheet
- Deno: https://eta.js.org/docs/4.x.x/resources/deno
- Express: https://eta.js.org/docs/4.x.x/resources/express
- Fastify: https://eta.js.org/docs/4.x.x/resources/fastify
- Integrations: https://eta.js.org/docs/4.x.x/resources/integrations
