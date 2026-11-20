# php-server

基于 [serversideup/php](https://github.com/serversideup/docker-php) 构建的 PHP 基础镜像，预装 Laravel 常用扩展，用于统一 Web、队列 Worker 和定时任务容器的运行环境。

## 镜像与标签

| 镜像地址 | 基础镜像 | 说明 |
| --- | --- | --- |
| `liushoukun/php-server:8.4-fpm-nginx` | `serversideup/php:8.4-fpm-nginx` | PHP 8.4 + PHP-FPM + Nginx，Docker Hub |
| `ghcr.io/liushoukun/php-server:8.4-fpm-nginx` | `serversideup/php:8.4-fpm-nginx` | PHP 8.4 + PHP-FPM + Nginx |
| `liushoukun/php-server:8.4-cli` | `serversideup/php:8.4-cli` | PHP 8.4 CLI，不包含 Nginx/FPM，Docker Hub |
| `ghcr.io/liushoukun/php-server:8.4-cli` | `serversideup/php:8.4-cli` | PHP 8.4 CLI，用于队列 Worker 和定时任务 |

标签说明：

- `8.4-fpm-nginx`：跟随 PHP 8.4 的稳定标签，用于 Web 服务
- `8.4-cli`：无 Nginx 的队列 Worker、定时任务镜像
- `latest`：Web 镜像的最新一次构建
- `sha-<短哈希>`：Web 镜像的固定版本
- `sha-<短哈希>-cli`：CLI 镜像的固定版本

## 快速使用

```dockerfile
FROM liushoukun/php-server:8.4-fpm-nginx

# 复制应用代码，并交给运行用户 www-data
COPY --chown=www-data:www-data . /var/www/html
```

FPM/Nginx 的运行方式、端口与健康检查等配置，请参考 [serversideup/php 官方文档](https://serversideup.net/open-source/docker-php/)。

队列 Worker 和定时任务使用无 Nginx 的 `cli` 镜像。镜像默认运行队列 Worker；定时任务容器将命令覆盖为 `php artisan schedule:work`。完整示例见 [`cli/README.md`](cli/README.md)。

## 已安装扩展

- bcmath
- gd
- intl
- mbstring
- pcntl
- redis
- igbinary
- msgpack
- exif
- opcache

默认配置：

| 环境变量 | 值 | 说明 |
| --- | --- | --- |
| `AUTORUN_LARAVEL_STORAGE_LINK` | `true` | 容器启动时自动创建 storage 软链接 |
| `PHP_OPCACHE_ENABLE` | `true` | 在 `fpm-nginx` 镜像中启用 OPcache |
| `PHP_OPCACHE_ENABLE_CLI` | `1` | 在 `cli` 镜像中启用 CLI OPcache |

## 目录结构

```text
.
├── .github/
│   └── workflows/
│       └── build.yml      # 构建并发布两个镜像变体到 GHCR 和 Docker Hub
├── fpm-nginx/
│   └── Dockerfile         # PHP-FPM + Nginx 镜像定义
└── cli/
    ├── Dockerfile         # PHP CLI 镜像定义
    └── README.md          # 队列与定时任务部署示例
```

## 新增扩展

1. 同时编辑 `fpm-nginx/Dockerfile` 和 `cli/Dockerfile`，在两个 `install-php-extensions` 列表中加上扩展名
2. 提交并推送到 `master`
3. GitHub Actions 自动构建并推送新镜像

## 发布说明

- 当 `fpm-nginx/**`、`cli/**` 或 `.github/workflows/build.yml` 发生变更并推送到 `master` 时，自动构建并推送两个镜像变体到 GHCR 和 Docker Hub
- 镜像平台：`linux/amd64`
- Web 镜像标签：`8.4-fpm-nginx`、`latest`、`sha-<短提交哈希>`
- CLI 镜像标签：`8.4-cli`、`sha-<短提交哈希>-cli`
- 首次发布后，需要到 GitHub 仓库的 **Packages** 页面将包可见性设置为 **Public**，否则其他项目无法匿名拉取

### Docker Hub 配置

在 GitHub 仓库的 **Settings > Secrets and variables > Actions** 中添加以下 Repository secrets：

| Secret | 值 |
| --- | --- |
| `DOCKERHUB_USERNAME` | Docker Hub 用户名，例如 `liushoukun` |
| `DOCKERHUB_TOKEN` | Docker Hub 的 Personal Access Token，需要 `Read & Write` 权限 |

同时需要在 Docker Hub 创建名为 `php-server` 的镜像仓库。如果 Docker Hub 用户名与 GitHub 用户名不同，请同步修改 README 中的镜像地址；流水线会根据 `DOCKERHUB_USERNAME` 自动使用正确的命名空间。
