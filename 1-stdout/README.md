## Introduction

This example will show how to setup a `fluentd` container with basic configuration, and how to forward logs from other containers to it using docker logging driver.

First, `fluentd.conf` should be defined to to control the behavior of `fluentd` container. In the following snippet, it first defines a `forward` input plug-in. It will receive log events from port `24224`. Then, it defines a `stdout` output plug-in. It will print out all log events from the previous step to console.

```xml
<source>
  @type forward
  port 24224
</source>

<match **>
  @type stdout
</match>
```

Then, a customized `fluentd` image should be created with our config file.

```dockerfile
FROM fluent/fluentd:latest

COPY fluent.conf /fluentd/etc/

EXPOSE 24224
```

After the customized `fluentd` image is ready, docker logging drivers could start to attach to it. The following snippet of `docker-compose.yaml` shows the basic settings for `fluentd` logging driver.

```yml
    logging:
      driver: fluentd
      options:
        fluentd-address: localhost:24224
        tag: logger
```

## Instructions

1. Start fluentd container

```bash
docker-compose up fluentd
```

2. Start logger container

```bash
docker-compose up logger
```
