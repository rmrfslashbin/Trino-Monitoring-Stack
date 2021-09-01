# Trino-Monitoring-Stack
An example [Docker](https://www.docker.com/) [stack](https://github.com/docker/compose) to deploy and monitor a [Trino](https://trino.io/) server using [jmxtrans](https://github.com/jmxtrans/jmxtrans), [Graphite](https://graphiteapp.org/), and [Grafana](https://grafana.com/).

## Security Note
This stack is NOT secure. It is only a PoC (Proof of Concept). YMMV in prod.

## Assumptions
This PoC is written assuming the following are available:

- A functioning Docker platform.
- A functioning docker-compose install.
- A functional understanding of Docker, Trino, and Grafana.

Nice to have:
- A data [connector](https://trino.io/docs/current/connector.html) to configure in Trino.

## Trino
### Build Trino Server
This PoC comes with a [Dockerfile](trino-server/Dockerfile) to build a custom Trino server with an additional [SerDe](https://cwiki.apache.org/confluence/display/Hive/SerDe/) to process gziped json files. Building the custom Trino server is optional. In fact, if you don't need to install any additional plugins, just use the offical [Trino image](https://hub.docker.com/r/trinodb/trino) from [Docker Hub](https://hub.docker.com).

In either case, be sure to use your selected Trino image in the [docker-compose.yml](docker-compose.yml) file.

### JSON SerDe
As mentioned above, this PoC adds a JSON SerDe plugin to the Trino server. The SerDe brings the ability to read gziped json files. The jar can be found at http://www.congiu.net/hive-json-serde/1.3.8/hdp23/json-serde-1.3.8-jar-with-dependencies.jar. You'll know you need this if you need it. Otherwise, move on!

### Trino Config
#### Connectors
This PoC provides a bare-bones example to connect to [AWS Glue](https://aws.amazon.com/glue). Fix up [trino-server/etc/trino/catalog/hive.properties](trino-server/etc/trino/catalog/hive.properties), provide your own connector, or skip this step (the only data available will be internal stats, but that's good enough for this PoC).

#### jvm.config
[trino-server/etc/trino/jvm.config](trino-server/etc/trino/jvm.config) defines the runtime settings for Trino. Fix up to suit your needs. This is where [JMX](https://en.wikipedia.org/wiki/Java_Management_Extensions) is enabled. Make note of `com.sun.management.jmxremote.port` value- that's one of the important bits to connect Trino JMX to jmxtrans.

#### Docker Set up
The contents of [trino-server/etc](trino-server/etc) should be copied to the root of a Docker volume named `trino-config`. Setting up and copying data to a Docker Volume is beyond the scope of this PoC, but it's not too difficult.


## jmxtrans
`jmxtrans` is the glue between Trino JMX stats and `Graphite`.

### Config
A volume (`jmxtrans-config`) is defined in the Docker Compose file. This volume contains config/json files that define what JMX endpoints to monitor. See https://github.com/jmxtrans/jmxtrans/wiki/MoreExamples for examples. A sameple file is provided in this PoC [jmxtrans/var/lib/jmxtrans/trino.json](jmxtrans/var/lib/jmxtrans/trino.json).

The host/port for the json config refelect the Trino host name (`trino` in this PoC, as defined inthe Docker Compose file) and the `com.sun.management.jmxremote.port` port noted above. This PoC defines the port as `9080`. The json config file also direct that stats output to Graphite.

## Graphite
There isn't much to do with Graphite. It just works. It will take about 20 to 30 minutes for initial data to populate.

## Grafana
Grafana is the presentation layer for this stack. Config and use is beyond the scope of this PoC, but there's plenty of guidance to be found elsewhere. See https://grafana.com/docs/grafana/latest/installation/docker/ for more info. Initial user id/passwd is admin/admin. 

### Set up
Once you're in, Add a `graphite` data source. The URL of the Graphite instance is `http://graphite:80`.

## That's it!
Feedback is appreciated as are pull requests for bugs and typos.

## Useful Links
- https://github.com/jmxtrans/jmxtrans/wiki/Installation
- http://graphiteapp.org/
- https://trino.io/docs/current/installation/deployment.html
- https://grafana.com/docs/grafana/latest/administration/configure-docker/
- https://hub.docker.com/r/graphiteapp/graphite-statsd
- https://hub.docker.com/r/jmxtrans/jmxtrans
