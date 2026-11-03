# CurrentMusic

![CurrentMusic](assets/brand/currentmusic-icon.png)

CurrentMusic 的完整开源仓库：Python 后端、Web 播放器、Android WebView 应用、Windows 原生客户端、官网与 API 文档。项目自有代码采用 [MIT](LICENSE) 许可。

- [Web 播放器](https://music.20110208.xyz/app/) · [API 文档](https://music.20110208.xyz/api-docs/)
- [Android Releases](https://github.com/backrooms-yrc/CurrentMusic/releases) · [更新日志](CHANGELOG.md)
- [后端部署](backend/README.md) · [Windows 客户端](windows/README.md) · [贡献指南](CONTRIBUTING.md)

公开库保留原前端壳的提交、标签与 Releases；后端和其他组件以经过审查的源码快照加入。历史私有仓库继续保密，不合并其 Git 历史。运行数据、密钥、账号凭据和生产环境文件不属于源码。

## 目录

| 目录 | 内容 |
| --- | --- |
| `backend/` | 账号、通行密钥、网易云网关、播放/歌单、聊天/房间、通行证与 AI |
| `web/` | Web 与 Android 共用的前端源码 |
| `app/` | Java WebView 壳及可重建的前端静态资源 |
| `windows/` | C# / WinUI 3 原生客户端及安装器源码 |
| `deploy/` | systemd、Nginx、PHP 入口、环境变量与备份模板 |
| `site/`、`api-docs/` | 官网与 API 文档站源码 |
| `tools/`、`docs/` | 构建、验证、维护脚本与接口说明 |

## 本地启动后端

需要 Python 3.11+。先在仓库根目录执行：

```bash
python3 -m venv .venv
. .venv/bin/activate
python3 -m pip install -r deploy/requirements-auth.txt
mkdir -p data
cp deploy/backend.env.example data/backend.env
# 编辑 data/backend.env，设置 CM_DATA_DIR/CM_DB 等路径为当前实例的绝对路径。
# 在初始化和启动之前加载同一份配置。
set -a; . data/backend.env; set +a
python3 backend/manage.py init-db
python3 backend/manage.py create-owner admin --nickname Administrator
python3 backend/cm_server.py
```

后端默认监听 `127.0.0.1:3010`；`GET /health` 可检查存活。创建站长时未提供密码会生成随机密码并仅显示一次。生产部署、外部服务和反向代理配置见 [backend deployment guide](backend/README.md)。

网易云 API 是独立部署的上游服务，通过 `CM_NCM_BASE` 指定；它的源码和登录 cookie 不包含在本仓库。AI、SMTP、SMS、分享图与 LDDC 均按需配置；没有相应服务时对应功能会明确降级或不可用。

## Web 与 Android

需要 Node.js 20+；Android 构建另需 JDK 17、Android SDK platform 34 和 build-tools 34.0.0。

```bash
npm --prefix web ci
python3 tools/check-js-refs.py
npm --prefix web run build
# 只部署 Web：先同步资源，再切换带内容指纹的 index.html。
python3 tools/deploy-web.py --target data/app

# Android：版本默认读取 web/src/version.js，也可显式设置 VER_NAME/VER_CODE。
export ANDROID_HOME=/path/to/android-sdk
bash build.sh
```

前端产物位于 `app/src/main/assets/www/`；APK 位于 `dist/`。首次 Android 构建会创建本地自签名密钥；自行签名的安装包无法覆盖官方签名版本。签名密码通过环境变量配置，勿提交密钥或 APK。

## 官网与文档

```bash
# 官网复用 web/node_modules，输出到 data/www。
node site/build.mjs
# 文档可独立从当前路由和登记表构建。
npm --prefix api-docs ci
npm --prefix api-docs run build
npm --prefix api-docs run publish
```

如需完整上游参数摘录，构建文档时设置 `NCM_API_DIR` 指向独立的 api-enhanced 源码目录。缺失上游源码时仍生成自有接口、权限登记与专题，并明确标注未收录部分。

## Audio quality

Playback requests the quality tier that the provider makes available; the EQ changes playback and does not create a mastered file. FLAC conversion or upsampling cannot restore quality missing from a lossy source. See [Audio quality and mastering boundaries](docs/audio-quality-and-mastering.md) for the current implementation status and requirements for a future, user-controlled local-file workflow.

## Verification and security

```bash
tools/run-tests.sh                  # Forces a temporary database and data directory
python3 tools/test-api-docs.py
python3 tools/test-open-source.py
python3 tools/check-open-source.py   # Scans tracked files; reports locations, never secret values
```

See the [contribution guide](CONTRIBUTING.md), [security policy](SECURITY.md), and [open-source checklist](docs/open-source-checklist.md). Media and account write operations are subject to provider terms and applicable laws.

## License and third-party resources

Project-owned code is licensed under [MIT](LICENSE). Dependencies, icons, fonts, upstream documentation, and optional services retain their own licenses; see [third-party notices](THIRD_PARTY_NOTICES.md). LDDC is installed and run separately; see its license and source notices before distributing it.

CurrentMusicX is independently maintained by community developers at [bileizhen/CurrentMusicX](https://github.com/bileizhen/CurrentMusicX); its repository and release process remain separate.
