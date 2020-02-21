FROM bitnami/minideb:stretch as build_stage

RUN install_packages ca-certificates
COPY ["certs/root_ca.crt", "/etc/ssl/Latenightdev Local Development CA.pem"]

FROM busybox:latest as intermediate_stage

COPY --from=build_stage /etc/ssl/ /etc/ssl/

FROM scratch

COPY --from=intermediate_stage / /
VOLUME /etc/ssl/

CMD [ '/bin/sh' ]
