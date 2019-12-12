## Introduction

This example will show how to mutates/transforms log events, e.g. add new fields, delete unnecessary fields, or modify events. `filter_record_transformer` is used for this purpose, and it is included in Fluentd's core and requires no installation.

Here we will try `fluent-plugin-record-modifier` plug-in. It is the light-weight and faster version of `filter_record_transformer`. 
The following config uses ${xxx} placeholders and accesses the values of tag, time and record through Ruby code to modify incoming events.

```xml
<source>
  @type forward
  port 24224
</source>

<filter **>
    @type record_modifier
    remove_keys source,container_id
    <record>
        foo bar
        tag ${tag}
        formatted_time ${Time.at(time).to_s}
        container_name ${record["container_name"].gsub('/', '')}
    </record>
</filter>

<match **>
  @type stdout
</match>
```

- If following record is passed:
```json
{
  "container_id":"88888",
  "container_name":"/6-modifier_logger",
  "source":"stdout",
  "log":"hello world!"
}
```

- then you got the new record like below:
```json
{
  "foo":"bar",
  "tag":"logger",
  "formatted_time":"2019-12-12 03:02:08 +0000",
  "container_name":"6-modifier_logger",
  "log":"hello world!"
}
```

`fluent-plugin-record-modifier` will need to be installed in `fluentd` image.

```dockerfile
FROM fluent/fluentd:latest

RUN apk add --no-cache --update --virtual .build-deps \
        sudo build-base ruby-dev \
 && sudo gem install fluent-plugin-record-modifier \
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

2. Start logger container

  ```bash
docker-compose up logger
```
