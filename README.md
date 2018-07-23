## Schema Registry UI ##
[![](https://images.microbadger.com/badges/image/lozanomatheus/schema-registry-ui:0.9.4.svg)](https://microbadger.com/images/lozanomatheus/schema-registry-ui:0.9.4")

This is a __non-official__ and small docker image for __Landoop's schema-registry-ui__ with multiple environment.
It will create a Jenkins Pipeline do delivery the schema-registry-ui in Mesos/Marathon.
It serves the schema-registry-ui from port 8000 by default.
A live version can be found at <https://schema-registry-ui.landoop.com>

The software is stateless and the only necessary option is your Schema Registry
URL.

To run it:

```bash
docker run --rm -p 8000:8000 \
  -e "NameFirst=QA-AWS" \
  -e "UrlFirst=http://schema-registry.qa-aws.example:8081" \
  -e "ColorFirst=black" \
  -e "NameSecond=QA-GCP" \
  -e "UrlSecond=http://schema-registry.qa-gcp.example:8081" \
  -e "ColorSecond=blue" \
  -e "Env=QA" \
    lozanomatheus/schema-registry-ui:0.9.4
```

Visit http://localhost:8000 to see the UI.

### Advanced Settings

Three of the Schema Registry UI settings need to be enabled explicitly. These
are:

1. Support for global compatibility level configuration support —i.e change the
   default compatibility level of your schema registry.
2. Support for transitive compatibility levels (Schema Registry version 3.1.1 or better).
3. Support for Schema deletion (Schema Registry version 3.3.0 or better).

They are handled by the `ALLOW_GLOBAL`, `ALLOW_TRANSITIVE` and `ALLOW_DELETION`
environment variables. E.g:

```bash
docker run --rm -p 8000:8000 \
  -e "NameFirst=QA-AWS" \
  -e "UrlFirst=http://schema-registry.qa-aws.example:8081" \
  -e "ColorFirst=black" \
  -e "NameSecond=QA-GCP" \
  -e "UrlSecond=http://schema-registry.qa-gcp.example:8081" \
  -e "ColorSecond=blue" \
  -e "Env=QA" \
  -e "ALLOW_GLOBAL=1" \
  -e "ALLOW_TRANSITIVE=1" \
  -e "ALLOW_DELETION=1" \
    lozanomatheus/schema-registry-ui:0.9.4
```

### Proxying Schema Registry

If you have CORS issues or want to pass through firewalls and maybe share your
server, we added the `PROXY` option. Run the container with `-e PROXY=true` and
Caddy server will proxy the traffic to Schema Registry:

```bash
docker run --rm -p 8000:8000 \
  -e "NameFirst=QA-AWS" \
  -e "UrlFirst=http://schema-registry.qa-aws.example:8081" \
  -e "ColorFirst=black" \
  -e "NameSecond=QA-GCP" \
  -e "UrlSecond=http://schema-registry.qa-gcp.example:8081" \
  -e "ColorSecond=blue" \
  -e "Env=QA" \
  -e "PROXY=true" \
    lozanomatheus/schema-registry-ui:0.9.4
```

> **Important**: When proxying, for the `SCHEMAREGISTRY_URL` you have to use an
> IP address or a domain that can be resolved to it. **You can't use**
> `localhost` even if you serve Schema Registry from your localhost. The reason
> for this is that a docker container has its own network, so your _localhost_
> is different from the container's _localhost_. As an example, if you are in
> your home network and have an IP address of `192.168.5.65` and run Schema
> Registry from your computer, instead of `http://127.0.1:8082` you must use
> `http://192.168.5.65:8082`.

If your Schema Registry uses self-signed SSL certificates, you can use the
`PROXY_SKIP_VERIFY=true` environment variable to instruct the proxy to
not verify the backend TLS certificate.

## Configuration options

### Schema Registry UI

You can control most of Kafka Topics UI settings via environment variables:

 * `SCHEMAREGISTRY_URL`
 * `ALLOW_GLOBAL=[true|false]` (default false)
 * `ALLOW_TRANSITIVE=[true|false]` (default false)
 * `ALLOW_DELETION=[true|false]` (default false).

## Docker Options

- `PROXY=[true|false]`
  
  Whether to proxy Schema Registry endpoint via the internal webserver
- `PROXY_SKIP_VERIFY=[true|false]`
  
  Whether to accept self-signed certificates when proxying Schema Registry
  via https
- `PORT=[PORT]`
  
  The port number to use for schema-registry-ui. The default is `8000`.
  Usually the main reason for using this is when you run the
  container with `--net=host`, where you can't use docker's publish
  flag (`-p HOST_PORT:8000`).
- `CADDY_OPTIONS=[OPTIONS]`
  
  The webserver that powers the image is Caddy. Via this variable
  you can add options that will be appended to its configuration
  (Caddyfile). Variables than span multiple lines are supported.
  
  As an example, you can set Caddy to not apply timeouts via:
  
      -e "CADDY_OPTIONS=timeouts none"
  
  Or you can set basic authentication via:
  
      -e "CADDY_OPTIONS=basicauth / [USER] [PASS]"

# Schema Registry Configuration

If you don't wish to proxy Schema Registry's api, you should permit CORS via setting
`access.control.allow.methods=GET,POST,PUT,DELETE,OPTIONS` and
`access.control.allow.origin=*`.

# Logging

In the latest iterations, the container will print informational messages during
startup at stderr and web server logs at stdout. This way you may sent the logs
(stdout) to your favorite log management solution.
