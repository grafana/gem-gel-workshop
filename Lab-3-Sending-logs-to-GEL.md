# Lab 3 - Sending logs to your Grafana Enterprise Logs instance/tenant and visualizing them in your Grafana instance

In this lab, you will be:
- Modifying your configuration file for the Grafana Agent that will allow it to also send logs from your machine to your personal instance/tenant in the shared Grafana Enterprise Logs cluster
- Running the Grafana Agent with the updated configuration file
- Verifying that the logs are arriving at the destination as expected
- Visualizing some of the logs on a Grafana dashboard

---

## Grafana Agent further information

As previously mentioned, the Grafana Agent is a lightweight component that embeds a lot of Prometheus code. However, in addition to that, it embeds the [Promtail](https://grafana.com/docs/loki/latest/clients/promtail/) binary which is another agent that can ship the content of log files to a Loki, Grafana Cloud Logs or Grafana Enterprise Logs instance/tenant.

While you can run `Promtail` as a standalone component, the Grafana Agent allows this to be done in an easier all-in-one fashion with less configuration needed than running both separately.

Just like with metrics, the exact steps can vary by operating system, but the biggest difference is if you are installing on Windows versus a UNIX-based system. Both will be covered here.

---

## Modifying your Grafana Agent configuration file to include logs

### **UNIX-based steps (Linux, Mac)**

Edit your `agent-config.yaml` from before to look something like the following:

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
loki:
  positions_directory: /tmp
  configs:
  - name: default
    scrape_configs:
      - job_name: varlogs
        static_configs:
          - targets: [localhost]
            labels:
              job: varlogs
              __path__: /var/log/*log
    clients:
      - url: https://<gel-url-provided-by-lab-lead>/loki/api/v1/push
        basic_auth:
          username: <my-gel-instance-tenant-name>
          password: <my-gel-api-token>
integrations:
  agent:
    enabled: true
  node_exporter:
    enabled: true
```
####
Note that there is a whole `loki` block inserted into the middle.

This short enhancement to your existing configuration does the following things:

- Creates a named config
- Sets a promtail `positions` filename location
- Sets a default static scrape config for logs that just grabs all files ending in `log` in the `/var/log` directory on the filesystem
- Sets the endpoint URL and credentials for writing logs

Next, restart the Grafana Agent (use the correct binary name for your system):

`./agent-linux-amd64 -config.file agent-config.yaml`

You should see lots of logs fly by. If you see any obvious errors, let your lab lead know.

**Note:** It's okay if you see a few `permission denied` errors for some logs, we don't mind too much if we don't get all the logs for this workshop, just that we get some.

If not, move on to the verifying step (skip the Windows step).

### **Windows-based steps**

Edit your `agent-config.yaml` from before to look something like the following:

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
loki:
  # This directory needs to already exist 
  positions_directory: "C:\\path\\to\\directory"
  configs:
  - name: windows
    scrape_configs:
      - job_name: windows
        windows_events:
          # Note the directory structure must already exist but the file will be created on demand
          bookmark_path: "C:\\path\\to\\bookmark\\directory\\bookmark.xml"
          use_incoming_timestamp: false
          eventlog_name: "Application"
          # Filter for logs
          xpath_query: '*'
          labels:
            job: windows
    clients:
      - url: https://<gel-url-provided-by-lab-lead>/loki/api/v1/push
        basic_auth:
          username: <my-gel-instance-tenant-name>
          password: <my-gel-api-token>
integrations:
  agent:
    enabled: true
  windows_exporter:
    enabled: true
```

Note that there is a whole `loki` block inserted into the middle.

This short enhancement to your existing configuration does the following things:

- Creates a named config
- Sets a promtail `positions` filename location
- Sets a default static scrape config for logs that grabs your Windwos Event Logs `Application` events and writes them as logs
- Sets the endpoint URL and credentials for writing logs

**Note:** The two directory paths - `positions_directory` and `bookmark_path` need to exist, they will not be automatically created.

Once you have edited the `agent-config.yaml` in Windows, restart the Grafana Agent service for the changes to take effect.

**Note:** You will need to run the Grafana Agent as Administrator for it to have easy permission to access your system logs. If you aren't comfortable with this, that's totally fine, feel free to skip the step.

Check the logs for the service to ensure you don't see any obvious errors - if you do, let your lab lead know.

As the particular Windows log being tailed may not be very chatty, it might be good to force at least one event to the log. You can do this with the following command:

`eventcreate /T ERROR /ID 500 /L APPLICATION /D "hello yes this is a forced log event"`

---
## Verifying your logs have made it to the Grafana Enterprise Logs cluster

Click the `Explore` icon in the left navigation menu - it looks like a compass.

At the top of the page, open the dropdown to change the selected data source.

Select your data source e.g. `GEL-Workshop` or whatever you named it.

Click the `Log browser` button to expand the log browser.

Instead of only seeing a label named `__name__` you should also see `job` and `filename`.

You should also see that `job` has a value of either `varlogs` or `windows` depending on your configuration.

If you do, then everything is good!

## Visualizing your logs on a Grafana Dashboard

For this step, you'll be importing one of two dashboards depending on whether you configured the Grafana Agent on a UNIX-based machine (Mac, Linux) or a Windows machine.

### **Importing a dashboard for Linux/Mac**

In your Grafana instance, hover over the `+` icon in the left-hand navigation menu.

Click `Import`.

In the `Import via grafana.com` field, enter `13639`

- Click `Load`

- Rename the dashboard if you would like and set a folder

- In the `Loki` data source dropdown, select the datasource you added in `Lab 1` e.g. `GEL-Workshop`

- Click `Import`

You should be presented with your newly imported dashboard showing a log rate timeline and logs from your machine.

These logs are being sent via the Grafana Agent, to your instance/tenant in a shared, distributed Grafana Enterprise Logs cluster running in single-binary mode on Kubernetes and read back out via Grafana!

**To reiterate:** All of this can be done on-premise in your company's own infrastructure, or your company's public cloud capacity - we are using Grafana Cloud's hosted Grafana offering purely for convenience here.

This concludes Lab 3!

Super job :)
