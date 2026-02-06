# Build Custom OpenTelemetry Collector for Red Hat OpenShift

This repository describes the procedure to build a custom otel collector (based on official RHEL image) compatible for Red Hat build of Open Telemetry Operator v0.140.0.

## Prerequisites

### Components
The versions of all used componets are listed below:
| Component       | version |
|------------|-----|
| go      | 1.24  |
| Red Hat build of Open Telemetry Operator        | v0.140.0-2  |
| ocb / otel collector builder        | 0.140.0  |
| Red Hat Openshift Container Platform    | 4.16  |

### Base Images
The build process use the following base images:
| Base Image       | Registry | |
|------------|------------|------------|
| ubi8/go-toolset:1.24      | registry.redhat.io | ```podman login``` required  | 
| ubi8/ubi-minimal      | registry.redhat.io | ```podman login``` required | 

### Additional Modules
The additional modules in addition to the standard ones available [here](https://github.com/open-telemetry/opentelemetry-collector-releases/blob/v0.140.0/distributions/otelcol/manifest.yaml):
| Repository  | Type  | Version |
|------------|------------|---------|
| github.com/open-telemetry/opentelemetry-collector-contrib/receiver/syslogreceiver |  receiver  | v0.140.1  | 
| github.com/open-telemetry/opentelemetry-collector-contrib/exporter/syslogexporter  | exporter | v0.140.1 | 
| github.com/open-telemetry/opentelemetry-collector-contrib/processor/transformprocessor |  processor  | v0.140.1  | 

> ⚠️ **Warning:** Using custom/unsupported components voids official vendor support.

### Set Environment
Configure the OTEL version:
```bash
export OTEL_VERSION=0.140.0
```

> ⚠️ **Attention:** The build process is highly resource-intensive. For a faster experience, we recommend running it on a machine with sufficient resources.

## OpenShift Build
Create `ImageStreams` and `BuildConfig`:
```bash
oc apply -f build.yaml
```

Start a new build:
```bash
oc start-build otelcollector-syslog-build --build-arg OTEL_VERSION=$OTEL_VERSION --follow
```

## Local Build

### Build image
Build image locally with podman:
```bash
podman build --build-arg OTEL_VERSION=$OTEL_VERSION -t <registry>/<namespace>/otelcollector-syslog:$OTEL_VERSION .
```

Push image to the rigistry with podman:
### Push image
```bash
podman push <registry>/<namespace>/otelcollector-syslog:$OTEL_VERSION
```

## Usage Example

```yaml
    receivers:
      syslog:
        tcp:
          listen_address: "0.0.0.0:54526"
        protocol: rfc5424

    processors:
      transform:
        log_statements:
          - context: log
            statements:
              # rewrite syslog header 'HOSTNAME'
              - set(attributes["hostname"], Concat([attributes["appname"], ".acme.org"], "")) 
                where attributes["appname"] != nil
              - set(attributes["hostname"], "unknown") 
                where attributes["appname"] == nil

    exporters:
      syslog:
        network: tcp
        endpoint: syslog.acme.org
        port: 514
        protocol: rfc5424
```

## Acknowledgments

A special thanks to [@ldileonardorh](https://github.com/ldileonardorh/otel-collector) for his invaluable contribution!