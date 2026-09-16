# Troubleshooting and Migration

Load this reference for runtime errors, missing templates, async failures, stale output, framework failures, or migration to Eta v4.

## `ERR_REQUIRE_ESM`

Eta v4 is ESM-only.

- Replace `require("eta")` with `import { Eta } from "eta"`.
- Confirm the project's ESM configuration.
- Follow the project's supported ESM interop strategy instead of adding an ad hoc loader.

## `import.meta.dirname` Is Undefined

Use Node.js 20.11 or newer, or derive the directory from `import.meta.url`:

```js
import path from "node:path"
import { fileURLToPath } from "node:url"

const dirname = path.dirname(fileURLToPath(import.meta.url))
```

## Template Not Found

Check:

- `views` is an absolute path to the expected directory.
- The render name is relative to `views`.
- The file uses the configured `defaultExtension`.
- Programmatic templates use a leading `@`.
- Browser code is not attempting filesystem rendering.
- Custom `resolvePath` behavior returns the intended path.

## HTML Appears Escaped

Use `<%=` for text and untrusted values. Use `<%~` only for intentional trusted HTML or reviewed rendered fragments. Do not disable global escaping to fix one fragment.

## Promise Text or Async Syntax Errors

- Use `renderAsync()` or `renderStringAsync()`.
- Await the top-level call.
- Use and await `includeAsync()`, `captureAsync()`, or `blockAsync()`.
- Mark programmatic async templates with `{ async: true }`.

## Template Changes Do Not Appear

Disable `cache` during development or clear the relevant Eta instance cache using the installed version's API. Confirm the process is reading the expected views directory.

## Express Rendering Fails

Do not register `app.engine("eta", eta.render)`. Render explicitly or install an Express-compatible callback adapter that reads the file and calls `renderString()`.

## Browser Filesystem Errors

Use `eta/core`. Browser builds cannot load templates from a Node.js-style `views` directory. Bundle template source or register templates programmatically.

## Migration to Eta v4

1. Confirm the existing Eta version and read the project's lockfile.
2. Convert CommonJS imports to ESM.
3. Instantiate `Eta`; do not assume old singleton-style APIs.
4. Replace unsupported Express `app.engine()` registration.
5. Verify configuration option names against v4.
6. Verify filesystem paths and `import.meta.dirname` runtime support.
7. Retest escaped versus raw interpolation.
8. Convert async paths end to end, not one helper at a time.
9. Check every community plugin or adapter for v4 compatibility.

## Diagnostic Sequence

1. Reproduce with the smallest failing render call.
2. Log or inspect the Eta options without exposing secrets.
3. Confirm the resolved template path and template name type: filesystem or `@`.
4. Determine whether failure occurs during resolution, compilation, rendering, or response integration.
5. Test with cache disabled.
6. Test a minimal trusted template using escaped interpolation.
7. Apply the narrow fix, then rerun the project checks.

## Official References

- Quickstart: https://eta.js.org/docs/4.x.x/intro/quickstart
- API overview: https://eta.js.org/docs/4.x.x/api/overview
- Configuration: https://eta.js.org/docs/4.x.x/api/configuration
- Express: https://eta.js.org/docs/4.x.x/resources/express
