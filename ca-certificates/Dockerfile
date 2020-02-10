FROM bitnami/minideb:stretch as build_stage

RUN install_packages ca-certificates
COPY ["certs/root_ca.crt", "/etc/ssl/Latenightdev Local Development CA.pem"]

FROM scratch

COPY --from=build_stage /etc/ssl/ /etc/ssl/
VOLUME /etc/ssl/
