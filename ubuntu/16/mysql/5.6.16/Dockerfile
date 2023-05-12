FROM thndocker/ubuntu16 AS builder

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update \
    && apt-get install -y mysql-common build-essential software-properties-common

RUN add-apt-repository 'deb http://archive.ubuntu.com/ubuntu trusty universe' \
    && sed -i 's/^# deb-src/deb-src/' /etc/apt/sources.list \
    && sed -i 's/^deb http:\/\/archive.ubuntu.com\/ubuntu trusty universe/#deb http:\/\/archive.ubuntu.com\/ubuntu trusty universe/' /etc/apt/sources.list \
    && apt-get update

RUN mkdir /tmp/mysql \
    && chmod 777 /tmp/mysql \
    && cd /tmp/mysql \
    && apt-get source mysql-server-5.6 \
    && apt-get build-dep -y mysql-server-5.6 \
    && cd mysql-5.6-5.6.16 \
    && dpkg-buildpackage -b -uc

RUN rm -rf /var/lib/apt/lists/*



FROM thndocker/ubuntu16

RUN mkdir /tmp/mysql
COPY --from=builder /tmp/mysql/*.deb /tmp/mysql


# add gosu for easy step-down from root
# https://github.com/tianon/gosu/releases
ENV GOSU_VERSION 1.16
RUN set -eux; \
    savedAptMark="$(apt-mark showmanual)"; \
    apt-get update; \
    apt-get install -y --no-install-recommends gnupg2 gnupg-curl ca-certificates wget; \
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

RUN set -eux; \
    apt-get update; \
    apt-get install -y --no-install-recommends \
    	bzip2 \
	openssl \
# FATAL ERROR: please install the following Perl modules before executing /usr/local/mysql/scripts/mysql_install_db:
# File::Basename
# File::Copy
# Sys::Hostname
# Data::Dumper
	perl \
	xz-utils \
	zstd \
    ; \
    rm -rf /var/lib/apt/lists/*



# the "/var/lib/mysql" stuff here is because the mysql-server postinst doesn't have an explicit way to disable the mysql_install_db codepath besides having a database already "configured" (ie, stuff in /var/lib/mysql/mysql)
# also, we set debconf keys to make APT a little quieter
RUN { \
		echo mysql-community-server mysql-community-server/data-dir select ''; \
		echo mysql-community-server mysql-community-server/root-pass password ''; \
		echo mysql-community-server mysql-community-server/re-root-pass password ''; \
		echo mysql-community-server mysql-community-server/remove-test-db select false; \
	} | debconf-set-selections \
	&& apt-get update \
	&& apt-get install -y mysql-common libdbi-perl libaio1 libwrap0 libdbd-mysql-perl libterm-readkey-perl psmisc \
	&& dpkg -i /tmp/mysql/mysql-server-core-5.6_5.6.16-1~exp1_arm64.deb \
	&& dpkg -i /tmp/mysql/mysql-client-core-5.6_5.6.16-1~exp1_arm64.deb \
	&& dpkg -i /tmp/mysql/mysql-client-5.6_5.6.16-1~exp1_arm64.deb \
	&& dpkg -i /tmp/mysql/mysql-server-5.6_5.6.16-1~exp1_arm64.deb \
# comment out a few problematic configuration values
	&& find /etc/mysql/ -name '*.cnf' -print0 \
		| xargs -0 grep -lZE '^(bind-address|log)' \
		| xargs -rt -0 sed -Ei 's/^(bind-address|log)/#&/' \
# don't reverse lookup hostnames, they are usually another container
	&& echo '[mysqld]\nskip-host-cache\nskip-name-resolve' > /etc/mysql/conf.d/docker.cnf \
	&& rm -rf /var/lib/apt/lists/* \
	&& rm -rf /var/lib/mysql && mkdir -p /var/lib/mysql /var/run/mysqld \
	&& chown -R mysql:mysql /var/lib/mysql /var/run/mysqld \
# ensure that /var/run/mysqld (used for socket and lock files) is writable regardless of the UID our mysqld instance ends up having at runtime
	&& chmod 1777 /var/run/mysqld /var/lib/mysql \
	&& cp /usr/share/doc/mysql-server-5.6/examples/my-default.cnf /usr/share/mysql

VOLUME /var/lib/mysql

COPY docker-entrypoint.sh /usr/local/bin/
RUN ln -s usr/local/bin/docker-entrypoint.sh /entrypoint.sh # backwards compat
ENTRYPOINT ["/usr/local/bin/docker-entrypoint.sh"]

EXPOSE 3306 33060
CMD ["mysqld"]


# RUN apt-get update \
#     && apt-get install -y mysql-common \
#     && dpkg -i mysql-server-5.6_5.6.16-1~exp1_arm64.deb \
#     && rm -rf /var/lib/apt/lists/*
#     && cd / \
#     && rm -rf /tmp/mysql

# WORKDIR /

# CMD ["mysqld"]
