# MT Photos Unraid 模板

[English](README.md) | 简体中文

本仓库提供 [MT Photos](https://mtmt.tech/) 的 Unraid Docker 模板，用于在 NAS 上管理私有照片和视频。

## 仓库内容

- `templates/mtphotos.xml` — 带内置 PostgreSQL 的 Unraid Docker v2 模板
- `templates/mtphotos-nodb.xml` — 连接外部 PostgreSQL 的 NoDB 模板
- `templates/mtphotos-ai.xml` — MT Photos 智能识别 API（ONNX）模板
- `templates/mtphotos-insightface-api.xml` — MT Photos InsightFace 人脸识别 API 模板
- `ca_profile.xml` — Community Applications 维护者资料
- `icons/mtphotos.png` — 仓库及应用图标
- `README.md` — 英文说明（默认）
- `README_CN.md` — 简体中文说明
- `LICENSE` — 模板仓库许可证

## 镜像与配置

- `mtphotos` 使用 `mtphotos/mt-photos:latest`，内置 PostgreSQL；数据库随 `/config` 持久化。
- `mtphotos-nodb` 使用 `mtphotos/mt-photos:nodb-latest`，需填写 `POSTGRES_HOST`、`POSTGRES_PORT`、`POSTGRES_DATABASE`、`POSTGRES_USER` 和 `POSTGRES_PASSWORD`，并单独备份外部数据库。

两个主应用默认使用相同的网页端口、上传目录和图库目录，不应以默认值同时运行。如需并行安装，请为其中一个修改宿主机端口及 `/config`、`/upload` 路径。

## 可选识别服务

| 应用 | 镜像 | 端口 | 配置 |
| --- | --- | --- | --- |
| `mtphotos-ai` | `mtphotos/mt-photos-ai:onnx-latest` | `8060/tcp` | `API_AUTH_KEY` |
| `mtphotos-insightface-api` | `devfox101/mt-photos-insightface-unofficial:latest` | `8066/tcp` | `API_AUTH_KEY` |

两个 API 容器都不需要目录映射。安装后在 MT Photos 后台分别添加对应的 API 地址，例如 `http://NAS局域网IP:8060` 和 `http://NAS局域网IP:8066`，并填写各模板中相同的 `API_AUTH_KEY`。InsightFace 使用社区镜像，并非 MT Photos 官方镜像。

| 配置 | 容器目标与宿主机默认值 |
| --- | --- |
| 网页端口 | `8063/tcp` |
| Tailscale 网页端口 | `8063` |
| 应用配置及缓存 | `/config` ← `/mnt/user/appdata/mtphotos` |
| 手机上传目录 | `/upload` ← `/mnt/user/photos/MTPhotos-Upload` |
| 已有图库 | `/photos` ← `/mnt/user/photos` |
| 时区 | `TZ=Asia/Shanghai` |
| 临时目录（可选，转码用途待验证） | `/temp` ← `/mnt/user/appdata/mtphotos/temp` |

## 硬件要求

根据[官方安装说明](https://mtmt.tech/docs/start/install/)，建议使用 x86_64 系统、至少 4 GB 内存及双核 2.0 GHz CPU。

内置数据库版通过 `/config` 持久化数据库；NoDB 版需要另行提供 PostgreSQL，人脸识别和以文搜图需要数据库支持 pgvector。

## 临时目录：转码用途待验证

可选的 `temp` 映射位于高级设置，为容器提供可写的 `/temp` 目录。它本身不会设置 MT Photos 的转码目录。目前未从官方文档或运行中的镜像确认应用会自动使用此路径，因此模板没有添加未经验证的转码重定向环境变量，也不再提供未经验证的应用设置操作。

使用该映射前，应执行一次转码，通过日志或进程参数及实际新增文件确认临时输出路径。如果应用使用其他路径，应将映射目标改为已确认的路径。永久缓存中生成了转码成品，并不能证明临时文件使用了 `/temp`。

## 发布前准备

仓库：[baofeidyz/unraid-template-mtphotos](https://github.com/baofeidyz/unraid-template-mtphotos)。维护者：baofeidyz。模板支持：[GitHub Issues](https://github.com/baofeidyz/unraid-template-mtphotos/issues)。

将文件发布到 `main` 分支后，检查以下原始文件地址：


```text
https://raw.githubusercontent.com/baofeidyz/unraid-template-mtphotos/main/templates/mtphotos-nodb.xml
https://raw.githubusercontent.com/baofeidyz/unraid-template-mtphotos/main/templates/mtphotos.xml
https://raw.githubusercontent.com/baofeidyz/unraid-template-mtphotos/main/templates/mtphotos-ai.xml
https://raw.githubusercontent.com/baofeidyz/unraid-template-mtphotos/main/templates/mtphotos-insightface-api.xml
https://raw.githubusercontent.com/baofeidyz/unraid-template-mtphotos/main/icons/mtphotos.png
https://raw.githubusercontent.com/baofeidyz/unraid-template-mtphotos/main/README.md
```

应用图标为 `icons/mtphotos.png`。如需替换，请使用有权发布的图片，并同步更新模板与 `ca_profile.xml` 中的路径。

## 验证与测试

在仓库根目录验证 XML 文件：

```bash
xmllint --noout ca_profile.xml templates/*.xml
```

提交到 Community Applications 前：

1. 推送到公开 GitHub 仓库的 `main` 分支。
2. 检查 `Icon`、`TemplateURL`、`ReadMe`、`Support`、`Project`、`WebPage` 和 `Forum` 引用，确保可访问且没有占位符。
3. 在 Unraid 手动安装模板，验证主应用启动、网页访问、数据库随 `/config` 持久化及备份恢复、`/upload` 手机备份和 `/photos` 图库访问。
4. 分别启动两个可选 API，使用对应密钥检查接口，并在 MT Photos 后台验证智能识别与人脸识别任务。
5. 验证实际临时转码路径；如使用可选 `temp` 映射，确认临时文件写入对应宿主机目录。核对宿主机路径和时区；如禁止修改原文件，将图库映射改为只读，并单独验证数据库备份。
6. 通过 [Unraid Community Applications 提交入口](https://ca.unraid.net/submit)提交公开仓库。

## 补充说明

- WebUI 地址使用容器端口占位符 `[PORT:8063]`，由 Unraid 解析为映射的宿主机端口。
- `/config` 保存内置数据库、应用配置、缩略图、预览和缓存；`/upload` 保存手机备份的照片和视频。两者都需要持久化和可写权限。
- 可在 Unraid 中增加路径映射以导入更多图库。
- 本仓库仅提供 Community Applications 元数据；MT Photos 及镜像遵循上游各自的使用条款。

## 上游参考

- [MT Photos](https://mtmt.tech/)
- [官方安装说明](https://mtmt.tech/docs/start/install/)
- [官方 Docker 镜像](https://hub.docker.com/r/mtphotos/mt-photos)
- [Unraid Community Apps 官方起始模板](https://github.com/unraid/unraid-community-apps-starter)
