# TapTap Multi (local Home Assistant add-on)

Local fork of the [TapTap add-on](https://github.com/litinoveweedle/hassio-addons/tree/main/taptap)
v0.3.4 that monitors **several Tigo CCAs at once** – one taptap-mqtt bridge per
RS485 converter (e.g. Elfin EW11), all inside one add-on.

- Built on the prebuilt upstream image `ghcr.io/litinoveweedle/hassio-addon-taptap:0.3.4`
  (taptap v0.2.6, taptap-mqtt v0.2.6); only the s6 service scripts are replaced
- New option `instances` (list of `name`, `address`/`serial`, `port`, `modules`)
- Per-instance HA device, MQTT topic, state file and log prefix
- If one bridge exits, the others are stopped gracefully and the add-on exits
  (enable the Watchdog to auto-restart)

See DOCS.md for wiring, EW11 settings, configuration and installation.
