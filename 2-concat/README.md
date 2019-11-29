## Introduction

```xml
<source>
  @type forward
  port 24224
</source>

<filter **>
  @type concat
  key log
  stream_identity_key container_id # prevent interference
  multiline_start_regexp /^\[/
  flush_interval 5s
  timeout_label @STDOUT
</filter>

<match **>
  @type relabel
  @label @STDOUT
</match>

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
 && sudo gem install fluent-plugin-concat \
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
docker-compose up logger1 logger2
```
