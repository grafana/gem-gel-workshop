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

**Mac Users:** Choose the `darwin` release.

If you have a newer M1 Mac, you will want the `arm64` version. If not, then the `amd64` version.

### **UNIX-based steps (Linux, Mac)**

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

**Mac Users:** If you want your filesystem metrics to work, you'll need to add two extra lines to the very end of the file, like so:

```
integrations:
  agent:
    enabled: true
  node_exporter:
    enabled: true
    filesystem_ignored_mount_points: "^/(dev)($|/)"
    filesystem_ignored_fs_types: "^devfs$"
```

This is because of a bug in the Grafana Agent that has since been fixed, but not yet released.

####
So, overall this is an extremely basic configuration that does the following things:

- Sets a port for the agent (12345 is not mandated, it can be anything you want)
- Sets a temporary directory for the Prometheus write-ahead log
- Sets a default scrape interval for metrics
- Sets the endpoint URL and credentials for writing metrics
- Enables the `agent` and `node_exporter` Agent integrations, this automatically configures and enables various metrics to be sent from your system, it is similar to manually configuring a Prometheus `node_exporter` to be scraped, but without the extra hassle. You can read more about Agent integration settings [here](https://grafana.com/docs/agent/latest/configuration/integrations/)

Next, start the Grafana Agent (use the correct binary name for your system):

`./agent-linux-amd64 -config.file agent-config.yaml`

**Mac Users:** If you are prevented from running the Grafana Agent because there is no developer signature, you will need to go to System Preferences -> Security & Privacy and then allow your Mac to run it.

You should see lots of logs fly by. If you see any obvious errors, let your lab lead know.

If not, move on to the verifying step (skip the Windows step).

### **Windows-based steps**

Download the [Windows installer of the Grafana Agent](https://github.com/grafana/agent/releases/download/v0.18.2/grafana-agent-installer.exe).

Next, run the executable to install it.

The installer by default installs the agent into `C:\Program Files\Grafana Agent` and also adds it as a Windows service.

Next, edit the default configuration file found at `C:\Program Files\Grafana Agent\agent-config.yaml`

It should, by default contain everything you need to grab Windows metrics, except that it doesn't have the necessary block to send the data anywhere. Edit the file to look like this instead:

```
server:
  http_listen_port: 12345
prometheus:
  wal_directory: $APPDATA\grafana-agent-wal
  global:
    scrape_interval: 15s
    remote_write:
      - url: https://<gem-url-provided-by-lab-lead>/api/v1/push
        basic_auth:
          username: <my-gem-instance-tenant-name>
          password: <my-gem-api-token>
  configs:
    - name: integrations
integrations:
  agent:
    enabled: true
  windows_exporter:
    enabled: true
```
This is an extremely basic configuration that does the following things:

- Sets a port for the agent (12345 is not mandated, it can be anything you want)
- Sets a temporary directory for the Prometheus write-ahead log
- Sets a default scrape interval for metrics
- Sets the endpoint URL and credentials for writing metrics
- Enables the `agent` and `windows_exporter` Agent integrations, this automatically configures and enables various metrics to be sent from your system, it is similar to manually configuring a Prometheus `windows_exporter` to be scraped, but without the extra hassle. You can read more about Agent integration settings [here](https://grafana.com/docs/agent/latest/configuration/integrations/)

**Note:** If you happened to examine both the non-Windows and Windows config files, you might have noticed that they are almost entirely identical, except that:
- `node_exporter` is traded for `windows_exporter`
- The `wal_directory` uses a Windows-style path variable

It's helpful to have similar experiences across operating systems!

Once you have edited the `agent-config.yaml` in Windows, restart the Grafana Agent service for the changes to take effect.

Check the logs for the service to ensure you don't see any obvious errors - if you do, let your lab lead know.

---
## Verifying your metrics have made it to the Grafana Enterprise Metrics cluster

Click the `Explore` icon in the left navigation menu - it looks like a compass.

At the top of the page, open the dropdown to change the selected data source.

Select your data source e.g. `GEM-Workshop` or whatever you named it.

Note that instead of `(No metrics found)` you should now see the words `Metrics browser`, click that.

You should now see a ton of metrics and labels - if so, then everything is good!

## Visualizing your metrics on a Grafana Dashboard

For this step, you'll be importing one of two dashboards depending on whether you configured the Grafana Agent on a UNIX-based machine (Mac, Linux) or a Windows machine.

In your Grafana instance, hover over the `+` icon in the left-hand navigation menu.

Click `Import`.

In the `Import via grafana.com` field, enter a number depending on what operating system the machine you configured the Grafana Agent on your machine used.

- For Linux: Enter `10242`

- For Mac: Enter `14866`

- For Windows: Enter `14694`

- Click `Load`

- Rename the dashboard if you would like and set a folder

- In the `Prometheus` data source dropdown, select the datasource you added in `Lab 1` e.g. `GEM-Workshop`

- Click `Import`

You should be presented with your newly imported dashboard showing metrics from your machine.

These metrics are being sent via the Grafana Agent, to your instance/tenant in a shared, distributed Grafana Enterprise Metrics cluster running in microservices mode on Kubernetes and read back out via Grafana!

**To reiterate:** All of this can be done on-premise in your company's own infrastructure, or your company's public cloud capacity - we are using Grafana Cloud's hosted Grafana offering purely for convenience here.

This concludes Lab 2!

Great job :)
