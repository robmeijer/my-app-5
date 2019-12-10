ARG ASSETS_IMAGE=robmeijer/symfony-app:latest

FROM ${ASSETS_IMAGE} AS assets

FROM nginx:1.17-alpine

COPY docker/nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf

COPY --from=assets /var/www/public /var/www/public
