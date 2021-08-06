# Lab 1 - Connecting Grafana to your personal Grafana Enterprise Metrics and Grafana Enterprise Logs instances

In this lab, you will be:
- Logging into your Grafana instance on Grafana Cloud
- Adding a new datasource to connect to your personal instance/tenant in the shared Grafana Enterprise **_Metrics_** (GEM) cluster
- Adding a new datasource to connect to your personal instance/tenant in the shared Grafana Enterprise **_Logs_** (GEL) cluster
- Verifying that connectivity
- Verifying there are currently no metrics or logs in those instances

---

## Logging into Grafana
Firstly, navigate to Grafana Cloud and login at https://grafana.com/login

(If you did not sign up in advance of the workshop, it only takes a moment. Click the [`Register`](https://grafana.com/signup) button on that page.)

Once logged in, you should be presented with the `Welcome to Grafana` dashboard.

Click the cog/Configuration menu button on the left bar and select `Data sources`.

You should see a relatively low number of existing data sources, unless you've previously been using Grafana Cloud.

---


## Connecting to the Grafana Enterprise Metrics (GEM) cluster

Firstly, we'll be adding the data source that will connect to your instance/tenant in the shared Grafana Enterprise Metrics cluster.

Click `Add data source`.

Select `Prometheus` as the data source type. Grafana Enterprise Metrics instances/tenants behave just like Prometheus from Grafana's perspective, so there is no special data source type needed.

On the data source configuration page, you can use whatever name you wish, something like `GEM-Workshop` might be appropriate.

For the `URL`, your workshop or lab lead will provide you with the URL directly in your chat.

Ensure it starts with `https://` and ends with `/prometheus`.

Select the `Basic auth` option.

Under `Basic Auth Details`:
- `User` will be the name of your GEM instance/tenant you got from the shared document.
- `Password` will be the API key also found on the document.

Everything else can be left as-is.

Click `Save & test` at the bottom.

You should see a `Data source is working` message. If you don't, let your lab lead know.

Your data source should now be ready for querying!

---

## Verifying the GEM instance/tenant is working and empty

Click the `Explore` icon in the left navigation menu - it looks like a compass.

At the top of the page, open the dropdown to change the selected data source.

Select your data source e.g. `GEM-Workshop` or whatever you named it.

If you see `(No metrics found)` on the left of the PromQL query box, then everything is good!

Since you have not sent any metrics to this instance/tenant yet, this is expected.

This also lets us know that general querying should be working.

---

## Connecting to the Grafana Enterprise Logs (GEL) cluster

These steps will be extremely similar to the ones for GEM.

Firstly, we'll be adding the data source that will connect to your instance/tenant in the shared Grafana Enterprise Logs cluster.

Click the cog/Configuration menu button on the left bar and select `Data sources`.

Click `Add data source`.

Select `Loki` as the data source type. Grafana Enterprise Logs instances/tenants behave just like Loki from Grafana's perspective, so there is no special data source type needed.

On the data source configuration page, you can use whatever name you wish, something like `GEL-Workshop` might be appropriate.

For the `URL`, your workshop or lab lead will provide you with the URL directly in your chat.

Ensure it starts with `https://`.

Select the `Basic auth` option.

Under `Basic Auth Details`:
- `User` will be the name of your GEL instance/tenant you got from the shared document.
- `Password` will be the API key also found on the document.

Everything else can be left as-is.

Click `Save & test` at the bottom.

You should see a `Data source connected and labels found.` message. If you don't, let your lab lead know.

Your data source should now be ready for querying!

---

## Verifying the GEL instance/tenant is working and empty

Click the `Explore` icon in the left navigation menu - it looks like a compass.

At the top of the page, open the dropdown to change the selected data source.

Select your data source e.g. `GEL-Workshop` or whatever you named it.

Click the `Log browser` button to expand the log browser.

If you see only a label named `__name__` then everything is good!

Since you have not sent any logs to this instance/tenant yet, this is expected.

This also lets us know that general querying should be working.

---

This concludes Lab 1!

Nice job :)