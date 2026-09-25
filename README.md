# BWCE Greeting

A sample [TIBCO BusinessWorks Container Edition (BWCE)](https://www.tibco.com/products/tibco-businessworks) application that exposes a simple REST service returning a greeting for the name passed in.

The greeting response is rendered as an HTML status card that also reports the runtime host, the BusinessWorks engine version and the application version, which makes it a handy way to confirm which instance/node served a request when running scaled or containerized deployments.

## What it does

`GET /greeting/{name}` returns an HTML page containing:

- **Hello {name}** — echoing the name from the URL path
- **Host** — the hostname of the container/node that served the request (`bw:getHostName()`)
- **BW Version** — the BusinessWorks runtime engine version (`BW.VERSION`)
- **App Version** — the deployed application version (`BW.APPLICATION.VERSION`), or `Local / Development` when unset

Each request also writes an `Info` log message: `<host> Received a greeting for <name>`.

### Example

```
GET http://localhost:8081/greeting/World
```

Returns an HTML page (`Content-Type: text/html; charset=UTF-8`) with a card reading "Hello World" and a table of Host / BW Version / App Version.

## Project structure

| Path | Description |
| --- | --- |
| `greeting/` | The BWCE application project (packaging, deployment manifests, global variables). |
| `greeting.module/` | The application module containing the process, schemas, REST service descriptor and HTTP connector resource. |
| `greeting.module/Processes/greeting/module/greeting.bwp` | The BW process implementing the `get` operation. |
| `greeting.module/Service Descriptors/` | The generated Swagger 2.0 descriptor for the `/greeting/{name}` endpoint. |
| `greeting.module/Schemas/` | XSDs for the REST parameters and transport headers. |
| `Artifacts/` | Pre-built EAR (`greeting_1.0.0.ear`) and a deployment manifest. |

## Endpoint and configuration

- Public HTTP endpoint listens on port **8081** (`BW.CLOUD.PORT`), bound to `BW.HOST.NAME` (defaults to `localhost`).
- A private, pingable HTTP endpoint is exposed on port **8090** for health checks.
- Base path: `/`, resource path: `/greeting/{name}`.

These values are defined in `greeting.module/META-INF/default.substvar` and `greeting/manifest-bwce.json`.

## Building and running

This project is developed with TIBCO Business Studio for BusinessWorks (BWCE edition, built against `TIBCO-BW-Version: 2.4.1 V6 2018-10-03`).

### Run locally

Import both projects into TIBCO Business Studio and run the `greeting` application, then browse to:

```
http://localhost:8081/greeting/World
```

### Run the pre-built EAR in a container

Build a BWCE Docker image from the base image plus the EAR in `Artifacts/greeting_1.0.0.ear`, then run it, mapping the HTTP port:

```
docker run -p 8081:8081 <your-greeting-image>
```

### Deploy to Cloud Foundry

A Cloud Foundry manifest is provided (`greeting/manifest.yml`):

```
cf push
```

The app is named `greeting` with a 512M memory allocation.
