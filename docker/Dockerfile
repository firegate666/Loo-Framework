FROM        ubuntu:14.04
MAINTAINER  Daniel Weiss "mail@weiss-daniel.de"
WORKDIR     /var/www/loo
ENV HOME    /root
EXPOSE      22 9000

# Locale
RUN locale-gen en_US.UTF-8 && \
    echo 'LANG="en_US.UTF-8"' > /etc/default/locale
RUN dpkg-reconfigure locales

# Install deps
RUN apt-get update && apt-get -y install vim software-properties-common python-software-properties curl git make htop fish supervisor

USER root

# Install PHP5 and modules along with composer binary
RUN add-apt-repository ppa:ondrej/php5-5.6
RUN apt-get update
RUN apt-get -y --force-yes install php5-fpm php5-pgsql php-apc php5-mcrypt php5-curl php5-gd php5-json php5-cli libssh2-php php5-xdebug
RUN sed -i -e "s/post_max_size = 8M/post_max_size = 20M/g" /etc/php5/fpm/php.ini
RUN sed -i -e "s/upload_max_filesize = 2M/upload_max_filesize = 20M/g" /etc/php5/fpm/php.ini
RUN echo "xdebug.remote_enable=on" >> /etc/php5/mods-available/xdebug.ini
RUN echo "xdebug.remote_port=9000" >> /etc/php5/mods-available/xdebug.ini
RUN echo "xdebug.remote_connect_back=On" >> /etc/php5/mods-available/xdebug.ini
RUN echo "xdebug.remote_handler=dbgp" >> /etc/php5/mods-available/xdebug.ini
RUN echo "xdebug.profiler_enable=0" >> /etc/php5/mods-available/xdebug.ini
RUN curl -sS https://getcomposer.org/installer | php
RUN mv composer.phar /usr/local/bin/composer

# Install openssh-server
RUN apt-get install -y openssh-server
RUN mkdir /var/run/sshd
RUN echo 'root:docker' | chpasswd
RUN sed -i 's/PermitRootLogin without-password/PermitRootLogin yes/' /etc/ssh/sshd_config

# SSH login fix. Otherwise user is kicked off after login
RUN sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd
ENV NOTVISIBLE "in users profile"
RUN echo "export VISIBLE=now" >> /etc/profile

# Configure nginx for PHP websites
RUN echo "cgi.fix_pathinfo = 0;" >> /etc/php5/fpm/php.ini
RUN echo "max_input_vars = 10000;" >> /etc/php5/fpm/php.ini
RUN echo "date.timezone = Europe/Berlin;" >> /etc/php5/fpm/php.ini

# Setup supervisor
ADD supervisor/php.conf /etc/supervisor/conf.d/
ADD supervisor/sshd.conf /etc/supervisor/conf.d/

# Disallow key checking
RUN echo "    StrictHostKeyChecking no" >> /etc/ssh/ssh_config

# Bash
ADD bash/bash_aliases /root/.bash_aliases

VOLUME /var/www/loo

CMD ["/usr/bin/supervisord", "-n"]
