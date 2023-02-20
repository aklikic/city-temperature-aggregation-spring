# Kalix Demo - City Temperature Aggregator - Spring
Not supported by Lightbend in any conceivable way, not open for contributions.<br>
## Prerequisite
Java 17<br>
Apache Maven 3.6 or higher<br>
[Kalix CLI](https://docs.kalix.io/kalix/install-kalix.html) <br>
Docker 20.10.8 or higher (client and daemon)<br>
Container registry with public access (like Docker Hub)<br>
Access to the `gcr.io/kalix-public` container registry<br>
cURL<br>
IDE / editor<br>
#Local test
##Setup local Zookeeper and Kafka
1. Start kafka and zookeeper <br>
```
docker-compose -f docker-compose-kafka.yaml up
```
3. Create Kafka topics <br>
[Open Kafka UI](http://localhost:8081/)
   1. Create input topic: `input`
   2. Create output topic: `output`
   
##Start Kalix Proxy
```
docker-compose up
```
##Start user function
```
mvn clean compile exec:exec
```
##Create one city
```
curl -XPOST -d '{ 
  "name": "Rotterdam",
  "aggregationLimit": 2,
  "aggregationTimeWindowSeconds": 60000
}' http://localhost:9000/city/rotterdam/create -H "Content-Type: application/json"
```
Note: `aggregationLimit` is 2 and `aggregationTimeWindowSeconds` is 1min

##Optional CURLs
1. Add temperature manually (optional for test):
```
curl -XPOST -d '{ 
  "recordId": "11111",
  "temperature": 20
}' http://localhost:9000/city/rotterdam/add-temperature -H "Content-Type: application/json"
```
2. Get city (optional for test):
```
curl -XGET http://localhost:9000/city/rotterdam -H "Content-Type: application/json"
```

##Test
[Open Kafka UI](http://localhost:8081/)
<br>
Produce message to `input` topic
- key does not need to be populated
- value: 
```
{
	"cityId": "rotterdam",
	"recordId": "22226",
	"temperature": "10",
	"timestamp": "2023-02-16T20:00:00.000Z"
}
```
- header:
```
{
	"ce-source": "manual",
	"ce-datacontenttype": "application/json",
	"ce-specversion": "1.0",
	"ce-type": "InputTemperatureMessage",
	"ce-id": "22226",
	"ce-time": "2023-02-16T20:00:00.000Z",
	"Content-Type": "application/json"
}
```

#Kalix deployment & test
##Confluent Cloud setup (using free tier)
1. Register: https://www.confluent.io/confluent-cloud/tryfree/
2. Add cluster:
   1. Cluster type: `Basic`
   2. Region/zones: `Google Cloud`, `N.Virginia (us-east4)`, Availability: `Single zone`
   3. Skip payment
   4. Cluster name: `kalix`
3. Create topics:
   1. `input` topic:
      1. Topic name: `input`
      2. Partitions: `2`
      3. Show advanced settings - Retention time: `1 hour` (just to have for testing)
   2. `output` topic:
      4. Topic name: `output`
      5. Partitions: `2`
      6. Show advanced settings - Retention time: `1 hour` (just to have for testing)
4. Export connection configiuration
   1. Clients - `New client`
   2. Choose language: `Java`
   3. `Create Kafka cluster API key`
   4. `Copy`
   5. Create file `confluent-kafka.properties` and paste copied content in

##Kalix project setup
1. Register for free
2. Create Kalix project:
```
kalix projects new city-temperature-aggregation --region gcp-us-east1
```
3. Set Kalix project in Kalix CLI
```
kalix config set project city-temperature-aggregation
```
4. Configure confluent kafka message broker in the Kalix project
```
kalix projects config set broker --broker-service kafka --broker-config-file confluent-kafka.properties
```
##Deploy to Kalix
###Configure container registry
`Note`: The most simple setup is to use public `hub.docker.com`. You just need to replace in `pom.xml`, `my-docker-repo` with your `dockerId`<br>
<br>
More options can be found here: <br>
https://docs.kalix.io/projects/container-registries.html
###Deploy
```
mvn deploy
```
`Note`: First deployment take few minutes for all required resources to be provisioned
##Kalix connection proxy
All services by default are not exposed to Internet and only local/private access is allowed. For local/private access Kalix connection proxy is used.<br>
Create Kalix connection proxy:
```
kalix service proxy city-temperature-aggregation-spring
```

##Create one city
```
curl -XPOST -d '{ 
  "name": "Rotterdam",
  "aggregationLimit": 2,
  "aggregationTimeWindowSeconds": 60000
}' http://localhost:8080/city/rotterdam/create -H "Content-Type: application/json"
```
Note: `aggregationLimit` is 2 and `aggregationTimeWindowSeconds` is 1min

##Optional CURLs
1. Add temperature manually (optional for test):
```
curl -XPOST -d '{ 
  "recordId": "11111",
  "temperature": 20
}' http://localhost:8080/city/rotterdam/add-temperature -H "Content-Type: application/json"
```
2. Get city (optional for test):
```
curl -XGET http://localhost:8080/city/rotterdam -H "Content-Type: application/json"
```

##Test
Unfortunately Confluent Cloud Console does not have an option to produce a message to a topic with headers so that feature can not be used.<>
So we are going to use REST API to produce an input message.

1. [Open Confluent Cloud](https://confluent.cloud/environments)
2. Client - `New client`
3. Choose your language: `REST API`
4. `Produce records using the Confluent Cloud REST API`
   1. Topic name: `input`
   2. Mode: `Non-streaming mode`
   3. `Create Kafka cluster API key`
   4. `Copy`
5. Produce message:
From copied CURL command replace set `-d @input_message.json`
```
 curl \
  -X POST \
  -H "Content-Type: application/json" \
  -H "Authorization: Basic XXXXXX" \
  https://<URL>:443/kafka/v3/clusters/lkc-nw619k/topics/input/records \
  -d @input_message.json 

```
`Note`: Header values need to be base64 encoded but you only need to change the `value` in json structure.<br>
6. Validate<br>
Check in Confluent cloud messages in `output` topic (You need to search by offset to be able to see all messages because consumer is configured with ). <br>
Check the service logs:
```
kalix service logs city-temperature-aggregation-spring
```