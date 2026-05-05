FROM composer/composer:2-bin AS composer

FROM dunglas/frankenphp:1-php8.2-bookworm AS base

RUN apt update && apt -y install \
    unzip

RUN install-php-extensions \
    opcache \
    mbstring \
    intl \
    dom \
    ctype \
    curl \
    phar \
    openssl \
    xml \
    xmlwriter \
    simplexml \
    pdo

ENV LC_ALL=C.UTF-8

#
# Development
#

FROM base AS dev

# Valores padrão para evitar erro de variável vazia no Windows
ARG USER_ID=1000
ARG GROUP_ID=1000
ARG USER_NAME=appuser
ARG GROUP_NAME=appgroup

RUN install-php-extensions \
    xdebug

COPY --from=composer /composer /usr/bin/composer

# Cria usuário/grupo e ajusta permissões em TODAS as pastas necessárias
RUN if [ -z "${GROUP_ID}" ] || [ "${GROUP_ID}" = "" ]; then GROUP_ID=1000; fi && \
    if [ -z "${USER_ID}" ] || [ "${USER_ID}" = "" ]; then USER_ID=1000; fi && \
    groupadd --gid $GROUP_ID $GROUP_NAME || true && \
    useradd --gid $GROUP_ID --uid $USER_ID -m $USER_NAME || true && \
    setcap CAP_NET_BIND_SERVICE=+eip /usr/local/bin/frankenphp && \
    # CORREÇÃO CRÍTICA: Dando permissão para o appuser criar pastas de certificados e logs
    mkdir -p /data/caddy /config/caddy /app/runtime && \
    chown -R $USER_NAME:$GROUP_NAME /data /config /app

WORKDIR /app

USER ${USER_NAME}

COPY docker/dev/Caddyfile /etc/frankenphp/Caddyfile

#
# Production
#

FROM base AS prod-builder
COPY --from=composer /composer /usr/bin/composer
COPY . /app
RUN --mount=type=cache,target=/tmp/cache \
    composer install --no-dev --no-progress --no-interaction --classmap-authoritative && \
    rm composer.lock composer.json

FROM base AS prod
ENV APP_ENV=prod
COPY docker/Caddyfile /etc/frankenphp/Caddyfile
COPY --from=prod-builder --chown=www-data:www-data /app /app
USER www-data