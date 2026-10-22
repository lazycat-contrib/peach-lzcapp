# Peach for LazyCat

将 [Peach](https://github.com/peach-mitao/peach)（蜜桃）打包为 LazyCat LPK v2。Peach 是单用户、本地优先的个人媒体系统：把本地硬盘、通过 CloudDrive2 挂载的网盘和关注的创作者更新集中放在一个地方浏览与播放。

> 上游标注 18+，仅供成年人管理自己的成人内容馆藏。仓库不含媒体或站点数据，连接外部站点需自行拥有相应账号和访问权。

## 首次使用

1. 首次打开时应用会生成配置、数据库和随机登录口令。日志只显示口令文件位置，可在应用数据目录中查看，或在容器内执行 `peach token`。
2. 用该口令在浏览器登录后，在设置页的「媒体目录」中添加 `/media` 或其子目录（例如 `/media/115`），选择媒体来源与读取策略，保存后开始扫描。
3. 需要局域网内其他设备访问时，在设置中开启并设置访问密码。

## 数据与运行方式

| 路径 | 用途 |
| --- | --- |
| `/lzcapp/var/peach` → `/data` | 配置、SQLite 数据库、凭据、图片缓存 |
| `/lzcapp/documents` → `/media`（只读） | 影片库，扫描与播放不需要写权限 |

- 上游镜像：`longmeidao/peach:0.47.3`（Docker Hub）
- 运行镜像：`docker.1ms.run/longmeidao/peach:0.47.3`（Docker Hub 默认加速）
- 容器内 HTTP 端口固定 `8900`，以 UID/GID 1000 非 root 运行
- `gpu_accel: true` 挂载 `/dev/dri`，AMD/Intel 可用 VAAPI 硬件转码；无可用渲染节点时回退软件转码
- 沿用上游健康检查 `/healthz?ready=1`

媒体目录也可以指向 CloudDrive2 挂载出来的网盘路径；那种情况需要在微服上另行运行 CloudDrive2，并把挂载目录共享给本应用容器。

## 本地构建

```bash
lzc-cli project build -o dist/application.lpk
lzc-cli lpk info dist/application.lpk
```

## 自动发布

`.github/workflows/lazycat.yml` 使用 `ca-x/lazycat-github-action` 跟踪上游镜像的不可变版本 tag，创建版本化 GitHub Release Asset 并只发布到喵喵私有商店。需要为仓库配置 `APPSTORE_URL`、`APPSTORE_TOKEN`，以及可选的 `APP_ID`、`PRIVATE_STORE_GROUP_CODES`。