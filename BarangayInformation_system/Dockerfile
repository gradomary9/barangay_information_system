FROM php:8.4-apache

WORKDIR /var/www/html

# I-install ang mga kailangang system tools at libraries
RUN apt-get update && apt-get install -y \
    git \
    unzip \
    zip \
    curl \
    libzip-dev \
    libpng-dev \
    libonig-dev \
    libxml2-dev \
    libsqlite3-dev \
    nodejs \
    npm \
    && docker-php-ext-install pdo pdo_mysql pdo_sqlite mbstring zip exif pcntl bcmath gd

# Kopyahin ang Composer mula sa official image
COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

# Kopyahin ang buong project files mo papasok sa container
COPY . .

# Gumawa ng directories para sa Laravel at ayusin ang permissions
RUN mkdir -p bootstrap/cache \
    storage/framework/cache \
    storage/framework/sessions \
    storage/framework/views \
    storage/logs \
    && chmod -R 775 bootstrap/cache storage \
    && chown -R www-data:www-data /var/www/html

# I-install ang backend (Composer) at frontend (NPM) dependencies
RUN composer install --no-dev --optimize-autoloader \
    && npm install \
    && npm run build

# I-clear ang mga lumang configurations ng Laravel
RUN php artisan config:clear \
    && php artisan route:clear \
    && php artisan view:clear

# Palitan ang Document Root ng Apache para tumuro sa /public folder ng Laravel
RUN sed -i 's|/var/www/html|/var/www/html/public|g' /etc/apache2/sites-available/000-default.conf \
    && a2enmod rewrite

EXPOSE 80

# Command para i-migrate ang SQLite database at simulan ang Apache server
CMD sh -c "touch /tmp/database.sqlite && chmod 666 /tmp/database.sqlite && php artisan migrate --force && php artisan db:seed --force && apache2-foreground"