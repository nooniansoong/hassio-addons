# https://developers.home-assistant.io/docs/add-ons/configuration#add-on-dockerfile
# Based on litinoveweedle/hassio-addons-dev taptap/Dockerfile (add-on v0.3.4:
# taptap v0.2.6 + taptap-mqtt v0.2.6), unchanged except for the rootfs part.

# Define build arguments with defaults (from build.yaml)
ARG BUILD_ARCH="amd64"
ARG BUILD_REPO="nooniansoong/hassio-addons"
ARG BUILD_DESCRIPTION="TapTap Multi - taptap add-on for multiple Tigo CCAs"
# renovate: datasource=github-releases depName=taptap packageName=litinoveweedle/taptap versioning=semver
ARG TAPTAP_VERSION="0.2.6"
# renovate: datasource=github-releases depName=taptap-mqtt packageName=litinoveweedle/taptap-mqtt versioning=semver
ARG TAPTAP_MQTT_VERSION="0.2.6"

# Use multi-arch base image
# renovate: datasource=github-releases depName=hassio-addon-base packageName=hassio-addons/addon-base versioning=semver
FROM ghcr.io/hassio-addons/base/${BUILD_ARCH}:20.1.0

# Bring build arguments into the build context
ARG BUILD_ARCH
ARG BUILD_DATE
ARG BUILD_DESCRIPTION
ARG BUILD_NAME
ARG BUILD_REF
ARG BUILD_REPO
ARG BUILD_VERSION
ARG TAPTAP_VERSION
ARG TAPTAP_MQTT_VERSION

WORKDIR /tmp

# Install python/pip
RUN apk upgrade --no-cache && apk add --no-cache python3 py3-pip

# Get taptap binary
RUN \
    if [ "${BUILD_ARCH}" = "aarch64" ]; then \
        TAPTAP_ARCH="musl-arm64"; \
    elif [ "${BUILD_ARCH}" = "amd64" ]; then \
        TAPTAP_ARCH="musl-x86_64"; \
    else \
        exit 1; \
    fi \
    && mkdir /usr/bin/taptap \
    && chmod 755 /usr/bin/taptap \
    && curl -sSLf -o /tmp/taptap.tgz \
    "https://github.com/litinoveweedle/taptap/releases/download/v${TAPTAP_VERSION}/taptap-Linux-${TAPTAP_ARCH}.tar.gz" \
    && tar -xzvf taptap.tgz \
    && cp taptap /usr/bin/taptap/ \
    && chmod 755 /usr/bin/taptap/taptap \
    && rm -Rf /tmp/*

# Get taptap-mqtt bridge
RUN \
    mkdir /etc/taptap \
    && chmod 755 /etc/taptap \
    && curl -sSLf -o /tmp/taptap-mqtt.tgz \
    "https://github.com/litinoveweedle/taptap-mqtt/archive/refs/tags/v${TAPTAP_MQTT_VERSION}.tar.gz" \
    && tar -xzvf taptap-mqtt.tgz \
    && cp taptap-mqtt-*/taptap-mqtt.py /usr/bin/taptap/ \
    && chmod 755 /usr/bin/taptap/taptap-mqtt.py \
    && pip install -r taptap-mqtt-*/requirements.txt \
    && rm -Rf /tmp/*

# Copy root filesystem (make scripts executable even if file modes were lost
# while copying the add-on folder to Home Assistant, e.g. via Samba)
COPY rootfs /
RUN chmod 775 /etc \
    && chmod 755 \
        /etc/s6-overlay/s6-rc.d/init-taptap/run \
        /etc/s6-overlay/s6-rc.d/taptap/run \
        /etc/s6-overlay/s6-rc.d/taptap/finish \
    && mkdir -p /etc/taptap/instances

# Labels
LABEL \
    io.hass.name="${BUILD_NAME}" \
    io.hass.description="${BUILD_DESCRIPTION}" \
    io.hass.arch="${BUILD_ARCH}" \
    io.hass.type="addon" \
    io.hass.version=${BUILD_VERSION} \
    maintainer="Dominik Strnad <litinoveweedle@gmail.com>" \
    org.opencontainers.image.title="${BUILD_NAME}" \
    org.opencontainers.image.description="${BUILD_DESCRIPTION}" \
    org.opencontainers.image.authors="Dominik Strnad <litinoveweedle@gmail.com>" \
    org.opencontainers.image.licenses="Apache 2.0" \
    org.opencontainers.image.url="https://github.com/${BUILD_REPO}" \
    org.opencontainers.image.source="https://github.com/${BUILD_REPO}" \
    org.opencontainers.image.documentation="https://github.com/${BUILD_REPO}/blob/main/README.md" \
    org.opencontainers.image.created=${BUILD_DATE} \
    org.opencontainers.image.revision=${BUILD_REF} \
    org.opencontainers.image.version=${BUILD_VERSION}
