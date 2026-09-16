# Template Syntax

Load this reference when authoring `.eta` content, interpolation, JavaScript control flow, whitespace trimming, or partial calls.

## Core Tags

Template data is available as `it` by default.

```eta
<!-- Escaped output -->
<h1><%= it.title %></h1>

<!-- Raw output: trusted HTML only -->
<main><%~ it.html %></main>

<!-- Execute JavaScript -->
<% const visibleItems = it.items.filter((item) => item.visible) %>

<!-- Comment -->
<% /* This does not appear in output. */ %>
```

Use `<%=` for normal dynamic data. Eta XML-escapes it when `autoEscape` is enabled. Use `<%~` only for trusted HTML or reviewed rendered fragments.

## Conditions and Loops

Eta uses normal JavaScript:

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

Keep complex business logic outside templates. Prepare view models before rendering when logic becomes difficult to scan or test.

## Partials

Partials return rendered markup, so include them through raw interpolation:

```eta
<%~ include("./header") %>
<%~ include("./header", { title: "Home" }) %>
```

For asynchronous partials:

```eta
<%~ await includeAsync("./header") %>
```

Use an async top-level rendering API when using `includeAsync()`.

Filesystem partials resolve from `views`. Programmatic partials use `@` names:

```eta
<%~ include("@header", { title: "Home" }) %>
```

## Whitespace Control

Opening delimiters may be followed by `-` or `_`; closing delimiters may be prefixed by them.

- `-` trims one adjacent newline.
- `_` trims all adjacent whitespace.

Use trimming only where exact output formatting requires it. Aggressive trimming reduces readability.

## Syntax Review Checklist

- Use `<%=` for every untrusted value.
- Review every `<%~` source.
- Ensure partial output is inserted as rendered markup intentionally.
- Ensure async partials are awaited inside an async render path.
- Keep template JavaScript focused on presentation.
- Test special characters and empty collections.

## Official References

- Template syntax: https://eta.js.org/docs/4.x.x/syntax/template-syntax
- Cheatsheet: https://eta.js.org/docs/4.x.x/syntax/cheatsheet
