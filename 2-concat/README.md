## Introduction

This example addresses the problem that fluentd driver will break multi-line logs into individual events. 

`fluent-plugin-concat` plug-in is used to concatenate log lines back into a single multi-line event. The following snippet shows the content of `fluentd.confg`. This example finds a certain pattern `multiline_start_regexp` in the `log` field of each event, and determines the starting line of each multi-line log. `stream_identity_key` is used to separate log from different container to prevent interference. `flush_interval` tells the plug-in to end a multi-line log if there's no more log lines received in 5 seconds.

```xml
<source>
  @type forward
  port 24224
</source>

<filter **>
  @type concat
  stream_identity_key container_id # prevent interference between containers
  key log                          # event field containing pattern of starting lines
  multiline_start_regexp /^\[/     # pattern only present in starting lines
  flush_interval 5s                # terminate multi-line logs if it's been idle for 5 secnds
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

`fluent-plugin-concat` will need to be installed in `fluentd` image.

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

2. Start 2 logger containers at the same time

```bash
docker-compose up logger1 logger2
```

Logs been print out by fluentd container should be mult-lined and logs from different containers should not be mixed up.

