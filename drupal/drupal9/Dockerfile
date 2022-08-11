FROM php:8.0-apache-bullseye
# наследуемся от пакета php с апачем

# Установка необходимых расширейний и библиотек для Drupal 9
RUN set -eux; \
    \
    if command -v a2enmod; then \
        a2enmod rewrite; \
    fi; \
    \
    savedAptMark="$(apt-mark showmanual)"; \
    \
    apt-get update; \
    apt-get install -y --no-install-recommends libfreetype6-dev libjpeg-dev libpng-dev libpq-dev libwebp-dev libzip-dev; \
    docker-php-ext-configure gd --with-freetype --with-jpeg=/usr --with-webp; \
    docker-php-ext-install -j "$(nproc)" gd opcache pdo_mysql pdo_pgsql zip; \
    apt-mark auto '.*' > /dev/null; \
    apt-mark manual $savedAptMark; \
    ldd "$(php -r 'echo ini_get("extension_dir");')"/*.so \
        | awk '/=>/ { print $3 }' \
        | sort -u \
        | xargs -r dpkg-query -S \
        | cut -d: -f1 \
        | sort -u \
        | xargs -rt apt-mark manual; \
    \
    apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
    rm -rf /var/lib/apt/lists/*

# настройка Opcache
# see https://secure.php.net/manual/en/opcache.installation.php
RUN { \
        echo 'opcache.memory_consumption=128'; \
        echo 'opcache.interned_strings_buffer=8'; \
        echo 'opcache.max_accelerated_files=4000'; \
        echo 'opcache.revalidate_freq=60'; \
        echo 'opcache.fast_shutdown=1'; \
    } > /usr/local/etc/php/conf.d/opcache-recommended.ini

# добавляем composer копируем файлы из докер образа composer:2
COPY --from=composer:2 /usr/bin/composer /usr/local/bin/

# установка Drush
# устанавливается глобально в директорию /root/.composer/vendor/bin
RUN set -eux; composer global require drush/drush

# добавляем Drush в PATH
ENV PATH=${PATH}:/root/.composer/vendor/bin

# определяем UID пользователя для www-data = 1000 для редактирования смотрированных директорий
RUN usermod -u 1000 www-data

# рабочая директория, в которой размещается основной сайт apache2
WORKDIR /var/www/html

# определяем скрипт для запуска
# скрипт запускает apache2, устанавливает владельца на директории и читает лог ошибок с выводом в консоль
RUN echo "#!/bin/sh\n\nservice apache2 start && chown -R www-data:www-data sites modules themes && tail -F /var/log/apache2/error.log" > /usr/local/bin/run.sh; \
    chmod a+x /usr/local/bin/run.sh

EXPOSE 80
