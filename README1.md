# AVRO schema Deserialization interseptor

To read data or log a key field from an event before it encounters a deserialization exception, you can modify your Kafka consumer logic to handle potentially problematic data gracefully. This involves catching deserialization errors and logging key information from the event payload before the error occurs. Below are some approaches to achieve this:

### Approach 1: Use a Custom Deserializer Wrapper
Create a custom deserializer that wraps around your main deserializer. This wrapper logs key information from the raw event payload before attempting to deserialize it, so you can capture details if deserialization fails.

Custom Deserializer Example:

```java
import org.apache.kafka.common.serialization.Deserializer;
import org.apache.kafka.common.errors.SerializationException;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import java.util.Map;
import java.nio.charset.StandardCharsets;

public class LoggingDeserializer<T> implements Deserializer<T> {
    private final Deserializer<T> inner;

    public LoggingDeserializer(Deserializer<T> inner) {
        this.inner = inner;
    }

    @Override
    public void configure(Map<String, ?> configs, boolean isKey) {
        inner.configure(configs, isKey);
    }

    @Override
    public T deserialize(String topic, byte[] data) {
        try {
            return inner.deserialize(topic, data);
        } catch (SerializationException e) {
            // Log the raw data as a string before the deserialization exception
            String rawData = data != null ? new String(data, StandardCharsets.UTF_8) : "null";
            System.err.println("Deserialization failed for topic: " + topic + ". Raw data: " + rawData);
            throw e; // Re-throw the exception after logging
        }
    }

    @Override
    public void close() {
        inner.close();
    }
}
```

## Integrate the Custom Deserializer:

```java
import org.apache.kafka.clients.consumer.KafkaConsumer;
import java.util.Properties;

public class KafkaConsumerExample {
    public static void main(String[] args) {
        Properties props = new Properties();
        props.put("bootstrap.servers", "localhost:9092");
        props.put("group.id", "test-group");
        props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        props.put("value.deserializer", LoggingDeserializer.class.getName());
        props.put("schema.registry.url", "http://localhost:8081");

        // Use the Avro deserializer within the custom wrapper
        props.put("value.deserializer.inner", "io.confluent.kafka.serializers.KafkaAvroDeserializer");

        KafkaConsumer<String, Object> consumer = new KafkaConsumer<>(props);
        consumer.subscribe(Collections.singletonList("your_topic"));

        while (true) {
            ConsumerRecords<String, Object> records = consumer.poll(Duration.ofMillis(100));
            for (ConsumerRecord<String, Object> record : records) {
                try {
                    System.out.println("Key: " + record.key() + ", Value: " + record.value());
                } catch (Exception e) {
                    System.err.println("Error processing record with key: " + record.key());
                }
            }
        }
    }
}

```

### Approach 2: Log Raw Data Using an Interceptor
Kafka supports interceptors that can be used to inspect messages before they are deserialized. An interceptor can log the raw byte array of incoming records, helping diagnose deserialization issues.

## Create a Consumer Interceptor:

```java
import org.apache.kafka.clients.consumer.ConsumerInterceptor;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.common.Configurable;

import java.util.Map;

public class LoggingInterceptor<K, V> implements ConsumerInterceptor<K, V>, Configurable {

    @Override
    public ConsumerRecords<K, V> onConsume(ConsumerRecords<K, V> records) {
        for (ConsumerRecord<K, V> record : records) {
            if (record.value() == null) {
                System.out.println("Raw data (pre-deserialization) key: " + record.key());
            }
        }
        return records;
    }

    @Override
    public void onCommit(Map offsets) {
        // No action needed
    }

    @Override
    public void close() {
        // No action needed
    }

    @Override
    public void configure(Map<String, ?> configs) {
        // Configure the interceptor if necessary
    }
}

```

### Approach 3: Deserialize with a Catch Block in the Consumer
You can add a catch block around the record.value() call when processing records to catch any deserialization errors.

## Example:

```java
while (true) {
    ConsumerRecords<String, Object> records = consumer.poll(Duration.ofMillis(100));
    for (ConsumerRecord<String, Object> record : records) {
        try {
            System.out.println("Key: " + record.key());
            System.out.println("Value: " + record.value());
        } catch (SerializationException e) {
            // Log key and raw byte array of the value before the deserialization failed
            System.err.println("Deserialization error for record with key: " + record.key());
            String rawValue = new String(record.valueBytes(), StandardCharsets.UTF_8);
            System.err.println("Raw value bytes: " + rawValue);
        }
    }
}

```

### Summary:
##### Custom Deserializer: Create a deserializer wrapper that logs data before deserialization.
##### Interceptor:  Use a consumer interceptor for pre-deserialization logging.
##### Catch Block:  Add a try-catch around the deserialization logic to capture exceptions and log details.

These methods help you capture important details from events before they run into deserialization errors, facilitating debugging and understanding problematic records.
