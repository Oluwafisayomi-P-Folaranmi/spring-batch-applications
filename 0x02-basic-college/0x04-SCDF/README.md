# SCDF

## **📌 1. Introduction to Spring Cloud Data Flow (SCDF)**
🔹 **What is SCDF?**  
SCDF is a **lightweight, cloud-native framework** that orchestrates both **batch** and **streaming** workloads using Spring technologies.  

🔹 **Key Features for Spring Batch**  
- **Job Orchestration** – Define and schedule batch jobs  
- **Monitoring & Logging** – Track job executions & view logs  
- **Scaling & Distributed Execution** – Run jobs across multiple instances  
- **REST API & UI** – Control batch jobs via API and dashboard  

🔹 **SCDF Components**  
- **Spring Cloud Data Flow Server** – The core SCDF server managing jobs  
- **Spring Cloud Skipper** – Handles deployment updates  
- **Task Launcher** – Executes Spring Batch jobs as tasks  
- **Data Flow Dashboard** – Web UI for monitoring and managing jobs  

## **📌 2. Setting Up Spring Cloud Data Flow Locally**
🔹 **Prerequisites**  
- Docker & Docker Compose  
- JDK 17+  
- Maven  

🔹 **Start SCDF using Docker Compose**
```bash
curl -O https://raw.githubusercontent.com/spring-cloud/spring-cloud-dataflow/main/src/docker-compose/docker-compose.yml
docker-compose up
```
This starts **SCDF, Skipper, MySQL, and RabbitMQ/Kafka**.

🔹 **Verify SCDF is Running**  
- Open **http://localhost:9393/dashboard** to access the UI  
- Test API:  
```bash
curl http://localhost:9393/about
```

## **📌 3. Creating a Spring Batch Task for SCDF**
SCDF executes **batch jobs as tasks**, so first, we create a **Spring Batch Task**.  

🔹 **Add Dependencies**
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-task</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.batch</groupId>
    <artifactId>spring-batch-core</artifactId>
</dependency>
```

🔹 **Define a Spring Batch Job**
```java
@EnableTask
@SpringBootApplication
public class BatchJobApplication {
    public static void main(String[] args) {
        SpringApplication.run(BatchJobApplication.class, args);
    }

    @Bean
    public Job sampleJob(JobBuilderFactory jobBuilderFactory, StepBuilderFactory stepBuilderFactory) {
        return jobBuilderFactory.get("sampleJob")
            .start(stepBuilderFactory.get("step1")
                .tasklet((stepContribution, chunkContext) -> {
                    System.out.println("Spring Batch Job Running...");
                    return RepeatStatus.FINISHED;
                })
                .build())
            .build();
    }
}
```

🔹 **Package as a JAR**
```bash
mvn clean package
```

## **📌 4. Deploying Spring Batch Job to SCDF**
### **1️⃣ Register the Batch Job as a Task**
SCDF executes batch jobs as **tasks**, so we first register our batch job.  

```bash
dataflow:>app register --name batch-task --type task --uri file:///path/to/your-batch-job.jar
```
For Docker-based deployment:
```bash
dataflow:>app register --name batch-task --type task --uri docker://your-docker-image
```

### **2️⃣ Create a Task Definition**
```bash
dataflow:>task create my-batch-task --definition "batch-task"
```

### **3️⃣ Launch the Batch Task**
```bash
dataflow:>task launch my-batch-task
```

SCDF will execute the batch job and log its progress.

### **4️⃣ Monitor the Job Execution**
- **UI:** Go to the **Tasks** tab in SCDF Dashboard  
- **API:** Fetch job execution status  
```bash
curl http://localhost:9393/tasks/executions
```

## **📌 5. Scheduling & Orchestrating Batch Jobs**
### **🔹 Scheduling Jobs in SCDF**
SCDF allows job scheduling using **Cron Expressions**.  

```bash
dataflow:>schedule create my-job-schedule --definition "my-batch-task" --expression "0 0 * * * *"
```
This schedules the batch job to run **every hour**.

### **🔹 Managing Job Restart & Retries**
- Set **restartability** for failed jobs  
- Configure **retry logic** for steps in `application.properties`
```properties
spring.batch.job.enabled=false
spring.cloud.task.closecontext.enabled=false
```

## **📌 6. Scaling Spring Batch Jobs in SCDF**
### **🔹 Parallel Execution of Jobs**
SCDF can **scale jobs horizontally** by running multiple instances.  
Modify the launch command:
```bash
dataflow:>task launch my-batch-task --properties "spring.cloud.task.execution.platform.kubernetes"
```
This runs the job on Kubernetes.

### **🔹 Partitioned Step Execution**
Use **Partitioner** to distribute work across multiple nodes.
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

## **📌 7. Monitoring & Logging Batch Jobs**
SCDF provides **real-time monitoring** of batch jobs.

### **🔹 Enable Logging**
```properties
logging.level.org.springframework.cloud.task=DEBUG
```
Check logs via:
```bash
dataflow:>task execution log --id 1
```

### **🔹 View Job Metrics in Grafana/Prometheus**
Enable **Micrometer Metrics** for monitoring.
```properties
management.metrics.export.prometheus.enabled=true
```
SCDF integrates with **Prometheus + Grafana** to visualize job performance.

## **📌 8. Advanced Features**
### **🔹 External Event Triggers (Kafka, RabbitMQ)**
SCDF can **trigger batch jobs based on events**.

Example: Trigger job when a Kafka message arrives.
```yaml
spring:
  cloud:
    stream:
      bindings:
        input:
          destination: batch-job-events
          group: batch
```

### **🔹 Integration with Cloud Platforms**
- **Kubernetes** – Deploy SCDF to **K8s** using Helm Charts  
- **AWS, Azure, GCP** – Run SCDF batch jobs on cloud-native environments  
