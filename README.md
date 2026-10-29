# static-php

Builds fully static PHP binaries (`php` + `php-fpm`, linux x86_64, musl) with
[StaticPHP v3](https://github.com/crazywhalecc/static-php-cli).

This repository contains only the build workflow. StaticPHP itself is checked out
from upstream at build time, so there is nothing to sync when upstream updates.

## Build

Actions → **Build PHP bulk (linux amd64, cli + fpm)** → Run workflow

| Input | Default | Notes |
|---|---|---|
| `php-version` | `8.5.12` | Must be an exact `x.y.z` version |
| `spc-ref` | `v3` | Upstream branch, tag or commit SHA. Pin a SHA for reproducible builds |
| `extensions` | bulk set | Same as `dl.static-php.dev/v3/php-bin/bulk`, minus `swoole` (see below) |
| `debug` | off | Full build logs |

PHP source is downloaded only from php.net (`--dl-no-alt`), and the job fails if
the built version does not match `php-version`.

Download the `php-<version>-cli-fpm-linux-x86_64` artifact from the run. It contains
`php`, `php-fpm`, `SHA256SUMS` and `BUILD-INFO.txt` (StaticPHP commit, PHP version,
module list).

## Extensions left out of the bulk set

- `swoole` (and `swoole-hook-*`): swoole v6.2.3 failed to compile against
  PHP 8.5.12 (`no member named 'ap_php_snprintf' in namespace 'std'`, from
  `php-src/main/snprintf.h` redefining `snprintf` inside the bundled nlohmann/json).

Add it back to `extensions` if you need it.

## When to rebuild

- New PHP release: run with the new `php-version`.
- Library security fixes (openssl, curl, libxml2, ...) arrive through StaticPHP
  updates: run again with `spc-ref: v3` to pick up the latest upstream.
