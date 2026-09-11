# MT Photos Unraid Templates

English | [简体中文](README_CN.md)

This repository contains an Unraid Docker template for [MT Photos](https://mtmt.tech/), a private photo and video management system for NAS users.

## Repository contents

- `templates/mtphotos.xml` — Unraid Docker template (v2) with bundled PostgreSQL
- `templates/mtphotos-nodb.xml` — Unraid Docker template (v2) for external PostgreSQL
- `ca_profile.xml` — Community Applications maintainer profile
- `icons/mtphotos.png` — Repository/application icon
- `README.md` — English documentation (default)
- `README_CN.md` — Simplified Chinese documentation
- `LICENSE` — Template repository license

## Choose a template

- `mtphotos` uses the official `mtphotos/mt-photos:latest` image with bundled PostgreSQL. Database files, application settings, thumbnails, previews and cache are stored under `/config`; persist and back up this directory.
- `mtphotos-nodb` uses `mtphotos/mt-photos:nodb-latest` and connects to a separately managed PostgreSQL service. Persist and back up that database independently.

## NoDB image and configuration

The template uses the official `mtphotos/mt-photos:nodb-latest` image, which does not bundle a database, as documented in the [official installation guide](https://mtmt.tech/docs/start/install/). Configure a separate database service. This template does not create a database container. Enter the connection settings below for your existing PostgreSQL service. Persist and back up the external database separately; backing up `/config` alone does not replace a database backup.

| Setting | Container target and host default |
| --- | --- |
| WebUI | `8063/tcp` |
| Tailscale WebUI port | `8063` |
| Settings and cache | `/config` ← `/mnt/user/appdata/mtphotos` |
| Mobile uploads | `/upload` ← `/mnt/user/photos/MTPhotos-Upload` |
| Existing library | `/photos` ← `/mnt/user/photos` |
| Timezone | `TZ=Asia/Shanghai` |
| PostgreSQL host | `POSTGRES_HOST` — required, no default |
| PostgreSQL port | `POSTGRES_PORT=5432` |
| PostgreSQL database | `POSTGRES_DATABASE=postgres` |
| PostgreSQL user | `POSTGRES_USER=postgres` |
| PostgreSQL password | `POSTGRES_PASSWORD` — required, no default; masked input |
| Temporary storage (optional; transcoding use unverified) | `/temp` ← `/mnt/user/appdata/mtphotos/temp` |

Configure the five `POSTGRES_*` variables to match your existing database, following the [official environment variable reference](https://mtmt.tech/docs/advanced/env/). Use an address and port reachable from the MT Photos container; in bridge mode, `localhost` and `127.0.0.1` refer to the MT Photos container itself. The password setting does not change the database user's password.

## Requirements and database networking

Recommended hardware: x86_64, at least 4 GB RAM and a dual-core 2.0 GHz CPU, following the [official installation guide](https://mtmt.tech/docs/start/install/).

Face recognition and text-to-image search require PostgreSQL with pgvector support. The [official database guide](https://mtmt.tech/docs/advanced/db/) provides `mtphotos/mt-photos-pg:latest`; a plain PostgreSQL installation does not provide these vector features by itself.

- With the default `bridge` network, use the NAS LAN IP and PostgreSQL's published host port. For example, if PostgreSQL publishes `5433:5432`, set `POSTGRES_PORT=5433`.
- To connect using a database container name, attach both containers to the same user-defined Docker network and use the database container port (normally `5432`). Default bridge does not automatically resolve container names.
- For a database on another server, use its reachable address and port. Do not use `localhost` or `127.0.0.1` for an external database.

See [Docker bridge networking](https://docs.docker.com/engine/network/drivers/bridge/).

## Temporary storage: verification pending

The optional `temp` mapping is shown in advanced settings and exposes writable storage at `/temp`. It does not by itself select MT Photos' transcoding directory. Automatic use of this path has not been established from official documentation or a running image. There is no verified application setting or environment variable supplied here to redirect transcoding.

Before relying on this mapping, run a transcoding task and confirm the actual temporary output path from logs or process arguments and newly created files. If the application uses a different path, map that confirmed path instead. Do not treat completed transcodes in the permanent cache as proof that temporary files used `/temp`.

## Required before publishing

Repository: [baofeidyz/unraid-template-mtphotos](https://github.com/baofeidyz/unraid-template-mtphotos). Maintainer: baofeidyz. Template support: [GitHub Issues](https://github.com/baofeidyz/unraid-template-mtphotos/issues).

Publish the files to the `main` branch and verify these raw URLs:



```text
https://raw.githubusercontent.com/baofeidyz/unraid-template-mtphotos/main/templates/mtphotos-nodb.xml
https://raw.githubusercontent.com/baofeidyz/unraid-template-mtphotos/main/templates/mtphotos.xml
https://raw.githubusercontent.com/baofeidyz/unraid-template-mtphotos/main/icons/mtphotos.png
https://raw.githubusercontent.com/baofeidyz/unraid-template-mtphotos/main/README.md
```

The application icon is `icons/mtphotos.png`. If you replace it, use an image you are authorized to publish and update both XML files if the path changes.

## Validate and test

Validate both XML files from the repository root:

```bash
xmllint --noout ca_profile.xml templates/mtphotos.xml templates/mtphotos-nodb.xml
```

Before submitting to Community Applications:

1. Push to the `main` branch of a public GitHub repository.
2. Check the URLs referenced by `Icon`, `TemplateURL`, `ReadMe`, `Support`, `Project`, `WebPage`, and `Forum`; ensure they are accessible and contain no placeholders.
3. Install the required template manually on Unraid. For the bundled-database template, verify database persistence under `/config` and backup restoration. For NoDB, configure and test the external database connection. For both, verify startup, WebUI access, mobile backup to `/upload`, and library access under `/photos`.
4. Verify the actual temporary transcoding path and, if using the optional `temp` mapping, confirm temporary files are written to its host directory. Check host paths and timezone; use a read-only library mapping to prevent changes to originals, and verify separate database backups.
5. Submit the public repository through the [Unraid Community Applications submission portal](https://ca.unraid.net/submit).

## Notes

- The WebUI URL uses `[PORT:8063]`, which Unraid resolves to the mapped host port.
- `/config` stores settings, thumbnails, previews and cache; `/upload` stores mobile photo and video backups. Keep both mappings persistent and writable.
- Add further path mappings in Unraid for additional libraries.
- This repository packages only Community Applications metadata. MT Photos and its Docker image remain subject to their respective upstream terms.

## Upstream references

- [MT Photos](https://mtmt.tech/)
- [Official installation guide](https://mtmt.tech/docs/start/install/)
- [Official Docker image](https://hub.docker.com/r/mtphotos/mt-photos)
- [Official Unraid Community Apps starter](https://github.com/unraid/unraid-community-apps-starter)
