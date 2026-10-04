# OpenFlux transport template

Fork this repository to write your own [OpenFlux](https://github.com/p1neappleXpress/OpenFlux)
transport as a JS script. Tag a release and CI signs it, publishes it, and
every app that installed your transport gets the update.

*[Читать на русском](README.ru.md)*

## Layout

| path | what |
|---|---|
| `src/main.js` | your transport (starts as the full annotated contract) |
| `manifest.json` | id, name, version, author, `wire`, `api` |
| `sdk/` | [OpenFluxSDK](https://github.com/p1neappleXpress/OpenFluxSDK) as a submodule: docs and the host-API reference (`sdk/docs`) |
| `.github/workflows/check.yml` | every push/PR: packs with a throwaway key, so a name/version mismatch or a script that does not load fails here |
| `.github/workflows/release.yml` | on a `v*` tag: signs, publishes the release, updates `update.json` |

```bash
git clone --recurse-submodules <your fork>     # sdk/ is big: add --depth 1 --shallow-submodules
```

## One-time setup

1. Generate your author key. **Never commit it.**
   ```bash
   go run github.com/p1neappleXpress/OpenFlux/transport/script/cmd/scriptsign@main genkey author.priv author.pub
   ```
2. Add the contents of `author.priv` as the repository secret **`SIGNING_KEY`**
   (Settings → Secrets and variables → Actions).
3. Publish the fingerprint of `author.pub` (SHA-256, shown by the app on first
   install) somewhere users can compare it - this README is a good place. The
   app pins this key the first time a user trusts your transport; updates
   signed by any other key are refused.
4. Edit `manifest.json` (`id`, `name`, `author`, `description`) and
   `info().name` in `src/main.js` to match.

## Releasing

```bash
# bump "version" in manifest.json AND info().version in src/main.js, commit, then:
git tag v0.2.0 && git push --tags
```

- `v1.4.0` → channel **stable**; `v1.5.0-beta.1` → channel **nightly** (a GitHub pre-release).
- CI refuses a tag that differs from `manifest.json`'s version.
- The signed `<id>-<version>.flux` is a release asset; `update.json` (what apps poll) is
  committed to the `updates` branch, one entry per channel.
- CI writes `update` into the signed manifest for you
  (`https://raw.githubusercontent.com/<you>/<repo>/updates/update.json`). Set your own
  `"update": [url, mirror-url…]` in `manifest.json` to use another host or add mirrors.

## Versioning rules

- `version` is semantic (`MAJOR.MINOR.PATCH`). Apps only ever move forward.
- **`wire`** is the generation of your wire format. A client and a node both run
  your transport: keep `wire` the same while old and new versions can talk to each other
  (updates then install on their own for official transports, with one tap for others).
  Bump `wire` when they cannot - apps then hold the update back until the user
  confirms, because the node has to be updated too.
- **`api`** is the host-API generation you wrote against; leave it as the template has it.
- Changing the signing key is not an update: users have to import and trust the new key again.
