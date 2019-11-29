## Introduction

This example shows how to used fluentd to forward log events to Splunk.

`fluent-plugin-splunk-hec` is used for this purpose. It communitates with Splunk through HEC (HTTP Event Collector) protocol. It requires an HEC token, which should be created in Splunk UI in advance. In this example, the HEC token is passed in from environment variable. It's a more secure way to avoid hard-coded secret tokens.

```xml
<source>
  @type forward
  port 24224
</source>

<match **>
  @type splunk_hec
  hec_host "#{ENV['SPLUNK_HEC_HOST']}"
  hec_port "#{ENV['SPLUNK_HEC_PORT']}"
  hec_token "#{ENV['SPLUNK_HEC_TOKEN']}"
  source_key service_name
  insecure_ssl true
  <buffer>
    flush_interval 1s
  </buffer>
</match>
```

`fluent-plugin-splunk-hec` will need to be installed in `fluentd` image.

```dockerfile
FROM fluent/fluentd:latest

RUN apk add --no-cache --update --virtual .build-deps \
        sudo build-base ruby-dev \
 && sudo gem install fluent-plugin-splunk-hec \
 && sudo gem sources --clear-all \
 && apk del .build-deps \
 && rm -rf /tmp/* /var/tmp/* /usr/lib/ruby/gems/*/cache/*.gem

COPY fluent.conf /fluentd/etc/

EXPOSE 24224
```

## Instructions

1. Start splunk container

  ```bash
docker-compose up splunk
```

2. Start fluentd container

  ```bash
docker-compose up fluentd
```

3. Start logger containers

  ```bash
docker-compose up logger
```

4. Login to splunk UI at `http://localhost:8000` with user `admin` and password `99E16DCD-E064-4D74-BBDA-E88CE902F600`. Search with query `index=*` to see the logs being forward from `fluentd`.
