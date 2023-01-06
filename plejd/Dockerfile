ARG BUILD_FROM=alpine:3.12.1
## hadolint ignore=DL3006
FROM ${BUILD_FROM}

# Environment variables
ENV \
    HOME="/root" \
    LANG="C.UTF-8" \
    PS1="$(whoami)@$(hostname):$(pwd)$ " \
    S6_BEHAVIOUR_IF_STAGE2_FAILS=2 \
    S6_CMD_WAIT_FOR_SERVICES=1 \
    TERM="xterm-256color"

# Copy root filesystem
COPY addon-base-rootfs /

# Set shell
SHELL ["/bin/ash", "-o", "pipefail", "-c"]

# Install base system
ARG BUILD_ARCH=armv7
RUN \
    set -o pipefail \
    \
    && echo '@edge http://dl-cdn.alpinelinux.org/alpine/edge/main' >> /etc/apk/repositories \
    && echo '@edge http://dl-cdn.alpinelinux.org/alpine/edge/community' >> /etc/apk/repositories \
    && echo '@edge http://dl-cdn.alpinelinux.org/alpine/edge/testing' >> /etc/apk/repositories \
    \
    && apk add --no-cache \
        libcrypto1.1=1.1.1g-r0 \
        libssl1.1=1.1.1g-r0 \
        musl-utils=1.1.24-r10 \
        musl=1.1.24-r10 \
    \
    && apk add --no-cache \
        bash=5.0.17-r0 \
        curl=7.79.1-r1 \
        jq=1.6-r1 \
        tzdata=2022a-r0 \
    \
    && S6_ARCH="${BUILD_ARCH}" \
    && if [ "${BUILD_ARCH}" = "i386" ]; then S6_ARCH="x86"; fi \
    && if [ "${BUILD_ARCH}" = "armv7" ]; then S6_ARCH="arm"; fi \
    \
    && curl -L -s "https://github.com/just-containers/s6-overlay/releases/download/v2.1.0.2/s6-overlay-${S6_ARCH}.tar.gz" \
        | tar zxvf - -C / \
    \
    && mkdir -p /etc/fix-attrs.d \
    && mkdir -p /etc/services.d

# Entrypoint & CMD
ENTRYPOINT ["/init"]

# Set shell
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Copy data for add-on
COPY ./*.js /plejd/
COPY ./config.json /plejd/
COPY ./package.json /plejd/

ARG BUILD_ARCH

# Install Node
RUN apk add --no-cache jq
RUN \
  apk add --no-cache --virtual .build-dependencies \
  g++ \
  gcc \
  libc-dev \
  linux-headers \
  make \
  python3 \
  bluez \
  eudev-dev \
  zlib-dev \
  \
  && apk add --no-cache \
  git \
  nodejs \
  npm \
  dbus-dev \
  glib-dev \
  \
  && npm config set unsafe-perm true

WORKDIR /plejd
RUN npm install \
  --no-audit \
  --no-update-notifier \
  --unsafe-perm

# Copy root filesystem
COPY rootfs /

# Set up empty /data/options.json
RUN mkdir -p /data && echo "{}" > /data/options.json

# Build arguments
ARG BUILD_DATE
ARG BUILD_REF
ARG BUILD_VERSION

# Labels
LABEL \
  io.hass.name="Plejd" \
  io.hass.description="Adds support for the Swedish home automation devices from Plejd." \
  io.hass.arch="${BUILD_ARCH}" \
  io.hass.type="addon" \
  io.hass.version=${BUILD_VERSION} \
  maintainer="Marcus Westin <marcus@sekurbit.se>" \
  org.label-schema.description="Adds support for the Swedish home automation devices from Plejd." \
  org.label-schema.build-date=${BUILD_DATE} \
  org.label-schema.name="Plejd" \
  org.label-schema.schema-version="1.0" \
  org.label-schema.usage="https://github.com/icanos/hassio-plejd/tree/master/README.md" \
  org.label-schema.vcs-ref=${BUILD_REF} \
  org.label-schema.vcs-url="https://github.com/icanos/hassio-plejd"
  
