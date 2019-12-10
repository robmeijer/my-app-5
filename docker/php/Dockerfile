# The PHP version that will be built by default
ARG PHP_VERSION=7.4

FROM php:${PHP_VERSION}-fpm-alpine

# Optionally include MongoDB support
ARG WITH_MONGODB=false

# Use the default production configuration
RUN cp ${PHP_INI_DIR}/php.ini-production ${PHP_INI_DIR}/php.ini

# Copy default settings
COPY docker/php/conf.d/default.ini ${PHP_INI_DIR}/conf.d/default.ini

# Install dependencies
RUN set -eux; \
    apk --update add --no-cache --virtual .build-deps \
        gmp-dev \
        icu-dev \
        libressl-dev \
        libzip-dev \
        zlib-dev \
    ; \
    if [ ${WITH_MONGODB} = true ]; then \
        apk add --no-cache --virtual .mongodb-deps ${PHPIZE_DEPS}; \
    fi; \
    docker-php-ext-configure zip; \
    docker-php-ext-install \
        gmp \
        intl \
        pcntl \
        pdo_mysql \
        zip \
    ; \
    if [ ${WITH_MONGODB} = true ]; then \
        pecl -q install mongodb; \
        pecl clear-cache; \
        docker-php-ext-enable mongodb; \
    fi; \
    docker-php-ext-enable \
        opcache \
    ; \
    RUNDEPS="$( \
        scanelf --needed --nobanner --format '%n#p' --recursive /usr/local/lib/php/extensions \
            | tr ',' '\n' \
            | sort -u \
            | awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
    )"; \
    apk add --no-cache --virtual .run-deps ${RUNDEPS}; \
    apk del .build-deps; \
    if [ ${WITH_MONGODB} = true ]; then \
        apk del .mongodb-deps; \
    fi

CMD ["php-fpm"]
