# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository reality

This repo contains exactly one file: [index.html](index.html), a ~2.5MB, self-contained, pre-built static web page. There is no `package.json`, no build config, no source tree, and no test suite — every prior commit was made via GitHub's "Add files via upload" web UI (see `git log`), not from a local build. Treat `index.html` as a compiled artifact, not hand-authored source.

There are no build, lint, or test commands to run, because none exist in this repo. Do not invent `npm run build`/`npm test` workflows — there is no `node_modules`, no lockfile, and no bundler config here.

## What's inside index.html

The file is a minified esbuild bundle inlined into a single `<script>` tag in `<head>`, mounted into `<div id="root">` via React 19's `createRoot(...).render(...)` (see the last line of the script). It bundles:

- **React 19 + react-dom** (development-free, minified, with the classic esbuild `ca()`/`Rn()` CJS-interop shims)
- **JSZip** — used for generating `.zip`/`.docx`/`.xlsx`-style archives client-side
- **xml-js** and related XML utilities — used for building/parsing OOXML
- A set of small inline SVG icon components (lucide-style, hand-rolled — search for `Re.jsx(or,...)` / `Re.jsxs(or,...)` patterns to find them)
- Large inline `data:image/png;base64,...` blobs representing rendered document pages (elements with classes like `pf`/`pc`/`bi`, i.e. "page frame" / "page content" / "background image" — this is a rendered-document-as-image viewer pattern, not hand-drawn UI)

The page title is "Office Register — Profiles & Directories", indicating this is a document/profile-register viewer-and-export tool: it renders paginated document images and uses JSZip/xml-js to let the user export data as an Office-compatible archive (docx/xlsx-style zip of XML parts).

## Working in this file

- **Lines are enormous.** The file is 1500ish lines but several individual lines exceed 100KB–300KB (minified JS bundle code, or base64 image data). Do not try to `Read` the whole file at once — it will exceed tool limits. Use `Grep` to locate what you need first, then `Read` with `offset`/`limit` targeted at specific line numbers.
- **Don't hand-edit the minified bundle logic unless necessary.** Variable names are single/double letters from esbuild's minifier and have no semantic meaning on their own; changes are easy to break silently. Prefer surgical, well-tested string replacements over broad rewrites.
- **The base64 image blocks are page renders, not logic.** If you need to change the app's behavior, look past the huge `data:image/...` lines — the interesting application code (icons, and any custom app logic beyond the vendor bundles) lives in the shorter surrounding lines near the top of the script and right before the final `createRoot(...).render(...)` call.
- If asked to add real functionality or fix a bug, first grep for distinguishing tokens (e.g. a nearby string constant, a class name, a JSX prop) to find the exact insertion point before editing — do not guess based on line numbers alone, since a single edit can shift a very long line.
