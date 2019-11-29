## Introduction

```xml
<source>
  @type forward
  port 24224
</source>

<match **>
  @type splunk_hec
  hec_host splunk
  hec_port 8088
  hec_token B5A79AAD-D822-46CC-80D1-819F80D7BFB0
  source_key service_name
  insecure_ssl true
  <buffer>
    flush_interval 1s
  </buffer>
</match>
```

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

4. Login to splunk UI at `http://localhost:8000` with user `admin` and password `99E16DCD-E064-4D74-BBDA-E88CE902F600`.

5. Search with query `index=*` to see the logs being forward from `fluentd`.
