# Validation, Data Binding, Type Conversion
## 1. Validation by Spring's Validation Interface
Can use ``Validator`` interface to validate objects. It works by using ``Errors`` object, so while validating, validators can report validation failures to ``Errors`` object.
```java
public class PersonValidator implements Validator {
    public boolean supports(Class clazz) {
        return Person.class.equals(clazz);
    }
    public void validate(Object obj, Errors e) {
        ValidationUtils.rejectIfEmpty(e, "name", "name.empty");
        Person p = (Person) obj;
        if (p.getAge() < 0) {
            e.rejectValue("age", "negativevalue");
        } else if (p.getAge() > 110) {
            e.rejectValue("age", "too.darn.old");
        }
    }
}
```
with the support from ``ValidationUtils``
## 2. Resolving code to Error Messages.
DefaultMessageCodesResolver is used as default.
## 3. BeanWrapper
``BeanWrapper`` interface  and corresponding implementation ``BeanWrapperImpl``.
offers functionality to set and get property values, determine if they are readable or writable, support nestd properties, add standard JavaBeans ``PropertyChangeListeners`` and ``VetoableChangeListeners`` , setting indexed properties. ``BeanWrapper`` not used in code derectly but used by DataBinder and BeanFactory.

### 4. Setting and Getting Basic and Nested properties.
through the ``setPropertyValue`` and ``getPropertyValue`` overloaded method veriants of ``BeanWrapper``
### 5. Built-in PropertyEditor Implementations
The custom implementation of ``PropertyEdiror`` can be support for convert to reable value from orginal value.
Try more in real application to see how to use

Add a custom Properies Editor.
Create by extend ``PropertyEditorSupport`` then register it in ``customPropertyEditor.registerCustomeEditors(WebDataBinder binder)``
### 6. Spring Typing Conversion
Spring provide ``Converter`` Interface to create converter class which can convert the value from a original value.

#### 6.1 Using ConverterFactory
If you would like centralize the conversion logic for a entire class hierachy, you can implement ``ConverterFactory`` interface.
Support a source to multiple target type which extend a difined parent.
```java
package org.springframework.core.convert.converter;
public interface ConverterFactory<S, R> {
    <T extends R> Converter<S, T> getConverter(Class<T> targetType);
}
```
All of T extend R can be get from S
#### 6.2 Using GenericConverter
Support multiple source to a target type.
```java
package org.springframework.core.convert.converter;
public interface GenericConverter {
    public Set<ConvertiblePair> getConvertibleTypes();
    Object convert(Object source, TypeDescriptor sourceType, TypeDescriptor targetType);
}
```
``TypeDescriptor`` provides access to field.

#### 6.3 Using ConditionalGenericConverter.
It is union of GenericConverter and ConditionalConverter. if the condition is true the convert action can works.
```java
public interface ConditionalGenericConverter extends GenericConverter, ConditionalConverter {
}
```
#### 6.4 The ConversionService API and config a ConversionService
 Define a unified API for executing type conversion logic at runtime. often run behind the following of facade interface
With xml
```xml
<bean id="conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean"/>
```
**Using a ConversionService**
```java
@Service
public class MyService {
    public MyService(ConversionService conversionService) {
        this.conversionService = conversionService;
    }
    public void doIt() {
        this.conversionService.convert(...)
    }
}
```
``DefaultConversionService`` automaticaly registers converters, that appropriate for most evn - collection converter, scalar converters, basic Object-to-String.

You can register the same converters with any ``ConverterRegistry`` by using the static ``addDefaultConverters`` method on ``DefaultConversionService`` . All of your converter will be run when we use the converter

#### 6.5 Spring field Formatting
- Formatter SPI - use ``Formatter`` interface of ``springframework.format`` to parse base on Locale (Date, language).
- Annotation-driven Formatting

```java
package org.springframework.format;

public interface AnnotationFormatterFactory<A extends Annotation> {
    Set<Class<?>> getFieldTypes();
    Printer<?> getPrinter(A annotation, Class<?> fieldType);
    Parser<?> getParser(A annotation, Class<?> fieldType);
}
```
Create a anotation and send it to Annotation FormatterFactory if the anotation be used in a field objecyt it will run the conversion method in ``AnnotationFormatterFactory``

- FormatterRegistry SPI - let you config configure formatteing rule centrally intead of dupplicating such configuration across your controllers.
- FormatterRegistrar SPI - for registering formatters and converter through the FormatterRegistry.
- Configuring a Global Date and Time Format, the ``DateTimeFormat`` don't know convert from strings by using the ``DateFormat.SHORT`` style so you can define your own global format.

Make sure spring not register the defaulr format, can register by your self by using
``org.springframework.format.datetime.standard.DateTimeFormatterRegistrar``
``org.springframework.format.datetime.DateFormatterRegistrar``
```java
@Configuration
public class AppConfig {
    @Bean
    public FormattingConversionService conversionService() {
        // Use the DefaultFormattingConversionService but do not register defaults
        DefaultFormattingConversionService conversionService =
            new DefaultFormattingConversionService(false);
        // Ensure @NumberFormat is still supported
        conversionService.addFormatterForFieldAnnotation(
            new NumberFormatAnnotationFormatterFactory());
        // Register JSR-310 date conversion with a specific global format
        DateTimeFormatterRegistrar dateTimeRegistrar = new DateTimeFormatterRegistrar();
        dateTimeRegistrar.setDateFormatter(DateTimeFormatter.ofPattern("yyyyMMdd"));
        dateTimeRegistrar.registerFormatters(conversionService);
        // Register date conversion with a specific global format
        DateFormatterRegistrar dateRegistrar = new DateFormatterRegistrar();
        dateRegistrar.setFormatter(new DateFormatter("yyyyMMdd"));
        dateRegistrar.registerFormatters(conversionService);

        return conversionService;
    }
}
```
we have xml conresponding:
```xml
<bean id="conversionService" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
... //config for DefaultFormattingConversionService and DateFormatterRegistrar
</bean>
```
### 6.7 Java Bean validation
if you don't use the Spring Validator, you can use Java Bean validation from JavaEE
by ``jakarta.validation.ValidatorFactory`` or ``jakarta.validation.Validator``

You can use ``LocalValidatorFactoryBean`` to config the default Validator as a Spring bean.

- Injecting a Validator ``@Autowired private Validator validator;`` from jakarta.validation.Validator or Spring
- Configuring custom constraints - ``@Constraint`` annotation references a ``ConstraintValidator`` implement class
```java
    @Target({ElementType.METHOD, ElementType.FIELD})
    @Retention(RetentionPolicy.RUNTIME)
    @Constraint(validatedBy=MyConstraintValidator.class)
    public @interface MyConstraint {}
```
```java
public class MyConstraintValidator implements ConstraintValidator {}
```
  - Spring-driven Method Validation - Use ``MethodValidationPostProcessor`` as default by pushing to Spring context with 
```java
@Configuration
public class AppConfig {
    @Bean
    public MethodValidationPostProcessor validationPostProcessor() {
        return new MethodValidationPostProcessor();
    }
}
```
or xml
```xml
<bean class="org.springframework.validation.beanvalidation.MethodValidationPostProcessor"/>
```
All of class use @Validator anotation of Bean Validation 1.1 or Hibernate Validator 4.3 that will use this MethodValidationPostProcessor.

### 6.8 Configuring a DataBinder
DataBinder is instance with a Validator, you can call binder.validate(). Error will added to BindingResult. So we can invoke the validation logic after binding to target object. // BUT i don't like this way


