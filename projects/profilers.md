# Java sampling and tracing profilers

## Goal

Your goal in this project is to build two different kinds of profilers. The first profiler is a sampling profiler that wakes up at regular intervals and asks Java where it is. The 2nd profiler is called a tracing profiler and it relies on snippets of code the start and stop of every method to track how long the method takes and how often it's called. See profiling.

## Sampling profiler

You must create a Java agent with a premain method that creates a sampling thread that wakes up every 1ms. At each clock tick, it must figure out the method in which the main thread is currently executing. To do that, it must look at the stack trace of the main thread and filter out anything on the top of the stack that is in the Java library. To get that stack trace, it must find the main thread. Here is some [code that finds all the threads in the system](http://www.coderanch.com/how-to/java/ThreadLister). You can learn more about [java agents](http://dhruba.name/2010/02/07/creation-dynamic-loading-and-instrumentation-with-javaagents/). You might find [java.lang.instrument](http://docs.oracle.com/javase/7/docs/api/java/lang/instrument/package-summary.html) api useful.

You need to count how many times your sampler awakens in each method. From that, we can determine a relative ratio of how much time is spent in each method. We won't know the amount of time per se, but we know relatively speaking which ones take the most time because we awaken most often in. I suggest you do something like the following to track:

```java
FrequencySet<String> samples = new FrequencySet<String>();
```

ANTLR's tool lib has [FrequencySet.java](https://github.com/antlr/antlr4/blob/master/tool/src/org/antlr/v4/misc/FrequencySet.java) and a [MutableInt.java](https://github.com/antlr/antlr4/blob/master/tool/src/org/antlr/v4/misc/MutableInt.java) as a convenience.

In order to print out the profiling information, you will need the following method: Runtime.addShutdownHook(). You can use that to execute code as the JVM is shutting down.

Here's a basic outline of your thread.

```java
public class SamplerThread implements Runnable {
	public void run() {
		while ( true ) {
			// find main thread (running main())
			// get its stack trace
			// ignore anything in trace from java.* or sun.* or com.* or javax.* or $*
			// and find the topmost trace element.
			// That is the current location in main code
			// count how many times you reach that method
			...
			Thread.sleep(1); // wake up about every 1ms
		}
	}
}
```

Obviously this thing is not super accurate, but it gives a pretty decent idea of which methods take the most time.

You should print out profiling information from most frequently to least frequently-visited method and include the count on the right:

```
pkg.foo 32
pkg.bar 29
another.pkg.main 8
...
```

You can't track the file/line with the method on top of stack trace otherwise it won't group methods together properly.

## Tracing profiler

A tracing profiler injects code the start of stop of every method that tracks how long each method invocation takes. It also counts how many times the method is called. Do not try to record an entry for every method invocation. That would be too slow and cost too much memory. Instead, compute the amount of time and add that to a record for each pkg.class.method. Also count the number of invocations for each method. Then, again with a shutdown hook, you can collate the information and generate a nice little report.

Note that you should use the most accurate timer Java has:

```java
long startTime = System.nanoTime();
...
long estimatedTime = System.nanoTime() - startTime;
```

In order to do the bytecode transformations, you need to read [ASM transformations](http://asm.ow2.org/current/asm-transformations.pdf). Also you should use ASMifier so you don't have to figure out all the library method calls. Still, I found getting this thing to work kind of tricky. ASM's use of visitors to visit and to generate I found to be profoundly unintuitive.

To ensure that your "method exit" code actually executes, you must enclose the method in a try-finally:

```java
try {
   ... original method code ...
}
finally {
    long estimatedTime = System.nanoTime() - startTime;
    ...
}
```

The Java code could execute a return instruction anywhere within the method. This will force your code to execute.

As part of your tracing profiler, you must also track the number of objects of each type that are created. In this case, it makes sense to instrument java.* classes so that you pick up things like String. That means, for each new X(...) expression in any method, you should add code to increment a count for that particular object. This will be tricky because you must make sure that the code you insert does not mess up the expression surrounding the new. For example, if you see new X().go(), your instrumentation code will either have to go in front of the expression or after the new finishes but before the go method call. You could also do the count after go(), but it would change the order in which you record things since that method call could create other objects.

You will need a map from pkg.class -> count of objects.

You should be careful that you don't slow down the host program too much by injecting really expensive code into the methods. Do as much processing at shutdown, rather than on the fly, as you can.

Your report should be include the package / class name with the invocation count as with the sampling profiler. But, you should also include the total amount of time spent in that method in milliseconds. Sort by the amount of time spent in the method in descending order (do not sort by the invocation count):

```
pkg.bar 1932ms 29
pkg.foo 988ms 32
another.pkg.main 20ms 8
...
```

Follow that report with another right afterwards that shows the number of objects created of each type:

```
org.jdom.Attribute 523416
char[] 356059
...
java.lang.String 23009
...
```

## Java agents

All of our work will be done using agents so that our profilers are as unobtrusive as possible. Agents make it possible to instrument or observe programs during execution without having to modify those programs and without having source code for them. For example, here is how I launched my sampling profiler on class SomeTestProgram:

```bash
$ java -javaagent:sampler.jar SomeTestProgram
```

That sampler.jar has to have a manifest file that indicates what the agent class is. Since we are using premain() method, we need to specify Premain-Class:

```bash
$ cat META-INF/MANIFEST.MF 
Premain-Class: tracer.TestAgent
```

When you jar up your code, make sure that you include the META-INF directory and the manifest file underneath.

## Deliverables

Please put your code in package `sampler` and `tracer`.

* sampler.jar. For consistency, please call your agent class SamplerAgent.
* tracer.jar. Please call your agent TracerAgent.

Here is precisely how we will run your software:

```bash
java -javaagent:sampler.jar SomeTestProgram
java -javaagent:tracer.jar SomeTestProgram
```

where SomeTestProgram must have a main method. The agent from the jars, identified by the Premain-Class property in the manifest file, will execute before the SomeTestProgram main method runs. This gives your agent an opportunity to rewrite the bytecodes or do whatever else you need to initialize your system.

## Submission


## Resources

```java
package sampler;

import java.util.Hashtable;

/** Count how many of each key we have; not thread safe */
public class FrequencySet<T> extends Hashtable<T, MutableInt> {
	public int count(T key) {
		MutableInt value = get(key);
		if (value == null) return 0;
		return value.v;
	}
	public void add(T key) {
		MutableInt value = get(key);
		if (value == null) {
			value = new MutableInt(1);
			put(key, value);
		}
		else {
			value.v++;
		}
	}
}
```

```java
package sampler;

public class MutableInt extends Number implements Comparable<Number> {
	public int v;

	public MutableInt(int v) { this.v = v; }
	@Override public int compareTo(Number o) { return v; }
	@Override public int intValue() { return v; }
	@Override public long longValue() { return v; }
	@Override public float floatValue() { return v; }
	@Override public double doubleValue() { return v; }

	@Override
	public String toString() {
		return String.valueOf(v);
	}
}
```