# ----------------------------------------
#  Stage 1: Build dependencies & vendor
# ----------------------------------------
FROM php:8.2-fpm AS builder

# Install system packages & PHP extensions
RUN apt-get update && apt-get install -y \
    git \
    curl \
    unzip \
    libpng-dev \
    libjpeg-dev \
    libfreetype6-dev \
    libzip-dev \
    zip \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install pdo_mysql gd zip opcache \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# Install Composer globally
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

WORKDIR /var/www/html

# Copy only composer files first for caching
COPY composer.json composer.lock ./

# Install dependencies (no dev for pre-prod)
RUN composer update --no-dev --no-scripts --no-interaction --prefer-dist --optimize-autoloader

# Copy the rest of the Laravel project
COPY . .

# Run post-install optimizations
RUN php artisan config:clear && php artisan cache:clear && php artisan view:clear

# ----------------------------------------
#  Stage 2: Final lightweight runtime
# ----------------------------------------
FROM php:8.2-fpm

# Copy PHP extensions & settings from builder
COPY --from=builder /usr/local/etc/php/conf.d /usr/local/etc/php/conf.d
COPY --from=builder /usr/local/bin/composer /usr/local/bin/composer

# Install minimal runtime dependencies (no build tools)
RUN apt-get update && apt-get install -y \
    libpng-dev \
    libjpeg-dev \
    libfreetype6-dev \
    libzip-dev \
    zip \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install pdo_mysql gd zip opcache \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

WORKDIR /var/www/html

# Copy application code & vendor from builder
COPY --from=builder /var/www/html /var/www/html

# Optimize Laravel for production
RUN php artisan config:cache && \
    php artisan route:cache && \
    php artisan view:cache && \
    chown -R www-data:www-data /var/www/html/storage /var/www/html/bootstrap/cache && \
    chmod -R 775 /var/www/html/storage /var/www/html/bootstrap/cache

# Expose FPM port
EXPOSE 9000

# Start PHP-FPM
CMD ["php-fpm"]