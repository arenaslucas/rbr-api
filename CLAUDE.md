# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`rbrapi` is an unofficial Python client library for the Rocket Bot Royale game API (built on Nakama backend at `https://dev-nakama.winterpixel.io/v2`). Version `0.7`, requires Python `>=3.8`, single dependency: `requests`.

## Commands

**Lint/format check (CI uses this):**
```bash
ruff format --check rbrapi
```

**Auto-format:**
```bash
ruff format rbrapi
```

**Build for distribution:**
```bash
python -m pip install build
python -m build
```

There is no test suite configured.

## Architecture

The library exposes a single `RocketBotRoyale` class (`rbrapi/__init__.py`) that authenticates on instantiation and wraps all API calls.

**Module responsibilities:**
- `rbrapi/__init__.py` — Main client class. Defines `BASE_URL`, `BASE_HEADERS` (with hardcoded game credentials), and `CLIENT_VERSION = "9999999999"` (intentional, bypasses version checks). All public methods map to Nakama REST or RPC endpoints.
- `rbrapi/session.py` — Thread-local `requests.Session` cache with a 600-second TTL. `make_request()` handles POST/GET, JSON/form-data payloads, and raises custom exceptions on non-2xx responses.
- `rbrapi/types.py` — Response types. `APIResponse` base class provides JSON serialization. Typed response objects (`AuthenticateResponse`, `AccountResponse`, `LootBoxResponses`, etc.) and `TypedDict` definitions for nested structures (`UserStats`, `UserMetadata`, `Wallet`, etc.).
- `rbrapi/errors.py` — Six custom exceptions, one per API operation that can fail (`AuthenticationError`, `SignUpError`, `CollectTimedBonusError`, `FriendRequestError`, `LootBoxError`, `UnknownUserError`).

**Request flow:**
```
RocketBotRoyale(email, password)
  └─> authenticate() ──> make_request() ──> thread-local requests.Session (TTL 600s)

client.buy_crate() / account() / etc.
  └─> make_request() ──> Response parsed into types.py objects
```

**RPC calls** (e.g., `collect_timed_bonus`, `send_friend_request`, `buy_crate`) POST to `/rpc/<rpc_name>` as `application/x-www-form-urlencoded` with a JSON-encoded body string as the form value.

## CI/CD

- **`ruff.yml`**: Runs `ruff format --check rbrapi` on every push/PR.
- **`pypi.yml`**: Publishes to PyPI automatically when `rbrapi/__init__.py` changes on `main`.
