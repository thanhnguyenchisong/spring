idref : reference base in bean id, it just pass name of bean
```xml
<bean id="theTargetBean" class="..."/>

<bean id="theClientBean" class="...">
    <property name="targetName">
        <idref bean="theTargetBean"/>
    </property>
</bean>
```
```xml
<bean id="theTargetBean" class="..." />

<bean id="client" class="...">
    <property name="targetName" value="theTargetBean"/>
</bean>
```


use ref to ``refer`` to other beans - It pass the bean object
We can define a inner bean in ``property`` element, inner bean is anomymous bean so we don't need id and scope
```xml
<bean id="outer" class="...">
    <property name="target">
        <bean class="com.example.Person">
         ...
        </bean>
    </property>
</bean>
```

**Collections**
```xml
<bean id="moreComplexObject" class="example.ComplexObject">
    <!-- results in a setAdminEmails(java.util.Properties) call -->
    <property name="adminEmails">
        <props>
            <prop key="administrator">administrator@example.org</prop>
            <prop key="support">support@example.org</prop>
            <prop key="development">development@example.org</prop>
        </props>
    </property>
    <!-- results in a setSomeList(java.util.List) call -->
    <property name="someList">
        <list>
            <value>a list element followed by a reference</value>
            <ref bean="myDataSource" />
        </list>
    </property>
    <!-- results in a setSomeMap(java.util.Map) call -->
    <property name="someMap">
        <map>
            <entry key="an entry" value="just some string"/>
            <entry key="a ref" value-ref="myDataSource"/>
        </map>
    </property>
    <!-- results in a setSomeSet(java.util.Set) call -->
    <property name="someSet">
        <set>
            <value>just some string</value>
            <ref bean="myDataSource" />
        </set>
    </property>
</bean>
```

**Collection merging**
Bean can define the parent and child so we can merge the collection, ``merge`` attribute must be specified on the lower - child defination.

Null and empty by ``value = "" and <null/>

``p-namespace`` from ``xmlns:p="http://www.springframework.org/schema/p"`` lets us use the beaan element's attributes instead of nested ``<peroperty>``
```xml
    <bean name="classic" class="com.example.ExampleBean">
        <property name="email" value="someone@somewhere.com"/>
    </bean>
    <bean name="p-namespace" class="com.example.ExampleBean" p:email="someone@somewhere.com"/>
```
``c-namespace`` lets inlined attributes for configuring the constructor anguments instead of ``constructor-arg`` elements

**depend-on** to explicitly force one or more beans to ben initialized before the bean using this. depen-on is a attribute of bean

**lazy-init** a attribute of ``bean`` if lazy-init = true then the bean will be create when it's first request rather than at startup
You can set ``default-lazy-init="true"`` attribute in ``<beans>`` elements to set all bean in beans is lazy initailizing

### 1.4.5 Autowiring Collaborators.
- Autowiring can reduce the nedd to specify properties or constructor argument and can update a configuration as your objecs evolve.

**Mode**
-  ``no`` autowiring, must use ref
- ``byName`` autowriting by peroperty name by loolking same name in bean
- ``byType`` autowrited if exactly one bean of property type
- ``constructor`` same as ``byType`` but applies to constructor arguments.

**Limitations and Disadvantages of Autowiring**
- Expicit dependencies in peroperty and constuctor-arg setting always override autowiring and can not autowire simple peoperties as primitives,  strings and Arrays, ...
- Less exact than explicit wiring, Spring managed objecs are no longer documented explicitly
-  May not available to tool that may generate documentation
-  Multiple bean definition within the container may match the type specific by setter method and contstructor argument to be autowired

**Excluding a Bean from Autowiring**
In Spring's XML format set the autowire-candidate attribute of the <bean> element to false so It bean is unavailable to the autowriting infrastructor or you can use ``default-autowire-candidates`` in ``<beans>``

1.4.6 Method Injection
The relation ship bw bean A and bean B express by B is property of A. But we meet a probleam when the bena lifecycles is different. A is singleton and B is non-singleton. So A never change into the container cannot provide bean with a new instance of bean B every time.

- Solution:  inject by method, fore go some ioc and call to the contaner  aks for bean B instance everytime bean A needs it.
- That mean aware by bussiness, not aware by IOC

**Replace method by other method with bean**
You can replace to other method create a class eimplement MethodReplacer interface. This class can be use in ``replacer`` can be refer as bean in
```xml
<bean id="myValueCalculator" class="x.y.z.MyValueCalculator">
    <replaced-method name="computeValue" replacer="replacementComputeValue">
        <arg-type>String</arg-type>
    </replaced-method>
</bean>
<bean id="replacementComputeValue" class="a.b.c.ReplacementComputeValue"/>
```

### 1.5 Bean Scopes
- singleton : Default - one instance to all inject/looked up
- prototype: a new object is created each time it is inject/looked up
- request: Lifecycle of single HTTP request
- session: lifecycle of a HTTP Session.
- application: life cycle of a ServletContext
- websocket: a lifecycal of a WebSocket.

Singleton-bean and Protype-bean dependencies are mentioned at Method Injection.

### 1.5.1 Request, Session, Application, and WebSocket Scopes
Just available only if you use a web-aware Spring ``ApplicationContext`` implementation(``XmlWebApplicationContext``). If use ``ClassPathXmlApplicationContext`` then application throw ``IllegalStateException``

Those are web-scopes beans. if you scoped beans within Spring Web MVC, in request that is processed by the Spring ``DispatcherServlet``, no special setup is necessary. ``DispatcherServlet`` exposes all relevant state.

if the requests processed outside of Spring's DispatcherServlet (when using JSF), you need to register the ``RequestContextListener , ServletRequestListener``, it can be done by using ``WebApplicationInitializer`` interface. 

```xml
<web-app>
    ...
    <listener>
        <listener-class>
            org.springframework.web.context.request.RequestContextListener
        </listener-class>
    </listener>
    ...
</web-app>
```

**Conlusion** : ``DispatcherServlet, RequestContextListener, and RequestContextFilter`` do exactly the samething, namely bind the HTTP request object to the Thread

**Request scope**
Use anotation ``@RequestScope`` or
```xml
<bean id="loginAction" class="com.something.LoginAction" scope="request"/>
```
when the request was finnished the bean is discarded
Use anotaion ``@SessionScope`` or
```xml
<bean id="userPreferences" class="com.something.UserPreferences" scope="session"/>
```
....same
``<aop:scoped-proxy/>``  is inject the dependencies without the probleam in scope. 
Example you have a singleton scope bean A and session scope bean B. With bean A depend on bean B. A always is a instance => bean B as a properties of bean A will also never change => to resovle we need ``<aop:scoped-proxy/>`` that will create the relationship A and B but still keep the the B always be updated when a need, it also know when the bean B should be down this is the way or you have use setter to control it.

**Custom scope**
You can implement ``org.springframework.beans.factory.config.Scope`` then override the method get, remove, registerDestructionCallback, getConversationId then can use ``registerScope`` from your IOC container (``BeanFactory``) to use your scope in your configuration metadata

**Customizing the Nature of a Bean**
Implement ``InitializingBean`` for void ``afterPropertiesSet()`` method and ``DisposableBean`` interfaces but it's not recommened Because spring provide  ``@PostConstruct `` or ``init-method`` attribute of bean and ``@PreDestroy`` or ``destroy-method`` attribute

When can custom by do some initialization work and same for destruction. 

So here we just custom by add some job in initialization and destruction bean.

**Conlusion**: We can combine it in a bean class with 

---- Have to read more about this part. DOn't understand

**Shutting down the spring IOC Container** :
```java
    public final class Boot {
    public static void main(final String[] args) throws Exception {
        ConfigurableApplicationContext ctx = new ClassPathXmlApplicationContext("beans.xml");
        // add a shutdown hook for the above context...
        ctx.registerShutdownHook();
        // app runs here...
        // main method exits, hook is called prior to the app shutting down...
    }
}
```
**ApplicationContextAware and BeanNameAware** : Get the bean configuration metadata from object.
We know how to getBean now we know how to get meatadata
-- Never use it before so don't understand deeper

1.7 Bean definition Inheritance
```xml
<bean id="inheritedTestBean" abstract="true"
        class="org.springframework.beans.TestBean">
    <property name="name" value="parent"/>
    <property name="age" value="1"/>
</bean>
<bean id="inheritsWithDifferentClass"
        class="org.springframework.beans.DerivedTestBean"
        parent="inheritedTestBean" init-method="initialize">  
    <property name="name" value="override"/>
    <!-- the age property value of 1 will be inherited from parent -->
</bean
```
Or you can create abstrac bean without specificly a class.

**Custom bean by BeanPostProcessor**
Can extend ``BeanPostProcessor`` then customer 2 methods of it ``postProcessBeforeInitialization`` and ``postProcessAfterInitialization`` by your customization. Then import it as a bean. It will work for your application.
--- Have to try it

*Configuration Metadata with a BeanFactoryPostProcessor but never use it, if you want to understand it. Have to try*

***PropertySourcesPlaceholderConfigurer*** : use it to read the properties in a separate file.
```xml
<bean class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer">
    <property name="locations" value="classpath:com/something/jdbc.properties"/>
</bean>

<bean id="dataSource" destroy-method="close"
        class="org.apache.commons.dbcp.BasicDataSource">
    <property name="driverClassName" value="${jdbc.driverClassName}"/>
    <property name="url" value="${jdbc.url}"/>
    <property name="username" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
</bean>
```
Can use the property in propertiy file to substutute class names.
```xml
<bean id="serviceStrategy" class="${custom.strategy.class}"/>
```
***PropertyOverrideConfigurer**: use for the case you would like to have default value if property file don't have this file.
The properties can be override and the last one override will be accept as value of properties.
It's not clear in config.

From Spring 2.5 they introduce a explicit override from context name-space
```xml
<context:property-override location="classpath:override.properties"/>
```
**FactoryBean**
Can implement FactoryBean and use three method getObject isSingleton and getObjectType.

### 1.9 Anotaion-based Container Configuration

- Anotaion better than than XML configuring Spring?
  "It depends". each of them have pros and cons.
  Anotation  provides a lot od context in their declaration leading to shorter and more concise configuration. However XML at wiring up component without touching their source code and recompliling them. Centralize the configuration in xml part so can keep the pure Java POJO object.

#### 1.9.1 @Autowired
You can apply this anotation for properties, constructor, setter method.
Also can instruct Spring to provide all beans of a particular typy by adding ``@Autowired`` annotation to inject component to a collection, Map
```java
 public class MovieRecommender {
    @Autowired
    private MovieCatalog[] movieCatalogs;
}
```

``@Order`` ordering of injected components to a collection

``@Autowired(required = false)`` that mean it is optional, the property will be ignored if it can't be autowired.
If we don't have @Autowired in class and have more than one constructor then default constructor will be used, if there is a constructor only, it will be used.
```java
public class SimpleMovieLister {
    @Autowired
    public void setMovieFinder(@Nullable MovieFinder movieFinder) {
        ...
    }
}
```
Can use @Nullable to set the Bean is optional as @Autowired(requited=false).
Can use it for interfaces that well-know resolvable dependencies. ``@Autowired
    private ApplicationContext context;``
#### 1.9.2 Fine-tuning Annotaion-based Autowiring with @Primary, @Qualifier
Autowiring by type may lead to multiple candidates. (we can create many beans with same type - class). In this case which have @Primary will be taken as value if exactly one primary bean exist among candidates.

Use ``@Primary`` in Java file or ``primary="true"`` attribute in xml file

If you would like to control more over the selection process (``@Primary`` make the selection process), you can use ``@Qualifier``. Use it with specific arguments or method parameters.
Example
```java
 public class MovieRecommender {

    private final MovieCatalog movieCatalog;

    private final CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public void prepare(@Qualifier("main") MovieCatalog movieCatalog,
            CustomerPreferenceDao customerPreferenceDao) {
        this.movieCatalog = movieCatalog;
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean class="example.SimpleMovieCatalog">
        <qualifier value="main"/> 

        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <qualifier value="action"/> 

        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean id="movieRecommender" class="example.MovieRecommender"/>

</beans>
```
The bean with ``qualifier  = main`` is wired with the constuctor argument that is qulified with same value ``main``

**Generics as Autowiring Qualifiers** to find the correct bean.
```java
 @Autowired
 private Store<String> s1;
```
```java
class A extend Store<String> {}
class B extend Store<Integer> {}
```
That  will take the object which have generictype is String this case that is A

**@Resource** inject by this.
You can use ``@Resource(name="myMovieFinder")`` , with name is bean name.
If there are no explicit name then the name is properties name or method name.
 
**Using @Value**
inject externalized properties.
Before use it, we need ``@PropertySource("classpath:application.properties")`` to know the external file path.

PropertySourcesPlaceholderConfigurer using JavaConfig, the @Bean method must be static.
```java
 @Configuration
public class AppConfig {
    @Bean
    public static PropertySourcesPlaceholderConfigurer propertyPlaceholderConfigurer() {
        return new PropertySourcesPlaceholderConfigurer();
    }
}
```
In spring boot it easier by deafult file for PropertySourcesPlaceholderConfigurer is ``application.properties`` and ``application.yml`` file.

 **Using @PostConstruct and @PreDestroy**
 Use it in Bean to inject your source code to construction process and deconstruction process

 #### 1.10. Classpath scanning and Managed Components.
@Component is generic type any Spring-managed component. @Repository, @Service, and @Controller are specializations of @Component for more specific use cases

#### 1.10.1 Meata annootations and Composed Annotations.
Compose anotation that is compose of 2 anotation example ``@RestController`` anoation from ``@Controller`` and ``@ResponseBody``

##### 1.10.2 Using Filters to Customize Scanning'
```java
@Configuration
@ComponentScan(basePackages = "org.example",
        includeFilters = @Filter(type = FilterType.REGEX, pattern = ".*Stub.*Repository"),
        excludeFilters = @Filter(Repository.class))
public class AppConfig {
    // ...
}
```
##### Bean in component
You can define Bean meatadata in Component by @Bean
```java
@Component
public class FactoryMethodComponent {
    @Bean
    @Qualifier("public")
    public TestBean publicInstance() {
        return new TestBean("publicInstance");
    }
    public void doWork() {
        // Component method implementation omitted
    }
}
```
Also provide the Laza, scope, qualifier as xml metadata
The bean will available when the component class was created. So you can you static for Bean method to 
-- never use like that, try it to understand

##### 1.10.6 Naming Autodetected Components
Each component have a specific name. ``@Service("myMovieLister")``
##### 1.10.7. Providing a Scope, qualifier for Autodetected Components
The components also be managed as bean so we can add scope, 

##### 1.10.8 Generating an Index of Candidate Components
Improve the startup performance of large applications by creating a static list of candidates at compilation time, need to import
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context-indexer</artifactId>
        <version>6.0.6</version>
        <optional>true</optional>
    </dependency>
</dependencies>

```
#### 1.11 Using JSR 330 
```xml
<dependency>
    <groupId>jakarta.inject</groupId>
    <artifactId>jakarta.inject-api</artifactId>
    <version>1</version>
</dependency>
```
Instead of ``@Autowired`` you can use ``@Inject``
Instead of ``@Qualifier`` you can use ``@Name`` or ``@Qualifier``
Also support for ``@Nullable`` in parameters.
Instead of ``@Component`` you can use ``@Name("name")`` or ``@ManagedBean("name")``

There is no ``@Value`` and ``@Lazy`` features.

### 1.12 Java-based Container Configuration
- Start with you have to create the ApplicationContext, which will be manage your bean. What type you would like to you. In the case you would like to use anotation config only can be create by ``new AnnotationConfigApplicationContext(Dependency1.class, Dependency2.class);`` or building your context by ``context.register(classes)`` || Not recomment should use step 2

- Enableing Component Scanning with ``@ComponentScan(basePackages = "com.acme")`` or use ``context.scan("com.acme")`` in main method. You can without define the class by manually with step 1

- Support for Web Applications with AnnotationConfigWebApplicationContext by add to ``web.xml`` file
  ```xml
     <context-param>
        <param-name>contextClass</param-name>
        <param-value>
            org.springframework.web.context.support.AnnotationConfigWebApplicationContext
        </param-value>
    </context-param>
  ```
  Also can use ``<import/>`` as ``@Import`` the class A is imported in Class B then we need to add class B to be managed by ctx then all Bean in class A also is managed by context

  #### Combining Java and XML config
  If you would like use both then uses AnnotationConfigApplicationContext and the ``@ImportResource`` anotation to import XML as needed.
  ```java
    @Configuration
    @ImportResource("classpath:/com/acme/properties-config.xml")
    public class AppConfig {}
  ```
  ##### 1.13 Environment Abstraction
  Can use Environment to read the properties configuration

#### 1.13.1 Bean Definition Profiles
We would like to deploy in some different enviroments, each of them have different properties files.

##### Using @Profile
With @Profile anotation we say the correct properties file you would to get with your enviroment. 
```java
@Configuration
@Profile("production")
public class JndiDataConfig {

    @Bean(destroyMethod = "") 
    public DataSource dataSource() throws Exception {
        Context ctx = new InitialContext();
        return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
    }
}
```
By XML 
```xml
    <beans profile="development">
        <jdbc:embedded-database id="dataSource">
            <jdbc:script location="classpath:com/bank/config/sql/schema.sql"/>
            <jdbc:script location="classpath:com/bank/config/sql/test-data.sql"/>
        </jdbc:embedded-database>
    </beans>

    <beans profile="production">
        <jee:jndi-lookup id="dataSource" jndi-name="java:comp/env/jdbc/datasource"/>
    </beans>
```

##### Activating a Profile
Set in context ``ctx.getEnvironment().setActiveProfiles("development", ...);``  should config your string in  properties file.

#### Default profile
``@Profile("default")``

#### Others
User ``@PropertySource`` to set the properties file path.
You can you this way as Placeholder Resolution in Statements
```xml
<beans>
    <import resource="com/bank/service/${customer}-config.xml"/>
</beans>
```
### 1.14 Registering a LoadTimeWeaver
 Never use it before - have to ask

### 1.15 More of ApplicationContext
**Message Source** : Provide i18n functionality.
