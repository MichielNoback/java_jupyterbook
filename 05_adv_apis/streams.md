# The Streams API

The Java stream API is constituted by a wonderful set of classes and methods that together make it possible to process, mutate, filter and collect (reduce) data(-objects) in a streaming fashion. By making use of parallel streams this is even possible in a multi-core manner. 

Streams in general have these three components:

1. **Stream creation**: One of several ways to instantiate a stream. This can be out of a normal collection such as a list, or from a file, or on demand.
2. **Stream processing**: filtering and mapping operations performed on each element (=object) in the stream. The stream will not end, but generate a new stream with the new or filtered objects.
3. **Stream termination**: All streams must end. This can be through writing each object to a new file, but usually you will collect 
the results in a new (reduced) form.

Here is a small example demonstrating all three components. 

```java
List<Integer> numbers = List.of(2,3,4,5,6,7);

List<String> messages = numbers.
        stream().
        map(n -> n * n).                    // squared
        filter(n -> n % 2 == 0).            // only even numbers pass
        map(n -> "an even square: " + n).   // map to a string object
        collect(Collectors.toList());       // collect into a list
```

gives 


<pre class="console_out">
an even square: 4
an even square: 16
an even square: 36
</pre>

or, if you are not interested in collecting the result

```java
List<Integer> numbers = List.of(2,3,4,5,6,7);

numbers.
        parallelStream().                    // PARALLEL!
        map(n -> n * n).                     // squared
        filter(n -> n % 2 == 0).             // only even numbers pass
        map(n -> "an even square: " + n).    // map to a string object
        forEach(x -> System.out.println(x)); // print each object
```

gives 

<pre class="console_out">
an even square: 36
an even square: 16
an even square: 4
</pre>

As you can see, you can build a whole workflow in just a single statement. Note that in the second example a parallel stream is used, making the workflow more efficient (in theory), but not reproducible in the order in which objects will be processed.


## Stream creation

A stream needs a data source. 
In this section, three means to create streams are discussed.

### From collections

### From files


### On demand

Stream.generate(new Supplier() {})



## Intermediate stream operations



## Terminal Stream operations







<pre class="console_out">
SimpleWorker 'My First Worker' doing my thing
</pre>
