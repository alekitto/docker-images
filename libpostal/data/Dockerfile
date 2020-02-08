FROM bitnami/minideb:stretch as build_stage

RUN install_packages curl ca-certificates make build-essential git-core \
    autoconf automake libtool pkg-config libcurl4-gnutls-dev lsb-core zlib1g-dev

RUN git clone --branch v1.1-alpha https://github.com/openvenues/libpostal.git /usr/local/src/libpostal && \
    cd /usr/local/src/libpostal && \
    ./bootstrap.sh && ./configure && \
    make -j4 && make install && ldconfig

FROM scratch

COPY --from=build_stage /usr/local/share/libpostal /usr/local/share/libpostal
VOLUME /usr/local/share/libpostal
