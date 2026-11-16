# AGENTS.md

## Project Purpose and Current State

- This project is a Raycast extension for quickly opening [Metrix.Bar](https://metrix.bar/) dashboards.
- Dashboards run in the local native Metrix.Bar app. Verify how to open a dashboard before implementing an integration; opening the website homepage does not complete dashboard integration.
- The extension entry point is `src/search-dashboards.tsx`, which uses Raycast's native list UI. See `README.md` for installation and configuration.
- Use English for project documentation, user-facing text, issues, and pull requests. Write this file, `README.md`, and `CHANGELOG.md` in English.

## Toolchain

- Use Vite+, Node.js 24.22.2, and npm 11.19.1. Versions are recorded in `.node-version` and `package.json`. Install dependencies from the official npm registry configured in `.npmrc`.
- Use the Raycast CLI bundled with `@raycast/api` for extension development and production builds. Vite+ handles environment management, task execution, and general checks. This extension does not use a standard Vite web app build entry point.
- Vite+'s built-in `vp dev` and `vp build` commands do not run the project scripts with those names. Use `vp run dev` and `vp run build` to invoke the Raycast CLI.
- Keep the extension-specific rules from Raycast's official ESLint configuration. Vite+'s general checks do not replace Raycast lint.
- OxFmt in `vite.config.ts` and Prettier in `.prettierrc.json` both use a print width of 120. Keep them in sync; Raycast lint uses the Prettier configuration.

## Common Commands

- `npm ci`: install dependencies from `package-lock.json`. Use `npm install` when updating dependencies.
- `vp run dev`: start Raycast development mode; Raycast must be installed locally.
- `vp run build`: run the production build and Raycast type checks; output goes to `dist/`.
- `vp check`: run OxFmt, Oxlint, and type-aware checks. Use `vp fmt` to format files.
- `vp run lint`: check the Raycast manifest, icons, ESLint rules, and Prettier formatting.
- `vp run typecheck`: run `tsc --noEmit` independently.
- `vp test --run`: run all behavior tests. Use `vp test --run test/metrix.test.ts` for a single test file.
- If `vp` is not installed globally, use the corresponding npm scripts in `README.md` to invoke the project's local Vite+ installation.

## Validation Requirements

- CI is configured in `.github/workflows/check.yml`. Update it when installation steps or check commands change.
- After changing extension code or configuration, run Raycast lint, the production build, and the configured formatting and type checks. Run relevant behavior tests when available. The official Raycast CLI production build includes additional type checks.
- Run the production build in Raycast and confirm that the command opens the intended dashboard. A successful terminal build does not replace desktop interaction testing. If desktop testing is unavailable, explicitly record what remains unverified.
- Local API integration must cover the dashboard list and selected item, the app not running, an incorrect address or token, and no installed dashboards. Test request logic with simulated responses; never put real tokens in test fixtures.

## Raycast and Metrix.Bar Integration

- When creating the manifest, command names must match their source entry files. Use `no-view` for commands that execute without a UI, and native Raycast components when users need to select a dashboard.
- Obtain the local API address and Bearer token from the app's Agent page. Port `6387` in the documentation is only an example; do not assume every installation uses it.
- `GET /v0/dashboards` lists dashboards. `title` is optional; fall back to the instance ID for display. Route actions using `enabled`, not `status`. For enabled dashboards, open `metrixbar://panel/<instanceID>` and offer Quicklink creation. For disabled dashboards, open the configuration page at `metrixbar://open/<instanceID>` and do not offer Quicklink creation.
- Use instance IDs from the local API, encoded as a single path segment with `encodeURIComponent`. Do not substitute remote catalog IDs. Metrix.Bar handles disabled-dashboard fallback and cold-start recovery for saved Quicklinks.
- The local API accepts only loopback access. All endpoints except `GET /v0/health` require Bearer authentication, and requests with an `Origin` header are rejected. Explicitly set `agent: false` on `node:http` requests to prevent Node 24's environment proxy support from forwarding the token. Do not follow redirects.
- If the integration needs a token, use a Raycast `password` preference. Do not read the app's private configuration to find credentials or include tokens in logs or response messages.
- Keep `package-lock.json` and do not introduce lockfiles for other package managers. Exclude `node_modules/`, `dist/`, `raycast-env.d.ts`, and test and tool caches through `.gitignore`. Do not commit generated artifacts.
- Allowed npm install scripts are pinned by version in `package.json` under `allowScripts`. When updating esbuild, review its install script and update the permission, then validate `npm ci` and the production build.

## Official References

- Raycast: [Manifest](https://developers.raycast.com/information/manifest), [CLI](https://developers.raycast.com/information/developer-tools/cli), [ESLint](https://developers.raycast.com/information/developer-tools/eslint), [Store Submission Requirements](https://developers.raycast.com/basics/prepare-an-extension-for-store).
- Vite+: [Environment Management](https://viteplus.dev/guide/env), [Task Execution and Built-in Commands](https://viteplus.dev/guide/run).
- Metrix.Bar: [Documentation Index](https://metrix.bar/llms.txt), [Local API](https://metrix.bar/docs/local-api.md), [Registries and External Links](https://metrix.bar/docs/registries.md), [OpenAPI](https://metrix.bar/openapi.json).
