# liquibase customChange

liquibase changeset 执行Java代码。

liquibase支持yml等文件，支持引入sql文件，还支持Java这种方式执行change。

对于执行 DDL DML 使用sql很方便，但是我想执行一些数据处理，将几个表中的数据放到新建的表中，那么就需要写存储过程。liquibase文档中支持使用存储过程，但是他也支持使用customChange来使用Java代码。

[customChange](https://docs.liquibase.com/change-types/custom-change.html)



## 先说sql文件的使用

maven：

```
        <dependency>
            <groupId>org.liquibase</groupId>
            <artifactId>liquibase-core</artifactId>
        </dependency>
```

application.yml

```
spring:
  liquibase:
    change-log: classpath:db/changelog/db.changelog-master.yml
    enabled: true
 
 ----------------------------
 spring数据库相关的配置 .......
```

目录结构

![image-20230811153742075](https://pic-keboom.oss-cn-hangzhou.aliyuncs.com/img/image-20230811153742075.png)

db.changelog-master.yml文件部分内容

```yml
databaseChangeLog:
  - preConditions:
      - dbms:
          type: mysql
      - runningAs:
          username: root
  - changeSet:
      id: 202308081722
      author: keboom
      changes:
        - sqlFile:
            dbms: mysql
            endDelimiter: ";"
            encoding: "UTF-8"
            path: classpath:/db/changelog/changes/add_board_manager.sql
  - changeSet:
      id: 202308090949
      author: keboom
      changes:
        - sqlFile:
            dbms: mysql
            endDelimiter: ";"
            encoding: "UTF-8"
            path: classpath:/db/changelog/changes/dest_device_table_add_isSDI_column.sql
```

然后启动项目即可使用。



## 再说Java文件的使用

在db.changelog-master.yml文件新添加：

```java
  - changeSet:
      id: 202308111016
      author: keboom
      changes:
        - customChange: { "class": "com.example.web.changelog.InitRouteDestRel" }
```

InitRouteDestRel 类如下：

```
public class InitRouteDestRel implements CustomTaskChange {

    @Override
    public void execute(Database database) throws CustomChangeException {
        ApplicationContext context = ApplicationContextProvider.getApplicationContext();
        RouteDao routeDao = context.getBean(RouteDao.class);
        RouteDestRelDao routeDestRelDao = context.getBean(RouteDestRelDao.class);

        List<Route> routeList = routeDao.list();
        LinkedList<RouteDestRelDO> insertCollections = new LinkedList<>();
        for (Route route : routeList) {
            String destIds = route.getDestIds();
            if (StringUtils.isEmpty(destIds)) {
                continue;
            }
            String[] destArr = StringUtils.split(destIds, ",");
            for (String s : destArr) {
                RouteDestRelDO routeDestRelDO = new RouteDestRelDO(route.getId(), Integer.valueOf(s));
                insertCollections.add(routeDestRelDO);
            }
        }
        routeDestRelDao.saveBatch(insertCollections);


    }

    @Override
    public String getConfirmationMessage() {
        return null;
    }

    @Override
    public void setUp() throws SetupException {

    }

    @Override
    public void setFileOpener(ResourceAccessor resourceAccessor) {

    }

    @Override
    public ValidationErrors validate(Database database) {
        return null;
    }
}
```

在execute方法里面执行要做的操作即可。但是liquibase这个是不归springboot管的，里面用各种注解，还是什么构造器注入都不行。我们可以通过获得ApplicationContext来获得想要的 dao 类。

参考：https://www.baeldung.com/spring-get-current-applicationcontext

```
import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.stereotype.Component;

/**
 * {@code @author:} keboom
 * {@code @date:} 2023/8/11
 * {@code @description:}
 */
@Component
public class ApplicationContextProvider implements ApplicationContextAware {
    private static ApplicationContext applicationContext;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        ApplicationContextProvider.applicationContext = applicationContext;
    }

    public static ApplicationContext getApplicationContext() {
        return applicationContext;
    }
}

```



参考：

[stackoverflow-use other spring beans in liquibase customtaskchange](https://stackoverflow.com/questions/32826600/use-other-spring-beans-in-liquibase-customtaskchange-class#:~:text=Add%20a%20comment-,4,to%20inject%20Spring%20beans%20into%20Non%2DSpring%20objects.%20See%20this%20answer%3A,-https%3A//stackoverflow.com)

