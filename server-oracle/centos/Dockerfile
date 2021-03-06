FROM centos:centos8

LABEL org.opencontainers.image.title="Zabbix server (Oracle)" \
      org.opencontainers.image.authors="Alexey Pustovalov <alexey.pustovalov@zabbix.com>, Mati Vapper" \
      org.opencontainers.image.vendor="Zabbix LLC" \
      org.opencontainers.image.url="https://zabbix.com/" \
      org.opencontainers.image.description="Zabbix server with Oracle database support" \
      org.opencontainers.image.licenses="GPL v2.0"

STOPSIGNAL SIGTERM

ENV TINI_VERSION=v0.19.0

RUN set -eux && \
    groupadd -g 1995 --system zabbix && \
    adduser -r --shell /sbin/nologin \
            -g zabbix -G dialout -G root \
            -d /var/lib/zabbix/ -u 1997 \
        zabbix && \
    mkdir -p /etc/zabbix && \
    mkdir -p /var/lib/zabbix && \
    mkdir -p /var/lib/zabbix/enc && \
    mkdir -p /var/lib/zabbix/export && \
    mkdir -p /var/lib/zabbix/mibs && \
    mkdir -p /var/lib/zabbix/modules && \
    mkdir -p /var/lib/zabbix/snmptraps && \
    mkdir -p /var/lib/zabbix/ssh_keys && \
    mkdir -p /var/lib/zabbix/ssl && \
    mkdir -p /var/lib/zabbix/ssl/certs && \
    mkdir -p /var/lib/zabbix/ssl/keys && \
    mkdir -p /var/lib/zabbix/ssl/ssl_ca && \
    mkdir -p /usr/lib/zabbix/alertscripts && \
    mkdir -p /usr/lib/zabbix/externalscripts && \
    mkdir -p /usr/share/doc/zabbix-server-oracle && \
	dnf --quiet makecache && \
    dnf -y install epel-release && \
    dnf -y install --setopt=tsflags=nodocs --setopt=install_weak_deps=False --best \
            fping \
            tzdata \
            iputils \
            traceroute \
            libcurl-minimal \
            libevent \
            libxml2 \
            libssh \
#            mysql \
#            mysql-libs \
            net-snmp-libs \
            OpenIPMI-libs \
            openldap \
            openssl-libs \
            pcre \
            zlib \
            unixODBC && \
    curl -L "https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini" -o /sbin/tini && \
    curl -L "https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini.asc" -o /tmp/tini.asc && \
    export GNUPGHOME="$(mktemp -d)" && \
    for server in $(shuf -e ha.pool.sks-keyservers.net \
                            hkp://p80.pool.sks-keyservers.net:80 \
                            ipv4.pool.sks-keyservers.net \
                            keyserver.ubuntu.com \
                            keyserver.pgp.com \
                            pgp.mit.edu) ; do \
        gpg --keyserver "$server" --recv-keys 595E85A6B1B4779EA4DAAEC70B588DFF0527A9B7 && break || : ; \
    done && \
    gpg --batch --verify /tmp/tini.asc /sbin/tini && \
    rm -rf "$GNUPGHOME" /tmp/tini.asc && \
    chmod +x /sbin/tini && \
    dnf -y clean all && \
    rm -rf /var/cache/yum /var/lib/yum/yumdb/* /usr/lib/udev/hwdb.d/* && \
    rm -rf /var/cache/dnf /etc/udev/hwdb.bin /root/.pki

ARG MAJOR_VERSION=5.0
ARG ZBX_VERSION=${MAJOR_VERSION}.4
ARG ZBX_SOURCES=https://git.zabbix.com/scm/zbx/zabbix.git

ENV TERM=xterm MIBDIRS=/usr/share/snmp/mibs:/var/lib/zabbix/mibs MIBS=+ALL \
    ZBX_VERSION=${ZBX_VERSION} ZBX_SOURCES=${ZBX_SOURCES}

LABEL org.opencontainers.image.documentation="https://www.zabbix.com/documentation/${MAJOR_VERSION}/manual/installation/containers" \
      org.opencontainers.image.version="${ZBX_VERSION}" \
      org.opencontainers.image.source="${ZBX_SOURCES}"

RUN set -eux && \
    sed -i 's/enabled=0/enabled=1/g' /etc/yum.repos.d/CentOS-PowerTools.repo && \
    dnf --quiet makecache && \
    dnf -y install --setopt=tsflags=nodocs --setopt=install_weak_deps=False --best \
            autoconf \
            automake \
            gcc \
            pcre-devel \
            libcurl-devel \
            libevent-devel \
            libssh-devel \
            libxml2-devel \
            make \
#            mysql-devel \
            net-snmp-devel \
            OpenIPMI-devel \
            openldap-devel \
            git \
            unixODBC-devel \
			python3 && \
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
            oracle-instantclient19.8-devel.x86_64 \
			oracle-instantclient19.8-sqlplus.x86_64 \
            libnsl && \
    sed -i 's/enabled=1/enabled=0/g' /etc/yum.repos.d/*oracle* && \
    sed -i 's/enabled=1/enabled=0/g' /etc/yum.repos.d/*ol7* && \
	pip3 install py-zabbix requests pyyaml anymarkup && \
    cd /tmp/ && \
    git clone ${ZBX_SOURCES} --branch ${ZBX_VERSION} --depth 1 --single-branch zabbix-${ZBX_VERSION} && \
    cd /tmp/zabbix-${ZBX_VERSION} && \
	sed -i "s/nvarchar/varchar/g" src/libs/zbxdbupgrade/dbupgrade.c && \
    zabbix_revision=`git rev-parse --short HEAD` && \
    sed -i "s/{ZABBIX_REVISION}/$zabbix_revision/g" include/version.h && \
    ./bootstrap.sh && \
    export CFLAGS="-fPIC -pie -Wl,-z,relro -Wl,-z,now" && \
    export INCLUDE_DIR=/usr/include/oracle/19.8/client64 && \
    export LD_LIBRARY_PATH=/usr/lib/oracle/19.8/client64/lib  && \
    ./configure \
            --datadir=/usr/lib \
            --libdir=/usr/lib/zabbix \
            --prefix=/usr \
            --sysconfdir=/etc/zabbix \
            --enable-agent \
            --enable-server \
#            --with-mysql \
            --with-oracle=yes \
            --with-oracle-lib=$LD_LIBRARY_PATH \
            --with-oracle-include=$INCLUDE_DIR \
            --with-ldap \
            --with-libcurl \
            --with-libxml2 \
            --with-net-snmp \
            --with-openipmi \
            --with-openssl \
            --with-ssh \
            --with-unixodbc \
            --enable-ipv6 \
            --silent && \
    make -j"$(nproc)" -s dbschema && \
    make -j"$(nproc)" -s && \
    cp src/zabbix_server/zabbix_server /usr/sbin/zabbix_server && \
    cp src/zabbix_get/zabbix_get /usr/bin/zabbix_get && \
    cp src/zabbix_sender/zabbix_sender /usr/bin/zabbix_sender && \
    cp conf/zabbix_server.conf /etc/zabbix/zabbix_server.conf && \
	sed -i "s/nvarchar/varchar/g" database/oracle/*.sql && \
    cat database/oracle/schema.sql > database/oracle/create.sql && \
    cat database/oracle/images.sql >> database/oracle/create.sql && \
    cat database/oracle/data.sql >> database/oracle/create.sql && \
    gzip database/oracle/create.sql && \
    cp database/oracle/create.sql.gz /usr/share/doc/zabbix-server-oracle/ && \
    cd /tmp/ && \
    rm -rf /tmp/zabbix-${ZBX_VERSION}/ && \
    chown --quiet -R zabbix:root /etc/zabbix/ /var/lib/zabbix/ && \
    chgrp -R 0 /etc/zabbix/ /var/lib/zabbix/ && \
    chmod -R g=u /etc/zabbix/ /var/lib/zabbix/ && \
#    dnf -y history undo `dnf -q history | sed -n 3p |column -t | cut -d' ' -f1` && \
    dnf -y clean all && \
    sed -i 's/enabled=1/enabled=0/g' /etc/yum.repos.d/CentOS-PowerTools.repo && \
    rm -rf /var/cache/yum /var/lib/yum/yumdb/* /usr/lib/udev/hwdb.d/* && \
    rm -rf /var/cache/dnf /etc/udev/hwdb.bin /root/.pki

EXPOSE 10051/TCP

WORKDIR /var/lib/zabbix

VOLUME ["/var/lib/zabbix/snmptraps", "/var/lib/zabbix/export"]

COPY ["docker-entrypoint.sh", "/usr/bin/"]
COPY ["docker-entrypoint-ora.sh", "/usr/bin/"]

ENTRYPOINT ["/sbin/tini", "--", "/usr/bin/docker-entrypoint-ora.sh"]

USER 1997

CMD ["/usr/sbin/zabbix_server", "--foreground", "-c", "/etc/zabbix/zabbix_server.conf"]
