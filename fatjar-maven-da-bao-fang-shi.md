---
description: 打包完成後，只會有一個jar檔，連同參數一起封裝。
---

# FatJar Maven 打包方式

### Jar檔內部resources 路徑取得方式

直接在根目錄就等同於resources資料夾內。

下列為spring 註冊檔手動載入範例。

`ApplicationContext context = new ClassPathXmlApplicationContext("/config/applicationContext.xml");`



### Pom檔設定，使用Shade套件

```
<profile>
            <id>dev</id>
            <build>
                <finalName>wcp-batch-service</finalName>
                <plugins>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-shade-plugin</artifactId>
                        <executions>
                            <execution>
                                <phase>package</phase>
                                <goals>
                                    <goal>shade</goal>
                                </goals>
                                <configuration>
                                    <filters>
                                        <filter>
                                        <!--移除重複依賴，例如log4j--!>
                                            <artifact>*:*</artifact>
                                            <excludes>
                                                <exclude>**/Log4j2Plugins.dat</exclude>
                                                <exclude>META-INF/*.SF</exclude>
                                                <exclude>META-INF/*.DSA</exclude>
                                                <exclude>META-INF/*.RSA</exclude>
                                            </excludes>
                                        </filter>
                                    </filters>
                                      <!--轉換主要入口點、Spring bean xml設定檔--!>
                                  <transformers>
                                        <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                            <mainClass>esb.wcp.batch.DailyBatchTrigger</mainClass>
                                        </transformer>
                                        <transformer implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                                            <resource>META-INF/spring.handlers</resource>
                                        </transformer>
                                        <transformer implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                                            <resource>META-INF/spring.schemas</resource>
                                        </transformer>
                                    </transformers>
                                </configuration>
                            </execution>
                        </executions>
                    </plugin>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-compiler-plugin</artifactId>
                        <version>3.8.1</version>
                        <configuration>
                            <source>1.8</source> <!-- depending on your project -->
                            <target>1.8</target> <!-- depending on your project -->
                            <annotationProcessorPaths>
                                <path>
                                    <groupId>org.mapstruct</groupId>
                                    <artifactId>mapstruct-processor</artifactId>
                                    <version>1.4.2.Final</version>
                                </path>
                                <!-- other annotation processors -->
                            </annotationProcessorPaths>
                        </configuration>
                    </plugin>
                    <plugin>
                        <artifactId>maven-resources-plugin</artifactId>
                        <version>3.1.0</version>
                        <executions>
                            <execution>
                                <id>copy-xmls</id>
                                <phase>process-sources</phase>
                                <goals>
                                    <goal>copy-resources</goal>
                                </goals>
                                <configuration>
                                    <outputDirectory>src/main/resources</outputDirectory>
                                    <resources>
                                        <resource>
                                            <directory>src/main/sit</directory>
                                            <filtering>true</filtering>
                                        </resource>
                                    </resources>
                                </configuration>
                            </execution>
                        </executions>
                    </plugin>
                </plugins>
                <resources>
                    <resource>
                        <directory>src/main/resources</directory>
                        <filtering>true</filtering>
                    </resource>
                </resources>
            </build>
        </profile>
```
