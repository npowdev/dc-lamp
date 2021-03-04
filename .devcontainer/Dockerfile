# Note: You can use any Debian/Ubuntu based image you want. 
# FROM mcr.microsoft.com/vscode/devcontainers/base:0-buster
FROM php:7.4-apache

# [Option] Install zsh
ARG INSTALL_ZSH="true"
# [Option] Upgrade OS packages to their latest versions
ARG UPGRADE_PACKAGES="false"
# [Option] Enable non-root Docker access in container
ARG ENABLE_NONROOT_DOCKER="true"
# [Option] Use the OSS Moby CLI instead of the licensed Docker CLI
ARG USE_MOBY="true"

# Install needed packages and setup non-root user. Use a separate RUN statement to add your
# own dependencies. A user of "automatic" attempts to reuse an user ID if one already exists.
ARG USERNAME=automatic
ARG USER_UID=1000
ARG USER_GID=$USER_UID
COPY library-scripts/*.sh /tmp/library-scripts/
RUN apt-get update \
    && /bin/bash /tmp/library-scripts/common-debian.sh "${INSTALL_ZSH}" "${USERNAME}" "${USER_UID}" "${USER_GID}" "${UPGRADE_PACKAGES}" \
    # Use Docker script from script library to set things up
    && /bin/bash /tmp/library-scripts/docker-debian.sh "${ENABLE_NONROOT_DOCKER}" "/var/run/docker-host.sock" "/var/run/docker.sock" "${USERNAME}" \
    # Clean up
    && apt-get autoremove -y && apt-get clean -y && rm -rf /var/lib/apt/lists/* /tmp/library-scripts/

# Setting the ENTRYPOINT to docker-init.sh will configure non-root access 
# to the Docker socket. The script will also execute CMD as needed.
ENTRYPOINT [ "/usr/local/share/docker-init.sh" ]
CMD [ "sleep", "infinity" ]
# use command: ["apache2-foreground"] in docker-compose to get apache running.

ENV QDBM_URL="https://fossies.org/linux/misc/qdbm-1.8.78.tar.gz"
#            "http://fallabs.com/qdbm/qdbm-1.8.78.tar.gz"
#            "https://dbmx.net/qdbm/qdbm-1.8.78.tar.gz"

# -----------------------------------------------------------------------------
# Install additional OS packages.
# -----------------------------------------------------------------------------
RUN apt-get update -yq && export DEBIAN_FRONTEND=noninteractive \
    && apt-get -yq install --no-install-recommends --no-install-suggests \
    # tools
    mc lynx nano wget dialog default-mysql-client \
    # libs
    zlib1g-dev libzip-dev libicu-dev libsqlite3-dev libsqlite3-0 \
    # -------------------------------------------------------------------------
    # Package for php extensions
    # -------------------------------------------------------------------------
    && apt-get install -yq --no-install-recommends --no-install-suggests \
    # For gmp
    libgmp-dev \
    # For bz2
    libbz2-dev \
    # For mbstring
    libonig-dev \
    # For imap
    libc-client-dev libkrb5-dev \
    # For mcrypt
    # libmcrypt-dev \
    # For xsl
    libxslt1-dev \
    # For WDDX
    # libexpat1-dev \
    # For SOAP, DOM, libxml, SimpleXML, WDDX, XSL, XML, XMLReader, XMLRPC, XMLWriter
    libxml2-dev \
    # For memcached
    libmemcached-dev libmemcached11 libhashkit2 libmemcachedutil2 zlib1g-dev \
    # For redis: to enable zstd, no liblzf-dev liblzf1 to enable lzf
    libzstd-dev libzstd1 \
    # for pdo_pgsql pgsql
    libpq-dev \
    # For gd with Freetype + JIS-mapped Japanese Font, 
    #             GIF, JPEG, PNG, WBMP, XPM, XBM, WebP
    libfreetype6-dev libfreetype6 libjpeg62-turbo-dev libjpeg62-turbo \
    libjpeg-dev libpng-dev libpng16-16 libxpm-dev libvpx-dev libwebp-dev \
    libwebp6 libxpm-dev libxpm4 libgd-dev libgd3 \
    # For imagick
    libmagickwand-dev \
    # -------------------------------------------------------------------------
    # For dba with qdbm filter
    && apt-get install -yq --no-install-recommends --no-install-suggests \
    liblzo2-dev liblzo2-2 liblmdb-dev liblmdb0 \
    # [opt-in / needed?]
    # libdb-dev \
    # libdb5.3-dev libdb5.3 
    # libdb5.3++-dev libdb5.3++ \
    # -> get QDBM source
    && cd /usr/src \
    && curl -sSL -o qdbm.tar.gz "${QDBM_URL}" \
    # -> extract files
    && mkdir -p /usr/src/qdbm \
    && tar -xf qdbm.tar.gz -C /usr/src/qdbm --strip-components=1 \
    # -> build and install
    && cd /usr/src/qdbm \
    && ./configure --enable-stable --enable-zlib --enable-bzip --enable-lzo --enable-iconv \
    && make \
    && make install \
    # -> cleanup qdbm source
    && cd /usr/src \
    && rm -f qdbm.tar.gz \
    && rm -rf /usr/src/qdbm \
    # -------------------------------------------------------------------------
    && apt-get install -yq --no-install-recommends --fix-missing \
    apt-utils build-essential git curl zip openssl \
    # libcurl3 & libcurl3-dev -> libcurl4 libcurl4-openssl-dev
    libcurl4 libcurl4-openssl-dev \
    # -------------------------------------------------------------------------
    # composer support
    && curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer \
    # -------------------------------------------------------------------------
    # Clean up
    && apt-get clean -y && rm -rf /var/lib/apt/lists/* /tmp/*

# -----------------------------------------------------------------------------
# Instal and setup PHP Extensions
# -----------------------------------------------------------------------------
RUN set -eu \
    # -------------------------------------------------------------------------
    # Package on php - disabled, not needed, or outdated.
    # -------------------------------------------------------------------------
    # # mcrypt (mcrypt-1.0.3 for php7.2+) need lib: libmcrypt-dev
    # && pecl install mcrypt-1.0.3 \
    # && docker-php-ext-enable mcrypt \
    # # hash with mhash need cfg: --with-mhash
    # && docker-php-ext-configure hash --with-mhash \
    # && docker-php-ext-install hash \
    # # pcntl
    # && docker-php-ext-install pcntl \
    # # wddx need lib: libexpat1-dev
    # && docker-php-ext-install wddx \
    #
    # -------------------------------------------------------------------------
    # Pre-bundled package on php - no install needed.
    # -------------------------------------------------------------------------
    # && docker-php-ext-install pdo_sqlite \
    # && docker-php-ext-install curl \
    # && docker-php-ext-install tokenizer \
    # && docker-php-ext-install json \
    # # mbstring need lib: libonig-dev
    # && docker-php-ext-install mbstring \
    #
    # -------------------------------------------------------------------------
    # New package on php - not pre-bundled - need install.
    # -------------------------------------------------------------------------
    && docker-php-ext-install bcmath \
    # bz2 need lib: libbz2-dev
    && docker-php-ext-install bz2 \
    && docker-php-ext-install calendar \
    && docker-php-ext-install pdo_mysql \
    && docker-php-ext-install mysqli \
    && docker-php-ext-install zip \
    && docker-php-ext-install -j$(nproc) intl \
    && docker-php-ext-install gettext \
    # gmp need lib: libgmp-dev
    && docker-php-ext-install gmp \
    && docker-php-ext-install exif \
    # apcu
    && pecl install apcu \
    && docker-php-ext-enable apcu \
    && echo "apc.shm_size=8M" >> /usr/local/etc/php/conf.d/docker-php-ext-apcu.ini \
    # pdo_pgsql
    && docker-php-ext-install -j$(nproc) pdo_pgsql \
    # pgsql
    && docker-php-ext-install -j$(nproc) pgsql \
    # imap need lib: libc-client-dev libkrb5-dev
    #      need cfg: --with-kerberos --with-imap-ssl
    && docker-php-ext-configure imap --with-kerberos --with-imap-ssl \
    && docker-php-ext-install imap \
    # shmop (Shared Memory)
    && docker-php-ext-install shmop \
    # soap need lib: libxml2-dev
    && docker-php-ext-install soap \
    # sockets
    && docker-php-ext-install sockets \
    # sysvmsg
    && docker-php-ext-install sysvmsg \
    # sysvsem
    && docker-php-ext-install sysvsem \
    # sysvshm
    && docker-php-ext-install sysvshm \
    # xmlrpc need lib: libxml2-dev
    && docker-php-ext-install xmlrpc \
    # xsl need lib: libxslt1-dev
    && docker-php-ext-install xsl \
    # -------------------------------------------------------------------------
    # dba need lib: liblzo2-dev liblzo2-2
    #               qdbm: get "https://fossies.org/linux/misc/qdbm-1.8.78.tar.gz"
    #                     cfg/make/inst: --enable-stable --enable-zlib --enable-bzip --enable-lzo --enable-iconv
    #     need cfg: --with-cdb --with-qdbm \
    #   failed cfg: --with-db4 
    && docker-php-ext-configure dba --with-cdb --with-qdbm --with-lmdb \
    && docker-php-ext-install dba \
    # -------------------------------------------------------------------------
    # Install gd with Freetype
    #   need libs: libfreetype6-dev libfreetype6 libjpeg62-turbo-dev libjpeg62-turbo libjpeg-dev 
    #              libpng-dev libpng16-16 libxpm-dev libvpx-dev libwebp-dev libwebp6 libxpm-dev 
    #              libxpm4 libgd-dev libgd3 
    #   cfg (php v7.3): --with-gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ --with-png-dir=/usr/include/ --with-xpm-dir=/usr/include/ --with-webp-dir=/usr/include/ --enable-gd-jis-conv
    #   cfg (php v7.4+): --enable-gd --with-external-gd --with-freetype --with-jpeg --with-webp --with-xpm
    # -------------------------------------------------------------------------
    \
    && docker-php-ext-configure gd --enable-gd --with-external-gd --with-freetype --with-jpeg --with-webp --with-xpm \
    && docker-php-ext-install -j$(nproc) gd \
    \
    # -------------------------------------------------------------------------
    # imagick need lib: libmagickwand-dev
    # need rebuild, no use of: pecl install imagick && docker-php-ext-enable imagick
    \
    && cd /tmp \
    && wget https://pecl.php.net/get/imagick-3.4.4.tgz \
    && tar xvzf imagick-3.4.4.tgz \
    && cd imagick-3.4.4 \
    && phpize \
    && ./configure \
    && make install \
    && cd /tmp \
    && rm -rf /tmp/imagick-3.4.4* \
    && echo extension=imagick >> /usr/local/etc/php/conf.d/docker-php-ext-imagick.ini \
    && echo "imagick.skip_version_check=true" >>/usr/local/etc/php/conf.d/docker-php-ext-imagick.ini \
    \
    # -------------------------------------------------------------------------
    # msgpack
    && pecl install msgpack \
    && docker-php-ext-enable msgpack \
    \
    # --------
    # igbinary
    # need rebuild, no use of: pecl install igbinary && docker-php-ext-enable igbinary
    \
    && curl -fsSL 'https://github.com/igbinary/igbinary/archive/3.2.1.tar.gz' -o igbinary.tar.gz \
    && mkdir -p igbinary \
    && tar -xf igbinary.tar.gz -C igbinary --strip-components=1 \
    && rm igbinary.tar.gz \
    && ( \
    cd igbinary \
    && phpize \
    && ./configure CFLAGS="-O2 -g" --enable-igbinary \
    && make -j$(nproc) \
    && make install \
    ) \
    && rm -r igbinary \
    && docker-php-ext-enable igbinary \
    \
    # -------------------------------------------------------------------------
    # memcached need lib: libmemcached-dev libmemcached11 libhashkit2 
    #                     libmemcachedutil2 zlib1g-dev 
    # need rebuild, no use of: pecl install memcached && docker-php-ext-enable memcached
    \
    && curl -fsSL 'https://github.com/php-memcached-dev/php-memcached/archive/v3.1.5.tar.gz' -o memcached.tar.gz \
    && mkdir -p memcached \
    && tar -xf memcached.tar.gz -C memcached --strip-components=1 \
    && rm memcached.tar.gz \
    && ( \
    cd memcached \
    && phpize \
    && ./configure CFLAGS="-O2 -g" --enable-memcached-igbinary --enable-memcached-json --enable-memcached-msgpack --enable-memcached-sasl \
    && make -j$(nproc) \
    && make install \
    ) \
    && rm -r memcached \
    && docker-php-ext-enable memcached \
    \
    # -------------------------------------------------------------------------
    # redis-5.3.2 - use printf: to enable igbinary serializer support
    #                           to disable lzf and zstd support
    && printf "y\n\n" | pecl install redis-5.3.2 \
    && docker-php-ext-enable redis \
    # -------------------------------------------------------------------------
    # opcache
    && docker-php-ext-enable opcache \
    && echo "opcache.enable_file_override=On" >>/usr/local/etc/php/conf.d/docker-php-ext-opcache.ini \
    && echo "opcache.force_restart_timeout=60" >>/usr/local/etc/php/conf.d/docker-php-ext-opcache.ini \
    && echo "opcache.revalidate_freq=30" >>/usr/local/etc/php/conf.d/docker-php-ext-opcache.ini \
    && echo "opcache.revalidate_path=On" >>/usr/local/etc/php/conf.d/docker-php-ext-opcache.ini \
    # -------------------------------------------------------------------------
    # setup xdebug: pecl install xdebug-2.7.2 -> xdebug (v3+)
    && pecl install xdebug \
    && docker-php-ext-enable xdebug \
    && echo "xdebug.remote_enable=1" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    # && echo "xdebug.remote_autostart=0" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.default_enable=0" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    # && echo "xdebug.remote_host=host.docker.internal" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.remote_port=9000" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.remote_connect_back=0" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.profiler_enable=0" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.remote_log=\"/tmp/xdebug.log\"" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    # -------------------------------------------------------------------------
    && rm -rf /tmp/*
