# Replicated Log Task

Team 6:
- Artem Avetian
- Dmytro Kushnir

## Simple Echo-Server application
The `echo-server.py` implements simple Echo-Server application. Accept requests on the *8080* port. Supports only GET and POST requests.

### Usage notes
1. Start Echo-Server application
```
chmod echo-server.py
./echo-server.py
```

2. Send GET requests:
```
curl localhost:8080
curl "localhost:8080?key1=value1"
curl "localhost:8080?key1=value1&key2=value2"
curl "localhost:8080?key1=value1&key2=value2&key3=value3"
```

3. Send POST requests with JSON data:
```
curl -X POST localhost:8080 -H 'Content-Type: application/json' -d '{"key1":"value1"}'
curl -X POST localhost:8080 -H 'Content-Type: application/json' -d '{"key1":"value1", "key2":"value2"}'
curl -X POST localhost:8080 -H 'Content-Type: application/json' -d '{"key1":"value1", "key2":"value2", "key3":"value3"}'
```

## Replicated Log
The Replicated Log implementation with the following architecture: one Master and any number of Secondaries.

Master exposes simple HTTP server on *8080* port.

Secondary exposes simple HTTP server *8081* and *8082* ports with.

### Usage notes:
1. Setting up
```
docker-compose build
docker-compose up
```

2. Master
- GET method - returns all messages from the in-memory list
```
curl localhost:8080
```
- POST method - appends a message *msg* into the in-memory list and accepts the following JSON format: `{"msg":"value1"}`
```
curl -X POST localhost:8080 -H 'Content-Type: application/json' -d '{"msg":"test value 1"}'
```
- POST method - appends a message *msg* with write concern *w=1,2,3,..,n* into the in-memory list and accepts the following JSON format: `{"msg":"value1", "w":1,2,3,..,n}`
```
curl -X POST localhost:8080 -H 'Content-Type: application/json' -d '{"msg":"test value 1", "w":1}'
curl -X POST localhost:8080 -H 'Content-Type: application/json' -d '{"msg":"test value 1", "w":2}'
curl -X POST localhost:8080 -H 'Content-Type: application/json' -d '{"msg":"test value 1", "w":3}'
```
- POST method - appends a message *msg* with delay on Secondary
```
curl -X POST localhost:8080 -H 'Content-Type: application/json' -d '{"msg":"wait", "w":3}'
```
- GET /health method - check secondaries’ health status
```
curl localhost:8080/health
```

3. Secondaries
- GET method - returns all replicated messages from the in-memory list
```
curl localhost:8081
curl localhost:8082
```

4. Testing eventual consistency, exactly-once delivering, total order, deduplication
```
docker network disconnect replicated-log_my-net secondary1
docker network connect replicated-log_my-net secondary1
```

5. Stop
```
docker-compose down
```

6. Clean-Up
```
docker-compose down --rmi 'all'
```

