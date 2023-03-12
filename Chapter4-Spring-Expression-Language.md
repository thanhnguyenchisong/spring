# 1. Evaluation
``ExpressionParser`` interface can be used, the impelementations:
- ``SpelExpressionParser`` 
```java
ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("'Hello World'.concat('!')"); 
String message = (String) exp.getValue();
```
```java
Inventor tesla = new Inventor("Nikola Tesla", c.getTime(), "Serbian");

ExpressionParser parser = new SpelExpressionParser();

Expression exp = parser.parseExpression("name"); // Parse name as an expression
String name = (String) exp.getValue(tesla);
```
## 1.1. EvaluationContext
EvaluationContext is used when evaluating an expression.
- ``SimpleEvaluationContext``: a subset of essential SpEL language feature and config options - Not include Java type references, constructors, bean references, requires you to explicitly choose the level of support of properties and method expressions (default read access to properties)
- ``StandardEvaluationContext``: full set of SpEL language feature and config options.

**Type Conversion**
``org.springframework.core.convert.ConversionService`` conversion service comes with may built-in converters for common conversion.
```java
EvaluationContext context = SimpleEvaluationContext.forReadOnlyDataBinding().build();

// "false" is passed in here as a String. SpEL and the conversion service
// will recognize that it needs to be a Boolean and convert it accordingly.
parser.parseExpression("booleanList[0]").setValue(context, simple, "false");

// b is false
Boolean b = simple.booleanList.get(0);
```

## 1.2 SpEL compilation
Spring Framework 4.1 Provides a lot of dynamic flexibility during evaluation but doesn't provide optimum performance.

SpEL compipler to resolve this problem, during the evaluation, compiler generates a Java CLass that embodies(hien than) the expression behavior at runtime. => So it faster

**Compiler Coonfiguration** : ``OFF``, ``IMMEDIATE``: expression are compiled as soon as possible, MIXED: expression silently switch bw interpreted and compiled mode over time.

**Compiper Limitations** since Spring Framework 4.1, not support compiling every kind of expression.
- assignment
- relying on the conversion service
- using custom resolvers or accessors.
- using selection or projection

# **Projection in spring** 
When use Spring JPA to implement persitence layer, sometime we don't need all the properties of returned objects. When projecting an entity, it relies on an interface, don't need to provide implementation.

Example can create a Projection by:
```java
public interface AddressView {
    String getZipCode();
}
```
```java
public interface AddressRepository extends Repository<Address, Long> {
    List<AddressView> getAddressByState(String state);
}
```
So by using ``AddressRepository.getAddressByState(state)`` for ``Address ``entity can return the value for  ``AddressView`` with a field ``zipCode`` only

Beside that we have open project with declare the field with custom input as example
```java
public interface PersonView {
    // ...
    @Value("#{target.firstName + ' ' + target.lastName}")
    String getFullName();
}
```
the ``fullName`` field is combined by real entity ``firstName`` and ``lastName`` fields.
- You also can use the projection class to build as DTO with some fields which are matched with entity.

So you can create many projections for a entity class.

# 2. Expression in bean definitions
- XML config: You can define the expression by ``#{ <expression string> }``. It can automaticly execute the expression in this form and assign it to bean property or args
```xml
<bean id="numberGuess" class="org.spring.samples.NumberGuess">
    <property name="randomNumber" value="#{ T(java.lang.Math).random() * 100.0 }"/>
    <!-- other properties -->
</bean>
<bean id="shapeGuess" class="org.spring.samples.ShapeGuess">
    <property name="initialShapeSeed" value="#{ numberGuess.randomNumber }"/>
    <!-- other properties -->
</bean>
```
- Anotation config: ``@Value("#{ systemProperties['user.region'] }")`` for properties or params or agurments.
```java 
public class FieldValueTestBean {
    @Value("#{ systemProperties['user.region'] }")
    private String defaultLocale;

    public void setDefaultLocale(String defaultLocale) {
        this.defaultLocale = defaultLocale;
    }
    public String getDefaultLocale() {
        return this.defaultLocale;
    }
}
```

# 3. Language Reference
Literal Expressions: ``parser.parseExpression("'Hello World'")``

Properties, Arrays, Lists, Maps, and Indexers : access as java code in string expression ``parser.parseExpression("birthdate.year + 1900")``

Inline Lists: ``parser.parseExpression("{1,2,3,4}")``

Inline Maps: ``parser.parseExpression("{{'a','b'},{'x','y'}}")``

Array Construction: ``parser.parseExpression("new int[4]")``

Methods: ``parser.parseExpression("'abc'.substring(1, 3)")``

Operators: ``parser.parseExpression("2 == 2")``

Types: ``parser.parseExpression(
        "'xyz' instanceof T(Integer)")``

Constructors: ``parseExpression(
        "new org.spring.samples.spel.inventor.Inventor('Albert Einstein', 'German')")``

Variables:  ``context.setVariable("newName", "Mike Tesla"); parser.parseExpression("name = #newName")``

Functions: ``context.setVariable("myFunction", method)``

Bean References: ``parser.parseExpression("@something")``

Ternary Operator (If-Then-Else): ``parseExpression(
        "false ? 'trueExp' : 'falseExp'")``

The Elvis Operator: ``parseExpression("name?:'Unknown'")``

Safe Navigation Operator: ``parseExpression("placeOfBirth?.city")``

Collection Selection: ``parser.parseExpression(
        "members.?[nationality == 'Serbian']")`` return a new collections that contains a subset of original element

Collection Projection: ``parser.parseExpression("members.![placeOfBirth.city]")`` return a list all value of ``placeOfBirth.city`` property.

Expression templating: Mixing literal text with one or more evaluation blocks ``parseExpression(
        "random number is #{T(java.lang.Math).random()}",
        new TemplateParserContext())``