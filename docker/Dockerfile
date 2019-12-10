ARG PHP_IMAGE=robmeijer/php:7.3-fpm-alpine

FROM composer:1.9 as composer

RUN rm -rf /var/www && mkdir /var/www

ENV COMPOSER_ALLOW_SUPERUSER=1
RUN set -eux; \
	composer global require hirak/prestissimo --prefer-dist --no-progress --no-suggest --no-plugins --no-scripts --classmap-authoritative; \
	composer clear-cache

WORKDIR /var/www

COPY composer.* /var/www/

ARG APP_ENV=prod

RUN set -xe \
    && if [ "$APP_ENV" = "prod" ]; then export ARGS="--no-dev"; fi \
    && composer install --prefer-dist --no-scripts --no-progress --no-suggest --no-interaction $ARGS; \
    composer clear-cache

COPY . /var/www

RUN composer dump-autoload --classmap-authoritative

FROM ${PHP_IMAGE}

ARG APP_ENV=prod

ENV APP_ENV $APP_ENV

WORKDIR /var/www

COPY --from=composer /var/www/ /var/www/
COPY --from=composer /usr/bin/composer /usr/bin/composer

# Memory limit increase is required by the dev image
RUN php -d memory_limit=256M bin/console cache:clear
RUN bin/console assets:install

COPY docker/php/docker-entrypoint.sh /usr/local/bin/docker-entrypoint
RUN chmod +x /usr/local/bin/docker-entrypoint

ENTRYPOINT ["docker-entrypoint"]
CMD ["php-fpm"]
