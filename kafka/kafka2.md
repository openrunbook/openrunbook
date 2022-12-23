# Kafka 2.x

Apache Kafka is a distributed streaming platform capable of handling trillions of events a day. Initially conceived as a messaging queue, Kafka is based on an abstraction of a distributed commit log. Since being created and open sourced by LinkedIn in 2011, Kafka has quickly evolved from messaging queue to a full-fledged streaming platform.

As a streaming platform, Apache Kafka provides low-latency, high-throughput, fault-tolerant publish and subscribe pipelines and is able to process streams of events. Kafka provides reliable, millisecond responses to support both customer-facing applications and connecting downstream systems with real-time data.

# Useful links

- Kafka documentation
  - [Kafka manual](https://kafka.apache.org/documentation/)
  - Primer on [Kafka client rebalancing](https://tomlee.co/2019/03/the-unofficial-kafka-rebalance-how-to/#retriable-vs-non-retriable-errors)
  - [Kafka Controller Internals](https://cwiki.apache.org/confluence/display/KAFKA/Kafka+Controller+Internals)
  - [Topic/Partition Design Principals](https://www.confluent.io/blog/how-to-choose-the-number-of-topicspartitions-in-a-kafka-cluster/)
  - [Apache Kafka source code](https://github.com/apache/kafka)
- Links to dashboards, build and release pipelines.

# Package management and release

## Cruise Control
[Cruise control](https://github.com/linkedin/cruise-control) UI is useful for automating most cluster management operations. 

## Kafka offset exporter
[Kafka offset exporter](https://github.com/echojc/kafka-offset-exporter) can be used to export consumer statistics to a metrics backend like Prometheus. 

## Kafka Manager UI
[Kafka manager UI](https://github.com/yahoo/CMAK) can be used to manage the Kafka clusters. 

# Runbook

## Restarting a single kafka broker on a node

To restart a process on the kafka broker just run `sudo systemctl restart kafka201`.
To check the status of the kafka process on the broker `sudo systemctl status kafka201`.

## Fixing URP(Under Replicated Partition) issues in Kafka cluster

Under replicated partitions happen in a Kafka cluster because one of the brokers in the cluster is dead or non-responsive. To fix URP please follow these steps:

Causes of URP:

- One or more brokers is dead.
- One or more brokers is wedged/stuck/semi-dead.

Detection and remediation

1.  If you have URP alert, don't restart the entire cluster. This makes the problem much worse.
2.  Determine the brokers causing the URP: Usually it's a single broker.
    - Using the kafka manager UI pick a topic with URP(Under replicated column) 
    - On the topic page, look at the partition information section. Pick a couple of partitions and see which broker is missing among them.
    - If the broker_id missing from both topics is the same then that is the problematic broker id. 
    - Confirm with a few more topics with URPs similarly to make sure that broker you identified is indeed the problem.
    - Once you have the broker id, go to the `brokers` tab in the Kafka manager UI to get the host name of the broker.
3.  Fixing the problem broker(s).
    - If the host is still up, do some investigation by ssh'ing into the host.
      - Look at the kafka log at /var/log/kafka/server.log to identify any errors in the log.
      - Look for errors in syslog.
      - Look for disk errors in dmesg.
      - If there are no hardware issues, restart the kafka process on the broker. This should usually bring the broker up right away.
      - If the broker doesn't seem healthy, and this is the only problem broker, terminate the host and wait for ASG to spin up a new host.
    - If the host is terminated already, then we need to move the partitions from the dead broker to the new broker.
      - Use the 'Free memory' graph panel in the Kafka dashboard to identify the new broker. The new broker will have 100% free memory usually.
      - Using the Kafka-manager UI, re-assign the partitions **for each topic one by one** by following the steps below.
      - Click "Generate partition assignments" and then Done.
      - Then go back to topics page and hit "Run partition assignments" and on that page hit "Done".
      - This will kick off a lengthy "Partitition assignment" operartion that will take a several hours to run.
      - You can monitor the partition assignment from the Kafka manager UI. The "Maximum replication lag" graph on Grafana dashboard should show you how much the lag is and which brokers are lagging very precisely.
    - Once the re-assignment partitions operation is completed, the "Maximum replication lag" operation is 0 and there should be no "Under replicated partitions".

## Check if there is a pending re-assignment on the Kafka cluster

Use the re-assign partitions tab in cruise control for your cluster. If there is an ongoing partition assignment, you would see the status as completed for the cluster. 

## Find consumer group status from command line

To get the status of the consumer group run the following command :
`/usr/local/kafka/bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group <group_name ex:logstash> --describe`

## Check if there is a pending re-assignment on the Kafka cluster

Use the re-assign partitions tab in cruise control for your cluster. If there is an ongoing partition assignment, you would see the status as completed for the cluster. 

## How to cancel a pending re-assignment

To cancel the pending re-assignment, we need to do it via ZK. Run the following command in the ZK shell on a ZK node.
`deleteall "/admin/reassign_partitions"`


## Fixing offline partition issues in Kafka cluster

Offline partitions happen in a Kafka cluster when the partition lost it's leader and kafka not able to assign a new leader.

Causes of offline partition:

- one or more Kafka brokers terminated before Kafka completed the reassign partition operation.
- The partition lost the leader while behind on replication.
- Three Kafka brokers that have replicas for the partition go offline or are terminated.

**Detection**:

- Using the kafka manager UI pick a topic with offline partition. 
- On the topic page, look at the partition information section. If the partition leader is `-1`, it is an offline partition.

**Remediation** :

1.  Reassign Partition with Kafka Manager:
    _Note_: it is best for Cruise Control to handle partition reassignment operations and in most cases.

    - Using the kafka-manager UI, re-assign the partitions **for each topic one by one** by following the steps below.
    -  Click "Generate partition assignments" and then Done. 
    - Then go back to topics page and hit "Run partition assignments" and on that page hit "Done".
    - This will kick off a lengthy "Partitition assignment" operartion that will take a several hours to run.
    - You can monitor the partition assignment from the Kafka manager UI. The "Maximum replication lag" graph on Grafana dashboard should show you how much the lag is and which brokers are lagging very precisely.
    - Once the re-assignment partitions operation is completed, the "Maximum replication lag" operation is 0 and there should be no "Under replicated partitions".


2.  Reassign Partition Manually: If kafka-manager is unable to generate partition assignments, use Zookeeper to manually assign a leader. **This option can lead to potential data loss.**
    - In kafka-manager UI, go to the cluster, then select "Update Config". For the topic in question, set `unclean.leader.election.enable` to `true` and hit "Update Config". This allows Kafka to elect a leader for a partition regardless of the state of the data. **This will lead to data loss in some cases.**
- Find the Zookeeper cluster for the corresponding kafka cluster.
- SSH to one of the the Zookeeper nodes and launch zookeeper cli by running `zookeeper-cli`
- `get /brokers/topics/[topic name]/partitions/5/state` will give the current state of the partition

```
[zk: localhost:2181(CONNECTED) 1] get /brokers/topics/apache-access/partitions/276/state
{"controller_epoch":3,"leader":-1,"version":1,"leader_epoch":10,"isr":[1019,1014,1020]}
```

- change the leader of the partition `set /brokers/topics/[topic name]]/partitions/5/state {"controller_epoch":19,"leader":1003,"version":1,"leader_epoch":88,"isr":[1003,1004, 1005]}`

```
[zk: localhost:2181(CONNECTED) 4] set /brokers/topics/apache-access/partitions/276/state {"controller_epoch":3,"leader":1019,"version":1,"leader_epoch":10,"isr":[1019,1014,1020]}
```

- verify the offline partition is fixed in the [Kafka Service Grafana dashboard].
- If the partition remains offline or the partition leader has not yet changed, we need to elect a new controller to refresh cluster metadata.
- In the zookeeper CLI, run: `deleteall /controller` - this will delete the controller information and Kafka will elect a new controller.
- The offline partition should now have a leader and be fixed.
- Be sure to disable `unclean.leader.election.enable` by deleting the value `true` and hitting "Update Config"

## Rebalance Kafka Cluster
Use Cruise-Control UI to rebalance the cluster. Cruise Control Manual Rebalance

## Enable unclean leader election for a topic via CLI

ssh into any Kafka brokers and run

```bash
/usr/local/kafka/bin/kafka-configs.sh --zookeeper <zookeeper host>:2181 --entity-type topics --entity-name <topic name> --alter --add-config unclean.leader.election.enable=true
```

## Elect a new cluster controller gracefully

- Find the Zookeeper cluster for the corresponding Kafka cluster.
- SSH to one of the the Zookeeper nodes and launch Zookeeper CLI by running `zookeeper-cli`
- In the Zookeeper CLI, run: `deleteall /controller` - this will delete the controller information and Kafka will elect a new controller.

## Reset Consumer Offset

1. Find the consumer name.
2. You'll need the topic name as well.
3. Stop consumer process on all nodes.
4. Go to any Kafka node in relevant cluster, and execute the following command.

    ```
    /usr/local/kafka/bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group <consumer group name> --topic <topic name> --reset-offsets --to-latest --execute
    ```

## Reset a consumer group offset to a specific datetime

You can reset to a particular date time using `--to-datetime` flag in place of `--to-latest`. Recall that our system clocks are in PST/PDT, whereas Kafka operates in UTC. This means you'll need to specify timestamp in corresponding UTC time.

- PDT (2020-10-23T00:24:12) to UTC: `date --utc +%Y-%m-%dT%H:%M:%S.000 --date='TZ="America/Los_Angeles" 2020-10-23T00:24:12'`

```
/usr/local/kafka/bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group <consumer group name> --topic <topic name> --reset-offsets --to-datetime <UTC YYYY-MM-DDTHH:mm:SS.sss> --execute
```

# Alerts

## Kafka disk space alert
We usually run our Kafka clusters at 50% disk utilization, so this should almost never be an issue. If you run into disk space issues, please do the following steps:

Add more nodes to the cluster by updating the terraform config or updating the asg.
Kicking off a partition relabance via cruise control once the new host is up.
We try to store 2 days worth of data in Kafka. In a pinch, you can also reduce the retention for the topics via the Kafka Manager UI.

## Cruise control stuck in a loop

Cruise control automatically rebalances the Kafka cluster. However, if it's partitions are corrupted it can be stuck in a loop and throw messages as shown below. 

```
BROKER_FAILURE detected {
    Broker 1023 failed at 18/10/2021 22:46:51
}. Self healing start time 18/10/2021 23:01:51.
Self-healing has been triggered.
Self-healing has been triggered.
```

To fix this issue, fix the URP in cruise control as follows:

- Find the zookeeper cluster for the corresponding kafka cluster.
- launch zookeeper cli by running `zookeeper-cli`.
- `get /brokers/topics/<topic name>/partitions/<partition number>/state` will give the current state of the partition. Ex: `get /brokers/topics/test/partitions/5/state`
- change the leader of the partition using `set /brokers/topics/<topic name>/partitions/<partition id>/state ..`. Example 
```
set /brokers/topics/<topic name>/partitions/<pa>/state 
{"controller_epoch":19,"leader":1003,"version":1,"leader_epoch":88,"isr":\[1003,1004, 1005]} 
```
- verify the offline partition is fixed in the grafana dashboard
- Restart Cruise control.


### When multiple topics need reassignment and cruise control is stuck

All of the following commands can be run on any Kafka host in the affected cluster.

1.  Get list of available broker ids. This command will take some time to complete as we run a large number of brokers. You can also get this list from `zookeeper-cli` using `ls /brokers/ids` on a zookeeper host, but that will require you to ssh into another machine. :shrug:

    ```
    printf '[' && kafka-list-brokers | cut -f 2 -d ' ' | tr -s '\n' ', ' && printf '\b]\n'
    ```

    example output

    ```
    [1003, 1004, 1005, 1008, 1012, 1022, 1025, 1026, 1028, 1029, 1030, 1032, 1035, 1152, 1155, 1159, 1163, 1166, 1169, 1171, 1172, 1250, 1266, 1268, 1269, 1270, 1272, 1273, 1275, 1276, 1277, 1279, 1280, 1281, 1283, 1284, 1285, 1286, 1287, 1288, 1290, 1291, 1292, 1293, 1294, 1295, 1297, 1298, 1299, 1300, 1301, 1302, 1304, 1305, 1306, 1307, 1308, 1309, 1310, 1312, 1313, 1314, 1315, 1316, 1317, 1318, 1320, 1321, 1322, 1323, 1324, 1325, 1327, 1328, 1330, 1331, 1332, 1333, 1334, 1335, 1337, 1338, 1341, 1342, 1343, 1344, 1345, 1346, 1347, 1348]
    ```

2.  Update the following ruby script (`partitions_to_json.rb`): Replace `[POPULATE_THIS_LIST]` with above list of available brokers. This script looks for missing brokers and replaces them for every partition reported in step 3. It does not apply anything, but writes out the plan in form of JSON to a file (`reassign-partitions.json`) while printing it on stdout. The script will automatically assign a leader if one does not exist for the partition. You will apply the plan in step 5.:

    ```
    #!/usr/bin/ruby

    require 'json'

    PARTITION_COUNT = 3
    f = File.read('partitions_to_fix.list')
    assignments = f.scan(/Topic:\s+(\S+)\s+Partition:\s+(\S+)\s+Leader:\s+(\-?\d+)\s+Replicas:\s+(\d+),(\d+),(\d+)/)
    partitions_fixed = []
    available_brokers = [POPULATE_THIS_LIST]

    assignments.each do |a|
        topic = a[0]
        partition = a[1].to_i
        leader = a[2].to_i
        replicas = [ a[3].to_i, a[4].to_i, a[5].to_i ]
        missing_brokers = replicas - available_brokers
        # remove broker ids that are missing
        replicas -= missing_brokers
        # add broker ids that we want to use in place of the ones that are missing.
        # choose from list of available brokers that does not contain brokers
        # already assigned to this partition.
        candidate_brokers = available_brokers - replicas
        while replicas.length < PARTITION_COUNT do
        replicas += [candidate_brokers.sample]
        replicas.uniq!
        end
        # assign existing broker as the leader, or choose one from replica list.
        # at this point in code replicas[0] will contain a pre-existing broker,
        # or a new broker if brokers for all replicas are dead. this is because
        # we have removed missing_brokers, and added new to end of array.
        leader = replicas[0] unless replicas.include?(leader)
        partitions_fixed << { topic: topic, partition: partition, replicas: replicas, leader: leader }
    end

    reassign_partitions = JSON.pretty_generate({ partitions: partitions_fixed, version: 1})

    puts reassign_partitions
    File.write('reassign-partitions.json', reassign_partitions, mode:'w')

    ```

3.  Generate the list of affected partitions from one the Kafka nodes (SSH into host and execute the following). Choose the command depending on the issue:

    a. Offline/unavailable partitions:

    ```
    kafka-cli topics --unavailable-partitions --describe > partitions_to_fix.list
    ```

    b. Under replicated partitions

    ```
    kafka-cli topics --under-replicated-partitions --describe > partitions_to_fix.list
    ```

4.  Run ruby script (`partitions_to_json.rb`) to convert output of above command to json while replacing 1029 and 1035 (bad) brokers

    ```
    ./partitions_to_json.rb
    ```

    output of above command -

    ```
    {
      "partitions": [
        {
           "topic": "relay-audit_logs0-heads",
           "partition": 13,
           "replicas": [
              1031,
              1022,
              1034
           ]
        },
         ...
    }
    ```

5.  Review the plan. If it looks OK, apply it using. The command will save assignment backup to `reassignment-backup.json` file:

    ```
    kafka-cli reassign-partitions --reassignment-json-file reassign-partitions.json --execute | grep -oP '^\{.+\}' > reassignment-backup.json
    ```

6.  Verify reassignment using:

    ```
    kafka-cli reassign-partitions --reassignment-json-file reassign-partitions.json --verify
    ```

### Recovery is still halted

Recovery can still be halted because Cruise Control is not able to generate a plan to apply, or an apply is failing. In such scenarios we need to manually assign a leader for a parition by updating the state in relevant zookeeper for the kafka cluster.

1.  Logon to a zookeeper host for that Kafka cluster, and fire up `zookeeper-cli`

    ```
    /usr/local/bin/zookeeper-cli
    ```

2.  For every paritition that is affected, get its state in order to get the epoch values from `zookeeper-cli`. In following example, we are getting state of paritition `9` in `tails` topic:

    ```
    get /brokers/topics/tails/partitions/9/state
    ```

    output:

    ```
    {"controller_epoch":11,"leader":-1,"version":1,"leader_epoch":17,"isr":[1022,1028,1034]}
    ```

3.  Update the `leader` value to one from `isr` list. Ideally, the leader should be the one already existed. For example, if `1022` and `1034` are new brokers (which means they will not contain any data for that partition), we need to assign a leader that has some data in it. So in this scenario, `1028` will be our preference. In `zookeeper-cli`, execute the following command to set the leader for the partitions. Kindly note that **no values other than `leader` are modified**:

    ```
    set /brokers/topics/tails/partitions/9/state {"controller_epoch":11,"leader":1028,"version":1,"leader_epoch":17,"isr":[1022,1028,1034]}
    ```
