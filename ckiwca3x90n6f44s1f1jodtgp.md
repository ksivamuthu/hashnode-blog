## Streaming Auth0 Logs to Datadog to monitor

Hello 👋,  Application Monitoring is critical for your business. It gives vital information on your customer experience. Monitoring your system and infrastructure is essential to ensure the performance of your services. As software development moves faster and faster, alerting and monitoring becomes an indispensable practice for modern DevOps teams.

The cloud applications' monitoring platform has to bring together data from servers, containers, databases, identity management, and third-party services to make the stack entirely observable.

In this post, we will see how we can stream Auth0 logs to Datadog to monitor, alert, and analyze Auth0 events.

### Prerequisites

* [Auth0 Account](https://auth0.com)
* [Datadog Account](https://www.datadoghq.com)
* [Datadog - API Key](https://docs.datadoghq.com/account_management/api-app-keys/)

### Setup Auth0 Log Streaming

The first step is to setup log streaming from the Auth0 dashboard. 

1. Log in to the Auth0 Dashboard.

2. Navigate to Logs > Streams.

3. Click "Create Stream."

4. Select Datadog, and enter a unique name for your new Datadog Event Stream.

5. On the next screen, provide the following settings for your Datadog Event Stream: API Key & Region. Click Save.


![Screen Shot 2020-12-19 at 5.44.34 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1608418385273/C542r19kU.png)

The setup is done. When Auth0 writes the next log event, the log events are streamed to Datadog.

### Monitor Auth0 Activities

As Auth0 streaming the logs to Datadog to ingest, it sends them through a log processing pipeline. The pipeline automatically parses each log to extract key data as standard attributes, which provide a naming convention you can use to correlate events from multiple sources easily.

Auth0 has a [set of event types with code and description](https://auth0.com/docs/logs/log-event-type-codes). Datadog automatically maps the Auth0 event type code to the event name. You can use this Event Name facet to analyze user activity such as account creation and deletion, password changes, and more.

![Screen Shot 2020-12-19 at 5.51.42 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1608418484291/LbQxYmrV4.png)

### Security Dashboard Visualization

If you want to visualize the Auth0 events using Datadog dashboards, Auth0 is providing the dashboard template to import in Datadog to visualize the metrics captured from the log stream.

The dashboard provides the recommended aggregations to view, and you can customize the visualization based on your needs.

Download the [dashboard template](https://cdn.auth0.com/website/docs/logs/streams/datadog/Auth0SecurityDashboard.json?_ga=2.54375016.1952945764.1608340101-189280238.1605710959) here. Import the JSON file in the Datadog Dashboard section created using the Screen Board Dashboard type.

![Screen Shot 2020-12-19 at 6.05.00 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1608419138036/BT1BAC5Za.png)

### Alerts and Threat Notification

Datadog's new Detection Rules give you a powerful way to detect security threats and suspicious behavior within all ingested logs in real-time. You can create the detection rules by defining conditional logic to the Auth0 log stream. If the rule is matched, Datadog evaluates the security and notify.

Datadog has pre-defined detection rules for Auth0 source.

* Auth0 user authenticating from multiple countries - High
* Auth0 user logged in with a breached password - Medium
* Brute force attack on an Auth0 user - Medium
* Credential stuffing attack on Auth0 - High

![Screen Shot 2020-12-19 at 6.17.05 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1608419863776/kaUJ7lgla.png)

You can also create custom rules with conditions. Datadog analyzes Auth0 logs to detect any violations and alert your team via notification channels or incident management system.

### Conclusion

Auth0's log streaming service streams the events and logs to a host of integration partners, such as Amazon EventBridge, Azure EventHub, Datadog, Splunk, Sumologic, and custom webhooks.

Log / Event streaming helps to export the log, take decisions based on the events for threat detection and notifications, and also process identity-information in near real-time. It also helps to set up the additional data pipelines for processing based on the Auth0 events using Amazon Event Bridge or Amazon Event Hub in Event-driven applications.

> Please follow me on my [twitter](https://www.twitter.com/ksivamuthu), [linkedin](https://www.linkedin.com/in/ksivamuthu/) and [github](https://www.github.com/ksivamuthu) for more articles/demos on security, cloud native, containers and mobile/web apps.
