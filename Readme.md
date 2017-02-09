# logspout-logstash

Dockerfile and modules.go file to package [Logspout](https://github.com/gliderlabs/logspout) with all the builtin modules together with the
minimalistic [looplab/logspout-logstash](https://github.com/looplab/logspout-logstash) module to send Docker container logs to [Logstash](https://github.com/elastic/logstash) via UDP or TCP.

The image is based on the [instructions](https://github.com/gliderlabs/logspout/tree/master/custom)

## Example

As container on single host
```
docker run -d --name logspout --publish 8080:80 --volume "/var/run/docker.sock:/var/run/docker.sock" dsadockuser/logspout-logstash
```

As docker service running on all hosts
```
```

You can verify logspout running and the gathering of the logs by using curl
```
curl http://localhost:8080/logs
```

"The DevOps Toolkit 2.1: Docker Swarm" by Viktor Farcic readers could use
```
docker service create --name logspout \
    --network elk \
    --mode global \
    --mount "type=bind,source=/var/run/docker.sock,target=/var/run/docker.sock" \
    -e SYSLOG_FORMAT=rfc3164 \
    dsadockuser/logspout-logstash logstash+tcp://logstash:51416
```


In your logstash config, set the input codec to `json` e.g:

```bash
input {
  udp {
    port  => 51416
    codec => json
  }
  tcp {
    port  => 51416
    codec => json
  }
}
```

## Available configuration options

For example, to get into the Logstash event's @tags field, use the ```LOGSTASH_TAGS``` container environment variable. Multiple tags can be passed by using comma-separated values

```bash
  # Add any number of arbitrary tags to your event
  -e LOGSTASH_TAGS="docker,production"
```

The output into logstash should be like:

```json
    "tags": [
      "docker",
      "production"
    ],
```

You can also add arbitrary logstash fields to the event using the ```LOGSTASH_FIELDS``` container environment variable:

```bash
  # Add any number of arbitrary fields to your event
  -e LOGSTASH_FIELDS="myfield=something,anotherfield=something_else"
```

The output into logstash should be like:

```json
    "myfield": "something",
    "another_field": "something_else",
```

Both configuration options can be set for every individual container, or for the logspout-logstash
container itself where they then become a default for all containers if not overridden there.

This table shows all available configurations:

| Environment Variable | Input Type | Default Value |
|----------------------|------------|---------------|
| LOGSTASH_TAGS        | array      | None          |
| LOGSTASH_FIELDS      | map        | None          |
