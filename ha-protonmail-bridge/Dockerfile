FROM golang:alpine AS build
LABEL authors="David BASTIEN"
ARG ENV_PROTONMAIL_BRIDGE_VERSION

# Install dependencies
RUN apk update
RUN apk add bash
RUN apk add make gcc g++ libc-dev musl musl-dev sed grep sed
RUN apk add git libsecret libsecret-dev pass

# Build stage
WORKDIR /build/
RUN git clone -b $ENV_PROTONMAIL_BRIDGE_VERSION https://github.com/ProtonMail/proton-bridge.git
WORKDIR /build/proton-bridge/
# Not needed anymore, golang 1.23 seems to have solved this.
# Solve bug https://github.com/mattn/go-sqlite3/pull/1177
# COPY fix_1177.sh /build/proton-bridge/
# RUN chmod u+x /build/proton-bridge/fix_1177.sh
# RUN /bin/bash -c '/build/proton-bridge/fix_1177.sh'
# make
RUN make build-nogui

# Working stage image
FROM golang:alpine
LABEL authors="David BASTIEN"
LABEL org.opencontainers.image.source="https://github.com/VideoCurio/ProtonMailBridgeDocker"

# Define arguments and env variables
# Indicate (NOT define) the ports/network interface really used by Proton bridge mail.
# It should be 1025/tcp and 1143/tcp but on some k3s instances it could be 1026 and 1144 (why ?)
# Launch `netstat -ltnp` on a running container to be sure.
ARG ENV_BRIDGE_SMTP_PORT=1025
ARG ENV_BRIDGE_IMAP_PORT=1143
ARG ENV_BRIDGE_HOST=127.0.0.1
# Change ENV_CONTAINER_SMTP_PORT only if you have a docker port conflict on host network namespace.
ARG ENV_CONTAINER_SMTP_PORT=25
ARG ENV_CONTAINER_IMAP_PORT=143
ENV PROTON_BRIDGE_SMTP_PORT=$ENV_BRIDGE_SMTP_PORT
ENV PROTON_BRIDGE_IMAP_PORT=$ENV_BRIDGE_IMAP_PORT
ENV PROTON_BRIDGE_HOST=$ENV_BRIDGE_HOST
ENV CONTAINER_SMTP_PORT=$ENV_CONTAINER_SMTP_PORT
ENV CONTAINER_IMAP_PORT=$ENV_CONTAINER_IMAP_PORT

# Install dependencies
RUN apk update
RUN apk upgrade
RUN apk add bash socat net-tools libsecret pass gpg gpg-agent ca-certificates

# RUN rc-update add dbus
# RUN touch /run/openrc/softlevel
# RUN rc-service dbus start

# Copy executables made during previous stage
WORKDIR /app/
COPY --from=build /build/proton-bridge/bridge /app/
COPY --from=build /build/proton-bridge/proton-bridge /app/

# Install needed scripts and files
COPY VERSION /app/
COPY entrypoint.sh /app/
RUN chmod u+x /app/entrypoint.sh
COPY GPGparams.txt /app/

COPY LICENSE.txt /app/

# SMTP and IMAP ports (25/tcp and 143/tcp) are not exposed by default, so you could adjust them if necessary with ENV
# variables CONTAINER_SMTP_PORT and CONTAINER_IMAP_PORT.
# See README.md and/or compose.yaml file.
# EXPOSE ${ENV_CONTAINER_SMTP_PORT}/tcp
# EXPOSE ${ENV_CONTAINER_IMAP_PORT}/tcp

# Volume to save pass and bridge configurations/data
VOLUME /root

ENTRYPOINT ["/app/entrypoint.sh"]
