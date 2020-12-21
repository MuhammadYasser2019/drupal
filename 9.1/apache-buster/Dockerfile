# from https://www.drupal.org/docs/system-requirements/php-requirements
FROM php:latest-10


# create user to run the container
# RUN useradd -u 1001 -r -g 0 -d /app -s /sbin/nologin -c "Default Application User" default 
# RUN useradd -u 1001 -r -g 0 -s /sbin/nologin -c "Default Application User" default 

# install the PHP extensions we need

RUN sudo set -eux; \
	\
	if command -v a2enmod; then \
		sudo a2enmod rewrite; \
	fi; \
	\
	savedAptMark="$(apt-mark showmanual)"; \
	\
	sudo apt-get update; \
	sudo apt-get install -y --no-install-recommends \
		libfreetype6-dev \
		libjpeg-dev \
		libpng-dev \
		libpq-dev \
		libzip-dev \
	; \
	\
	sudo docker-php-ext-configure gd \
		--with-freetype \
		--with-jpeg=/usr \
	; \
	\
	sudo docker-php-ext-install -j "$(nproc)" \
		gd \
		opcache \
		pdo_mysql \
		pdo_pgsql \
		zip \
	; \
	\
# reset apt-mark's "manual" list so that "purge --auto-remove" will remove all build dependencies
	sudo apt-mark auto '.*' > /dev/null; \
	sudo apt-mark manual $savedAptMark; \
	ldd "$(php -r 'echo ini_get("extension_dir");')"/*.so \
		| awk '/=>/ { print $3 }' \
		| sort -u \
		| xargs -r dpkg-query -S \
		| cut -d: -f1 \
		| sort -u \
		| xargs -rt apt-mark manual; \
	\
	sudo apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
	ls -ld /var/lib/apt/lists/ ; \
	sudo rm -rf /var/lib/apt/lists/* 


# set recommended PHP.ini settings
# see https://secure.php.net/manual/en/opcache.installation.php
RUN sudo chown -R 1001:0 /usr/local/etc/php/ && chmod -R g+rwX /usr/local/etc/php/ ; \
	ls -ld /usr/local/etc/php/ ;
RUN { \
		echo 'opcache.memory_consumption=128'; \
		echo 'opcache.interned_strings_buffer=8'; \
		echo 'opcache.max_accelerated_files=4000'; \
		echo 'opcache.revalidate_freq=60'; \
		echo 'opcache.fast_shutdown=1'; \
	} > /usr/local/etc/php/conf.d/opcache-recommended.ini
	# && chmod g+rwX /usr/local/etc/php/conf.d/opcache-recommended.ini
# add default user permission on php settings

# https://github.com/drupal/drupal/blob/9.0.1/composer.lock#L4052-L4053
COPY --from=composer:1.10 /usr/bin/composer /usr/local/bin/
# https://www.drupal.org/node/3060/release
ENV DRUPAL_VERSION 9.1.0

# give default user permission to the working dir
WORKDIR /opt/drupal
RUN sudo chown -R 1001:0 /opt/drupal && sudo chmod -R g+rwX /opt/drupal ; id ; ls -ld /opt/drupal/ ;

RUN set -eux; \
	export COMPOSER_HOME="$(mktemp -d)"; \
	sudo chown -R 1001:0 "$COMPOSER_HOME" && sudo chmod -R g+rwX "$COMPOSER_HOME" ;ls -ld "$COMPOSER_HOME" ; \
	sudo composer create-project --no-interaction "drupal/recommended-project:$DRUPAL_VERSION" ./ ; \
	sudo chown -R www-data:www-data web/sites web/modules web/themes; \
	sudo rmdir /var/www/html; \
	sudo ln -sf /opt/drupal/web /var/www/html; \
	# delete composer cache
	sudo rm -rf "$COMPOSER_HOME"

ENV PATH=${PATH}:/opt/drupal/vendor/bin

# RUN \
# 	apt-get update && \
# 	apt-get install sudo && \
#     sed -i /etc/sudoers -re 's/^%sudo.*/%sudo ALL=(ALL:ALL) NOPASSWD: ALL/g' && \
#     sed -i /etc/sudoers -re 's/^root.*/root ALL=(ALL:ALL) NOPASSWD: ALL/g' && \
#     sed -i /etc/sudoers -re 's/^#includedir.*/## **Removed the include directive** ##"/g' && \
#     echo "default ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

USER 1001

# CMD [ "sudo /bin/sh", "-c", "service apache2 start ; pm2-runtime process.yml --no-daemon" ]

# vim:set ft=dockerfile:
