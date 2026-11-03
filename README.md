# CurrentMusic

![CurrentMusic](assets/brand/currentmusic-icon.png)

CurrentMusic is a complete open-source music platform project: Python backend, Web player, Android WebView app, Windows native client, official website, and API documentation. The project’s own code is released under the [MIT](LICENSE) license.

- [Web Player](https://music.20110208.xyz/app/) · [API Docs](https://music.20110208.xyz/api-docs/)
- [Android Releases](https://github.com/backrooms-yrc/CurrentMusic/releases) · [Changelog](CHANGELOG.md)
- [Backend Deployment](backend/README.md) · [Windows Client](windows/README.md) · [Contributing Guide](CONTRIBUTING.md)

The public repository keeps the original frontend shell’s commits, tags, and releases. The backend and other components are included as reviewed source snapshots. The private historical repository remains confidential and its Git history is not merged. Runtime data, server state, and account information are not included in the public source.

## Table of Contents

- [Overview](#overview)
- [Project Structure](#project-structure)
- [Running the Backend Locally](#running-the-backend-locally)
- [Web and Android](#web-and-android)
- [Website and Documentation](#website-and-documentation)
- [Testing and Security](#testing-and-security)
- [License and Third-Party Resources](#license-and-third-party-resources)

## Overview

CurrentMusic is a self-hosted music ecosystem that combines:

- account and session management
- NetEase Cloud Music proxying and API compatibility
- playlist and listening history syncing
- room/chat features and community interactions
- share image generation and metadata support
- OAuth/passkey-based authentication flows
- AI-assisted music interaction and recommendations
- Android and Windows client support alongside the web experience

The project is designed to be extensible. In addition to the default NetEase-based experience, it is structured to support additional music services and account integrations via provider adapters, OAuth, and upstream gateway configuration. This includes the ability to integrate services such as YouTube Music, Spotify, QQ Music, and other supported upstream providers when configured for a specific deployment.

## Project Structure

| Directory | Contents |
| --- | --- |
| `backend/` | Accounts, passkeys, NetEase gateway, playback, playlists, chat/rooms, passkeys, and AI |
| `web/` | Frontend source shared by Web and Android |
| `app/` | Java WebView shell and rebuildable frontend static resources |
| `windows/` | C# / WinUI 3 native client and installer source |
| `deploy/` | systemd, Nginx, PHP entrypoints, environment variables, and backup templates |
| `site/`, `api-docs/` | Website and API documentation source |
| `tools/`, `docs/` | Build, validation, maintenance scripts, and API notes |

## Running the Backend Locally

Requires Python 3.11+. Run the following from the repository root:

```bash
python3 -m venv .venv
. .venv/bin/activate
python3 -m pip install -r deploy/requirements-auth.txt
mkdir -p data
cp deploy/backend.env.example data/backend.env
# Edit data/backend.env and set CM_DATA_DIR / CM_DB and other paths to absolute paths for this instance.
# Load the same configuration before initialization and startup.
set -a; . data/backend.env; set +a
python3 backend/manage.py init-db
python3 backend/manage.py create-owner admin --nickname Administrator
python3 backend/cm_server.py
```

The backend listens on `127.0.0.1:3010` by default. `GET /health` can be used to check health. If no password is provided when creating the site owner, a random password is generated and displayed only once. Production deployment, external services, and reverse proxy configuration are documented in [deploy/README.md](deploy/README.md).

The NetEase API is a separately deployed upstream service, configured via `CM_NCM_BASE`. Its source code and login cookies are not included in this repository. AI, SMTP, SMS, share images, and LDDC are all optional and configured on demand. If not configured, the relevant features fall back or return explicit errors.

## Web and Android

Requires Node.js 20+. Android builds also require JDK 17, Android SDK platform 34, and build-tools 34.0.0.

```bash
npm --prefix web ci
python3 tools/check-js-refs.py
npm --prefix web run build
# To deploy only the Web app: sync static assets first, then switch the index.html with content hashes.
python3 tools/deploy-web.py --target data/app

# Android: version defaults to web/src/version.js, or set VER_NAME / VER_CODE explicitly.
export ANDROID_HOME=/path/to/android-sdk
bash build.sh
```

Frontend output is stored under `app/src/main/assets/www/`; APK outputs are located in `dist/`. The first Android build creates a local self-signed key. Self-signed APKs cannot overwrite officially signed versions. Signing passwords and device-specific build settings must be managed locally.

## Website and Documentation

```bash
# The website reuses the Web node_modules and outputs to data/www.
node site/build.mjs
# The documentation can be built independently from the current routing and registry.
npm --prefix api-docs ci
npm --prefix api-docs run build
npm --prefix api-docs run publish
```

If you need full upstream parameter excerpts, set `NCM_API_DIR` to a separate `api-enhanced` source directory when building the docs. If the upstream source is missing, the project still generates its own APIs, permission registries, and topics and clearly marks the gaps.

## Testing and Security

```bash
tools/run-tests.sh                  # Always use a temporary database and data directory
python3 tools/test-api-docs.py
python3 tools/test-open-source.py
python3 tools/check-open-source.py   # Checks tracked files; reports only locations and does not print secrets
```

The contribution flow is described in [CONTRIBUTING.md](CONTRIBUTING.md). Security issues should be reported via [SECURITY.md](SECURITY.md). Public boundary and migration details are documented in the [open-source checklist](docs/open-source-checklist.md). Media and account write operations are subject to upstream and self-hosted policy enforcement.

## License and Third-Party Resources

The project’s own code is licensed under the [MIT](LICENSE). Dependencies, icon fonts, upstream documentation, and optional services keep their own licenses as described in [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md). LDDC is installed separately and is not bundled into this repository.

CurrentMusicX is maintained independently by community contributors in [bileizhen/CurrentMusicX](https://github.com/bileizhen/CurrentMusicX). Its repository and release process remain independent.

## Multi-Account and Third-Party Integration Notes

CurrentMusic is designed to work as a self-hosted music platform with a clean multi-source architecture. The current codebase favors provider-based integration and can be extended to support additional music ecosystems and login flows. In practice, this may include:

- NetEase Cloud Music
- YouTube Music
- Spotify
- QQ Music
- other upstream providers through API gateway adapters and OAuth-enabled account linking

This makes the project suitable for custom deployments that want a single self-hosted interface while connecting to multiple music services or account ecosystems. Configuration and compatibility depend on the specific deployment and upstream provider support.

