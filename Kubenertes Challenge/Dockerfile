# base image
FROM php:7.4-apache

# Install mysqli extension for PHP
RUN docker-php-ext-install mysqli

# Copy application source code to /var/www/html/
COPY learning-app-ecommerce/* /var/www/html/

# Update database connection strings to point to a Kubernetes service named mysql-service unsing ENV vvariable
ENV DB_HOST="mysql-service"


# Expose port 80
EXPOSE 80/tcp

# Start Apache in the foreground
CMD ["apache2-foreground"]