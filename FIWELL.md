# Fiwell fork of actual-mcp-server

This is Fiwell's fork of [agigante80/actual-mcp-server](https://github.com/agigante80/actual-mcp-server),
the MCP server that lets Claude and other assistants talk to an Actual Budget server.
Fiwell v2 is itself a fork of Actual Budget ([danielkuhnen/actual](https://github.com/danielkuhnen/actual)).

## Branches

- `main` mirrors upstream. Never commit to it; update it with GitHub's **Sync fork** button.
- `fiwell` is what we deploy. It is pinned to a reviewed upstream release tag plus the
  Fiwell-only files below. Changes reach it through pull requests.

Keep Fiwell changes in new files where possible (a new tool is one new file in
`src/tools/`), so updating from upstream stays a clean merge. Fixes that are not
Fiwell-specific should go upstream as a pull request instead.

## Fiwell-only files

| File | Purpose |
|------|---------|
| `FIWELL.md` | This page |
| `fly.staging.toml` | Fly.io app `fiwell-v2-mcp-staging` (region cdg), read-only, pointing at `https://fiwell-v2-staging.fly.dev`, sign-in through Auth0 |
| `.github/workflows/fiwell-staging-deploy.yml` | Deploys to Fly on every push to `fiwell` |

## Sign-in through Auth0

Clients such as claude.ai sign in with OAuth (`AUTH_PROVIDER=oidc`) against Fiwell's Auth0
tenant. Auth0 needs:

- An API whose identifier is exactly `https://fiwell-v2-mcp-staging.fly.dev/http`
  (`OIDC_RESOURCE`), so access tokens are JWTs with that audience.
- The tenant's Resource Parameter Compatibility Profile turned on, so the `resource`
  parameter MCP clients send is treated as the audience.
- An application for Claude (Regular Web Application) with the callback
  `https://claude.ai/api/mcp/auth_callback`. Its client ID and secret go into the claude.ai
  connector's advanced settings, not into Fly.

`AUTH_BUDGET_ACL` must list the Auth0 user ids allowed in. Everyone else who can sign in to
the tenant gets no budget. Leaving it unset or `{}` would let every Auth0 user read every
budget, because the server reaches Actual with the sync-server password.

## Updating to a new upstream release

1. Sync `main` with upstream.
2. Read the upstream changes between the pinned tag and the new one (release notes and diff),
   paying attention to auth, HTTP and anything that sends data off the server.
3. Check that the new release's `@actual-app/api` version is at least the version the
   Fiwell sync-server runs. An older API cannot be trusted to open a budget a newer client
   has migrated.
4. Open a pull request merging the new tag into `fiwell`, and bump `VERSION` in
   `fly.staging.toml`. Merging it deploys.

## Upstream workflows

The upstream workflows (`ci-cd.yml`, `dependency-update.yml`, `api-surface-drift.yml`, and
the rest) publish npm packages and Docker images and open release pull requests. They are not
meant to run in this fork: disable them in the repository's Actions tab and keep only
**Fiwell MCP staging deploy** enabled.
