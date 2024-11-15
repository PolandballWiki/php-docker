# Base image with PHP extensions and Composer
FROM php:8.1-fpm-alpine AS php-base

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
		opcache \
	; \
	pecl install \
		APCu-5.1.22 \
		LuaSandbox-4.1.0 \
		redis-5.3.7 \
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

# Install Composer
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" \
	&& php composer-setup.php --install-dir=/usr/local/bin --filename=composer \
	&& php -r "unlink('composer-setup.php');"

# Entry point
CMD ["php-fpm"]
