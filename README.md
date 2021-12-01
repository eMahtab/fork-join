# Fork Join

The fork/join framework provides support for parallel programming by splitting up a task into smaller tasks to process them using the available CPU cores.

In fact, Java 8's parallel streams and the method Arrays#parallelSort use under the hood the fork/join framework to execute parallel tasks.

Applying a divide and conquer principle, the framework recursively divides the task into smaller subtasks until a given threshold is reached. This is the fork part.

Then, the subtasks are processed independently and if they return a result, all the results are recursively combined into a single result. This is the join part.


![Fork Join](fork-join.gif?raw=true)


To execute the tasks in parallel, the framework uses a pool of threads, with a number of threads equal to the number of processors available to the Java Virtual Machine (JVM) by default.

**💥💥💥 Note :**

Each thread has its own double-ended queue (deque) to store the tasks that will execute.

A deque is a type of queue that supports adding or removing elements from either the front (head) or the back (tail). This allows two things:

A thread can execute only one task at a time (the task at the head of its deque).
A work-stealing algorithm s implemented to balance the thread’s workload.
With the work-stealing algorithm, threads that run out of tasks to process can steal tasks from other threads that are still busy (by removing tasks from the tail of their deque).



# References :
https://www.pluralsight.com/guides/introduction-to-the-fork-join-framework

