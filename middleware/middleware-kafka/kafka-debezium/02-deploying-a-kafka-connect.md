After you set up a Kafka cluster, deploy Kafka Connect in a custom container image for Debezium. 
The Kafka Connect service provides a framework for managing Debezium connectors.

You can create a custom container image by downloading the Debezium MySQL connector archive from the [Red Hat Integration](https://access.redhat.com/jbossnetwork/restricted/listSoftware.html?product=red.hat.integration&downloadType=distributions) download site and extracting it to create the directory structure for the connector plugin.

After you obtain the connector plugin, you can create and publish a custom Linux container image by running the `docker build` or `podman build` commands with a custom Dockerfile.

> To save some time, we have already created an image for you. 
For detailed information about deploying Kafka Connect with Debezium, see the [Debezium documentation](https://access.redhat.com/documentation/en-us/red_hat_integration/2020-q3/html-single/getting_started_with_debezium/index#deploying-kafka-connect).

To deploy the Kafka Connect cluster with the custom image that is provided in the scenario, enter the following command:

``oc -n debezium apply -f /root/projects/debezium/kafka-connect.yaml``{{execute interrupt}}

After a few minutes, the Kafka Connect node is deployed. To check the pod status, enter the following command:

``oc get pods -w -l app.kubernetes.io/name=kafka-connect``{{execute}}

The command returns status in the following format:

```bash
NAME                                READY   STATUS              RESTARTS   AGE
debezium-connect-6fc5b7f97d-g4h2l   0/1     ContainerCreating   0          3s
debezium-connect-6fc5b7f97d-g4h2l   0/1     ContainerCreating   0          9s
debezium-connect-6fc5b7f97d-g4h2l   0/1     Running             0          25s
debezium-connect-6fc5b7f97d-g4h2l   1/1     Running             0          90s
```
After a couple of minutes, the pod status changes to `Running`.  
When the **READY** column shows **1/1**, you are ready to proceed.

Enter <kbd>Ctrl</kbd>+<kbd>C</kbd> to stop the process.

`^C`{{execute ctrl-seq}}

## Verify that Kafka Connect is running with Debezium

After the Connect node is running, you can verify that the Debezium connectors are available. 
Because AMQ Streams lets you manage most components of the Kafka ecosystem as Kubernetes custom resources, you can obtain information about Kafka Connect from the `status` section of the `KafkaConnect` resource.

Enter the following command to list the connector plugins that are available on the Kafka Connect node.

``oc get kafkaconnect/debezium -o json | jq .status.connectorPlugins``{{execute interrupt}}

The command returns output that is similar to the following example:

```json
[
  {
    "class": "io.debezium.connector.db2.Db2Connector",
    "type": "source",
    "version": "1.2.4.Final-redhat-00001"
  },
  {
    "class": "io.debezium.connector.mongodb.MongoDbConnector",
    "type": "source",
    "version": "1.2.4.Final-redhat-00001"
  },
  {
    "class": "io.debezium.connector.mysql.MySqlConnector",
    "type": "source",
    "version": "1.2.4.Final-redhat-00001"
  },
  {
    "class": "io.debezium.connector.postgresql.PostgresConnector",
    "type": "source",
    "version": "1.2.4.Final-redhat-00001"
  },
  {
    "class": "io.debezium.connector.sqlserver.SqlServerConnector",
    "type": "source",
    "version": "1.2.4.Final-redhat-00001"
  }
  ...
]
```

> Note: The preceding output is formatted to improve readability

Our Debezium `MySqlConnector` connector is now available for use on the Connect node.

You have successfully deployed a Kafka Connect node and configured it to contain Debezium.

In the next step of this scenario we will finish the deployment by creating a connection between the MySQL database source and Kafka Connect.
