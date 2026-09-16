# Integrations

Load this reference when connecting Eta to Express, Fastify, Deno, browser tooling, or a community integration.

## Express

Eta v4 does not support passing `eta.render` directly to `app.engine()`. Prefer explicit rendering:

```js
import express from "express"
import path from "node:path"
import { Eta } from "eta"

const app = express()
const eta = new Eta({
  views: path.join(import.meta.dirname, "views"),
  cache: true,
})

app.get("/", (req, res) => {
  const html = eta.render("index", { title: "Home" })
  res.status(200).send(html)
})
```

When an application requires `res.render()`, provide an Express-compatible callback adapter:

```js
function buildEtaEngine(eta) {
  return (filePath, options, callback) => {
    try {
      const source = eta.readFile(filePath)
      callback(null, eta.renderString(source, options))
    } catch (error) {
      callback(error)
    }
  }
}
```

Register the adapter, not `eta.render` directly.

## Fastify

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

Check the installed `@fastify/view` version and verify Eta v4 compatibility.

## Deno

Prefer JSR:

```js
import { Eta } from "jsr:@bgub/eta"

const eta = new Eta({
  views: `${Deno.cwd()}/views/`,
  cache: true,
})
```

## Browser

Use `eta/core`. Browser code normally renders trusted strings or programmatically registered templates because filesystem APIs are unavailable.

## Community Integrations

Community integrations are not necessarily official, security-vetted, or current with Eta v4. Inspect source, release dates, peer dependencies, and open compatibility issues before adoption.

Documented ecosystem examples include Opine, Alosaur, Fastify, Koa middleware, Rollup plugins, editor extensions, and ESLint plugins.

## Integration Checklist

- Reuse a shared Eta instance.
- Confirm runtime and framework versions.
- Confirm Eta v4 compatibility for adapters and plugins.
- Propagate synchronous and asynchronous errors correctly.
- Keep template paths constrained to the intended views directory.
- Test production cache behavior.
- Apply framework-specific escaping and response headers as needed.

## Official References

- Express: https://eta.js.org/docs/4.x.x/resources/express
- Fastify: https://eta.js.org/docs/4.x.x/resources/fastify
- Deno: https://eta.js.org/docs/4.x.x/resources/deno
- Integrations: https://eta.js.org/docs/4.x.x/resources/integrations
