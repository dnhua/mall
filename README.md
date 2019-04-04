# 一个简易的商城项目

## 一、项目初始化

### 1.1 数据库初始化

导入mmall.sql即可

### 1.2 初始化项目并推送到GitHub

初始化项目 <br>
初始化git <br>
推送 <br> 

1. 在github上新建一个项目mall <br>
2. 在idea中创建README.md和.gitignore文件 <br>
3. git init <br>
4. git status --查看 <br>
5. git commit -am 'first commit' --提交到本地仓库 <br>
6. git remote add origin ******(地址) <br>
7. git push -u origin master <br>
8. 第七步可能有问题，需要生成密钥：ssh-keygen -t rsa -C "*****@outlook.com" <br>
9. 创建分支：git checkout -b v1.0 origin/master     git push origin HEAD -u

### 1.3 创建pom文件

### 1.4 项目包结构初始化

### mybatis-generator配置

1. 将插件配置到pom文件中
![icon](https://raw.githubusercontent.com/dnhua/mall/v1.0/readmesrc/1.mybatis-pom.jpg)
2. generatorConfig.xml配置，datasource.properties配置
3. 双击右侧maven->mybatis-generator:generate生成pojo等
4. 将map里面的的更新时间改为now()（mysql的内置函数，这样就不用我们自己编程更新时间了）
5. 配置mybatis-plugin插件，此插件是为了方便开发用的。在dao和xml中跳转很方便
6. mybatis-pagehelper插件配置

### spring配置

1. 官方demo指引及配置<br>
    projects.spring.io/spring-framework
    <br>github.com/spring-projects/spring-petclinic
    <br>github.com/spring-projects/greenhouse
    <br>github.com/spring-projects/spring-boot
2. web.xml配置
   <br>2.1 过滤器配置
   ```xml
    <filter>
        <filter-name>characterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
        <init-param>
            <param-name>forceEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
    ```
   <br>2.2 RequestContestListener web容器启动和关闭的监听器
   ```xml
    <listener>
        <listener-class>org.springframework.web.context.request.RequestContextListener</listener-class>
    </listener>
    ```
   <br>2.3 ContextLoaderListener将web容器和spring容器整合的一个监听器
   ```xml
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    ```
    <br>2.4 applicationContext.xml配置，通过ContexLoaderListener加载
    ```xml
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>
            classpath:applicationContext.xml
        </param-value>
    </context-param>
    ```
    <br>2.5 spring mvc的配置，拦截*.do的请求。
    ```xml
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>*.do</url-pattern>
    </servlet-mapping>
    ```
 3. applicationContext.xml配置
    <br>配置包扫描
    <br>配置aop
    <br>分离spring容器
    ```xml
    <context:component-scan base-package="com.mmall" annotation-config="true"/>
    <!--<context:annotation-config/>-->
    <aop:aspectj-autoproxy/>
    <import resource="applicationContext-datasource.xml"/>
    ```
4. applicationContext-datasource.xml配置
    1. 包扫描
        ```xml
        <context:component-scan base-package="com.mmall" annotation-config="true"/>
        ```
    2. 常量的扫描
        ```xml
        <bean id="propertyConfigurer"
              class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
            <property name="order" value="2"/>
            <property name="ignoreUnresolvablePlaceholders" value="true"/>
            <property name="locations">
                <list>
                    <value>classpath:datasource.properties</value>
                </list>
            </property>
            <property name="fileEncoding" value="utf-8"/>
        </bean>
        ```
    3. db连接池
    4. sqlSessionFactory配置
        ```xml
        <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
            <property name="dataSource" ref="dataSource"/>
            <property name="mapperLocations" value="classpath*:mappers/*Mapper.xml"></property>
    
            <!-- 分页插件 -->
            <property name="plugins">
                <array>
                    <bean class="com.github.pagehelper.PageHelper">
                        <property name="properties">
                            <value>
                                dialect=mysql
                            </value>
                        </property>
                    </bean>
                </array>
            </property>
    
        </bean>
        ```
    5. dao层的扫描
        ```xml
        <bean name="mapperScannerConfigurer" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
            <property name="basePackage" value="com.mmall.dao"/>
        </bean>
        ```
    6. 事务管理的配置
        ```xml
        <!-- 使用@Transactional进行声明式事务管理需要声明下面这行 -->
        <tx:annotation-driven transaction-manager="transactionManager" proxy-target-class="true" />
        <!-- 事务管理 -->
        <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
            <property name="dataSource" ref="dataSource"/>
            <property name="rollbackOnCommitFailure" value="true"/>
        </bean>
        ```
5. spring-mvc的配置，dispatcher-servlet.xml
    1. 包扫描
        ```xml
        <context:component-scan base-package="com.mmall" annotation-config="true"/>
        ```
    2. 编码设置等
        ```xml
            <mvc:annotation-driven>
                <mvc:message-converters>
                    <bean class="org.springframework.http.converter.StringHttpMessageConverter">
                        <property name="supportedMediaTypes">
                            <list>
                                <value>text/plain;charset=UTF-8</value>
                                <value>text/html;charset=UTF-8</value>
                            </list>
                        </property>
                    </bean>
                    <bean class="org.springframework.http.converter.json.MappingJacksonHttpMessageConverter">
                        <property name="supportedMediaTypes">
                            <list>
                                <value>application/json;charset=UTF-8</value>
                            </list>
                        </property>
                    </bean>
                </mvc:message-converters>
            </mvc:annotation-driven>
        ```
    3. 文件上传的配置
    ```xml
        <!-- 文件上传 -->
        <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
            <property name="maxUploadSize" value="10485760"/> <!-- 10m -->
            <property name="maxInMemorySize" value="4096" />
            <property name="defaultEncoding" value="UTF-8"></property>
        </bean>
        ```
    