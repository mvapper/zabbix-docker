FROM centos:centos8

LABEL org.opencontainers.image.title="Zabbix web-interface (Nginx, Oracle)" \
      org.opencontainers.image.authors="Alexey Pustovalov <alexey.pustovalov@zabbix.com>, Mati Vapper" \
      org.opencontainers.image.vendor="Zabbix LLC" \
      org.opencontainers.image.url="https://zabbix.com/" \
      org.opencontainers.image.description="Zabbix web-interface based on Nginx web server with Oracle database support" \
      org.opencontainers.image.licenses="GPL v2.0"

STOPSIGNAL SIGTERM

RUN set -eux && \
    groupadd -g 1995 --system zabbix && \
    adduser -r --shell /sbin/nologin \
            -g zabbix -G root \
            -d /var/lib/zabbix/ -u 1997 \
        zabbix && \
    mkdir -p /etc/zabbix && \
    mkdir -p /etc/zabbix/web && \
    mkdir -p /etc/zabbix/web/certs && \
    mkdir -p /var/lib/php/session && \
    dnf --quiet makecache && \
    dnf -y install epel-release && \
    dnf -y install --setopt=install_weak_deps=False --best --setopt=tsflags=nodocs \
            curl \
            mysql \
            nginx \
            php-bcmath \
            php-fpm \
            php-gd \
            php-ldap \
            php-mbstring \
            php-mysqlnd \
            php-json \
            php-xml \
            supervisor \
			libnsl \
			php-pear \
			php-devel \ 
			make && \
    rm -f /etc/nginx/conf.d/*.conf  && \
    rm -f /etc/php-fpm.d/www.conf && \
    dnf -y clean all && \
    rm -rf /var/cache/yum /var/lib/yum/yumdb/* /usr/lib/udev/hwdb.d/* && \
    rm -rf /var/cache/dnf /etc/udev/hwdb.bin /root/.pki && \
echo $'[ol7_latest] \n\
name=Oracle Linux $releasever Latest ($basearch) \n\
baseurl=https://yum.oracle.com/repo/OracleLinux/OL7/latest/$basearch/ \n\
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-oracle \n\
gpgcheck=1 \n\
enabled=1' >/etc/yum.repos.d/oracle.repo && \
        curl https://yum.oracle.com/RPM-GPG-KEY-oracle-ol6 -o /etc/pki/rpm-gpg/RPM-GPG-KEY-oracle && \
    dnf -y install oracle-release-el7 && \
    dnf -y install --setopt=tsflags=nodocs --setopt=install_weak_deps=False --best \
            oracle-instantclient19.8-basic.x86_64 \
            oracle-instantclient19.8-devel.x86_64 && \
	export C_INCLUDE_PATH=/usr/include/oracle/19.8/client64 && \
    echo 'instantclient,/usr/lib/oracle/19.8/client64/lib' | pecl install oci8 && \
    echo "extension=oci8.so" >> /etc/php.ini
ARG MAJOR_VERSION=5.0
ARG ZBX_VERSION=${MAJOR_VERSION}.4
ARG ZBX_SOURCES=https://git.zabbix.com/scm/zbx/zabbix.git

ENV TERM=xterm ZBX_VERSION=${ZBX_VERSION} ZBX_SOURCES=${ZBX_SOURCES}

LABEL org.opencontainers.image.documentation="https://www.zabbix.com/documentation/${MAJOR_VERSION}/manual/installation/containers" \
      org.opencontainers.image.version="${ZBX_VERSION}" \
      org.opencontainers.image.source="${ZBX_SOURCES}"

COPY ["conf/etc/", "/etc/"]

RUN set -eux && \
    dnf --quiet makecache && \
    dnf -y install --setopt=install_weak_deps=False --best --setopt=tsflags=nodocs \
            glibc-locale-source && \
    dnf -y install --setopt=install_weak_deps=False --best --setopt=tsflags=nodocs \
            gettext \
            git && \
    cd /usr/share/ && \
    git clone ${ZBX_SOURCES} --branch ${ZBX_VERSION} --depth 1 --single-branch zabbix-${ZBX_VERSION} && \
    mkdir /usr/share/zabbix/ && \
    cp -R /usr/share/zabbix-${ZBX_VERSION}/ui/* /usr/share/zabbix/ && \
    rm -rf /usr/share/zabbix-${ZBX_VERSION}/ && \
    cd /usr/share/zabbix/ && \
    rm -f conf/zabbix.conf.php && \
    rm -f conf/maintenance.inc.php && \
    rm -rf tests && \
    ./locale/make_mo.sh && \
    ln -s "/etc/zabbix/web/zabbix.conf.php" "/usr/share/zabbix/conf/zabbix.conf.php" && \
    ln -s "/etc/zabbix/web/maintenance.inc.php" "/usr/share/zabbix/conf/maintenance.inc.php" && \
    cat /usr/share/zabbix/include/locales.inc.php | grep display | grep true | awk '{$1=$1};1' | \
                cut -d"'" -f 2 | sort | \
                xargs -I '{}' bash -c 'echo "{}" && localedef -c -i {} -f UTF-8 {}.UTF-8 2>/dev/null' && \
    chown --quiet -R zabbix:root /etc/zabbix/ /usr/share/zabbix/modules/ && \
    chgrp -R 0 /etc/zabbix/ /usr/share/zabbix/modules/ && \
    chmod -R g=u /etc/zabbix/ /usr/share/zabbix/modules/ && \
    chown --quiet -R zabbix:root /etc/nginx/ /etc/php-fpm.d/ /etc/php-fpm.conf && \
    chgrp -R 0 /etc/nginx/ /etc/php-fpm.d/ /etc/php-fpm.conf && \
    chmod -R g=u /etc/nginx/ /etc/php-fpm.d/ /etc/php-fpm.conf && \
    chown --quiet -R zabbix:root /var/lib/php/session/ && \
    chgrp -R 0 /var/lib/php/session/ && \
    chmod -R g=u /var/lib/php/session/ && \
    chown --quiet -R zabbix:root /usr/share/zabbix/include/defines.inc.php && \
    chgrp -R 0 /usr/share/zabbix/include/defines.inc.php && \
    chmod -R g=u /usr/share/zabbix/include/defines.inc.php && \
#    dnf -y history undo `dnf -q history | sed -n 3p |column -t | cut -d' ' -f1` && \
    dnf -y erase glibc-locale-source glibc-langpack-en && \
    dnf -y clean all && \
    rm -rf /var/cache/yum /var/lib/yum/yumdb/* /usr/lib/udev/hwdb.d/* && \
    rm -rf /var/cache/dnf /etc/udev/hwdb.bin /root/.pki

EXPOSE 8080/TCP 8443/TCP

WORKDIR /usr/share/zabbix

COPY ["docker-entrypoint.sh", "/usr/bin/"]

USER 1997

ENTRYPOINT ["docker-entrypoint.sh"]
