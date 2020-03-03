FROM postgres:11 as build_stage

RUN apt-get update && \
    apt-get install --no-install-recommends -y curl ca-certificates make build-essential git-core autoconf automake libtool pkg-config libcurl4-gnutls-dev lsb-core zlib1g-dev && \
    apt-get install --no-install-recommends -y postgis postgresql-11-postgis-2.5 postgresql-11-postgis-2.5-scripts postgresql-11-ip4r postgresql-plpython3-11 && \
    apt-get install --no-install-recommends -y postgresql-server-dev-11
RUN git clone git://sigaev.ru/smlar.git /usr/local/src/smlar && \
    cd /usr/local/src/smlar && \
    make USE_PGXS=1 && make USE_PGXS=1 install
RUN git clone --branch v1.1-alpha https://github.com/openvenues/libpostal.git /usr/local/src/libpostal && \
    cd /usr/local/src/libpostal && \
    ./bootstrap.sh && ./configure && \
    make -j4 && make install && ldconfig
RUN git clone https://github.com/pramsey/pgsql-postal.git /usr/local/src/pgsql-postal && \
    cd /usr/local/src/pgsql-postal && \
    make -j4 && make install
RUN git clone --branch develop https://github.com/zombodb/zombodb.git /usr/local/src/zombodb && \
    cd /usr/local/src/zombodb && \
    make -j4 && make install
RUN git clone https://github.com/gavinwahl/postgres-json-schema.git /usr/local/src/postgres-json-schema && \
    cd /usr/local/src/postgres-json-schema && \
    make install

FROM bitnami/minideb:stretch as intermidiate_stage

# explicitly set user/group IDs
RUN set -eux; \
	groupadd -r postgres --gid=999; \
    # https://salsa.debian.org/postgresql/postgresql-common/blob/997d842ee744687d99a2b2d95c1083a2615c79e8/debian/postgresql-common.postinst#L32-35
	useradd -r -g postgres --uid=999 --home-dir=/var/lib/postgresql --shell=/bin/bash postgres; \
    # also create the postgres user's home directory with appropriate permissions
    # see https://github.com/docker-library/postgres/issues/274
	mkdir -p /var/lib/postgresql; \
	chown -R postgres:postgres /var/lib/postgresql

# grab gosu for easy step-down from root
ENV GOSU_VERSION 1.11
RUN set -x \
	&& install_packages -y --no-install-recommends gnupg2 dirmngr ca-certificates wget \
	&& wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$(dpkg --print-architecture)" \
	&& wget -O /usr/local/bin/gosu.asc "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$(dpkg --print-architecture).asc" \
	&& export GNUPGHOME="$(mktemp -d)" \
	&& gpg --batch --keyserver hkps://keys.openpgp.org --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4 \
	&& gpg --batch --verify /usr/local/bin/gosu.asc /usr/local/bin/gosu \
	&& { command -v gpgconf > /dev/null && gpgconf --kill all || :; } \
	&& rm -rf "$GNUPGHOME" /usr/local/bin/gosu.asc \
	&& chmod +x /usr/local/bin/gosu \
	&& gosu nobody true \
	&& apt-get purge -y --auto-remove ca-certificates wget

# make the "en_US.UTF-8" locale so postgres will be utf-8 enabled by default
RUN set -eux \
	&& install_packages locales \
	&& localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8

RUN set -eux \
    && install_packages postgresql-common libpq5 libxml2 libedit2 libnghttp2-14 \
       libcurl3-gnutls libicu57

RUN set -eux \
    && mkdir /docker-entrypoint-initdb.d \
    && touch /docker-entrypoint-initdb.d/.exists

COPY --from=build_stage /usr/lib/postgresql /usr/lib/postgresql
COPY --from=build_stage /etc/apt /etc/apt
COPY --from=build_stage /usr/share/postgresql /usr/share/postgresql
COPY --from=build_stage /usr/local/bin/docker-entrypoint.sh /usr/local/bin/docker-entrypoint.sh
COPY --from=build_stage /usr/local/bin/libpostal_data /usr/local/bin/libpostal_data
COPY --from=build_stage /usr/local/lib/libpostal.so.1 /usr/local/lib/libpostal.so.1
COPY --from=build_stage /usr/lib/x86_64-linux-gnu/libpq.* /usr/lib/x86_64-linux-gnu/
COPY --from=build_stage /usr/lib/postgresql/11/lib/plpython3.so /usr/lib/postgresql/11/lib/plpython3.so

FROM scratch

ENV PG_MAJOR=11 PG_VERSION=11.6-1.pgdg90+1 PATH=$PATH:/usr/lib/postgresql/11/bin \
    PGDATA=/var/lib/postgresql/data

COPY --from=intermidiate_stage / /
RUN mkdir -p /var/run/postgresql && chown -R postgres:postgres /var/run/postgresql && chmod 2777 /var/run/postgresql \
    && mkdir -p "$PGDATA" && chown -R postgres:postgres "$PGDATA" && chmod 777 "$PGDATA"

VOLUME /var/lib/postgresql/data /usr/local/share/libpostal

ENTRYPOINT ["/usr/local/bin/docker-entrypoint.sh"]

EXPOSE 5432
CMD ["postgres"]
