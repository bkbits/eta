# Composition

Load this reference when building partials, layouts, blocks, reusable rendered fragments, or direct template output.

## Layouts

A child template selects one parent with `layout()`. Layouts may themselves have parents.

Child:

```eta
<% layout("./base", { title: "Account" }) %>
<h1><%= it.heading %></h1>
```

Layout:

```eta
<!doctype html>
<html>
  <head><title><%= it.title %></title></head>
  <body><main><%~ it.body %></main></body>
</html>
```

Child output becomes `it.body` in the layout. It is rendered markup, so layouts normally insert it with `<%~`.

## Blocks

Define named content in a child:

```eta
<% layout("./base") %>

<% block("sidebar", () => { %>
  <nav>Account navigation</nav>
<% }) %>
```

Render it in the layout, optionally with a fallback:

```eta
<aside>
  <%~ block("sidebar", () => { %>Default sidebar<% }) %>
</aside>
```

Behavior:

- A child-defined block overrides the layout fallback.
- An undefined block returns its fallback or nothing.
- Without an active layout, a block renders inline.
- Use `blockAsync()` and await it for asynchronous block content.

## `output()`

`output(value)` appends directly to template output:

```eta
<% for (const item of it.items) {
  output("<li>" + item + "</li>")
} %>
```

`output()` does not provide the safety of normal `<%=` interpolation. Escape untrusted values before concatenating them, or prefer ordinary interpolation.

## `capture()`

Capture a rendered fragment for reuse:

```eta
<% const greeting = capture(() => { %>
  <h1>Hello, <%= it.name %>!</h1>
<% }) %>

<%~ greeting %>
<%~ greeting %>
```

The captured value is rendered markup. Insert it as raw output only when its internal values were safely escaped.

## `captureAsync()`

Use inside an async render path:

```eta
<% const fragment = await captureAsync(async () => { %>
  <%= await it.fetchData() %>
<% }) %>

<%~ fragment %>
```

## Composition Checklist

- Keep layout data explicit.
- Use raw insertion for rendered template fragments, not arbitrary data.
- Review values concatenated through `output()`.
- Match `blockAsync()` and `captureAsync()` with async rendering.
- Test missing blocks, fallbacks, nested layouts, and partial failures.

## Official References

- Layouts and blocks: https://eta.js.org/docs/4.x.x/syntax/layouts-and-blocks
- Helpers: https://eta.js.org/docs/4.x.x/syntax/helpers
