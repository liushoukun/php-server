# CLI 镜像

`cli` 变体用于 Laravel 队列 Worker 和定时任务，基于 PHP CLI 构建，不包含 Nginx、FPM 或任何 Web 服务。

镜像地址：

- `liushoukun/php-server:8.4-cli`
- `ghcr.io/liushoukun/php-server:8.4-cli`

镜像默认执行队列 Worker：

```bash
php artisan queue:work --sleep=3 --tries=3 --timeout=90
```

## Docker Compose 示例

队列 Worker 可以横向扩容；定时任务只保留一个实例，避免同一任务被重复调度。
Worker 的停止宽限期设置为 2 分钟，大于单任务 `--timeout=90`，便于容器重启时完成当前任务。

```yaml
services:
  queue-worker:
    image: ghcr.io/liushoukun/php-server:8.4-cli
    working_dir: /var/www/html
    restart: unless-stopped
    init: true
    stop_grace_period: 2m
    command:
      - php
      - artisan
      - queue:work
      - --sleep=3
      - --tries=3
      - --timeout=90

  scheduler:
    image: ghcr.io/liushoukun/php-server:8.4-cli
    working_dir: /var/www/html
    restart: unless-stopped
    init: true
    stop_grace_period: 2m
    command:
      - php
      - artisan
      - schedule:work
```

应用代码应挂载到 `/var/www/html`，或在下游 Dockerfile 中复制到该目录并保持文件属主为 `www-data`。该镜像不暴露端口。
