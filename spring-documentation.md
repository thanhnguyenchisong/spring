# Spring core (Core Technologies)

## Chappter 1. The IOC contrainer
### 1. Introduction Spring IOC container and Beans
- IOC is Inversion of Control or we know it as   Dependencies Injection (DI), a process whereby object define their dependencies.

 Hence IOC name: The container injects those dependencies when it creates the bean.

 ``BeanFactory`` provides the configuration framework and basic functionality

 ``ApplicationContext`` is sub-interface of ``BeanFactory``, it adds more enterpricse specific functionality.

 ``Bean`` are objects that form to backbone of application and managed by Spring IOC container.

### 2. IOC Container overview
``ApplicationContext`` is responsible for instantiating, configuring, assembling the beans.

Standalone application is common to create an instance of ``ClassPathXmlApplicationContext`` or ``FileSystemXmlApplicationContext`` (traditional format) or can use java anotation to define the configuration metadata.

### 2.1 Configuration metadata
Configauration metadata is intruction to tell the Spring container to instaniate, configure and assembe the objects in your application.

We can use xml file or java anotation to make the configuration metadata

### 2.2 Instantiating a container
It is straightforward, the local path and paths is supperlied to ApplicationContex constructor are actually resource strings then AplicationContext con load the metadata from variety of external resources as local file system.

``ApplicationContext context = new ClassPathXmlApplicationContex(new String[] {"services.xml", "daos.xml"});``

The XML format:

```xml
<bean id="id of current bean" class="class path to your java class file"> 
<property name = "name of your property in your bean" ref="id of another bean" />
</bean>
```

You also can compose the XML-based configuration metadata by use import keyword

```xml
<beans>
    <import resource="services.xml"/>
    <import resource="resources/messageSource.xml"/>
    <import resource="/resources/themeSource.xml">
    <bean id="bean1" class="..."/>
    <bean id="bean2" class="..."/>
</beans>
```

### 3. Bean overview
Bean is ... from previous part.
Bean definition (information create a specific bean): class, name, scope, constuctor arguments, properties, autowiring mode, lazy-initialization mode, initialization method, destruction method

The is all configuration metatdata information which is used to create a bean.

### 3.1 Naming beans
Name of bean defined in ``id`` or ``name`` attributes, name conventions same as the java variable name convertion.

Sometime you can import the bean from many the common extenal resources so we have the case duplicate the bean name. So in this case we can use ``alias`` to specify the name of bean.
```xml
<alias name="subsystemA-dataSource" alias="subsystemB-dataSource"/>
<alias name="subsystemA-dataSource"
alias="myApp-dataSource" />
```
Tow bean name subsystemA-dataSource but have different alias name "myapp-datasource"

### 3.2 Instantiating Beans
The container looks for naming bean and configuration meatadata to create a bean.

- **Use the contructor to create a bean**:  With ``class`` attribute in ``bean`` element we can know the class and the bean is created by calling its contructor reflectively.
- **Use factory method**: In class we can define a static method to create this object so we can add it to conf meatadata by ``factory-method`` attribute after ``class`` atrribute in ``bean`` element, then the container will use this factory-method to create this object.

Netsed class names: use the binary name or source name of nested class. (``...class$nestedClass; ...class.nestedClasss``)

- In the case you would like to use both, we can use factory-bean
```xml
<!-- the factory bean, which contains a method called createInstance() -->
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<!-- the bean to be created via the factory bean -->
<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>
```
A factory class can hold more than one factory, can define many bean with same factory-bean but different factory-method.

To get bean runtime type you can use BeanFactory.getType with bean name and get BeanFactory.getBean with bean name to get this bean object.

## 4. Dependencies
All of thing related to the objects.
### 4.1 Dependency Injection.
4.1.1 Contructor-based 
 
 Using contructor injection.
```java
public class SimpleMovieLister {
    // the SimpleMovieLister has a dependency on a MovieFinder
    private final MovieFinder movieFinder;
    // a constructor so that the Spring container can inject a MovieFinder
    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```
Constructor Argument Resolution: you can use the constructor-arg  to config the bean definition for object.
```xml
<bean id="beanOne" class="x.y.ThingOne">
        <constructor-arg ref="beanTwo"/>
        <constructor-arg ref="beanThree"/>
</bean>
<bean id="beanTwo" class="x.y.ThingTwo"/>
<bean id="beanThree" class="x.y.ThingThree"/>
```
**Some ways for matching the argument with xml config file**
- Contructor angument type matching
```xml
 <bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg type="int" value="7500000"/>
    <constructor-arg type="java.lang.String" value="42"/>
</bean>
```
- Contructor argument index
```xml
 <bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg index="0" value="7500000"/>
    <constructor-arg index="1" value="42"/>
</bean>
```
- Constructor angumeent name
Replace ``index`` by name ``attribute``

Use ``@ConstructorProperties``  JDK anotation to explicitly contructor angurment name.

4.1.2 Use setter-based

Use pure setter injection by setter method.

**The rules** : contrcutor-based is used for mandatory dependencies and setter-based for optional dependencies. if we use ``@Autowrited`` anotation on setter method, it make the properties be a required dependency, but it is not good ways.

Conclution: The setter method should is pure setter method. The dependencies should be inject by manually with setter method as optional dependencies, and we change re-inject the new dependencies after creating the object by setter method.

**Circular dependencies** : A depend B and B depen A. IOC container throws a ``BeanCurrentlyInCreationException`` (not recommend this case)

Solution: use setter rather contructor

**Example** : the <property> is used to define the bean in setter method, name of properties is name of instance variale and container will inject the value by setter method.
```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- setter injection using the nested ref element -->
    <property name="beanOne">
        <ref bean="anotherExampleBean"/>
    </property>
    <!-- setter injection using the neater ref attribute -->
    <property name="beanTwo" ref="yetAnotherBean"/>
    <property name="integerProperty" value="1"/>
</bean>
<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```
The corresponding Java code
```java
public class ExampleBean {
    private AnotherBean beanOne;
    private YetAnotherBean beanTwo;
    private int i;
    public void setBeanOne(AnotherBean beanOne) {
        this.beanOne = beanOne;
    }
    public void setBeanTwo(YetAnotherBean beanTwo) {
        this.beanTwo = beanTwo;
    }
    public void setIntegerProperty(int i) {
        this.i = i;
    }
}
```
Like as the example we can use this way to config all bean by setter method data source jdbc connection with primitives, Strings and so on we can use value to set directly value to that properties.

**Config Properties instance as follow**
```xml
<bean id="mappings"
    class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer">
    <!-- typed as a java.util.Properties -->
    <property name="properties">
        <value>
            jdbc.driver.className=com.mysql.jdbc.Driver
            jdbc.url=jdbc:mysql://localhost:3306/mydb
        </value>
    </property>
</bean>
```
This is esier way to set the configuration metadata for bean.

### **idref element**








 
