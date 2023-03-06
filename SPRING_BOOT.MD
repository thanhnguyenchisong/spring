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
