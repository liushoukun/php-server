# php-server

基于 [serversideup/php](https://github.com/serversideup/docker-php) 构建的 PHP 服务基础镜像，预装 Laravel 常用扩展，用于统一各项目的运行环境。

## 镜像与标签

| 镜像地址 | 基础镜像 | 说明 |
| --- | --- | --- |
| `liushoukun/php-server:8.4-fpm-nginx` | `serversideup/php:8.4-fpm-nginx` | PHP 8.4 + PHP-FPM + Nginx，Docker Hub |
| `ghcr.io/liushoukun/php-server:8.4-fpm-nginx` | `serversideup/php:8.4-fpm-nginx` | PHP 8.4 + PHP-FPM + Nginx |

标签说明：

- `8.4-fpm-nginx`：跟随 PHP 8.4 的稳定标签，推荐使用
- `latest`：最新一次构建
- `sha-<短哈希>`：按提交固定版本，便于精确追溯

## 快速使用

```dockerfile
FROM liushoukun/php-server:8.4-fpm-nginx

# 复制应用代码，并交给运行用户 www-data
COPY --chown=www-data:www-data . /var/www/html
```

FPM/Nginx 的运行方式、端口与健康检查等配置，请参考 [serversideup/php 官方文档](https://serversideup.net/open-source/docker-php/)。

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
| `PHP_OPCACHE_ENABLE` | `true` | 启用 OPcache |

## 目录结构

```text
.
├── .github/
│   └── workflows/
│       └── build.yml      # 构建并发布镜像到 GHCR 和 Docker Hub
└── fpm-nginx/
    └── Dockerfile         # 镜像定义
```

## 新增扩展

1. 编辑 `fpm-nginx/Dockerfile`，在 `install-php-extensions` 列表中加上扩展名
2. 提交并推送到 `master`
3. GitHub Actions 自动构建并推送新镜像

## 发布说明

- 当 `fpm-nginx/**` 或 `.github/workflows/build.yml` 发生变更并推送到 `master` 时，自动构建并推送镜像到 GHCR 和 Docker Hub
- 镜像平台：`linux/amd64`
- 生成标签：`8.4-fpm-nginx`、`latest`、`sha-<短提交哈希>`
- 首次发布后，需要到 GitHub 仓库的 **Packages** 页面将包可见性设置为 **Public**，否则其他项目无法匿名拉取

### Docker Hub 配置

在 GitHub 仓库的 **Settings > Secrets and variables > Actions** 中添加以下 Repository secrets：

| Secret | 值 |
| --- | --- |
| `DOCKERHUB_USERNAME` | Docker Hub 用户名，例如 `liushoukun` |
| `DOCKERHUB_TOKEN` | Docker Hub 的 Personal Access Token，需要 `Read & Write` 权限 |

同时需要在 Docker Hub 创建名为 `php-server` 的镜像仓库。如果 Docker Hub 用户名与 GitHub 用户名不同，请同步修改 README 中的镜像地址；流水线会根据 `DOCKERHUB_USERNAME` 自动使用正确的命名空间。
