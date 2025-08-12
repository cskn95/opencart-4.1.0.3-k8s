FROM php:8.2-apache

# Install dependencies and tools
RUN apt-get update \
  && apt-get install -y --no-install-recommends \
       wait-for-it \
       unzip \
       libfreetype6-dev \
       libjpeg62-turbo-dev \
       libpng-dev \
       libzip-dev \
       libcurl3-dev \
       libwebp-dev \
  && docker-php-ext-configure gd --with-freetype --with-jpeg --with-webp \
  && docker-php-ext-install -j"$(nproc)" gd zip mysqli curl \
  && docker-php-ext-enable gd zip mysqli curl \
  && a2enmod rewrite \
  && rm -rf /var/lib/apt/lists/*

WORKDIR /var/www/html

# Copy OpenCart application from repo
COPY upload/ /var/www/html/

# Permissions (be permissive for dev)
RUN chown -R www-data:www-data /var/www/html \
  && find /var/www/html/system/storage -type d -exec chmod 775 {} + 2>/dev/null || true \
  && find /var/www/html/system/storage -type f -exec chmod 664 {} + 2>/dev/null || true \
  && chmod -R 775 /var/www/html/image 2>/dev/null || true \
  && chmod 664 /var/www/html/config.php 2>/dev/null || true \
  && chmod 664 /var/www/html/admin/config.php 2>/dev/null || true \
  && rm -f /var/www/html/install.lock

# Default command
CMD ["apache2-foreground"]
