#Instructions:
1. Start kafka and zookeeper <br>
```
docker-compose -f docker-compose-kafka.yaml up
```
3. Create Kafka topics <br>
[Open Kafka UI](http://localhost:8081/)
   1. Create input topic: `input`
   2. Create output topic: `output`
4. Start proxy:
```
docker-compose up
```
6. Start user function
```
mvn clean compile exec:exec
```
7. Create city:
```
curl -XPOST -d '{ 
  "name": "Rotterdam",
  "aggregationLimit": 2,
  "aggregationTimeWindowSeconds": 60000
}' http://localhost:9000/city/rotterdam/create -H "Content-Type: application/json"
```
Note: `aggregationLimit` is 2 and `aggregationTimeWindowSeconds` is 1min

4. Add temperature manually (optional for test):
```
curl -XPOST -d '{ 
  "recordId": "11111",
  "temperature": 20
}' http://localhost:9000/city/rotterdam/add-temperature -H "Content-Type: application/json"
```
5. Get city (optional for test):
```
curl -XGET http://localhost:9000/city/rotterdam -H "Content-Type: application/json"
```
7. Produce message to `input` topic
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