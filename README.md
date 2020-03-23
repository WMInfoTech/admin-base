# Bandock Banner 9 Admin Base
Banner 9 Admin Base is a base image to build the Banner Admin applications on top of.

The intent is that images built on this base will be portable across production, test, and
development instance of your install. We attempt to set everything required by Tomcat, and
the settings that are contained in your war file, typically those found in xml files.

Runtime configuation is managed by the settings in the run.sh file. The resulting
image should work on Docker Swarm, Kubernetes, and other popular container orchestrators.

## How to use

Copy extracted war files to `/usr/local/tomcat/webapps/`, where application.war becomes
`webapps/application`. 

A Dockerfile might look like
```Dockerfile
FROM admin-base

COPY --chown=tomcat:tomcat application /usr/local/tomcat/webapps/application/
COPY --chown=tomcat:tomcat application.ws /usr/local/tomcat/webapps/application.ws/
COPY --chown=tomcat:tomcat applicationHelp /usr/local/tomcat/webapps/applicationHelp/
```

Properties used by Tomcat can be loaded from environment variables or files
(typically provided by your orchestrator as configs or secrets). The settings
required by Tomcat for JNDI connections are appended to catalina.properties before Tomcat starts.

If you don't set something, the default values from the Dockerfile are used.


## Properties from Environment Variables

Environment variables are used to setup the base parameters for tomcat to connect to the database.

### Defaults

If an environment variable is not specificed at runtime then the defaults for the variable will be used. At a minimum BANNERDB_JDBC, BANPROXY_PASSWORD, CAS_URL, and BANNER9_URL need to be set
for your application to be useful. It is highly recommended setting the TIMEZONE to the timezone of your Banner database too.

### Environment Variables

```Shell
TIMEZONE - default: America/New_York
XMS - default: 2g
XMX - default: 4g
BANNERDB_JDBC - default: jdbc:oracle:thin:@//oracle.example.edu:1521/prod
BANPROXY_USERNAME - default: banproxy
BANPROXY_PASSWORD - no default
BANPROXY_INITALSIZE - default: 25
BANPROXY_MAXTOTAL - default: 400
BANPROXY_MAXIDLE - default: -1
BANPROXY_MAXWAIT - default: 30000
BANPROXY_MAXACTIVE - default: 100
REMOVE_ABANDONED_ON_MAINTENANCE - default: true
REMOVE_ABANDONED_ON_BORROW - default: true
REMOVE_ABANDONED_TIMEOUT - default: 2100
LOG_ABANDONED - default: true
CAS_URL - example: https://cas.local.com/cas
BANNER9_URL - example: https://banner9.school.edu
THEME_URL - example: https://banner9.school.edu/BannerExtensibility/theme/getTheme?name=dev&amp;template=admin
```

(Note: If a THEME_URL is not supplied, themeing will not be enabled.)

## Properties from Config file

When loading from a config file, set the environment variable `CONFIG_FILE` to the location
of the configuration file, from there all other environment variables will be ignored by run.sh.

```Shell
bannerdb.jdbc=jdbc:oracle:thin:@//oracle.example.edu:1521/prod
banproxy.username=banproxy
banproxy.password=password
banproxy.initialsize=25
banproxy.maxtotal=400
banproxy.maxidle=-1
banproxy.maxwait=30000
banproxy.maxactive=100
remove.abandoned.on.maintenance = true
remove.abandoned.on.borrow = true
remove.abandoned.timeout = 2100
log.abandoned = true
cas.url=https://cas.local.com/cas
banner9.baseurl=https://banner9.school.edu
theme.url=https://banner9.school.edu/BannerExtensibility/theme/getTheme?name=dev&template=admin
```

## Monitoring

JMX can be enabled by setting the `JMX_PORT` environment variable.

### Prometheus

If you'd prefer to directly expose metrics in a format that Prometheus
can scrape, set the `PROMETHEUS_JMX_PORT` environment variable to the
port number you'd like for the
[JMX exporter](https://github.com/prometheus/jmx_exporter) to listen on.

The [example tomcat config file](https://github.com/prometheus/jmx_exporter/blob/master/example_configs/tomcat.yml)
is included in the image, but it can over overriden by setting the
`JMX_EXPORTER_CONFIG` environment variable to the location of the config
file you'd like to use.