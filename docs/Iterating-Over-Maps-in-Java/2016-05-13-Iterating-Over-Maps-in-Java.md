# Iterating Over Maps in Java

In a great article titled [Iterating Java Map Entries](https://dzone.com/articles/iterating-java-map-entries), the brilliant John Thompson offers a clean way to iterate over java maps.

I kindly here offer my own preference on how this should be done, which is only mildly different from John's. I feel that leaking the mutable list outside the lambda during the iteration might be both conceptually, and practically hazardous. The style below is expression-base, more expressive, less error prone, and fairly concise.

I appreciate feedback.

```java
	@Test
    public void testMapIteration(){
        //given
        final Map<Integer, String> map = new HashMap<Integer, String>(){{
            put(1, "Java");
            put(2, "Groovy");
            put(3, "Scala");
            put(4, "Clojure");
            put(5, "jRuby");
        }};
        //when
        final List<String> jvmLangs1 = map
                .entrySet()
                .stream()
                .map(Map.Entry::getValue)
                .collect(toList());
        final List<String> jvmLangs2 = map
                .entrySet()
                .stream()
                .map(Map.Entry::getValue)
                .collect(toCollection(() -> new ArrayList<>()));
        //then
        assert jvmLangs1.size() == 5;
        assert jvmLangs2.size() == 5;
    }
```
   
Some smart people have actually pointed out that the code above is more obscure than John's. I actually completely agree. However, in real-world code, in my experience, the tables on average are turned. The leaking mutable `List` will cause bugs due to its wider scope. The above code will be a better approach in terms of easier to reason about, read, and maintain code. I guess ultimately there are always trade-offs. If obscurity is a big no-no in your context, absolutely prioritize eliminating it.

From a different angle also, the code above is more functional. For each should be for consumers, and NOT mutators. Think IO in the monadic sense. For example, a `System.out::println` or `LOGGER::info` or `DB::persist` are ideal candidates for `forEach` approach. 
