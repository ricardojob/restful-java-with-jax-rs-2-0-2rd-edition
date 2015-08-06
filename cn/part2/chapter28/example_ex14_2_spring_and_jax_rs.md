# Example ex14_2: Spring and JAX-RS


There isn’t much difference between the code of *ex14_1* and *ex14_2*. The Java classes are basically the same, except all the EJB **@Stateless** annotations were removed from the JAX-RS resource classes because the example is using Spring instead of EJB for its component model.


Besides the removal of EJB metadata, the differences between the two projects are mainly packaging and configuration. If you look through the *ex14_2* directory, you’ll see that we’re back to using embedded Jetty. The *web.xml* file is a tiny bit different than the EJB example, so let’s take a look at that first:


```xml:src/main/webapp/WEB-INF/web.xml
<web-app>
    <context-param>
        <param-name>spring-beans-file</param-name>
        <param-value>META-INF/applicationContext.xml</param-value>
    </context-param>
</web-app>
```


This example follows the Spring integration conventions discussed in [Chapter 14](../../part1/chapter14/deployment_and_integration.md). The *web.xml* file adds a **&lt;context-param&gt;** to point to the Spring XML file that holds all of the example’s Spring configuration. Let’s look at this Spring XML file:



```xml:src/main/resources/applicationContext.xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx.xsd"
       default-autowire="byName">

    <bean id="entityManagerFactory"
         class="org.springframework.orm.jpa.
                         LocalContainerEntityManagerFactoryBean">
        <property name="jpaVendorAdapter">
            <bean class="org.springframework.orm.jpa.vendor
                                     .HibernateJpaVendorAdapter">
                <property name="showSql" value="false"/>
                <property name="generateDdl" value="true"/>
                <property name="databasePlatform"
                          value="org.hibernate.dialect.HSQLDialect"/>
            </bean>
        </property>
    </bean>

    <bean id="dataSource"
          class="org.apache.commons.dbcp.BasicDataSource"
          destroy-method="close">
        <property name="driverClassName"
                  value="org.hsqldb.jdbcDriver"/>
        <property name="url" value="jdbc:hsqldb:test/db/myDB"/>
        <property name="username" value="sa"/>
        <property name="password" value=""/>
    </bean>

    <bean id="transactionManager"
          class="org.springframework.orm.jpa.JpaTransactionManager"/>

    <tx:annotation-driven/>
```


The first part of the Spring configuration file is the configuration required to get JPA and Spring to work together. While the package structure for the Spring example is simpler than the EJB one, you can see that the configuration is a bit more complex.


```xml
    <bean class="org.springframework.orm
             .jpa.support.PersistenceAnnotationBeanPostProcessor"/>

    <bean id="customer" class="com.restfully.shop.services
                                            .CustomerResourceBean"/>
    <bean id="product" class="com.restfully.shop.services
                                            .ProductResourceBean"/>
    <bean id="order" class="com.restfully.shop.services
                                            .OrderResourceBean"/>
    <bean id="store" class="com.restfully.shop.services
                                            .StoreResourceBean"/>
</beans>
```


The rest of the Spring XML file defines all of the JAX-RS resource beans.


The Spring XML file is loaded and registered with the JAX-RS runtime by the **ShoppingApplication** class:


```Java:src/main/java/com/restfully/shop/services/ShoppingApplication.java
@ApplicationPath("/services")
public class ShoppingApplication extends Application
{
   private Set<Class<?>> classes = new HashSet<Class<?>>();

   public ShoppingApplication()
   {
      classes.add(EntityNotFoundExceptionMapper.class);
   }

   public Set<Class<?>> getClasses()
   {
      return classes;
   }

   protected ApplicationContext springContext;

   @Context
   protected ServletContext servletContext;

   public Set<Object> getSingletons()
   {
      try
      {
         InitialContext ctx = new InitialContext();
         String xmlFile =
             (String)servletContext.getInitParameter("spring-beans-file");
         springContext = new ClassPathXmlApplicationContext(xmlFile);
      }
      catch (Exception ex)
      {
         ex.printStackTrace();
         throw new RuntimeException(ex);
      }
      HashSet<Object> set = new HashSet();
      set.add(springContext.getBean("customer"));
      set.add(springContext.getBean("order"));
      set.add(springContext.getBean("product"));
      set.add(springContext.getBean("store"));
      return set;
   }

}
```


The **getSingletons()** method is responsible for initializing Spring and registering any JAX-RS resource beans created by Spring with the JAX-RS runtime. It first looks up the name of the Spring XML configuration file. The filename is stored in a servlet context’s init parameter named **spring-beans-file**. The **getSingletons()** method looks up this init parameter via the injected **ServletContext**.


After **getSingletons()** gets the name of the config file, it then initializes a Spring **ApplicationContext** from it. Finally, it looks up each JAX-RS bean within the project and registers it with the JAX-RS runtime.



### Build and Run the Example Program


Perform the following steps:

1. Open a command prompt or shell terminal and change to the *ex14_2* directory of the workbook example code.
2. Make sure your PATH is set up to include both the JDK and Maven, as described in [Chapter 17](../chapter17/workbook_introduction.md).
3. Perform the build and run the example by typing **maven install**.