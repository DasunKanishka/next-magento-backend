# next-magento-backend

Magento **Open Source 2.4.9** backend for the **Next B&S** headless commerce platform (first brand: **TopDrinks**), run locally via [markshust/docker-magento](https://github.com/markshust/docker-magento). The decoupled Next.js storefront lives in a separate repo and consumes this backend's GraphQL API.

## What this repo tracks

The docker-magento **wrapper config only** — `bin/`, `compose*.yaml`, `env/`, `Makefile`, and `deploy/`. The Magento application (`src/`) and generated/build artifacts are gitignored (pulled via Composer during setup).

## Local setup

Prerequisites: Docker (≥ 6 GB allocated), `curl unzip rsync libnss3-tools`, `mkcert`.

```bash
bin/download community 2.4.9
bin/setup 249.magento.default
bin/magento sampledata:deploy && bin/magento setup:upgrade && bin/magento indexer:reindex
```

## Ports & reverse proxy

To coexist with a host already using `:80`/`:443`/`:3306`, the stack is remapped in `compose.override.yaml`: **app `:8443`, db `:3307`, valkey `:6380`, opensearch `:9201`**. A system-nginx reverse proxy (`deploy/nginx/249.magento.default.conf`) fronts `https://249.magento.default` → `https://127.0.0.1:8443` (mkcert TLS).

## GraphQL

Endpoint: `https://249.magento.default/graphql` — consumed headlessly by the Next.js storefront.

## Branching model (Git Flow)

`main` (production) ← `develop` (integration) ← `feature/*`. Feature branches PR into `develop`; `develop` promotes to `main` at releases.
