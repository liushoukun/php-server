############################################
# CLI Image - 队列与定时任务镜像
# 基于 PHP CLI，不包含 Nginx/FPM 等 Web 服务
############################################

FROM serversideup/php:8.4-cli

# 切换到 root 用户安装 PHP 扩展
USER root

# 与 fpm-nginx 镜像保持一致
RUN install-php-extensions \
    bcmath \
    gd \
    intl \
    mbstring \
    pcntl \
    igbinary \
    msgpack \
    exif \
    opcache

# 基础镜像已预装 redis，但未启用 igbinary/msgpack 序列化器。
# 删除旧配置与 PECL 注册信息后强制重装，确保 phpredis 编入两个序列化器。
RUN rm -f /usr/local/etc/php/conf.d/docker-php-ext-redis.ini \
             /usr/local/lib/php/.registry/.channel.pecl.php.net/redis.reg \
    && install-php-extensions redis

# CLI 镜像不监听 HTTP 端口，也不需要基础镜像继承的 Web 健康检查
ENV AUTORUN_LARAVEL_STORAGE_LINK=true \
    PHP_OPCACHE_ENABLE_CLI=1
HEALTHCHECK NONE

WORKDIR /var/www/html

# 使用非 root 用户运行应用命令
USER www-data

# 默认运行 Laravel 队列 Worker；定时任务容器可覆盖为 php artisan schedule:work
CMD ["php", "artisan", "queue:work", "--sleep=3", "--tries=3", "--timeout=90"]
