### canal
 
>[阿里巴巴GitHubcanal官网](https://github.com/alibaba/canal)

TIP：
1. linux 需要java8环境 canal为canal.deployer-1.1.6.tar.gz
2. ``` sql 
    select * from mysql.user;
    DROP USER IF EXISTS 'canal'@'%';
    CREATE USER 'canal'@'%' IDENTIFIED BY 'canal';
    GRANT ALL PRIVILEGES ON *.* TO 'canal'@'%' with grant option ;
    FLUSH PRIVILEGES;
    ALTER USER 'canal'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
    SELECT * FROM mysql.user;
    ```
3. >SHOW VARIABLES LIKE 'log_bin'; 一定要开启log_bin的写入功能


- 下载canal linux 环境
  官网下载 [阿里巴巴GitHubcanal官网](https://github.com/alibaba/canal)
- 解压
  - tar -zxvf canal.deployer-1.1.6.tar.gz
- 配置
  - 修改 /mycanal/conf/example 路径下的 instance.properties 文件
    ``` shell
        # position info  
        canal.instance.master.address= 自己mysql服务器地址 
        # username/password
        canal.instance.dbUsername=canal  // 操控数据库的账号
        canal.instance.dbPassword=123456  //  超控数据库的密码
    ```
- 开启canal
    - cd bin/
    - ./startup.sh
    - 查看 logs/canal/canal.log 
        ``` log
                2024-07-16 02:19:20.043 [main] INFO  com.alibaba.otter.canal.deployer.CanalStarter - ## start the canal server.
                2024-07-16 02:19:20.070 [main] INFO  com.alibaba.otter.canal.deployer.CanalController - ## start the canal server[192.168.122.1(192.168.122.1):11111]
                2024-07-16 02:19:20.956 [main] INFO  com.alibaba.otter.canal.deployer.CanalStarter - ## the canal server is running now ...... 
        ```
- 测试java程序     
  - 导入依赖
    ``` xml
          <?xml version="1.0" encoding="UTF-8"?>
      <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
          <modelVersion>4.0.0</modelVersion>
          <parent>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-parent</artifactId>
              <version>3.3.1</version>
              <relativePath/> <!-- lookup parent from repository -->
          </parent>

          <groupId>com.example</groupId>
          <artifactId>redis-demo</artifactId>
          <version>0.0.1-SNAPSHOT</version>
          <name>redis-demo</name>
          <description>redis-demo</description>
          <url/>
          <licenses>
              <license/>
          </licenses>
          <developers>
              <developer/>
          </developers>
          <scm>
              <connection/>
              <developerConnection/>
              <tag/>
              <url/>
          </scm>
          <properties>
              <java.version>17</java.version>
          </properties>
          <dependencies>
              <dependency>
                  <groupId>org.springframework.boot</groupId>
                  <artifactId>spring-boot-starter-data-redis</artifactId>
              </dependency>
              <dependency>
                  <groupId>org.springframework.boot</groupId>
                  <artifactId>spring-boot-starter-jdbc</artifactId>
              </dependency>
              <dependency>
                  <groupId>org.springframework.boot</groupId>
                  <artifactId>spring-boot-starter-web</artifactId>
              </dependency>

              <dependency>
                  <groupId>org.mybatis.spring.boot</groupId>
                  <artifactId>mybatis-spring-boot-starter</artifactId>
                  <version>3.0.3</version>
              </dependency>
              <dependency>
                  <groupId>com.alibaba</groupId>
                  <artifactId>druid-spring-boot-3-starter</artifactId>
                  <version>1.2.23</version>

              </dependency>
              <dependency>
                  <groupId>org.springdoc</groupId>
                  <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
                  <version>2.1.0</version>
              </dependency>


              <dependency>
                  <groupId>com.alibaba.otter</groupId>
                  <artifactId>canal.client</artifactId>
                  <version>1.1.0</version>
              </dependency>
      <!--	驼峰转化的包	-->
              <dependency>
                  <groupId>com.google.guava</groupId>
                  <artifactId>guava</artifactId>
                  <version>21.0</version>
              </dependency>

              <dependency>
                  <groupId>com.mysql</groupId>
                  <artifactId>mysql-connector-j</artifactId>
                  <scope>runtime</scope>
              </dependency>
              <dependency>
                  <groupId>org.projectlombok</groupId>
                  <artifactId>lombok</artifactId>
                  <optional>true</optional>
              </dependency>
              <dependency>
                  <groupId>org.springframework.boot</groupId>
                  <artifactId>spring-boot-starter-test</artifactId>
                  <scope>test</scope>
              </dependency>
              <dependency>
                  <groupId>org.mybatis.spring.boot</groupId>
                  <artifactId>mybatis-spring-boot-starter-test</artifactId>
                  <version>3.0.3</version>
                  <scope>test</scope>
              </dependency>

          </dependencies>

          <build>
              <plugins>
                  <plugin>
                      <groupId>org.springframework.boot</groupId>
                      <artifactId>spring-boot-maven-plugin</artifactId>
                      <configuration>
                          <excludes>
                              <exclude>
                                  <groupId>org.projectlombok</groupId>
                                  <artifactId>lombok</artifactId>
                              </exclude>
                          </excludes>
                      </configuration>
                  </plugin>
              </plugins>
          </build>

      </project>


    ```
  - 实现双写一致性代码
      ``` java

    package com.example.redisdemo;

    import com.google.common.base.CaseFormat;
    import jakarta.annotation.Resource;
    import org.junit.jupiter.api.Test;
    import org.springframework.boot.test.context.SpringBootTest;
    import com.alibaba.fastjson.JSONObject;
    import com.alibaba.otter.canal.client.CanalConnector;
    import com.alibaba.otter.canal.client.CanalConnectors;
    import com.alibaba.otter.canal.protocol.CanalEntry.*;
    import com.alibaba.otter.canal.protocol.Message;
    import org.springframework.data.redis.core.RedisTemplate;


    import java.net.InetSocketAddress;
    import java.util.List;
    import java.util.UUID;
    import java.util.concurrent.TimeUnit;

    @SpringBootTest
    class RedisDemoApplicationTests {

        @Resource
        RedisTemplate redisTemplate;

        public static final Integer _60SECONDS = 60;
        public static final String REDIS_IP_ADDR = "192.168.153.128";
        public static final String CACHE_KEY_USER = "user:";
        @Test
        void contextLoads() {
            System.out.println("---------O(∩_∩)O哈哈~ initCanal() main方法-----------");

            //=================================
            // 创建链接canal服务端
            CanalConnector connector = CanalConnectors.newSingleConnector(new InetSocketAddress(REDIS_IP_ADDR,
                    11111), "example", "", "");  // 这里用户名和密码如果在这写了，会覆盖canal配置文件的账号密码，如果不填从配置文件中读
            int batchSize = 1000;
            //空闲空转计数器
            int emptyCount = 0;
            System.out.println("---------------------canal init OK，开始监听mysql变化------");
            try {
                connector.connect();
                //connector.subscribe(".*\\..*");
                connector.subscribe("springboot_study.user");   // 设置监听哪个表
                connector.rollback();
                int totalEmptyCount = 10 * _60SECONDS;
                while (emptyCount < totalEmptyCount) {
                    System.out.println("我是canal，每秒一次正在监听:" + UUID.randomUUID().toString());
                    Message message = connector.getWithoutAck(batchSize); // 获取指定数量的数据
                    long batchId = message.getId();
                    int size = message.getEntries().size();
                    if (batchId == -1 || size == 0) {
                        emptyCount++;
                        try {
                            TimeUnit.SECONDS.sleep(1);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    } else {
                        //计数器重新置零
                        emptyCount = 0;
                        printEntry(message.getEntries());
                    }
                    connector.ack(batchId); // 提交确认
                    // connector.rollback(batchId); // 处理失败, 回滚数据
                }
                System.out.println("已经监听了" + totalEmptyCount + "秒，无任何消息，请重启重试......");
            } finally {
                connector.disconnect();
            }


        }

        private void redisInsert(List<Column> columns) {

            JSONObject jsonObject = new JSONObject();
            for (Column column : columns) {
                System.out.println(column.getName() + " : " + column.getValue() + "    update=" + column.getUpdated());
                jsonObject.put(LowerUnderLineToCamel(column.getName()), column.getValue());
            }
            if (columns.size() > 0) {
                redisTemplate.opsForValue().set(CACHE_KEY_USER+columns.get(0).getValue(), jsonObject);


            }
        }


        private void redisDelete(List<Column> columns) {
            JSONObject jsonObject = new JSONObject();
            for (Column column : columns) {
                jsonObject.put(LowerUnderLineToCamel(column.getName()), column.getValue());
            }
            if (columns.size() > 0) {

                redisTemplate.delete(CACHE_KEY_USER+columns.get(0).getValue());

            }
        }

        private void redisUpdate(List<Column> columns) {
            JSONObject jsonObject = new JSONObject();
            for (Column column : columns) {
                System.out.println(column.getName() + " : " + column.getValue() + "    update=" + column.getUpdated());
                jsonObject.put(LowerUnderLineToCamel(column.getName()), column.getValue());
            }
            if (columns.size() > 0) {

                redisTemplate.opsForValue().set(CACHE_KEY_USER+columns.get(0).getValue(), jsonObject);
                System.out.println("---------update after: " + redisTemplate.opsForValue().get(columns.get(0).getValue()));

            }
        }

        public void printEntry(List<Entry> entrys) {
            for (Entry entry : entrys) {
                if (entry.getEntryType() == EntryType.TRANSACTIONBEGIN || entry.getEntryType() == EntryType.TRANSACTIONEND) {
                    continue;
                }

                RowChange rowChage = null;
                try {
                    //获取变更的row数据
                    rowChage = RowChange.parseFrom(entry.getStoreValue());
                } catch (Exception e) {
                    throw new RuntimeException("ERROR ## parser of eromanga-event has an error,data:" + entry.toString(), e);
                }
                //获取变动类型
                EventType eventType = rowChage.getEventType();
                System.out.println(String.format("================&gt; binlog[%s:%s] , name[%s,%s] , eventType : %s",
                        entry.getHeader().getLogfileName(), entry.getHeader().getLogfileOffset(),
                        entry.getHeader().getSchemaName(), entry.getHeader().getTableName(), eventType));

                for (RowData rowData : rowChage.getRowDatasList()) {
                    if (eventType == EventType.INSERT) {
                        redisInsert(rowData.getAfterColumnsList());
                    } else if (eventType == EventType.DELETE) {
                        redisDelete(rowData.getBeforeColumnsList());
                    } else {//EventType.UPDATE
                        redisUpdate(rowData.getAfterColumnsList());
                    }
                }
            }
        }



        public String LowerUnderLineToCamel(String lowerUnderLine) {


            return CaseFormat.LOWER_UNDERSCORE.to(CaseFormat.LOWER_CAMEL, lowerUnderLine);
        }
    }
    ```