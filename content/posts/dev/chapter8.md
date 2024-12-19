---
title: "Mybatis 浅显解读及基础 CRUD 封装"
date: 2023-12-15T20:00:00+08:00
categories: ["development"]
tags: ["java"]
draft: false
---

## 介绍

在开始之前，我简单阅读了 Mybatis-plus 的代码，不得不说 Mybatis-plus 是一个非常完善、功能齐全的框架，但是它对原有的 Mybatis 的改动也不小，重写了诸如 Configuration、SqlSessionFactoryBuilder、MapperRegistry 等等原有代码，而我的想法是在尽可能保持 Mybatis 原汁原味的基础上，用最少的代码对原框架进行封装。

## 1 前置工作

### 1.1 Mybatis

#### 1.1.1 简单理解

Mybatis 是一个基于 Java 的持久层框架，它的内部封装了 Jdbc，简化了 Jdbc 建立连接、执行、断开连接等繁琐的操作。Mybatis 的工作以 SqlSessionFactory 实例为中心展开，SqlSessionFactory 实例是由 SqlSessionFactoryBuilder 从 Xml 配置文件或 Java 配置（SpringBoot 项目中通过 application 简化 Java 配置）中构建。在 SqlSessionFactory 实例准备完毕后，从 SqlSessionFactory 实例中获取 SqlSession 对象，SqlSession 对象可以直接执行指定的 MappedStatement 的 sql 语句，或者从 SqlSession 对象中获取 mapper 对象来执行。

#### 1.1.2 示例

直接执行
```
SqlSession sqlSession = sqlSessionFactory.openSession();
List<Xxx> xxxs = sqlSession.selectList("xxx.xxx");  // xxx.xxx 为 MappedStatement 的 id
```
通过 mapper 执行
```
SqlSession sqlSession = sqlSessionFactory.openSession();
XxxMapper xxxMapper = sqlSession.getMapper(XxxMapper.class);
List<Xxx> xxxs = xxxMapper.selectList();
```

### 1.2 mybatis-spring-boot-starter

mybatis-spring-boot-starter 的核心是 MybatisAutoConfiguration 类，该类定义了默认情况下自动生成的 Mybatis 配置。

#### 1.2.1 sqlSessionFactory 方法

该方法从 SpringBoot 中读取定义的 Mybatis 配置并生成 SqlSessionFactory 对象。

#### 1.2.2 MapperScannerRegistrarNotFoundConfiguration 类

该类为默认 Mybatis 扫描器配置，扫描规则由导入的 AutoConfiguredMapperScannerRegistrar 定义。扫描器将扫描带有 @Mapper 注解的类，以及指定路径下的 Java 类，并为它们生成 Spring Bean。

> 最初的想法是通过自定义扫描器，对扫描到的类生成 Bean 及添加自定义的方法到 Mybatis 配置中，但是该方法生成的 Bean 会与 Mybatis 的默认配置冲突，为了避免冲突必须放弃 Mybatis 的配置，于是我放弃了这个方法。

### 1.3 自定义注解

我对自定义注解的理解就是通过一个特殊的标志来标记类、字段或方法，在代码中可以对含有这些标志的对象做特殊处理。自定义注解通过关键字 @interface 实现，自定义注解之上通常还有 @Target 和 @Retention 注解，@Target 表示注解的作用对象，如类、字段或方法等，@Retention 表示注解的生命周期。

## 2 开始

### 2.1 注解定义

#### 2.1.1 @Table 注解

该注解用于定义表名
```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface Table {
    String value();
}
```

#### 2.1.2 @Column 注解

该注解用于定义字段名以及是否模糊查询
```
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Column {
    String value() default "";
    boolean fuzzy() default false;
}
```

#### 2.1.3 @Id 注解

该注解用于标记表的主键字段
```
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Id {
}
```

### 2.2 Mybatis 基础方法封装

#### 2.2.1 CrudSqlSource 类定义

该类用于解析指定的类信息，并生成 SqlSource 对象，如
```
public SqlSource listByParamSqlSource() {
    SqlNode sqlNode = listSqlNode();
    return new DynamicSqlSource(configuration, new MixedSqlNode(Arrays.asList(sqlNode, applyWhere)));
}
private SqlNode listSqlNode() {
    SQL sql = new SQL();
    for (Field field : fields) {
        sql.SELECT(ModelUtil.getColumn(field));
    }
    sql.FROM(table);
    return new StaticTextSqlNode(sql.toString());
}
```

#### 2.2.2 CrudMappedStatement 类定义

该类用于为指定的 mapper 类生成自定义 MappedStatement 并添加到 Mybatis 配置中，如
```
public void addListByParamStatement() {
    SqlSource listByParam = crudSqlSource.listByParamSqlSource();
    MappedStatement.Builder mappedStatementBuilder = new MappedStatement.Builder(
            configuration,
            String.format("%s.listByParam", mapperInterface.getTypeName()),
            listByParam,
            SqlCommandType.SELECT
    );
    mappedStatementBuilder.resultMaps(resultMaps);
    configuration.addMappedStatement(mappedStatementBuilder.build());
}
```

#### 2.2.3 CrudMapperFactoryBean 类定义

该类为本次工作的核心，它的作用是对扫描到的 mapper 对象的处理，结合 Mybatis 自带的扫描注解 @MapperScan 完美实现对 mapper 对象添加自定义方法。

CrudMapperFactoryBean.java
```
public class CrudMapperFactoryBean<T> extends MapperFactoryBean<T> {
    public CrudMapperFactoryBean() {
        super();
    }

    public CrudMapperFactoryBean(Class<T> mapperInterface) {
        super(mapperInterface);
    }

    @Override
    protected void checkDaoConfig() {
        super.checkDaoConfig();
        // 添加Crud方法
        CrudMappedStatement crudMappedStatement = new CrudMappedStatement.Builder(super.getSqlSession().getConfiguration(), super.getMapperInterface()).build();
        crudMappedStatement.addCrudStatement();
    }
}
```
MybatisConfig.java
```
@AutoConfiguration
@MapperScan(basePackages = { "${mybatis.crud-mapper-package}" }, factoryBean = CrudMapperFactoryBean.class)
public class MybatisConfig {
    ...
}
```

### 2.3 基类方法封装

#### 2.3.1 CrudMapper 类

该类与生成的 mybatis 方法（MappedStatement）一一对应
```
public interface CrudMapper<T, ID extends Serializable> {
    int countByParam(T param);
    List<T> listByParam(T param);
    int save(T entity);
    int update(T entity);
    int remove(ID id);
    T get(ID id);
}
```

#### 2.3.2 CrudService 类和 CrudServiceImpl 类

CrudService.java
```
public interface CrudService<T, ID extends Serializable> {
    int save(T entity);
    int update(T entity);
    int remove(ID id);
    T get(ID id);
}
```
CrudServiceImpl.java
```
public abstract class CrudServiceImpl<T, ID extends Serializable> implements CrudService<T, ID> {
    protected abstract CrudMapper<T, ID> getMapper();

    @Override
    public int save(T entity) {
        return getMapper().save(entity);
    }

    @Override
    public int update(T entity) {
        return getMapper().update(entity);
    }

    @Override
    public int remove(ID id) {
        return getMapper().remove(id);
    }

    @Override
    public T get(ID id) {
        return getMapper().get(id);
    }
}
```

#### 2.3.3 CrudController 类

```
public abstract class CrudController<T, ID extends Serializable> {
    protected abstract CrudService<T, ID> getService();

    @PostMapping("/save")
    public Result<T> save(T entity) {
        int r = getService().save(entity);
        if (r > 0) {
            return Result.ok(entity);
        }
        return Result.error();
    }

    @PutMapping("/update")
    public Result<T> update(T entity) {
        int r = getService().update(entity);
        if (r > 0) {
            return Result.ok(entity);
        }
        return Result.error();
    }

    @DeleteMapping("/remove")
    public Result<Void> remove(ID id) {
        int r = getService().remove(id);
        if (r > 0) {
            return Result.ok();
        }
        return Result.error();
    }

    @GetMapping("/get")
    public Result<T> get(ID id) {
        T t = getService().get(id);
        if (t != null) {
            return Result.ok(t);
        }
        return Result.error();
    }
}
```

## 3 额外

### 3.1 分页功能

#### 3.1.1 分页类定义

Pageable.java
```
public class Pageable {
    private Integer page = 1;
    private Integer size = 10;

    public Integer getPage() {
        return page;
    }

    public void setPage(Integer page) {
        this.page = page;
    }

    public Integer getSize() {
        return size;
    }

    public void setSize(Integer size) {
        this.size = size;
    }
}
```
Page.java
```
public class Page<T> extends Pageable {
    private Integer total;
    private List<T> content;

    public Page(Integer page, Integer size) {
        super.setPage(page);
        super.setSize(size);
    }

    public Integer getTotal() {
        return total;
    }

    public void setTotal(Integer total) {
        this.total = total;
    }

    public List<T> getContent() {
        return content;
    }

    public void setContent(List<T> content) {
        this.content = content;
    }

    public static <T> Page<T> of(Pageable pageable) {
        return new Page<>(pageable.getPage(), pageable.getSize());
    }
}
```

#### 3.1.2 分页插件定义

该插件通过拦截 Executor 的 query 方法，如果 mapper 方法带有 Pageable 参数，则执行分页查询
```
@Component
@Intercepts({@Signature(
        type = Executor.class,
        method = "query",
        args = {MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class}
), @Signature(
        type = Executor.class,
        method = "query",
        args = {MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class, CacheKey.class, BoundSql.class}
)})
public class PageInterceptor implements Interceptor {
    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        ...

        // 分页参数
        Optional<Pageable> optional = ParameterUtils.getPageable(parameter);
        if (optional.isEmpty()) {
            return invocation.proceed();
        }
        Pageable pageable = optional.get();

        Page<Object> page = Page.of(pageable);
        MappedStatement countMs = buildCountMappedStatement(ms);
        Integer count = executeCount(executor, countMs, parameter, boundSql, rowBounds, resultHandler);
        page.setTotal(count);

        if (count > 0) {
            List<Object> list = pageQuery(executor, ms, parameter, rowBounds, resultHandler, boundSql, cacheKey, pageable);
            page.setContent(list);
        } else {
            page.setContent(Collections.emptyList());
        }

        return Collections.singletonList(page);
    }
}
```

#### 3.1.3 使用

```
public interface CrudMapper<T, ID extends Serializable> {
    ...

    // 分页
    Page<T> page(@Param("param") T param, @Param("page") Pageable pageable);
}
```

### 3.2 结果集处理

Mybatis 基础的 CRUD 方法封装完成之后，我发现还有一点瑕疵，因为我只生成了方法，但是对结果集的处理并不会按照我所定义的注解来映射字段，于是我打算再定义一个 ResultSetHandler 的插件来处理结果集，但是后来发现这并不是一个明智的方法，因为在 ResultSetHandler 中实际对结果集字段映射的代码只占据整个代码的一小部分，而我需要对整个 ResultSetHandler 重写才能实现这个功能。在多次阅读代码后发现字段映射的核心代码在于 MetaObject 的 findProperty 方法，该方法往上最终调用 Reflector 的 findPropertyName 方法，于是只需自定义 Reflector 的处理逻辑，并将 ReflectorFactory 添加到 Mybatis 配置中即可

AnnotationReflector.java
```
public class AnnotationReflector extends Reflector {
    private final Map<String, String> annotationPropertyMap = new HashMap<>();

    public AnnotationReflector(Class<?> clazz) {
        super(clazz);
        this.addAnnotationField(clazz);
    }

    private void addAnnotationField(Class<?> clazz) {
        Field[] fields = clazz.getDeclaredFields();
        for (Field field : fields) {
            // 下划线转驼峰
            String column = ModelUtil.getColumn(field).map(Column::value).map(value -> value.replace("_", ""))
                    .orElse(field.getName());
            annotationPropertyMap.put(column.toUpperCase(Locale.ENGLISH), field.getName());
        }
    }

    @Override
    public String findPropertyName(String name) {
        return Optional.ofNullable(annotationPropertyMap.get(name.toUpperCase(Locale.ENGLISH)))
                .orElse(super.findPropertyName(name));
    }
}
```
AnnotationReflectorFactory.java
```
public class AnnotationReflectorFactory implements ReflectorFactory {
    private boolean classCacheEnabled = true;
    private final ConcurrentMap<Class<?>, Reflector> reflectorMap = new ConcurrentHashMap<>();

    @Override
    public boolean isClassCacheEnabled() {
        return this.classCacheEnabled;
    }

    @Override
    public void setClassCacheEnabled(boolean classCacheEnabled) {
        this.classCacheEnabled = classCacheEnabled;
    }

    @Override
    public Reflector findForClass(Class<?> type) {
        return this.classCacheEnabled ? MapUtil.computeIfAbsent(this.reflectorMap, type, AnnotationReflector::new) : new AnnotationReflector(type);
    }
}
```
MybatisConfig.java
```
@AutoConfiguration
@MapperScan(basePackages = { "${mybatis.crud-mapper-package}" }, factoryBean = CrudMapperFactoryBean.class)
public class MybatisConfig {
    @Bean
    public ConfigurationCustomizer configurationCustomizer() {
        return config -> {
            config.setReflectorFactory(new AnnotationReflectorFactory());
            // 开启下划线转驼峰
            config.setMapUnderscoreToCamelCase(true);
            // 分页插件
            config.addInterceptor(new PageInterceptor());
        };
    }
}
```