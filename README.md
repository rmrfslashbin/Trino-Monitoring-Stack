# Trino-Stack
An example Docker stack to deploy and monitor a Trino server.

## Security Note
This stack is NOT secure. It is only a PoC. YMMV in prod.

## Build Trino
If needed, build a new Docker image for Trino. This is only needed if additional plugins are required. Replace your Trino image as needed in `docker-compose.yml`.

## Trino Config
### Hive
See `./trino-server/etc/trino/catalog/hive.properties`. Fill in the blanks.

### jvm.config
See `./trino-server/etc/trino/jvm.config` and fix up as needed. Make note of `com.sun.management.jmxremote.port` value.

### Docker Set up
The contents of `./trino-server/etc` should be copied to a Docker volume named `trino-config`.

### JSON SerDe
This example addes a JSON SerDe plugin to the Trino server. The SerDe brings the ability to read gziped json files. The jar can be found at http://www.congiu.net/hive-json-serde/1.3.8/hdp23/json-serde-1.3.8-jar-with-dependencies.jar.

## Grafana
See https://grafana.com/docs/grafana/latest/installation/docker/ for more info. Initial user id/passwd is admin/admin. the app will prompt for a password change at initial login.

### Set up
Add a `graphite` data source. The URL of the Graphite instance is `http://graphite:80`.

## Graphite
There isn't much to do with Graphite. It just works. It will take about 20 to 30 minutes for initial data to populate.

## jmxtrans
This is where the magic happens. `jmxtrans` is the glue between Trino jmx stats and `Graphite`.

### Config
A volume `jmxtrans-config` is defined. In this volume, place a config file to reference the Trino jmx endpoint to monitor. See https://github.com/jmxtrans/jmxtrans/wiki/MoreExamples for examples. A sameple file is provided in this repo `./jmxtrans/var/lib/jmxtrans/trino.json`.

The host/port for the json config refelect the Trino host name (`trino` in this example) and the `com.sun.management.jmxremote.port` value noted above- in this example `9080`.

