# https://developers.home-assistant.io/docs/add-ons/configuration#add-on-dockerfile
#
# TapTap Multi is built on top of the prebuilt upstream TapTap add-on image
# (litinoveweedle/hassio-addons taptap v0.3.4 = taptap v0.2.6 + taptap-mqtt
# v0.2.6, multi-arch). Only the s6 service scripts are replaced, so the local
# build on Home Assistant is a quick layer on top of an image that is usually
# already cached (if the original TapTap add-on was installed before).
#
# To follow a new upstream release, bump TAPTAP_ADDON_VERSION and the add-on
# version in config.yaml.

# renovate: datasource=docker depName=ghcr.io/litinoveweedle/hassio-addon-taptap
ARG TAPTAP_ADDON_VERSION="0.3.4"
FROM ghcr.io/litinoveweedle/hassio-addon-taptap:${TAPTAP_ADDON_VERSION}

ARG BUILD_ARCH
ARG BUILD_DATE
ARG BUILD_DESCRIPTION="TapTap Multi - taptap add-on for multiple Tigo CCAs"
ARG BUILD_NAME
ARG BUILD_REF
ARG BUILD_REPO="nooniansoong/hassio-addons"
ARG BUILD_VERSION

# Replace the upstream single-instance service scripts with the multi-instance
# ones and taptap-mqtt.py with the patched copy (device per optimizer);
# make them executable even if file modes were lost while copying
COPY rootfs /
RUN rm -f /etc/taptap/config.gtpl /etc/taptap/config.ini \
    && mkdir -p /etc/taptap/instances \
    && chmod 755 \
        /etc/s6-overlay/s6-rc.d/init-taptap/run \
        /etc/s6-overlay/s6-rc.d/taptap/run \
        /etc/s6-overlay/s6-rc.d/taptap/finish \
        /usr/bin/taptap/taptap-mqtt.py \
    && test -x /usr/bin/taptap/taptap \
    && test -x /usr/bin/taptap/taptap-mqtt.py

LABEL \
    io.hass.name="${BUILD_NAME}" \
    io.hass.description="${BUILD_DESCRIPTION}" \
    io.hass.arch="${BUILD_ARCH}" \
    io.hass.type="addon" \
    io.hass.version=${BUILD_VERSION} \
    org.opencontainers.image.title="${BUILD_NAME}" \
    org.opencontainers.image.description="${BUILD_DESCRIPTION}" \
    org.opencontainers.image.licenses="Apache 2.0" \
    org.opencontainers.image.url="https://github.com/${BUILD_REPO}" \
    org.opencontainers.image.source="https://github.com/${BUILD_REPO}" \
    org.opencontainers.image.created=${BUILD_DATE} \
    org.opencontainers.image.revision=${BUILD_REF} \
    org.opencontainers.image.version=${BUILD_VERSION}
