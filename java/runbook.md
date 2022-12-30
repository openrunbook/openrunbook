# Java runbook

## How do I debug out of memory exceptions in Java?

Analyzing and fixing out-of-memory errors in Java is very simple.

In Java the objects that occupy memory are all linked to some other objects, forming a giant tree. The idea is to find the largest branches of the tree, which will usually point to a memory leak situation (in Java, you leak memory not when you forget to delete an object, but when you forget to forget the object, i.e. you keep a reference to it somewhere).

Step 1. Enable heap dumps at run time
Run your process with '-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/tmp'

(It is safe to have these options always enabled. Adjust the path as needed, it must be writable by the java user)

Step 2. Reproduce the error
Let the application run until the OutOfMemoryError occurs.

The JVM will automatically write a file like java_pid12345.hprof.

Step 3. Fetch the dump
Copy java_pid12345.hprof to your PC (it will be at least as big as your maximum heap size, so can get quite big - gzip it if necessary).

Step 4. Open the dump file with Heap Analyzer tool or Eclipse's Memory Analyzer, jhat, jvisualvm
The Heap Analyzer will present you with a tree of all objects that were alive at the time of the error. Chances are it will point you directly at the problem when it opens.

Note: give HeapAnalyzer enough memory, since it needs to load your entire dump!

java -Xmx10g -jar ha456.jar
Step 5. Identify areas of largest heap use
Browse through the tree of objects and identify objects that are kept around unnecessarily.

Note it can also happen that all of the objects are necessary, which would mean you need a larger heap. Size and tune the heap appropriately.

Step 6. Fix your code
Make sure to only keep objects around that you actually need. Remove items from collections in a timely manner. Make sure to not keep references to objects that are no longer needed, only then can they be garbage-collected.

[source](https://stackoverflow.com/questions/4512147/how-to-debug-java-outofmemory-exceptions)


