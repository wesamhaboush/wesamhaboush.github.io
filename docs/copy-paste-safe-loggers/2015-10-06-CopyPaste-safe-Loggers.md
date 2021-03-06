# Copy/Paste-safe Loggers


### Loggers Problem

Developers commonly create loggers for classes by copying logger creation lines from other classes. The process is so repetitive and error-prone that often they forget to update the line to reflect the actual class where the statement was pasted. This is compounded by the fact that IDEs normally auto-import the foreign class. We all saw classes that had the wrong logger like this:


```java
public class X {
 private static final Logger LOG = Logger.getLogger(Y.class);
}
```



In this post, I offer a solution that worked for me.

### Target Scheme For Obtaining Loggers


Ultimately, I would like to be able to do some thing like:

```java
public class X {
 private static final Logger LOG = createLogger();
}
```

It is almost like we need an annotation or injected/configured service a la Spring. In the absence of these I came up with an implementation that relies on static loading of classes to obtain the class name. Have a look:

```java
public class X {
 private static final Logger LOG = SLF4JLoggerFactory.INSTANCE.create();
}
```

copying the above code will simply work without needing to change anything on this line! it is essentially copy/paste safe!

### Implementation Details

The main components of this implementation are:

* The Factory interface is a simple interface that marks classes that make things.
* The enum SLF4JLoggerFactory implements the above interface with an output product of type Logger. You can use it with Slf4j or log4j (and possibly others).
* The Classes class has a couple of helper methods to find the class (or className) from the runtime static invocation of Logger creation invocation.

Here is the implementation of these components:

First the Factory:


```java
package uk.co.codebreeze.commons.lang;
 
public interface Factory<T> {
    T create();
}
```

Then the enum implementing the Factory interface:

```java
package uk.co.codebreeze.commons.lang;
 
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
 
public enum SLF4JLoggerFactory implements Factory<Logger> {
 
    INSTANCE;
 
    @Override
    public Logger create() {
        return LoggerFactory.getLogger(Classes.getInvokerClassOfCurrentMethod());
    }
}
```

Finally, the Classes helper class:

```java
package uk.co.codebreeze.commons.lang;
 
import java.util.List;
 
public class Classes {
  private Classes() {
  }
 
  public static Class<?> getInvokerClassOfCurrentMethod() {
    final Class<?>[] classes = new ClassesHelper().getClassContext();
    return classes[3];
  }
 
  private static final class ClassesHelper extends SecurityManager {
    @Override
    protected Class<?>[] getClassContext() {
      return super.getClassContext();
    }
  }
}
```

### Final Thoughts

I do not think this is the only or the best implementation. It is just an implementation that worked for me. I am sure many other engineers will be able to come up with other ideas.

I would recommend you put such implementation in a logger-extra module if you use maven, and make your projects pull it as a dependency, as opposed to embedding this code in every project.

As always, constructive feedback is welcome, and actually strongly appreciated.
