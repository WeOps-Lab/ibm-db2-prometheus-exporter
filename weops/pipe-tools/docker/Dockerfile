#
# build container
#
FROM --platform=linux/amd64 golang:1.19-alpine as builder
WORKDIR /

COPY db2_exporter_linux_amd64 /db2_exporter_linux_amd64

RUN chmod +x /db2_exporter_linux_amd64

EXPOSE     9953
ENTRYPOINT [ "/db2_exporter_linux_amd64" ]
