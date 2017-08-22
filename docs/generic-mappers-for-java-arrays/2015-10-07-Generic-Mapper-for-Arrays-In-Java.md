# Generic Mapper for Arrays In Java

## What is a Mapper?

In functional programming, maps are used to apply a function to each element in a list and return the result list. For example, if you apply the function `f(x) = x . x` to the list `[1, 2, 3]`, then you get the result list [1,4,9]. We will call a class that does this job in Java an Array or Collection Mapper.

In more general terms, functions in the mathematical sense are mappers because they map from one space (input space or domain) to another (output space or domain). For more on the subject, see [Higher Order Functions on Wiki](http://en.wikipedia.org/wiki/Map_%28higher-order_function%29).

## Arrays =~ Lists In Java

In Java, arrays have list semantics, but they are not `List`s. They do not implement the List interface for example. Here, I present a mapper that will apply a function to all elements in an array, and return the array result.

## Why Map Arrays?

It makes for a better code in terms of readability. It makes for a far more concise and terse coding structures. It also allows you to think more abstractly about what you are trying to apply on each element (without thinking about the loops), then at will decide to apply that to lists, arrays, sets, or whatever.

So, first, here is the mapper that does the deed. The class takes the output class and a single item mapper:

```java
package uk.co.codebreeze.arraymapper;

import java.lang.reflect.Array;

public class ArrayMapper<I,O> implements Mapper<I[], O[]> {
    private final Mapper<I, O> itemMapper;
    private final Class oClass;

    public ArrayMapper(final Mapper<I,O> itemMapper, final Class oClass){
        this.itemMapper = itemMapper;
        this.oClass = oClass;
    }

    public O[] map(final I[] input) {
        O[] output = (O[]) Array.newInstance(oClass, input.length);
        for(int i = 0; i < input.length; i++){
            output[i] = itemMapper.map(input[i]);
        }
        return output;
    }
}
```

The following code is a supporting interface. The `Mapper` implemented by both the single item mapper provided to the array mapper constructor above, and the generic array mapper itself:

```java
package uk.co.codebreeze.arraymapper;

public interface Mapper<I,O> {
    O map(I i);
}
```

Here is also a test to show how you may use the mapper, and prove it works as expected. Note how the separation of concerns works. We defined the string to integer conversion/mapping as a separate code, then, we could apply it to an array.

```java
package uk.co.codebreeze.arraymapper;

import static org.junit.Assert.*;

import java.lang.reflect.Array;
import org.junit.Test;

public class ArrayMapperTest {

    @Test
    public void testMap() {
        //given
        final Mapper<String, Integer> string2Integer = new Mapper<String, Integer>() {
            public Integer map(String i) {
                return Integer.valueOf(i);
            }
        };

        final String[] strings = new String[]{"1", "2", "4", "66"};

        //when
        final Mapper<String[], Integer[]> am = new ArrayMapper<String, Integer>(string2Integer, Integer.class);

        //then
        final Integer[] result = am.map(strings);
        assertArrayEquals(new Integer[]{1, 2, 4, 66}, result);
        assertFalse(Array.newInstance(Integer.class, 0).equals(Array.newInstance(Integer.class, 0)));
        assertFalse(Array.newInstance(Integer.class, 1).equals(Array.newInstance(Integer.class, 1)));
    }
}
```

As always, constructive feedback is always welcome.
