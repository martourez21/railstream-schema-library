# Rail Stream Avro Schema Library

This library contains Avro schemas and generated Java classes for the Rail Stream IoT data processing platform. It provides the contract definitions for data flowing through Kafka topics, ensuring type safety and schema evolution compatibility.

## 📋 Overview

The library defines the data models used across the Rail Stream platform, particularly for Flink stream processing and Kafka message serialization/deserialization. All schemas are registered with Confluent Schema Registry for compatibility enforcement.

## 📊 Schema Definitions

### SensorData
**Topic:** `sensor-raw-data`  
**Description:** Raw telemetry data from industrial IoT sensors.

**Fields:**
- `sensorId` (string): Unique sensor identifier
- `equipmentId` (string): Associated equipment identifier
- `timestamp` (long): Unix timestamp in seconds
- `temperature` (double): Temperature reading
- `pressure` (double, optional): Pressure reading
- `unit` (string): Measurement unit (Celsius, Fahrenheit, PSI)
- `location` (string): Physical location
- `status` (enum): Sensor status (ONLINE, OFFLINE, ERROR, MAINTENANCE)
- `metadata` (map<string, string>, optional): Additional metadata

### SensorOutput
**Topic:** `aggregated-sensor-metrics`  
**Description:** Windowed aggregates of sensor data for monitoring.

**Fields:**
- `equipmentId` (string): Equipment identifier
- `windowStart` (long): Window start time (ms)
- `windowEnd` (long): Window end time (ms)
- `averageTemperature` (double): Mean temperature
- `maxTemperature` (double): Maximum temperature
- `minTemperature` (double): Minimum temperature
- `sensorCount` (int): Number of sensors aggregated
- `unit` (string): Measurement unit
- `location` (string): Equipment location
- `processingTime` (long): Processing timestamp
- `anomalyScore` (double, optional): Anomaly detection score

### AlertEvent
**Topic:** `sensor-alerts`  
**Description:** Alert events generated from anomaly detection.

**Fields:**
- `alertId` (string): Unique alert identifier
- `sensorId` (string): Triggering sensor ID
- `equipmentId` (string): Associated equipment
- `timestamp` (long): Alert generation time
- `temperature` (double): Temperature value
- `threshold` (double): Threshold exceeded
- `location` (string): Sensor location
- `severity` (enum): Alert severity (LOW, MEDIUM, HIGH, CRITICAL)
- `message` (string): Human-readable message
- `acknowledged` (boolean): Acknowledgement status

## 🚀 Quick Start

### Prerequisites

- Java 11 or higher
- Maven 3.6+
- Confluent Schema Registry (for schema management)

### Installation

1. **Clone and build the library:**
```bash
git clone https://github.com/martourez21/railstream-schema-library.git
cd railstream-schema-library
mvn clean install
```

2. **Add dependency to your project:**
```xml
<dependency>
    <groupId>com.railstream</groupId>
    <artifactId>railstream-schema-library</artifactId>
    <version>1.1.0</version>
</dependency>
```

### Schema Registration

Schemas are automatically registered with Schema Registry during the first time they're used. For manual registration:

```bash
# Register SensorData schema
curl -X POST -H "Content-Type: application/vnd.schemaregistry.v1+json" \
  --data @src/main/avro/SensorData.avsc \
  http://schema-registry:8081/subjects/sensor-raw-data-value/versions

# Register other schemas similarly...
```

## 🔧 Development

### Generating Java Classes

Java classes are automatically generated during the build process using the Avro Maven plugin. To generate manually:

```bash
mvn generate-sources
```

### Adding New Schemas

1. Create new `.avsc` file in `src/main/avro/`
2. Follow Avro schema syntax
3. Rebuild the project: `mvn clean compile`
4. The corresponding Java class will be generated automatically

### Schema Evolution

This library supports backward and forward compatible schema evolution:

**Backward Compatible Changes (safe):**
- Adding new fields with default values
- Removing fields (if consumers ignore them)

**Forward Compatible Changes (safe):**
- Adding new fields (consumers ignore unknown)
- Removing fields with default values

**Breaking Changes (avoid):**
- Changing field types
- Renaming required fields
- Removing required fields

## 📖 Usage Examples

### Flink Job Usage

```java
import com.railstream.schemas.SensorData;
import com.railstream.schemas.SensorOutput;
import org.apache.flink.formats.avro.registry.confluent.ConfluentRegistryAvroDeserializationSchema;

// Kafka source with Avro deserialization
KafkaSource<SensorData> source = KafkaSource.<SensorData>builder()
    .setValueOnlyDeserializer(
        ConfluentRegistryAvroDeserializationSchema.forSpecific(
            SensorData.class, 
            "http://schema-registry:8081"
        )
    )
    .build();

// Create SensorData objects
SensorData sensor = SensorData.newBuilder()
    .setSensorId("sensor-001")
    .setEquipmentId("boiler-a")
    .setTimestamp(System.currentTimeMillis() / 1000)
    .setTemperature(75.5)
    .setUnit("Celsius")
    .setLocation("Plant-A")
    .setStatus(SensorStatus.ONLINE)
    .build();
```

### Kafka Producer Usage

```java
Properties props = new Properties();
props.put("bootstrap.servers", "kafka:9092");
props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
props.put("value.serializer", "io.confluent.kafka.serializers.KafkaAvroSerializer");
props.put("schema.registry.url", "http://schema-registry:8081");

Producer<String, SensorData> producer = new KafkaProducer<>(props);
producer.send(new ProducerRecord<>("sensor-raw-data", sensor));
```

## 🔒 Compatibility Matrix

| Schema Version | Compatibility | Changes |
|---------------|---------------|---------|
| v1.0.0 | Initial release | Base schemas |
| v1.1.0 | Backward compatible | Added `anomalyScore` field (optional) |

## 🐛 Troubleshooting

### Common Issues

1. **Schema Registry Connection Errors**
    - Verify Schema Registry is running
    - Check network connectivity
    - Validate schema registry URL in configuration

2. **Schema Compatibility Errors**
    - Ensure all consumers use compatible schemas
    - Check schema evolution rules
    - Use schema registry UI to inspect conflicts

3. **Class Generation Failures**
    - Verify Avro schema syntax
    - Check Maven Avro plugin configuration
    - Clean and rebuild: `mvn clean generate-sources`

### Debugging

Enable debug logging for schema registry interactions:

```properties
logging.level.io.confluent.kafka.serializers=DEBUG
```

## 🏷️ Versioning

We use [Semantic Versioning](http://semver.org/). For available versions, see the tags on this repository.

## 📞 Support

- Schema Issues: Create GitHub issue
- Urgent Production Issues: Contact platform team
- Schema Registry: http://schema-registry:8081

---

**Maintained by:** Nestor Martourez  
**Last Updated:** October 2024