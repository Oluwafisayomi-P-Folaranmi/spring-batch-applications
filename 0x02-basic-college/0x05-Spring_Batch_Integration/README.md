# Spring Batch Integration

## **📌 1. Introduction to Spring Batch Integration**
🔹 **What is Spring Batch Integration?**  
Spring Batch Integration extends **Spring Batch** by adding **Spring Integration patterns** like:  
- **Messaging with Kafka, RabbitMQ, and JMS**  
- **Real-time event-driven batch execution**  
- **Asynchronous batch processing**  
- **Parallel processing using message queues**  

🔹 **Common Use Cases**  
✔️ **Trigger Batch Jobs from Kafka or RabbitMQ**  
✔️ **Process files as they arrive (e.g., CSV, XML, JSON)**  
✔️ **Use REST APIs to start batch jobs**  
✔️ **Implement event-driven architecture with batch jobs**  
✔️ **Scale jobs using partitioning and message queues**  

## **📌 2. Setting Up Spring Batch Integration**
🔹 **Add Dependencies in `pom.xml`**  
```xml
<dependency>
    <groupId>org.springframework.batch</groupId>
    <artifactId>spring-batch-integration</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.integration</groupId>
    <artifactId>spring-integration-core</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.integration</groupId>
    <artifactId>spring-integration-kafka</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.integration</groupId>
    <artifactId>spring-integration-rabbit</artifactId>
</dependency>
```

🔹 **Enable Spring Batch Integration**
```java
@EnableBatchProcessing
@EnableIntegration
@SpringBootApplication
public class BatchIntegrationApplication {
    public static void main(String[] args) {
        SpringApplication.run(BatchIntegrationApplication.class, args);
    }
}
```

## **📌 3. Event-Driven Batch Job Execution**
### **1️⃣ Trigger a Batch Job from Kafka**
You can start a batch job when a Kafka event arrives.  

🔹 **Kafka Listener to Trigger Job**
```java
@Configuration
public class KafkaBatchTrigger {

    @Bean
    public IntegrationFlow kafkaListenerFlow(JobLauncher jobLauncher, Job job) {
        return IntegrationFlows.from(Kafka.inboundChannelAdapter(consumerFactory(), "batch-job-topic"))
            .handle(String.class, (payload, headers) -> {
                JobParameters jobParameters = new JobParametersBuilder()
                        .addString("event", payload)
                        .toJobParameters();
                jobLauncher.run(job, jobParameters);
                return null;
            })
            .get();
    }

    private ConsumerFactory<String, String> consumerFactory() {
        return new DefaultKafkaConsumerFactory<>(Map.of(
            ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092",
            ConsumerConfig.GROUP_ID_CONFIG, "batch-group"
        ));
    }
}
```
🔹 **Send a Kafka Event to Start the Batch Job**
```bash
echo '{"message": "start batch job"}' | kafkacat -b localhost:9092 -t batch-job-topic
```
This will **start the batch job dynamically** when a Kafka message is received.

### **2️⃣ Trigger a Batch Job from RabbitMQ**
🔹 **RabbitMQ Listener to Trigger Job**
```java
@Configuration
public class RabbitBatchTrigger {

    @Bean
    public IntegrationFlow rabbitListenerFlow(JobLauncher jobLauncher, Job job) {
        return IntegrationFlows.from(Amqp.inboundAdapter(rabbitTemplate(), "batch.queue"))
            .handle(String.class, (payload, headers) -> {
                JobParameters jobParameters = new JobParametersBuilder()
                        .addString("message", payload)
                        .toJobParameters();
                jobLauncher.run(job, jobParameters);
                return null;
            })
            .get();
    }

    private RabbitTemplate rabbitTemplate() {
        return new RabbitTemplate(new CachingConnectionFactory("localhost"));
    }
}
```
🔹 **Send a RabbitMQ Event to Start the Batch Job**
```bash
rabbitmqadmin publish exchange=amq.direct routing_key=batch.queue payload="start"
```
This will **trigger the batch job** whenever a RabbitMQ message is received.

### **3️⃣ Trigger a Batch Job from a REST API**
🔹 **Expose a REST Endpoint to Start a Batch Job**
```java
@RestController
public class BatchJobController {

    @Autowired
    private JobLauncher jobLauncher;

    @Autowired
    private Job job;

    @PostMapping("/start-batch-job")
    public ResponseEntity<String> startBatchJob(@RequestBody Map<String, String> params) throws Exception {
        JobParameters jobParameters = new JobParametersBuilder()
                .addString("input", params.get("input"))
                .toJobParameters();
        jobLauncher.run(job, jobParameters);
        return ResponseEntity.ok("Batch Job Started!");
    }
}
```
🔹 **Trigger the Job Using Postman or Curl**
```bash
curl -X POST http://localhost:8080/start-batch-job -H "Content-Type: application/json" -d '{"input": "data"}'
```
This API will **start the batch job dynamically** when called.

## **📌 4. Real-Time File Processing with Spring Batch Integration**
Instead of scheduling a file processing job, Spring Batch can **detect new files** and process them automatically.

🔹 **Watch a Directory for New Files**
```java
@Bean
public IntegrationFlow fileWatcherFlow(JobLauncher jobLauncher, Job job) {
    return IntegrationFlows.from(Files.inboundAdapter(new File("/data/input"))
            .patternFilter("*.csv"))
        .handle(File.class, (file, headers) -> {
            JobParameters jobParameters = new JobParametersBuilder()
                    .addString("file", file.getAbsolutePath())
                    .toJobParameters();
            jobLauncher.run(job, jobParameters);
            return null;
        })
        .get();
}
```
Whenever a **new CSV file** is added to `/data/input`, the batch job will process it **in real time**.

## **📌 5. Parallel Batch Processing Using Messaging Queues**
Spring Batch Integration allows **parallel processing** using message queues like **Kafka, RabbitMQ, or JMS**.

### **🔹 Use Kafka for Parallel Step Execution**
- **Partition the dataset** and send partitions to multiple workers.
- Each worker processes a subset of the data **in parallel**.

🔹 **Partitioned Job Step**
```java
@Bean
public Step partitionedStep(StepBuilderFactory stepBuilderFactory, PartitionHandler partitionHandler) {
    return stepBuilderFactory.get("partitionedStep")
        .partitioner("workerStep", new ColumnRangePartitioner())
        .step(workerStep())
        .gridSize(4)
        .partitionHandler(partitionHandler)
        .build();
}
```

🔹 **Send Partitions to Kafka Workers**
```java
@Bean
public IntegrationFlow partitionedFlow() {
    return IntegrationFlows.from("partitionedRequests")
        .handle(Kafka.outboundAdapter(producerFactory())
            .topic("batch-partitions"))
        .get();
}
```
This enables **scaling batch jobs dynamically**.
