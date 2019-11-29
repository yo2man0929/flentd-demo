## Introduction

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

3. See logs being output to folder `output`.
