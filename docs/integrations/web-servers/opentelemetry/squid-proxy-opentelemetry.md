---
id: squid-proxy-opentelemetry
title: Squid Proxy - OpenTelemetry Collector
sidebar_label: Squid Proxy - OTel Collector
description: Learn about the Sumo Logic OpenTelemetry App for Squid Proxy.
---

import useBaseUrl from '@docusaurus/useBaseUrl';
import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

<img src={useBaseUrl('img/integrations/web-servers/squid-proxy.png')} alt="Thumbnail icon" width="75"/> <img src={useBaseUrl('img/send-data/otel-color.svg')} alt="Thumbnail icon" width="45"/>

The [Squid](http://www.squid-cache.org/Intro/) Proxy app is a logs app that helps you monitor activity in Squid Proxy. The preconfigured dashboards provide insight into served and denied requests; HTTP response codes; URLs experiencing redirects, client errors, and server errors; and quality of service data that helps you understand your users' experience.

Squid logs are sent to Sumo Logic through OpenTtelemetry [filelog receiver](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/receiver/filelogreceiver).

<img src={useBaseUrl('https://sumologic-app-data-v2.s3.amazonaws.com/dashboards/Squid-Proxy-OpenTelemetry/Squid-Proxy-Schematics.png')} alt="Schematics" />

## Prerequisites

By default, the squid proxy will write the access log to the log directory that was configured during installation. For example, on Linux, the default log directory is /var/log/squid/access.log. If the access log is disabled then you must enable the access log by following these [instructions](https://wiki.squid-cache.org/SquidFaq/SquidLogs).

## Fields creation in Sumo Logic for Squid

Following are the [fields](https://help.sumologic.com/docs/manage/fields/) which will be created as part of Squid App install if not already present:

- **`webengine.cluster.name`** - User configured. Enter a name to identify this Squid cluster. This cluster name will be shown in the Sumo Logic dashboards.
- **`webengine.system`**  - Has a fixed value of **squidproxy**.
- **`sumo.datasource`** - Has a fixed value of **squidproxy**.

## Collection Configuration & App installation

As part of setting up the collection process and app installation user can select the App from App Catalog and follow the steps below:

### Step 1: Set up Collector

:::note
If you want to use an existing OpenTelemetry Collector, you can skip this step by selecting the "Use an existing Collector" option.
:::

1. Select the **Add a new Collector** option.
2. Select the platform for which you want to install the Sumo OpenTelemetry Collector.

This will generate a command which can be executed in the machine which needs to get monitored. Once executed it will install the Sumo Logic OpenTelemetry Collector agent.  

<img src={useBaseUrl('https://sumologic-app-data-v2.s3.amazonaws.com/dashboards/Squid-Proxy-OpenTelemetry/Squid-Proxy-Collector.png')} alt="Collector" />

## Step 2: Configure integration

Open Telemetry works with a [configuration](https://opentelemetry.io/docs/collector/configuration/) yaml file which has all the details with respect to the data which needs to be collected. For example, it specifies the location of a log file that is read and sent to the Sumo Logic platform..

In this step we will be configuring the yaml required for Squid Collection.

Path of the log file configured to capture Squid logs is needed to be given here.

The files are typically located in /var/log/squid/access.log. Please refer "Prerequisite" section for more details.
You can add any custom fields which you want to tag along with the data ingested in sumo.
Click on the **Download YAML File** button to get the yaml file.

<img src={useBaseUrl('https://sumologic-app-data-v2.s3.amazonaws.com/dashboards/Squid-Proxy-OpenTelemetry/Squid-Proxy-YAML.png')} alt="YAML" />

## Step 3: Sending logs to Sumo

Once you have the yaml file downloaded in step 2, please follow the below steps based on your environment

<Tabs
  className="unique-tabs"
  defaultValue="Linux"
  values={[
    {label: 'Linux', value: 'Linux'},
    {label: 'macOS', value: 'macOS'},
  ]}>

<TabItem value="Linux">

1.  Copy the yaml at /etc/otelcol-sumo/conf.d/ folder in the Squid instance which needs to be monitored.
2.  Restart the otelcol-sumo process using the below command 
```sh
sudo systemctl restart otelcol-sumo
```

</TabItem>
<TabItem value="macOS">

1.  Copy the yaml at `/etc/otelcol-sumo/conf.d/` folder in the Squid instance which needs to be monitored.
2.  Restart the otelcol-sumo process using the below command 
```sh
 otelcol-sumo --config /etc/otelcol-sumo/sumologic.yaml --conf "glob:/etc/otelcol-sumo/conf.d/*.yaml"
```

</TabItem>
</Tabs>

After successful execution of the above command, Sumo will start receiving the data from your host machine. 

Press **Next**. This will install the app to your Sumo Logic Org. The app consists of Dashboards.

Panels will start to fill automatically. It's important to note that each panel fills with data matching the time range query and received since the panel was created. Results won't immediately be available. Your data should appear in the dashboard within 20 minutes and you'll see full graphs and map.

### Sample Log Messages in Non-Kubernetes environments

```sql
1674483805.439 44 192.168.100.40 TCP_REFRESH_MODIFIED/301 514 GET http://openstack.org/ - HIER_DIRECT/192.168.100.40 text/html
```

### Sample Query

This sample Query is from the Squid Proxy - HTTP Response Analysis > URLs Experiencing Redirections panel.

Query String

```sql
%"sumo.datasource"=squidproxy %"webengine.cluster.name"=*  host.name=* %"webengine.system"=squidproxy
| json "log" as message nodrop 
| if (isEMpty(message), _raw, message) as message
| parse regex field = message "(?<time>[\d.]+)\s+(?<elapsed>[\d]+)\s+(?<remotehost>[^\s]+)\s+(?<action>[^/]+)/(?<status_code>[\d]+)\s+(?<bytes>[\d]+)\s+(?<method>[^\s]+)\s+(?<url>[^\s]+)\s(?<rfc931>[^\s]+)\s+(?<peerstatus>[^/]+)/(?<peerhost>[^\s]+)\s+(?<type>[^\s|$]+?)(?:\s+|$)" nodrop | parse field = message "Connection: *\\r\\n" as connection_status nodrop | parse field = message "Host: *\\r\\n" as host nodrop | parse field = message "User-Agent: *\\r\\n" as user_agent nodrop | parse field = message "TE: *\\r\\n" as te nodrop
| where status_code matches "3*"
| count as eventCount by url, status_Code
| sort by status_code, eventCount, url
```

## Viewing Squid Dashboards

### Squid Proxy - Overview

The **Squid Proxy - Overview** Dashboard provides an at-a-glance view of the activity and health of the SquidProxy clusters and servers by destination locations, error and denied requests, URLs accessed.

<img src={useBaseUrl('https://sumologic-app-data-v2.s3.amazonaws.com/dashboards/Squid-Proxy-OpenTelemetry/Squid-Proxy-Overview.png')} alt="Overview" />

### Squid Proxy - Activity Trend

**Squid Proxy - Activity Trend** dashboard provides trends around denied request trend, action trend, time spent to serve, success and non-success response, remote hosts.

<img src={useBaseUrl('https://sumologic-app-data-v2.s3.amazonaws.com/dashboards/Squid-Proxy-OpenTelemetry/Squid-Proxy-Activity-Trend.png')} alt="Activity Trend" />

### Squid Proxy - HTTP Response Analysis

**The Squid Proxy -  HTTP Response Analysis** dashboard provides insights into HTTP response, HTTP code, the number of client errors, server errors, redirections outlier, URLs experiencing server errors.

<img src={useBaseUrl('https://sumologic-app-data-v2.s3.amazonaws.com/dashboards/Squid-Proxy-OpenTelemetry/Squid-Proxy-HTTP-Response-Analysis.png')} alt="HTTP Response Analysis" />

### Squid Proxy - Quality of Service

The **Squid Proxy -  Quality of Service** dashboard provides insights into latency, the response time of requests according to HTTP action, and the response time according to location.

<img src={useBaseUrl('https://sumologic-app-data-v2.s3.amazonaws.com/dashboards/Squid-Proxy-OpenTelemetry/Squid-Proxy-Quality-of-Service.png')} alt="Quality of Service" />