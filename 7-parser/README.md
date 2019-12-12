## Introduction

This example will show how to parse a specific filed in event records and modify its records with the parsed result.

`fluent_parser` is used to parse the user's custom data format, and it is in Fluentd's core and requires no installation. It allows to use built-in parser plugins or your own customized parser plugin.

In the following config, we use the `regexp` built-in parsers to retrieve key-value pairs in the `log` filed.

```xml
<source>
  @type forward
  port 24224
</source>

<filter **>
    @type parser
    key_name log
    reserve_data false
    remove_key_name_field true
    <parse>
        @type regexp
        expression /^\[(?<logTime>.+)\]\s+provider=(?<provider>\S+),\s+item=(?<item>\S+),\s+merchant=(?<merchant>\S+)\s+$/
        time_key logTime
        time_format %a %d %b %Y %T,000 %z
    </parse>
</filter>

<match **>
  @type stdout
</match>
```

- If following record is passed:
```json
{
  "container_id":"88888",
  "container_name":"/7-parser_logger",
  "source":"stdout",
  "log":"[Thu 12 Dec 2019 03:42:02,000 +0000] provider=provide1, item=item1, merchant=merchant1 "
}
```
- then you got the new record like below:
```json
{
  "provider":"provide1",
  "item":"item1",
  "merchant":"merchant1"
}
```

The following config uses another built-in parser `ltsv`. It will get the same results as the previous one, but you can add any key-value pairs without modifying the config.
You can assign your own delimiter pattern and the character between your key and value to parse your data.

```xml
<source>
  @type forward
  port 24224
</source>

<filter **>
    @type parser
    key_name log
    reserve_data false
    remove_key_name_field true
    <parse>
        @type regexp
        expression /^\[(?<logTime>.+)\]\s+(?<logMessage>.+)$/
        time_key logTime
        time_format %a %d %b %Y %T,000 %z
    </parse>
</filter>

<filter **>
    @type parser
    key_name logMessage
    remove_key_name_field true
    <parse>
        @type ltsv
        delimiter_pattern /,\s+/
        label_delimiter =
    </parse>
</filter>

<match **>
  @type stdout
</match>

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
