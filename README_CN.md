# MT Photos Unraid 模板

[English](README.md) | 简体中文

本仓库提供 [MT Photos](https://mtmt.tech/) 的 Unraid Docker 模板，用于在 NAS 上管理私有照片和视频。

## 仓库内容

- `templates/mtphotos-nodb.xml` — 可选择内置或外部 PostgreSQL 的统一 Unraid Docker v2 模板
- `ca_profile.xml` — Community Applications 维护者资料
- `icons/mtphotos.png` — 仓库及应用图标
- `README.md` — 英文说明（默认）
- `README_CN.md` — 简体中文说明
- `LICENSE` — 模板仓库许可证

## 模板选择

- 默认 `latest` 分支内置 PostgreSQL。数据库文件与应用配置、缩略图、预览和缓存一同保存在 `/config`，必须持久化并备份该目录。
- `nodb-latest` 分支适合连接单独维护的 PostgreSQL 服务。选择后需在高级设置填写五个 `POSTGRES_*` 变量，数据库需要独立持久化和备份。

## NoDB 分支配置

选择 `nodb-latest` 后，模板使用不含数据库的官方镜像，需另行配置数据库服务。请在高级设置填写已有 PostgreSQL 服务的连接信息。外部数据库需单独持久化和备份，仅备份 `/config` 不能替代数据库备份。

| 配置 | 容器目标与宿主机默认值 |
| --- | --- |
| 网页端口 | `8063/tcp` |
| Tailscale 网页端口 | `8063` |
| 应用配置及缓存 | `/config` ← `/mnt/user/appdata/mtphotos` |
| 手机上传目录 | `/upload` ← `/mnt/user/photos/MTPhotos-Upload` |
| 已有图库 | `/photos` ← `/mnt/user/photos` |
| 时区 | `TZ=Asia/Shanghai` |
| NoDB 数据库地址 | `POSTGRES_HOST` — 选择 NoDB 后填写 |
| NoDB 数据库端口 | `POSTGRES_PORT` — 通常为 `5432` |
| NoDB 数据库名称 | `POSTGRES_DATABASE` — 通常为 `postgres` |
| NoDB 数据库用户 | `POSTGRES_USER` |
| NoDB 数据库密码 | `POSTGRES_PASSWORD` — 输入时隐藏 |
| 临时目录（可选，转码用途待验证） | `/temp` ← `/mnt/user/appdata/mtphotos/temp` |

按照[官方环境变量说明](https://mtmt.tech/docs/advanced/env/)，将五个 `POSTGRES_*` 配置项设置为实际数据库的连接信息。地址和端口必须能从 MT Photos 容器访问；bridge 模式下，`localhost` 和 `127.0.0.1` 指向 MT Photos 容器自身。密码配置不会修改数据库用户的密码。

## 硬件要求与数据库网络

根据[官方安装说明](https://mtmt.tech/docs/start/install/)，建议使用 x86_64 系统、至少 4 GB 内存及双核 2.0 GHz CPU。

人脸识别和以文搜图需要 PostgreSQL 支持 pgvector。[官方数据库说明](https://mtmt.tech/docs/advanced/db/)提供了 `mtphotos/mt-photos-pg:latest` 镜像；普通 PostgreSQL 安装本身不具备这些向量功能。

- 使用默认 `bridge` 网络时，填写 NAS 局域网 IP 和数据库发布的宿主机端口。例如数据库映射为 `5433:5432`，则填写 `POSTGRES_PORT=5433`。
- 如需使用数据库容器名连接，须将两个容器加入同一个自定义 Docker 网络，并填写数据库容器内端口（通常为 `5432`）。默认 bridge 不会自动解析容器名。
- 数据库位于其他服务器时，填写其可访问地址和端口。连接外部数据库不要使用 `localhost` 或 `127.0.0.1`。

参考 [Docker bridge 网络说明](https://docs.docker.com/engine/network/drivers/bridge/)。

## 临时目录：转码用途待验证

可选的 `temp` 映射位于高级设置，为容器提供可写的 `/temp` 目录。它本身不会设置 MT Photos 的转码目录。目前未从官方文档或运行中的镜像确认应用会自动使用此路径，因此模板没有添加未经验证的转码重定向环境变量，也不再提供未经验证的应用设置操作。

使用该映射前，应执行一次转码，通过日志或进程参数及实际新增文件确认临时输出路径。如果应用使用其他路径，应将映射目标改为已确认的路径。永久缓存中生成了转码成品，并不能证明临时文件使用了 `/temp`。

## 发布前准备

仓库：[baofeidyz/unraid-template-mtphotos](https://github.com/baofeidyz/unraid-template-mtphotos)。维护者：baofeidyz。模板支持：[GitHub Issues](https://github.com/baofeidyz/unraid-template-mtphotos/issues)。

将文件发布到 `main` 分支后，检查以下原始文件地址：


```text
https://raw.githubusercontent.com/baofeidyz/unraid-template-mtphotos/main/templates/mtphotos-nodb.xml
https://raw.githubusercontent.com/baofeidyz/unraid-template-mtphotos/main/icons/mtphotos.png
https://raw.githubusercontent.com/baofeidyz/unraid-template-mtphotos/main/README.md
```

应用图标为 `icons/mtphotos.png`。如需替换，请使用有权发布的图片；路径发生变化时，同步更新两个 XML 文件。

## 验证与测试

在仓库根目录验证 XML 文件：

```bash
xmllint --noout ca_profile.xml templates/mtphotos-nodb.xml
```

提交到 Community Applications 前：

1. 推送到公开 GitHub 仓库的 `main` 分支。
2. 检查 `Icon`、`TemplateURL`、`ReadMe`、`Support`、`Project`、`WebPage` 和 `Forum` 引用，确保可访问且没有占位符。
3. 在 Unraid 分别选择两个镜像分支测试。`latest` 需验证数据库随 `/config` 持久化及备份恢复；`nodb-latest` 需填写高级数据库变量并验证连接。两个分支均需验证启动、网页访问、`/upload` 手机备份和 `/photos` 图库访问。
4. 验证实际临时转码路径；如使用可选 `temp` 映射，确认临时文件写入对应宿主机目录。核对宿主机路径和时区；如禁止修改原文件，将图库映射改为只读，并单独验证数据库备份。
5. 通过 [Unraid Community Applications 提交入口](https://ca.unraid.net/submit)提交公开仓库。

## 补充说明

- WebUI 地址使用容器端口占位符 `[PORT:8063]`，由 Unraid 解析为映射的宿主机端口。
- `/config` 保存应用配置、缩略图、预览和缓存，`latest` 分支还在此保存内置数据库；`/upload` 保存手机备份的照片和视频。两者都需要持久化和可写权限。
- 可在 Unraid 中增加路径映射以导入更多图库。
- 本仓库仅提供 Community Applications 元数据；MT Photos 及镜像遵循上游各自的使用条款。

## 上游参考

- [MT Photos](https://mtmt.tech/)
- [官方安装说明](https://mtmt.tech/docs/start/install/)
- [官方 Docker 镜像](https://hub.docker.com/r/mtphotos/mt-photos)
- [Unraid Community Apps 官方起始模板](https://github.com/unraid/unraid-community-apps-starter)
