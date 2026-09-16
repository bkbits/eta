# Security

Load this reference first for security reviews and whenever work touches template source, raw output, custom tags, output helpers, escape functions, filters, or plugins.

## Threat Model

Eta templates compile to JavaScript functions. Rendering template source is equivalent to executing code. Eta does not sandbox templates.

Never compile user-controlled source:

```js
// Dangerous: equivalent to evaluating user input.
eta.renderString(req.body.template, data)
```

Use a developer-controlled template and pass user input as data:

```js
eta.renderString("Hello <%= it.name %>!", { name: req.body.name })
```

If users must author templates, use a logic-less engine or execute templates inside a purpose-built isolated environment.

## Output Boundaries

Safe default:

```eta
<p><%= it.userInput %></p>
```

Raw output requires review:

```eta
<div><%~ it.html %></div>
```

Audit these output paths:

- `<%~ value %>`
- `output(value)`
- custom tag return values
- custom `escapeFunction`
- custom filters assumed to sanitize content
- `it.body`, partials, blocks, and captured fragments inserted as raw markup
- helpers that return HTML

Rendered fragments may be inserted raw only when every dynamic value inside them was escaped or otherwise made safe.

## Configuration Risks

- Disabling `autoEscape` expands XSS risk across all templates.
- `functionHeader` inserts JavaScript into compiled functions and must remain trusted and static.
- `useWith` increases ambiguity and can cause collisions; it is not a security boundary.
- Plugins can transform source, ASTs, or generated functions. Review plugin source and version compatibility.
- Custom tag output is not auto-escaped.
- Custom file resolution must prevent unintended path access.

## Review Procedure

1. Trace every template source to a developer-controlled file or static string.
2. Search templates for `<%~` and code for `renderString`, `loadTemplate`, `output`, `customTags`, `escapeFunction`, `functionHeader`, and `plugins`.
3. Trace data entering each raw-output path.
4. Verify normal values use `<%=` with `autoEscape` enabled.
5. Verify custom tag and helper output escapes untrusted values.
6. Test payloads containing `<script>`, event handlers, quotes, `<`, `>`, and `&`.
7. Verify errors do not expose sensitive template source or data in production.

## Security Validation

Test at minimum:

```text
<script>alert(1)</script>
"><img src=x onerror=alert(1)>
'&<>
```

Expected behavior depends on output context, but normal interpolation must not produce executable markup.

## Official Reference

- Security: https://eta.js.org/docs/4.x.x/intro/security
