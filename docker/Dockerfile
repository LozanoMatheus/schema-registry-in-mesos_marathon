FROM alpine:3.8
LABEL Source="https://github.com/Landoop/schema-registry-ui/" \
      Author="Matheus Lozano Sanches <LozanoMatheus@Gmail.com>"

EXPOSE 8000

ADD schema-registry-ui-package-__LAST_VERSION__.tar.gz /

COPY run.sh /run.sh

ENTRYPOINT ["/run.sh"]
