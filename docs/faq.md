# FAQ

## How can I contact the developers?

- [Community forum](https://www.percona.com/forums/questions-discussions/percona-monitoring-and-management).
- [Discord chat](http://per.co.na/discord).
- [PMM project in JIRA](https://jira.percona.com/projects/PMM).

## What are the minimum system requirements?

- Server:
    - Disk: 1 GB per monitored database (1 week data retention)
    - Memory: 2 GB per monitored database
    - CPU: Supports [`SSE4.2`](https://wikipedia.org/wiki/SSE4#SSE4.2)
- Client:
    - Disk: 100 MB

!!! seealso alert alert-info "See also"
    - [Setting up PMM Server](setting-up/server/index.md)
    - [Setting up PMM Client](setting-up/client/index.md)

## How can I upgrade from version 1?

There is no direct software upgrade path.

You must [set up](setting-up/index.md) PMM 2 and connect your existing clients to it.

When all data is registered in PMM2 and expired in PMM1, decommission your PMM1 instance.

!!! seealso alert alert-info "See also"
    - [Upgrade from PMM1](how-to/upgrade.md#upgrade-from-pmm-1)
    - [Percona blog: Running PMM1 and PMM2 Clients on the Same Host](https://www.percona.com/blog/2019/11/27/running-pmm1-and-pmm2-clients-on-the-same-host/)

## How to control data retention?

Go to <i class="uil uil-cog"></i> *Configuration* → <i class="uil uil-setting"></i> *Settings* → *Advanced Settings* → *Data retention* to adjust the value in days.

!!! seealso alert alert-info "See also"
    [Configure data retention](how-to/configure.md#data-retention)

## How often are NGINX logs rotated?

Daily.

PMM Server runs `logrotate` daily to rotate NGINX logs, keeping up to ten of the most recent log files.

## What privileges are required to monitor a MySQL instance?

```sql
SELECT, PROCESS, SUPER, REPLICATION CLIENT, RELOAD
```

!!! seealso alert alert-info "See also"
    [Setting Up/Client/MySQL](setting-up/client/mysql.md#create-a-database-account-for-pmm).

## Can I monitor multiple service instances?

Yes.

You can add multiple instances of MySQL or any other service to be monitored from the same PMM Client.

To do this, you provide a unique port and IP address, or a socket for each instance, and specify a unique name for each. (If a name is not provided, PMM uses the name of the PMM Client host.)

For example, to add MySQL monitoring for two local MySQL servers:

```sh
pmm-admin add mysql --username root --password root instance-01 127.0.0.1:3001
pmm-admin add mysql --username root --password root instance-02 127.0.0.1:3002
```

!!! seealso alert alert-info "See also"
    [`pmm-admin add mysql`](details/commands/pmm-admin.md#mysql)

## Can I rename instances?

Yes, by removing and re-adding with a different name.

When you remove a monitoring service, previously collected data remains available in Grafana.  However, the metrics are tied to the instance name.  So if you add the same instance back with a different name, it will be considered a new instance with a new set of metrics.  So if you are re-adding an instance and want to keep its previous data, add it with the same name.

## Can I add an AWS RDS MySQL or Aurora MySQL instance from a non-default AWS partition?

By default, the RDS discovery works with the default `aws` partition. But you can switch to special regions, like the [GovCloud](https://aws.amazon.com/govcloud-us/) one, with the alternative [AWS partitions](https://docs.aws.amazon.com/sdk-for-go/api/aws/endpoints/#pkg-constants) (e.g. `aws-us-gov`) adding them to the *Settings* via the PMM Server [API](details/api.md).

![!image](_images/aws-partitions-in-api.png)

To specify other than the default value, or to use several, use the JSON Array syntax: `["aws", "aws-cn"]`.

## How do I troubleshoot communication issues between PMM Client and PMM Server?

See [Troubleshoot PMM Server/PMM Client connection](how-to/troubleshoot.md#troubleshoot-connection).

## What resolution is used for metrics?

The default values (in seconds):

| Preset            | Low  | Medium | High |
|-------------------|------|--------|------|
| Rare              | 300  | 180    | 60   |
| Standard          | 60   | 10     | 5    |
| Frequent          | 30   | 5      | 1    |
| Custom (defaults) | 60   | 10     | 5    |

!!! seealso alert alert-info "See also"
    [Metrics resolution](how-to/configure.md#metrics-resolution)

## How do I set up Alerting?

When a monitored service metric reaches a defined threshold, PMM Server can trigger alerts for it either using the Grafana Alerting feature or by using an external alert manager.

With these methods you must configure alerting rules that define conditions under which an alert should be triggered, and the channel used to send the alert (e.g. email).

Alerting in Grafana allows attaching rules to your dashboard panels.  Grafana Alerts are already integrated into PMM Server and may be simpler to get set up.

Alertmanager allows the creation of more sophisticated alerting rules and can be easier to manage installations with a large number of hosts. This additional flexibility comes at the expense of simplicity.

We only offer support for creating custom rules to our customers, so you should already have a working Alertmanager instance prior to using this feature.

!!! seealso alert alert-info "See also"
    [PMM Alerting with Grafana: Working with Templated Dashboards](https://www.percona.com/blog/2017/02/02/pmm-alerting-with-grafana-working-with-templated-dashboards/)

## How do I use a custom Prometheus configuration file?

Normally, PMM Server fully manages the [Prometheus configuration file](https://prometheus.io/docs/prometheus/latest/configuration/configuration/).

However, some users may want to change the generated configuration to add additional scrape jobs, configure remote storage, etc.

From version 2.4.0, when `pmm-managed` starts the Prometheus file generation process, it tries to load the `/srv/prometheus/prometheus.base.yml` file first, to use it as a base for the `prometheus.yml` file.

The `prometheus.yml` file can be regenerated by restarting the PMM Server container, or by using the `SetSettings` API call with an empty body.

!!! seealso alert alert-info "See also"
    - [API](details/api.md)
    - [Percona blog: Extending PMM’s Prometheus Configuration](https://www.percona.com/blog/2020/03/23/extending-pmm-prometheus-configuration/)

## How to troubleshoot an Update?

See [Troubleshoot update](how-to/troubleshoot.md#update).

## What are my login credentials when I try to connect to a Prometheus Exporter?

- User name: `pmm`
- Password: Agent ID

PMM protects an exporter's output from unauthorized access by adding an authorization layer. To access an exporter you can use `pmm` as a user name and the Agent ID as a password. You can find the Agent ID corresponding to a given exporter by running `pmm-admin list`.

!!! seealso alert alert-info "See also"
    [`pmm-admin list`](details/commands/pmm-admin.md#information-commands)

## How to provision PMM Server with non-default admin password?

Currently there is no API available to change the `admin` password. If you're deploying through Docker you can use the following code snippet to change the password after starting the Docker container:

```sh
PMM_PASSWORD="mypassword"
echo "Waiting for PMM to initialize to set password..."
until [ "`docker inspect -f {% raw %}{{.State.Health.Status}}{% endraw %} pmm-server`" = "healthy" ]; do sleep 1; done
docker exec -t pmm-server bash -c  "grafana-cli --homepath /usr/share/grafana admin reset-admin-password $PMM_PASSWORD"
```

(This example assumes your Docker container is named `pmm-server`.)