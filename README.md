# Kafka trigger
A CLI application for subscribing to Apache Kafka topics and triggering Ylem pipelines using messages as input in real-time streaming mode.

![GitHub branch check runs](https://img.shields.io/github/check-runs/ylem-co/kafka-trigger/main?color=green)
![GitHub go.mod Go version](https://img.shields.io/github/go-mod/go-version/ylem-co/kafka-trigger?color=black)
<a href="https://github.com/ylem-co/kafka-trigger?tab=Apache-2.0-1-ov-file">![Static Badge](https://img.shields.io/badge/license-Apache%202.0-black)</a>
<a href="https://ylem.co" target="_blank">![Static Badge](https://img.shields.io/badge/website-ylem.co-black)</a>
<a href="https://docs.datamin.io" target="_blank">![Static Badge](https://img.shields.io/badge/documentation-docs.datamin.io-black)</a>
<a href="https://join.slack.com/t/datamincommunity/shared_invite/zt-2nawzl6h0-qqJ0j7Vx_AEHfnB45xJg2Q" target="_blank">![Static Badge](https://img.shields.io/badge/community-join%20Slack-black)</a>

## How it works
Kafka-trigger listens to Kafka topics and calls the [pipeline run endpoint](https://docs.datamin.io/datamin-api/api-endpoints#run-workflow) for each message, passing the message body as pipeline input.

In the pipeline, the input can be received by the task "External trigger". When the pipeline is run, this task gets the input data and passes it further to the next pipeline task.

## Configuration
Kafka-trigger is configured with environment variables. 
Besides the conventional way, the config variables can also be specified in the `.env` or `.env.local` file.

Main variables:
- **YLEM_KT_API_URL** - Ylem's API URL. If you use the cloud version of Ylem, it is https://api.ylem.co, othervise add URL of your custom Ylem API instance here.
- **YLEM_KT_API_CLIENT_ID** — OAuth client ID.
- **YLEM_KT_API_CLIENT_SECRET** — OAuth client secret.
- **YLEM_KT_KAFKA_VERSION=3.1.0** — version of the Kafka server.
- **YLEM_KT_KAFKA_BOOTSTRAP_SERVERS="127.0.0.1:9092"** — a comma-separated list of Kafka bootstrap servers.
- **YLEM_KT_KAFKA_TOPIC_MAPPING="topic_1:pipeline_uuid_1,topic_1:pipeline_uuid_2,topic_2:pipeline_uuid_2"** — topic-to-pipeline mapping, a comma-separated list of `<topic name>:<pipeline uuid>` pairs.

Other supported env variables are in the [.env.dist file](https://github.com/ylem-co/kafka-trigger/blob/main/.env.dist)

More information about how to obtain your OAuth credentials is in our [documentation](https://docs.datamin.io/datamin-api/oauth-clients)

## Usage

### 1. Create a new topic in Kafka

Let's call it `test_kafka_trigger`

### 2. Create a new pipeline in Ylem

Let's create a simple pipeline that contains only two tasks: External_trigger and Aggregator.

<img width="1402" alt="352275586-0f20ce9b-cf7e-46fd-8159-b036400370d8" src="https://github.com/user-attachments/assets/13415f4b-84ce-4279-b76a-86957c363e41">

An Aggregator will just send what it receives as input from Kafka to the output.

<img width="1294" alt="352275653-60c9d414-d2b9-4d74-810c-2766b08758de" src="https://github.com/user-attachments/assets/065d327e-75ac-45b9-92d1-d04ed3324bb3">

### 3. Configure the Kafka trigger service

Create the new `.env` and copy the content from `.env.dist` to it and then customize values in the list.

Let's assume, you run this application in a Docker container, with your Kafka set up on the hostmachine port 9092 and you want to trigger pipelines in the Cloud version of Datamin. In Kafka you created a topic called `test_kafka_trigger` and you want to subscribe to the new messages there and trigger the pipeline UUID `8feaae75-7234-4f2f-9e4e-0b491e4a4331`

The .env variables will look like this:

```
YLEM_KT_API_CLIENT_ID=%%REPLACE_IT_WITH_YOUR_CLIENT_ID%%
YLEM_KT_API_CLIENT_SECRET=%%REPLACE_IT_WITH_YOUR_CLIENT_SECRET%%
YLEM_KT_API_URL=https://api.ylem.co
YLEM_KT_KAFKA_BOOTSTRAP_SERVERS="host.docker.internal:9092"
YLEM_KT_KAFKA_TOPIC_MAPPING="test_kafka_trigger:8feaae75-7234-4f2f-9e4e-0b491e4a4331"
```

### 4. Run the container

`docker-compose up --build`

If everything goes well, the service will be able to subscribe to your newly created topic and in the CLI output it will print something like that:

<img width="917" alt="352276443-650be831-8157-4b7b-9a87-6045be661f60" src="https://github.com/user-attachments/assets/38e7d755-4de1-4339-8dc5-fcbc94612d44">

### 5. Produce the new message

If you now produce a simple new message to the topic:

<img width="441" alt="352276803-946128fd-3a50-4aeb-929d-7e7d5e7a32bb" src="https://github.com/user-attachments/assets/514aefcc-e7c2-4034-9567-1dfff13f6ddc">

You will see in the CLI that Kafka trigger successfully consumed it and triggered the pipeline:

<img width="1180" alt="352277102-0774515f-134d-4961-83eb-d5533c39fe37" src="https://github.com/user-attachments/assets/6b2c3aca-3f5d-4db3-b66d-12f91b0b45bd">

And the pipeline logs also contain a new item stating that pipeline was successfully executed and each of two tasks returned a correct output:

<img width="1335" alt="352277352-4d092760-821b-4cd1-930e-39361a7c6d9f" src="https://github.com/user-attachments/assets/ee7305b8-da8f-4b51-9376-90c0e4000238">

<img width="1330" alt="352277402-dca0ed64-6aa9-4e55-b379-3d4b6907c7f0" src="https://github.com/user-attachments/assets/d197e86c-13e1-4b2d-a36d-b733ece4f580">
