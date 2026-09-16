# Rendering API

Load this reference when rendering files or strings, registering named templates, or choosing synchronous versus asynchronous APIs.

## API Selection

| API | Use |
| --- | --- |
| `eta.render(name, data)` | Render a filesystem or named template synchronously. |
| `eta.renderAsync(name, data)` | Render a filesystem or named template asynchronously. Await it. |
| `eta.renderString(source, data)` | Render a trusted template string synchronously. |
| `eta.renderStringAsync(source, data)` | Render a trusted template string asynchronously. Await it. |
| `eta.loadTemplate(name, source, options)` | Register a trusted template programmatically. |

## Filesystem Templates

Template names passed to `render()` resolve relative to `views`:

```js
const html = eta.render("./simple", { name: "Ben" })
```

Check the configured `defaultExtension` when a file cannot be found. Its default is `.eta`.

## Trusted Template Strings

```js
const html = eta.renderString("Hello <%= it.name %>", { name: "Ben" })
```

Template source is executable JavaScript. Never pass user input as `source`. Pass user input through the data object.

## Asynchronous Rendering

```js
const html = await eta.renderAsync("./account", { userId })
```

```js
const html = await eta.renderStringAsync(
  "Hello <%= await it.getName() %>",
  { getName: async () => "Ben" },
)
```

Once a template path uses async expressions, async partials, async captures, or async blocks, use async APIs for the whole path and await each async helper.

## Programmatic Templates

Name non-filesystem templates with a leading `@`:

```js
eta.loadTemplate("@header", "<header><h1><%= it.title %></h1></header>")
const html = eta.render("@header", { title: "Home" })
```

Mark an async template explicitly:

```js
eta.loadTemplate("@profile", source, { async: true })
const html = await eta.renderAsync("@profile", data)
```

## Resolution Rules

- Filesystem names resolve relative to `views`.
- Programmatic templates use names beginning with `@`.
- Browser code should not depend on filesystem resolution.
- Partial and layout names follow the same filesystem versus `@` distinction.

## Validation

- Render with expected, empty, and missing optional data.
- Verify asynchronous results are awaited.
- Verify rejected promises reach the application's error path.
- Verify named templates use `@` and filesystem templates resolve from `views`.
- Verify no untrusted string reaches a render method as template source.

## Official Reference

- API overview: https://eta.js.org/docs/4.x.x/api/overview
