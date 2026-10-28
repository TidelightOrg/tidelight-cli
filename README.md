# Tidelight CLI

The official command-line client for supported Tidelight API v1 workflows. It provides secure terminal access to market instruments, company research jobs, and research webhooks.

[Product](https://tidelight.app) · [Developer API guide](https://dev.tidelight.app/api) · [npm package](https://www.npmjs.com/package/@tidelight/cli)

## Repository and source of truth

This repository is a public mirror of the CLI package in the [canonical Tidelight monorepo](https://github.com/TidelightOrg/tidelight/tree/main/packages/cli). The monorepo is the sole source of truth for implementation code. Make code changes there; this mirror is refreshed from the monorepo's `main` branch each day and can also be synchronized manually from its Actions page. The mirror workflow preserves this README and its own workflow file.

## Install

Node.js 22 or later is required.

```sh
npm install --global @tidelight/cli@0.1.2
tidelight --help
```

Create and manage project API keys in the [Tidelight Developer Center](https://tidelight.app/developers). Use a key with only the scopes needed for the commands you run.

## Sign in

```sh
tidelight auth login
tidelight auth status
```

The CLI prompts for the key without echoing it and stores it in the operating system credential store: macOS Keychain, Windows Credential Manager, or Linux Secret Service. Linux requires an unlocked Secret Service provider such as GNOME Keyring or KWallet.

If a credential store is unavailable, set `TIDELIGHT_API_KEY` for the current process or shell. The CLI does not write a plaintext credential file. Environment credentials take precedence over the credential store. `auth status` reports the credential source without revealing the key; `auth logout` removes the stored key but does not change shell environment variables.

## Supported workflows

```sh
# Browse supported tokenized-equity spot instruments
tidelight market instruments --ticker NVDA --limit 20
tidelight market instruments --json

# Start and follow an asynchronous company research job
tidelight research start NVDA
tidelight research start NVDA --wait --timeout-ms 120000
tidelight research get 00000000-0000-4000-8000-000000000000 --json
tidelight research cancel 00000000-0000-4000-8000-000000000000

# Manage research-job webhooks
tidelight webhooks list
tidelight webhooks create --name "Research alerts" --url https://example.com/tidelight --events research.job.completed
tidelight webhooks update 00000000-0000-4000-8000-000000000000 --rotate-secret
tidelight webhooks disable 00000000-0000-4000-8000-000000000000
tidelight webhooks deliveries --json
```

These commands follow the supported SDK and OpenAPI contract. They do not create research reports, strategies, scenarios, live orders, accounts, projects, or keys. Service scopes, ownership, quotas, and provider terms remain authoritative.

Research calls use a generated UUID idempotency key by default. Pass `--idempotency-key` to reuse a stable key across retries. `--wait` polls until a terminal state or bounded timeout; Ctrl-C stops local polling, while `research cancel` explicitly requests server cancellation.

Market responses include server-provided provenance and instrument type. If a provider feed is unavailable, the CLI reports the service response and does not substitute another feed. Tokenized-equity spot products are not direct stock shares or perpetual-contract prices.

## Webhook secret handling

A signing secret is shown only in a successful create or rotation response. The CLI warns on stderr and does not save or retry mutations. Copy the secret to a secrets manager immediately, and avoid sending secret-bearing output to logs.

## Development and validation

Run development commands from the canonical monorepo so the CLI resolves its matching local SDK dependency:

```sh
npm ci --prefix sdk/typescript
npm test --prefix sdk/typescript
npm ci --prefix packages/cli
npm test --prefix packages/cli
```

The monorepo also provides `npm run test:package-consumers`, which rehearses installation of the packaged CLI with its matching SDK in an isolated consumer project.

## Compatibility and security

The supported runtime is Node.js 22 or later, with the matching `@tidelight/sdk` release and the versioned Tidelight API v1. Transient retry behavior is provided by the shared SDK. Errors may include the HTTP status, stable API error code, and request ID; credentials and raw provider response bodies are excluded.

See [CHANGELOG.md](./CHANGELOG.md) for release history. This project is licensed under Apache-2.0; see [LICENSE](./LICENSE).
