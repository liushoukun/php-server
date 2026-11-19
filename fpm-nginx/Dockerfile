############################################
# Base Image - 基础镜像
# 包含 PHP 扩展和基础配置，这些内容很少变化
# 可以单独构建并推送到镜像仓库，避免每次重复构建
############################################

FROM serversideup/php:8.4-fpm-nginx

# 切换到 root 用户安装 PHP 扩展
USER root

# 安装 PHP 扩展（已去重）
# 基础镜像预装了 phpredis，直接列 redis 会被跳过，导致 igbinary/msgpack
# 无法编入 phpredis 的序列化器。因此先装序列化扩展，再删除旧 redis
# 配置与 PECL 注册信息后强制重装 redis。
RUN install-php-extensions \
    bcmath \
    gd \
    intl \
    mbstring \
    pcntl \
    igbinary \
    msgpack \
    exif \
    opcache \
    && rm -f /usr/local/etc/php/conf.d/docker-php-ext-redis.ini \
             /usr/local/lib/php/.registry/.channel.pecl.php.net/redis.reg \
    && install-php-extensions redis

# 设置环境变量
ENV AUTORUN_LARAVEL_STORAGE_LINK=true \
    PHP_OPCACHE_ENABLE=true

# 设置工作目录
WORKDIR /var/www/html

# 切换回 www-data 用户
USER www-data
