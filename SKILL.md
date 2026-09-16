---
name: eta
description: Use when building, reviewing, migrating, or debugging Eta v4 templates in Node.js, Deno, browsers, Express, or Fastify. Routes each Eta task to focused guidance for setup, rendering, syntax, composition, configuration, integrations, security, and troubleshooting.
---

# Eta v4

Use this skill for Eta v4 work. Apply progressive disclosure: identify the task, load the smallest relevant reference, then load supporting references only when needed.

## Core Rules

- Treat template source as executable JavaScript. Never compile untrusted or user-controlled template strings.
- Eta v4 is ESM-only. Use `import`, not `require()`.
- Keep `autoEscape` enabled unless the application has a reviewed escaping strategy.
- Use `<%=` for untrusted dynamic values. Use `<%~` only for trusted HTML or reviewed rendered fragments.
- Match synchronous and asynchronous APIs throughout a render path.
- Check the installed Eta version before applying v4 guidance.
- Preserve the project's package manager, runtime, directory layout, and existing conventions.

## Start Here

Before changing code:

1. Inspect `package.json`, lockfiles, runtime versions, and the installed `eta` version.
2. Find the shared `Eta` instance, its options, and render call sites.
3. Locate `.eta` templates and determine how `views` is configured.
4. Determine whether the target runs in Node.js, Deno, a browser, or a framework.
5. Determine whether the render path is synchronous or asynchronous.
6. Identify all untrusted data and any raw-output path.

Do not load every reference by default. Choose a task below and read the primary reference first.

## Task Router

### Install or initialize Eta

Read [Setup and Runtime](references/setup-and-runtime.md).

Also read:

- [Rendering API](references/rendering-api.md) when adding the first render call.
- [Integrations](references/integrations.md) when using Express, Fastify, or Deno.
- [Troubleshooting and Migration](references/troubleshooting.md) when converting CommonJS or older Eta code.

### Render files, strings, or named templates

Read [Rendering API](references/rendering-api.md).

Also read:

- [Template Syntax](references/template-syntax.md) when authoring template content.
- [Composition](references/composition.md) when the render uses partials, layouts, blocks, or captures.
- [Security](references/security.md) when template source or output may contain user-controlled content.

### Author or edit `.eta` templates

Read [Template Syntax](references/template-syntax.md).

Also read:

- [Composition](references/composition.md) for partials, layouts, blocks, and reusable fragments.
- [Security](references/security.md) before adding `<%~`, `output()`, custom output helpers, or user-provided HTML.

### Build layouts, partials, blocks, or reusable fragments

Read [Composition](references/composition.md).

Also read [Rendering API](references/rendering-api.md) if any included content is asynchronous or registered programmatically.

### Configure or extend Eta

Read [Configuration and Extension](references/configuration.md).

Always also read [Security](references/security.md) when changing escaping, filtering, `functionHeader`, plugins, custom tags, or output behavior.

### Integrate Eta with a runtime or framework

Read [Integrations](references/integrations.md).

Also read:

- [Setup and Runtime](references/setup-and-runtime.md) for runtime-specific imports and paths.
- [Troubleshooting and Migration](references/troubleshooting.md) for `ERR_REQUIRE_ESM`, Express engine issues, or plugin compatibility.

### Review security or investigate XSS/code execution risk

Read [Security](references/security.md) first.

Then read the reference that owns the affected feature:

- [Template Syntax](references/template-syntax.md) for `<%=`, `<%~`, and partial output.
- [Composition](references/composition.md) for `output()`, layouts, blocks, and captures.
- [Configuration and Extension](references/configuration.md) for custom tags, filters, escape functions, and plugins.

### Debug errors or migrate existing code

Read [Troubleshooting and Migration](references/troubleshooting.md).

Load only the feature reference named by the failure. For example, load [Rendering API](references/rendering-api.md) for async failures or [Integrations](references/integrations.md) for framework failures.

## Common Workflow

1. Confirm Eta major version and runtime constraints.
2. Trace the target render path from Eta instance creation to final output.
3. Load the primary reference selected above.
4. Load supporting references only when the task crosses those concerns.
5. Make the smallest change that follows project conventions.
6. Validate normal, empty, special-character, and failure inputs.
7. Re-check every raw-output and template-source boundary.

## Validation Baseline

After changes:

1. Run focused tests, type checks, lint checks, and the relevant build.
2. Render representative templates with `<`, `>`, `&`, quotes, and empty values.
3. Exercise partials, layouts, block fallbacks, and async paths when used.
4. Confirm production cache behavior separately from development behavior.
5. Confirm no user-controlled string is compiled as a template.

## Official Documentation

The reference files summarize Eta v4 documentation. When behavior is version-sensitive or unclear, verify against https://eta.js.org/docs/4.x.x/intro/quickstart and the linked official page in the relevant reference.
