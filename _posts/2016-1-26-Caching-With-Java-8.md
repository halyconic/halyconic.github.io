---
layout: post
title: Caching with Java 8
category: Code
tags: [java, lambda]
---

When working with legacy code, or just co-workers with distinct programming styles, it can be difficult to refactor. However, to save resources, be it time or cpu cycles, it is often necessary. Often enterprise level code is convoluted, unconventional, and downright ugly.

Using Java, you likely have great IDE support and tooling. However, no amount of tooling can give you the power to manipulate parameter lists and modify execution strategies. You don't have the luxuries that languages like Haskell or Scheme provide. Still, we can apply principles found. Here are a few techniques you can use to implement caching in your projects.

## The Traditional Approach ##

Most Java developers have probably used the following code at some point in their careers. This approach isn’t specific to Java 8, so it has the advantage of backwards compatibility. We introduce a few classes in our example. **i** is an input to the computation you would like to cache, and **o** is the output, of the types **I** and **O** respectively. The input class I usually has an **equals()** and **hashcode()** function. **f** is the function we would like to cache. Here we have a cache that stores an input to the function **f**.

~~~java
    import java.util.Map;

    Map<I, O> cache;

    O o = cache.get(i);
    if (o == null) {
        o = f(i);
        cache.put(i, o);
    }
~~~

This approach is simple, but quickly becomes tedious boilerplate. We can pass the cache around easily enough, but using it requires four lines of code and we must have access to **f** wherever the cache exists. It would be wonderful if we could attach the cache directly to the function **f**.

## Memoization: The Simple Case ##

Memoization[^Memoization] is a the technique by which we cache computations automatically when calling functions. While not a language feature in Java (unlike other modern languages), we can still manually memoize functions on a case by case basis. If you are lucky, the function you would like to cache has simple, pure outputs and inputs. In that case, a simple utility function is all you need. Below we define a function that takes a function, and returns a new function with a cache attached. Repeated calls to this new function with an equivalent input will return the cached computation!

~~~java
    import java.util.Map;
    import java.util.function;
     
    public <I, O> Function<I, O> memoize(Function<I, O> f) {
        Map<I, O> cache = new HashMap<>();
        return input -> cache.computeIfAbsent(input, f);
    }
~~~

Optionally, you can customize your memoization function to fit your function signature, but this is less elegant and requires multiple memoize functions to support different numbers of parameters. Having one input and one output is preferable, as it lets your code be more composable[^composable].[^1]

## A Better Cache ##

If we want a thread safe implementation, we can simply use a **ConcurrentHashMap** instead.

~~~java
    public static <I, O> Function<I, O> concurrentMemoize(Function<I, O> f) {
        Map<I, O> cache = new ConcurrentHashMap<>();
        return input -> cache.computeIfAbsent(input, f);
    }
~~~

Note that using a **ConcurrentHashmap** won’t work for recursive calls (ie, a fibonacci function that calls itself), because it breaks the contract of the **ConcurrentHashmap** class, which states you cannot modify the hashmap while inside the **computeIfAbsent** function.

If we want to be fancy, we could even substitute our map for a library such as **ehcache**[^ehcache]. This grants access to features such as an eviction policy, to prevent our cache from growing too large.[^2]

## Once More, With State

Likely, the code you are working with is probably a lot messier than this. Sometimes your input contains complicated parameters with internal state, like logs or database connections. These parameters shouldn't affect the output of your function. At the very least, we want to pretend they don't. However, there's no great way to check whether these two inputs are equal. Sure, you could craft the **equals** function on your input class to ignore these parameters. This feels disingenuous to me, since redefining equality can have undesirable side effects elsewhere in your code, and others may not expect there to be special logic in the **equals** function. Let's explore two other options that don’t feel so underhanded.
    
One option is to create a function that binds part of your parameter list to an environment. Then, we can pass in the environment alongside our function. We pass in the environment and our impure function, and return a pure function that has the environment bound (that's right, we used a **closure**[^closure]).
    
We use currying[^currying] to take our impure function and return a function with the impure part of the parameter list bound.

~~~java
    public static <E, I, O> Function<I, O> memoizePartial(BiFunction<E, I, O> f, E e) {
        Map<I, O> lookup = new HashMap<>();
        Function<E, Function<I, O>> currier = a -> b -> f.apply(a, b);
        Function<I, O> curried = currier.apply(e);
        return input -> lookup.computeIfAbsent(input, curried);
    }
~~~

## Objectively Better ##

But wait! This is Java! Aren’t closures supposed to be the poor man’s object? That's right, we can do things the way the OOP gods intended. Here we create a class **C** that binds the environment. The class exposes a function **f** that we can then memoize.

~~~java
    C {
        private E e;
        
        public C(E e) {
            this.e = e;
        }
        
        public O f(I i) {
            // work, using e
        }
    }
    
    C c = new C(e);
    Function<I, O> f = memoize(c::f);
~~~

Of course we can embed the cache into the class directly. This forces you to generate boilerplate code associated with the cache. However, If you want to pass around a wrapper object instead of a function, this option is available to you.

~~~java
    C {
        private E e;
        private Map<I, O> cache;
        
        public C(E e) {
            this.e = e;
            this.cache = new HashMap<>();
        }
        
        public O f(I i) {
            return cache.computeIfAbsent(i, this::compute);
        }
        
        private O compute(I i) {
            // work
        }
    }
~~~

## Exceptional Circumstances ##

Unfortunately, **computeIfAbsent** doesn't work with checked exceptions. There is no good way to do this. This is the same reason why checked exceptions don't mesh well with streams. The most straightforward way is to handle checked exceptions, but you can also use wrapper classes for your checked exceptions that extend **RuntimeException**.

For other approaches and more explanation, consider Lombok's sneakyThrow support[^Lombok's sneakyThrow support] or one of these approaches[^approaches].

## Conclusion ##

If you are lucky enough to work on a section of code without checked exceptions, consider caching with a simple memoize function. It'll save you lots of pain, and make refactoring easy.

## References ##

[^Memoization]: https://en.wikipedia.org/wiki/Memoization

[^composable]: http://programmers.stackexchange.com/a/185592

[^1]: https://www.opencredo.com/2015/08/19/lambda-memoization-in-java-8/

[^2]: https://gist.github.com/timyates/7674005    

[^Lombok's sneakyThrow support]: https://projectlombok.org/features/SneakyThrows.html

[^approaches]: http://stackoverflow.com/a/27644392/1427261

[^closure]: https://en.wikipedia.org/wiki/Closure_%28computer_programming%29

[^ehcache]: http://www.ehcache.org/

[^currying]: http://stackoverflow.com/questions/36314/what-is-currying