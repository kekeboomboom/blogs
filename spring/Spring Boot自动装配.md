文章挺长，表达不好，希望能有获~~~~

Spring也提供使用注解来注册bean，为什么要用SpringBoot呢？

使用Spring应用，比如SpringMVC还行需要配置ViewResolver、DispatcherServlet，使用Mybatis等也需要进行其他配置。

如下为spring-mybatis配置文件：

```xml

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	   xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.2.xsd
   http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.2.xsd">
 
	<!-- 导入属性配置文件 -->
	<context:property-placeholder location="classpath:jdbc.properties" />
 
	<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
		<property name="driverClassName" value="${jdbc.driverClassName}" />
		<property name="url" value="${jdbc.url}" />
	</bean>
	<!-- 将数据源映射到sqlSessionFactory中 -->
	<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
		<property name="configLocation" value="classpath:mybatis/mybatis-config.xml" />
		<property name="dataSource" ref="dataSource" />
		<!--<property name="mapperLocations" value="classpath:mybatis/mapper/*.xml" />-->
	</bean>
 
	<!-- SqlSession模板类实例 -->
	<bean id="sessionTemplate" class="org.mybatis.spring.SqlSessionTemplate" destroy-method="close">
		<constructor-arg index="0" ref="sqlSessionFactory" />
	</bean>
 
	<!--======= 事务配置 Begin ================= -->
	<!-- 事务管理器（由Spring管理MyBatis的事务） -->
	<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<!-- 关联数据源 -->
		<property name="dataSource" ref="dataSource"></property>
	</bean>
	<!--======= 事务配置 End =================== -->
	
	<!--mapper配置-->
	<bean id="kbCityMapper" class="org.mybatis.spring.mapper.MapperFactoryBean">
		<property name="mapperInterface" value="com.spring.dao.KbCityMapper" />
		<property name="sqlSessionFactory" ref="sqlSessionFactory" />
	</bean>
</beans>
```

SpringBoot进行自动配置不在需用使用者进行这些配置。而且使用SpringBoot也不需要管理Mybatis、log4j、jackson等依赖包的版本问题，SpringBoot使用starter进行依赖管理。

使用SpringBoot只需在`application.yml`或`application.xml`添加少量配置，通过`@SpringBootApplication`即可启动应用。

## 我们从spring文档看一看

自动装配的意义在于SpringBoot提前为我们初始化了一些东西，从SpringBoot文档中搜索`Auto-configuration`

对于JSON mapping libraries：

![image-20210425201411731](https://gitee.com/keke518/MarkDownPicture/raw/master/20210425201411.png)

对于Spring MVC：

![image-20210425201444384](https://gitee.com/keke518/MarkDownPicture/raw/master/20210425201444.png)

可以看到有对`BeanNameViewResolver`的自动配置。

对于模板引擎：

![image-20210425201709479](https://gitee.com/keke518/MarkDownPicture/raw/master/20210425201709.png)

可以看到对`Thymeleaf`的自动配置，我们只需要引入Thymeleaf依赖就能直接用了，不需要任何配置。

对Reids：

![image-20210425202428406](https://gitee.com/keke518/MarkDownPicture/raw/master/20210425202428.png)

还有对Security，Spring Data JPA，Elasticsearch等等很多很多，SpringBoot都对他们提前进行了一些配置，这样使得我们只需引入其依赖，只需要在`application.yml`少量配置甚至不需要配置就可以直接使用。



------

## 我们从jar包的源码看一看

![image-20210425205723641](https://gitee.com/keke518/MarkDownPicture/raw/master/20210426230312.png)

文档说让我们浏览`spring-boot-autoconfigure`的源代码，那么我们就看看源码：

![image-20210425205939115](https://gitee.com/keke518/MarkDownPicture/raw/master/20210425205956.png)

先说一下META-INF包下的，我们看一看`additional-spring-configuration-metadata.json`这个文件里有啥呢？

我们看一看其中`server.port`:

![image-20210425210933833](https://gitee.com/keke518/MarkDownPicture/raw/master/20210425210933.png)

嗯，他默认值是8080，是不是很熟悉呢？我们进入我们项目的`application.properties`

![image-20210425211045190](https://gitee.com/keke518/MarkDownPicture/raw/master/20210425211045.png)

![image-20210425211121166](https://gitee.com/keke518/MarkDownPicture/raw/master/20210425211121.png)

那么我们再看看`server.servlet.encoding.enabled`，`server.servlet.jsp.class-name`这两个

![image-20210425211338516](https://gitee.com/keke518/MarkDownPicture/raw/master/20210425211338.png)

![image-20210425211418156](https://gitee.com/keke518/MarkDownPicture/raw/master/20210425211418.png)

这回我们知道原来`application.properties`中的默认值是从`additional-spring-configuration-metadata.json`这个文件来的呀。



接下来看看`spring.factories`文件：

![image-20210425212431574](https://gitee.com/keke518/MarkDownPicture/raw/master/20210425212431.png)

里面都是一些`AutoConfiguration`类的完整类路径，那么我们有了类路径能干啥呢？那当然通过反射创建该类了。那么有这么多自动配置类肯定不会全部加载，那些我们要用到SpringBoot就加载哪些。

`spring-autoconfigure-metadata.properties`：看其内容，等号左边为类路径或属性，右边或者类路径或者值或者空。应该跟我们平常的properties文件一样的意思吧，为了不将这些配置属性在代码中写死，将其提取出来放到properties文件中。

`spring-autoconfigure-metadata.json`：只知道name与sourceType，type，sourceMethod建立映射关系，不知道干啥用。或许为了通过name标识值在代码中更清晰易懂吧。



继续向下看：

![image-20210425220234468](https://gitee.com/keke518/MarkDownPicture/raw/master/20210425220234.png)

这里面可以看到很多熟悉的名字，我们以web包为例看一看吧：



![image-20210425220439111](https://gitee.com/keke518/MarkDownPicture/raw/master/20210425220439.png)

这些类大致分为两种，一种`AutoConfiguration`，一种`Properties`。

题外话：我们知道SpringBoot扫描静态资源时会在/resources/, /static/, /public/这些路径下。我们看看`WebProperties.java`

![image-20210425221258845](https://gitee.com/keke518/MarkDownPicture/raw/master/20210425221258.png)

接着我们以`WebMvcProperties`和`WebMvcAutoConfiguration`为例看一看吧。

`WebMvcProperties`：就在这个属性类中完成了webMvc相关属性的初始化工作。

某些属性如下：

```java
	private String staticPathPattern = "/**";

	public static class Servlet {

		/**
		 * Path of the dispatcher servlet. Setting a custom value for this property is not
		 * compatible with the PathPatternParser matching strategy.
		 */
		private String path = "/";
    
		public String getServletMapping() {
			if (this.path.equals("") || this.path.equals("/")) {
				return "/";
			}
			if (this.path.endsWith("/")) {
				return this.path + "*";
			}
			return this.path + "/*";
		}
	}
```



`WebMvcAutoConfiguration`：这个类就通过get方法，从上面的properties文件中取值，有了这些属性值，那么就能完成webMvc相关类的初始化工作了！！！





## 接下来从springbootApplication注解，进入来看看吧

![image-20210426204624517](https://gitee.com/keke518/MarkDownPicture/raw/master/20210426204630.png)

`@SpringBootConfiguration`：进去看后是一个`@Configuration`，因此SpringBootConfiguration注解的作用是将其注解的类注册到spring中。

`@ComponentScan`：组件扫描类，在此包下的被controller，service，component等注解类注册到spring中。

`@EnableAutoConfiguration`：以下主要分析。

![image-20210426205040918](https://gitee.com/keke518/MarkDownPicture/raw/master/20210426205040.png)

查看`@AutoConfigurationPackage`：

![image-20210426205147178](https://gitee.com/keke518/MarkDownPicture/raw/master/20210426205147.png)

进入`AutoConfigurationPackages.Registrar.class`：

![image-20210426205320143](https://gitee.com/keke518/MarkDownPicture/raw/master/20210426205320.png)

debug一下`registerBeanDefinitions`方法：![login](https://gitee.com/keke518/MarkDownPicture/raw/master/20210426205715.gif)

通过上面的gif我们可以看到`registerBeanDefinitions`这个方法应该就是注册我们自己包下所有bean。



接着进入`AutoConfigurationImportSelector.class`：

进入`getAutoConfigurationEntry`->`getCandidateConfigurations`->`loadFactoryNames`->`loadSpringFactories`

![image-20210426215456442](https://gitee.com/keke518/MarkDownPicture/raw/master/20210426215456.png)

这里会获取到spring.factories文件，然后加载他，然后循环遍历将其放入result中。

![image-20210426215716660](https://gitee.com/keke518/MarkDownPicture/raw/master/20210426215716.png)

我们debug进入时：

![image-20210426221005559](https://gitee.com/keke518/MarkDownPicture/raw/master/20210426221005.png)

result为一个map，我们以`org.springframework.boot.autoconfigure.EnableAutoConfiguration`为key，取出那size为130的`List<String>`。

![image-20210426221252130](https://gitee.com/keke518/MarkDownPicture/raw/master/20210426221252.png)

现在我们从`getCandidateConfigurations`出来了，`configuration`的size大小130，然后去重，排除。

![image-20210426221723546](https://gitee.com/keke518/MarkDownPicture/raw/master/20210426221723.png)

在执行`getConfigurationClassFilter().filter(configurations);`方法前`configurations`的size还为130。

执行之后：

![image-20210426221756839](https://gitee.com/keke518/MarkDownPicture/raw/master/20210426221756.png)

最终我们加载了这23个配置类。

![image-20210426222136717](https://gitee.com/keke518/MarkDownPicture/raw/master/20210426222136.png)





## 如何创建自己的starter？

![image-20210426222710238](https://gitee.com/keke518/MarkDownPicture/raw/master/20210426222710.png)

[demo project](https://github.com/snicoll/spring-boot-master-auto-configuration)我看了，六年前更新的，使用的springboot1，很老的版本了。这个示例的pom的依赖引入的很繁杂，可以看看人家写的自动配置类，挺齐全的。

![image-20210426225436189](https://gitee.com/keke518/MarkDownPicture/raw/master/20210426225436.png)

三个点：

- pom文件引入springboot依赖（我看guide哥的博客，直接引入这个依赖就能用了。）

```xml
<dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter</artifactId>
      <version>2.4.5</version>
      <scope>compile</scope>
</dependency>
```



- 创建自动配置类，其上有必须`@Configuration`，其他约束性注解根据情况添加
- 将这个自动配置类写到，spring.factories文件



终于完结。

:smile: