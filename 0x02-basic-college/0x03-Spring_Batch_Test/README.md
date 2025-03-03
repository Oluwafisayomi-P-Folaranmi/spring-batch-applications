# Spring Batch

## **📌 1. Introduction to Spring Batch Test**  
🔹 **Why Testing is Important in Spring Batch?**  
- Ensure correctness of batch processing logic  
- Validate data transformations and flows  
- Detect failures in multi-step jobs  
- Maintainability and reusability of batch components  

🔹 **Spring Batch Test Overview**  
- `spring-batch-test` module provides utilities for testing  
- Includes reusable test components for **Job, Step, Reader, Processor, and Writer**  

🔹 **Adding Spring Batch Test Dependency**  
```xml
<dependency>
    <groupId>org.springframework.batch</groupId>
    <artifactId>spring-batch-test</artifactId>
    <scope>test</scope>
</dependency>
```

## **📌 2. Setting Up Spring Batch Test Environment**  
🔹 **Use JUnit & Spring Test Context**  
- Use **JUnit 5** or **JUnit 4** with `@SpringBootTest`  
- Use **Spring Boot’s TestConfiguration** for batch components  

🔹 **Mocking Dependencies for Isolated Testing**  
- Use **Mockito** for mocking services, repositories, and batch components  
- Use **H2 Database** for in-memory batch testing  

## **📌 3. Testing Individual Batch Components**  
🔹 **Testing ItemReader**
- Verify that an **ItemReader** correctly reads and transforms data  
- Example: Test `FlatFileItemReader` for CSV files  
```java
@Test
void testCsvItemReader() throws Exception {
    FlatFileItemReader<Person> reader = new FlatFileItemReader<>();
    reader.setResource(new ClassPathResource("test-data.csv"));
    reader.open(new ExecutionContext());
    
    Person person = reader.read();
    assertNotNull(person);
    assertEquals("John", person.getFirstName());
}
```

🔹 **Testing ItemProcessor**
- Verify that the processor applies **business logic correctly**  
- Example: Ensure uppercase transformation in a processor  
```java
@Test
void testItemProcessor() {
    NameProcessor processor = new NameProcessor();
    Person processed = processor.process(new Person("john", "doe"));
    
    assertNotNull(processed);
    assertEquals("JOHN DOE", processed.getFullName());
}
```

🔹 **Testing ItemWriter**
- Verify that an **ItemWriter** correctly writes data  
- Example: Writing data to a **database or file**  
```java
@Test
void testItemWriter() throws Exception {
    List<Person> items = List.of(new Person("John", "Doe"));
    JdbcBatchItemWriter<Person> writer = new JdbcBatchItemWriter<>();
    writer.setDataSource(dataSource);
    writer.setSql("INSERT INTO people (first_name, last_name) VALUES (:firstName, :lastName)");
    
    writer.write(items);
    
    int count = jdbcTemplate.queryForObject("SELECT COUNT(*) FROM people", Integer.class);
    assertEquals(1, count);
}
```

## **📌 4. Testing a Complete Batch Job Execution**  
🔹 **Testing Steps Using StepScopeTestExecutionListener**
- Use `JobLauncherTestUtils` to launch individual steps  
```java
@Autowired
private JobLauncherTestUtils jobLauncherTestUtils;

@Test
void testStepExecution() throws Exception {
    JobExecution jobExecution = jobLauncherTestUtils.launchStep("processStep");
    assertEquals(ExitStatus.COMPLETED, jobExecution.getExitStatus());
}
```

🔹 **Testing Full Job Execution**  
- Run a full job and assert expected job outcomes  
```java
@Test
void testFullJobExecution() throws Exception {
    JobExecution jobExecution = jobLauncherTestUtils.launchJob();
    assertEquals(ExitStatus.COMPLETED, jobExecution.getExitStatus());
}
```

## **📌 5. Advanced Spring Batch Testing Strategies**  
🔹 **Testing Skipped Records & Retry Mechanisms**  
- Use **JobRepositoryTestUtils** to verify failed records  
```java
@Test
void testJobWithSkippedRecords() {
    JobExecution jobExecution = jobLauncherTestUtils.launchJob();
    
    int skippedCount = jobExecution.getStepExecutions().stream()
        .mapToInt(StepExecution::getSkipCount)
        .sum();
    
    assertEquals(2, skippedCount);  // Assuming 2 records should be skipped
}
```

🔹 **Mocking External Systems in Batch Jobs**  
- Use **Mockito** to mock external APIs or database interactions  
```java
@Mock
private ExternalApiService externalApiService;

@Test
void testProcessorWithMockedExternalService() throws Exception {
    when(externalApiService.fetchData()).thenReturn("Mocked Response");

    MyProcessor processor = new MyProcessor(externalApiService);
    String result = processor.process("input");

    assertEquals("Mocked Response Processed", result);
}
```

🔹 **Using Embedded Databases for Testing**  
- Use **H2 Database** for an in-memory batch test setup  
```java
@Configuration
public class TestBatchConfig {
    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.H2)
            .addScript("schema.sql")
            .build();
    }
}
```

## **📌 6. Monitoring & Debugging Batch Jobs in Tests**  
🔹 **Enable Batch Job Logs for Debugging**  
- Enable logging in `application-test.properties`
```properties
logging.level.org.springframework.batch=DEBUG
logging.level.org.springframework.jdbc.core.JdbcTemplate=DEBUG
```

🔹 **Using JobExplorer to Inspect Job History**  
```java
@Autowired
private JobExplorer jobExplorer;

@Test
void testJobHistory() {
    List<JobExecution> executions = jobExplorer.findJobInstancesByName("myJob", 0, 5)
        .stream()
        .map(jobExplorer::getJobExecutions)
        .flatMap(List::stream)
        .collect(Collectors.toList());

    assertFalse(executions.isEmpty());
}
```

## **📌 7. Performance Testing & Optimization in Batch Jobs**  
🔹 **Test with Large Data Sets**  
- Use **JUnit Parameterized Tests** for testing large files  
- Simulate batch processing with **millions of records**  

🔹 **Measure Job Performance**
- Record execution time and memory usage in tests  
```java
long startTime = System.currentTimeMillis();
jobLauncherTestUtils.launchJob();
long endTime = System.currentTimeMillis();
System.out.println("Job Execution Time: " + (endTime - startTime) + "ms");
```
