## Introduction

```xml
<source>
  @type forward
  port 24224
</source>

<match **>
  @type s3
  aws_key_id YOUR_ACCESS_KEY_ID
  aws_sec_key YOUR_SECRET_ACCESS_KEY
  s3_endpoint http://s3mock:9090/
  force_path_style true # s3mock server

  s3_bucket logBucket
  path ${tag}/
  s3_object_key_format %{path}%{time_slice}_%{index}.%{file_extension}

  <buffer tag,time>
    @type file
    path /var/log/fluent/s3
    timekey 10
    timekey_wait 0s
  </buffer>
</match>
```

```dockerfile
FROM fluent/fluentd:latest

RUN apk add --no-cache --update --virtual .build-deps \
        sudo build-base ruby-dev \
 && sudo gem install fluent-plugin-s3 \
 && sudo gem sources --clear-all \
 && apk del .build-deps \
 && rm -rf /tmp/* /var/tmp/* /usr/lib/ruby/gems/*/cache/*.gem

COPY fluent.conf /fluentd/etc/

EXPOSE 24224
```

## Instructions

1. Start s3mock container

```bash
docker-compose up s3mock
```

2. Start fluentd container

```bash
docker-compose up fluentd
```

3. Start logger containers

```bash
docker-compose up logger
```

4. Check the content of S3 bucket to see the logs being forward from `fluentd`.

```bash
curl http://localhost:9090/logBucket | xmllint --format -
```
