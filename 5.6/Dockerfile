FROM debian:stretch-slim

#
# Step 1: Installation
#

# Set frontend. We'll clean this later on!
ENV DEBIAN_FRONTEND noninteractive

# Locale
ENV LOCALE pt_BR.UTF-8

# PHP Timezone
ENV TZ=America/Maceio

# PHP Default Charset
ENV DEFAULT_CHARSET=UTF-8

# Set repositories
RUN \
  echo "deb http://ftp.br.debian.org/debian/ stretch main contrib non-free" > /etc/apt/sources.list && \
  echo "deb-src http://ftp.br.debian.org/debian/ stretch main contrib non-free" >> /etc/apt/sources.list && \
  echo "deb http://security.debian.org/debian-security stretch/updates main contrib non-free" >> /etc/apt/sources.list && \
  echo "deb-src http://security.debian.org/debian-security stretch/updates main contrib non-free" >> /etc/apt/sources.list && \
  apt-get -qq update --fix-missing && apt-get -qqy upgrade

# Install some basic tools needed for deployment
RUN apt-get -yqq install \
  apt-utils \
  build-essential \
  debconf-utils \
  debconf \
  mysql-client \
  locales \
  curl \
  wget \
  unzip \
  patch \
  rsync \
  vim \
  openssh-client \
  git \
  bash-completion \
  locales \
  libjpeg-turbo-progs libjpeg-progs \
  pngcrush optipng \
  libaio1

# Install locale
RUN \
  sed -i -e "s/# $LOCALE/$LOCALE/" /etc/locale.gen && \
  echo "LANG=$LOCALE">/etc/default/locale && \
  dpkg-reconfigure --frontend=noninteractive locales && \
  update-locale LANG=$LOCALE

# Configure Sury sources
# @see https://www.noobunbox.net/serveur/auto-hebergement/installer-php-7-1-sous-debian-et-ubuntu
RUN \
  apt-get -yqq install apt-transport-https lsb-release ca-certificates && \
  wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg && \
  echo "deb https://packages.sury.org/php/ stretch main" > /etc/apt/sources.list.d/php.list && \
  apt-get -qq update && apt-get -qqy upgrade

# Install PHP5.6 with Xdebug (dev environment). WARNING with install php5.6-apcu.
RUN apt-get -yqq install \
  php5.6 		\
  php5.6-bcmath     \
  php5.6-bz2        \
  php5.6-curl 		\
  php5.6-dev 		\
  php5.6-dba        \
  php5.6-dom		\
  php5.6-enchant    \
  php5.6-fpm        \
  php5.6-gd 		\
  php5.6-gmp        \
  php5.6-imap       \
  php5.6-interbase  \
  php5.6-imagick    \
  php5.6-intl 		\
  php5.6-json 		\
  php5.6-ldap       \
  php5.6-mbstring	\
  php5.6-mcrypt 	\
  php5.6-memcache 	\
  php5.6-memcached 	\
  php5.6-mysql		\
  php5.6-oauth      \
  php5.6-odbc       \
  php5.6-opcache	\
  php5.6-pgsql      \
  php5.6-pspell     \
  php5.6-readline   \
  php5.6-recode     \
  php5.6-redis		\
  php5.6-snmp       \
  php5.6-soap       \
  php5.6-sqlite3    \
  php5.6-ssh2		\
  php5.6-sybase     \
  php5.6-tidy       \
  php5.6-uploadprogress \
  php5.6-xdebug		\
  php5.6-xml		\
  php5.6-xmlrpc     \
  php5.6-xsl        \
  php5.6-zip		\
  libapache2-mod-php5.6 && \
  apt-get -yqq install php5.6-apcu --no-install-recommends

# PHP Timezone
RUN \
  echo $TZ | tee /etc/timezone && \
  dpkg-reconfigure --frontend noninteractive tzdata && \
  echo "date.timezone = \"$TZ\";" > /etc/php/5.6/apache2/conf.d/timezone.ini && \
  echo "date.timezone = \"$TZ\";" > /etc/php/5.6/cli/conf.d/timezone.ini

RUN sed -i -e "s/short_open_tag = Off/short_open_tag = On/" /etc/php/5.6/apache2/php.ini && \
    sed -i -e "s/default_charset = \"UTF-8\"/default_charset = $DEFAULT_CHARSET/" /etc/php/5.6/apache2/php.ini

# Install manually xhprof
RUN \
  pecl install xhprof-beta && \
  echo "extension=xhprof.so" > /etc/php/5.6/mods-available/xhprof.ini && \
  echo "xhprof.output_dir=/var/www/xhprof" >> /etc/php/5.6/mods-available/xhprof.ini

# Install manually APC
RUN \
  echo "extension=apcu.so" > /etc/php/5.6/mods-available/apcu_bc.ini && \
  echo "extension=apc.so" >> /etc/php/5.6/mods-available/apcu_bc.ini

# Install SMTP.
RUN apt-get install -y ssmtp && \
  echo "FromLineOverride=YES" >> /etc/ssmtp/ssmtp.conf

# Install Oracle
ADD oracle/oracle-instantclient12.2-basic_12.2.0.1.0-2_amd64.deb /tmp
ADD oracle/oracle-instantclient12.2-devel_12.2.0.1.0-2_amd64.deb /tmp
RUN dpkg -i /tmp/oracle-instantclient12.2-basic_12.2.0.1.0-2_amd64.deb
RUN dpkg -i /tmp/oracle-instantclient12.2-devel_12.2.0.1.0-2_amd64.deb

ENV LD_LIBRARY_PATH /usr/lib/oracle/12.2/client64/lib/
ENV ORACLE_HOME /usr/lib/oracle/12.2/client64
RUN export LD_LIBRARY_PATH=$LD_LIBRARY_PATH
RUN export ORACLE_HOME=$ORACLE_HOME
RUN echo $ORACLE_HOME 

ADD oracle/oci8-2.0.12.zip /tmp

RUN echo $LD_LIBRARY_PATH && echo $ORACLE_HOME && \
  ln -s /usr/include/oracle/12.2/client64 $ORACLE_HOME/include && \
  cd /tmp && unzip oci8-2.0.12.zip -d /usr/lib/oracle/src && \
  cd /usr/lib/oracle/src/oci8-2.0.12 && \
  phpize && \
  ./configure --with-oci8=instantclient,/usr/lib/oracle/12.2/client64/lib/ && \
  make && make install && \
  echo "extension=oci8.so" > /etc/php/5.6/mods-available/oci8.ini && \
  ln -s /etc/php/5.6/mods-available/oci8.ini /etc/php/5.6/apache2/conf.d/oci8.ini

ADD oracle/php-5.6.38.zip /tmp

RUN echo $LD_LIBRARY_PATH && echo $ORACLE_HOME && \
    ln -s /usr/include/oracle/12.2/client64 $ORACLE_HOME/include && \
    cd /tmp && unzip php-5.6.38.zip -d /usr/lib/oracle/src && \
    cd /usr/lib/oracle/src/php-5.6.38/ext/pdo_oci/ && \
    phpize && \
#    ./configure --with-pdo-oci=instantclient,$LD_LIBRARY_PATH,12.2 && \
    ./configure --with-pdo-oci=instantclient,/usr,12.2 && \
    make && make install && \
    echo "extension=pdo_oci.so" > /etc/php/5.6/mods-available/pdo_oci.ini && \
ln -s /etc/php/5.6/mods-available/pdo_oci.ini /etc/php/5.6/apache2/conf.d/pdo_oci.ini

# Install Apache web server.
RUN apt-get -yqq install apache2

#
# Step 2: Configuration
#

# Enable uploadprogress, imagick, redis and solr.
RUN phpenmod uploadprogress imagick redis solr

# Disable by default apcu, apcu_bc, opcache, xdebug and xhprof. Use docker-compose.yml to add file.
RUN phpdismod apcu apcu_bc opcache xdebug xhprof

# Remove all sites enabled
RUN rm /etc/apache2/sites-enabled/*

# Configure needed apache modules and disable default site
RUN a2dismod   mpm_event  cgi # mpm_worker enabled.
RUN a2enmod		\
  access_compat		\
  actions		\
  alias			\
  auth_basic		\
  authn_core		\
  authn_file		\
  authz_core		\
  authz_groupfile	\
  authz_host 		\
  authz_user		\
  autoindex		\
  dir			\
  env 			\
  expires 		\
  filter 		\
  headers		\
  mime 			\
  negotiation 		\
  php5.6 		\
  mpm_prefork 		\
  reqtimeout 		\
  rewrite 		\
  setenvif 		\
  status 		\
  ssl

RUN sed -i 's/AllowOverride\ None/AllowOverride\ All/g' /etc/apache2/apache2.conf

# Install composer (latest version) | prestissimo to speed up composer
RUN curl -sS https://getcomposer.org/installer | php && \
  mv composer.phar /usr/local/bin/composer && \
  composer global require "hirak/prestissimo:^0.3"

### Install DRUSH (latest stable) ###
# Run this in your terminal to get the latest DRUSH project version:
RUN composer global require drush/drush && \
  ~/.composer/vendor/bin/drush init

### Install DRUPAL CONSOLE (latest version) ###
# Run this in your terminal to get the latest project version:
RUN curl https://drupalconsole.com/installer -L -o drupal.phar && \
  mv drupal.phar /usr/local/bin/drupal && \
  chmod +x /usr/local/bin/drupal  && \
  drupal self-update

### Install PHP CodeSniffer ###
RUN curl -OL https://squizlabs.github.io/PHP_CodeSniffer/phpcs.phar && \
  curl -OL https://squizlabs.github.io/PHP_CodeSniffer/phpcbf.phar && \
  mv phpcs.phar /usr/local/bin/phpcs && \
  mv phpcbf.phar /usr/local/bin/phpcbf && \
  chmod +x /usr/local/bin/phpcs && \
  chmod +x /usr/local/bin/phpcbf

# Set some useful defaults to phpcs
# show_progress - I like to see a progress while phpcs does its magic
# colors - Enable colors; My terminal supports more than black and white
# report_width - I am using a large display so I can afford a larger width
# encoding - Unicode all the way

RUN /usr/local/bin/phpcs --config-set show_progress 1 && \
    /usr/local/bin/phpcs --config-set colors 1 && \
    /usr/local/bin/phpcs --config-set report_width 140 && \
    /usr/local/bin/phpcs --config-set encoding utf-8

# Bash setup.
RUN echo ". /usr/share/bash-completion/bash_completion" >> ~/.bashrc && echo "alias ll='ls -lahs'" >> ~/.bashrc

#
# Step 3: Clean the system
#

# Cleanup some things.
RUN apt-get -q autoclean && \
  rm -rf /var/lib/apt/lists/* && \
  rm -rf /tmp/*

#
# Step 4: Run
#

# Create 'me' user like local machime user.
RUN useradd helder && usermod -G www-data -a helder

# Working dir
WORKDIR /var/www/html

# Volume for Apache2 data
VOLUME /var/www/html

COPY scripts/apache2-foreground /usr/bin/
RUN chmod +x /usr/bin/apache2-foreground

EXPOSE 80 443

CMD ["apache2-foreground"]
