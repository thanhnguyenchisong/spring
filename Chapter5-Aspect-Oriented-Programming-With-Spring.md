**Short name is AOP - complements  OOP. It is  programming paradigm with the goal increasing modularity by allowing the separation of cross-cutting concerns**, about the concepts: adding additional behavior to existing code without mofigying the code itself

OOP key unit of modularity is class

AOP key unit of modularity is the aspect.

# 1. AOP Concepts
Central AOP concepts and terminology
- **Aspect**: a modularization(mô đun hóa) of a concern that cuts cross multiple classes (Transaction management í a good example)
- ``Join point``: represents a method execution (execution of method)
- ``Advice``: actions at a particular ``join point`` include ``around``, ``before``, ``after``
- ``Pointcut``: A predicate that matchs join points
- ``Introduction``: Declaring additional method or fields on behalf of type.
- ``Target object``: object be advised by one or more aspect
- ``AOP proxy``: a object is created by the AOP framework in order to implmenet the aspect contracts.
- ``Weaving``: linking aspects with other application types or object to create an advised object.


**Types of advice**
- Before advice: Runs before a join point (can not prevent execution unless it throws a exception)
- After returning advice: After join point completes normally (without throwing an exception)
- After throwing advice: run when method throw exception.
- After(finally) advice: Run when the join point exits (normal or exception return)
- Around  advice: surrounds a join point as method invocation, can perform custom behavior before and after the method invovation.
- Some advices can support for a target so recommendation from Spring - use least powerfull advice type. If you just need to update a cache with the runturn value then choose after return advice, it is better than around advice, althrough an around advice can accomplish a same thing, it can less the protential for errors.

# 2. Spring AOP Capabilities and Goals.
- Supports only method execution joint point, if you need support for field acccess and update join points, consider language such as AspectJ.
- Spring AOP's approach to AOP differs from that of most other AOP frameworks. It doesn't provide the most complete AOP implementation (althrough Spring AOP is quite caable). Rather, providing a close interation between AOP implemenation and Spring IOC to help resolve sommon problems in Enterprise Application.
- Aspect are configured by using normal bean defination syntax.
- Sping semplessly intergrates Sring AOP and IoC with AspectJ to enable all uses of AOP in consitent Spring-based application architecture. It can work good with Spring AOP remains backward-compatible.

- One of central tenets of Spring Framework is that non-invasiveness, so you can choice of AspectJ, SpringAOP or both, using @AspectJ annotation-stype or Spring XML-stype approach,

# 3. AOP Proxies
- Default to using standard JDK dynamic procies for AOP proxies, enabling any interface or set of interfaces to be proxied.
- Sring AOP can also use CGLIB proxies, that is nessary to proxy classes rather than interfaces, CGLIB is used for bussiness object which isn't implemented interface but normally, for good practices, the class always implement a interface, you can force use this kind of proxy but don't recommend.
- Spring AOP is proxy-based.

# 4. Ennabling @AspectJ Support
To use @AspectJ aspects in a Spring configuration. Can use XML
```xml
<aop:aspectj-autoproxy/>
```
or Java-style configuration.

```java
@Configuration
@EnableAspectJAutoProxy
public class AppConfig {
}
```

## 4.1 Declaring an Aspect
```xml
<bean id="myAspect" class="com.xyz.NotVeryUsefulAspect">
    <!-- configure properties of the aspect here -->
</bean>
```
```java
@Aspect
public class NotVeryUsefulAspect {
}
```
## 4.2 Declaring a Pointcut
```java
@Pointcut("execution(* transfer(..))") // the pointcut expression
private void anyOldTransfer() {} // the pointcut signature
```
**Pointcut designators (PCD) for use pointcut expression**
- execution: matching method execution join points.
- within: Limit matching to join points within certain types.
- this: Limit matching to joint points where the bean reference is an instance of given type.
- target: Limits matching of joint points where the targte object is an instance of given type.
- args: ... where argument are instance of given types
- @target: ... where class of executing object has an annotation of given type
- @args: ... where runtime type of actual arguments passed have annotations of given types.
- @within: Limits matching to join points within types that have the given annotation.
- @annotation: ... where the subject of join point have given annotation. 
- bean: .... a particular named Spring bean or set of named Spring beans (supported on Spring AOP only)

**COmbining Pointcut Expressions**

&&, ||, !
```java
package com.xyz;
@Aspect
public class Pointcuts {
    @Pointcut("execution(public * *(..))")
    public void publicMethod() {} 
    @Pointcut("within(com.xyz.trading..*)")
    public void inTrading() {} 
    @Pointcut("publicMethod() && inTrading()")
    public void tradingOperation() {} 
}
```

Some common pointcut expressions:
- The execution of any public method:
``execution(public * *(..))``
- The execution of any method with a name that begins with set: ``execution(* set*(..))``
- The execution of any method defined by the AccountService interface:
``execution(* com.xyz.service.AccountService.*(..))``
- The execution of any method defined in the service package:
``execution(* com.xyz.service.*.*(..))``
- The execution of any method defined in the service package or one of its sub-packages:

``execution(* com.xyz.service..*.*(..))``
- Any join point (method execution only in Spring AOP) within the service package:
``within(com.xyz.service.*)``
- Any join point (method execution only in Spring AOP) within the service package or one of its sub-packages:
``within(com.xyz.service..*)``
- Any join point (method execution only in Spring AOP) where the proxy implements the AccountService interface:
``this(com.xyz.service.AccountService)``

**Good pointcuts**
- Particular kind of join point: execution, get, set, call, handler
- Scope: within, withincode
- Contextual: this, target, @annocation

With contexttiual will take alot time and memory so It should is on of 2 first kind of pointcut.

## 4.3 Advice
**Before advice**
@Before with in side is pointcut expression
```java
@Aspect
public class BeforeExample {
    @Before("execution(* com.xyz.dao.*.*(..))")
    public void doAccessCheck() {
        // ...
    }
}
```
**After returning Advice**
```java
@Aspect
public class AfterReturningExample {

    @AfterReturning("execution(* com.xyz.dao.*.*(..))")
    public void doAccessCheck() {
        // ...
    }
}
```
If you need to access to return value:
```java
@Aspect
public class AfterReturningExample {

    @AfterReturning(
        pointcut="execution(* com.xyz.dao.*.*(..))",
        returning="retVal")
    public void doAccessCheck(Object retVal) {
        // ...
    }
}
```
**After throwing**
You can decide to use ``throwing="ex"`` or not depend on your bussiness
```java
@Aspect
public class AfterThrowingExample {

    @AfterThrowing(
        pointcut="execution(* com.xyz.dao.*.*(..))",
        throwing="ex")
    public void doRecoveryActions(DataAccessException ex) {
        // ...
    }
}
```
**After advice**
Use with @After annotation and pointcut expression

**Around Advice**
It will work both before and after. used if you need share the state before and after a method execution in thread-safe manner-Ex stop and start a timmer.

``@Around`` annotation.
if the method is void, null will be return for ``peoceed()`` method

### **Advice Parameter**
**Default we have ``JoinPoint`` object with some useful method**
- getArgs(): return method arguments
- getThis(): return proxy object
- getTarget(): target object.
- getSignature(): desciption of method being advised.
- toString: print desciption of method being advied.
JUST NEED TO REMEMBER IT


** Passing Parameters to Advice**
```java
@Pointcut("execution(* com.xyz.dao.*.*(..)) && args(account,..)")
private void accountDataAccessOperation(Account account) {}

@Before("accountDataAccessOperation(account)")
public void validateAccount(Account account) {
    // ...
}
```
More example
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Auditable {
    AuditCode value();
}
```
```java
@Before("com.xyz.Pointcuts.publicMethod() && @annotation(auditable)") 
public void audit(Auditable auditable) {
    AuditCode code = auditable.value();
    // ...
}
```

- Advice parameters and Generics.
```java
public interface Sample<T> {
    void sampleGenericMethod(T param);
    void sampleGenericCollectionMethod(Collection<T> param);
}
```
You can retrict by your type
```java
@Before("execution(* ..Sample+.sampleGenericMethod(*)) && args(param)")
public void beforeSampleMethod(MyType param) {
    // Advice implementation
}
```
### Determining Argument Names
Don't like this because some the methos have differnce argument name.
- ``AspectJAnnotationParameterNameDiscoverer`` by ``argNames='bean, audit'`` atrribute
- ``AspectJAdviceParameterNameDiscoverer``  returning, throwing clauses.

** Proceeding with Arguments**
For around function because having  default ``ProceedingJoinPoint`` object
```java
@Around("execution(List<Account> find*(..)) && " +
        "com.xyz.CommonPointcuts.inDataAccessLayer() && " +
        "args(accountHolderNamePattern)") 
public Object preProcessQueryPattern(ProceedingJoinPoint pjp,
        String accountHolderNamePattern) throws Throwable {
    String newPattern = preProcess(accountHolderNamePattern);
    return pjp.proceed(new Object[] {newPattern});
}
```


# Advice Ordering
Multiple pices of advices running at the same join point so we have order advice execution.

The default - the order is undefined. but highest precedence advice run first for before and run last for after advice.

For clearly we can specify precedence by ``@Ordered`` from org.``springframework.core.Ordered``
For differnce advices the
highest to lowest precedence: ``@Around, @Before, @After, @AfterReturning, @AfterThrowing``.

# 4. Introductions
@DeclareParents  annotation is used do declare the matching type have new parent.
Example:
```java
@Aspect
public class UsageTracking {

    @DeclareParents(value="com.xyz.service.*+", defaultImpl=DefaultUsageTracked.class)
    public static UsageTracked mixin;

    @Before("execution(* com.xyz..service.*.*(..)) && this(usageTracked)")
    public void recordUsage(UsageTracked usageTracked) {
        usageTracked.incrementUseCount();
    }

}
```

Matching any bean of matching type implementation the UsageTracked

# 5. Different Sping AOP and AspectJ
- Spring AOP aims to provide a simple AOP implementation across Spring IOC to resolve the common problems. It's not intended as a complete AOP solution
- AspectJ is original AOP technology which airms to provide complete AOP solution, robust but more complicated than Spring AOP.

## 5.1 Weaving
- Both AspectJ and Spring AOP uses the different type of weaving, affects performance and ease of use.

AspectJ makes use of three different types of weaving.
1.