# Configuration and Extension

Load this reference when changing Eta options, escaping, filters, delimiters, custom tags, plugins, or template data access.

## Common Configuration

```js
const eta = new Eta({
  views: templatesDirectory,
  cache: process.env.NODE_ENV === "production",
  autoEscape: true,
  debug: process.env.NODE_ENV !== "production",
})
```

| Option | Meaning |
| --- | --- |
| `views` | Filesystem template directory. |
| `defaultExtension` | Default template extension; default `.eta`. |
| `autoEscape` | XML-escape interpolations; default `true`. |
| `cache` | Cache templates; default `false`. |
| `cacheFilepaths` | Cache resolved paths; set `false` to disable. |
| `debug` | Improve runtime error formatting at a performance cost. |
| `tags` | Delimiters; default `["<%", "%>"]`. |
| `varName` | Template data variable; default `it`. |
| `useWith` | Expose data properties without `it`. Avoid when possible. |
| `functionHeader` | Trusted JavaScript inserted into compiled functions. |
| `autoTrim` | Automatic whitespace trimming; default `[false, "nl"]`. |
| `rmWhitespace` | Remove empty lines and inter-line whitespace. |
| `autoFilter` | Apply `filterFunction` to interpolated values. |
| `filterFunction` | Function used when `autoFilter` is enabled. |
| `escapeFunction` | Interpolation escaping function. |
| `customTags` | Custom prefix-to-handler map. |
| `parse` | Evaluation, interpolation, and raw-prefix configuration. |
| `plugins` | Parser and compiler hooks. |

## Data Variable Access

Change the data variable name:

```js
const eta = new Eta({ varName: "data" })
```

Avoid `useWith` because it can cause naming collisions and reduce performance. Prefer a trusted static `functionHeader` for selected aliases:

```js
const eta = new Eta({
  functionHeader: "const { name, age } = it",
})
```

Never construct `functionHeader` from user input.

## Filtering

```js
const eta = new Eta({
  autoFilter: true,
  filterFunction: (value) => {
    if (typeof value === "string") return value.toUpperCase()
    return value
  },
})
```

Filtering and escaping solve different problems. Do not assume a filter sanitizes HTML unless it is explicitly designed and tested for that purpose.

## Custom Tags

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

Rules:

- Tag content is a static string, not evaluated JavaScript.
- The second handler argument is the full template data object.
- Handler return values are concatenated directly without automatic escaping.
- Prefixes cannot conflict with `=`, `~`, the empty prefix, `-`, or `_`.

Escape untrusted handler output explicitly.

## Plugins and File Handling

Review all plugin hooks before adoption. They can alter template source, ASTs, or compiled functions.

For custom storage, extend `Eta` and override `readFile` and `resolvePath`. Keep path resolution constrained to intended template locations.

## Configuration Review Checklist

- Preserve `autoEscape: true` unless a reviewed design replaces it.
- Audit custom `escapeFunction` and `filterFunction` behavior with special characters.
- Ensure `functionHeader` and custom tag definitions are static and trusted.
- Verify custom tag output escaping.
- Review plugin source and Eta v4 compatibility.
- Test cache and path behavior in production mode.

## Official References

- Configuration: https://eta.js.org/docs/4.x.x/api/configuration
- API overview: https://eta.js.org/docs/4.x.x/api/overview
- Custom tags: https://eta.js.org/docs/4.x.x/syntax/custom-tags
