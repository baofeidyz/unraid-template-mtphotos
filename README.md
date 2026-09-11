# MT Photos Unraid Templates

English | [简体中文](README_CN.md)

This repository contains an Unraid Docker template for [MT Photos](https://mtmt.tech/), a private photo and video management system for NAS users.

## Repository contents

- `templates/mtphotos.xml` — Unraid Docker template (v2) with bundled PostgreSQL
- `templates/mtphotos-nodb.xml` — NoDB template for external PostgreSQL
- `templates/mtphotos-ai.xml` — MT Photos AI recognition API (ONNX) template
- `templates/mtphotos-insightface-api.xml` — MT Photos InsightFace facial recognition API template
- `ca_profile.xml` — Community Applications maintainer profile
- `icons/mtphotos.png` — Repository/application icon
- `README.md` — English documentation (default)
- `README_CN.md` — Simplified Chinese documentation
- `LICENSE` — Template repository license

## Image and configuration

- `mtphotos` uses `mtphotos/mt-photos:latest` with bundled PostgreSQL; its database is persisted under `/config`.
- `mtphotos-nodb` uses `mtphotos/mt-photos:nodb-latest`. Configure `POSTGRES_HOST`, `POSTGRES_PORT`, `POSTGRES_DATABASE`, `POSTGRES_USER` and `POSTGRES_PASSWORD`, and back up the external database separately.

Both main apps default to the same WebUI port, upload path and library path. Do not run both with unchanged defaults. For a parallel installation, change one app's host port and its `/config` and `/upload` host paths.

## Optional recognition services

| App | Image | Port | Configuration |
| --- | --- | --- | --- |
| `mtphotos-ai` | `mtphotos/mt-photos-ai:onnx-latest` | `8060/tcp` | `API_AUTH_KEY` |
| `mtphotos-insightface-api` | `devfox101/mt-photos-insightface-unofficial:latest` | `8066/tcp` | `API_AUTH_KEY` |

Neither API container requires a storage mapping. After installation, add their API addresses in MT Photos, for example `http://NAS-LAN-IP:8060` and `http://NAS-LAN-IP:8066`, using the matching `API_AUTH_KEY` from each template. InsightFace uses a community image and is not an official MT Photos image.

| Setting | Container target and host default |
| --- | --- |
| WebUI | `8063/tcp` |
| Tailscale WebUI port | `8063` |
| Settings and cache | `/config` ← `/mnt/user/appdata/mtphotos` |
| Mobile uploads | `/upload` ← `/mnt/user/photos/MTPhotos-Upload` |
| Existing library | `/photos` ← `/mnt/user/photos` |
| Timezone | `TZ=Asia/Shanghai` |
| Temporary storage (optional; transcoding use unverified) | `/temp` ← `/mnt/user/appdata/mtphotos/temp` |

## Requirements

Recommended hardware: x86_64, at least 4 GB RAM and a dual-core 2.0 GHz CPU, following the [official installation guide](https://mtmt.tech/docs/start/install/).

The bundled-database template persists PostgreSQL through `/config`. NoDB requires a separate PostgreSQL service; face recognition and text-to-image search require pgvector support.

## Temporary storage: verification pending

The optional `temp` mapping is shown in advanced settings and exposes writable storage at `/temp`. It does not by itself select MT Photos' transcoding directory. Automatic use of this path has not been established from official documentation or a running image. There is no verified application setting or environment variable supplied here to redirect transcoding.

Before relying on this mapping, run a transcoding task and confirm the actual temporary output path from logs or process arguments and newly created files. If the application uses a different path, map that confirmed path instead. Do not treat completed transcodes in the permanent cache as proof that temporary files used `/temp`.

## Required before publishing

Repository: [baofeidyz/unraid-template-mtphotos](https://github.com/baofeidyz/unraid-template-mtphotos). Maintainer: baofeidyz. Template support: [GitHub Issues](https://github.com/baofeidyz/unraid-template-mtphotos/issues).

Publish the files to the `main` branch and verify these raw URLs:



```text
https://raw.githubusercontent.com/baofeidyz/unraid-template-mtphotos/main/templates/mtphotos-nodb.xml
https://raw.githubusercontent.com/baofeidyz/unraid-template-mtphotos/main/templates/mtphotos.xml
https://raw.githubusercontent.com/baofeidyz/unraid-template-mtphotos/main/templates/mtphotos-ai.xml
https://raw.githubusercontent.com/baofeidyz/unraid-template-mtphotos/main/templates/mtphotos-insightface-api.xml
https://raw.githubusercontent.com/baofeidyz/unraid-template-mtphotos/main/icons/mtphotos.png
https://raw.githubusercontent.com/baofeidyz/unraid-template-mtphotos/main/README.md
```

The main application icon is `icons/mtphotos.png`. If you replace it, use an image you are authorized to publish and update both main app templates and `ca_profile.xml` if the path changes.

## Validate and test

Validate the XML files from the repository root:

```bash
xmllint --noout ca_profile.xml templates/*.xml
```

Before submitting to Community Applications:

1. Push to the `main` branch of a public GitHub repository.
2. Check the URLs referenced by `Icon`, `TemplateURL`, `ReadMe`, `Support`, `Project`, `WebPage`, and `Forum`; ensure they are accessible and contain no placeholders.
3. Install manually on Unraid and verify the main app's startup, WebUI access, database persistence under `/config`, backup restoration, mobile backup to `/upload`, and library access under `/photos`.
4. Start each optional API, check it with the corresponding key, and verify AI and facial recognition jobs from MT Photos.
5. Verify the actual temporary transcoding path and, if using the optional `temp` mapping, confirm temporary files are written to its host directory. Check host paths and timezone; use a read-only library mapping to prevent changes to originals, and verify separate database backups.
6. Submit the public repository through the [Unraid Community Applications submission portal](https://ca.unraid.net/submit).

## Notes

- The WebUI URL uses `[PORT:8063]`, which Unraid resolves to the mapped host port.
- `/config` stores the bundled database, settings, thumbnails, previews and cache; `/upload` stores mobile photo and video backups. Keep both mappings persistent and writable.
- Add further path mappings in Unraid for additional libraries.
- This repository packages only Community Applications metadata. MT Photos and its Docker image remain subject to their respective upstream terms.

## Upstream references

- [MT Photos](https://mtmt.tech/)
- [Official installation guide](https://mtmt.tech/docs/start/install/)
- [Official Docker image](https://hub.docker.com/r/mtphotos/mt-photos)
- [Official Unraid Community Apps starter](https://github.com/unraid/unraid-community-apps-starter)
