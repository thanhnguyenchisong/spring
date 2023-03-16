# Logging
Since Spring Framework 5.0, Common Logging bridge implemented in spring-jcl module, the implementation checks for presence of Log4j 2.x API and SLFJ 1.7 API, and use the first one of those found as the loggin implementaion (if it see Log4j first will use it first). falling back to Java platform's core logging facilities (`JUL or java.util.logging`)

`private final Log log = LogFactory.getLog(getClass());`