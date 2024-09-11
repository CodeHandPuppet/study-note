### 分析
> Spring Cloud Sleuth 无法与 Spring Boot 3.x 及更高版本兼容。Sleuth 支持的 Spring Boot 的最后一个主要版本是 2.x。

官网 [spring-cloud-sleuth](https://docs.spring.io/spring-cloud-sleuth/docs/3.1.10-SNAPSHOT/reference/html/)

选择 [Micromemter](https://micrometer.io/) 代替 Sleuth

> Micromemter 和 Sleuth 都是 进行数据采样 
> [ZipKin](https://zipkin.io) 进行图形的显示


### Micromemter + ZipKin 实战
1. springboot 依赖导入
   ``` xml
     <properties>
       <micrometer-tracing.version>1.2.0</micrometer-tracing.version>
        <micrometer-observation.version>1.12.0</micrometer-observation.version>
        <feign-micrometer.version>12.5</feign-micrometer.version>
        <zipkin-reporter-brave.version>2.17.0</zipkin-reporter-brave.version>
     </properties>
    
    <dependecies>
            <dependency>
            <groupId>io.micrometer</groupId>
            <artifactId>micrometer-tracing-bom</artifactId>
            <version>${micrometer-tracing.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        <!--micrometer-tracing指标追踪  2-->
        <dependency>
            <groupId>io.micrometer</groupId>
            <artifactId>micrometer-tracing</artifactId>
            <version>${micrometer-tracing.version}</version>
        </dependency>
        <!--micrometer-tracing-bridge-brave适配zipkin的桥接包 3-->
        <dependency>
            <groupId>io.micrometer</groupId>
            <artifactId>micrometer-tracing-bridge-brave</artifactId>
            <version>${micrometer-tracing.version}</version>
        </dependency>
        <!--micrometer-observation 4-->
        <dependency>
            <groupId>io.micrometer</groupId>
            <artifactId>micrometer-observation</artifactId>
            <version>${micrometer-observation.version}</version>
        </dependency>
        <!--feign-micrometer 5-->
        <dependency>
            <groupId>io.github.openfeign</groupId>
            <artifactId>feign-micrometer</artifactId>
            <version>${feign-micrometer.version}</version>
        </dependency>
        <!--zipkin-reporter-brave 6-->
        <dependency>
            <groupId>io.zipkin.reporter2</groupId>
            <artifactId>zipkin-reporter-brave</artifactId>
            <version>${zipkin-reporter-brave.version}</version>
        </dependency>
    
    </dependecies>
   ```
2. yml 配置
    - 源码配置 
      - `package org.springframework.boot.actuate.autoconfigure.tracing.zipkin.ZipkinProperties`
      - `package org.springframework.boot.actuate.autoconfigure.tracing.TracingProperties`
    - 
    ``` yml
        management:
          zipkin:
            tracing:
              endpoint: http://localhost:9411/api/v2/spans
          tracing:
            sampling:
              probability: 1.0 #采样率默认为0.1(0.1就是10次只能有一次被记录下来)，值越大收集越及时。 
        ``` 
3. 打开 ZipKin  localhost:9411