# I. AVRO schema Deserialization interseptor

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

---
# II. Complex AVRO schema conflicts finder
Deeper nested field comparisons in Avro schemas, we need to create a method that recursively checks nested types (records, arrays, maps, unions, etc.). Here's an updated version of the code with deeper nested field comparison:

```java
import org.apache.avro.Schema;
import org.apache.avro.Schema.Field;
import org.apache.avro.SchemaCompatibility;
import org.apache.avro.SchemaCompatibility.SchemaCompatibilityType;
import java.io.File;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Set;
import java.util.stream.Collectors;

public class AvroSchemaConflictChecker {

    public static void main(String[] args) {
        String schemaDirectoryPath = "path/to/your/schemas"; // Replace with your schema directory
        Map<String, Schema> schemas = loadSchemas(schemaDirectoryPath);

        if (schemas.isEmpty()) {
            System.out.println("No schemas found in the directory.");
            return;
        }

        boolean conflictsFound = false;
        for (Map.Entry<String, Schema> entry1 : schemas.entrySet()) {
            for (Map.Entry<String, Schema> entry2 : schemas.entrySet()) {
                if (!entry1.getKey().equals(entry2.getKey())) {
                    Schema schema1 = entry1.getValue();
                    Schema schema2 = entry2.getValue();
                    SchemaCompatibilityType compatibilityType = SchemaCompatibility.checkReaderWriterCompatibility(schema1, schema2)
                            .getType();

                    if (compatibilityType != SchemaCompatibilityType.COMPATIBLE) {
                        conflictsFound = true;
                        System.out.println("Conflict found between schemas:");
                        System.out.println(" - Schema 1: " + entry1.getKey());
                        System.out.println(" - Schema 2: " + entry2.getKey());
                        System.out.println(" - Conflict Type: " + compatibilityType);
                        printConflictingFieldsRecursive(schema1, schema2, "");
                        System.out.println();
                    }
                }
            }
        }

        if (!conflictsFound) {
            System.out.println("No conflicts detected among schemas.");
        }
    }

    private static void printConflictingFieldsRecursive(Schema schema1, Schema schema2, String path) {
        if (schema1.getType() != schema2.getType()) {
            System.out.println("   - Field path '" + path + "' type differs:");
            System.out.println("     - In Schema 1: " + schema1.getType());
            System.out.println("     - In Schema 2: " + schema2.getType());
            return;
        }

        switch (schema1.getType()) {
            case RECORD:
                List<Field> fields1 = schema1.getFields();
                List<Field> fields2 = schema2.getFields();

                Map<String, Field> fieldsMap1 = fields1.stream()
                        .collect(Collectors.toMap(Field::name, field -> field));
                Map<String, Field> fieldsMap2 = fields2.stream()
                        .collect(Collectors.toMap(Field::name, field -> field));

                Set<String> allFieldNames = fieldsMap1.keySet();
                allFieldNames.addAll(fieldsMap2.keySet());

                for (String fieldName : allFieldNames) {
                    Field field1 = fieldsMap1.get(fieldName);
                    Field field2 = fieldsMap2.get(fieldName);

                    if (field1 == null) {
                        System.out.println("   - Field path '" + path + "." + fieldName + "' is missing in Schema 1.");
                    } else if (field2 == null) {
                        System.out.println("   - Field path '" + path + "." + fieldName + "' is missing in Schema 2.");
                    } else {
                        printConflictingFieldsRecursive(field1.schema(), field2.schema(), path + "." + fieldName);
                    }
                }
                break;

            case ARRAY:
                printConflictingFieldsRecursive(schema1.getElementType(), schema2.getElementType(), path + "[]");
                break;

            case MAP:
                printConflictingFieldsRecursive(schema1.getValueType(), schema2.getValueType(), path + "{}");
                break;

            case UNION:
                List<Schema> types1 = schema1.getTypes();
                List<Schema> types2 = schema2.getTypes();

                for (int i = 0; i < Math.min(types1.size(), types2.size()); i++) {
                    printConflictingFieldsRecursive(types1.get(i), types2.get(i), path + ".<union>");
                }
                break;

            default:
                if (!schema1.equals(schema2)) {
                    System.out.println("   - Field path '" + path + "' differs in definition:");
                    System.out.println("     - In Schema 1: " + schema1.toString(true));
                    System.out.println("     - In Schema 2: " + schema2.toString(true));
                }
                break;
        }
    }

    private static Map<String, Schema> loadSchemas(String directoryPath) {
        Map<String, Schema> schemas = new HashMap<>();
        try {
            Files.walk(Paths.get(directoryPath))
                    .filter(Files::isRegularFile)
                    .filter(path -> path.toString().endsWith(".avsc"))
                    .forEach(path -> {
                        try {
                            String schemaJson = new String(Files.readAllBytes(path));
                            Schema schema = new Schema.Parser().parse(schemaJson);
                            schemas.put(path.getFileName().toString(), schema);
                        } catch (IOException e) {
                            System.err.println("Error reading schema from file: " + path.getFileName());
                            e.printStackTrace();
                        }
                    });
        } catch (IOException e) {
            System.err.println("Error reading schemas from directory: " + directoryPath);
            e.printStackTrace();
        }
        return schemas;
    }
}


```

## Explanation of Recursive Comparison
##### Recursive Field Check:

The printConflictingFieldsRecursive method checks nested fields by recursively diving into RECORD, ARRAY, MAP, and UNION types.
For RECORD types, it checks each field by name and recurses further if both schemas have a field with the same name.
For ARRAY and MAP types, it checks the element and value types, respectively.
For UNION types, it checks each type within the union in order.
Path Tracking:

The path variable keeps track of the current field path (e.g., record1.field.subfield) to help identify the exact location of conflicts.
Field Type Differences:

If the type of a field differs between two schemas, the method prints a message indicating the type difference.
Missing Fields:

The method also reports if a field exists in one schema but not in the other.

## Example Output

```css
Conflict found between schemas:
 - Schema 1: schema1.avsc
 - Schema 2: schema2.avsc
 - Conflict Type: INCOMPATIBLE
   - Field path '.userDetails.address' differs in definition:
     - In Schema 1: {"type":"string"}
     - In Schema 2: {"type":"int"}
   - Field path '.userDetails.contactInfo.phoneNumber[]' differs in definition:
     - In Schema 1: {"type":"int"}
     - In Schema 2: {"type":"string"}
   - Field path '.orderDetails[].items[].price' type differs:
     - In Schema 1: FLOAT
     - In Schema 2: DOUBLE
```
This recursive method ensures that the comparison checks every nested level in the schemas and reports all field conflicts.

---

# I. AVRO schema Deserialization interseptor

The error java.lang.ArrayIndexOutOfBoundsException: null during Avro message deserialization typically indicates an issue with the way the Avro schema or data is handled. Here are some common causes and steps to troubleshoot:

## 1. Schema Mismatch
Cause: The producer (writer) and consumer (reader) are using incompatible schemas. Avro requires that schemas be compatible for deserialization.
### Fix:
Verify that both producer and consumer are using the same schema or compatible schemas.
If using a Schema Registry (e.g., Confluent Schema Registry):
Check if the schema registered matches the one being used.
Use tools like curl to fetch and validate the schema from the registry.
## 2. Incorrect Avro Decoder
Cause: The deserializer may not be using the correct Avro decoder, or the data being read is not properly formatted.
## Fix:
Ensure you’re using the correct deserializer for your Avro data. For example:

```java
DatumReader<GenericRecord> reader = new GenericDatumReader<>(schema);
Decoder decoder = DecoderFactory.get().binaryDecoder(data, null);
GenericRecord record = reader.read(null, decoder);

```

Confirm the input data is valid Avro-encoded data.
## 3. Corrupted or Incomplete Data
Cause: The Avro message being deserialized may be corrupted, truncated, or improperly serialized.
### Fix:
Check the source of the data (e.g., Kafka, file, etc.) to ensure that the messages are complete and correctly serialized.
Validate the Avro binary data using a tool like avro-tools:

```java
java -jar avro-tools-<version>.jar tojson --schema-file <schema.avsc> <data.avro>

```

## 4. Incorrect Byte Offsets
Cause: The deserializer might be starting at the wrong byte position in the Avro binary.
### Fix:
If using Kafka, check if the consumer is correctly handling the byte offsets.
For Schema Registry-based systems, verify the prefix byte handling (often the first few bytes represent schema ID).
## 5. Null Schema or Missing Schema Metadata
Cause: The deserialization process does not find a schema or receives a null schema ID.
## Fix:
Verify that the schema is available and correctly registered in the Schema Registry (if applicable).
If not using a Schema Registry, ensure that the schema is correctly passed to the deserialization logic.

## 6. Array Indexing Issue
Cause: If your schema includes arrays, an ArrayIndexOutOfBoundsException could result from reading array data with incorrect indexing or corrupted metadata.
### Fix:
Check the schema definition for array fields.
Inspect the data to ensure it conforms to the expected structure and length.
Debugging Steps:
Log More Information: Add debug logs to capture details of the schema and message during deserialization.
Test with Avro Tools:
Convert the data to JSON to validate its structure.
## Example

```java
java -jar avro-tools-<version>.jar tojson --schema-file <schema.avsc> <data.avro>

```

## Inspect Schema Registry:
Check the schema compatibility settings (BACKWARD, FORWARD, FULL).
## Reproduce in Isolation:
Create a standalone test with the schema and data in question to reproduce the error.
If the problem persists after these steps, share additional details such as the schema, data format, and deserialization code for more specific advice.

---

Here’s how you can use avro-tools to work with Avro data, diagnose issues, and inspect its contents.

# Download Avro Tools
You can download the Avro Tools JAR from the official Apache Avro website. (https://avro.apache.org/)

## 1. Validate an Avro File
To ensure your Avro file conforms to its schema:

```java
java -jar avro-tools-<version>.jar getschema <data.avro>

```
### Input: data.avro is the binary Avro file.
### Output: The schema embedded in the Avro file will be displayed.
## 2. Convert Avro to JSON
This helps to inspect the content of an Avro file in a human-readable JSON format:

```java
java -jar avro-tools-<version>.jar tojson --schema-file <schema.avsc> <data.avro>

```

## Input:
### schema.avsc: The Avro schema file.
### data.avro: The Avro file to be converted.
### Output: Prints JSON-formatted data to the console.
### Example:
```bash
java -jar avro-tools-1.11.0.jar tojson --schema-file user.avsc user.avro
```
If user.avro contains:
```avro
{"name": "John", "age": 30, "email": "john.doe@example.com"}

```
It will print:

```json
{"name":"John","age":30,"email":"john.doe@example.com"}

```

## 3. Validate Schema Compatibility
Check whether a new schema is compatible with an existing schema (useful in Schema Registry setups):

```bash
java -jar avro-tools-<version>.jar compat --reader-schema <new-schema.avsc> --writer-schema <old-schema.avsc>

```
## Input:
### new-schema.avsc: The new schema to test.
### old-schema.avsc: The existing schema.
### Output: Prints whether the schemas are compatible.

## 4. Convert JSON to Avro
If you have a JSON file and want to encode it into Avro:

```bash
java -jar avro-tools-<version>.jar fromjson --schema-file <schema.avsc> <data.json> > <output.avro>

```
## Input:
### schema.avsc: The Avro schema.
### data.json: The JSON file containing data.
### Output: Encoded binary Avro file.

## 5. Extract Data with Specific Schema
If you have an Avro file and want to deserialize it using a specific schema:

```bash
java -jar avro-tools-<version>.jar tojson --schema-file <desired-schema.avsc> <data.avro>

```
This is useful when the schema used for writing and reading are different (but compatible).

## Example Workflow
### 1. Inspect the Schema:
```bash
java -jar avro-tools-1.11.0.jar getschema data.avro
```
### 2. Convert Avro to JSON:
```bash
java -jar avro-tools-1.11.0.jar tojson --schema-file schema.avsc data.avro

```
### 3. Check Compatibility:
```bash
java -jar avro-tools-1.11.0.jar compat --reader-schema new-schema.avsc --writer-schema old-schema.avsc

```
Using avro-tools can help identify whether your issue lies with the data structure, the schema compatibility, or how the binary data is being serialized/deserialized.

---
# 1. Here's a Java example that extracts and prints all field paths from a nested Avro schema
```java
import org.apache.avro.Schema;
import org.apache.avro.Schema.Field;

public class AvroSchemaFieldPathsExtractor {

    public static void main(String[] args) {
        // Example schema as a JSON string
        String schemaJson = "{"
                + "\"type\": \"record\","
                + "\"name\": \"Person\","
                + "\"fields\": ["
                + "  {\"name\": \"name\", \"type\": \"string\"},"
                + "  {\"name\": \"age\", \"type\": \"int\"},"
                + "  {\"name\": \"address\", \"type\": {"
                + "    \"type\": \"record\","
                + "    \"name\": \"Address\","
                + "    \"fields\": ["
                + "      {\"name\": \"city\", \"type\": \"string\"},"
                + "      {\"name\": \"zipcode\", \"type\": \"int\"}"
                + "    ]"
                + "  }},"
                + "  {\"name\": \"phoneNumbers\", \"type\": {"
                + "    \"type\": \"array\","
                + "    \"items\": \"string\""
                + "  }},"
                + "  {\"name\": \"metadata\", \"type\": {"
                + "    \"type\": \"map\","
                + "    \"values\": \"string\""
                + "  }},"
                + "  {\"name\": \"preferences\", \"type\": [\"null\", {"
                + "    \"type\": \"enum\","
                + "    \"name\": \"Preference\","
                + "    \"symbols\": [\"HIGH\", \"MEDIUM\", \"LOW\"]"
                + "  }]},"
                + "  {\"name\": \"identifier\", \"type\": {"
                + "    \"type\": \"fixed\", \"name\": \"ID\", \"size\": 16"
                + "  }}"
                + "]"
                + "}";

        Schema schema = new Schema.Parser().parse(schemaJson);
        System.out.println("Field Paths and Their Types:");
        extractFieldPathsAndTypes(schema, "");
    }

    /**
     * Recursively extracts and prints field paths and their types from an Avro schema.
     *
     * @param schema     the current schema
     * @param parentPath the current path prefix
     */
    private static void extractFieldPathsAndTypes(Schema schema, String parentPath) {
        switch (schema.getType()) {
            case RECORD:
                for (Field field : schema.getFields()) {
                    String fieldPath = parentPath.isEmpty() ? field.name() : parentPath + "." + field.name();
                    extractFieldPathsAndTypes(field.schema(), fieldPath);
                }
                break;

            case ARRAY:
                String arrayPath = parentPath + "[]";
                System.out.println(arrayPath + " : array of " + schema.getElementType().getType());
                extractFieldPathsAndTypes(schema.getElementType(), arrayPath);
                break;

            case MAP:
                String mapPath = parentPath + "<>";
                System.out.println(mapPath + " : map of " + schema.getValueType().getType());
                extractFieldPathsAndTypes(schema.getValueType(), mapPath);
                break;

            case UNION:
                for (Schema subSchema : schema.getTypes()) {
                    if (subSchema.getType() != Schema.Type.NULL) { // Skip 'null' in union
                        extractFieldPathsAndTypes(subSchema, parentPath);
                    }
                }
                break;

            case ENUM:
                System.out.println(parentPath + " : enum (" + schema.getEnumSymbols() + ")");
                break;

            case FIXED:
                System.out.println(parentPath + " : fixed (size=" + schema.getFixedSize() + ")");
                break;

            default:
                // Handle primitive types like STRING, INT, BOOLEAN, etc.
                System.out.println(parentPath + " : " + schema.getType());
                break;
        }
    }
}

```
---




