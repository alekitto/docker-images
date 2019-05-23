FROM postgres:11

RUN apt-get update && \
    apt-get install -y curl ca-certificates git-core make build-essential && \
    curl https://www.postgresql.org/media/keys/ACCC4CF8.asc | apt-key add - && \
    echo "deb http://apt.postgresql.org/pub/repos/apt/ $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list && \
    apt-get update && \
    apt-get install --no-install-recommends -y postgis postgresql-11-postgis-2.5 postgresql-11-postgis-2.5-scripts postgresql-11-ip4r && \
    apt-get install --no-install-recommends -y postgresql-server-dev-11 && \
    rm -rf /var/lib/apt/lists/*

RUN git clone git://sigaev.ru/smlar.git /usr/local/src/smlar && \
    cd /usr/local/src/smlar && \
    make USE_PGXS=1 && make USE_PGXS=1 install
