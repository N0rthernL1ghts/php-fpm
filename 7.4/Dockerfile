ARG ALPINE_VERSION=latest
ARG PHP_VERSION=7.4

# Stage 1 - Prepare PHP and copy ENV
FROM --platform=${TARGETPLATFORM} php:${PHP_VERSION}-fpm-alpine${ALPINE_VERSION} as php


# Stage 2 - Build rootfs
FROM scratch AS rootfs
# TODO: Copy only relevant sections, to reduce risk of potentional issues with mixing two images
COPY --from=php /usr/local/include/php /usr/local/include/php/
COPY --from=php /usr/local/lib/php /usr/local/lib/php/
COPY --from=php /usr/local/sbin /usr/local/sbin/
COPY --from=php /usr/local/php /usr/local/php/
COPY --from=php /usr/local/bin /usr/local/bin/
COPY --from=php /usr/local/etc /usr/local/etc/
COPY --from=php /var/www/html /var/www/html/
COPY --from=php /usr/src /usr/src/

# Initialize files
COPY ["./rootfs", "/"]


# Stage 3 - Final stage
FROM --platform=${TARGETPLATFORM} nlss/base-alpine:${ALPINE_VERSION}

# We copy these manually from php image as we have no way to export to another stage
ENV PHP_INI_DIR "/usr/local/etc/php"
ENV PHP_VERSION "${PHP_VERSION}"
ENV PHP_URL     "https://www.php.net/distributions/php-${PHP_VERSION}.tar.xz" PHP_ASC_URL="https://www.php.net/distributions/php-${PHP_VERSION}.tar.xz.asc"
ENV PHPIZE_DEPS \
		autoconf \
		dpkg-dev dpkg \
		file \
		g++ \
		gcc \
		libc-dev \
		make \
		pkgconf \
		re2c
# Note: PHPIZE_DEPS are not going to be installed. This env is utilized by internal utilities, per need. 

# Copy ready rootfs
COPY --from=rootfs / /

# This part is adapted from php official image
RUN addgroup -g 82 -S www-data \
	&& adduser -u 82 -D -S -G www-data www-data \
    && RUNTIME_DEPS="$( \
		scanelf --needed --nobanner --format '%n#p' --recursive /usr/local \
			| tr ',' '\n' \
			| sort -u \
			| awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
	)" \
	&& apk add --no-cache ${RUNTIME_DEPS} ca-certificates curl openssl tar xz
