# Grafana Monitoring Stack Example

This example demonstrates a simple microservice application that exposes Prometheus metrics, used to validate and demonstrate the Grafana + Prometheus + Loki stack. This was demonstrated in a talk at the FLISOL 2021 - DevParana + UniALFA Umuarama event, [the recording can be seen here](https://youtu.be/vv3LlJlR2k0?t=8893) (in portuguese).

## Application stack

There are four "clock" microservices in this example, written in NodeJS. 
- `seconds`: Returns the current time seconds
- `minutes`: Returns the current time minutes
- `hours`: Returns the current time hours
- `controller`: Calls the three microservices above, and returns a string in the format: "HH:MM:SS"

To make things more interesting, the microservices randomly return a 5XX error. The controller handles this, and uses a "xx" in the corresponding time slot before returning the string (eg. "12:xx:59").

The library `express-prometheus-middleware` is used to export the application metrics in Prometheus format. 

All the applications share the same Dockerfile, but each build to their own Docker image, with the source code and dependencies installed in the image.

## Monitoring stack

For simplicity and ease to reproduce, the monitoring stack is deployed along with the application, in the same Compose file. Ideally they should be separated, specially Loki that is the log source all applications will send their logs to. This is circumvented by making all containers depend on `loki`, so it is the first one to go up.

Grafana is deployed with provisioning enabled, with a config and volume to the repository's `grafana/dashboards` folder. It will also configure Prometheus and Loki as datasources automatically.

Prometheus is deployed in the stack, with a simple static config to scrape all of the applications metrics.

## Configuring

Before you send logs to Loki, you will need to add the necessary docker plugin with the following command:

```sh
docker plugin install grafana/loki-docker-driver:latest --alias loki --grant-all-permissions
```

After that, you need to build the Docker images:

```sh
docker-compose build
```

And you're good to go

## Running

As simple as doing:
```sh
docker-compose up -d
```

## Testing

The stack will be deployed with the following endpoints:
- http://localhost:8080/v1/current-time - Controller App, access this to send data to Prometheus/Loki
- http://localhost:3000 - Grafana (user: `admin`, pass: `admin`)
- http://localhost:9090 - Prometheus

The "time" applications are not exposed since they are only accessed from the Controller App, but you might change the port exposing in the `docker-compose.yaml` file if you wish.