## Introduction

This example will show how to sent batches of log events to S3 bucket.

`fluent-plugin-s3` is used for this purpose. This plug-in uses environment variable to pass in AWS credentials. (`AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`) It defines `s3_object_key_format` to customize object keys in S3 bucket. Furthermore, we enable S3 server-side encryption and file compression in this plug-in.

```xml
<source>
  @type forward
  port 24224
</source>

<match **>
  @type s3
  s3_bucket "#{ENV['S3_BUCKET']}"
  s3_region "#{ENV['S3_REGION']}"
  path ${tag}/
  s3_object_key_format %{path}%{time_slice}_%{index}.%{file_extension}
  use_server_side_encryption AES256
  store_as gzip_command

  s3_endpoint http://s3mock:9090/ # s3mock server
  force_path_style true           # s3mock server

  <buffer tag,time>
    @type file
    path /var/log/fluent/s3
    timekey 10
    timekey_wait 0s
  </buffer>
</match>
```

`fluent-plugin-s3` will need to be installed in `fluentd` image.

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
