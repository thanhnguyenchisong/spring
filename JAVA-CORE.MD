### JDK, JRE, JVM
JDK include all thing to develop the Java application include JRE, JVM, documentation, compilers, Debugger, etc...
JRE java runtime environment provide the runtime environment to run a bytecode, Those are libraries of Java, JVM and other components for running the application
JVM Java Virtual Machine is a platform-independent abstract machine and execute the Java bytecode.

### Stack and Heap
Stack: Last IN, First OUT
Heap: it is a space in memory
Everyone method has own stack, basiclly, the stack have key value, with key is variable name and value is value of that variable.
The vaiable is not in method => no stay in stack -> so we have heap memory that is intance variable, method.

Situation: a object was created in method so the intance variable and method of that object stay in heap but the name and address stay in stack.
So we have the relation bw stack and heap as reference
So a refernce object stays in heap.

### Multi Dimensional Array
int num[][] = new int[3][4];
