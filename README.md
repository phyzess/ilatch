# ilatch

This repository is the production Cloudflare deployment instance for Latch.

It is not the Latch runtime source and it is not the upstream deployment
template. Cloudflare builds this repository as the `ilatch` Worker, while the
application runtime is installed from the public npm package `@phyzess/latch`.

## Repository Roles

- `phyzess/latch-core` is the runtime source. UI, Worker behavior, CLI behavior,
  config validation, and tests belong there.
- `phyzess/latch` is the reusable deployment template used by the Cloudflare
  import flow.
- `phyzess/ilatch` is this production deployment instance. It keeps instance
  configuration such as the Worker name, KV namespace binding, admin emails, and
  the pinned `@phyzess/latch` runtime version.

## Runtime Updates

This instance receives Latch application updates by updating the
`@phyzess/latch` dependency in `package.json` and `pnpm-lock.yaml`.

The `.github/workflows/update-latch-runtime.yml` workflow checks npm for the
latest `@phyzess/latch` version, updates the dependency, runs `pnpm build` and
`pnpm doctor`, then commits the version bump back to this repository. Cloudflare
Workers Builds redeploys after that commit reaches the connected production
branch.

After publishing a new runtime from `phyzess/latch-core`, either run the
`Update Latch Runtime` workflow manually in this repository or wait for its
scheduled run.

## Local Checks

```sh
mise install
pnpm install
pnpm dev
pnpm build
pnpm doctor
pnpm deploy:dry
```

`pnpm dev` builds the packaged Latch assets and starts Wrangler on an available
local port. Localhost skips Cloudflare Access and treats `dev@localhost` as an
admin, so `/settings` can be used immediately.

## Instance Configuration

Production links live in the `LATCH_CONFIG` Workers KV namespace. They are
edited from `/settings` and are not committed to this repository.

Production deployments must keep Cloudflare Access in front of the whole Latch
domain. `POLICY_AUD` and `TEAM_DOMAIN` are configured as Worker secrets in
Cloudflare. `LATCH_ADMIN_EMAILS` lists the Access emails allowed to edit links
from `/settings`.

Do not make application runtime changes in this repository. Make those changes
in `phyzess/latch-core`, publish `@phyzess/latch`, then update this instance to
the published runtime version.
