ARG BUILD_FROM

# hadolint ignore=DL3006
FROM ${BUILD_FROM} as base

# Common build arguments
ARG BUILD_ARCH
ARG BUILD_DATE
ARG BUILD_DESCRIPTION
ARG BUILD_NAME
ARG BUILD_REF
ARG BUILD_REPOSITORY
ARG BUILD_VERSION

#ENV MQTT_IO_REPO=https://github.com/Hacker-CB/mqtt-io
#ENV MQTT_IO_VERSION=handle_initial_none

ENV MQTT_IO_REPO=https://github.com/flyte/mqtt-io
ENV MQTT_IO_VERSION=2.2.8


#
# Generate requirements.txt with poetry
#
FROM base AS requirements

RUN apk add --no-cache \
        openssl=1.1.1t-r2 \
        gcc=11.2.1_git20220219-r2 \
        musl=1.2.3-r2 \
        musl-dev=1.2.3-r2 \
        libffi-dev=3.4.2-r1 \
        git=2.36.6-r0   \
 \
    && pip install --no-cache-dir poetry==1.3.2

WORKDIR /usr/src
#RUN git clone --single-branch --branch 2.2.6 https://github.com/flyte/mqtt-io mqtt_io

# hadolint ignore=DL3003
RUN git clone -n --single-branch --branch "$MQTT_IO_VERSION" "$MQTT_IO_REPO" mqtt_io \
    && cd mqtt_io \
    && git checkout "$MQTT_IO_VERSION"


WORKDIR /usr/src/mqtt_io
RUN mkdir -p /home/mqtt_io \
    && poetry export -o /home/mqtt_io/requirements.txt

WORKDIR /home/mqtt_io
RUN python3 -m venv venv \
    && venv/bin/pip install wheel==0.37.0

#
# Generate final image
#
FROM base

# Install global packages
RUN pip install --no-cache-dir yq==2.12.2

WORKDIR /home/mqtt_io

# Copy mqtt-io
COPY --from=requirements /home/mqtt_io/venv ./venv
COPY --from=requirements /home/mqtt_io/requirements.txt ./
COPY --from=requirements /usr/src/mqtt_io/mqtt_io mqtt_io

RUN venv/bin/pip install --no-cache-dir -r requirements.txt && \
    venv/bin/pip install --no-cache-dir gpiod==1.5.0

# Copy root filesystem
COPY rootfs /

#
# Labels
#
LABEL \
    io.hass.name="${BUILD_NAME}" \
    io.hass.description="${BUILD_DESCRIPTION}" \
    io.hass.arch="${BUILD_ARCH}" \
    io.hass.type="addon" \
    io.hass.version=${BUILD_VERSION} \
    maintainer="Pavel Sokolov <pavel@jethome.ru>" \
    org.opencontainers.image.title="${BUILD_NAME}" \
    org.opencontainers.image.description="${BUILD_DESCRIPTION}" \
    org.opencontainers.image.vendor="JetHome" \
    org.opencontainers.image.authors="Pavel Sokolov <pavel@jethome.ru>" \
    org.opencontainers.image.licenses="MIT" \
    org.opencontainers.image.url="https://jethome.ru" \
    org.opencontainers.image.source="https://github.com/${BUILD_REPOSITORY}" \
    org.opencontainers.image.documentation="https://github.com/${BUILD_REPOSITORY}/blob/main/README.md" \
    org.opencontainers.image.created=${BUILD_DATE} \
    org.opencontainers.image.revision=${BUILD_REF} \
    org.opencontainers.image.version=${BUILD_VERSION}
