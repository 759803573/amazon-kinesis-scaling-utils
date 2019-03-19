## [Github](https://github.com/759803573/amazon-kinesis-scaling-utils/tree/alo7)
##### Fork from: https://github.com/awslabs/amazon-kinesis-scaling-utils

## amazon-kinesis-scaling-utils


The Kinesis Scaling Utility is designed to give you the ability to scale Amazon Kinesis Streams in the same way that you scale EC2 Auto Scaling groups – up or down by a count or as a percentage of the total fleet. You can also simply scale to an exact number of Shards. There is no requirement for you to manage the allocation of the keyspace to Shards when using this API, as it is done automatically.

## Automatic Scaling

### Conf
a streamMonitor object is a definition of an Autoscaling Policy applied to a Kinesis Stream, and this array allows a single Autoscaling Web App to monitor multiple streams. A streamMonitor object is configured by:
```json
{"streamName":"String - name of the Stream to be Monitored",
 "region":"String - a Valid AWS Region Code, such as us-east-1 or eu-west-1",
 "scaleOnOperation":"List<String> - the types of metric to be monitored, including PUT or GET. Both PutRecord and PutRecords are monitored with PUT",
 "minShards":"Integer - the minimum number of Shards to maintain in the Stream at all times",
 "maxShards":"Integer - the maximum number of Shards to have in the Stream regardless of capacity used",
 "refreshShardsNumberAfterMin":"Integer - minutes interval after which the Stream Monitor should refresh the Shard count on the stream, to accomodate manual scaling activities. If unset, defaults to 10 minutes"
 "scaleUp": {
     "scaleThresholdPct":Integer - at what threshold we should scale up,
     "scaleAfterMins":Integer - how many minutes above the scaleThresholdPct we should wait before scaling up,
     "scaleCount":Integer - number of Shards to scale up by (prevails over scalePct),
     "scalePct":Integer - % of current Stream capacity to scale up by,
     "notificationARN" : String - the ARN of an SNS Topic to send notifications to after a scaleUp action has been taken
 },
 "scaleDown":{
     "scaleThresholdPct":Integer - at what threshold we should scale down,
     "scaleAfterMins":Integer - how many minutes below the scaleThresholdPct we should wait before scaling down,
     "scaleCount":Integer - number of Shards to scale down by (prevails over scalePct),
     "scalePct":Integer - % of current Stream capacity to scale down by,
     "coolOffMins":Integer - number of minutes to wait after a Stream scale down before we scale down again,
     "notificationARN" : String - the ARN of an SNS Topic to send notifications to after a scaleDown action has been taken
 }
}
```
demo:
```json
[
	{
        "streamName": "dev-log",
        "region": "cn-north-1",
        "scaleOnOperation": ["PUT","GET"],
        "minShards": 2,
        "maxShards": 32,
        "refreshShardsNumberAfterMin": 2,
        "scaleUp": {
            "scaleThresholdPct": 75,
            "scaleAfterMins": 2,
            "scalePct": 200
        },
        "scaleDown": {
            "scaleThresholdPct": 25,
            "scaleAfterMins": 2,
            "scalePct": 50,
            "coolOffMins": 2
        }
    }
]
```

### Running
```bash
docker run -v ~/.aws:/root/.aws -v `pwd`/conf/configuration.json:/usr/local/kinesis_scaling/conf/configuration.json  759803573/kinesis-scaling:0.9.5.8

# ~/.aws is aws credentials
# `pwd`/conf/configuration.json is config file
```

### Autoscaling Behaviour ##

In version .9.5.0, Autoscaling added the ability to scale on the basis of PUT ___and___ GET utilisation. This change means that you carefully have to consider your actual utilisation of each metric prior to configuring autoscaling with both metrics. For information on how the AutoScaling module will react with both metrics, consider the following table:

| | | PUT | | |
| :-- | :-- | :--: | :--: | :--: |
| | __Range__ | Below | In | Above |
|__GET__ | Below | Down | Down | Up |
| | In | Down | Do Nothing | Up |
| | Above | Up | Up | Up