# ----------------------------------------
# Stage 1: Build dependencies & vendor
# ----------------------------------------
FROM php:8.2-fpm AS builder

# Install system deps
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

# Install Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

WORKDIR /var/www/html

# Copy composer files
COPY composer.json composer.lock* ./

# Install dependencies (optimized, no dev)
RUN composer install --no-dev --no-scripts --no-interaction --prefer-dist --optimize-autoloader

# Copy full app
COPY . .

# Generate autoloader + optimize
RUN composer dump-autoload --no-dev --classmap-authoritative

# Fix permissions
RUN chown -R www-data:www-data /var/www/html

# ----------------------------------------
# Stage 2: Runtime
# ----------------------------------------
FROM php:8.2-fpm

# Install runtime deps
RUN apt-get update && apt-get install -y \
    libpng-dev \
    libjpeg-dev \
    libfreetype6-dev \
    libzip-dev \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install pdo_mysql gd zip opcache \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

WORKDIR /var/www/html

# Copy app + vendor from builder
COPY --from=builder /var/www/html /var/www/html

# Laravel optimizations (safe in production)
RUN php artisan config:cache && \
    php artisan route:cache && \
    php artisan view:cache && \
    chown -R www-data:www-data /var/www/html/storage /var/www/html/bootstrap/cache && \
    chmod -R 775 /var/www/html/storage /var/www/html/bootstrap/cache

EXPOSE 9000
CMD ["php-fpm"]