# Event-driven microservice choreography with Eventuate + Spring Data REST

## How to run

- firstly install and run infrastructure including the kafka, mysql, eventuate cdc
```
cd kafka
docker-compose up
```

- run each microservice
```
cd order
mvn spring-boot:run

#another terminal
cd inventory
mvn spring-boot:run
```

- test the service:

register a product with stock 10 and create an order for the product:
```
http :8082/inventories id=1 stock=10
http :8081/orders productId=1 qty=5
```

check the stock remaining:
```
http :8082/inventories/1    #stock must be 5
```

- check the database:

```
docker exec -it kafka-mysql-1 /bin/bash

mysql --user=root --password=$MYSQL_ROOT_PASSWORD
use eventuate;
select * from message;
```

you can see:
```
+-----------------------------------+-------------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------+-----------+-------------------+---------------+
| id                                | destination                   | headers                                                                                                                                                                                                                                                                               | payload | published | message_partition | creation_time |
+-----------------------------------+-------------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------+-----------+-------------------+---------------+
| 00000183c7c93c05-f2d3907995f30000 | labshopeventuate.domain.Order | {"PARTITION_ID":"1","event-aggregate-type":"labshopeventuate.domain.Order","DATE":"Tue, 11 Oct 2022 16:03:17 GMT","event-aggregate-id":"1","event-type":"labshopeventuate.domain.OrderPlaced","DESTINATION":"labshopeventuate.domain.Order","ID":"00000183c7c93c05-f2d3907995f30000"} | {}      |         0 |              NULL | 1665504197650 |
+-----------------------------------+-------------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------+-----------+-------------------+---------------+
```

you can find the kafka log:

```
docker exec -it kafka-kafka-1 /bin/bash
cd /bin

kafka-topics --bootstrap-server=localhost:9092 --list
kafka-console-consumer --bootstrap-server localhost:9092 --topic labshopeventuate.domain.Order --from-beginning
kafka-console-consumer --bootstrap-server localhost:9092 --topic labshopeventuate.domain.Order --from-beginning
kafka-consumer-groups --bootstrap-server localhost:9092 --group inventory --topic labshopeventuate.domain.Order --reset-offsets --to-last --execute
```

you can see:
```
{"payload":"{}","headers":{"PARTITION_ID":"1","event-aggregate-type":"labshopeventuate.domain.Order","DATE":"Tue, 11 Oct 2022 16:03:17 GMT","event-aggregate-id":"1","event-type":"labshopeventuate.domain.OrderPlaced","DESTINATION":"labshopeventuate.domain.Order","ID":"00000183c7c93c05-f2d3907995f30000"}}
```


Possible Errors:

- org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'domainEventDispatcher' defined in labshopeventuate.InventoryApplication: Unsatisfied dependency expressed through method 'domainEventDispatcher' parameter 0; nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'domainEventDispatcherFactory' defined in class path resource [io/eventuate/tram/spring/events/subscriber/TramEventSubscriberConfiguration.class]: Unsatisfied dependency expressed through method 'domainEventDispatcherFactory' parameter 0; nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'messageConsumer' defined in class path resource [io/eventuate/tram/spring/consumer/common/TramConsumerCommonConfiguration.class]: Unsatisfied dependency expressed through method 'messageConsumer' parameter 0; nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'messageConsumerImplementation' defined in class path resource [io/eventuate/tram/spring/consumer/kafka/EventuateTramKafkaMessageConsumerConfiguration.class]: Unsatisfied dependency expressed through method 'messageConsumerImplementation' parameter 0; nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'messageConsumerKafka' defined in class path reso

- when you missed to configure eventuate to point to the kafka in the application.yaml



내부에 뭔가 이상한 토픽에다가 mysql 과 동기화하는 작업에 대한 값을 관리한다:
```
./kafka-console-consumer --bootstrap-server localhost:9092 --topic offset.storage.topic --from-beginning

{"binlogFilename":"mysql-bin.000003","offset":152871,"rowsToSkip":0}
{"binlogFilename":"mysql-bin.000003","offset":153055,"rowsToSkip":0}
{"binlogFilename":"mysql-bin.000003","offset":153223,"rowsToSkip":0}
{"binlogFilename":"mysql-bin.000003","offset":153407,"rowsToSkip":0}
{"binlogFilename":"mysql-bin.000003","offset":153575,"rowsToSkip":0}
{"binlogFilename":"mysql-bin.000003","offset":153759,"rowsToSkip":0}
{"binlogFilename":"mysql-bin.000003","offset":153927,"rowsToSkip":0}
{"binlogFilename":"mysql-bin.000003","offset":154111,"rowsToSkip":0}
{"binlogFilename":"mysql-bin.000003","offset":154279,"rowsToSkip":0}
{"binlogFilename":"mysql-bin.000003","offset":154463,"rowsToSkip":0}
{"binlogFilename":"mysql-bin.000003","offset":154631,"rowsToSkip":0}
{"binlogFilename":"mysql-bin.000003","offset":154815,"rowsToSkip":0}
```