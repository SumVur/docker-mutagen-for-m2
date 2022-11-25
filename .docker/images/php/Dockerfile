FROM php:8.1-fpm

ENV COMPOSER_HOME /var/www/.composer

RUN set -eux; \
    apt-get update; \
    apt-get install -y --no-install-recommends \
        g++ \
        git \
        golang-go \
        libfreetype6-dev \
        libicu-dev \
        libjpeg62-turbo-dev \
        libpng-dev \
        libxml2-dev \
        libxslt-dev \
        libzip-dev \
        unzip \
        zip \
        zlib1g-dev; \
    docker-php-ext-configure intl; \
    docker-php-ext-configure gd --with-freetype --with-jpeg; \
    docker-php-ext-install -j$(nproc) \
        bcmath \
        gd \
        intl \
        pdo_mysql \
        soap \
        sockets \
        xsl \
        zip; \
	\
# update pecl channel definitions https://github.com/docker-library/php/issues/443
	pecl update-channels; \
    printf "\n" | pecl install redis xdebug; \
    docker-php-ext-enable redis; \
# install mailhog sendmail
    go get github.com/mailhog/mhsendmail; \
    go clean; \
    mv $HOME/go/bin/mhsendmail /usr/bin/mhsendmail; \
    chmod +x /usr/bin/mhsendmail; \
    \
# install magerun
    EXPECTED_CHECKSUM="$(php -r 'copy("https://files.magerun.net/sha256.php?file=n98-magerun2-latest.phar", "php://stdout");' | grep -o '^\S*')"; \
    php -r "copy('https://files.magerun.net/n98-magerun2-latest.phar', '/usr/bin/n98-magerun2');"; \
    ACTUAL_CHECKSUM="$(php -r "echo hash_file('sha256', '/usr/bin/n98-magerun2');")"; \
    if [ "$EXPECTED_CHECKSUM" != "$ACTUAL_CHECKSUM" ]; then \
        >&2 echo 'ERROR: Invalid magerun checksum'; \
        exit 1; \
    fi; \
    chmod +x /usr/bin/n98-magerun2; \
    n98-magerun2 --version; \
    \
# install composer
    EXPECTED_CHECKSUM="$(php -r 'copy("https://composer.github.io/installer.sig", "php://stdout");')"; \
    php -r "copy('https://getcomposer.org/installer', '/tmp/composer-setup.php');"; \
    ACTUAL_CHECKSUM="$(php -r "echo hash_file('sha384', '/tmp/composer-setup.php');")"; \
    if [ "$EXPECTED_CHECKSUM" != "$ACTUAL_CHECKSUM" ]; then \
        >&2 echo 'ERROR: Invalid composer installer checksum'; \
        exit 1; \
    fi; \
    php /tmp/composer-setup.php --quiet --install-dir=/usr/bin; \
    composer clear-cache; \
# allow mutagen to create session https://github.com/mutagen-io/mutagen/issues/46
    chown -R www-data:www-data /var/www; \
    chmod -R 777 /var/www; \
    \
    rm -rf ~/.pearrc /tmp/* /usr/src/* /var/lib/apt/lists/*

USER www-data
ENV HOME /var/www
ENV NVM_DIR /var/www/.nvm

RUN set -eux; \
# install magepack
    mkdir -p $NVM_DIR; \
    touch "$HOME/.bashrc"; \
    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/master/install.sh | bash; \
    chmod +x "$NVM_DIR/nvm.sh"; \
    sed -i -e 's@NVM_SCRIPT_SOURCE="\$_"@NVM_SCRIPT_SOURCE="$NVM_DIR/nvm.sh"@g' "$NVM_DIR/nvm.sh"; \
    [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"; \
    [ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"; \
    nvm install --lts=dubnium; \
    npm install -g magepack; \
    magepack --version
