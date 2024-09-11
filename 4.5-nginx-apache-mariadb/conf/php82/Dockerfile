FROM php:8.2-apache

# Install packages
RUN apt-get update && apt-get install -y --no-install-recommends \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libpng-dev \
		libzip-dev \
		zip \
		openssl \
		librdkafka-dev  \
	&& mv "$PHP_INI_DIR/php.ini-production" "$PHP_INI_DIR/php.ini"  \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j$(nproc) gd pdo_mysql zip  \
    && a2enmod rewrite \
    && a2enmod ssl \
    && a2enmod socache_shmcb \
    && a2enmod headers \
#	&& pecl install rdkafka \
#	&& echo "extension=rdkafka.so" > "$PHP_INI_DIR/conf.d/rdkafka.ini"  \
#   && docker-php-ext-configure opcache --enable-opcache \
#	&& docker-php-ext-install opcache \
    && echo "ServerSignature Off" >> /etc/apache2/apache2.conf \
    && echo "ServerTokens Prod" >> /etc/apache2/apache2.conf \
    && sed -i -e "s/expose_php\s*=\s*On/expose_php = Off/g" "$PHP_INI_DIR/php.ini" \
	&& sed -i -e "s/memory_limit\s*=\s*.*/memory_limit = 128M/g" "$PHP_INI_DIR/php.ini" \
    && sed -i -e "s/upload_max_filesize\s*=\s*2M/upload_max_filesize = 250M/g" "$PHP_INI_DIR/php.ini" \
    && sed -i -e "s/post_max_size\s*=\s*8M/post_max_size = 250M/g" "$PHP_INI_DIR/php.ini"

RUN openssl req -newkey rsa:2048 -nodes -keyout /etc/ssl/private/ssl-cert-snakeoil.key -x509 -days 3650 -out /etc/ssl/certs/ssl-cert-snakeoil.pem -subj "/C=VN/ST=HaNoi/L=HaNoi/O=Vinades JSC/OU=DauThau/CN=CA/emailAddress=webmaster@vinades.vn"
RUN a2ensite default-ssl

# Đổi port mặc định 80 => 8080
# Đổi port mặc định 443 => 8443
# NukeViet bình thường không sao nhưng SSO Server trên HSTDT nếu proxy khác post sẽ gặp vấn đề lớn
#RUN sed -i -e "s/80/8080/g" "/etc/apache2/ports.conf"
#RUN sed -i -e "s/443/8443/g" "/etc/apache2/ports.conf"
#RUN sed -i -e "s/:\s*80/:8080/g" "/etc/apache2/sites-enabled/000-default.conf"
#RUN sed -i -e "s/:\s*443/:8443/g" "/etc/apache2/sites-enabled/default-ssl.conf"

# Set a volume mount point for your code
VOLUME /var/www/html

COPY --chmod=755 ./docker-entrypoint.sh /var/www/tmp/docker-entrypoint.sh
COPY /.env /var/www/tmp/.env
# COPY ./opcache.ini $PHP_INI_DIR/conf.d/

# Ensure the entrypoint file can be run
# RUN chmod +x /var/www/tmp/docker-entrypoint.sh
ENTRYPOINT ["/var/www/tmp/docker-entrypoint.sh"]

# The default apache run command
CMD ["apache2-foreground"]
