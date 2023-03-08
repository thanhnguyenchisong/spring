# RESOURCE
## 1. Introduction
Java provide URL but it's not enough or all access low-level resource so we spring provides ``Resource`` interface.
## 2. Resource Interface
More ablebility to access to low-level resource.
```java
public interface Resource extends InputStreamSource {
    boolean exists();
    boolean isReadable();
    boolean isOpen();
    boolean isFile();
    URL getURL() throws IOException;
    URI getURI() throws IOException;
    File getFile() throws IOException;
    ReadableByteChannel readableChannel() throws IOException;
    long contentLength() throws IOException;
    long lastModified() throws IOException;
    Resource createRelative(String relativePath) throws IOException;
    String getFilename();
    String getDescription();
}
```
## 3. Resource implementations
- UrlResource: UrlResource wraps a java.net.URL and can be used to access any object that is normally accessible with a URL such as files, HTTPS, FTP
- ClassPathResource: Resource implementation supports resolution as a java.io.File if the class path resource resides in the file system, not in jar file.
- FileSystemResource: This is a Resource implementation for java.io.File handles
- PathResource: This is a Resource implementation for java.nio.file.Path handles,
- ServletContextResource : is a Resource implementation for ServletContext resources, support straem access URL access but allow ``File `` access only the web-app archive is expanded and resource is physiscally on system
- InputStreamResource: is a ``Resource`` implementation for given ``InputStream`` . Should be used only if no specific ``Resource implementation is applicable.
- ByteArrayResource: is a Resource implementation for a given byte array, it create ``ByteArrayInputStream`` from given byte array

## 4. ResourceLoader interface
That can return (that is, load) Resource instances
```java
public interface ResourceLoader {
    Resource getResource(String location);
    ClassLoader getClassLoader();
}
```
``ResourceLoader``:  ``ClassPathXmlApplicationContext`` return ``ClassPathResource``, ``FileSystemXmlApplicationContext`` return ``FileSystemResource``, ``WebApplicationContext`` return ``ServletContextResource``

**all application contexts in Spring implement ``ResourceLoader``**

Prefix: ``classpath: , file: , https:, none``

## 5. ResourcePatternResolver interface
Extension to the ResourceLoader interface which defines a strategy for resolving a location pattern. 
Support the pattern: ``String CLASSPATH_ALL_URL_PREFIX = "classpath*:";`` and when you get resource ``Resource[] getResources(CLASSPATH_ALL_URL_PREFIX)``

## 6.ResourceLoaderAware interface
Provides method to get ``ResourceLoader``
The bean implement ``ResourceLoaderAware`` then your bean is notified of the ResourceLoader. So that mean it don't need go to by application context. It is auto inside of framework, can use ResourceLoader directly
# NOTE: Spring ``AWARE`` interface
Allow you to look into the inner workings of SpringFramework. you can access the Spring context, Spring bean life cycle events.
Example: Your Spring beans might require access th framework object such as ApplicationContext, BeanFactory, ResourceLoader.
## 7. Resources as dependencies.
It can become a property in bean
```xml
<bean id="myBean" class="example.MyBean">
    <property name="template" value="some/resource/path/myTemplate.txt"/>
</bean>
```
```java
public class MyBean {
    private Resource template;
    public setTemplate(Resource template) {
        this.template = template;
    }
}
```
```java
@Component
public class MyBean {
    private final Resource template;
    public MyBean(@Value("${template.path}") Resource template) {
        this.template = template;
    }
```
## 8. Application contexts and Resource paths.
As preceding parts, we know all of contexts are implemented ResourceLoader
We can create context base on your demand it can be 
- ``ClassPathXmlApplicationContext`` : bean definition loaded from classpath in your system
-  ``FileSystemXmlApplicationContext``: Load the file system resource.
### 8.1 Wildcards in Application Context Contructor Resource Paths
/WEB-INF/*-context.xml
com/mycompany/**/applicationContext.xml
file:C:/some/path/*-context.xml
classpath:com/mycompany/**/applicationContext.xml

classpath*:..
## 8.3 FileSystemResource Caveats
Relative paths and absolute paths.
``FileSystemApplicationContext`` forces all attached ``FileSystemResource`` instances to treat all location path s as relative. So don't use absolute paths, it desn't make sence

