# Batch Core & Data

## Understanding Batch Processing
### **1️⃣ Understanding Batch Processing**
- What is **batch processing**, and why is it needed?
- Difference between **batch processing** and **real-time processing**
- Common use cases of Spring Batch in the industry:
  - Data migration (CSV, XML, JSON → Database)
  - ETL (Extract, Transform, Load) pipelines
  - Large-scale file processing
  - Scheduled jobs for reports, invoices, etc.

## Spring Batch Core Architecture
### **2️⃣ Spring Batch Core Architecture**
- Understanding **Spring Batch Architecture**:
  - **Job**: Represents the entire batch process
  - **Step**: A unit of work in a job
  - **ItemReader**: Reads data from a source
  - **ItemProcessor**: Processes the data
  - **ItemWriter**: Writes data to a destination
- **Chunk-Oriented Processing** vs **Tasklet Processing**
- **Spring Batch Metadata Tables** (JobRepository, JobExecution, etc.)

## Spring Batch Job
### **3️⃣ Setting Up a Simple Spring Batch Job**
🔹 **Project Setup:**
- Add **Spring Batch dependencies** in `pom.xml`:
  ```xml
  <dependency>
      <groupId>org.springframework.batch</groupId>
      <artifactId>spring-batch-core</artifactId>
  </dependency>
  ```
- Create a **basic batch configuration** using `@EnableBatchProcessing`
- Define a simple **Tasklet** to print "Hello, Spring Batch!"

🔹 **Run the Job Using JobLauncher**
- Define `JobLauncher` to manually trigger jobs
- Execute batch jobs via **Spring Boot Application Runner**
- Use **CommandLineJobRunner** for command-line execution
