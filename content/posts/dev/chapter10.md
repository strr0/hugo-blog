---
title: "Spring Batch 批处理框架"
date: 2024-03-14T15:00:00+08:00
categories: ["development"]
tags: ["java"]
draft: false
---

## 1 介绍

Spring Batch 是一个轻量级的、完善的批处理框架,旨在帮助企业建立健壮、高效的批处理应用。Spring Batch 的核心为一个作业启动器和一个作业主体（Job），一个作业主体会包含若干个步骤（Step），一个步骤可以是 Chunk 类型或 Tasklet 类型，作业启动器用于执行定义的作业。

### 1.1 Job 配置

可以定义 JobListener 在 Job 执行期间监听 Job 的执行情况
```
public interface JobExecutionListener {
    void beforeJob(JobExecution jobExecution);
    void afterJob(JobExecution jobExecution);
}
```
Job 配置示例如下
```
public Job footballJob(JobRepository jobRepository) {
    return new JobBuilder("footballJob", jobRepository)
                     .listener(sampleListener())
                     ...
                     .build();
}
```

### 1.2 Step 配置

可以定义 StepListener 在 Step 执行期间监听 Step 的执行情况
```
public interface StepExecutionListener extends StepListener {
    default void beforeStep(StepExecution stepExecution) {
    }
    default ExitStatus afterStep(StepExecution stepExecution) {
        return null;
    }
}
```
Step 配置示例如下
```
// chunk 类型
public Step sampleStep(JobRepository jobRepository, 
		PlatformTransactionManager transactionManager) { 
	return new StepBuilder("sampleStep", jobRepository)
				.<String, String>chunk(10, transactionManager) 
				.reader(itemReader())
				.writer(itemWriter())
				.build();
}

// tasklet 类型
public Step step(JobRepository jobRepository, PlatformTransactionManager transactionManager) {
    return new StepBuilder("step", jobRepository)
    			.tasklet(myTasklet(), transactionManager)
    			.build();
}
```

### 1.3 并行 Step 配置

```
public Job job(JobRepository jobRepository) {
    return new JobBuilder("job", jobRepository)
        .start(splitFlow())
        .next(step4())
        .build()        //builds FlowJobBuilder instance
        .build();       //builds Job instance
}

// flow1 flow2 并行
public Flow splitFlow() {
    return new FlowBuilder<SimpleFlow>("splitFlow")
        .split(taskExecutor())
        .add(flow1(), flow2())
        .build();
}

public Flow flow1() {
    return new FlowBuilder<SimpleFlow>("flow1")
        .start(step1())
        .next(step2())
        .build();
}

public Flow flow2() {
    return new FlowBuilder<SimpleFlow>("flow2")
        .start(step3())
        .build();
}

public TaskExecutor taskExecutor() {
    return new SimpleAsyncTaskExecutor("spring_batch");
}
```

### 1.4 作业执行

```
@SpringBootTest
public class JobLauncherTest {

    @Autowired
    JobLauncher jobLauncher;

    @Autowired
    Job job;

    @Test
    public void test() throws Exception {
        jobLauncher.run(job, new JobParameters());
    }
}
```

## 2 数据清洗作业

### 2.1 项目配置

引入依赖
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-batch</artifactId>
    <version>3.1.5</version>
    <exclusions>
        <exclusion>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
        </exclusion>
    </exclusions>
</dependency>

<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>dynamic-datasource-spring-boot3-starter</artifactId>
    <version>4.2.0</version>
</dependency>

<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <version>8.0.31</version>
</dependency>

<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>42.6.0</version>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <version>3.1.5</version>
    <scope>test</scope>
</dependency>
```
application 配置
```
spring:
  datasource:
    type: com.zaxxer.hikari.HikariDataSource
    dynamic:
      primary: postgres
      strict: true
      datasource:
        mysql:
          type: ${spring.datasource.type}
          driver-class-name: com.mysql.cj.jdbc.Driver
          url: jdbc:mysql://localhost:3306/testdb
          username: root
          password: password
        postgres:
          type: ${spring.datasource.type}
          driver-class-name: org.postgresql.Driver
          url: jdbc:postgresql://localhost:5432/postgres
          username: postgres
          password: password
```

### 2.2 定义分区器

用于将数据分成若干分区并行处理
```
public class JdbcRownumPartitioner implements DataPartitioner {
    private Long total = 0L;  // 数据总数
    private Long pageSize = 5000L;  // 分页大小

    public void setTotal(Long total) {
        this.total = total;
    }

    public void setPageSize(Long pageSize) {
        this.pageSize = pageSize;
    }

    @Override
    public Map<String, ExecutionContext> partition(int gridSize) {
        Map<String, ExecutionContext> contextMap = new HashMap<>();
        int idx = 0;
        for (int row = 0; row < total; row += pageSize) {
            ExecutionContext context = new ExecutionContext();
            context.put("startRow", row);
            context.put("pageSize", pageSize);
            contextMap.put(String.valueOf(idx++), context);
        }
        return contextMap;
    }
}
```

### 2.3 定义读取器

从指定的数据源按一定规则读取数据
```
public class JdbcPartitionReader extends AbstractItemCountingItemStreamItemReader<Map<String, Object>> {
    ...
    @Override
    protected Map<String, Object> doRead() throws Exception {
        synchronized (this.lock) {
            if (results == null || results.isEmpty()) {
                return null;
            }
            return results.remove(results.size() - 1);
        }
    }

    @Override
    protected void doOpen() throws Exception {
        fetchData();
    }

    private void fetchData() {
        if (dataSource == null || script == null || executionContext == null) {
            return;
        }
        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
        int startRow = executionContext.getInt("startRow");
        long pageSize = executionContext.getLong("pageSize");
        results = jdbcTemplate.queryForList(String.format("%s limit %d offset %d", script, pageSize, startRow));
    }
    ...
}
```

> 分区并行处理有可能会遇到线程安全问题，可通过使用 ThreadLocal 等方式来避免读取脏数据。

### 2.4 定义数据处理器

字段名称下划线转驼峰
```
public class CamelCaseProcessor implements DataItemProcessor<Map<String, Object>, Map<String, Object>> {
    @Override
    public Map<String, Object> process(Map<String, Object> item) throws Exception {
        if (item == null || item.isEmpty()) {
            return null;
        }
        Map<String, Object> out = new HashMap<>();
        for (Map.Entry<String, Object> entry : item.entrySet()) {
            out.put(StringUtils.toCamelCase(entry.getKey()), entry.getValue());
        }
        return out;
    }
}
```

### 2.5 定义数据输出器

按一定规则将数据输出到指定数据源
```
public class JdbcBatchWriter implements DataItemWriter<Map<String, Object>> {
    ...
    @Override
    public void write(Chunk<? extends Map<String, Object>> chunk) throws Exception {
        if (dataSource == null || script == null) {
            return;
        }
        List<? extends Map<String, Object>> items = chunk.getItems();
        NamedParameterJdbcTemplate jdbcTemplate = new NamedParameterJdbcTemplate(dataSource);
        for (Map<String, Object> item : items) {
            jdbcTemplate.update(script, item);
        }
    }
}
```

### 2.6 作业主体配置

```
@Configuration
public class JobPartitionConfig {
    @Autowired
    private JobRepository jobRepository;
    @Autowired
    private PlatformTransactionManager transactionManager;
    @Autowired
    private DynamicRoutingDataSource dataSource;

    @Bean
    public Job job() {
        return new JobBuilder("testJob", jobRepository)
                .start(createPartStep())
                .build();
    }

    public Step createPartStep() {
        // 读取器
        JdbcPartitionReader reader = new JdbcPartitionReader();
        reader.setDataSource(dataSource.getDataSource("postgres"));
        reader.setScript("select test_key, test_value from test_demo");
        // 处理器
        CamelCaseProcessor processor = new CamelCaseProcessor();
        // 写出器
        JdbcBatchWriter writer = new JdbcBatchWriter();
        writer.setDataSource(dataSource.getDataSource("mysql"));
        writer.setScript("insert into test_demo(test_key, test_value) values(:testKey, :testValue)");
        // 分区器
        JdbcRownumPartitioner partitioner = new JdbcRownumPartitioner();
        partitioner.setTotal(reader.getCount());
        partitioner.setPageSize(10L);
        return new StepBuilder("testPartStep", jobRepository)
                .allowStartIfComplete(true)
                .partitioner("testPart", partitioner)
                .step(createStep(reader, processor, writer))
                .build();
    }

    public Step createStep(ItemReader<Map<String, Object>> reader, ItemProcessor<Map<String, Object>, Map<String, Object>> processor, ItemWriter<Map<String, Object>> writer) {
        return new StepBuilder("testStep", jobRepository)
                .allowStartIfComplete(true)
                .<Map<String, Object>, Map<String, Object>>chunk(10, transactionManager)
                .reader(reader)
                .processor(processor)
                .writer(writer)
                .build();
    }
}
```

> DynamicRoutingDataSource 是 baomidou 的动态多数据源框架，用于指定作业读取及输出的数据源

### 2.7 作业执行

```
@SpringBootTest
public class ApplicationTests {
    @Autowired
    private JobLauncher jobLauncher;
    @Autowired
    private Job job;

    @Test
    void loadContext() throws Exception {
        jobLauncher.run(job, new JobParameters());
    }
}
```