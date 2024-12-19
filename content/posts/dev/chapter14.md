---
title: "Spring 整合 Minio"
date: 2024-10-12T10:00:00+08:00
categories: ["development"]
tags: ["java"]
draft: false
---

## 介绍

对象存储

## 1 环境配置

参考 [Minio 安装](../../ops/chapter14)

## 2 示例

### 2.1 Spring 配置

添加 application 配置
```
minio:
  url: http://localhost:9000
  accessKey: admin
  secretKey: password
  bucketName: xxx
```
定义配置类 OssProperties（用于获取配置信息）
```java
@Component
@ConfigurationProperties("minio")
public class OssProperties {
    private String url;

    private String accessKey;

    private String secretKey;

    private String bucketName;

    ...
}
```
定义 Oss 服务接口 IOssService
```java
public interface IOssService {
    /**
     * 是否存在桶
     */
    boolean existBucket(String name);

    /**
     * 创建桶
     */
    void createBucket(String name);

    /**
     * 删除桶
     */
    void removeBucket(String name);

    /**
     * 上传文件
     */
    void uploadFile(String bucket, File file);

    /**
     * 下载文件
     */
    void downloadFile(String bucket, String filename, String target);

    /**
     * 删除文件
     */
    void removeFile(String bucket, String filename);
}
```

### 2.2 Minio 客户端

添加依赖
```
<dependency>
    <groupId>io.minio</groupId>
    <artifactId>minio</artifactId>
    <version>${minio.version}</version>
</dependency>
```
实现 IOssService 接口
```java
public class OssMinioServiceImpl implements IOssService {
    private final String BASE_PATH = "test/";
    private final MinioClient minioClient;

    public OssMinioServiceImpl(OssProperties properties) {
        this.minioClient = MinioClient.builder()
                .endpoint(properties.getUrl())
                .credentials(properties.getAccessKey(), properties.getSecretKey())
                .build();
    }

    ...

    /**
     * 上传文件
     */
    @Override
    public void uploadFile(String bucket, File file) {
        try (FileInputStream in = new FileInputStream(file)) {
            minioClient.putObject(PutObjectArgs.builder()
                    .bucket(bucket)
                    .object(BASE_PATH + file.getName())
                    .stream(in, in.available(), -1)
                    .build());
        } catch (ErrorResponseException | InsufficientDataException | InternalException | InvalidKeyException
                | InvalidResponseException | IOException | NoSuchAlgorithmException | ServerException | XmlParserException e) {
            e.printStackTrace();
        }
    }

    /**
     * 下载文件
     */
    @Override
    public void downloadFile(String bucket, String filename, String target) {
        InputStream in = null;
        FileOutputStream out = null;
        try {
            in = minioClient.getObject(GetObjectArgs.builder()
                    .bucket(bucket)
                    .object(BASE_PATH + filename)
                    .build());
            out = new FileOutputStream(target);
            in.transferTo(out);
        } catch (ErrorResponseException | InsufficientDataException | InternalException | InvalidKeyException
                | InvalidResponseException | IOException | NoSuchAlgorithmException | ServerException | XmlParserException e) {
            e.printStackTrace();
        } finally {
            // close
            ...
        }
    }

    /**
     * 删除文件
     */
    @Override
    public void removeFile(String bucket, String filename) {
        try {
            minioClient.removeObject(RemoveObjectArgs.builder()
                    .bucket(bucket)
                    .object(BASE_PATH + filename)
                    .build());
        } catch (ErrorResponseException | InsufficientDataException | InternalException | InvalidKeyException
                | InvalidResponseException | IOException | NoSuchAlgorithmException | ServerException | XmlParserException e) {
            e.printStackTrace();
        }
    }
}
```

### 2.3 Amazon 客户端

添加依赖
```
<dependency>
    <groupId>com.amazonaws</groupId>
    <artifactId>aws-java-sdk-s3</artifactId>
    <version>${aws-java-sdk-s3.version}</version>
</dependency>
```
实现 IOssService 接口
```java
public class OssAmazonServiceImpl implements IOssService {
    private final String BASE_PATH = "test/";
    private final AmazonS3 client;

    public OssAmazonServiceImpl(OssProperties properties) {
        AwsClientBuilder.EndpointConfiguration endpointConfig = new AwsClientBuilder.EndpointConfiguration(properties.getUrl(), null);
        AWSCredentials credentials = new BasicAWSCredentials(properties.getAccessKey(), properties.getSecretKey());
        AWSCredentialsProvider credentialsProvider = new AWSStaticCredentialsProvider(credentials);
        ClientConfiguration clientConfig = new ClientConfiguration();
        clientConfig.setProtocol(Protocol.HTTP);
        AmazonS3ClientBuilder build = AmazonS3Client.builder()
                .withEndpointConfiguration(endpointConfig)
                .withClientConfiguration(clientConfig)
                .withCredentials(credentialsProvider)
                .disableChunkedEncoding()
                .enablePathStyleAccess();
        this.client = build.build();
    }

    ...

    /**
     * 上传文件
     */
    @Override
    public void uploadFile(String bucket, File file) {
        try (FileInputStream in = new FileInputStream(file)) {
            ObjectMetadata metadata = new ObjectMetadata();
            PutObjectRequest putObjectRequest = new PutObjectRequest(bucket, BASE_PATH + file.getName(), in, metadata);
            putObjectRequest.setCannedAcl(CannedAccessControlList.Private);
            client.putObject(putObjectRequest);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * 下载文件
     */
    @Override
    public void downloadFile(String bucket, String filename, String target) {
        InputStream in = null;
        FileOutputStream out = null;
        try {
            S3Object object = client.getObject(bucket, BASE_PATH + filename);
            in = object.getObjectContent();
            out = new FileOutputStream(target);
            in.transferTo(out);
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            // close
            ...
        }
    }

    /**
     * 删除文件
     */
    @Override
    public void removeFile(String bucket, String filename) {
        client.deleteObject(bucket, BASE_PATH + filename);
    }
}
```

### 2.4 测试

```java
@SpringBootTest
class ApplicationTests {
    private final String BUCKET = "xxx";
    @Autowired
    private IOssService ossService;

    @Test
    void uploadFileTest() {
        if (!ossService.existBucket(BUCKET)) {
            ossService.createBucket(BUCKET);
        }
        File file = new File("src/test/resources/test.txt");
        ossService.uploadFile(BUCKET, file);
    }

    @Test
    void downloadFileTest() {
        ossService.downloadFile(BUCKET, "test.txt", "src/test/resources/test.txt");
    }

    @Test
    void removeFileTest() {
        ossService.removeFile(BUCKET, "test.txt");
    }
}
```