# 国图大云
采用微服务实现的国图应用服务框架，以轻量化Restful标准定义服务接口，统一应用与服务边界，提供一致性跨语言的应用解决方案。

## 一、云服务治理框架内涵

- 云服务技术标准
- 云服务注册中心（center）
- 云服务配置管理（config）
- 云服务调用追踪（collector）
- 云服务日志管理（admin）
- 云服务网关（gateway）
- 异构云服务注册
- 容器管理（admin）
- 应用模板管理（admin）
- 主机管理（admin）
- 镜像库管理（admin）

## 二、核心云服务
国图大云除提供标准技术方案还提供基础云服务，包括：

- 统一授权认证中心（account）
- 资源共享中心
- 云门户
- 文档管理中心（storage）
- 全文索引
- 消息服务
- 工作流引擎
- 表单服务
- 行政区区划服务（region）
- 开发中心

## 三、基于框架如何实现基础云服务（gtcc/samples/cloud-app-sample）
### 1.新建Maven工程，POM中设置parnet、添加依赖、设置编译方式
```xml
    <parent>
        <artifactId>parent</artifactId>
        <groupId>cn.gtmap.gtcc</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
```
```xml
    <dependencies>

        <dependency>
            <groupId>cn.gtmap.gtcc</groupId>
            <artifactId>gtmap-cloud-app-starter</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <!-- additional -->

    </dependencies>
```
```xml
    <build>
        <plugins>
            <!-- important -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-antrun-plugin</artifactId>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-assembly-plugin</artifactId>
            </plugin>
            <!-- important -->

            <!-- additional -->

        </plugins>
    </build>
```
### 2.新建主入口类，注解 @GTMapCloudApplication 
```java
@GTMapCloudApplication
public class App {

    public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }

}
```
### 3.添加配置文件，设置相关配置参数，添加自定义参数配置
```bash
application.yml
bootstrap.yml
logback.xml
banner.txt
```
### 4.拷贝build内容（工程编译打包方式）
```bash
.bin/docker-compose.yml
.bin/Dockerfile
.bin/image-build.bat
.bin/image-push.bat
.bin/start.bat
.bin/start.sh
assembly.xml
```
### 5.主入口运行启动应用
### 6.编译打包
```bash
mvn clean package
```
## 四、使用授权认证框架云服务实现（gtcc/samples/oauth-app-sample）
### 1.Maven dependency 改为：
```xml
        <dependency>
            <groupId>cn.gtmap.gtcc</groupId>
            <artifactId>gtmap-cloud-app-security-starter</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
```
### 2.主入口应用注解改为 @GTMapSecurityApplication：

## 五、重点配置详解
### application.yml
```yaml
app:
  # 云应用注册中心地址
  center: http://172.18.31.191:8000/eureka/ 
  # 授权认证服务器地址
  oauth: http://172.18.31.191:8888/account
  # 安全过滤规则，使用Spring EL表达式配置，除配置外还可使用方法注解实现安全过滤
  security:
    ignores: /image/**,/css/**,/js/**,/webjars/**
    # 本工程安全过滤选项
    authorities:
      permitAll: /index2
      hasAnyAuthority('ROLE_USER'): /index,/call2
      hasAnyAuthority('ROLE_ADMIN_2'): /call
    # oauth2 过滤
    resources:
      hasScope('read'): /rest/**

spring:
  # 默认通过MQ收集各个服务节点产生的日志，亦可用于系统其它用处
  rabbitmq:
    host: 172.24.19.183
    port: 5672
    username: guest
    password: guest

server:
  # 指定应用服务开放端口
  port: ${port:10001}

# 调整日志输出级别
logging:
  level:
    org.springframework: INFO
    cn.gtmap.gtcc: DEBUG
    cn.gtmap.gtcc.client: WARN
```
### bootstrap.yml
```yaml
spring:
  application:
    # 云应用名称，用于标识该应用，服务发现、动态负载也根据此名称
    name: oas-app
```

## 六、云服务接口定义与调用
云服务接口统一约束定义为轻量级的Restful形式，可跨语言调用。客户端通过定义FeignClient调用服务。

### 服务提供者：
```java
@RestController
@RequestMapping("/rest/region")
public class RegionController {
    
    ...
    
    @GetMapping("/{id}")
    public Region get(@PathVariable(name = "id") String id) {
        return regionService.getRegion(id);
    }
    
    ...
}
```

### 服务调用者：
#### 1.定义RegionClient接口
```java
/**
* 通过注册中心获取服务的填写name参数，值为对应服务服务名称；
* 未注册到注册中心上的直接服务调用填写url参数
*/
@FeignClient("region-app")
@RequestMapping("/rest/region")
public interface RegionClient {
    
    ...
    
    @GetMapping("/{id}")
    Region get(@PathVariable(name = "id") String id);
    ...
    
}
```
#### 2.调用服务
```java
    ...
    
    @Autowired
    RegionClient regionClient;

    ...
    
    public Region getRegion(String id) {
        return regionClient.get(id);
    }
        
```