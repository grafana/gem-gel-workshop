# Lab 2 - Sending metrics to your Grafana Enterprise Metrics instance/tenant and visualizing them in your Grafana instance

In this lab, you will be:
- Downloading the Grafana Agent
- Setting up a configuration file for the Grafana Agent that will allow it to send metrics from your machine to your personal instance/tenant in the shared Grafana Enterprise Metrics cluster
- Running the Grafana Agent with that configuration file
- Verifying that the metrics are arriving at the destination as expected
- Visualizing some of the metrics on a Grafana dashboard

---

## Grafana Agent background
Some information on the [Grafana Agent](https://github.com/grafana/agent/).

The Grafana Agent is a lightweight component that embeds a lot of Prometheus code, allowing it to use exactly the same service discovery and scraping methods as Prometheus, without the local time-series database or querying capabilities. It instead is designed to send all data via the `remote_write` protocol to compatible endpoints like Prometheus, Cortex, Grafana Enterprise Metrics and so on.

It is compatible with lots of systems including Linux (amd64, armv6, armv7 arm64), Mac (darwin amd64 and darwin arm64), FreeBSD and Windows.

For this lab, you can install it on whatever compatible system you like, including your local machine, as long as it has access to send data to the necessary endpoints over http on port 443.

The exact steps can vary by operating system, but the biggest difference is if you are installing on Windows versus a UNIX-based system. Both will be covered here.

The agent is also extremely configurable. The examples given in this guide will keep things as simple as possible for speed reasons but just know that the [main Grafana Agent documentation](https://grafana.com/docs/agent/latest/) contains far more options if they are needed.

---

## Downloading, installing and configuring the Grafana Agent

From the [Grafana Agent releases page](https://github.com/grafana/agent/releases), download the relevant binary for your operating system.

### UNIX-based steps (Linux, Mac)

Extract the binary:

`unzip agent-*.zip`

Make sure the binary is executable:

`chmod a+x agent-*`

Place the following text into a config file named `agent-config.yaml` in the same directory as the binary:

```
server:
  log_level: info
  http_listen_port: 12345
prometheus:
  wal_directory: /tmp/wal
  global:
    scrape_interval: 15s
    remote_write:
      - url: https://<gem-url-provided-by-lab-lead>/api/v1/push
        basic_auth:
          username: <my-gem-instance-tenant-name>
          password: <my-gem-api-token>

integrations:
  agent:
    enabled: true
  node_exporter:
    enabled: true
```
####
This is an extremely basic configuration that does the following things:

- Sets a standard logging level
- Sets a port for the agent (12345 is not mandated, it can be anything you want)
- Sets a temporary directory for the Promeheus write-ahead log
- Sets a default scrape interval for metrics
- Sets the endpoint URL and credentials for writing metrics
- Enables the `agent` and `node_exporter` Agent integrations, this automatically configures and enables various metrics to be sent from your system, it is similar to manually configuring a Prometheus `node_exporter` to be scraped, but without the extra hassle. You can read more about Agent integration settings [here](https://grafana.com/docs/agent/latest/configuration/integrations/)

Next, start the Grafana Agent (use the correct binary name for your system):

`./agent-linux-amd64 -config.file agent-config.yaml`

You should see lots of logs fly by. If you see any obvious errors, let your lab lead know.

If not, move on to the verifying step (skip the Windows step).

### Windows-based steps
