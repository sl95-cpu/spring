### SSM 整合

#### 基本环境搭建

##### 1.新建 Maven 项目! ssmbuild,添加web的支持

##### 2.导入相关的pom(maven) 依赖

```xml
    <!--依赖问题
      junit  ,数据库驱动 连接池 ,servlet jsp
      mybatis  mybatis-spring spring
     -->
       <dependencies>
           <!--junit-->
           <dependency>
               <groupId>junit</groupId>
               <artifactId>junit</artifactId>
               <version>4.12</version>
           </dependency>
           <!--数据驱动-->
           <dependency>
               <groupId>mysql</groupId>
               <artifactId>mysql-connector-java</artifactId>
               <version>5.1.46</version>
           </dependency>
           <!--数据库连接池-->
           <dependency>
               <groupId>com.mchange</groupId>
               <artifactId>c3p0</artifactId>
               <version>0.9.5.2</version>
           </dependency>
           <!--Servlet jsp-->
           <dependency>
               <groupId>javax.servlet</groupId>
               <artifactId>servlet-api</artifactId>
               <version>2.5</version>
           </dependency>
           <dependency>
               <groupId>javax.servlet.jsp</groupId>
               <artifactId>jsp-api</artifactId>
               <version>2.2</version>
           </dependency>
           <dependency>
               <groupId>jstl</groupId>
               <artifactId>jstl</artifactId>
               <version>1.2</version>
           </dependency>
            <!--mybatis-->
           <dependency>
               <groupId>org.mybatis</groupId>
               <artifactId>mybatis</artifactId>
               <version>3.5.2</version>
           </dependency>
           <dependency>
               <groupId>org.mybatis</groupId>
               <artifactId>mybatis-spring</artifactId>
               <version>1.3.2</version>
           </dependency>

           <!--spring-->
           <dependency>
               <groupId>org.springframework</groupId>
               <artifactId>spring-webmvc</artifactId>
               <version>5.1.9.RELEASE</version>
           </dependency>
           <dependency>
               <groupId>org.springframework</groupId>
               <artifactId>spring-jdbc</artifactId>
               <version>5.1.9.RELEASE</version>
           </dependency>
       </dependencies>
```

##### 3.静态文件导出问题

```xml
<build>
        <resources>
            <resource>
                <directory>src/min/java</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>false</filtering>
            </resource>
        </resources>
    </build>
```

##### 4.建立基本结构和配置框架

com.sl.pojo

com.sl.dao

com.sl.service

com.sl.contrioller

###### mybatis配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0/EN "
        "http://mybatis.org/dtd/mybatis-3-config.dtd"
        >
<!--mybatis的主要配置文件-->
<configuration>
<!--mybatis的主要配置文件-->
<configuration>
<typeAliases>
    <package name="com.sl.pojo"/>
</typeAliases>
    <mappers>
        <mapper class="com.sl.dao.BookMapper"/>
    </mappers>
</configuration>
```

###### mapper 配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Config 3.0/EN "
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd"
        >
<!--mybatis的主要配置文件-->
<mapper namespace="com.sl.dao.BookMapper">
    <insert id="addBook" parameterType="books">
        insert into ssmbuild.books (bookName, bookCounts, detail)
        values (#{bookName},#{bookCounts},#{detail});
    </insert>
    <delete id="deleteBookById" parameterType="int">
        delete from ssmbuild.books where bookID=#{bookid};
    </delete>
    <update id="updateBook" parameterType="books">
        update ssmbuild.books
        set bookName=#{bookName},bookCounts=#{bookCounts},detail=#{detail}
        where bookID=#{bookID}
    </update>
    <select id="queryBookById" resultType="books" parameterType="int">
        select *from ssmbuild.books where bookID=#{bookid}
    </select>
    <select id="queryAllBook" resultType="books">
        select * from ssmbuild.books
    </select>
</mapper>
```



###### apploctiong主配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"    xsi:schemaLocation="http://www.springframework.org/schema/beans     https://www.springframework.org/schema/beans/spring-beans.xsd
   ">
 <import resource="classpath:spring-dao.xml"/>
    <import resource="classpath:spring-service.xml"/>
    <import resource="classpath:springmvc-servlet.xml"/>
</beans>
```

###### 数据库配置文件

如果使用的 时 mysql 8.0 + 要增加一个时区的配置 &serverTimezone=Asia/Shanghai

```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/ssmbuild?useSSL=true&useUnicode=true&characterEncoding=utf-8
jdbc.username=root
jdbc.password=1234
```

###### springdao 配置文件

```xml
    <!--1.关联数据库配置文件-->
    <context:property-placeholder location="classpath:database.properties"/>


    <!--连接池
    dbcp: 半自动化 手动连接
    cp30: 自动化操作 自动加载配置文件 ,并可以自动配置导对象中;
    deuid:
    hikart
    -->
    <!--2.连接池-->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="${jdbc.driver}"/>
        <property name="jdbcUrl" value="${jdbc.url}"/>
        <property name="user" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
     </bean>

    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"></property>
        <property name="configLocation" value="classpath:mybatis-config.xml"></property>
    </bean>


    <!--4.配置dao接口扫描包,动态实现了Dao杰奎可以注入到Spring容器当中去-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <!--注入 sqlSessionFactory-->
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
        <!--要扫描的到包-->
        <property name="basePackage" value="com.sl.dao"/>
    </bean>
```

###### c3p0 l连接池

```xml
    <!--连接池
    dbcp: 半自动化 手动连接
    cp30: 自动化操作 自动加载配置文件 ,并可以自动配置导对象中;
    deuid:
    hikart
    -->
    <!--1.连接池-->
    <bean id="dataSourcr" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="${jdbc.driver}"/>
        <property name="jdbcUrl" value="${jdbc.url}"/>
        <property name="user" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
        <!--c3p0连接池的私有属性-->
        <property name="maxPoolSize" value="30"/>
        <property name="minPoolSize" value="10"/>
        <!--关闭连接后不自动commit 连接-->
        <property name="autoCommitOnClose" value="false"/>
        <!--获取连接超时-->
        <property name="checkoutTimeout" value="10000"/>
        <!--获取连接失败重试次数-->
        <property name="acquireRetryAttempts" value="2"/>
     </bean>
```

###### web 配置文件

```xml
<!--DispatachServlet-->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!--关联一个springmvc的配置文件:[servlet].xml-->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:apploctioncontext.xml</param-value>
        </init-param>
        <!--启动级别1-->
        <load-on-startup>1</load-on-startup>
    </servlet>
    <!-- / 匹配所有请求 : 不包括(jsp)-->
    <!-- /* 匹配所有的请求 : 包括jsp-->
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
    <!--乱码过滤-->
    <!--配置SpringMVC的乱码过滤器 -->
    <filter>
        <filter-name>encoding</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>utf-8</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>encoding</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <!--Session-->
    <session-config>
        <session-timeout>15</session-timeout>
    </session-config>
```

###### spring-service 配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
         https://www.springframework.org/schema/context/spring-context.xsd">

    <!--1.扫描service 下的包-->
    <context:component-scan base-package="com.sl.services"/>

    <!--2.将我们的所有业务类注入到spring,可以通过配置-->
    <bean id="BookServiceImpl" class="com.sl.services.BookServiceImpl">
        <property name="bookMapper" ref="bookMapper"/>
    </bean>

    <!--3.声明式事务-->
    <bean id="transactionManage" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
          <!--注入数据源-->
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!--4.aop横事务支持-->
</beans>
```

###### spring-mvc

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
         https://www.springframework.org/schema/context/spring-context.xsd
         http://www.springframework.org/schema/mvc
         https://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <mvc:default-servlet-handler />
    <mvc:annotation-driven/>
    <!--自动扫描包,让指定包下的注解生效 ,用IOC容器管理 -->
    <context:component-scan base-package="com.sl.controller"/>
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="internalResourceViewResolver">
        <!--前缀-->
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <!--后缀-->
        <property name="suffix" value=".jsp"/>
    </bean>
</beans>
```

#### AJax



