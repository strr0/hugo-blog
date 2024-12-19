---
title: "ShardingSphere 分库分表"
date: 2024-07-20T10:00:00+08:00
categories: ["development"]
tags: ["java"]
draft: false
---

## 1 介绍

Apache ShardingSphere 是一款分布式 SQL 事务和查询引擎，可通过数据分片、弹性伸缩、加密等能力对任意数据库进行增强。

## 2 使用

### 2.1 基础配置和使用

创建 maven 项目，并引入依赖
```
<dependency>
    <groupId>org.apache.shardingsphere</groupId>
    <artifactId>shardingsphere-jdbc-core</artifactId>
    <version>5.4.1</version>
</dependency>
```
构建数据源
```java
String databaseName = "db"; // 指定逻辑 Database 名称
ModeConfiguration modeConfig = ... // 构建运行模式
Map<String, DataSource> dataSourceMap = ... // 构建真实数据源
Collection<RuleConfiguration> ruleConfigs = ... // 构建具体规则
Properties props = ... // 构建属性配置
DataSource dataSource = ShardingSphereDataSourceFactory.createDataSource(databaseName, modeConfig, dataSourceMap, ruleConfigs, props);
```
使用数据源
```java
try {
    DataSource dataSource = MyShardingSphereFactory.createDataSource();
    try (Connection conn = dataSource.getConnection(); Statement stmt = conn.createStatement()) {
        try (ResultSet rs = stmt.executeQuery("select id, test_key, value from my_demo where id = 11")) {
            while (rs.next()) {
                System.out.printf("id: %d, test_key: %s, value: %s\n", rs.getInt(1), rs.getString(2), rs.getString(3));
            }
        }
    }
} catch (SQLException e) {
    e.printStackTrace();
}
```

### 2.2 模式配置

单机模式
```java
private ModeConfiguration createModeConfiguration() {
    return new ModeConfiguration("Standalone", new StandalonePersistRepositoryConfiguration("JDBC", new Properties()));
}
```
集群模式
```java
private ModeConfiguration createModeConfiguration() {
    return new ModeConfiguration("Cluster", new ClusterPersistRepositoryConfiguration("ZooKeeper", "governance-sharding-db", "localhost:2181", new Properties()));
}
```

### 2.3 数据源配置

```java
private Map<String, DataSource> createDataSources() {
    Map<String, DataSource> dataSourceMap = new HashMap<>();
    // 配置第 1 个数据源
    HikariDataSource dataSource1 = new HikariDataSource();
    dataSource1.setDriverClassName("com.mysql.jdbc.Driver");
    dataSource1.setJdbcUrl("jdbc:mysql://localhost:3306/ds1");
    dataSource1.setUsername("root");
    dataSource1.setPassword("");
    dataSourceMap.put("ds1", dataSource1);
    
    // 配置第 2 个数据源
    HikariDataSource dataSource2 = new HikariDataSource();
    dataSource2.setDriverClassName("com.mysql.jdbc.Driver");
    dataSource2.setJdbcUrl("jdbc:mysql://localhost:3306/ds2");
    dataSource2.setUsername("root");
    dataSource2.setPassword("");
    dataSourceMap.put("ds2", dataSource2);
}
```

### 2.4 规则配置

#### 2.4.1 表规则配置

配置表的逻辑名和实际节点等
```java
private ShardingTableRuleConfiguration getOrderTableRuleConfiguration() {
    ShardingTableRuleConfiguration config = new ShardingTableRuleConfiguration("t_order", "demo_ds_${0..1}.t_order_${[0, 1]}");
    config.setKeyGenerateStrategy(new KeyGenerateStrategyConfiguration("order_id", "snowflake"));
    config.setAuditStrategy(new ShardingAuditStrategyConfiguration(Collections.singleton("sharding_key_required_auditor"), true));
    return config;
}
```
这里的 `${0..1}` 和 `${[0, 1]}` 表示实际节点名称的变化范围，如果表名没有规律也可以通过枚举的方式  
```java
...
ShardingTableRuleConfiguration config = new ShardingTableRuleConfiguration("my_demo", "ds.test_demo,ds.test_demo_copy1,ds.test_demo_copy2,ds.test_demo_copy3,ds.test_demo_copy4,ds.test_demo_copy5");
...
```

#### 2.4.2 分片策略配置

##### 2.4.2.1 配置策略

指定库名或表名分片策略（user_id 为分片字段，inline 为分片算法名称）
```java
...
config.setDefaultDatabaseShardingStrategy(new StandardShardingStrategyConfiguration("user_id", "inline"));
...
```

##### 2.4.2.2 配置算法

分片算法逻辑（定义 inline 算法，algorithm-expression 为表达式类型）
```java
...
Properties props = new Properties();
props.setProperty("algorithm-expression", "demo_ds_${user_id % 2}");
config.getShardingAlgorithms().put("inline", new AlgorithmConfiguration("INLINE", props));
...
```

##### 2.4.2.3 配置自定义算法

自定义算法
```java
/**
 * 自定义分片算法
 */
public class MyTableShardingAlgorithm implements StandardShardingAlgorithm<Integer> {
    @Override
    public String doSharding(Collection<String> collection, PreciseShardingValue<Integer> preciseShardingValue) {
        Integer value = preciseShardingValue.getValue();
        if (value > 50) {
            return "test_demo_copy5";
        }
        if (value > 40) {
            return "test_demo_copy4";
        }
        if (value > 30) {
            return "test_demo_copy3";
        }
        if (value > 20) {
            return "test_demo_copy2";
        }
        if (value > 10) {
            return "test_demo_copy1";
        }
        return "test_demo";
    }

    @Override
    public Collection<String> doSharding(Collection<String> collection, RangeShardingValue<Integer> rangeShardingValue) {
        return null;
    }
}
```
添加算法到配置中
```java
...
Properties props = new Properties();
props.setProperty("algorithmClassName", "com.strr.config.MyTableShardingAlgorithm");
props.setProperty("strategy", "standard");
config.getShardingAlgorithms().put("my-table-sharding-algorithm", new AlgorithmConfiguration("CLASS_BASED", props));
...
```