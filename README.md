# next-magento-backend

Magento **Open Source 2.4.9** backend for the **next-bns** headless commerce platform, run locally via [markshust/docker-magento](https://github.com/markshust/docker-magento). The decoupled Next.js storefront lives in a separate repo and consumes this backend's GraphQL API.

## What this repo tracks

The docker-magento **wrapper config only** — `bin/`, `compose*.yaml`, `env/`, `Makefile`, and `deploy/`. The Magento application (`src/`) and generated/build artifacts are gitignored (pulled via Composer during setup).

## Local setup

Prerequisites: Docker (≥ 6 GB allocated), `curl unzip rsync libnss3-tools`, `mkcert`.

```bash
bin/download community 2.4.9
bin/setup 249.magento.default
bin/magento sampledata:deploy && bin/magento setup:upgrade && bin/magento indexer:reindex
```

## Store currency (scope-driven)

The headless storefront reads the display currency from the Magento **store scope** (`storeConfig`), not from any frontend config — so set the dev store view to the currency you want served. Stock Luma sample data ships **USD-only**; set the scope to **EUR**:

```bash
bin/magento config:set currency/options/base EUR
bin/magento config:set currency/options/default EUR
bin/magento config:set currency/options/allow EUR
bin/magento indexer:reindex && bin/magento cache:flush
```

This config lives in the database (default scope), so it must be re-applied after a fresh `bin/setup` — it is not part of the tracked wrapper config.

## Ports & reverse proxy

To coexist with a host already using `:80`/`:443`/`:3306`, the stack is remapped in `compose.override.yaml`: **app `:8443`, db `:3307`, valkey `:6380`, opensearch `:9201`**. A system-nginx reverse proxy (`deploy/nginx/249.magento.default.conf`) fronts `https://249.magento.default` → `https://127.0.0.1:8443` (mkcert TLS).

## GraphQL

Endpoint: `https://249.magento.default/graphql` — consumed headlessly by the Next.js storefront.

## Branching model (Git Flow)

`main` (production) ← `develop` (integration) ← `feature/*`. Feature branches open PRs into `develop`; after a PR merges, `main` is fast-forwarded to `develop` so the two stay in sync (`main == develop`). Never commit directly to `main`.
