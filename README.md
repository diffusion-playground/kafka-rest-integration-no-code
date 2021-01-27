[![Video Tutorial](https://github.com/pushtechnology/tutorials/blob/master/data-store/video.png)](https://www.pushtechnology.com/blog/fine-grained-fan-out-and-replication-of-kafka-event-firehose-between-clusters/)

# Fine-grained fan-out and replication of Kafka event firehose between clusters/sites.

Introduction to Diffusion Real-Time Event Stream through a simple application using [Diffusion](https://www.pushtechnology.com/product-overview) Cloud and Apache Kafka.

A simple projects illustrating **real-time replication and fan-out** of foreign exchange (fx) event streams from Kafka cluster A to Kafka cluster B, through **Diffusion Cloud** instance via the use of our [Kafka Adapter](https://www.pushtechnology.com/wp-content/uploads/2020/08/Diffusion-Cloud-Kafka-adapter.pdf).

These JavaScript code examples will help you publish fx events on real-time from a front-end app to a Kafka cluster A, consume from it and transform data on-the-fly via our powerful [Topic Views](https://docs.pushtechnology.com/docs/6.5.2/manual/html/designguide/data/topictree/topic_views.html) feature.

Although we provide a fx data simulation client using JavaScript, to populate Kafka cluster A, this tutorial purely focus on the **no-code solution** to deliver event data between remote Kafka sites/clusters where not all the firehose data from one site needs to be replicated/fanned-out to the other.

![](https://raw.githubusercontent.com/diffusion-playground/kafka-integration-no-code/master/kafka-app-L1/images/kafkaL2.png)

# Fine-grained distribution of Kafka event firehose with Topic Views
**kafka-app-L1** introduces the concept of [**Topic Views**](https://docs.pushtechnology.com/docs/6.5.2/manual/html/designguide/data/topictree/topic_views.html), a dynamic mechanism to map part of a server's [Topic Tree](https://docs.pushtechnology.com/docs/6.5.2/manual/html/designguide/data/topictree/topic_tree.html) to another. This enables real-time data transformation before replicating to a remote cluster as well as to create dynamic data models based on on-the-fly data (eg: Kafka firehose data).
This lesson also shows how to use our [**Kafka adapter**](https://www.pushtechnology.com/blog/connect-diffusion-with-apache-kafka/) to ingest and broadcast fx data using Diffusion Topic Views in order to consume what you need, not all the Kafka stream.

# Features used in this lesson

## Step 1: Configure Kafka Adapter in Cloud to ingest from Kafka Cluster A
### Go to: [Diffusion Cloud > Manage Service > Adapters > Kafka Adapter > Ingest from Kafka](https://dashboard.diffusion.cloud)
![](https://raw.githubusercontent.com/diffusion-playground/kafka-integration-no-code/master/kafka-app-L1/images/ingest.png)

```
Adapters > Kafka Adapter > Ingest from Kafka Config:

	Bootstrap Server > connect to you Kafka cluster A (eg: "kafka-sasl.preprod-demo.pushtechnology")
	Diffusion service credentials > admin, password (use the "Security" tab to create a user or admin account)
	Kafka Topic subscription > the source topic from your Kafka cluster (eg: "FXPairs")
	Kafka Topic value type > we are using JSON, but can be string, integer, byte, etc.
	Kafka Topic key type > use string type for this code example.
```

## Step 2: Check the Kafka stream `FXPairs` is ingested
### Go to: [Diffusion Cloud > Manage Service > Console > Topics](https://dashboard.diffusion.cloud)
We can see the events from `FXPairs` Kafka topic (from cluster A) is now being published to Diffusion topic path: `FXPairs`. If there are no new events, it might be because the `FXPairs` topic has not received any updates from Kafka yet.

![](https://github.com/diffusion-playground/kafka-integration-no-code/blob/master/kafka-app-L1/images/kafka%20firehose.png)

## Step 3: Create Topic Views using [Source value directives](https://docs.pushtechnology.com/docs/6.5.2/manual/html/designguide/data/topictree/topic_views.html)
Source value directives use the keyword [`expand()` value directive](https://docs.pushtechnology.com/docs/6.5.2/manual/html/designguide/data/topictree/topic_views.html) to create multiple reference topics from a single JSON source topic, or [`value()` directive](https://docs.pushtechnology.com/docs/6.5.2/manual/html/designguide/data/topictree/topic_views.html) to create a [new JSON value](https://www.pushtechnology.com/blog/new-topic-view-features-in-6.4) with a subset of a JSON source topic.

### Go to: [Diffusion Cloud > Manage Service > Console > Topics > Topic Views](https://management.ad.diffusion.cloud/#!/login)
We are going to `map` the topic `FXPairs` stream (we get from Kafka cluster A) `to` a new Diffusion Topic View with path: `pairs/<expand(/value/pairs,/pairName)>/` where `/value/pairs,/pairName` is the Kafka payload currency `pairName` (part of the JSON structure in the FXPairs Kafka topic).

![](https://github.com/diffusion-playground/kafka-integration-no-code/blob/master/kafka-app-L1/images/topic%20views.png)

***This Topic View will take care of dynamic branching and routing of event streams in real-time, by only sending the specific currency pair from Kafka cluster A, to Kafka cluster B, and not the whole stream.***

##### Topic View Specification for Currency Breakout:
##### `map ?FXPairs// to pairs/<expand(/value/pairs,/pairName)>/`

![](https://github.com/diffusion-playground/kafka-integration-no-code/blob/master/kafka-app-L1/images/topic%20views%202.png)

##### Topic View Specification for Tiers Expansion:
##### `map ?pairs// to tiers/<expand(/tiers)>/<path(1)>`

![](https://github.com/diffusion-playground/kafka-integration-no-code/blob/master/kafka-app-L1/images/topic%20views%203.png)

## Step 4: Dynamic branching and routing of Kafka events firehose
As new events are coming in from the Kafka cluster A firehose, Diffusion is dynamically branching and routing the currency pairs, on-the-fly when replicating and fan-out to Kafka cluster B.

**Note:** The topic path will dynamically change as new currency pair values come in.

### Go to: [Diffusion Cloud > Manage Service > Console > Topics](https://management.ad.diffusion.cloud/#!/login)

##### The following image shows a Topic View for the following specification:
###### +pairs > `map ?FXPairs// to pairs/<expand(/value/pairs,/pairName)>/`
###### +tiers > `map ?pairs// to tiers/<expand(/tiers)>/<path(1)>`
![](https://github.com/diffusion-playground/kafka-integration-no-code/blob/master/kafka-app-L1/images/topic%20path.png)

##### When clicking on the "+" for "tiers" topic tree, the following image shows a Topic View for the following specification:
###### `map ?pairs// to tiers/<expand(/tiers)>/<path(1)>`
![](https://github.com/diffusion-playground/kafka-integration-no-code/blob/master/kafka-app-L1/images/expand.png)

### Suggested: 6 Lessons Using Topic Views
#### Lesson 1: [Mapping Topics](https://www.pushtechnology.com/blog/tutorial/using-topic-views-1.mapping-topics/)
#### Lesson 2: [Mapping Topic Branches](https://www.pushtechnology.com/blog/tutorial/using-topic-views-2.mapping-topic-branches/)
#### Lesson 3: [Extracting Source Topic Values](https://www.pushtechnology.com/blog/tutorial/using-topic-views-3.-extracting-source-topic-values/)
#### Lesson 4: [Throttling Reference Topics](https://www.pushtechnology.com/blog/tutorial/using-topic-views-4.throttling-reference-topics/)
#### Lesson 5: [Naming Reference Topic With Topic Content](https://www.pushtechnology.com/blog/tutorial/using-topic-views-5.naming-reference-topic-with-topic-content/)
#### Lesson 6: [Changing Topic Properties Of Reference Topics](https://www.pushtechnology.com/blog/tutorial/using-topic-views-6.changing-topic-properties-of-reference-topics/)

## Step 5: Configure Kafka Adapter in Cloud to broadcast to Kafka cluster B
### Go to: Diffusion Cloud > Manage Service > Adapters > Kafka Adapter > Broadcast to Kafka
![](https://github.com/diffusion-playground/kafka-integration-no-code/blob/master/kafka-app-L1/images/broadcast.png)

```
Adapters > Kafka Adapter > Broadcast to Kafka Config:

	Bootstrap Server > connect to you Kafka cluster A (eg: "kafka-plain.preprod-demo.pushtechnology")
	Diffusion service credentials > admin, password (use the "Security" tab to create a user or admin account)
	Topic subscription > the source topic from Diffusion to be broadcasted to Kafka cluster B (eg: from "tiers/2/GBP-USD" to "FXPairs.tier2.GBP.USD")
	Topic value type > we are using JSON, but can be string, integer, byte, etc.
	Kafka Topic key type > use string type for this code example.
```

# Pre-requisites

*  Download our code examples or clone them to your local environment:
```
 git clone https://github.com/diffusion-playground/kafka-integration-no-code/
```
* A Diffusion service (Cloud or On-Premise), version 6.6 (update to latest preview version) or greater. Create a service [here](https://management.ad.diffusion.cloud/).
* Follow our [Quick Start Guide](https://docs.pushtechnology.com/quickstart/#diffusion-cloud-quick-start) and get your service up in a minute!

# FX Data Generator

Really easy, just open the index.html file locally and off you go!

