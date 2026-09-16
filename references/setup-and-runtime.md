# Setup and Runtime

Load this reference when installing Eta, creating an `Eta` instance, selecting imports, or configuring runtime paths.

## Install

Use the project's package manager. For npm:

```sh
npm install eta
```

Eta v4 is ESM-only. Use `import`, not `require()`.

## Node.js ESM

```js
import { Eta } from "eta"
import path from "node:path"

const eta = new Eta({
  views: path.join(import.meta.dirname, "templates"),
})
```

`import.meta.dirname` requires Node.js 20.11 or newer. For older supported Node.js versions:

```js
import path from "node:path"
import { fileURLToPath } from "node:url"

const dirname = path.dirname(fileURLToPath(import.meta.url))
const eta = new Eta({ views: path.join(dirname, "templates") })
```

Prefer an absolute `views` path. Create and reuse one Eta instance instead of constructing one per request.

## Browser

Import the browser-friendly core build:

```js
import { Eta } from "eta/core"

const eta = new Eta()
const html = eta.renderString("Hi <%= it.name %>!", { name: "Ben" })
```

Browsers do not provide filesystem template loading. Use `renderString()`, `renderStringAsync()`, or programmatically loaded templates. Configure a bundler or import map when the browser cannot resolve the bare `eta/core` specifier.

## Deno

Prefer JSR:

```js
import { Eta } from "jsr:@bgub/eta"

const eta = new Eta({
  views: `${Deno.cwd()}/views/`,
  cache: true,
})
```

## Environment Defaults

A typical shared instance:

```js
const eta = new Eta({
  views: templatesDirectory,
  cache: process.env.NODE_ENV === "production",
  debug: process.env.NODE_ENV !== "production",
  autoEscape: true,
})
```

Keep caching deliberate. Development usually needs uncached templates; production often benefits from caching.

## Setup Checklist

- Confirm `eta` major version is 4.
- Confirm the project is ESM-compatible.
- Confirm runtime support for `import.meta.dirname` or use `import.meta.url` fallback.
- Set `views` to the intended absolute directory for filesystem templates.
- Use `eta/core` in browsers.
- Reuse the Eta instance.

## Official References

- Quickstart: https://eta.js.org/docs/4.x.x/intro/quickstart
- API overview: https://eta.js.org/docs/4.x.x/api/overview
- Deno: https://eta.js.org/docs/4.x.x/resources/deno
