FROM thndocker/ubuntu16-build AS builder

ENV DEBIAN_FRONTEND=noninteractive

RUN groupadd mysql \
    && useradd -r -g mysql -s /bin/false mysql

RUN set -eux; \
    apt-get update; \
    apt-get install -y --no-install-recommends \
    	bzip2 \
	ca-certificates \
	cmake \
	gnupg2 \
	gnupg-curl \
	libaio1 \
	libncurses5-dev \
	libwrap0 \
	openssl \
	perl \
	psmisc \
	wget \
	xz-utils \
	zstd \
    ; \
    rm -rf /var/lib/apt/lists/*

RUN mkdir /tmp/mysql \
    && cd /tmp/mysql \
    && wget https://downloads.mysql.com/archives/get/p/23/file/mysql-5.6.35.tar.gz \
    && tar xvf mysql-5.6.35.tar.gz \
    && cd mysql-5.6.35 \
    && mkdir bld \
    && cd bld \
    && cmake .. -DINSTALL_LAYOUT=RPM -DMYSQL_UNIX_ADDR=/var/run/mysqld/mysqld.sock -DCMAKE_INSTALL_PREFIX=/usr -DMYSQL_USER=mysql -DINSTALL_MYSQLTESTDIR= \
    && make \
    && make DESTDIR=/tmp/usr install




FROM thndocker/ubuntu16

COPY --from=builder /tmp/usr /

RUN groupadd mysql \
    && useradd -r -g mysql -s /bin/false mysql

RUN set -eux; \
    apt-get update; \
    apt-get install -y --no-install-recommends \
    	bzip2 \
	ca-certificates \
	gnupg2 \
	gnupg-curl \
	libaio1 \
	libwrap0 \
	mysql-common \
	openssl \
	perl \		
	psmisc \
	wget \
	xz-utils \
	zstd \
    ; \
    rm -rf /var/lib/apt/lists/*

# add gosu for easy step-down from root
# https://github.com/tianon/gosu/releases
ENV GOSU_VERSION 1.16
RUN set -eux; \
    savedAptMark="$(apt-mark showmanual)"; \
    apt-get update; \
    apt-get install -y --no-install-recommends ca-certificates wget; \
    rm -rf /var/lib/apt/lists/*; \
    dpkgArch="$(dpkg --print-architecture | awk -F- '{ print $NF }')"; \
    wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$dpkgArch"; \
    wget -O /usr/local/bin/gosu.asc "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$dpkgArch.asc"; \
    export GNUPGHOME="$(mktemp -d)"; \
    gpg --batch --keyserver hkps://keys.openpgp.org --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4; \
    gpg --batch --verify /usr/local/bin/gosu.asc /usr/local/bin/gosu; \
    # gpgconf --kill all; \
    gpgconf --reload; \
    rm -rf "$GNUPGHOME" /usr/local/bin/gosu.asc; \
    apt-mark auto '.*' > /dev/null; \
    [ -z "$savedAptMark" ] || apt-mark manual $savedAptMark > /dev/null; \
    apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
    chmod +x /usr/local/bin/gosu; \
    gosu --version; \
    gosu nobody true

RUN mkdir /docker-entrypoint-initdb.d



RUN find /etc/mysql/ -name '*.cnf' -print0 \
    	 | xargs -0 grep -lZE '^(bind-address|log)' \
    	 | xargs -rt -0 sed -Ei 's/^(bind-address|log)/#&/' \
    && echo '[mysqld]\nskip-host-cache\nskip-name-resolve' > /etc/mysql/conf.d/docker.cnf \
    && rm -rf /var/lib/mysql && mkdir -p /var/lib/mysql /var/run/mysqld /var/lib/mysql-files \
    && chown -R mysql:mysql /var/lib/mysql /var/run/mysqld /var/lib/mysql-files\
    && chmod 1777 /var/run/mysqld /var/lib/mysql \
    && chmod 750 /var/lib/mysql-files

VOLUME /var/lib/mysql

COPY docker-entrypoint.sh /usr/local/bin/
RUN ln -s usr/local/bin/docker-entrypoint.sh /entrypoint.sh
ENTRYPOINT ["/usr/local/bin/docker-entrypoint.sh"]

EXPOSE 3306 33060
CMD ["mysqld"]
