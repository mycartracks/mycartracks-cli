# MyCarTracks CLI

This repository contains a Restish project configuration for using the MyCarTracks API v3 as a CLI. Restish reads the OpenAPI document, generates commands dynamically, and honors the API's `x-cli-name`, `x-cli-aliases`, `x-cli-description`, `x-cli-hidden`, and `x-cli-ignore` extensions.

## API Links

- API docs: https://mycartracks.com/api-docs/v3/index.html
- OpenAPI JSON: https://mycartracks.com/services/rest/openapi.json
- API base URL: https://mycartracks.com/services/rest/v3

## Install Restish

```sh
brew install rest-sh/tap/restish
```

Or install with Go:

```sh
go install github.com/rest-sh/restish/v2/cmd/restish@latest
```

## Quick Start

```sh
restish config trust
restish --rsh-profile public api sync mycartracks

export MYCARTRACKS_CLIENT_ID="..."
export MYCARTRACKS_CLIENT_SECRET="..."

restish mycartracks list-vehicles --limit 10
```

## Use The Project Config

The committed `.restish.json` points requests at the live MyCarTracks API:

```sh
https://mycartracks.com/services/rest/v3
```

Trust the project config once:

```sh
restish config trust
```

Sync the live OpenAPI spec into Restish's local command cache. The `public` profile is only used for fetching the public spec without API credentials:

```sh
restish --rsh-profile public api sync mycartracks
```

Then run API commands:

```sh
restish mycartracks --help
restish mycartracks list-vehicles
restish mycartracks tracks --limit 10
```

If you do not want to trust project config globally, pass it explicitly:

```sh
restish --rsh-config ./.restish.json --rsh-profile public api sync mycartracks
restish --rsh-config ./.restish.json mycartracks --help
```

## Authentication

Get or manage API credentials in MyCarTracks:

- Community guide: https://community.mycartracks.com/t/where-to-get-your-api-credentials/59
- API settings: https://mycartracks.com/portal/settings?#api

The default profile uses read-only OAuth2 client credentials from environment variables:

```sh
export MYCARTRACKS_CLIENT_ID="..."
export MYCARTRACKS_CLIENT_SECRET="..."

restish mycartracks list-vehicles
```

Restish fetches and caches the bearer token using:

```text
https://mycartracks.com/oauth/token
```

For write-capable credentials, change both `scopes` values in `.restish.json` from `read` to `read write`.

Clear cached OAuth tokens after testing:

```sh
restish api auth logout mycartracks
```

## Sync OpenAPI Changes

When the API publishes new operations or updated `x-cli-*` metadata:

```sh
restish --rsh-profile public api sync mycartracks
```

Check how Restish sees the generated command surface:

```sh
restish doctor api mycartracks
```

## Commands

Restish generates commands from the current live OpenAPI document, so the command list updates when the API spec changes.

List all available commands:

```sh
restish mycartracks --help
```

Inspect a command's flags, aliases, request schema, and response schema:

```sh
restish mycartracks list-vehicles --help
restish mycartracks tracks --help
```

Inspect generated command metadata, including `x-cli-*` effects:

```sh
restish doctor api mycartracks
```

Command names, aliases, descriptions, hidden operations, and ignored operations are controlled by `x-cli-name`, `x-cli-aliases`, `x-cli-description`, `x-cli-hidden`, and `x-cli-ignore` in the OpenAPI spec.

## Output And Queries

Return JSON or YAML:

```sh
restish mycartracks list-vehicles --limit 10 -o json
restish mycartracks list-vehicles --limit 10 -o yaml
```

Filter output with a query:

```sh
restish mycartracks list-vehicles --limit 10 -q 'data[].{id:id,name:name}'
```

Use command help to see supported parameters:

```sh
restish mycartracks list-vehicles --help
```

## Troubleshooting

If commands are missing or stale, sync the live spec again:

```sh
restish --rsh-profile public api sync mycartracks
```

If OAuth tokens are stale or credentials were changed, clear the auth cache:

```sh
restish api auth logout mycartracks
```

If token fetching fails with `invalid_scope`, keep the default `read` scope for read-only credentials. Only use `read write` when the client credentials are allowed to request write access.

If project config is not trusted, either run `restish config trust` in this repository or pass the config explicitly:

```sh
restish --rsh-config ./.restish.json --rsh-profile public api sync mycartracks
restish --rsh-config ./.restish.json mycartracks --help
```

## Initial Setup Command

This is the command used to create `.restish.json`:

```sh
restish --rsh-config .restish.json api connect mycartracks \
  https://mycartracks.com/services/rest/v3 \
  --spec https://mycartracks.com/services/rest/openapi.json \
  --yes \
  'prompt.credentials.OAuth2.client_id: env:MYCARTRACKS_CLIENT_ID' \
  'prompt.credentials.OAuth2.client_secret: env:MYCARTRACKS_CLIENT_SECRET'
```

The committed config also includes a `public` profile for unauthenticated OpenAPI sync.
