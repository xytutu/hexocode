---
title: Maven AutoConfig使用
date: 2017-01-10 18:56:31
tags: Deploy
---

最近使用MVN的AutoConfig功能比较多，这里详细了解一下具体的使用：
### MVN 依赖
```
<plugin>
      <groupId>com.alibaba.citrus.tool</groupId>
      <artifactId>maven-autoconfig-plugin</artifactId>
      <version>1.0.10</version>
      <configuration>
        <exploding>true</exploding>
      </configuration>
      <executions>
        <execution>
          <phase>package</phase>
          <goals>
            <goal>autoconfig</goal>
          </goals>
        </execution>
      </executions>
</plugin>
```

### 使用方式
文档结构
```
META-INF
  --autoconfig
    --auto-config.xml
```

* autoConfig支持在build的时候动态替换存在的某文件。（注意事项：要替换的文件相对位置是针对META-INF来说的）

```
<?xml version="1.0" encoding="UTF-8"?>
<config>
    <group>
        <property name="version" defaultValue="1.0.0.daily" description="版本" />
    </group>
    <script>
        <generate template="WEB-INF/config/spring.xml" charset="UTF-8" />
    </script>
</config>
```

* autoConfig支持在build的时候动态替换存在的某文件，并且将文件拷贝到目的地。

```
  <?xml version="1.0" encoding="UTF-8"?>
  <config>
    <group>
       <property name="version" defaultValue="1.0.0.daily" description="版本" />
    </group>
    <script>
        <generate template="spring.xml.vm" charset="UTF-8" destfile="WEB-INF/spring.xml"/>
    </script>
  </config>
```

  
  
[参考-autoconfig文档](http://openwebx.org/docs/autoconfig.html)
