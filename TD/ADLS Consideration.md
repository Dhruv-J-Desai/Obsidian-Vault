Absolutely—let’s add **custom headers** to the messages you’re already producing with `KafkaTemplate` and read them in your consumer.

Below are two drop-in ways; pick one and paste into your project.

---

# Option A: Use `MessageBuilder` (clean & Spring-y)

### Producer (in your `DataGeneratorService`)

```java
import static java.nio.charset.StandardCharsets.UTF_8;
import java.time.Instant;
import java.util.UUID;
import org.springframework.messaging.support.MessageBuilder;
import org.springframework.kafka.support.KafkaHeaders;

public void produceEmployeeData(String topicName) {
    try {
        for (int i = 0; i < 10; i++) {
            Map<String, Object> employee = new HashMap<>();
            // ... fill your employee map (empId, name, etc.)
            String empId = "EMP" + (1000 + new Random().nextInt(9000));
            employee.put("employeeId", empId);

            String traceId = UUID.randomUUID().toString();

            var msg = MessageBuilder.withPayload(employee)
                .setHeader(KafkaHeaders.TOPIC, topicName)
                .setHeader(KafkaHeaders.MESSAGE_KEY, empId)              // partitioning key
                // custom headers (all become byte[] on the wire)
                .setHeader("x-team",          "risk")
                .setHeader("x-source",        "employee-generator")
                .setHeader("x-produced-at",   Instant.now().toString())
                .setHeader("x-schema-version","v1")
                .setHeader("x-trace-id",      traceId)
                .build();

            kafkaTemplate.send(msg);
            log.info("Produced employee data: {}", employee);
        }
    } catch (Exception e) {
        log.error("Produce failed", e);
    }
}
```

### Consumer (read headers + metadata)

```java
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.messaging.handler.annotation.Header;
import org.springframework.messaging.handler.annotation.Payload;
import org.springframework.kafka.support.KafkaHeaders;

@KafkaListener(topics = "${topic.poc_only}", groupId = "${consumer_group}")
public void consumeEmployeeData(
        @Payload Map<String, Object> employeeData,
        @Header(name = "x-team",           required = false) String team,
        @Header(name = "x-source",         required = false) String source,
        @Header(name = "x-produced-at",    required = false) String producedAt,
        @Header(name = "x-schema-version", required = false) String schemaVer,
        @Header(name = "x-trace-id",       required = false) String traceId,
        @Header(KafkaHeaders.RECEIVED_TOPIC) String topic,
        @Header(KafkaHeaders.RECEIVED_PARTITION) int partition,
        @Header(KafkaHeaders.OFFSET) long offset,
        @Header(KafkaHeaders.RECEIVED_TIMESTAMP) long brokerTimestamp
) {
    log.info("Consumed employee data: {}", employeeData);
    log.info("meta team={}, source={}, producedAt={}, schema={}, traceId={}, topic={}, partition={}, offset={}, brokerTs={}",
            team, source, producedAt, schemaVer, traceId, topic, partition, offset, brokerTimestamp);
}
```

---

# Option B: Use `ProducerRecord` (full Kafka control)

### Producer

```java
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.common.header.internals.RecordHeaders;
import static java.nio.charset.StandardCharsets.UTF_8;
import java.time.Instant;
import java.util.UUID;

public void produceEmployeeData(String topicName) {
    try {
        for (int i = 0; i < 10; i++) {
            Map<String, Object> employee = new HashMap<>();
            // ... fill employee
            String empId = "EMP" + (1000 + new Random().nextInt(9000));
            employee.put("employeeId", empId);

            var headers = new RecordHeaders()
                .add("x-team",          "risk".getBytes(UTF_8))
                .add("x-source",        "employee-generator".getBytes(UTF_8))
                .add("x-produced-at",   Instant.now().toString().getBytes(UTF_8))
                .add("x-schema-version","v1".getBytes(UTF_8))
                .add("x-trace-id",      UUID.randomUUID().toString().getBytes(UTF_8));

            // if your KafkaTemplate value serializer handles Map -> JSON, this is fine
            var rec = new ProducerRecord<String, Object>(
                topicName,
                null,                               // partition (null = default)
                System.currentTimeMillis(),         // record timestamp
                empId,                              // key
                employee,                           // value (Map)
                headers
            );

            kafkaTemplate.send(rec);
            log.info("Produced employee data: {}", employee);
        }
    } catch (Exception e) {
        log.error("Produce failed", e);
    }
}
```

### Consumer (raw record if you prefer)

```java
import org.apache.kafka.clients.consumer.ConsumerRecord;

@KafkaListener(topics = "${topic.poc_only}", groupId = "${consumer_group}")
public void consumeEmployeeData(ConsumerRecord<String, Map<String,Object>> record) {
    var headers = record.headers();
    String team      = header(headers, "x-team");
    String source    = header(headers, "x-source");
    String produced  = header(headers, "x-produced-at");
    String schemaVer = header(headers, "x-schema-version");
    String traceId   = header(headers, "x-trace-id");

    log.info("Employee: {}", record.value());
    log.info("meta team={}, source={}, producedAt={}, schema={}, traceId={}, topic={}, partition={}, offset={}, ts={}",
             team, source, produced, schemaVer, traceId,
             record.topic(), record.partition(), record.offset(), record.timestamp());
}

private static String header(org.apache.kafka.common.header.Headers hs, String key) {
    var h = hs.lastHeader(key);
    return h == null ? null : new String(h.value(), java.nio.charset.StandardCharsets.UTF_8);
}
```

---

## Notes / gotchas

- **Serializers**: you’re sending `Map<String,Object>`; ensure your producer uses a JSON serializer (Spring’s `JsonSerializer` or your secure JSON serializer) and consumer uses a matching deserializer.
    
- **Header sizes**: keep headers small (a few bytes to a few hundred bytes), they count toward message size.
    
- **Partitioning**: headers don’t affect partition; your **key** (e.g., `employeeId`) does.
    
- **DLQ**: if you use Spring’s `DeadLetterPublishingRecoverer`, configure it to **copy headers** so `traceId` etc. survive in DLQ.
    

If you paste your current producer `KafkaTemplate` bean config (serializers), I’ll align the snippets exactly to your setup.