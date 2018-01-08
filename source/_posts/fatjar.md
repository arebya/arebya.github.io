---
title: spring boot fatjar的扫描
date: 2018-01-05 13:57:27
tags: springboot
---

### spring-boot-maven-plugin

fat jar的结构
!["结构"](/img/layout.png)

### fat jar问题

问题：[Jersey doesn't always work with Spring Boot fat jars](https://github.com/spring-projects/spring-boot/issues/1468)

发布说明：1.4版本提供了处理方法https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-1.4-Release-Notes

### 处理方式

#### 打包

打包插件添加需要特别处理的jar包如：
```
<build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <requiresUnpack>
                        <!--指定需要unpack的依赖-->
                        <dependency>
                            <groupId>com.netflix.eureka</groupId>
                            <artifactId>eureka-core</artifactId>
                        </dependency>
                        <dependency>
                            <groupId>com.netflix.eureka</groupId>
                            <artifactId>eureka-client</artifactId>
                        </dependency>
                    </requiresUnpack>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

查看插件打完的fat jar信息

!["comment"](/img/zipnote.png)

#### spring boot 应用启动逻辑

springboot的JarLauncher继承自ExecutableArchiveLauncher，会从manifest文件中获取main-class，start-class

spring boot 实现了自定义的classLoader——LaunchedURLClassLoader，以当前fat jar开始进行扫描BOOT-INF/classes/，BOOT-INF/lib/
```
protected Archive getNestedArchive(Archive.Entry entry) throws IOException {
    JarEntry jarEntry = ((JarFileEntry)entry).getJarEntry();
    // 对需要unpack的jar 特殊处理
    if (jarEntry.getComment().startsWith("UNPACK:"))
      return getUnpackedNestedArchive(jarEntry);
    try
    {
      JarFile jarFile = this.jarFile.getNestedJarFile(jarEntry);
      return new JarFileArchive(jarFile);
    }
    catch (Exception ex)
    {
      throw new IllegalStateException("Failed to get nested archive for entry " + entry
        .getName(), ex);
    }
  }
```
首次运行时进行解压：

```
 private Archive getUnpackedNestedArchive(JarEntry jarEntry) throws IOException {
    String name = jarEntry.getName();
    if (name.lastIndexOf("/") != -1) {
      name = name.substring(name.lastIndexOf("/") + 1);
    }
    File file = new File(getTempUnpackFolder(), name);
    if ((!file.exists()) || (file.length() != jarEntry.getSize())) {
       // 解压到tmp目录，子目录是如fatjar name-spring-boot-libs-2264fe79-9952-40a4-a81e-4c71af37eb57/之类的，具体目录生成规则见最后一个函数 
        unpack(jarEntry, file);
    }
    return new JarFileArchive(file, file.toURI().toURL());
  }

  private File getTempUnpackFolder() {
    if (this.tempUnpackFolder == null) {
      File tempFolder = new File(System.getProperty("java.io.tmpdir"));
      this.tempUnpackFolder = createUnpackFolder(tempFolder);
    }
    return this.tempUnpackFolder;
  }

  private File createUnpackFolder(File parent) {
    int attempts = 0;
    while (attempts++ < 1000) {
      String fileName = new File(this.jarFile.getName()).getName();

      File unpackFolder = new File(parent, fileName + "-spring-boot-libs-" + 
        UUID.randomUUID());
      if (unpackFolder.mkdirs()) {
        return unpackFolder;
      }
    }
    throw new IllegalStateException("Failed to create unpack folder in directory '" + parent + "'");
  }
```

