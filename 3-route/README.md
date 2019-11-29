## Introduction

This example shows how to send the same event to two different destination, e.g. Splunk for monitoring, S3 bucket for archiving & auditing, ...

`fluent-plugin-route` plug-in is used for the purpose. The following config make every forwarded event into 2 copies. One for writing to files and one for printing out to console. `label` is utilized to config more complex pipeline like this.

```xml
<source>
  @type forward
  port 24224
</source>

<match **>
    @type route
    <route **>
        copy
        @label @FILE
    </route>
    <route **>
        copy
        @label @STDOUT
    </route>
</match>

<label @FILE>
  <match **>
    @type file
    path /output/
  </match>
</label>

<label @STDOUT>
  <match **>
    @type stdout
  </match>
</label>
```

`fluent-plugin-route` will need to be installed in `fluentd` image.

```dockerfile
FROM fluent/fluentd:latest

RUN apk add --no-cache --update --virtual .build-deps \
        sudo build-base ruby-dev \
 && sudo gem install fluent-plugin-route \
 && sudo gem sources --clear-all \
 && apk del .build-deps \
 && rm -rf /tmp/* /var/tmp/* /usr/lib/ruby/gems/*/cache/*.gem

COPY fluent.conf /fluentd/etc/

EXPOSE 24224
```

## Instructions

1. Start fluentd container

  ```bash
docker-compose up fluentd
```

2. Start logger containers

  ```bash
docker-compose up logger
```

  Logs should be printed out by `fluentd` container and also be saved to files in folder `output`.
