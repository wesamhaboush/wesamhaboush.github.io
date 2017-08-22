# Project Euler Problem 29 : Distinct Powers

## Introduction

I have been lately playing with Project Euler, and after few weeks, I hit problem 29. I had no idea how to solve it to start with, but after much thinking, searching, reading, and learning, I made two solutions that are NOT brute force.

The problem statement can be found here: [Euler Project Problem 29](https://projecteuler.net/problem=29)

My understanding of the problem and the nature of the project is that we are trying to not use brute force solutions (even if they are feasible), but rather use the problem statements to learn about number theory, and other clever ways to optimize the `O(n)` of the problem to a minimum in terms of both time and space.

To start with, I tried to find a clever solution myself. After few days of trying, I started focusing on the basic solution strategy: simply find the number of bases and powers that are a unique combination, i.e. all unique `(a,b)` ordered pairs, and then remove the duplicates (extra equivalent ones). But how? go google-anza!

Google told me that everybody seem to give up and use the brute force solution. I am despaired. I do NOT like it. 


## The Storage Algorithm

After couple of nights and continuous thinking and more importantly someone mentioning represeting unmashable values as vectors of parameters - i.e. do not calculate `100^100`, just deal with it as `(100,100)`, I thought about representing the unique base-power combinations as ordered pairs.

I also understood from my research that all numbers can be simplified to a basic base and power combinations form. For example, `2^3` cannot be simplified any further, but `4^9` can be simplified into `2^18`, also `9^10` to `3^20`, etc. So those combinations are equivalent or duplicates- e.g. `(9,10)` and `(3,20)`.

Next, I learnt that there are two types of base a in the `(a,b)` combinations: 1) perfect powers 2) non-perfect powers. For non-perfect powers, they cannot be simplified any further. Non-perfect power are numbers that cannot be expressed as `x^z`, e.g. `6`, `13`, `18`. On the other hand, perfect powers bases can be, e.g. `9 = 3^2`, `1000 = 10^3`, `64 = 4^4`. How does this help? ok, so it turns out that combinations that are made of non-perfect power a's are non-decomposable or are irreduceable, i.e., we need not process them, just use them as is. For ordered pairs with perfect power as, you need to find `x` and `y`, where `x^y = a`.

Note, that perfect numbers are far inbetween, therefore processing them is not an `O(n)` situation. 

So, the next step is storing all the unique items in a data structure. I decided to use a map of bases to their associated powers, instead of a set of ordered pairs, although both would work and effectively fulfill the `Set` semantics. The reason is that this will reduce storage space. So in stead of the memory holding: `(2,2),(2,3),(2,4)`, it holds `(2,(2,3,4))`.

So, finally what is the complexity of the unique values storage algorithm? the high level algorithm works out `O(n^2)`, however, we never have to calculate any large powers, and also this solution is generic and should work with larger values from those requested in Project Euler Problem 29.

Here is the pseudo code of the algorithm, followed by a java implementation:

```
 distinct-powers(min, max)
   set = []
   for base in min to max
       if base is perfect power
           reduced-base, additional-power = reduce(base)
           for power in min to max
               set = set + (reduced-base, power * additional-power)
       else
           for power in min to max
               set = set + (base, power)
   return size-of set
```

What we are trying to do here is to simplify every number to its base `a^b`, by finding out if a is perfect power, then finding the smallest base and updating the power accordingly, this way we can identify duplicates. For example:

```
    9^2 = (3^2)^2 = 3 ^ 4
    8^2 = (2^3)^2 = 2 ^ 6
    4^3 = (2^2)^3 = 2 ^ 6
```

you can see that the last two are duplicates, while we did not know that earlier!
     
```java

    public static long calculateStoreReduced(final long min, final long max)
    {
        final Map<Long, Set<Long>> values = new HashMap<>();
        for(long i = min; i <= max; i++)
        {
            final Exponent perfectPower = findPerfectPower(i);
            if (perfectPower == null) // non perfect powers add unique combinations, so just add them
            {
                for (long j = min; j <= max; j++)
                {
                    if(values.get(i) == null)
                    {
                        values.put(i, new HashSet<>(singleton(j)));
                    }
                    else
                    {
                        values.get(i).add(j);
                    }
                }
            }
            else
            {
                for (long j = min; j <= max; j++)
                {
                    if(values.get(perfectPower.base) == null)
                    {
                        values.put(perfectPower.base, new HashSet<>(singleton(j * perfectPower.power)));
                    }
                    else
                    {
                        values.get(perfectPower.base).add(j * perfectPower.power);
                    }
                }
            }
        }
        return values.values() // sets of distinct powers associated with each basic base
                     .stream()
                     .mapToInt(Set::size) //count those distinct power set
                     .sum(); // sum those counts (sizes of power sets)
    }
```

Finally, this code took less than 100ms to find the correct answer to euler's problem 29.



## The Counting Algorithm

Having produced the set solution above, I was fairly satisfied with a non brute force solution, but I still really was itching to get another solution strategy working. I wanted to be able to count and identify duplicates, without storing all the combinations. I have read on some threads that you can actually count the duplicates by hand, and give the answer to euler's project. However, I felt I would like to be able to give a more generic solution, and computerize the process. I set out to do exactly that. It was not easy, but i think i got there. 

OK, so how do we count duplicates basically goes like that: we assume no duplicates, and find the number of all possible combinations `(min - max + 1)^2`, then we take out from that number, all the repeats that we identify by iterating through the combinations.

Here is a pseudocode, followed by a java implementation too:

```
	distinct-powers(min, max)
        max-count = square (max - min + 1)
        repeat-count = 0
        for base in min to max
        	if base is perfect power
            	reduced-base, additional-power = reduce(base)
                for power in min to max
                    new-power = power * additional-power
                    if new-power <= max
                        repeat-count = repeat-count + 1
                    else
                        x, y = find divisors of new-power such that x < power and y between min and max
                        if exist x, y
                            repeat-count = repeat-count + 1
         return max-count - repeat-count
```                        
                
```java
public static long calculateCounting(final long min, final long max)
    {
        long count = (max - min + 1) * (max - min + 1);
        long repeats = 0;
        for(long i = min; i <= max; i++)
        {
            final Exponent perfectPower = findPerfectPower(i);
            if (perfectPower == null) // non perfect powers add unique combinations, so just add them
            {
                //nothing to do, cz non perfect power cannot generate duplicates.
            }
            else
            {
                for(long j = min; j < max; j++)
                {
                    final long reducedCandidateB = perfectPower.power * j;
                    if(reducedCandidateB <= max)
                    {
                        //this is the simple case of: a^b = c^d^e = c^(d*e), where d * e < max
                        //this is always a repeat with c^ whatever.
                        repeats++;
                    }
                    else
                    {
                        //we want to find if this perfect power number i^j can be decomposed to another perfect power before it, focus on before it, i.e. a < current a
                        //if yes, then it is a repeat of that number. For example:
                        //when min,max = 2,8   then 4^6 and 8^4 are equivalent, but neither are a repeat of 2^12, because 12 is greater than the max 8
                        //so when we hit 4^6, it is decomposed into 2^2^6 = 2^12. Now, can we decompose it to a^b such that a < 4, i.e. current a,
                        //and with both b < max and > min? the answer is no (i tried and failed). so 4^6 is NOT a repeat.
                        //however, once we hit 8^4 = 2^3^4 = 2^12, we ask the same question, can we find 8^4 = a^b, such that a < 8 and > min,
                        // and b < max and b > min? the answer is yes. With 2^12, we can find divisors of 12 = 2, 3, 4, 6, 12, then try each as follows:
                        // b = 12/12 = 1 => fails cz b is less than minimum
                        // b = 12/6  = 2 => fails cz 6 is greater than 3 (the current decomposed power), which means it is not before it
                        // b = 12/4 = 3  => fails cz 4 is greater than 3 (the current decomposed power), which means it is not before it
                        // b = 12/3 = 4  => fails cz 3 is equal to 3 (the power we decomposed the current a, which is 8, to)
                        // b = 12/2 = 6  => succeeds cz 2 is less than 3 (so the base is another base before the current base), AND 6, which is b, is > min and < max.
                        // This indicates 8^4 is a repeat
                        // so, let's implement that
                        final Set<Long> candidateBDivisors = divisorsOf(reducedCandidateB, false, false);
                        final boolean isRepeatOfSomethingNonBasicBefore = candidateBDivisors
                                .stream()
                                .filter(divisor -> divisor < perfectPower.power)
                                .anyMatch(divisor -> {
                                    final long newB = reducedCandidateB / divisor;
                                    return newB >= min && newB <= max;
                                });
                        if(isRepeatOfSomethingNonBasicBefore)
                        {
                            repeats++;
                        }
                    }
                }
            }
        }
        return count - repeats;
    }
```


This solution also gives the same results as the previous one, but does not use any memory at all, it does not compute any power values, and also runs in `O(n^2)` time complexity.

I am more satisfied now having produced two non-brute force solutions, and having learnt so much about number theory that I did not know. Having said that, I cannot help but think, can we do better? can we do something with a `O(n)` time complexity? I am happy to be given pointers!

Finally, here is a link to the code on my github account

[Distinct Powers Solution in Java](https://raw.githubusercontent.com/wesamhaboush/algorithms/master/algorithms/src/main/java/com/codebreeze/algorithms/DistinctPowers.java)
