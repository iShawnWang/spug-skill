---
name: spug-skill
description: Use this skill when you need to log into a Spug deployment platform with username and password, list authorized apps and deploy configs, create release requests for a target app and environment, trigger publish, and poll release status from a CLI written in TypeScript and executed with npx tsx.
---

# Spug CLI Skill

This skill provides a zero-runtime-dependency TypeScript CLI for Spug.

## Runtime

- Node `20+`
- Execute with `npx tsx src/cli.ts ...`
- Runtime dependencies are limited to Node built-ins; `tsx`, `typescript`, and `@types/node` are dev-only.

## Commands

```bash
npx tsx src/cli.ts list-envs --base-url https://demo.spug.cc --username admin --password 'spug.cc'
npx tsx src/cli.ts list-apps --base-url https://demo.spug.cc --username admin --password 'spug.cc'
npx tsx src/cli.ts list-deploys --base-url https://demo.spug.cc --username admin --password 'spug.cc'
npx tsx src/cli.ts list-versions --base-url https://demo.spug.cc --username admin --password 'spug.cc' --app api_order --env pro
npx tsx src/cli.ts create-request --base-url https://demo.spug.cc --username admin --password 'spug.cc' --app api_order --env pro --name 'release test' --branch 1.x --commit 67c137da9ea1bc451bfa1c34ebfb35c4821825c2
npx tsx src/cli.ts deploy-and-watch --base-url https://demo.spug.cc --username admin --password 'spug.cc' --app api_order --env pro --name 'release test' --branch 1.x --commit 67c137da9ea1bc451bfa1c34ebfb35c4821825c2 --json
```

## Resolution Rules

- `--app` accepts app `id`, `key`, or exact `name`.
- `--env` accepts environment `id`, `key`, or exact `name`.
- `--deploy-id` can be used instead of `--app` plus `--env`.
- If `--host-ids` is omitted, the CLI uses the default `host_ids` from the deploy config.

## API Flow

1. `POST /api/account/login/`
2. `GET /api/config/environment/`
3. `GET /api/app/`
4. `GET /api/app/deploy/`
5. `GET /api/app/deploy/<deployId>/versions/`
6. `POST /api/deploy/request/ext1/`
7. `GET /api/deploy/request/`
8. `POST /api/deploy/request/<requestId>/`
9. `GET /api/deploy/request/info/?id=<requestId>`

## Notes

- Spug's create-request endpoint returns an empty `data` payload, so the CLI resolves the new `request_id` by reading `/api/deploy/request/` and matching the newest request by `deploy_id` and `name`.
- The current implementation treats statuses `1` and `2` as in-progress and `3` as success. Any other final status returns a non-zero exit code in `deploy-and-watch`.
- If your Spug instance uses different status codes or custom approval flow, update the status mapping in [src/cli.ts](/Users/shawn/spug-skill/src/cli.ts).
