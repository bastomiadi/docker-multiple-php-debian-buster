FROM php:8.0.10-apache-buster

# Surpresses debconf complaints of trying to install apt packages interactively
# https://github.com/moby/moby/issues/4032#issuecomment-192327844
 
ARG DEBIAN_FRONTEND=noninteractive
ARG UNAME=bastomi
ARG UID=1000
ARG GID=1000

# Update
RUN apt-get -y update --fix-missing && \
    apt-get upgrade -y && \
    apt-get --no-install-recommends install -y apt-utils && \
    rm -rf /var/lib/apt/lists/*


# Install useful tools and install important libaries
RUN apt-get -y update && \
    apt-get -y --no-install-recommends install nano wget \
dialog \
sudo \
libsqlite3-dev \
libsqlite3-0 && \
    apt-get -y --no-install-recommends install default-mysql-client \
zlib1g-dev \
libzip-dev \
libicu-dev && \
    apt-get -y --no-install-recommends install --fix-missing apt-utils \
build-essential \
git \
curl \
libonig-dev && \ 
    apt-get -y --no-install-recommends install --fix-missing libcurl4 \
libcurl4-openssl-dev \
zip \
unzip \
gzip \
openssl && \
    rm -rf /var/lib/apt/lists/* && \
    curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Install xdebug
RUN pecl install xdebug-3.0.0 && \
    docker-php-ext-enable xdebug

# Install redis
RUN pecl install redis-5.3.3 && \
    docker-php-ext-enable redis

# Install imagick
RUN apt-get update && \
    apt-get -y --no-install-recommends install --fix-missing libmagickwand-dev && \
    rm -rf /var/lib/apt/lists/*

# Workarround until imagick is available in pecl with php8 support
# Imagick Commit to install
# https://github.com/Imagick/imagick
ARG IMAGICK_COMMIT="132a11fd26675db9eb9f0e9a3e2887c161875206"

RUN cd /usr/local/src && \
    git clone https://github.com/Imagick/imagick && \
    cd imagick && \
    git checkout ${IMAGICK_COMMIT} && \
    phpize && \
    ./configure && \
    make && \
    make install && \
    cd .. && \
    rm -rf imagick && \
    docker-php-ext-enable imagick

# Other PHP8 Extensions
    
RUN docker-php-ext-install bcmath && \
    docker-php-ext-install intl && \
    docker-php-ext-install pdo_mysql && \
    docker-php-ext-install pdo_sqlite && \
    docker-php-ext-install mysqli && \
    docker-php-ext-install curl && \
    docker-php-ext-install tokenizer && \
    docker-php-ext-install zip && \
    docker-php-ext-install -j$(nproc) intl && \
    docker-php-ext-install mbstring && \
    docker-php-ext-install gettext && \
    docker-php-ext-install calendar && \
    docker-php-ext-install exif


# Install Freetype 
RUN apt-get -y update && \
    apt-get --no-install-recommends install -y libfreetype6-dev \
libjpeg62-turbo-dev \
libpng-dev && \
    rm -rf /var/lib/apt/lists/* && \
    docker-php-ext-configure gd --enable-gd --with-freetype --with-jpeg && \
    docker-php-ext-install gd
    
# add user & permission
RUN groupadd -g $GID -o $UNAME && \ 
    useradd -m -u $UID -g $GID -o -s /bin/bash $UNAME && \
    echo "${UNAME}:bastomi" | chpasswd && \
    addgroup wheel && \
    echo '%wheel ALL=(ALL) ALL' >> /etc/sudoers && \
    echo '${UNAME} ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers && \
    usermod -aG wheel $UNAME && \
    chown -R $UNAME:$UNAME /var/www
    
# add aliases
RUN echo "alias yii-advanced='composer create-project --prefer-dist yiisoft/yii2-app-advanced'" >> /home/${UNAME}/.bash_aliases && \
    echo "alias yii-basic='composer create-project --prefer-dist yiisoft/yii2-app-basic'" >> /home/${UNAME}/.bash_aliases && \
    echo "alias laravel='composer create-project laravel/laravel'" >> /home/${UNAME}/.bash_aliases && \
    echo "alias ci4='composer create-project codeigniter4/appstarter'" >> /home/${UNAME}/.bash_aliases
    #source /home/${UNAME}/.bash_aliases && \
    #source /home/${UNAME}/.bashrc

# Enable apache modules
RUN a2enmod rewrite headers

# Cleanup
RUN rm -rf /usr/src/*

USER $UNAME
