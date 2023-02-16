```
curl -XPOST -d '{ 
  "name": "Rotterdam",
  "aggregationLimit": 2,
  "aggregationTimeWindowSeconds": 60000
}' http://localhost:9000/city/rotterdam/create -H "Content-Type: application/json"
```

```
curl -XPOST -d '{ 
  "recordId": "11111",
  "temperature": 20
}' http://localhost:9000/city/rotterdam/add-temperature -H "Content-Type: application/json"
```

```
curl -XGET http://localhost:9000/city/rotterdam -H "Content-Type: application/json"
```

```
{
	"cityId": "rotterdam",
	"recordId": "22226",
	"temperature": "10",
	"timestamp": "2023-02-16T20:00:00.000Z"
}

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