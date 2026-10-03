# Base image with PHP extensions and Composer
FROM php:8.5.11-fpm-alpine@sha256:fa01fb1645cd0fc566a5f146b099adace33b906571f972f71f2182a7c12d1cd7 AS php-base

# System dependencies
RUN set -eux; \
	apk add --no-cache \
    # Nginx and supervisor
    ## For some reason supervisor uses bash instead of sh and i don't want to mess with the config
        bash \
        nginx \
        supervisor \
    # MediaWiki dependencies
		diffutils \
		git \
		imagemagick \
		python3 \
		py3-yaml \
		vips-tools \
	;

# Install PHP extensions
RUN set -eux; \
	apk add --no-cache --virtual .build-deps \
		$PHPIZE_DEPS \
		icu-dev \
		lua5.1 \
		lua5.1-dev \
		oniguruma-dev \
	; \
	docker-php-ext-install -j"$(nproc)" \
		calendar \
		intl \
		mbstring \
		mysqli \
	; \
	pecl install \
		APCu-5.1.28 \
		LuaSandbox-4.1.3 \
		redis-6.3.0 \
	; \
	docker-php-ext-enable \
		apcu \
		luasandbox \
		redis \
	; \
	rm -r /tmp/pear; \
	runDeps="$( \
		scanelf --needed --nobanner --format '%n#p' --recursive /usr/local/lib/php/extensions \
			| tr ',' '\n' \
			| sort -u \
			| awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
	)"; \
	apk add --virtual .phpext-rundeps $runDeps; \
	apk del .build-deps

# OPcache is built into PHP 8.5 and must not be built as a separate extension.
# Download an exact Composer release and verify its SHA256 before executing it.
ARG COMPOSER_VERSION=2.10.3
ARG COMPOSER_SHA256=7a2d379d5b8ffdaa028580ef26494c36d2feef4b178d3dd1473a4dbc5e17c8d6
RUN set -eux; \
    curl -fsSL "https://getcomposer.org/download/${COMPOSER_VERSION}/composer.phar" -o /usr/local/bin/composer; \
    echo "${COMPOSER_SHA256}  /usr/local/bin/composer" | sha256sum -c -; \
    chmod 0755 /usr/local/bin/composer; \
    composer --version

# Entry point
CMD ["php-fpm"]
