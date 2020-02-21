FROM bitnami/minideb:stretch as build_stage

RUN install_packages ca-certificates && \
    mkdir /usr/local/share/ca-certificates/latenightdev && \
    chmod 755 /usr/local/share/ca-certificates/latenightdev

COPY ["certs/root_ca.crt", "/usr/local/share/ca-certificates/latenightdev/Latenightdev_Local_Development_CA.crt"]

RUN chmod 644 /usr/local/share/ca-certificates/latenightdev/Latenightdev_Local_Development_CA.crt && \
    update-ca-certificates

FROM busybox:latest as intermediate_stage

COPY --from=build_stage /usr/local/share/ca-certificates/latenightdev /usr/local/share/ca-certificates/latenightdev
COPY --from=build_stage /etc/ssl/ /etc/ssl/

FROM scratch

COPY --from=intermediate_stage / /

VOLUME /usr/local/share/ca-certificates/
VOLUME /etc/ssl/

CMD [ '/bin/true' ]
